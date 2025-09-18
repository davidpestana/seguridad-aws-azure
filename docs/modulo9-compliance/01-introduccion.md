# ✅ Buenas prácticas en compliance y gobernanza (AWS y Azure)

---

## 🗂️ Introducción

Una arquitectura cloud segura no solo debe resistir ataques, sino también **cumplir normativas**, ser **auditable** y garantizar que los recursos desplegados respetan las políticas de la organización. En este apartado recopilamos las **buenas prácticas clave** para aplicar compliance y gobernanza de forma proactiva y automatizada en AWS y Azure.

---

## 🧩 Principios rectores

| Principio                 | Aplicación en cloud                            |
| ------------------------- | ---------------------------------------------- |
| **Compliance as Code**    | Reglas escritas y versionadas (Config, Policy) |
| **Gobernanza preventiva** | No permitir recursos que no cumplan            |
| **Automatización**        | Remediación automática de desviaciones         |
| **Auditoría continua**    | Logs, alertas y revisiones regulares           |
| **Multi-nube coherente**  | Políticas comunes aunque cambie el proveedor   |

---

## 🔐 Buenas prácticas clave

### 1. Compliance como código (`Compliance as Code`)

> Automatiza la detección y prevención de configuraciones incorrectas.

| AWS                            | Azure                                         |
| ------------------------------ | --------------------------------------------- |
| AWS Config Rules + Remediation | Azure Policy (efectos: `deny`, `audit`, etc.) |
| Versionado en Git              | Versionado en Azure Repos/GitHub              |
| Validación en pipelines (IaC)  | Azure DevOps + Policy Check                   |

✅ **Ejemplo**:

* Política de Azure que **impide crear recursos sin etiquetas**.
* Config rule de AWS que **detecta S3 sin cifrado SSE-KMS** y lo remedia automáticamente.

---

### 2. Gobernanza de múltiples cuentas

> Evita “drift” entre equipos usando gestión centralizada.

| AWS                                  | Azure                                    |
| ------------------------------------ | ---------------------------------------- |
| AWS Organizations + SCP              | Azure Management Groups + Policy         |
| Control Tower para despliegue seguro | Blueprints para entornos predefinidos    |
| CloudTrail centralizado              | Log Analytics + Activity Logs unificados |

✅ **Recomendaciones**:

* Usar **Organizational Units (OUs)** para agrupar cuentas por entorno (dev/prod).
* Definir **políticas de acceso, regiones permitidas y servicios autorizados**.
* Centralizar logs en una única cuenta/suscripción con acceso restringido.

---

### 3. Etiquetado obligatorio (`Tagging`)

> El etiquetado permite saber **quién es responsable**, **cuál es el propósito** y **qué coste implica** cada recurso.

| Clave             | Ejemplo               |
| ----------------- | --------------------- |
| `Owner`           | `equipo-devops`       |
| `Environment`     | `production` / `test` |
| `CostCenter`      | `marketing-001`       |
| `DataSensitivity` | `high` / `personal`   |

✅ **Buenas prácticas**:

* Aplicar **políticas que impidan crear recursos sin tags**.
* Establecer una **taxonomía común de etiquetas** en la organización.
* Auditar recursos sin etiquetas con scripts o reglas de compliance.

---

### 4. Automatización de remediaciones

> Alertar no es suficiente. Lo ideal es **corregir automáticamente**.

| Tipo de desviación        | Remediación automatizada                     |
| ------------------------- | -------------------------------------------- |
| Bucket sin cifrado (AWS)  | Añadir SSE-KMS                               |
| Storage sin HTTPS (Azure) | Denegar creación con Policy                  |
| VM sin backup             | Añadir al Vault de backup (Azure Policy)     |
| Tag faltante              | Etiquetado automático vía Lambda o Logic App |

✅ **Recomendación**:

* Empieza en **modo audit**, pero establece plazos para pasar a **modo `deny`**.
* Usa **funciones sin servidor (Lambda / Logic Apps)** para aplicar correcciones.

---

### 5. Auditoría continua y trazabilidad

> No basta con estar seguro, hay que poder **demostrarlo**.

| Elemento                        | AWS                                     | Azure                                   |
| ------------------------------- | --------------------------------------- | --------------------------------------- |
| Registro de actividad           | CloudTrail                              | Azure Activity Logs                     |
| Registro de cambios de recursos | AWS Config                              | Azure Resource Graph / Change History   |
| Reportes de cumplimiento        | Security Hub / Config Conformance Packs | Defender for Cloud / Compliance Manager |

✅ **Checklist mínimo**:

* Logs centralizados y con retención mínima de 1 año.
* Alertas por cambios críticos (ej. eliminación de roles, apertura de puertos).
* Dashboards de cumplimiento por entorno y equipo.

---

### 6. Sector y normativa específica

| Normativa     | Requisitos típicos                               | Herramientas recomendadas                           |
| ------------- | ------------------------------------------------ | --------------------------------------------------- |
| **GDPR**      | Localización, consentimiento, borrado de datos   | Azure Purview, Amazon Macie, tagging sensible       |
| **PCI DSS**   | Cifrado, control de accesos, auditoría           | KMS + Config + Security Hub                         |
| **HIPAA**     | Protección de datos médicos, registros de acceso | Azure Policy, Azure Monitor, AWS Config             |
| **ISO 27001** | Gestión global de la seguridad                   | Compliance Manager (Azure), Conformance Packs (AWS) |

---

## ❌ Errores comunes

| Error                                                    | Consecuencia típica                                  |
| -------------------------------------------------------- | ---------------------------------------------------- |
| Dejar políticas en modo `audit` sin pasar a `deny`       | Recursos incumpliendo sin bloqueo                    |
| Usar configuraciones diferentes entre cuentas o regiones | Inconsistencias, difícil de mantener                 |
| Falta de tags clave (`Owner`, `CostCenter`)              | Dificultad para asignar costes o responder auditoría |
| No remediar, solo alertar                                | Desviaciones se acumulan y se normalizan             |
| Confundir compliance del proveedor con el del cliente    | Creer que AWS/Azure ya “cumplen por ti”              |

---

## 🧪 Checklist práctico de cumplimiento

| Elemento                                                  | ✔ / ❌ |
| --------------------------------------------------------- | ----- |
| Todas las cuentas están en una OU o Management Group      |       |
| Existen políticas activas que impiden mal uso             |       |
| Se remedian desviaciones automáticamente cuando es viable |       |
| Todos los recursos tienen tags obligatorios               |       |
| Hay informes periódicos de cumplimiento                   |       |
| Los logs están centralizados y con retención suficiente   |       |
| Se revisa el cumplimiento antes de cada despliegue        |       |

---

## 🔗 Recursos recomendados

* [AWS Config Best Practices](https://docs.aws.amazon.com/config/latest/developerguide/best-practices.html)
* [Azure Policy Samples](https://github.com/Azure/azure-policy)
* [AWS Artifact: Certificaciones y auditorías](https://aws.amazon.com/artifact/)
* [Microsoft Compliance Manager](https://learn.microsoft.com/en-us/microsoft-365/compliance/compliance-manager)
* [AWS Control Tower Overview](https://docs.aws.amazon.com/controltower/latest/userguide/what-is-control-tower.html)
* [Azure Blueprints](https://learn.microsoft.com/en-us/azure/governance/blueprints/overview)