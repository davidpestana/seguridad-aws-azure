# 🚨 Alertas y Métricas de Seguridad (AWS y Azure)

## 🧭 Introducción

La monitorización pasiva —ver métricas y logs “a mano”— no es suficiente en entornos cloud productivos. Es necesario implementar **sistemas de alerta automatizados** que detecten y avisen de comportamientos anómalos, fallos de seguridad o degradaciones del sistema **en tiempo real**.

Este archivo aborda cómo definir alertas efectivas en AWS y Azure, con especial foco en la **seguridad**: actividad sospechosa, errores repetidos, configuraciones peligrosas, etc.

---

## 🎯 ¿Qué hace a una alerta efectiva?

Una alerta útil debe:

✅ Detectar una **condición real de riesgo**
✅ Ser **accionable** (saber qué hacer al recibirla)
✅ Evitar el **ruido excesivo** (alert fatigue)
✅ Integrarse con sistemas de respuesta o equipos SOC

---

## 🔧 AWS: Alertas con CloudWatch Alarms y SNS

### 1. Crear una alarma de seguridad

Ejemplo: alertar si se crea una política IAM con permisos totales (`"Action": "*"` y `"Resource": "*"`)

Primero, habilita CloudTrail + CloudWatch Logs.

Luego, crea una métrica de log:

```bash
aws logs put-metric-filter \
  --log-group-name /aws/cloudtrail/logs \
  --filter-name "IAMFullAccessPolicy" \
  --filter-pattern '{ ($.eventName = CreatePolicy) && ($.requestParameters.policyDocument.policyDocument.Statement[0].Action = "*") && ($.requestParameters.policyDocument.policyDocument.Statement[0].Resource = "*") }' \
  --metric-transformations metricName=FullAccessPolicyCreated,metricNamespace=SecurityAlerts,metricValue=1
```

Después, crea una alarma sobre esa métrica:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "FullAccessPolicyAlarm" \
  --metric-name FullAccessPolicyCreated \
  --namespace SecurityAlerts \
  --statistic Sum \
  --period 60 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:SecurityAlertsTopic
```

### 2. Notificación con SNS

Crea un topic y suscripción:

```bash
aws sns create-topic --name SecurityAlertsTopic
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:123456789012:SecurityAlertsTopic \
  --protocol email \
  --notification-endpoint alerta@empresa.com
```

📌 Puedes usar SNS también para invocar una Lambda, webhook o enviar a otros sistemas (Slack, Teams...).

---

## 🔧 Azure: Alertas con KQL + Action Groups

### 1. Alerta por intentos de login fallidos en Azure AD

Primero, habilita el envío de logs a Log Analytics.

Consulta KQL:

```kql
SigninLogs
| where ResultType != 0
| summarize FailedLogins = count() by bin(TimeGenerated, 5m)
| where FailedLogins > 10
```

Luego crea una alerta basada en esa consulta, asociada a un **Action Group**.

```bash
az monitor scheduled-query create \
  --name "LoginFailures" \
  --resource-group rg-monitor \
  --workspace "workspace-id" \
  --scopes "/subscriptions/xxx" \
  --query "SigninLogs | where ResultType != 0 | summarize count() by bin(TimeGenerated, 5m) | where count_ > 10" \
  --description "Detectar múltiples fallos de login" \
  --action "/subscriptions/xxx/resourceGroups/rg-monitor/providers/microsoft.insights/actionGroups/SOCAlertGroup"
```

### 2. Acciones posibles

Los Action Groups permiten:

* Enviar emails / SMS / notificaciones push
* Ejecutar Azure Function, Logic App
* Publicar en Webhook o ITSM
* Integración con **Microsoft Sentinel**

---

## 🧠 Integración con SIEM

### En AWS

* Alarma → SNS → Security Hub
* Security Hub puede integrarse con Amazon Detective o exportar a SIEM externo

### En Azure

* Azure Monitor se integra directamente con **Microsoft Sentinel**
* Sentinel puede correlacionar múltiples fuentes (Office 365, Azure AD, Defender...)

---

## ⚠️ Evitar alert fatigue

Demasiadas alertas irrelevantes → se ignoran las importantes.

🛡️ Recomendaciones:

* Solo alertar en condiciones anómalas o críticas
* Usar niveles de severidad
* Correlacionar eventos para evitar duplicidades
* Revisar métricas de ruido (ej. ratio de alertas atendidas)

---

## 📊 Ejemplos prácticos (AWS ↔ Azure)

| **Caso de seguridad**                   | **AWS (CloudWatch)**                                         | **Azure (Monitor + KQL)**                                      |                                              |
| --------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------- | -------------------------------------------- |
| Intentos de login fallidos              | Métrica a partir de CloudTrail logs filtrados → alarma + SNS | `SigninLogs` con `ResultType != 0` → alerta basada en KQL      |                                              |
| Bucket/Blob se vuelve público           | Métrica por cambio de ACL o `PutBucketPolicy` en CloudTrail  | Azure Policy + `AzureDiagnostics` para detectar `PublicAccess` |                                              |
| Creación de recursos sin etiquetas      | Filtro en CloudTrail + métrica personalizada                 | KQL: \`Resources                                               | where isnull(tags.ambiente)\`                |
| Acceso a datos fuera de horario laboral | Correlación de `GetObject` con hora (CloudTrail + horario)   | `StorageRead` filtrado por hora local en KQL                   |                                              |
| Eliminación de recursos                 | `Delete*` en CloudTrail logs                                 | \`AzureActivity                                                | where OperationNameValue contains "Delete"\` |

---

## ✅ Buenas prácticas

✔️ Usar métricas + logs como base de detección.
✔️ Agrupar alertas por contexto (ej. acceso IAM, datos, red).
✔️ Aplicar umbrales razonables y evitar falsos positivos.
✔️ Integrar con flujos de respuesta automatizada.
✔️ Testear alertas regularmente (simulación de incidente).
✔️ Usar dashboards de seguimiento para alertas activas.

---

## ❌ Errores comunes

* Crear alertas sin acciones → se disparan pero nadie las ve
* No probar las alertas → confianza falsa en que “están funcionando”
* Exceso de alertas triviales → ignorancia por saturación
* No registrar plano de datos → sin base para alertas de acceso real
* Alertas sin severidad o sin clasificación

---

## 📚 Recursos oficiales

* **AWS**

  * [Using Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)
  * [SNS Setup Guide](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html)

* **Azure**

  * [Log Alerts with KQL](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-log)
  * [Action Groups](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/action-groups)
  * [Integrate Monitor with Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/connect-azure-monitor)