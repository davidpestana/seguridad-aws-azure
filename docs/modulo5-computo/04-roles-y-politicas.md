# 🧾 Roles, políticas e identidades administradas (AWS y Azure)

---

## 🗂️ Introducción

Uno de los errores más frecuentes al desplegar máquinas en la nube es **incrustar credenciales (API keys, contraseñas)** dentro del código, variables de entorno o archivos de configuración. Esto **aumenta drásticamente el riesgo de filtraciones**, especialmente en proyectos versionados en Git.

La solución moderna y segura es usar **roles o identidades administradas**, que permiten a las instancias **obtener credenciales temporales automáticamente**, sin intervención del desarrollador.

---

## 🔑 Principio de mínimo privilegio

> 🔐 “Cada entidad debe tener sólo los permisos estrictamente necesarios para realizar su tarea.”

Esto aplica tanto a **usuarios humanos** como a **instancias o procesos automáticos**. Las buenas prácticas dictan que:

* Una instancia no debe tener acceso total a todos los servicios.
* Cada rol o identidad debe ser **específica** y con **política acotada**.
* Las credenciales deben ser **temporales y rotadas automáticamente**.

---

## 🟡 Roles en AWS EC2

### ✅ ¿Qué es un IAM Role para instancias?

Un **IAM Role** permite que una instancia EC2 obtenga **credenciales temporales** (via STS) para acceder a servicios como:

* S3 (leer/escribir datos).
* DynamoDB (leer registros).
* Parameter Store / Secrets Manager (leer secretos).

### 🔧 Cómo asignar un rol a una instancia

1. Crear el rol con una **policy** acotada:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [ "s3:GetObject" ],
      "Resource": "arn:aws:s3:::mi-bucket/*"
    }
  ]
}
```

2. Crear el **Instance Profile** y asociarlo al rol.

3. Al lanzar la EC2, asignar el perfil:

```bash
aws ec2 run-instances \
  --iam-instance-profile Name=rol-lectura-s3 \
  ...
```

✅ Las credenciales STS se inyectan automáticamente en `http://169.254.169.254/latest/meta-data/iam/` y pueden usarse desde el SDK de AWS.

---

## 🔵 Managed Identities en Azure

### ✅ ¿Qué es una Managed Identity?

Una **identidad administrada** es una cuenta controlada por Azure que:

* No tiene claves ni contraseñas.
* Está vinculada a una VM o recurso.
* Permite autenticación automática a otros servicios Azure.

Dos tipos:

| Tipo                | Descripción                           |
| ------------------- | ------------------------------------- |
| **System-assigned** | Se crea y destruye junto con la VM    |
| **User-assigned**   | Reutilizable entre múltiples recursos |

---

### 🔧 Cómo usar una Managed Identity

1. Habilitar identidad administrada al crear la VM:

```bash
az vm create \
  --resource-group mi-rg \
  --name mi-vm \
  --image Ubuntu2204 \
  --assign-identity
```

2. Asignar permisos RBAC sobre el recurso destino:

```bash
az role assignment create \
  --assignee <IDENTITY-ID> \
  --role "Reader" \
  --scope "/subscriptions/xxx/resourceGroups/mi-rg/providers/Microsoft.Storage/storageAccounts/mistorage"
```

3. Usar el SDK con autenticación automática:

```python
from azure.identity import ManagedIdentityCredential
from azure.storage.blob import BlobServiceClient

credential = ManagedIdentityCredential()
client = BlobServiceClient(account_url="https://mistorage.blob.core.windows.net", credential=credential)
```

---

## 🤖 ¿Qué resuelven roles/identidades?

| Problema                   | Solución con rol/identidad administrada    |
| -------------------------- | ------------------------------------------ |
| Incrustar claves en código | ❌ → se accede vía rol automáticamente      |
| Rotación de credenciales   | ✅ automática vía STS / Azure Token Service |
| Permisos demasiado amplios | ✅ políticas específicas por recurso        |
| Trazabilidad de acciones   | ✅ se asocian al rol/identidad en los logs  |

---

## 🧠 Buenas prácticas

| Recomendación                                 | Justificación                                       |
| --------------------------------------------- | --------------------------------------------------- |
| Usar **1 rol/identidad por función**          | Facilita trazabilidad y control de permisos         |
| Nunca incrustar claves en código/repositorios | Riesgo de exposición crítica                        |
| Revisar políticas con frecuencia              | Verificar que no tengan acciones o recursos “\*”    |
| Limitar acceso por tiempo o contexto (tags)   | Políticas condicionales (ej. IP, etiqueta, horario) |
| Auditar uso de credenciales y tokens          | CloudTrail / Azure Monitor                          |

---

## 🏢 Caso empresarial

**Escenario**: una empresa tiene una aplicación desplegada en VMs que necesita leer archivos de un bucket (S3 o Azure Blob) y consultar una base de datos.

| Requisito                      | AWS                               | Azure                                               |
| ------------------------------ | --------------------------------- | --------------------------------------------------- |
| Acceso a almacenamiento        | IAM Role con `s3:GetObject`       | Managed Identity con rol `Storage Blob Data Reader` |
| Acceso a base de datos         | IAM auth (RDS) o secreto vía SSM  | Identity-based access a SQL DB                      |
| No usar credenciales en código | Se accede vía STS automáticamente | Se accede con Token vía MSI                         |
| Control de permisos            | Política adjunta al role          | RBAC sobre recurso + grupo                          |

---

## ❌ Errores comunes

1. Incrustar claves en variables de entorno, archivos `.env` o scripts.
2. Asignar roles con permisos globales (`s3:*`, `*:*`) sin necesidad.
3. Reutilizar un solo rol o identidad para múltiples apps diferentes.
4. No usar roles en instancias: “funciona con clave” pero es frágil.
5. No revocar roles al eliminar una VM o recurso asociado.
6. Olvidar configurar el **scope correcto** en Azure al asignar permisos.

---

## ✅ Checklist rápido

| Elemento                 | AWS                  | Azure                         |
| ------------------------ | -------------------- | ----------------------------- |
| Identidad automática     | IAM Role para EC2    | Managed Identity para VM      |
| Sin claves explícitas    | ✅                    | ✅                             |
| Permisos mínimos         | Política IAM acotada | Rol RBAC acotado              |
| Control de ciclo de vida | IAM Role + Profile   | System-assigned Identity      |
| Auditoría de uso         | CloudTrail           | Azure Monitor + Activity Logs |

---

## 🔗 Recursos recomendados

* [AWS: Using IAM Roles for EC2](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html)
* [STS Temporary Credentials – AWS](https://docs.aws.amazon.com/STS/latest/APIReference/Welcome.html)
* [Azure: Managed Identity Overview](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)
* [Azure SDK – Authentication via Managed Identity](https://learn.microsoft.com/en-us/dotnet/azure/sdk/authentication/managed-identity)