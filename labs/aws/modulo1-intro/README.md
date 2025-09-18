# 🧪 Laboratorio 01 – Explorando la Consola de Seguridad de AWS

## 🎯 Objetivos del laboratorio

Al completar este laboratorio, el alumno será capaz de:

* Navegar por la consola de AWS con foco en la seguridad.
* Identificar los servicios clave relacionados con seguridad (IAM, KMS, CloudTrail, etc.).
* Localizar recursos de auditoría, buenas prácticas y documentación oficial.
* Comprender cómo se estructura el acceso y la visibilidad en AWS.

---

## 🧵 Escenario práctico

Eres parte del equipo de TI de una pequeña empresa que acaba de crear su cuenta en AWS. Tu primer objetivo es **familiarizarte con el entorno** y **localizar las funciones relacionadas con la seguridad**, para comenzar a configurar buenas prácticas desde el primer día.

---

## 🧰 Requisitos previos

* Cuenta en AWS (puede ser una cuenta gratuita).
* Acceso a la [consola de AWS](https://console.aws.amazon.com/) con un **usuario administrador** o root.
* Navegador actualizado.
* No se requiere AWS CLI para este laboratorio.

---

## 🧭 Pasos detallados

### 1️⃣ Iniciar sesión y revisar la cuenta

1. Accede a la consola: [https://console.aws.amazon.com](https://console.aws.amazon.com)
2. En la parte superior derecha, haz clic en tu nombre de cuenta.
3. Revisa los siguientes elementos:

   * **Número de cuenta AWS**
   * **Nombre del usuario conectado** (ej: `admin@empresa.com`)
   * **Región por defecto** (ej: `us-east-1`)

✅ *Esto es útil para comprender dónde se aplican las políticas y configuraciones.*

---

### 2️⃣ Acceder al panel de IAM (gestión de identidades)

1. En la consola, escribe “IAM” en la barra de búsqueda y accede al servicio.
2. En el **Dashboard de IAM**, observa el bloque “Security recommendations”.
3. Revisa los siguientes puntos:

   * ¿El usuario root tiene MFA activado?
   * ¿Hay políticas con acceso total (`AdministratorAccess`)?
   * ¿Existen usuarios sin MFA?

✅ *IAM es el núcleo de la seguridad en AWS: aquí se controlan los permisos.*

---

### 3️⃣ Explorar la auditoría con CloudTrail

1. Vuelve a la barra de búsqueda y accede al servicio **CloudTrail**.
2. En el menú lateral, haz clic en **Event history**.
3. Observa el listado de eventos recientes:

   * ¿Qué acciones aparecen?
   * ¿Quién las ejecutó?
   * ¿Desde qué IP se hicieron?

✅ *CloudTrail registra todo lo que pasa en tu cuenta.*

---

### 4️⃣ Consultar claves con AWS KMS (Key Management Service)

1. Busca y entra a **KMS**.
2. Observa si hay claves existentes en la región actual.
3. En caso de no haber ninguna, toma nota: **sin claves personalizadas, los servicios usan claves administradas por AWS.**

✅ *KMS es clave para cifrado en reposo (S3, RDS, EBS, etc.).*

---

### 5️⃣ Localizar recursos de buenas prácticas

1. En IAM → sección **"Security recommendations"**, haz clic en “Learn more” para cada recomendación.
2. Explora enlaces como:

   * **Security Hub**
   * **AWS Well-Architected Framework**
   * **Documentación de IAM**

✅ *Estos recursos son esenciales para tomar decisiones informadas sobre seguridad.*

---

## ✅ Validación

Al terminar el laboratorio, asegúrate de poder responder:

* ¿Qué servicios están relacionados con la seguridad en AWS?
* ¿Qué usuarios existen en IAM y qué nivel de acceso tienen?
* ¿Está activado CloudTrail? ¿Puedes ver tu propio login?
* ¿Tienes activado el MFA en la cuenta root o de usuario?

---

## ⚠️ Errores comunes

| Problema                            | Causa común                    | Solución                                                   |
| ----------------------------------- | ------------------------------ | ---------------------------------------------------------- |
| No aparece CloudTrail               | Región errónea                 | Verifica que estés en la región donde se generaron eventos |
| No puedes acceder a IAM             | Permisos insuficientes         | Debes iniciar sesión como administrador o root             |
| El dashboard de IAM muestra alertas | Falta de configuración inicial | Aplica las recomendaciones de seguridad que propone AWS    |

---

## 🧩 Extensiones opcionales

* Habilita MFA para el usuario root si no está activo.
* Crea un nuevo usuario en IAM con acceso limitado.
* Activa AWS Security Hub y revisa las primeras alertas generadas.