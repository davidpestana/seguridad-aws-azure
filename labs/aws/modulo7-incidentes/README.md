# 🧪 Laboratorio 07 – Detección de amenazas con Amazon GuardDuty y automatización de respuesta

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Activar **Amazon GuardDuty** para detección de amenazas.
* Simular un comportamiento sospechoso para generar una alerta.
* Revisar los hallazgos de seguridad desde la consola.
* Automatizar una **respuesta a incidentes** usando EventBridge.

---

## 🧵 Escenario práctico

Tu organización quiere mejorar su capacidad de detectar amenazas y responder automáticamente a ciertos eventos de seguridad. En este laboratorio, vas a activar **GuardDuty**, simular una amenaza y generar una acción automática (por ejemplo, enviar una alerta o detener una instancia) mediante **EventBridge**.

---

## 🧰 Requisitos previos

* Cuenta AWS con permisos administrativos.
* Región soportada por GuardDuty (ej: `us-east-1`).
* Instancia EC2 lanzada previamente (opcional para respuesta automática).
* AWS CLI (opcional para validación).

---

## 🧭 Pasos detallados

### 1️⃣ Activar Amazon GuardDuty

1. Ve a: [https://console.aws.amazon.com/guardduty](https://console.aws.amazon.com/guardduty)
2. Haz clic en **Enable GuardDuty**
3. Acepta las opciones por defecto:

   * Protección de cuentas
   * Protección de S3
   * Protección de EKS (si aplica)

✅ En unos minutos, GuardDuty comenzará a analizar logs y tráfico DNS/VPC.

---

### 2️⃣ Simular una amenaza para generar hallazgos

AWS permite simular amenazas controladas para probar GuardDuty:

```bash
aws guardduty create-sample-findings \
  --detector-id $(aws guardduty list-detectors --query 'DetectorIds[0]' --output text)
```

Esto genera hallazgos simulados como intentos de acceso desde IPs maliciosas o comportamiento anómalo.

---

### 3️⃣ Revisar los hallazgos en consola

1. Ve a **Findings**
2. Aplica el filtro **Type = Recon\:EC2/PortProbeUnprotectedPort**
3. Haz clic en un hallazgo para ver:

   * IP de origen
   * Recurso afectado
   * Severidad
   * Recomendaciones

✅ Puedes usar estos hallazgos como desencadenantes para automatizaciones.

---

### 4️⃣ Crear una regla de EventBridge para responder

#### A. Ve a **Amazon EventBridge → Rules → Create rule**

1. Nombre: `respuesta-guardduty`
2. Event bus: default
3. Event source: `AWS services`
4. Event pattern: **GuardDuty finding**

```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [{ "numeric": [">=", 5] }]
  }
}
```

#### B. Acción de respuesta (ejemplo: notificación)

1. Target: **SNS topic** o **Lambda function**

   * Puedes crear una función Lambda que imprima el evento
   * O usar un topic SNS con email de alerta

✅ GuardDuty → EventBridge → Acción automatizada

---

### 5️⃣ (Opcional) Crear una Lambda de respuesta

1. Ve a **Lambda → Create function**
2. Nombre: `responder-guardduty`
3. Runtime: Python 3.x
4. Código:

```python
def lambda_handler(event, context):
    print("🚨 Alerta de GuardDuty recibida:")
    print(event)
```

5. Asocia esta Lambda como **target** de la regla de EventBridge creada antes.

---

## ✅ Validación

* ¿GuardDuty está activo?
* ¿Se han generado hallazgos de muestra?
* ¿Puedes ver los detalles de un hallazgo?
* ¿Tu regla de EventBridge ha capturado el evento?
* (Opcional) ¿Lambda o SNS ejecutó la respuesta automáticamente?

---

## ⚠️ Errores comunes

| Problema                 | Causa común                                 | Solución                                                  |
| ------------------------ | ------------------------------------------- | --------------------------------------------------------- |
| No aparecen hallazgos    | GuardDuty recién activado                   | Usa `create-sample-findings` para generar                 |
| EventBridge no reacciona | Patrón mal definido o servicio no conectado | Revisa JSON del evento y configuración del target         |
| Lambda no se ejecuta     | Permisos faltantes                          | Añade permisos a Lambda para ser invocada por EventBridge |

---

## 🧩 Extensiones opcionales

* Automatiza el **apagado de la instancia EC2** afectada (acción en Lambda).
* Integra con **AWS Security Hub** para orquestar respuestas globales.
* Añade **filtros por tipo de amenaza** o servicio afectado.
* Enviar alertas a **Slack, Teams o correo** usando SNS.