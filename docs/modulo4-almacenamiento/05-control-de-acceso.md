# 🔐 Control de acceso a datos en AWS y Azure

---

## 🗂️ Introducción

Una configuración segura del almacenamiento **no depende solo del cifrado**, sino también de controlar **quién puede acceder**, **cómo**, **desde dónde** y **durante cuánto tiempo**. El control de acceso debe ser:

* **Granular** (por usuario, aplicación, ruta o recurso).
* **Temporal** (reducir superficie de ataque).
* **Auditado** (registro de accesos y operaciones).
* **Aislado por red** (cuando sea posible, sin pasar por Internet).

Este archivo presenta las estrategias de control de acceso más efectivas en **AWS** y **Azure**.

---

## 🧩 Principios clave

| Requisito                      | Ejemplo AWS                  | Ejemplo Azure                        |
| ------------------------------ | ---------------------------- | ------------------------------------ |
| Identidad y permisos           | IAM Policies + Bucket Policy | RBAC + SAS                           |
| Condiciones de acceso          | IPs, VPC Endpoint, TLS       | Firewall, Private Endpoint           |
| Accesos temporales y delegados | STS, Presigned URLs          | SAS Tokens (expiración + revocación) |
| Aislamiento de red             | VPC Gateway Endpoint         | Private Endpoint                     |
| Prevención de acceso público   | Block Public Access (S3)     | Public Access = Disabled             |

---

## 🔐 Control de acceso en AWS

### ✅ 1. IAM + Bucket Policy (S3)

* **IAM Policies** → definen lo que una **identidad (user/role)** puede hacer.
* **Bucket Policies** → aplican a **recursos específicos** (buckets/objetos).
* **ACLs** → desaconsejadas, solo para compatibilidad heredada.

🔸 Recomendación: combinar IAM (autenticación) + Bucket Policy (condiciones, red, cifrado).

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::mi-bucket/*",
  "Condition": {
    "Bool": {
      "aws:SecureTransport": "false"
    }
  }
}
```

---

### ✅ 2. Acceso por red: VPC Endpoint

Permite acceso a S3, EFS, etc., **sin salir a Internet**. Ideal para workloads privados.

```bash
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-1234 \
  --service-name com.amazonaws.eu-west-1.s3 \
  --route-table-ids rtb-5678 \
  --vpc-endpoint-type Gateway
```

🔸 En la política puedes exigir que solo se permita acceso desde ese Endpoint:

```json
"Condition": {
  "StringEquals": {
    "aws:SourceVpce": "vpce-abc123"
  }
}
```

---

### ✅ 3. Accesos temporales: STS + Presigned URLs

* **STS (Security Token Service)**: genera credenciales **temporales** con permisos definidos.
* **Presigned URLs**: URLs firmadas para acceso temporal a objetos S3.

```python
import boto3
s3 = boto3.client('s3')
url = s3.generate_presigned_url('get_object',
    Params={'Bucket': 'mi-bucket', 'Key': 'archivo.pdf'},
    ExpiresIn=3600)
```

---

### ✅ 4. Access Points (S3)

Permiten definir **puntos de entrada separados por aplicación**. Cada uno puede tener su propia política.

```bash
aws s3control create-access-point \
  --account-id 123456789012 \
  --name miapp-accesspoint \
  --bucket mi-bucket
```

---

## 🔐 Control de acceso en Azure

### ✅ 1. Azure RBAC (con Azure AD)

* **Asignación de roles** a usuarios, grupos o apps (via AAD).
* Nivel de granularidad: cuenta de almacenamiento / blob container / file share.

```bash
az role assignment create \
  --assignee <obj-id> \
  --role "Storage Blob Data Contributor" \
  --scope "/subscriptions/.../resourceGroups/.../providers/Microsoft.Storage/storageAccounts/miaccount"
```

---

### ✅ 2. SAS (Shared Access Signatures)

Permiten compartir acceso temporal con terceros, sin necesidad de gestionar credenciales.

🔐 Tipos:

* **Account SAS**: acceso a múltiples servicios de la cuenta.
* **Service SAS**: acceso a recursos específicos (blob, file).
* **User Delegation SAS**: firmada con una identidad AAD → trazabilidad.

```bash
az storage blob generate-sas \
  --account-name miaccount \
  --container-name datos \
  --name archivo.txt \
  --permissions r \
  --expiry 2025-12-31T23:59Z \
  --as-user
```

> ⚠️ Nunca generes SAS sin expiración ni limites de IP/red.

---

### ✅ 3. Acceso restringido por red

* **Private Endpoints**: acceden a la cuenta sin pasar por Internet.
* **Firewall**: permite solo rangos IP o subredes (VNet).

```bash
az storage account network-rule add \
  --resource-group mi-rg \
  --account-name mistorage \
  --vnet-name mi-vnet \
  --subnet subnet-segura
```

---

### ✅ 4. Políticas de acceso público

Las cuentas nuevas deben tener el acceso público **deshabilitado**:

```bash
az storage account update \
  --name mistorage \
  --resource-group mi-rg \
  --default-action Deny
```

---

## 🏢 Ejemplo empresarial

**Escenario**: una startup de IA gestiona petabytes de imágenes y videos sensibles.

| Requisito                            | AWS                             | Azure                                          |
| ------------------------------------ | ------------------------------- | ---------------------------------------------- |
| Acceso de apps internas              | IAM roles + VPC Endpoint        | RBAC con AAD + Private Endpoint                |
| Compartir recurso temporal a cliente | Presigned URL con expiración 1h | SAS firmada con User Delegation Key            |
| Bloquear acceso desde fuera de red   | `aws:SourceVpce` en política    | Firewall + “Public Access = Deny” + vNet rules |
| Auditoría de accesos a datos         | CloudTrail Data Events          | Azure Monitor + Diagnostic Settings            |

---

## ❌ Errores comunes

1. **Usar ACLs o permisos implícitos** sin entender su alcance.
2. **Compartir SAS por WhatsApp o email sin revocación posible**.
3. **Dejar SAS sin expiración ni restricción de red**.
4. **No usar Access Points o RBAC**, y sobrecargar permisos en el bucket/cuenta.
5. **Permitir acceso público “temporalmente”**, y olvidarlo activo.
6. **No registrar los accesos** (logs desactivados o sin retención).

---

## ✅ Checklist rápido

| Control de acceso         | AWS                     | Azure                         |
| ------------------------- | ----------------------- | ----------------------------- |
| IAM / RBAC                | ✅                       | ✅                             |
| Políticas por recurso     | ✅ (Bucket policy)       | ✅ (RBAC a nivel de container) |
| Acceso delegado temporal  | ✅ (Presigned URL, STS)  | ✅ (SAS Token)                 |
| Red privada               | ✅ (VPC Endpoint)        | ✅ (Private Endpoint)          |
| Restricción por IP        | ✅                       | ✅                             |
| Bloqueo de acceso público | ✅ (Block Public Access) | ✅ (DefaultAction = Deny)      |
| Logs de acceso            | ✅ (CloudTrail)          | ✅ (Azure Monitor)             |

---

## 🔗 Recursos oficiales

* [AWS S3 Access Control](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-overview.html)
* [AWS VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html)
* [Azure Storage RBAC Guide](https://learn.microsoft.com/en-us/azure/storage/common/storage-auth-aad-rbac)
* [SAS Best Practices - Azure](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)
* [Azure Storage Firewalls and Networks](https://learn.microsoft.com/en-us/azure/storage/common/storage-network-security)