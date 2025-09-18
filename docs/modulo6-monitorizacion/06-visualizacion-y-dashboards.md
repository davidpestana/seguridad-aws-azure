# 📊 Visualización y Dashboards en AWS y Azure

## 🧭 Introducción

La monitorización efectiva no termina en la recolección de métricas y logs. La clave está en **visualizar esa información de forma clara, contextual y útil** para diferentes roles: técnicos, seguridad, operaciones o directivos.

En este archivo aprenderás a construir **dashboards** que consoliden datos relevantes de seguridad y operatividad, usando las herramientas nativas de:

* **AWS CloudWatch Dashboards**
* **Azure Workbooks**

El objetivo es ofrecer **visibilidad en tiempo real** y capacidad de análisis desde una única vista.

---

## 🧰 AWS: CloudWatch Dashboards

### Características

* Paneles personalizados con **widgets** (gráficos, texto, KPIs).
* Soporte para métricas, logs y expresiones matemáticas.
* Se pueden compartir por cuenta o con IAM policies específicas.
* Permiten visualizar hasta 500 widgets por dashboard.

### Tipos de widgets

* **Line / stacked / number widgets**: para métricas continuas.
* **Text widgets**: para anotaciones o instrucciones.
* **Log query widgets**: desde CloudWatch Logs Insights.

### Ejemplo: Crear dashboard vía CLI

```bash
aws cloudwatch put-dashboard \
  --dashboard-name "SecurityOverview" \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "x": 0, "y": 0, "width": 12, "height": 6,
        "properties": {
          "metrics": [
            [ "AWS/EC2", "CPUUtilization", "InstanceId", "i-1234567890abcdef0" ]
          ],
          "period": 300,
          "stat": "Average",
          "title": "CPU Utilization"
        }
      }
    ]
  }'
```

### Desde consola

1. Ir a **CloudWatch > Dashboards > Create Dashboard**
2. Añadir widgets:

   * CPU usage
   * Login failures (desde log metric filter)
   * Public S3 buckets (desde métricas o script externo)
   * Texto con recomendaciones

---

## 🧰 Azure: Workbooks

### Características

* Paneles interactivos basados en **KQL + visualizaciones**
* Pueden incluir:

  * **Filtros** y variables interactivas
  * **Gráficos** (líneas, barras, áreas)
  * **Mapas geográficos**
  * **Texto, imágenes, markdown**
* Diseñados para equipos SOC, operaciones o gobernanza

### Crear un Workbook desde consola

1. Ir a **Azure Monitor > Workbooks**
2. Crear uno nuevo
3. Añadir pasos de consulta:

```kql
SigninLogs
| where ResultType != 0
| summarize FailedLogins = count() by bin(TimeGenerated, 1h)
```

4. Añadir gráficos: línea temporal, KPI con color, lista de usuarios, etc.

### Elementos comunes

* **Gráficos de tendencias**: CPU, almacenamiento, errores
* **KPIs**: intentos de login fallidos, buckets públicos, usuarios sin MFA
* **Listados filtrables**: recursos sin etiquetas, cambios recientes
* **Filtros dinámicos**: por región, equipo, severidad

---

## 🧪 Ejemplo empresarial: dashboard de seguridad

> Escenario: La empresa quiere una vista unificada con indicadores de seguridad clave.

| KPI / gráfico                  | ¿Qué muestra?           | AWS                              | Azure                                  |                          |
| ------------------------------ | ----------------------- | -------------------------------- | -------------------------------------- | ------------------------ |
| CPU de instancias clave        | Carga de trabajo        | CloudWatch Metric                | VM metrics                             |                          |
| Intentos de login fallidos     | Brute force sospechoso  | CloudTrail → log metric          | KQL sobre `SigninLogs`                 |                          |
| Recursos sin tags obligatorios | Problemas de gobernanza | Script externo + CloudWatch      | \`Resources                            | where isnull(tags.app)\` |
| Buckets/Blobs públicos         | Riesgo de fuga de datos | `PutBucketPolicy` logs           | `StorageDiagnostics` logs              |                          |
| Actividad fuera de horario     | Anomalías               | CloudTrail con expresión horaria | KQL con `where hour(TimeGenerated)`    |                          |
| Accesos sin MFA                | Falta de control        | IAM user list + scripting        | `SigninLogs` + `AuthenticationDetails` |                          |

---

## 📌 Buenas prácticas de visualización

✅ **Pensar por roles**: SOC ≠ Sysadmin ≠ CISO
✅ Usar **KPIs claros, con umbrales visuales** (colores, iconos)
✅ Separar paneles por tema: acceso, red, cumplimiento, etc.
✅ Incluir **anotaciones o texto explicativo**
✅ Mostrar métricas clave de seguridad: usuarios sin MFA, logs no habilitados, errores críticos, etc.
✅ Mantener los dashboards **actualizados y probados**

---

## ❌ Errores comunes

* Incluir demasiada información irrelevante → dashboards sobrecargados
* Usar solo métricas operativas (CPU, RAM) → sin perspectiva de seguridad
* No incluir widgets de texto o contexto → difícil de interpretar
* No compartir los dashboards → conocimiento encerrado
* Ignorar la personalización por equipo/rol

---

## 🧪 Tips para probar

| Acción                         | AWS                      | Azure                            |
| ------------------------------ | ------------------------ | -------------------------------- |
| Crear dashboard                | CloudWatch > Dashboards  | Azure Monitor > Workbooks        |
| Agregar métrica                | Desde widget o CLI       | KQL query                        |
| Visualizar logs en gráfico     | CloudWatch Logs Insights | KQL en Workbook                  |
| Filtro por severidad o recurso | Dimensiones en métricas  | KQL `where` + parámetro dinámico |
| Exportar/compartir             | IAM Policy o consola     | Compartir workbook con RBAC      |

---

## 📚 Recursos oficiales

* **AWS**

  * [CloudWatch Dashboards](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html)
  * [Creating Widgets](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/add_graph_widget.html)

* **Azure**

  * [Azure Monitor Workbooks](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-overview)
  * [Workbooks Examples](https://learn.microsoft.com/en-us/azure/azure-monitor/visualize/workbooks-gallery)