## 📊 Módulo 6: Monitorización y Registro de Actividades (AWS y Azure)

---

## 🎯 Objetivos del módulo

La seguridad no solo consiste en prevenir ataques, sino también en **detectar y responder**. Este módulo cubre cómo recolectar métricas, logs y eventos de seguridad en AWS y Azure, correlacionarlos y generar alertas.

Al terminar el módulo, serás capaz de:

* Diferenciar métricas, logs y trazas en entornos cloud.
* Configurar servicios de monitorización nativos (CloudWatch, Azure Monitor).
* Habilitar auditoría detallada (CloudTrail, Activity Logs, Diagnostic Settings).
* Crear alertas basadas en métricas y eventos de seguridad.
* Construir dashboards de visibilidad operativa y de seguridad.
* Adoptar buenas prácticas para centralizar y conservar logs según normativas.

---

## 🧭 Índice detallado del módulo

---

### 1) Introducción a la monitorización en cloud

✅ Objetivo
Comprender el rol de la monitorización como parte de la seguridad cloud y por qué los logs son la “caja negra” de la infraestructura.

📌 Claves
Métricas vs Logs vs Eventos · SIEM · Observabilidad · Retención

🧩 Contenido

* La monitorización no es solo rendimiento → también seguridad.
* Diferencias:

  * **Métricas**: números agregados (CPU %, tráfico, latencia).
  * **Logs**: registros detallados de eventos (API calls, accesos, errores).
  * **Eventos**: sucesos clave disparados por sistemas (alerta de login sospechoso).
* Concepto de **observabilidad**: visibilidad completa en métricas, logs y trazas.
* **Retención**: clave para normativas (ej. GDPR, PCI DSS, ISO 27001).

Ejemplo: detectar un acceso sospechoso a S3/Azure Blob → requiere métricas (picos de tráfico) + logs (quién accedió) + alertas (enviar notificación).
📚 docs/modulo6-monitorizacion/01-introduccion.md

---

### 2) CloudWatch en AWS

✅ Objetivo
Aprender a usar CloudWatch como plataforma de métricas, logs, dashboards y alarmas.

📌 Claves
Metrics · Logs · Events · Dashboards · Alarms

🧩 Contenido

* **Métricas nativas**: CPU, red, memoria (parcial), latencia de balanceadores.
* **Custom metrics**: enviar métricas propias con la API/agent.
* **Logs**: integración con CloudWatch Logs (EC2, Lambda, aplicaciones).
* **Events/EventBridge**: detección de cambios en estado o seguridad (ej. creación de recursos, IAM cambios).
* **Alarmas**: thresholds en métricas (ej. CPU > 80% → alerta + autoescalado).
* **Dashboards**: paneles de estado personalizables.

Caso de uso: monitorizar un clúster web y disparar autoescalado si CPU > 75% durante 5 min.
📚 docs/modulo6-monitorizacion/02-cloudwatch.md

---

### 3) Azure Monitor

✅ Objetivo
Usar Azure Monitor como hub de métricas, logs y alertas en entornos Microsoft.

📌 Claves
Metrics · Logs (KQL) · Diagnostic Settings · Alerts · Workbooks

🧩 Contenido

* **Métricas nativas**: rendimiento de VM, bases de datos, App Services.
* **Logs centralizados**: enviados a Log Analytics Workspace, consultables en KQL.
* **Diagnostic Settings**: configuración para enviar logs a Monitor, Event Hub o Storage.
* **Alertas**: reglas basadas en métricas, logs o activity logs.
* **Workbooks**: dashboards gráficos personalizables.

Ejemplo: alertar cuando se cree un recurso sin tags obligatorios o cuando haya más de 5 intentos de login fallidos en Azure AD.
📚 docs/modulo6-monitorizacion/03-azure-monitor.md

---

### 4) Logs y auditoría

✅ Objetivo
Registrar cada acción relevante en la nube para trazabilidad, auditoría y cumplimiento.

📌 Claves
CloudTrail · Activity Logs · Data Plane vs Control Plane · Retención

🧩 Contenido

* **AWS CloudTrail**:

  * Registra todas las llamadas a la API de AWS (quién, qué, desde dónde).
  * Distingue **Control Plane** (gestión de recursos) y **Data Plane** (uso de recursos, ej. leer objetos).
  * Puede enviar eventos a CloudWatch Logs o S3.

* **Azure Activity Logs**:

  * Registra operaciones administrativas a nivel de suscripción y recursos.
  * Complementado con **Diagnostic Logs** para operaciones de datos.
  * Exportable a Log Analytics, Event Hub o Storage.

Ejemplo: investigar “¿quién borró este bucket/VM?”. Sin logs, no hay respuesta. Con auditoría, trazabilidad total.
📚 docs/modulo6-monitorizacion/04-logs-y-auditoria.md

---

### 5) Alertas y métricas de seguridad

✅ Objetivo
Detectar problemas de seguridad y rendimiento en tiempo real mediante alertas.

📌 Claves
Thresholds · Logs queries · Notificaciones · Integración con SOC

🧩 Contenido

* **AWS**:

  * CloudWatch Alarms + SNS para alertar por correo, SMS, Lambda.
  * Ejemplo: alerta si se crea una policy IAM con `*:*`.

* **Azure**:

  * Alert Rules sobre métricas o consultas KQL en Log Analytics.
  * Ejemplo: alerta si hay más de 10 intentos de login fallidos en 5 min.

* Integración con SIEM: CloudWatch → Security Hub; Azure Monitor → Sentinel.

* Buen diseño = evitar “alert fatigue” (demasiadas alertas triviales).
  📚 docs/modulo6-monitorizacion/05-alertas-y-metricas.md

---

### 6) Visualización y dashboards

✅ Objetivo
Consolidar datos en paneles que muestren el estado de seguridad y salud de la infraestructura.

📌 Claves
Dashboards · Widgets · Workbooks · KPIs de seguridad

🧩 Contenido

* **AWS CloudWatch Dashboards**: métricas + logs en gráficos personalizables.
* **Azure Workbooks**: paneles interactivos con KQL y visualizaciones ricas.
* Ejemplo: un panel que muestre → uso de CPU, intentos fallidos de login, buckets públicos, recursos sin tags.
* Importancia de mostrar **KPIs de seguridad** (ej. nº de usuarios sin MFA, nº de recursos expuestos públicamente).

📚 docs/modulo6-monitorizacion/06-visualizacion-y-dashboards.md

---

### 7) Buenas prácticas en monitorización

✅ Objetivo
Diseñar una estrategia de monitorización efectiva, útil y sostenible.

📌 Claves
Centralización · Retención adecuada · Automatización · Detección temprana

🧩 Contenido

* Centralizar logs en una cuenta/subscripción dedicada.
* Configurar retención según compliance (90 días mínimo, 1-7 años en entornos regulados).
* Automatizar despliegue de diagnósticos/logs (AWS Config, Azure Policy).
* Usar alertas accionables, no solo métricas de CPU.
* Integrar con SOC o SIEM para correlación avanzada.
* Testear alertas con simulaciones de incidentes.

Errores comunes:

* Dejar logs en “default” (retención mínima, sin export).
* No activar auditoría de plano de datos → se pierden accesos reales a datos.
* Alertas mal configuradas → nadie responde porque son demasiado ruidosas.

📚 docs/modulo6-monitorizacion/07-buenas-practicas.md