# 🧠 Azure Sentinel: Detección y Respuesta Inteligente con SIEM/SOAR

---

## 🧭 ¿Qué es Azure Sentinel?

**Azure Sentinel** es una solución **SIEM/SOAR nativa en la nube**, diseñada para:

* **Recopilar** datos de seguridad desde cualquier fuente (cloud, on-prem, SaaS).
* **Detectar** amenazas con inteligencia artificial y reglas personalizadas.
* **Responder** a incidentes automáticamente mediante Playbooks.
* **Investigar** correlando eventos a través del tiempo.

Sentinel forma parte de **Microsoft Defender XDR** y se integra con todos los servicios de Azure, así como con entornos **multi-cloud** como AWS, GCP y dispositivos on-premise.

---

## 🧩 SIEM vs SOAR: ¿en qué se diferencian?

| Concepto | Significado                                     | Función principal                                          |
| -------- | ----------------------------------------------- | ---------------------------------------------------------- |
| **SIEM** | Security Information and Event Management       | Recopila, normaliza y correlaciona eventos de seguridad.   |
| **SOAR** | Security Orchestration, Automation and Response | Ejecuta respuestas automáticas ante incidentes detectados. |

Sentinel **integra ambos enfoques**, permitiendo pasar **de la detección a la acción sin intervención humana**.

---

## 🔗 Arquitectura de Sentinel

```plaintext
[ Fuentes de datos ]
     ↓
[ Log Analytics Workspace ] ← almacenamiento
     ↓
[ Azure Sentinel ]
     ├─ Analítica y detección (KQL queries)
     ├─ Investigaciones guiadas
     └─ Respuesta automatizada (Playbooks)
```

---

## 🧪 Ingesta de datos: ¿qué puedes conectar?

| Origen     | Tipo          | Ejemplo                           |
| ---------- | ------------- | --------------------------------- |
| Azure      | Nativo        | Azure AD, Defender, Activity Logs |
| AWS        | Cloud         | CloudTrail, GuardDuty, VPC Logs   |
| On-premise | Syslog, CEF   | Firewalls, IDS, antivirus         |
| SaaS       | API o agentes | Office 365, Okta, Salesforce      |

```bash
# Habilitar conector de AWS (desde portal o CLI)
az sentinel data-connector create \
  --workspace-name mi-workspace \
  --rule-name AWS-Connector \
  --resource-group seguridad \
  --connector-id AmazonWebServicesCloudTrail
```

---

## 🧠 Detección con consultas KQL

Sentinel utiliza el lenguaje **KQL (Kusto Query Language)** para escribir reglas de detección y explorar grandes volúmenes de logs.

### 🔍 Ejemplo: detectar intentos fallidos masivos

```kql
SigninLogs
| where ResultType == 50074
| summarize Count = count() by UserPrincipalName, bin(TimeGenerated, 5m)
| where Count > 10
```

### 🔍 Otro ejemplo: geolocalización inesperada

```kql
SigninLogs
| where Location != "Spain"
| summarize count() by IPAddress, UserPrincipalName
```

---

## ⚙️ Automatización con Playbooks (SOAR)

Un **Playbook** en Sentinel es una automatización construida con **Logic Apps** que se dispara cuando se genera una alerta.

### 🔧 Ejemplos de acciones en un playbook:

* Bloquear usuario en Azure AD.
* Notificar a un canal de Teams.
* Crear ticket en ServiceNow.
* Aislar una máquina virtual.
* Enviar correo al SOC.

### 🎯 Ejemplo empresarial

> Un usuario intenta iniciar sesión 20 veces fallidamente desde múltiples IPs.

**Acciones automáticas:**

1. Playbook bloquea al usuario en Azure AD.
2. Se notifica al SOC por Teams y email.
3. Se registra el incidente en Azure DevOps.

---

## 📐 Gestión de incidentes

Sentinel incluye un **panel de incidentes** desde el cual los analistas pueden:

* Ver el **historial de alertas correladas**.
* Usar la función “Investigación” para ver relaciones visuales entre eventos.
* Lanzar consultas adicionales en KQL.
* Ejecutar playbooks manual o automáticamente.

---

## 🧪 Tips para usar Sentinel

| Acción                            | Descripción                                  |
| --------------------------------- | -------------------------------------------- |
| Crear una alerta programada       | Basada en una consulta KQL                   |
| Ejecutar playbook desde incidente | Se vincula desde la interfaz de incidente    |
| Importar datos de AWS             | Usar conector CloudTrail o GuardDuty         |
| Probar integración SOAR           | Crear un playbook que mande mensajes a Teams |

---

## ⚠️ Errores comunes

* ❌ Cargar datos sin configurar retención ni almacenamiento adecuado.
* ❌ Crear demasiadas reglas KQL sin priorización → exceso de alertas.
* ❌ No documentar qué hace cada Playbook → difícil de mantener.
* ❌ Pensar que “ya está todo cubierto” solo por conectar fuentes.

---

## ✅ Buenas prácticas

✔ Empezar con pocos **conectores clave** y expandir.
✔ Mantener **nombres y descripciones claras** para todas las reglas y playbooks.
✔ Revisar mensualmente las **consultas KQL** y afinar umbrales.
✔ Establecer una **nomenclatura estándar** para los incidentes.
✔ Medir efectividad: cuántos incidentes se resolvieron automáticamente vs manualmente.

---

## 📚 Recursos recomendados

* [Azure Sentinel – documentación oficial](https://learn.microsoft.com/en-us/azure/sentinel/)
* [Lista de conectores de datos](https://learn.microsoft.com/en-us/azure/sentinel/connectors)
* [KQL Query Playground](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
* [Tutorial: Crear un playbook en Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/tutorial-respond-threats-playbook)