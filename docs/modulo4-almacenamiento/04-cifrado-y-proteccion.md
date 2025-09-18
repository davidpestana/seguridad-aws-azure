# 🔐 Cifrado en tránsito y en reposo (AWS y Azure)

---

## 🗂️ Introducción

En la nube, **el cifrado de datos no es opcional**, sino un pilar esencial de la seguridad. Las buenas prácticas actuales exigen:

* **Cifrado en tránsito**, para proteger los datos mientras se mueven entre cliente y servicio.
* **Cifrado en reposo**, para proteger los datos almacenados frente a acceso físico o lógico no autorizado.
* En ciertos sectores regulados, incluso se exige **doble cifrado** o control total sobre las claves (**CMK** / **BYOK**).

Este archivo te guía para configurar correctamente el cifrado en **AWS** y **Azure**, tanto en tránsito como en reposo.

---

## 📡 Cifrado en tránsito (TLS/HTTPS)

### ✅ ¿Qué es?

El cifrado en tránsito asegura que los datos **no puedan ser interceptados** durante su transferencia entre cliente ↔ servidor, mediante el uso de **TLS (Transport Layer Security)**.

### 🔒 Buenas prácticas

| Requisito                      | AWS                                       | Azure                                                |
| ------------------------------ | ----------------------------------------- | ---------------------------------------------------- |
| **Forzar HTTPS**               | Política `aws:SecureTransport`            | `Secure transfer required = true` en Storage Account |
| **TLS ≥ 1.2**                  | Configuración en clientes/SDKs            | Puede requerir ajuste en navegadores o apps legacy   |
| **Validación de certificados** | Evitar bypass de CA o ignorar errores SSL | Igual; usar certificados válidos y actualizados      |

---

### 🔧 Ejemplo: forzar HTTPS en AWS S3

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::mi-bucket",
        "arn:aws:s3:::mi-bucket/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

---

### 🔧 Ejemplo: forzar HTTPS en Azure

```bash
az storage account update \
  --name mistorage \
  --resource-group mi-rg \
  --enable-https-traffic-only true
```

---

## 💾 Cifrado en reposo (en almacenamiento)

### ✅ ¿Qué es?

El cifrado en reposo protege los datos almacenados en discos, objetos o archivos, usando claves simétricas (AES-256). Existen dos modelos:

| Modelo                     | ¿Quién gestiona la clave?     | Casos comunes                        |
| -------------------------- | ----------------------------- | ------------------------------------ |
| **Proveedor gestionado**   | AWS o Azure                   | Por defecto, sin configuración extra |
| **CMK / Customer Managed** | Cliente (con KMS o Key Vault) | Escenarios con control/auditoría     |

---

### 🔐 Tipos de cifrado en AWS

| Servicio                | Cifrado por defecto | Opción de CMK personalizada   |
| ----------------------- | ------------------- | ----------------------------- |
| **S3**                  | SSE-S3 (AES-256)    | SSE-KMS con clave del cliente |
| **EBS/EFS**             | KMS AES-256         | CMK gestionada por el cliente |
| **RDS, DynamoDB, etc.** | Sí                  | CMK opcional                  |

```bash
aws s3api put-bucket-encryption --bucket mi-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "arn:aws:kms:region:account-id:key/key-id"
      }
    }]
  }'
```

---

### 🔐 Tipos de cifrado en Azure

| Servicio          | Cifrado por defecto | Opción de CMK (Key Vault)        |
| ----------------- | ------------------- | -------------------------------- |
| **Blob/Files**    | Microsoft-managed   | Customer-managed con Key Vault   |
| **Managed Disks** | Microsoft-managed   | Disk Encryption Sets (DES + CMK) |

```bash
az storage account update \
  --name mistorage \
  --resource-group mi-rg \
  --encryption-key-source Microsoft.Keyvault \
  --keyvault-encryption-key-uri https://mi-kv.vault.azure.net/keys/clave/versión
```

---

