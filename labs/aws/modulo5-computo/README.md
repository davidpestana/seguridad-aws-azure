# 🧪 Laboratorio 05 – Lanzar una instancia EC2 segura con autenticación por clave y rol IAM

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Lanzar una instancia EC2 con acceso SSH mediante clave.
* Crear y asociar un **rol IAM** con permisos mínimos.
* Asociar el rol a la instancia para permitirle acceder a otros servicios (como S3).
* Validar el acceso mediante SSH y el uso de credenciales temporales.

---

## 🧵 Escenario práctico

Estás configurando un servidor de backend en EC2. Necesitas asegurarte de que:

* Solo tú puedas acceder por SSH (clave privada).
* La instancia pueda acceder a un bucket S3 sin guardar credenciales en disco, **usando un rol IAM seguro**.

---

## 🧰 Requisitos previos

* Cuenta AWS con permisos sobre EC2, IAM y S3.
* Una VPC y subred pública configuradas (puedes usar las del lab de red).
* AWS CLI (opcional para validaciones).
* Una clave SSH (`.pem`) disponible o lista para generar.

---

## 🧭 Pasos detallados

### 1️⃣ Crear un rol IAM para EC2 con acceso a S3

1. Ve a **IAM** → **Roles** → **Create role**
2. Tipo de entidad confiable: **AWS service**
3. Use case: **EC2**
4. Políticas:

   * ✅ `AmazonS3ReadOnlyAccess`
5. Nombre del rol: `ec2-lectura-s3`
6. Crear rol

---

### 2️⃣ Crear un Security Group que permita SSH

1. Ve a **EC2** → **Security Groups** → **Create**
2. Nombre: `sg-ssh-ec2`
3. Reglas de entrada:

   * Tipo: SSH
   * Puerto: 22
   * Origen: *Mi IP* (o una IP controlada)
4. Reglas de salida: dejar por defecto

---

### 3️⃣ Lanzar una instancia EC2

1. Ve a **EC2** → **Launch Instance**
2. Nombre: `ec2-segura`
3. AMI: Amazon Linux 2
4. Tipo: `t2.micro` (dentro del Free Tier)
5. Red:

   * VPC: la que usas para los labs
   * Subred: pública
   * IP pública: ✅ activada
6. Security Group: `sg-ssh-ec2`
7. Clave SSH: usar una existente o generar una nueva
8. En la sección **Advanced details**:

   * IAM role: `ec2-lectura-s3`
9. Lanzar instancia

---

### 4️⃣ Conectarse por SSH

Una vez esté en estado "running":

```bash
ssh -i mi-clave.pem ec2-user@<ip-publica>
```

✅ Estás dentro de una máquina segura en la nube.

---

### 5️⃣ Validar que la instancia puede acceder a S3

1. Instala el cliente AWS CLI:

```bash
sudo yum install -y awscli
```

2. Ejecuta (sin configurar `aws configure`):

```bash
aws s3 ls
```

✅ Verás la lista de buckets (o error si no hay ninguno, pero no de permisos)

3. (Opcional) Accede a un objeto de prueba:

```bash
aws s3 ls s3://nombre-de-tu-bucket/
```

✅ Esto se puede hacer gracias al **rol IAM adjunto a la instancia**.

---

## ✅ Validación

* ¿Puedes conectarte por SSH a la instancia desde tu IP?
* ¿La instancia tiene el rol `ec2-lectura-s3`?
* ¿Puedes acceder a buckets S3 desde dentro de EC2 sin configurar credenciales?

---

## ⚠️ Errores comunes

| Problema                            | Causa común                                 | Solución                                         |
| ----------------------------------- | ------------------------------------------- | ------------------------------------------------ |
| "Permission denied" al conectar     | IP no permitida en el SG o clave incorrecta | Revisa el Security Group y usa la clave correcta |
| "Unable to locate credentials"      | El rol IAM no está asociado a la instancia  | Detén y relanza con rol correcto                 |
| "Access denied" al usar `aws s3 ls` | Permisos del rol insuficientes              | Añade `AmazonS3ReadOnlyAccess` o similar         |

---

## 🧩 Extensiones opcionales

* Usa `User Data` para instalar paquetes al lanzar la instancia.
* Adjunta un volumen EBS adicional y móntalo en `/mnt/datos`.
* Crea una política IAM personalizada más restrictiva (solo acceso a 1 bucket).
* Habilita **CloudWatch Agent** para monitorizar CPU y memoria.