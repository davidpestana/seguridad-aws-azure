# 🧾 Logs y Auditoría: Trazabilidad en AWS y Azure

## 🧭 Introducción

En la nube, **todo ocurre a través de llamadas API**: crear una máquina, eliminar un bucket, modificar una política IAM… todo deja un rastro, **siempre que esté habilitado el sistema de auditoría**.

Registrar esas acciones es lo que permite **reconstruir lo sucedido ante un incidente**, cumplir requisitos legales y aplicar buenas prácticas de gobernanza.

Este módulo se centra en los mecanismos de **auditoría de operaciones** en AWS y Azure: **CloudTrail**, **Activity Logs**, y sus variantes para el plano de datos.

---

## 🔍 Plano de control vs. plano de datos

| Plano             | ¿Qué registra?      | Ejemplo                           |
| ----------------- | ------------------- | --------------------------------- |
| **Control Plane** | Gestión de recursos | Crear una VM, borrar un bucket    |
| **Data Plane**    | Uso de recursos     | Leer un fichero, acceder a una BD |

✅ Ambos son **esenciales** para una auditoría completa.
⚠️ Por defecto, muchas plataformas **solo activan el plano de control** → se pierden trazas de accesos reales.

---

## 🧰 AWS: CloudTrail

### ¿Qué es?

**AWS CloudTrail** registra todas las llamadas API realizadas en una cuenta AWS, desde consola, SDK, CLI o servicios internos.

Incluye detalles como:

* Usuario o rol que hizo la llamada
* IP de origen
* Hora
* Parámetros usados
* Resultado (éxito o error)

### Características

| Función           | Descripción                                                      |
| ----------------- | ---------------------------------------------------------------- |
| **Trails**        | Configuraciones que envían eventos a S3, CloudWatch Logs u otros |
| **Event history** | Visualización rápida de los últimos 90 días                      |
| **Multi-región**  | Trails pueden ser globales o por región                          |
| **Data Events**   | Auditan acceso a objetos S3, funciones Lambda, etc.              |

### Activar CloudTrail (CLI)

```bash
aws cloudtrail create-trail \
  --name audit-global \
  --s3-bucket-name mi-bucket-logs \
  --is-multi-region-trail
```

### Activar Data Events para S3

```bash
aws cloudtrail put-event-selectors \
  --trail-name audit-global \
  --event-selectors '[{
    "ReadWriteType": "All",
    "IncludeManagementEvents": true,
    "DataResources": [{
      "Type": "AWS::S3::Object",
      "Values": ["arn:aws:s3:::mi-bucket/"]
    }]
  }]'
```

### Consultar eventos recientes

```bash
aws cloudtrail lookup-events --max-results 10
```

📌 O también desde **AWS Console > CloudTrail > Event history**.

---

## 🧰 Azure: Activity Logs + Diagnostic Settings

### Activity Logs

Registra eventos del **plano de control** en la suscripción:

* Creación/modificación de recursos
* Cambios en configuraciones
* Alertas administrativas

Se accede desde:

```bash
az monitor activity-log list \
  --max-events 5
```

### Diagnostic Settings

Para auditar el **plano de datos** (accesos, uso), cada recurso debe tener habilitado el envío de logs a Log Analytics o Storage.

Ejemplo: habilitar auditoría en Azure Storage

```bash
az monitor diagnostic-settings create \
  --name audit-storage \
  --resource "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<storage>" \
  --workspace "/subscriptions/<sub>/resourceGroups/<rg>/providers/Microsoft.OperationalInsights/workspaces/<workspace>" \
  --logs '[{"category":"StorageRead","enabled":true},{"category":"StorageWrite","enabled":true}]'
```

📌 **Es imprescindible habilitar esto** para detectar accesos reales a los datos.

---

## 🧪 Caso de uso: “¿Quién borró esta máquina virtual / bucket?”

### 🔎 En AWS

1. Acceder a **CloudTrail Event History**
2. Buscar por `DeleteBucket` o `TerminateInstances`
3. Filtrar por usuario, IP, hora
4. Correlacionar con logs de seguridad (CloudWatch Logs, GuardDuty)

### 🔎 En Azure

1. Ir a **Azure Monitor > Logs**
2. Ejecutar KQL en `AzureActivity`

```kql
AzureActivity
| where OperationNameValue contains "Delete"
| where ActivityStatusValue == "Succeeded"
| project TimeGenerated, Caller, ResourceGroup, Resource
```

---

## ⚠️ Riesgos de una auditoría incompleta

* Sin plano de datos: **no sabrás quién accedió a qué archivo o base de datos**
* Retención corta (7–30 días): **imposibilidad de investigar incidentes antiguos**
* Logs dispersos por recurso → **baja trazabilidad**
* Falta de correlación entre eventos → **sin contexto para análisis**

---

## ✅ Buenas prácticas

✔️ **Activar auditoría desde el inicio del proyecto**.
✔️ Habilitar **logs del plano de datos** para objetos sensibles.
✔️ Centralizar logs en una cuenta/subscripción de monitorización.
✔️ Exportar logs críticos a almacenamiento duradero (S3, Azure Storage).
✔️ Configurar **retención mínima de 1 año** para entornos regulados.
✔️ Asociar eventos críticos a **alertas automáticas o flujos de respuesta**.
✔️ Aplicar etiquetas a recursos para facilitar auditoría y filtrado.

---

## ❌ Errores comunes

* No activar CloudTrail o dejarlo en una sola región.
* No registrar acceso a objetos S3, blobs, bases de datos.
* Olvidar activar Diagnostic Settings → sin datos en Log Analytics.
* Limitar la retención a 30 días sin exportar.
* Confiar solo en Activity Logs (solo control plane).

---

## 📚 Recursos oficiales

* **AWS**

  * [CloudTrail overview](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)
  * [Data Events en CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/logging-data-events-with-cloudtrail.html)

* **Azure**

  * [Azure Activity Logs](https://learn.microsoft.com/en-us/azure/azure-monitor/platform/activity-log)
  * [Diagnostic Settings](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/diagnostic-settings)
  * [KQL reference for Azure Logs](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)