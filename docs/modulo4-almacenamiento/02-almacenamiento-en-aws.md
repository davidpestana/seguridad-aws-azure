# 🔐 Almacenamiento seguro en AWS: S3, EBS y EFS

---

## 🗂️ Introducción

Amazon Web Services ofrece tres soluciones principales de almacenamiento, cada una orientada a un tipo distinto de uso, rendimiento y arquitectura:

| Tipo         | Servicio | Uso principal                        |
| ------------ | -------- | ------------------------------------ |
| **Objetos**  | S3       | Almacenamiento masivo, no jerárquico |
| **Bloques**  | EBS      | Discos virtuales para instancias EC2 |
| **Archivos** | EFS      | Sistema de archivos compartido (NFS) |

Desde el punto de vista de la seguridad, cada servicio incluye mecanismos propios de **cifrado**, **acceso granular**, **aislamiento de red**, y **protección contra errores o ataques**. Este archivo detalla cómo aplicar buenas prácticas y configuraciones seguras en cada uno.

---

## 🪣 Amazon S3 (almacenamiento de objetos)

### ✅ Seguridad básica

* **Bloqueo de acceso público**:

  * A nivel **de cuenta** y **de bucket**: activa `Block Public Access` para prevenir exposición accidental.
  * Se recomienda usarlo **por defecto** en todo nuevo bucket.

```bash
aws s3api put-public-access-block \
  --bucket mi-bucket \
  --public-access-block-configuration 'BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true'
```

### 🔐 Cifrado en reposo

Tres opciones principales:

| Tipo                       | Gestión de clave | Servicio asociado                   |
| -------------------------- | ---------------- | ----------------------------------- |
| **SSE-S3**                 | AWS              | Transparente, sin control de clave  |
| **SSE-KMS**                | Cliente (CMK)    | Integrado con AWS KMS               |
| **Client-Side Encryption** | Cliente completo | No recomendado si no es obligatorio |

```bash
aws s3api put-bucket-encryption --bucket mi-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms",
        "KMSMasterKeyID": "arn:aws:kms:...:key/clave-id"
      }
    }]
  }'
```

### 👮 Control de acceso

| Mecanismo           | Uso recomendado           |
| ------------------- | ------------------------- |
| **IAM policies**    | Control por usuario/rol   |
| **Bucket policies** | Reglas a nivel de recurso |
| **ACLs**            | Obsoleto, evitar          |

> ✅ Buenas prácticas: combinar `Bucket Policy` para restricciones de red + `IAM Policy` para permisos por identidad.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowVPCEOnly",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::mi-bucket",
        "arn:aws:s3:::mi-bucket/*"
      ],
      "Condition": {
        "StringNotEquals": {
          "aws:SourceVpce": "vpce-xxxxxxxxxxxxx"
        }
      }
    }
  ]
}
```

### 🔁 Versionado e inmutabilidad

* **Versioning**: protege ante sobrescritura o eliminación accidental.
* **Object Lock (WORM)**: impide la modificación/borrado durante un período.

```bash
aws s3api put-bucket-versioning --bucket mi-bucket \
  --versioning-configuration Status=Enabled

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

### 🌐 Aislamiento de red

* Usa **VPC Gateway Endpoint** para que el tráfico S3 no salga a Internet.
* Configura políticas condicionales que exijan `aws:SourceVpce`.

```bash
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-abc123 \
  --service-name com.amazonaws.eu-west-1.s3 \
  --route-table-ids rtb-xyz456 \
  --vpc-endpoint-type Gateway
```

---

## 💽 Amazon EBS (almacenamiento de bloques)

### 🔐 Cifrado en reposo

* Se activa por **defecto** en cuentas nuevas.
* Utiliza **AWS KMS** para cifrado transparente de volúmenes y snapshots.
* Puedes seleccionar una **CMK personalizada** para mayor control.

```bash
aws ec2 create-volume \
  --size 20 \
  --availability-zone eu-west-1a \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id "arn:aws:kms:region:account:key/key-id"
```

### 🧩 Gestión de snapshots

* Cifrados automáticamente si el volumen está cifrado.
* Pueden copiarse a otra región o a otra cuenta (para DR y aislamiento).

```bash
# Copiar snapshot a otra región
aws ec2 copy-snapshot \
  --source-region eu-west-1 \
  --source-snapshot-id snap-xxxx \
  --destination-region eu-central-1 \
  --encrypted \
  --kms-key-id arn:aws:kms:eu-central-1:account:key/key-id
```

### 🛡️ Buenas prácticas

* **Permitir solo a instancias dentro de la misma AZ.**
* **Controlar quién puede crear, borrar o compartir snapshots.**
* **Integrar con AWS Backup** para automatizar retención y cumplimiento.

---

## 📂 Amazon EFS (almacenamiento de archivos)

### 🔐 Cifrado en tránsito y en reposo

* Cifrado en **reposo** con AWS KMS.
* Cifrado en **tránsito** activado al montar (`tls` en el cliente NFS).

```bash
sudo mount -t nfs4 -o tls fs-abc123.efs.eu-west-1.amazonaws.com:/ efs
```

### 👥 Access Points

Permiten separar el acceso por aplicación o usuario, con rutas, UID/GID, y permisos propios.

```bash
aws efs create-access-point \
  --file-system-id fs-abc123 \
  --posix-user Uid=1001,Gid=1001 \
  --root-directory '{
    "Path": "/appdata",
    "CreationInfo": {
      "OwnerUid": 1001,
      "OwnerGid": 1001,
      "Permissions": "750"
    }
  }'
```

### 🌐 Aislamiento de red

* EFS se monta mediante **Mount Targets**, que deben estar en **subredes privadas**.
* **Security Groups** definen qué instancias pueden conectarse.

---

## 🏢 Ejemplo empresarial

**Escenario**: una empresa de análisis de datos tiene tres necesidades:

1. **Logs de sensores IoT** → almacenar masivamente datos no estructurados.
2. **Disco para VM de procesamiento intensivo**.
3. **Carpeta compartida entre varios microservicios.**

| Requisito              | Servicio | Configuración recomendada                                       |
| ---------------------- | -------- | --------------------------------------------------------------- |
| Logs IoT               | S3       | Bloqueo de público + SSE-KMS + Versioning + VPC Endpoint        |
| Disco de procesamiento | EBS      | Volumen cifrado con CMK + snapshot cross-region                 |
| Carpeta compartida     | EFS      | Access Points + montaje en subred privada + cifrado en tránsito |

---

## ❌ Errores comunes

* Buckets públicos sin control en pruebas.
* Uso de `ACLs` heredadas sin necesidad.
* No activar versioning ni object lock en buckets críticos.
* Snapshots sin cifrar o sin replicar.
* EFS montado en subred pública o sin TLS.
* Uso de claves KMS sin rotación ni control de acceso granular.

---

## ✅ Checklist rápido

| Seguridad              | S3           | EBS | EFS     |
| ---------------------- | ------------ | --- | ------- |
| Cifrado reposo         | ✅            | ✅   | ✅       |
| Cifrado tránsito       | -            | -   | ✅ (TLS) |
| IAM granular           | ✅            | ✅   | ✅       |
| Versionado             | ✅            | N/A | N/A     |
| Inmutabilidad          | ✅            | ❌   | ❌       |
| Aislamiento red        | ✅ (Endpoint) | ✅   | ✅       |
| Integración con Backup | ✅            | ✅   | ✅       |

---

## 🔗 Recursos oficiales

* [AWS S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)
* [EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSEncryption.html)
* [Amazon EFS Security](https://docs.aws.amazon.com/efs/latest/ug/security-considerations.html)
