# 🧪 Laboratorio 02 – Activar MFA y crear un usuario IAM seguro

## 🎯 Objetivos del laboratorio

Al completar este laboratorio, el alumno será capaz de:

* Activar MFA (autenticación multifactor) en la cuenta root de AWS.
* Crear un usuario IAM con permisos administrativos.
* Generar credenciales seguras para trabajar desde CLI.
* Aplicar buenas prácticas iniciales de acceso en AWS.

---

## 🧵 Escenario práctico

Estás configurando por primera vez la seguridad de una cuenta AWS recién creada. Para proteger el acceso, debes **activar MFA en la cuenta root** y **crear un usuario IAM separado con permisos administrativos**, evitando el uso directo de la cuenta raíz.

---

## 🧰 Requisitos previos

* Acceso a la cuenta root de AWS.
* Dispositivo móvil con app de autenticación TOTP (ej: Google Authenticator, Authy, etc.).
* Navegador web para acceder a la consola de AWS.
* AWS CLI opcional (si se desea probar el acceso programático).

---

## 🧭 Pasos detallados

### 1️⃣ Activar MFA para la cuenta root

1. Accede a la consola: [https://console.aws.amazon.com](https://console.aws.amazon.com).
2. Inicia sesión con la cuenta root (correo y contraseña principal).
3. En la esquina superior derecha, haz clic en tu nombre → **“Security credentials”**.
4. En la sección **“Multi-factor authentication (MFA)”**, haz clic en **“Activate MFA”**.
5. Elige **“Virtual MFA device”** y haz clic en continuar.
6. Abre la app de autenticación en tu móvil y escanea el código QR.
7. Introduce los dos códigos TOTP generados por la app y confirma.

✅ *¡Listo! Tu cuenta root está protegida por MFA.*

---

### 2️⃣ Crear un grupo de IAM con permisos administrativos

1. En la consola, ve a **IAM** → **User groups** → **Create group**.
2. Nombre: `Administradores`
3. En **Attach permissions policies**, busca y selecciona:

   * `AdministratorAccess`
4. Crea el grupo.

---

### 3️⃣ Crear un nuevo usuario IAM con acceso a consola y programático

1. Ve a **IAM** → **Users** → **Add users**
2. Nombre del usuario: `admin-lab`
3. Marca:

   * ✅ Acceso a la consola de AWS
   * ✅ Acceso programático (CLI/SDK)
4. Establece una contraseña temporal o personalizada.
5. En la sección de grupos, selecciona: `Administradores`.
6. Crear usuario.

📝 **Anota las credenciales del usuario** (Access key ID y Secret access key), si vas a probar la CLI.

---

### 4️⃣ Probar inicio de sesión del nuevo usuario

1. Cierra sesión de la cuenta root.
2. Ve a: `https://<account-id>.signin.aws.amazon.com/console`
   (puedes obtener el `account-id` en la parte superior de IAM → "Dashboard")
3. Inicia sesión con `admin-lab` y la contraseña establecida.
4. Cambia la contraseña si se solicita.

---

### 5️⃣ (Opcional) Configurar y probar acceso por CLI

1. Instala AWS CLI si no lo tienes (`brew`, `apt`, `choco`, etc.)
2. Ejecuta:

```bash
aws configure
```

3. Introduce:

   * Access Key ID
   * Secret Access Key
   * Región por defecto (ej: `us-east-1`)
   * Formato de salida (ej: `json`)

4. Prueba:

```bash
aws iam list-users
```

✅ Deberías ver los usuarios de tu cuenta.

---

## ✅ Validación

* ¿El usuario root tiene MFA activado?
* ¿Existe un usuario IAM con permisos administrativos?
* ¿Puedes iniciar sesión con ese usuario y ver recursos?
* (Opcional) ¿Puedes usar la CLI con ese usuario?

---

## ⚠️ Errores comunes

| Problema                                      | Causa                                          | Solución                                 |
| --------------------------------------------- | ---------------------------------------------- | ---------------------------------------- |
| No puedes activar MFA                         | Ya activado o app mal sincronizada             | Revisa zona horaria del móvil o reinicia |
| No se puede iniciar sesión con el usuario IAM | Password vencida o mal creada                  | Intenta restablecer desde IAM            |
| CLI devuelve “Access Denied”                  | No se aplicó la política `AdministratorAccess` | Asegúrate de que el grupo tenga permisos |

---

## 🧩 Extensiones opcionales

* Obliga el uso de MFA para usuarios IAM desde una política.
* Crea un usuario IAM con permisos solo de lectura.
* Configura un perfil en `~/.aws/config` con nombre y múltiples cuentas.
* Desactiva el acceso programático si no se necesita.