## 🔁 Gestión de claves (KMS / Key Vault)

### 🎯 Por qué usar CMK (Customer Managed Keys)

* Control del **ciclo de vida**: creación, rotación y destrucción.
* Posibilidad de **revocar acceso** de forma inmediata.
* Configuración de **roles separados**:

  * Quién puede **usar** la clave.
  * Quién puede **administrar** la clave.
* **Auditoría detallada**: acceso a eventos de uso de la clave.

---

### 🔧 AWS KMS: ejemplo de política de uso

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowUseOfTheKey",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/app-role"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt"
      ],
      "Resource": "*"
    }
  ]
}
```

---

### 🔧 Azure Key Vault: configurar rol de acceso

```bash
az role assignment create \
  --assignee <app-id-o-usuario> \
  --role "Key Vault Crypto User" \
  --scope /subscriptions/.../resourceGroups/.../providers/Microsoft.KeyVault/vaults/mi-kv
```

> 💡 Puedes combinarlo con `Key Vault Access Policies` o `Azure RBAC` según necesidad.

---

## 🔄 Rotación de claves y cumplimiento

### 🧩 Claves rotadas = menos exposición

| Plataforma | Funcionalidad                    | Configuración                     |
| ---------- | -------------------------------- | --------------------------------- |
| **AWS**    | KMS con auto-rotación cada 1 año | Activar con `enable-key-rotation` |
| **Azure**  | Key Vault con `rotation policy`  | `rotation-enabled = true`         |

```bash
# AWS
aws kms enable-key-rotation --key-id <clave-id>

# Azure
az keyvault key rotation-policy update \
  --vault-name mi-kv \
  --name clave \
  --expiry-time "P2Y" \
  --lifetime-actions rotate
```

---

## 🏢 Ejemplo empresarial

**Escenario**: Una empresa de seguros almacena historiales médicos de clientes y los transmite a microservicios en backend.

| Requisito                          | Configuración recomendada                                              |
| ---------------------------------- | ---------------------------------------------------------------------- |
| Transmisión cifrada                | TLS 1.2+, HTTPS-only en almacenamiento y API Gateway                   |
| Almacenamiento cifrado con control | CMK en KMS (AWS) o Key Vault (Azure)                                   |
| Auditoría                          | Logs activados en KMS / Key Vault, integrados con CloudTrail / Monitor |
| Clave con rotación automática      | Política activada de rotación anual                                    |
| Separación de roles                | Unos usuarios administran claves, otros las usan                       |

---

## ❌ Errores comunes

1. **No forzar HTTPS** → tráfico legible por atacantes si red comprometida.
2. **Usar claves por defecto** del proveedor sin control ni auditoría.
3. **SAS o presigned URLs sin expiración**.
4. **No rotar claves** → mayor exposición si se filtran.
5. **No auditar el uso de claves** ni alertar accesos indebidos.
6. **Permitir que un mismo rol cree y use claves sin separación**.

---

## ✅ Checklist rápido

| Buenas prácticas                           | ✔️ |
| ------------------------------------------ | -- |
| HTTPS-only en almacenamiento               | ✅  |
| TLS ≥ 1.2 en todos los accesos             | ✅  |
| Cifrado en reposo activado                 | ✅  |
| Uso de CMK/Key Vault cuando se requiere    | ✅  |
| Claves con política de rotación activada   | ✅  |
| Separación de roles en uso/admin de claves | ✅  |
| Logs y auditoría de acceso a claves        | ✅  |

---

## 🔗 Recursos oficiales

* [AWS KMS Best Practices](https://docs.aws.amazon.com/kms/latest/developerguide/best-practices.html)
* [Azure Key Vault Overview](https://learn.microsoft.com/en-us/azure/key-vault/general/basic-concepts)
* [Azure Storage Encryption](https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption)
* [Encrypting Data at Rest in AWS](https://docs.aws.amazon.com/whitepapers/latest/encryption-at-rest/encryption-at-rest.html)
