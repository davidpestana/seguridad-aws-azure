# 🧪 Laboratorio 01 – Explorando el Portal y Centro de Seguridad de Azure

## 🎯 Objetivos del laboratorio

Al completar este laboratorio, el alumno será capaz de:

* Navegar por el portal de Azure con enfoque en la seguridad.
* Identificar los servicios clave de seguridad: **Azure AD**, **Microsoft Defender for Cloud**, **Key Vault**, **Activity Log**, etc.
* Reconocer configuraciones iniciales de seguridad y auditoría.
* Consultar recursos oficiales y recomendaciones de buenas prácticas.

---

## 🧵 Escenario práctico

Eres un nuevo miembro del equipo de IT en una organización que ha comenzado a migrar sus recursos a Azure. Como primer paso, necesitas **familiarizarte con los servicios de seguridad disponibles en el portal** y validar que se estén aplicando medidas básicas de protección.

---

## 🧰 Requisitos previos

* Cuenta de Azure activa (puede ser una cuenta gratuita: [https://azure.microsoft.com/free](https://azure.microsoft.com/free)).
* Acceso al portal de Azure: [https://portal.azure.com](https://portal.azure.com).
* Rol de **Administrador** o permisos para visualizar recursos de seguridad.
* Navegador web actualizado.

---

## 🧭 Pasos detallados

### 1️⃣ Iniciar sesión en el portal y revisar la cuenta

1. Accede a [https://portal.azure.com](https://portal.azure.com).
2. Verifica tu **usuario** (esquina superior derecha).
3. Haz clic en el icono de directorio ➜ selecciona el directorio correcto si hay varios.
4. Revisa la **suscripción activa** (parte superior de la pantalla principal).

✅ *Esto asegura que estés trabajando en el entorno y directorio correcto.*

---

### 2️⃣ Acceder a Azure Active Directory (AAD)

1. En la barra de búsqueda, escribe **“Azure Active Directory”** y accede.
2. Explora estas secciones:

   * **Usuarios**: ¿qué usuarios hay creados?
   * **Grupos**: ¿existen grupos de seguridad?
   * **Roles y administradores**: ¿qué permisos tienen los usuarios?

✅ *AAD es el núcleo de identidad y autenticación en Azure.*

---

### 3️⃣ Explorar Microsoft Defender for Cloud

1. En el portal, busca **Microsoft Defender for Cloud**.
2. En el panel principal, revisa:

   * **Secure Score** de la suscripción.
   * **Recomendaciones** activas (Security Recommendations).
   * **Alertas recientes** (si las hay).

✅ *Defender for Cloud proporciona un análisis de seguridad continuo con sugerencias y alertas.*

---

### 4️⃣ Consultar Activity Logs (registro de auditoría)

1. En el buscador del portal, escribe **“Activity Log”**.
2. Selecciona la suscripción activa.
3. Observa las entradas:

   * ¿Qué acciones aparecen?
   * ¿Quién las ejecutó?
   * ¿Cuándo?

✅ *El Activity Log permite auditar acciones administrativas dentro de Azure.*

---

### 5️⃣ Revisar Key Vault (gestión de claves y secretos)

1. En la barra de búsqueda, escribe **“Key Vault”**.
2. Si no existe ningún Vault, crea uno básico:

   * Recurso: `vault-seguridad-lab`
   * Región: misma que tu suscripción
   * Permitir acceso al administrador actual
3. Abre el Vault creado y observa:

   * Secciones: Claves, Secretos, Certificados
   * Accesos y políticas

✅ *Key Vault es esencial para almacenar información sensible de forma segura.*

---

### 6️⃣ Localizar documentación y recomendaciones

1. En **Defender for Cloud**, abre una recomendación y accede a su documentación.
2. Visita el **Microsoft Learn**: [https://learn.microsoft.com/azure/security](https://learn.microsoft.com/azure/security)
3. Opcional: consulta las **guías de seguridad para administradores** en la sección de referencias.

✅ *Microsoft Learn es el recurso oficial para capacitarse en seguridad de Azure.*

---

## ✅ Validación

Asegúrate de poder responder:

* ¿Qué usuarios y roles están configurados en AAD?
* ¿Cuál es el **Secure Score** de tu suscripción?
* ¿Has localizado eventos en el **Activity Log**?
* ¿Has explorado un **Key Vault** y sus secciones?

---

## ⚠️ Errores comunes

| Problema                              | Causa común                                  | Solución                                                           |
| ------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------ |
| No se ve Microsoft Defender for Cloud | Región o permisos incorrectos                | Verifica la suscripción y usa una cuenta con permisos de Seguridad |
| No puedes acceder a Key Vault         | Rol insuficiente                             | Solicita permisos de “Key Vault Contributor”                       |
| Activity Log vacío                    | Sin acciones recientes o en región diferente | Filtra por más días o cambia región                                |

---

## 🧩 Extensiones opcionales

* Crea un **usuario limitado** en Azure AD y asígnale un rol básico.
* Habilita MFA (autenticación multifactor) para tu usuario.
* Configura una alerta de seguridad en Defender for Cloud.
* Agrega un **secreto de prueba** en tu Key Vault y visualízalo.