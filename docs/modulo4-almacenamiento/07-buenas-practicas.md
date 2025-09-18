# ✅ Buenas prácticas en la gestión de almacenamiento seguro (AWS y Azure)

---

## 🗂️ Introducción

El almacenamiento en la nube **no es seguro por defecto**. Si bien los proveedores ofrecen herramientas potentes (cifrado, políticas, auditoría, etc.), **la responsabilidad de configurarlas correctamente recae en el cliente**.

Este archivo recopila las **buenas prácticas esenciales** para garantizar la **seguridad, sostenibilidad y cumplimiento normativo** del almacenamiento en **AWS y Azure**, combinando lo aprendido en los archivos anteriores.

---

## ✅ Checklist de buenas prácticas esenciales

### 🔐 1. Cifrado siempre activado

| Buenas prácticas                                        | AWS                                       | Azure                                |
| ------------------------------------------------------- | ----------------------------------------- | ------------------------------------ |
| Cifrado en reposo activado en todos los servicios       | ✅ (S3, EBS, EFS, etc.)                    | ✅ (Blob, Files, Disks)               |
| Preferir claves gestionadas por el cliente (CMK / BYOK) | ✅ (KMS con CMK)                           | ✅ (Key Vault + Disk Encryption Sets) |
| Cifrado en tránsito: HTTPS forzado                      | `aws:SecureTransport=true` en S3 policies | `--enable-https-traffic-only true`   |
| TLS ≥ 1.2 en clientes y APIs                            | Configurable en SDKs y balancers          | Obligatorio en Azure por defecto     |

---

### 🚫 2. Bloqueo de acceso público

| Acción recomendada                   | AWS                                                  | Azure                                           |
| ------------------------------------ | ---------------------------------------------------- | ----------------------------------------------- |
| Bloquear acceso público por defecto  | S3 Block Public Access (cuenta + bucket)             | `"Public access = Disabled"` en Storage Account |
| Revisar configuraciones “temporales” | Eliminar bucket policies amplias                     | Revisar SAS sin expiración o sin restricciones  |
| Automatizar detección                | AWS Config Rules: `s3-bucket-public-read-prohibited` | Azure Policy: `Audit Public Access`             |

---

### 📜 3. Políticas de acceso granular y auditado

| Buenas prácticas                 | AWS                              | Azure                                         |
| -------------------------------- | -------------------------------- | --------------------------------------------- |
| Usar **IAM + Bucket Policy**     | ✅ Reglas por identidad y recurso | N/A                                           |
| Usar **RBAC** a nivel de recurso | N/A                              | ✅ Roles por blob/file/container               |
| Restringir por IP / VPC / Subred | `aws:SourceIp`, `aws:SourceVpce` | Private Endpoint + Firewall                   |
| Accesos temporales y delegados   | STS + Presigned URLs             | SAS Tokens (firmados con User Delegation Key) |
| Activar logs de acceso           | S3 Access Logs / CloudTrail      | Azure Monitor + Diagnostic Settings           |

---

### ♻️ 4. Versionado, retención e inmutabilidad

| Medida de protección                  | AWS                           | Azure                                     |
| ------------------------------------- | ----------------------------- | ----------------------------------------- |
| Versionado de objetos                 | S3 Versioning                 | Blob Versioning                           |
| Inmutabilidad por tiempo o legal hold | S3 Object Lock                | Blob Immutability Policy                  |
| Snapshots programados                 | AWS Backup (EBS, RDS, EFS...) | Azure Backup + Policy                     |
| Vaults aislados o cross-account       | ✅                             | ✅ (Backup Vault en subscripción separada) |

---

### 📉 5. Optimización de costes sin sacrificar seguridad

| Estrategia                            | AWS                                      | Azure                 |
| ------------------------------------- | ---------------------------------------- | --------------------- |
| Políticas de **lifecycle automático** | S3 → IA / Glacier / Glacier Deep Archive | Blob → Cool / Archive |
| Cifrado y acceso se mantienen         | Sí                                       | Sí                    |
| Control de acceso y logs se heredan   | Sí, si se define correctamente           | Sí                    |

---

### 📡 6. Aislamiento de red

| Recomendación                          | AWS                             | Azure                 |
| -------------------------------------- | ------------------------------- | --------------------- |
| Acceso sin pasar por Internet          | VPC Gateway Endpoint (S3, etc.) | Private Endpoint      |
| Limitar acceso por IP o SG             | SG + `aws:SourceIp`             | Firewall + VNet rules |
| Bloquear acceso desde Internet público | Block Public Access             | DefaultAction = Deny  |

---

### 🧪 7. Auditoría continua

| Elemento                            | AWS                                      | Azure                               |
| ----------------------------------- | ---------------------------------------- | ----------------------------------- |
| Registro de operaciones             | AWS CloudTrail (data events + KMS usage) | Azure Monitor + Diagnostic Settings |
| Alertas de accesos inusuales        | Amazon GuardDuty / Macie                 | Microsoft Defender for Storage      |
| Revisión de configuración periódica | AWS Config / Security Hub                | Azure Policy / Microsoft Purview    |

---

## 🧠 Buenas prácticas adicionales

### 👥 Separación de roles

* Quien **usa** las claves no debe poder **gestionarlas**.
* Quien **crea buckets/discos** no debería poder modificarlos sin auditoría.
* Aplicaciones deben tener **permisos mínimos necesarios (PoLP)**.

---

### 🔐 SAS, URLs y accesos temporales

| Práctica recomendada                | AWS                           | Azure                       |
| ----------------------------------- | ----------------------------- | --------------------------- |
| URLs con expiración breve           | Presigned URL con `ExpiresIn` | SAS Token con `--expiry`    |
| Delegación firmada por identidad    | STS + Presigned por IAM       | SAS con User Delegation Key |
| Nunca compartir URLs sin expiración | ❌                             | ❌                           |

---

### 🧰 Herramientas útiles

| Funcionalidad                | AWS                       | Azure                         |
| ---------------------------- | ------------------------- | ----------------------------- |
| Detección de datos sensibles | Amazon Macie              | Microsoft Purview             |
| Escaneo de compliance        | AWS Config + Security Hub | Azure Policy + Blueprints     |
| Control de uso de claves     | AWS CloudTrail + KMS logs | Key Vault logging via Monitor |

---

## 🏢 Caso práctico resumen

**Escenario**: una consultora legal almacena contratos confidenciales para varios clientes.

* **Blob/S3 con versión + inmutabilidad** activada.
* **Cifrado con claves propias (CMK / Key Vault)** y rotación anual.
* **Acceso restringido por IP y por rol**, sin acceso desde Internet.
* **Backups en región secundaria, en vault aislado.**
* **Alertas ante accesos anómalos o cambios en configuración.**

---

## ❌ Errores frecuentes a evitar

1. **"Era solo una prueba"** → buckets públicos expuestos durante meses.
2. **SAS sin expiración ni IPs** → acceso abierto fuera de control.
3. **Claves sin rotación ni logs** → sin trazabilidad.
4. **Desactivar TLS “en entornos internos”** → expone datos por costumbre.
5. **Permisos excesivos a apps** → principio de menor privilegio ignorado.
6. **Sin versión ni soft delete** → cualquier borrado es irreversible.

---

## 🔗 Recursos oficiales

* [AWS Well-Architected: Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar)
* [Azure Storage Security Guide](https://learn.microsoft.com/en-us/azure/storage/common/storage-security-guide)
* [AWS Macie](https://aws.amazon.com/macie/)
* [Microsoft Purview Overview](https://learn.microsoft.com/en-us/azure/purview/overview)
