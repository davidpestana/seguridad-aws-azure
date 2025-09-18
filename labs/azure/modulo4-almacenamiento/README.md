# 🧪 Laboratorio 04 – Crear una cuenta de almacenamiento segura en Azure

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Crear una cuenta de almacenamiento en Azure.
* Configurar el **cifrado en reposo** con claves gestionadas.
* Activar el **bloqueo de acceso público** a contenedores.
* Aplicar **roles de acceso (RBAC)** a nivel de cuenta o contenedor.
* Subir y acceder a blobs mediante el portal y la CLI.

---

## 🧵 Escenario práctico

Como parte del equipo de seguridad, debes configurar una cuenta de almacenamiento en Azure para guardar documentos internos. Debes asegurarte de que:

* Todo esté **cifrado** automáticamente.
* El acceso público esté completamente **bloqueado**.
* Solo usuarios autorizados puedan acceder, usando **Azure AD y roles**.

---

## 🧰 Requisitos previos

* Suscripción activa de Azure.
* Acceso al portal de Azure: [https://portal.azure.com](https://portal.azure.com)
* Azure CLI instalada (opcional, para pruebas)
* Rol con permisos para crear recursos de almacenamiento y asignar roles (por ejemplo, *Owner* o *Storage Account Contributor* + *User Access Administrator*).

---

## 🧭 Pasos detallados

### 1️⃣ Crear una cuenta de almacenamiento

1. En el portal, busca **“Storage accounts”** → **Create**

2. Configura:

   * **Nombre**: `almacenamientoseguro<tuusuario>`
   * **Región**: la misma que usas para otros recursos
   * **Performance**: Standard
   * **Redundancy**: LRS (Locally-redundant storage)
   * **Access tier**: Hot

3. En **Advanced**, activa:

   * **Secure transfer required**: ✅
   * **Allow public access**: ❌ Desactivado (importantísimo)
   * **Blob public access**: Disabled
   * **Default encryption type**: Microsoft-managed keys (o tu propia KMS)

4. Clic en **Review + Create** → **Create**

✅ Esto asegura una base segura desde el principio.

---

### 2️⃣ Crear un contenedor privado (tipo blob)

1. Dentro de la cuenta de almacenamiento → pestaña **Containers**
2. Crear un nuevo contenedor:

   * **Nombre**: `documentos`
   * **Public access level**: Private (no anonymous access)

---

### 3️⃣ Subir un archivo de prueba

1. Entra en el contenedor → **Upload**
2. Selecciona un archivo (`prueba.txt`)
3. Sube el archivo y verifica:

   * Propiedades ➜ **Access tier**, **Encryption scope**, etc.
   * Confirma que no hay URL pública accesible

✅ Todos los blobs están cifrados por defecto y no accesibles públicamente.

---

### 4️⃣ Asignar un rol RBAC a un usuario (lectura)

1. Ve a la **cuenta de almacenamiento** → pestaña **Access control (IAM)**
2. Haz clic en **Add → Add role assignment**
3. Configura:

   * **Role**: *Storage Blob Data Reader*
   * **Assign access to**: *User, group or service principal*
   * **Select**: tu usuario (por ejemplo, `usuario-lab@...`)

✅ Este usuario podrá acceder mediante CLI o SDK, autenticándose con Azure AD.

---

### 5️⃣ (Opcional) Validar acceso con Azure CLI

#### A. Iniciar sesión con el usuario autorizado

```bash
az login
```

#### B. Listar blobs en el contenedor:

```bash
az storage blob list \
  --account-name almacenamientoseguro<tuusuario> \
  --container-name documentos \
  --auth-mode login \
  --output table
```

#### C. Descargar un blob:

```bash
az storage blob download \
  --account-name almacenamientoseguro<tuusuario> \
  --container-name documentos \
  --name prueba.txt \
  --file prueba-descargada.txt \
  --auth-mode login
```

---

## ✅ Validación

* ¿La cuenta de almacenamiento tiene **cifrado** y **bloqueo de acceso público**?
* ¿El contenedor es privado?
* ¿Solo usuarios con roles asignados pueden ver o descargar blobs?
* ¿El CLI devuelve acceso denegado si no tienes permisos?

---

## ⚠️ Errores comunes

| Problema                            | Causa común                          | Solución                                   |
| ----------------------------------- | ------------------------------------ | ------------------------------------------ |
| Blob visible públicamente           | Contenedor creado con acceso público | Borra y vuelve a crearlo como privado      |
| CLI falla con “authorization error” | Usuario sin rol asignado             | Asigna `Storage Blob Data Reader`          |
| Archivos sin cifrar                 | Cifrado desactivado o incorrecto     | Revisa propiedades del contenedor y cuenta |

---

## 🧩 Extensiones opcionales

* Usa **Azure Key Vault** para gestionar tus propias claves de cifrado.
* Configura **Shared Access Signatures (SAS)** con expiración y permisos limitados.
* Aplica **políticas de retención automática** (lifecycle management).
* Habilita **auditoría** con Azure Monitor o Defender for Storage.