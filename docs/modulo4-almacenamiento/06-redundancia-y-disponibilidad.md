# ♻️ Redundancia, replicación y disponibilidad de datos (AWS y Azure)

---

## 🗂️ Introducción

En entornos cloud, la **alta disponibilidad** y la **resiliencia ante fallos o ataques** no se logran simplemente "haciendo backups", sino que requieren una **estrategia combinada de replicación, snapshots, versionado, inmutabilidad y recuperación**.

Este archivo cubre los principales mecanismos que ofrecen **AWS y Azure** para cumplir objetivos de recuperación (**RPO** / **RTO**) y proteger los datos ante fallos, ataques o errores humanos.

---

## 📈 Objetivos de recuperación: RPO y RTO

| Concepto                           | Definición                                                                       | Ejemplo práctico                       |
| ---------------------------------- | -------------------------------------------------------------------------------- | -------------------------------------- |
| **RPO** (Recovery Point Objective) | Máximo tiempo tolerable de pérdida de datos (frecuencia de backup o replicación) | “No perder más de 15 minutos de datos” |
| **RTO** (Recovery Time Objective)  | Tiempo máximo permitido para restaurar el sistema                                | “Volver a estar operativos en 2h”      |

Elegir la combinación adecuada de mecanismos depende de estos dos objetivos, y del **coste aceptable por downtime/pérdida**.

---

## 🔁 En AWS: versionado, replicación y snapshots

### ✅ S3 (objetos)

| Mecanismo       | Descripción                                                 |
| --------------- | ----------------------------------------------------------- |
| **Versioning**  | Guarda versiones sucesivas de un objeto (protección lógica) |
| **Object Lock** | Impide modificar/borrar versiones (modo WORM)               |
| **Replication** | Copia automática entre buckets (misma o distinta región)    |

🔸 Tipos de replicación:

* **SRR (Same Region Replication)**: dentro de la misma región.
* **CRR (Cross Region Replication)**: entre regiones distintas (para tolerancia regional).

```bash
aws s3api put-bucket-versioning --bucket mi-bucket --versioning-configuration Status=Enabled

aws s3api put-object-lock-configuration --bucket mi-bucket \
  --object-lock-configuration '{
    "ObjectLockEnabled": "Enabled",
    "Rule": {
      "DefaultRetention": {
        "Mode": "COMPLIANCE",
        "Days": 30
      }
    }
  }'
```

---

### ✅ EBS (bloques)

* **Snapshots** de volúmenes, cifrados automáticamente.
* Se pueden copiar a otra región o cuenta (**aislamiento** frente a compromiso de cuenta principal).

```bash
aws ec2 create-snapshot \
  --volume-id vol-abc123 \
  --description "Backup semanal cifrado"

aws ec2 copy-snapshot \
  --source-region eu-west-1 \
  --source-snapshot-id snap-xyz \
  --destination-region eu-central-1
```

🔒 Buenas prácticas:

* Cifrar todos los snapshots.
* Automatizar con **AWS Backup**.
* Usar cuentas separadas para almacenar backups sensibles.

---

### ✅ EFS (archivos)

* Soporta **replicación automática** entre regiones.
* Integración con **AWS Backup** para retención y restauración granular.

```bash
aws efs create-replication-configuration \
  --source-file-system-id fs-abc123 \
  --destinations "Region=eu-central-1"
```

---

## 🌍 En Azure: redundancia y copias automatizadas

### ✅ Tipos de redundancia (storage)

| Tipo       | Regiones | Descripción                                                 |
| ---------- | -------- | ----------------------------------------------------------- |
| **LRS**    | Local    | 3 copias en el mismo datacenter                             |
| **ZRS**    | Zonal    | Copias sincronizadas en 3 zonas (alta disponibilidad local) |
| **GRS**    | Global   | Copia secundaria en otra región (no accesible)              |
| **RA-GRS** | Global   | Igual que GRS, pero permite lectura en la región secundaria |

