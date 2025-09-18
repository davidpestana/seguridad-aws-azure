# 📊 Introducción a la Monitorización en Cloud (AWS y Azure)

## 🧭 Introducción

La monitorización es una **pieza fundamental en la seguridad cloud**, aunque a menudo se confunde con un componente puramente operativo. En realidad, **sin monitorización no hay detección** y, por tanto, no hay capacidad de respuesta ante amenazas o fallos.

En entornos como AWS y Azure, donde la infraestructura puede escalar dinámicamente y los recursos se aprovisionan de forma efímera, la **visibilidad continua** de lo que ocurre —quién hace qué, cuándo, cómo y desde dónde— se vuelve esencial.

Como analogía, podríamos decir que los logs, métricas y trazas son la **“caja negra”** de nuestros entornos cloud: no evitan el incidente, pero permiten **entenderlo, rastrearlo y prevenir su repetición**.

---

## 🔍 Diferencias clave: métricas, logs y eventos

### 📈 Métricas

* Datos numéricos agregados.
* Se recopilan en intervalos regulares.
* Ejemplos:

  * Uso de CPU (%)
  * Número de peticiones por segundo
  * Latencia promedio

📌 **AWS**: CloudWatch Metrics
📌 **Azure**: Azure Monitor Metrics

---

### 📄 Logs

* Registros textuales detallados de eventos individuales.
* Incluyen contexto: qué recurso, qué usuario, desde dónde.
* Se generan de forma asincrónica, no en intervalos fijos.
* Ejemplos:

  * Registro de acceso a un bucket S3 o Blob
  * Cambios en una policy de IAM
  * Errores de aplicación

📌 **AWS**: CloudWatch Logs, CloudTrail
📌 **Azure**: Log Analytics, Activity Logs

---

### ⚡ Eventos

* Notificaciones de sucesos relevantes que pueden activar acciones automáticas.
* Útiles para disparar flujos de trabajo (automatización).
* Ejemplos:

  * Creación de una instancia EC2 o VM
  * Eliminación de un recurso crítico
  * Detectar login sospechoso

📌 **AWS**: EventBridge
📌 **Azure**: Azure Event Grid / Action Groups

---

## 🔭 Observabilidad en la nube

La **observabilidad** es la capacidad de comprender el estado interno de un sistema a partir de sus salidas externas. En la nube, se apoya en:

* **Métricas**: visión numérica del rendimiento.
* **Logs**: trazabilidad detallada de eventos.
* **Trazas (traces)**: seguimiento distribuido entre servicios.

📌 AWS y Azure ofrecen herramientas integradas, pero en entornos híbridos o multi-cloud se pueden centralizar con soluciones como:

* **SIEMs** (Security Information and Event Management)
* **Grafana, Prometheus, Datadog**, etc.

---

## 🛡️ Retención de datos y cumplimiento

La duración durante la cual se conservan logs y métricas es crítica para:

* **Auditorías de seguridad**
* **Investigaciones forenses**
* **Cumplimiento normativo** (GDPR, ISO 27001, PCI DSS)

| **Plataforma** | **Retención por defecto** | **Retención configurable**      |
| -------------- | ------------------------- | ------------------------------- |
| AWS CloudTrail | 90 días                   | Hasta 7 años (en S3)            |
| Azure Logs     | 31 días                   | Hasta 2 años (o más con export) |

🔒 Es una **mala práctica** dejar los valores por defecto si tu organización tiene requisitos legales o de cumplimiento.

---

## 🧪 Ejemplo empresarial

> **Escenario:** Una empresa detecta que un archivo confidencial almacenado en S3/Azure Blob fue accedido fuera del horario laboral.

### ¿Cómo se detecta?

* **Métrica**: Pico anómalo en el tráfico de red del bucket/contenedor.
* **Log**: Registro de acceso a nivel de objeto (IP, usuario, hora).
* **Evento**: Notificación disparada por política de seguridad.

### ¿Cómo se responde?

1. Revisión de logs históricos → quién accedió, desde dónde, cuándo.
2. Validación de alertas → ¿alerta falló? ¿reglas mal configuradas?
3. Aplicación de contramedidas → revocar accesos, actualizar policies, informar al equipo de cumplimiento.

---

## 💻 Código y configuración

### 🔧 AWS – Habilitar métricas y logs para un bucket S3

```bash
aws s3api put-bucket-logging \
  --bucket mi-bucket \
  --bucket-logging-status '{
    "LoggingEnabled": {
      "TargetBucket": "mi-bucket-logs",
      "TargetPrefix": "logs/"
    }
  }'
```

```bash
aws cloudtrail create-trail \
  --name trail-global \
  --s3-bucket-name mi-bucket-logs
```

---

### 🔧 Azure – Configurar diagnóstico para Blob Storage

```bash
az monitor diagnostic-settings create \
  --resource "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<storage>" \
  --name "blob-logging" \
  --workspace "<log-analytics-id>" \
  --logs '[{"category":"StorageRead","enabled":true},{"category":"StorageWrite","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

---

## 🧪 Tips para probar

| Acción                              | AWS                          | Azure                     |
| ----------------------------------- | ---------------------------- | ------------------------- |
| Ver logs de acceso a un bucket/blob | CloudTrail + CloudWatch Logs | Log Analytics             |
| Consultar métricas en tiempo real   | CloudWatch Metrics           | Azure Monitor             |
| Disparar una alerta                 | EventBridge rule + SNS       | Alert Rule + Action Group |
| Exportar logs a almacenamiento      | S3                           | Azure Storage o Event Hub |

---

## ❌ Errores comunes

* **No activar logs** de acceso a recursos sensibles.
* Conservar logs por solo 7–30 días sin exportación.
* No separar métricas de seguridad y operativas.
* Ignorar las trazas (útiles en microservicios).
* Crear alertas pero no probarlas → falsa sensación de seguridad.

---

## ✅ Buenas prácticas

✔️ Activar logs y métricas desde el día 1.
✔️ Exportar logs críticos a almacenamiento duradero.
✔️ Establecer políticas de retención y acceso.
✔️ Revisar alertas de forma periódica.
✔️ Hacer simulacros de incidentes para validar visibilidad.
✔️ Usar dashboards para KPIs de seguridad (intentos fallidos, accesos fuera de horario, etc.)

---

## 📚 Recursos recomendados

* **AWS**

  * [Amazon CloudWatch](https://docs.aws.amazon.com/cloudwatch/)
  * [AWS CloudTrail](https://docs.aws.amazon.com/awscloudtrail/)
  * [Security Monitoring Strategies](https://docs.aws.amazon.com/whitepapers/latest/security-monitoring-strategies/)

* **Azure**

  * [Azure Monitor overview](https://learn.microsoft.com/en-us/azure/azure-monitor/overview)
  * [Log Analytics workspace](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview)
  * [Diagnostic Settings](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/diagnostic-settings)