# 🧪 Laboratorio 07 – Detección y respuesta a amenazas con Defender for Cloud y Logic Apps

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Habilitar **Microsoft Defender for Cloud** para protección avanzada.
* Visualizar **recomendaciones de seguridad y alertas**.
* Simular un comportamiento anómalo o una amenaza.
* Crear una automatización de respuesta usando **Logic Apps** (ej. envío de email, bloqueo de IP, etc.).

---

## 🧵 Escenario práctico

Tu organización quiere mejorar su postura de seguridad y capacidad de respuesta. En este laboratorio habilitarás **Microsoft Defender for Cloud**, revisarás las amenazas detectadas y automatizarás una respuesta básica (como una notificación) mediante **Azure Logic Apps**.

---

## 🧰 Requisitos previos

* Suscripción de Azure activa.
* Permisos para habilitar Microsoft Defender for Cloud y crear Logic Apps.
* Una VM creada (Linux o Windows) en ejecución (para simular actividad).
* Opcional: acceso a una dirección de correo electrónico (para notificaciones).

---

## 🧭 Pasos detallados

### 1️⃣ Habilitar Microsoft Defender for Cloud

1. Ve al portal de Azure → busca **“Defender for Cloud”**.
2. En el menú lateral, ve a **Environment settings**.
3. Selecciona tu suscripción → clic en **Enable enhanced security features**.
4. Activa los planes para:

   * Servidores (Virtual Machines)
   * Identity & Access
   * (Opcional) App Services, Storage, etc.

✅ Esto activa la supervisión avanzada en tus recursos.

---

### 2️⃣ Ver recomendaciones y alertas

1. Vuelve al panel principal de **Defender for Cloud**.
2. Accede a:

   * **Secure Score** – mide el nivel de seguridad general.
   * **Recommendations** – acciones sugeridas (ej: habilitar MFA, cerrar puertos, etc.).
   * **Security alerts** – lista de amenazas detectadas.

✅ Si tienes una VM expuesta con SSH o RDP, puede que veas alertas reales tras unas horas.

---

### 3️⃣ Simular una amenaza (opcional)

Azure genera alertas automáticamente, pero puedes forzar una alerta de prueba:

1. Ve a la VM → **Run command** (Linux) → Ejecuta:

```bash
curl http://testmyids.com
```

2. Esto simula una conexión a un dominio malicioso.
3. En unos minutos, revisa **Defender for Cloud → Alerts**.

✅ Aparecerá una alerta con tipo “Possible outbound attack” o similar.

---

### 4️⃣ Crear una Logic App para responder a alertas

1. Ve a **Logic Apps → Create**
2. Nombre: `responder-alertas-seguridad`
3. Grupo de recursos: `rg-incidentes`
4. Tipo: **Consumption**
5. Región: la misma que Defender for Cloud
6. Crear

---

### 5️⃣ Diseñar el flujo de automatización

1. En la Logic App → **Designer**

2. Selecciona **Blank Logic App**

3. Añade trigger:

   * **When a response to an Azure Defender alert is triggered**

4. Luego añade una acción:

   * **Send an email (Outlook, Office 365, Gmail)**

     * Para: tu correo
     * Asunto: `🚨 Alerta de Seguridad en Azure`
     * Cuerpo: `@{triggerBody()?['AlertDisplayName']}`

✅ Este flujo se disparará cuando Defender emita una alerta.

---

### 6️⃣ Vincular el trigger con Defender for Cloud

1. Ve a **Defender for Cloud → Automation**
2. Crea una nueva regla de automatización:

   * **Nombre**: `responder-logicapp`
   * Condición: *All alerts* (o tipo específico)
   * Acción: **Call Logic App**
   * Selecciona: `responder-alertas-seguridad`

---

## ✅ Validación

* ¿Está Defender for Cloud activo en la suscripción?
* ¿Puedes ver recomendaciones y alertas de seguridad?
* ¿La Logic App se ejecuta cuando se detecta una alerta?
* ¿Recibes un correo o se lanza la acción configurada?

---

## ⚠️ Errores comunes

| Problema                | Causa común                                     | Solución                                             |
| ----------------------- | ----------------------------------------------- | ---------------------------------------------------- |
| No se generan alertas   | No hay actividad sospechosa                     | Usa `curl testmyids.com` o expón la VM temporalmente |
| Logic App no se ejecuta | No está conectada con Defender                  | Revisa la sección de "Automations" en Defender       |
| No llegan correos       | Acción mal configurada o credenciales inválidas | Verifica el conector de correo y permisos            |

---

## 🧩 Extensiones opcionales

* En lugar de un email, haz que la Logic App:

  * **Apague una VM**
  * **Cree un ticket en Azure DevOps o ServiceNow**
  * **Notifique por Teams o Slack**
* Usa Azure Sentinel para crear una **investigación automática** a partir de la alerta.
* Exporta alertas a **Log Analytics** para análisis histórico.
