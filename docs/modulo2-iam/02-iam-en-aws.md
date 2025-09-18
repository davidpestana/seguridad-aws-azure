# 📄 `02-iam-en-aws.md`


## 🧭 Introducción – ¿Qué es IAM en AWS y por qué es tan crítico?

**AWS Identity and Access Management (IAM)** es el servicio nativo de AWS para gestionar identidades y controlar qué acciones pueden realizar los usuarios o servicios sobre los recursos de la nube.

En AWS, **todo acceso está mediado por IAM**: desde quién puede lanzar una instancia EC2 hasta qué función Lambda puede leer un bucket S3.
Un error en su configuración puede significar que un atacante obtenga control total de tu infraestructura con solo una clave expuesta.

IAM es gratuito, potente y flexible, pero también complejo. En este archivo aprenderás a dominar sus componentes esenciales.

---

## 🔎 Explicación detallada – Componentes clave de IAM

### 👤 1. Usuarios IAM

Representan **personas** que necesitan acceso a la consola de AWS o a la API.

* Pueden tener:

  * Nombre de usuario
  * Claves de acceso programático (`access key ID` y `secret`)
  * Contraseña para la consola
  * Dispositivo MFA

✔️ **Recomendado solo para cuentas humanas (admins, desarrolladores, auditores).**

---

### 👥 2. Grupos

Permiten **agrupar usuarios IAM** y asignarles políticas compartidas.

* Ejemplo: grupo `Desarrolladores` con permisos de lectura en S3 y escritura en Lambda.
* Simplifican la administración: puedes modificar los permisos de muchos usuarios editando una sola política.

---

### 🧰 3. Roles

Una **identidad sin credenciales fijas** que puede ser “asumida” temporalmente por:

* Servicios de AWS (ej. EC2, Lambda)
* Usuarios federados (SAML, OIDC)
* Cuentas externas (cross-account access)

🔑 Ideal para acceso **delegado, temporal y seguro**, eliminando la necesidad de claves permanentes.

> 🟡 **Caso común**: una instancia EC2 necesita acceso a un bucket S3 → se le asocia un rol IAM con los permisos necesarios.

---

### 📜 4. Políticas (Policies)

Son **documentos en formato JSON** que describen qué acciones están permitidas o denegadas, sobre qué recursos, y bajo qué condiciones.

#### Estructura típica:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::mi-bucket/*"
    }
  ]
}
```

| Campo       | Significado                                                          |
| ----------- | -------------------------------------------------------------------- |
| `Action`    | Operaciones permitidas (`s3:ListBucket`, `ec2:StartInstances`, etc.) |
| `Resource`  | Objetos afectados (buckets, funciones, tablas, etc.)                 |
| `Condition` | Filtros por IP, tags, tiempo, etc.                                   |
| `Effect`    | Allow o Deny                                                         |

✔️ AWS evalúa todas las políticas aplicables y **solo permite acciones que estén explícitamente autorizadas**.

---

### 🔐 5. STS – Security Token Service

Permite emitir **credenciales temporales válidas por minutos u horas**, usadas para:

* Federación de identidades externas (SSO)
* Acceso cruzado entre cuentas
* Sesiones temporales con permisos limitados

✔️ Reduce el riesgo de exposición de claves permanentes.

---

## 🏢 Caso empresarial – Rol para EC2 con acceso a S3

La empresa **DataAnalytics Co.** ejecuta scripts en EC2 que descargan datasets desde un bucket S3 privado.

✅ En lugar de configurar claves IAM en cada servidor (riesgoso):

1. **Definen un rol IAM** con permisos `s3:GetObject` para ese bucket.
2. **Asocian el rol a la instancia EC2** al lanzarla.
3. El script usa la librería SDK (`boto3`, `aws-sdk`) y obtiene las credenciales automáticamente.

🔐 Resultado: sin claves estáticas, menor riesgo, mayor control.

---

## 💻 Configuración – Ejemplos prácticos

### ✅ Crear un usuario IAM con acceso de solo lectura en S3

```bash
aws iam create-user --user-name auditor

aws iam attach-user-policy \
  --user-name auditor \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

---

### ✅ Crear un rol para EC2 con acceso restringido a un bucket

#### 1. Política personalizada

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::mi-bucket/*"]
    }
  ]
}
```

#### 2. Crear el rol con CLI (resumen)

```bash
aws iam create-role \
  --role-name EC2-S3-Reader \
  --assume-role-policy-document file://trust-policy-ec2.json

aws iam put-role-policy \
  --role-name EC2-S3-Reader \
  --policy-name S3AccessPolicy \
  --policy-document file://s3-read-policy.json
```

#### 3. Asociar el rol a la instancia EC2 (vía consola o CLI)

---

## 🧪 Tips para probar

### En consola web

* Ir a IAM → Usuarios → Ver políticas efectivas.
* Usar **Policy Simulator** para probar si una acción está permitida.
* Crear una función Lambda y asignarle un rol → validar que puede acceder a un recurso S3 o DynamoDB.

### En CLI

* Obtener lista de permisos del usuario:

```bash
aws iam list-attached-user-policies --user-name auditor
```

* Simular una política:

```bash
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::<cuenta>:user/auditor \
  --action-names s3:GetObject \
  --resource-arns arn:aws:s3:::mi-bucket/*
```

---

## ❌ Errores comunes

| Error                                   | Consecuencia                    | Solución                                                |
| --------------------------------------- | ------------------------------- | ------------------------------------------------------- |
| Usar la cuenta root para tareas comunes | Riesgo total ante filtración    | Crear usuarios IAM limitados                            |
| Usar claves de acceso permanentes       | Alta probabilidad de filtración | Usar roles temporales con STS                           |
| Permitir `*:*` en políticas             | Acceso completo, sin control    | Aplicar mínimo privilegio (principio “least privilege”) |

---

## ✅ Buenas prácticas IAM en AWS

✔️ Checklist inicial:

* [ ] Desactivar acceso root (o proteger con MFA).
* [ ] Crear grupos y roles según funciones (dev, auditor, ops…).
* [ ] Usar políticas administradas + políticas personalizadas cuando sea necesario.
* [ ] Rotar claves IAM cada 90 días (o eliminarlas).
* [ ] Revisar logs de acceso con CloudTrail.

---

## 📚 Recursos recomendados

| Recurso                                                                                                       | Descripción                                         |
| ------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| [AWS IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/)                                       | Manual completo de usuarios, roles, políticas y STS |
| [IAM Policy Simulator](https://policysim.aws.amazon.com/)                                                     | Simulador para probar permisos de una identidad     |
| [AWS Well-Architected – Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/) | Buenas prácticas para diseñar sistemas seguros      |
| [STS Documentation](https://docs.aws.amazon.com/STS/latest/APIReference/Welcome.html)                         | Uso de credenciales temporales en AWS               |
