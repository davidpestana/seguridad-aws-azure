# 🧠 Buenas Prácticas en Monitorización y Registro en Cloud (AWS y Azure)

## 🧭 Introducción

Una estrategia de monitorización eficaz no se trata solo de activar métricas y logs. Requiere planificación, automatización, centralización, cumplimiento normativo y revisión continua.

Este archivo recopila las **mejores prácticas** para implementar una estrategia de visibilidad sólida, eficiente y alineada con los requisitos operativos y de seguridad de cualquier organización en la nube.

Aplica tanto para entornos en **AWS** como **Azure**.

---

## 🎯 Principios fundamentales

✔️ **Visibilidad completa**: métricas + logs + eventos + trazas
✔️ **Centralización**: evitar la dispersión entre cuentas, regiones o grupos
✔️ **Automatización**: nada manual, especialmente en producción
✔️ **Alertas útiles**: pocas pero significativas
✔️ **Retención adecuada**: legal y técnica
✔️ **Revisión continua**: dashboards, alertas y políticas deben evolucionar

---

## 🏗️ Arquitectura recomendada

| Elemento                   | Recomendación                                       |
| -------------------------- | --------------------------------------------------- |
| 🧩 Logs de actividad       | Activar siempre CloudTrail / Activity Logs          |
| 🧩 Logs de datos           | S3/Object logs en AWS, Diagnostic Settings en Azure |
| 📊 Métricas personalizadas | Enviar desde apps críticas con namespace propio     |
| 🧠 Workspace central       | Log Analytics (Azure), cuenta de logs (AWS)         |
| 📈 Dashboards por rol      | Seguridad, operaciones, negocio                     |
| 🔔 Alertas por severidad   | Alta = correo + webhook; baja = solo dashboard      |

---

## 🔒 Seguridad y cumplimiento

### AWS

* Activar **CloudTrail** en todas las regiones y cuentas
* Habilitar **Data Events** para buckets sensibles y Lambda
* Exportar logs a **S3 versionado + cifrado**
* Aplicar **políticas de retención** (ej. 1 año en logs críticos)
* Revisar permisos de acceso a los logs (S3, CloudWatch Logs)

### Azure

* Configurar **Diagnostic Settings** en todos los recursos
* Centralizar en un **Log Analytics Workspace**
* Usar **Azure Policy** para imponer configuración de logs
* Activar retención mínima legal (31 días o más)
* Monitorizar cambios en logs con Sentinel (opcional)

---

## ⚙️ Automatización

### AWS con CloudFormation / Terraform

```yaml
# CloudTrail + S3 + Logs en CloudFormation
Resources:
  CloudTrail:
    Type: AWS::CloudTrail::Trail
    Properties:
      S3BucketName: !Ref LogsBucket
      IsMultiRegionTrail: true
      IncludeGlobalServiceEvents: true
```

### Azure con ARM / Bicep / Terraform

```bicep
resource diag 'Microsoft.Insights/diagnosticSettings@2021-05-01-preview' = {
  name: 'enable-logs'
  scope: webApp
  properties: {
    workspaceId: logAnalytics.id
    logs: [
      {
        category: 'AppServiceConsoleLogs'
        enabled: true
      }
    ]
    metrics: [
      {
        category: 'AllMetrics'
        enabled: true
      }
    ]
  }
}
```

📌 También puedes usar:

* **AWS Config** para detectar falta de logs
* **Azure Policy** con iniciativa de cumplimiento

---

## 🧪 Validación y pruebas

| Prueba                                         | Objetivo                        |
| ---------------------------------------------- | ------------------------------- |
| 🔧 Desplegar recurso sin logs                  | Ver si la política lo detecta   |
| 🕵️‍♂️ Simular acceso no autorizado            | Validar que alerta se dispare   |
| 🚨 Desconectar recurso crítico                 | Evaluar visibilidad y respuesta |
| 📬 Cortar salida a SNS / Action Group          | Ver si se notifica el fallo     |
| 📊 Revisar dashboards vacíos o desactualizados | Garantizar que están en uso     |

---

## ⚠️ Errores frecuentes

| Error                                     | Consecuencia                        |
| ----------------------------------------- | ----------------------------------- |
| ❌ Dejar retención por defecto (7–30 días) | Pérdida de logs ante investigación  |
| ❌ Activar solo plano de control           | No se registra acceso real a datos  |
| ❌ No exportar logs                        | Pérdida irreversible tras retención |
| ❌ Usar alarmas triviales                  | Saturación y falsas alarmas         |
| ❌ No revisar alertas antiguas             | Quedan obsoletas o inservibles      |
| ❌ Dashboards sin filtros ni contexto      | Inutilizables por equipos distintos |

---

## ✅ Checklist rápida por entorno

### ✅ AWS

* [ ] CloudTrail activado (multi-región)
* [ ] Logs a S3 versionado y cifrado
* [ ] Data Events activados en buckets críticos
* [ ] Log metric filters para seguridad
* [ ] Al menos un dashboard por dominio (seguridad, ops)
* [ ] Alarmas integradas con SNS y/o Lambda

### ✅ Azure

* [ ] Diagnostic Settings en todos los recursos
* [ ] Workspace central para logs
* [ ] Alertas con KQL + Action Groups
* [ ] Workbooks compartidos por rol
* [ ] Retención > 90 días o export a Storage
* [ ] Integración con Sentinel si aplica

---

## 📚 Recursos recomendados

* **AWS**

  * [Security Best Practices for CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/best-practices.html)
  * [AWS CloudTrail Best Practices](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-best-practices.html)
  * [Logging Strategies Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/security-monitoring-strategies/)

* **Azure**

  * [Azure Monitor Logging Strategy](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/logging-strategy)
  * [Azure Policy for Monitoring](https://learn.microsoft.com/en-us/azure/governance/policy/samples/monitor)
  * [Security Workbook Gallery](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-gallery)

---

## 🏁 Conclusión

Una estrategia de monitorización efectiva **no es solo técnica**, también es organizativa.
Implica colaboración entre seguridad, operaciones y arquitectura para:

* Tener visibilidad real
* Detectar antes que reaccionar
* Cumplir normativas y evitar sanciones
* Tomar decisiones informadas sobre el estado de los sistemas

🧭 La nube da herramientas poderosas: **la clave está en usarlas bien y de forma consistente.**