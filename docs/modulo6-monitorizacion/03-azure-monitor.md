# 🔭 Azure Monitor: Métricas, Logs, Alertas y Workbooks

## 🧭 Introducción

**Azure Monitor** es el servicio centralizado de observabilidad en Microsoft Azure. Proporciona una infraestructura robusta para recolectar, analizar y actuar sobre:

* **Métricas** (rendimiento, disponibilidad)
* **Logs** (auditoría, actividad, errores)
* **Alertas** (en tiempo real)
* **Dashboards** (visualización y análisis)

A diferencia de AWS, donde diferentes servicios gestionan distintos aspectos (CloudWatch, CloudTrail, Config...), Azure Monitor **integra todo en una única interfaz**, con un enfoque más unificado.

Su verdadero poder emerge al combinarlo con herramientas como:

* **Log Analytics** (consultas con KQL)
* **Diagnostic Settings** (captura de eventos)
* **Application Insights** (observabilidad de aplicaciones)
* **Workbooks** (paneles interactivos)

---

## 🧱 Componentes clave de Azure Monitor

### 1. **Métricas**

* Métricas estándar para cada recurso (VM, App Services, SQL, etc.)
* Recolección cada 1 minuto (o menos con premium)
* Alta eficiencia para análisis de rendimiento

📌 Ejemplos:

* `Percentage CPU` (VM)
* `Requests` (App Service)
* `DTU Usage` (SQL Database)

📍 Se visualizan en:

* Azure Portal (gráficas)
* Dashboards
* Workbooks

---

### 2. **Logs (Log Analytics)**

* **Consultas avanzadas en KQL** (Kusto Query Language)
* Logs de recursos, seguridad, actividad
* Centralizados en un **Log Analytics Workspace**
* Consultas en tiempo real o programadas

📌 Ejemplos de consultas:

```kql
SigninLogs
| where ResultType != 0
| summarize FailedLogins = count() by bin(TimeGenerated, 1h)
```

```kql
AzureDiagnostics
| where ResourceType == "MICROSOFT.STORAGE/STORAGEACCOUNTS"
| where OperationName == "GetBlob"
```

---

### 3. **Diagnostic Settings**

Permite **enviar logs y métricas a otros destinos**:

* **Log Analytics Workspace**
* **Azure Storage Account**
* **Event Hub (integración con SIEM)**

📌 Se configura por recurso.
Ejemplos de categorías:

* `AuditLogs`
* `SignInLogs`
* `AppServiceConsoleLogs`

---

### 4. **Alertas**

* **Reglas de alerta** basadas en:

  * Métricas
  * Consultas de log (KQL)
  * Activity Logs (plano de control)
* Asociadas a **Action Groups**:

  * Email
  * Webhook
  * Azure Function
  * ITSM, etc.

📌 Ejemplo práctico:

```kql
SigninLogs
| where ResultType != 0 and Location != "Spain"
| summarize Count = count() by bin(TimeGenerated, 15m)
```

Si `Count > 3`, lanzar alerta por intento de acceso no autorizado desde el extranjero.

---

### 5. **Workbooks**

* Dashboards dinámicos e interactivos
* Basados en resultados de KQL
* Permiten combinar:

  * Gráficos (barras, líneas, KPI)
  * Mapas geográficos
  * Filtros interactivos
  * Texto explicativo

🧠 Ideal para crear paneles de seguridad, actividad de red, uso de recursos, cumplimiento normativo, etc.

---

## 🧪 Caso de uso: detectar creación de recursos sin etiquetas o intentos de login sospechosos

### Escenario 1: Recursos sin etiquetas obligatorias

Consulta KQL:

```kql
Resources
| where isnull(tags.environment)
```

📌 Se puede usar para:

* Auditoría diaria de cumplimiento
* Alertar al equipo de gobernanza
* Automatizar correcciones con Azure Policy

---

### Escenario 2: Login fallido múltiple

Consulta KQL:

```kql
SigninLogs
| where ResultType != 0
| summarize FailedLogins = count() by bin(TimeGenerated, 5m)
| where FailedLogins > 5
```

Se vincula a una **alerta programada** y se configura un **Action Group** para:

* Enviar email
* Invocar un Logic App (bloquear IP)
* Notificar por Teams o Slack

---

## 💻 Código y configuración

### 🔧 Crear configuración de diagnóstico (CLI)

```bash
az monitor diagnostic-settings create \
  --name "enable-logs" \
  --resource "/subscriptions/xxxx/resourceGroups/rg-demo/providers/Microsoft.Web/sites/mi-app" \
  --workspace "/subscriptions/xxxx/resourceGroups/rg-monitor/providers/Microsoft.OperationalInsights/workspaces/mi-workspace" \
  --logs '[{"category":"AppServiceConsoleLogs","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

---

### 🔧 Crear una alerta basada en log (KQL)

```bash
az monitor scheduled-query create \
  --name "LoginFailuresAlert" \
  --resource-group rg-monitor \
  --workspace mi-workspace \
  --scopes "/subscriptions/xxxx" \
  --description "Más de 5 fallos de login en 5 min" \
  --query "SigninLogs | where ResultType != 0 | summarize count() by bin(TimeGenerated, 5m) | where count_ > 5" \
  --action "/subscriptions/xxxx/resourceGroups/rg-monitor/providers/microsoft.insights/actionGroups/mi-alert-group"
```

---

## 🔍 Tips para probar

| Acción                        | Herramienta                                           |
| ----------------------------- | ----------------------------------------------------- |
| Consultar métricas de VM      | Azure Portal > VM > Monitoring                        |
| Buscar logs de actividad      | Azure Monitor > Logs                                  |
| Probar alerta de login        | Iniciar sesión con contraseña incorrecta varias veces |
| Visualizar datos en Workbooks | Azure Monitor > Workbooks > +Nuevo                    |
| Exportar logs a Event Hub     | Configurar Diagnostic Settings + destino Event Hub    |

---

## ❌ Errores comunes

* **No habilitar Diagnostic Settings** → no se capturan logs detallados.
* Usar solo métricas básicas → sin trazabilidad real.
* Crear alertas sin asociar Action Groups → no hay notificación.
* Falta de estrategia de retención → pérdida de información para auditoría.
* Workbooks vacíos o sin contexto → sin utilidad práctica.

---

## ✅ Buenas prácticas

✔️ Habilitar logs para todos los recursos críticos.
✔️ Centralizar en un único **Log Analytics Workspace**.
✔️ Usar **KQL** para detección avanzada.
✔️ Crear Workbooks por áreas (seguridad, operaciones, red, etc.).
✔️ Revisar y probar alertas regularmente.
✔️ Configurar retención según cumplimiento (ej. 1 año para ISO 27001).

---

## 📚 Recursos oficiales

* [Azure Monitor overview](https://learn.microsoft.com/en-us/azure/azure-monitor/overview)
* [Log Analytics workspace](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview)
* [KQL Language Reference](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
* [Diagnostic settings](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/diagnostic-settings)
* [Create log alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-log)