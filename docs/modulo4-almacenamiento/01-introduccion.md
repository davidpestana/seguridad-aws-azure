## 🗂️ Introducción al almacenamiento seguro en la nube

### 🧭 ¿Qué es y por qué importa?

En entornos tradicionales, el almacenamiento de datos suele ser un componente físico o lógico controlado directamente por los equipos de infraestructura: discos duros, cabinas SAN, NFS internos, etc. En la nube, en cambio, el almacenamiento pasa a ser un **servicio gestionado por el proveedor**, compartido entre múltiples clientes (**multi-inquilino**) y **accesible globalmente por red**, lo que introduce nuevos riesgos… y también nuevos mecanismos de protección.

Por tanto, la **seguridad del almacenamiento en la nube** no depende sólo de cómo se cifra o quién accede, sino también de **qué tipo de almacenamiento se elige**, **qué controles se aplican por defecto**, y **cómo se clasifican y protegen los datos según su sensibilidad**.

---

## 📊 Tipos de almacenamiento en AWS y Azure

En ambos proveedores, existen **tres grandes tipos** de almacenamiento con perfiles de seguridad y uso distintos:

| Tipo         | AWS | Azure         | Casos de uso típicos                 |
| ------------ | --- | ------------- | ------------------------------------ |
| **Objetos**  | S3  | Blob Storage  | Archivos estáticos, backups, logs    |
| **Bloques**  | EBS | Managed Disks | Discos de VMs, BD en IaaS, alta IOPS |
| **Archivos** | EFS | Azure Files   | Compartidos NFS/SMB entre instancias |

Cada uno de ellos requiere decisiones distintas en cuanto a:

* Cifrado (en reposo y en tránsito).
* Control de acceso (identidades, políticas, redes).
* Persistencia e inmutabilidad.
* Coste asociado al volumen y frecuencia de acceso.
* Opciones de backup, replicación y recuperación.

---

## 🔐 Seguridad en contexto Cloud: cambia el modelo

### 🎯 Modelo de responsabilidad compartida

Cuando usamos almacenamiento en la nube, **el proveedor asegura la infraestructura**, pero **el cliente sigue siendo responsable de la configuración segura de los datos**:

| Responsable            | AWS / Azure             | Cliente                                 |
| ---------------------- | ----------------------- | --------------------------------------- |
| Seguridad física       | ✅                       | ❌                                       |
| Alta disponibilidad    | ✅                       | ❌                                       |
| Cifrado activado       | 🔸 Depende del servicio | ✅ (recomendado gestionarlo activamente) |
| Políticas de acceso    | ❌ (el cliente decide)   | ✅                                       |
| Clasificación de datos | ❌                       | ✅                                       |

---

## 🧩 Clasificación y protección de datos

La gestión segura comienza clasificando los datos **según su sensibilidad y requisitos legales**. A partir de ahí, se deben aplicar **controles adecuados a cada tipo de dato**.

| Clasificación      | Ejemplos                               | Requisitos de seguridad                                    |
| ------------------ | -------------------------------------- | ---------------------------------------------------------- |
| **Públicos**       | Archivos estáticos web, folletos       | HTTPS, sin escritura pública, versionado opcional          |
| **Internos**       | Logs, informes, dashboards             | Acceso limitado por roles, cifrado en tránsito y reposo    |
| **Confidenciales** | Datos personales, correos internos     | Cifrado con CMK, auditoría, acceso mínimo necesario        |
| **Regulados**      | Datos financieros, sanitarios, legales | Inmutabilidad, retención, backups aislados, cifrado fuerte |

---

## 💸 Coste vs seguridad

Uno de los errores frecuentes es elegir la clase de almacenamiento **solo por precio**, sin considerar implicaciones de seguridad:

| Caso                        | Elección barata 💸                 | Riesgo asociado 😬                                | Alternativa segura ✅                              |
| --------------------------- | ---------------------------------- | ------------------------------------------------- | ------------------------------------------------- |
| Bucket público sin control  | S3 sin bloquear acceso público     | Datos expuestos en Internet                       | S3 con Block Public Access + política restringida |
| Blob con SAS sin expiración | Azure Blob con token compartido    | Acceso indefinido sin trazabilidad                | SAS con expiración corta + logs en Monitor        |
| Backups sin replicación     | EBS snapshots en misma cuenta/zona | Pérdida por fallo regional o compromiso de cuenta | Snapshots cifrados cross-account y cross-region   |

---

## ❌ Errores comunes

1. **Exponer buckets/contenedores públicamente “para pruebas”** y olvidarlos activos durante años.
2. **No forzar HTTPS** en acceso a datos: permite MITM y lectura de datos.
3. **No activar versionado** ni mecanismos de retención ante errores o ataques.
4. **No auditar el uso de claves KMS/Key Vault** ni rotarlas periódicamente.
5. **Usar claves por defecto del proveedor** sin tener control sobre revocación o rotación.
6. **Permitir acceso desde cualquier IP** en producción sin filtrar por red/endpoint privado.

---

## ✅ Buenas prácticas clave (resumen)

| Área                       | Buenas prácticas clave                                     |
| -------------------------- | ---------------------------------------------------------- |
| **Tipo de almacenamiento** | Elegir objetos/bloques/archivos según caso de uso          |
| **Acceso**                 | Usar IAM/RBAC + políticas específicas por recurso          |
| **Cifrado**                | Activar cifrado en reposo y tránsito, preferir CMK/BYOK    |
| **Clasificación**          | Mapear datos → sensibilidad → controles técnicos           |
| **Auditoría**              | Habilitar logs (S3 Access Logs, CloudTrail, Azure Monitor) |
| **Aislamiento de red**     | Usar VPC/Private Endpoints para evitar salida a Internet   |

---

## 🧪 Cómo probar y validar

### En AWS:

```bash
# Crear bucket con bloqueo de acceso público
aws s3api create-bucket --bucket mi-bucket-seguro --region eu-west-1

aws s3control put-public-access-block \
  --account-id <ID_CUENTA> \
  --public-access-block-configuration 'BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true'
```

### En Azure:

```bash
# Crear storage account con transferencia segura y acceso privado
az storage account create \
  --name mistorageaccount \
  --resource-group mi-rg \
  --location westeurope \
  --sku Standard_LRS \
  --https-only true \
  --default-action Deny
```

---

## 🔗 Recursos recomendados

* [AWS S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)
* [Azure Storage Security Guide](https://learn.microsoft.com/en-us/azure/storage/common/storage-security-guide)
* [Comparativa tipos de almacenamiento en AWS](https://aws.amazon.com/products/storage/)
* [Comparativa servicios de almacenamiento en Azure](https://learn.microsoft.com/en-us/azure/architecture/data-guide/technology-choices/data-storage)