```bash
az storage account create \
  --name mistorage \
  --resource-group mi-rg \
  --sku Standard_ZRS
```

> 🔸 GRS/RA-GRS es ideal para escenarios de recuperación ante desastre.

---

### ✅ Blob Storage

* **Versioning** activable por contenedor: retiene múltiples versiones de objetos.
* **Soft Delete**: permite recuperar objetos borrados (hasta 365 días).
* **Lifecycle policies**: automatizan archivado, borrado o retención.

```bash
az storage blob service-properties update \
  --account-name mistorage \
  --enable-versioning true
```

---

### ✅ Managed Disks y Azure Files

* **Snapshots** incrementales para discos y compartidos.
* **Backup automático** mediante **Azure Backup Vault**, con políticas por workload.

```bash
az snapshot create \
  --resource-group mi-rg \
  --source "/subscriptions/.../discos/vm1-disk" \
  --name snap-vm1
```

---

### ✅ Azure Backup

* Políticas de retención por tiempo o por número de versiones.
* Vaults aislados para proteger los backups ante ransomware o borrados accidentales.
* Soporte para restauración granular.

```bash
az backup vault create \
  --name miVault \
  --resource-group mi-rg \
  --location westeurope
```

---

## 🏢 Ejemplo empresarial

**Escenario**: una fintech europea necesita proteger sus datos de transacciones y usuarios, garantizando:

* 24h/7 disponibilidad.
* 1h de RPO.
* Retención legal de 7 años.
* Protección ante borrado o ransomware.

| Requisito                       | AWS                                   | Azure                                          |
| ------------------------------- | ------------------------------------- | ---------------------------------------------- |
| Backups diarios cifrados        | AWS Backup (EBS, RDS, S3)             | Azure Backup Vault                             |
| Evitar pérdida por error humano | S3 Versioning + Object Lock (30 días) | Blob Versioning + Soft Delete                  |
| Redundancia geográfica          | CRR (S3) / Snapshot cross-region      | RA-GRS o backup en vault en otra región        |
| Cumplimiento legal              | Inmutabilidad + retención + auditoría | Immutability Policy + retention + Monitor logs |

---

## ❌ Errores comunes

1. **No activar versionado o soft delete**: datos perdidos por sobrescritura o eliminación accidental.
2. **Snapshots sin cifrar ni protegidos**: fácil compromiso si se filtran credenciales.
3. **No usar cuentas o vaults separados para backups**: expone backups al mismo vector de ataque.
4. **No definir políticas de retención** → borrados masivos sin recuperación.
5. **Confundir disponibilidad con recuperación**: LRS ≠ DR.

---

## ✅ Checklist rápido

| Mecanismo                            | AWS                      | Azure                    |
| ------------------------------------ | ------------------------ | ------------------------ |
| Versionado de objetos                | ✅ (S3)                   | ✅ (Blob)                 |
| Inmutabilidad                        | ✅ (Object Lock)          | ✅ (Immutability Policy)  |
| Snapshots                            | ✅ (EBS/EFS)              | ✅ (Disks/Files)          |
| Copia entre regiones                 | ✅ (CRR / copy-snapshot)  | ✅ (RA-GRS, Backup Vault) |
| Vaults aislados para backup          | ✅ (Cross-account backup) | ✅ (Azure Backup Vault)   |
| Automatización de backup y retención | ✅ (AWS Backup)           | ✅ (Azure Backup)         |

---

## 🔗 Recursos oficiales

* [AWS Backup Overview](https://docs.aws.amazon.com/aws-backup/latest/devguide/whatisbackup.html)
* [S3 Versioning and Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html)
* [Azure Blob Data Protection](https://learn.microsoft.com/en-us/azure/storage/blobs/data-protection-overview)
* [Azure Backup Documentation](https://learn.microsoft.com/en-us/azure/backup/backup-overview)