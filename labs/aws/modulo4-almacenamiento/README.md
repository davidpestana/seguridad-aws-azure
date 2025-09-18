# 🧪 Laboratorio 04 – Configurar un bucket S3 seguro en AWS

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Crear un bucket de S3 con nombre único.
* Habilitar el **cifrado en reposo** (AES-256 o KMS).
* Configurar las **políticas de bloqueo de acceso público**.
* Aplicar un **control de acceso con políticas de bucket** o IAM.
* Validar el acceso mediante subida/lectura de objetos.

---

## 🧵 Escenario práctico

Tu empresa necesita almacenar documentos sensibles en AWS. Como parte del equipo de seguridad, debes asegurarte de que:

* El bucket **no sea accesible públicamente**.
* Todos los objetos estén **cifrados en reposo**.
* Solo ciertos usuarios o servicios internos tengan permiso para leer o escribir.

---

## 🧰 Requisitos previos

* Cuenta de AWS con permisos para usar S3 e IAM.
* Acceso a la consola de AWS y opcionalmente AWS CLI.
* Conocer tu **ARN de usuario IAM** si vas a usar políticas personalizadas.

---

## 🧭 Pasos detallados

### 1️⃣ Crear el bucket S3 desde la consola

1. Ve a [https://s3.console.aws.amazon.com/s3](https://s3.console.aws.amazon.com/s3)
2. Haz clic en **“Create bucket”**
3. Configura:

   * **Bucket name**: `documentos-seguros-<tutoken>` (único en todo AWS)
   * **Region**: selecciona una como `eu-west-1` o `us-east-1`
   * **Block all public access**: ✅ Mantener activado
   * **Bucket Versioning**: opcionalmente activado
4. Clic en **Create bucket**

---

### 2️⃣ Habilitar cifrado en reposo

1. Ve al bucket → pestaña **Properties**
2. Sección **Default encryption** → Edit
3. Elige una opción:

   * ✅ **SSE-S3 (AES-256)** – cifrado por AWS
   * 🔐 (Opcional) **SSE-KMS** – usa una clave KMS (propia o administrada)

✅ Así todos los objetos subidos estarán cifrados automáticamente.

---

### 3️⃣ Probar subida de archivos

1. Ve a la pestaña **Objects** → **Upload**
2. Sube un archivo de prueba (`documento.txt`)
3. En los detalles del objeto, verifica:

   * **Encryption**: debería decir `SSE-S3` o `SSE-KMS`
   * **Public access**: bloqueado

---

### 4️⃣ Aplicar política de bucket para controlar acceso

Ejemplo de política para permitir solo lectura a un usuario IAM específico:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadOnlyAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/usuario-lab"
      },
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::documentos-seguros-<tutoken>/*"
    }
  ]
}
```

1. Ve al bucket → pestaña **Permissions** → **Bucket policy**
2. Pega la política, sustituyendo el ARN por el tuyo
3. Guarda

✅ Así solo ese usuario podrá leer los objetos.

---

### 5️⃣ Validar el acceso desde otro usuario

1. Cierra sesión o cambia de usuario IAM.
2. Intenta acceder al archivo desde AWS CLI:

```bash
aws s3 cp s3://documentos-seguros-<tutoken>/documento.txt .
```

✅ Solo funcionará si tienes el permiso adecuado.

---

## ✅ Validación

* ¿El bucket tiene **bloqueo público** activado?
* ¿El objeto está **cifrado** en reposo?
* ¿Solo usuarios autorizados pueden leerlo?
* ¿El acceso no autorizado está bloqueado?

---

## ⚠️ Errores comunes

| Problema                         | Causa común                            | Solución                                        |
| -------------------------------- | -------------------------------------- | ----------------------------------------------- |
| “Access Denied” al subir/leer    | No hay permisos IAM o política bloquea | Revisa las políticas IAM y del bucket           |
| El objeto no está cifrado        | No activaste el cifrado por defecto    | Sube nuevamente tras habilitar cifrado          |
| El bucket permite acceso público | No se bloquearon accesos públicos      | Activa “Block all public access” en propiedades |

---

## 🧩 Extensiones opcionales

* Habilita **versionado** en el bucket para proteger contra sobrescritura.
* Usa **S3 Access Points** para dar acceso controlado por servicio.
* Activa **logging** o **CloudTrail** para auditar accesos al bucket.
* Configura **retención automática** o políticas de ciclo de vida.