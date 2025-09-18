# ✅ Buenas prácticas en compliance y gobernanza (AWS y Azure)

---

## 🎯 Introducción

El cumplimiento normativo en la nube no se alcanza simplemente con configurar bien los servicios: requiere una **estrategia de gobierno**, **auditoría continua** y **automatización del cumplimiento**. En este archivo consolidamos las **buenas prácticas clave** que organizaciones deben adoptar para mantener la seguridad, la legalidad y el control operativo en entornos cloud multi-servicio y multi-cuenta.

---

## 🛠️ Compliance as Code

> **Definición:** Representar las políticas y controles de cumplimiento como código versionado.

### Ventajas:

* Auditabilidad: puedes ver cuándo y quién cambió una política.
* Automatización: despliegue con herramientas como Terraform o Bicep.
* Reutilización: definir plantillas aplicables a múltiples entornos.

### En AWS:

* **AWS Config Rules** definidas en JSON.
* **SCPs** gestionadas desde Organizations como política centralizada.
* Integración con **Control Tower** para despliegues seguros por defecto.

### En Azure:

* **Azure Policy** en JSON: efectos `deny`, `audit`, `append`, `deployIfNotExists`.
* **Blueprints** que combinan políticas, RBAC y recursos.
* Repositorios Git con políticas versionadas + integración con pipelines.

---

## 🔁 Remediación automática

> Detectar incumplimientos está bien, **corregirlos automáticamente es mejor**.

### AWS:

* **AWS Config** puede disparar remediaciones automáticas vía SSM Automation Documents.
* **Security Hub** puede integrarse con EventBridge para actuar ante findings.

### Azure:

* **Azure Policy con efecto `deployIfNotExists`** permite que se despliegue el recurso correcto si no existe.
* **Azure Automation + Logic Apps** para respuestas personalizadas ante fallos de cumplimiento.

📌 Ejemplo: si se detecta un bucket/Storage sin cifrado, se puede aplicar automáticamente una política que active la encriptación.

---

## 🏷️ Etiquetado obligatorio (Tagging)

> Sin etiquetas, no se sabe quién es responsable de un recurso, qué coste tiene ni qué datos contiene.

### Recomendaciones:

* Definir un esquema de tags obligatorio: `owner`, `env`, `data-classification`, `cost-center`.
* Aplicar políticas que impidan crear recursos sin etiquetas.
* Usar las etiquetas para filtrar en auditorías, costes y alertas.

### Herramientas:

* **AWS**: Tag Policies + AWS Config.
* **Azure**: Azure Policy para requerir tags o autoaplicarlos.

📌 Error típico: recursos sin etiquetas en cuentas compartidas → difícil de rastrear en incidentes o facturación.

---

## 📜 Auditoría continua y centralización de logs

> Toda acción relevante debe dejar rastro y ese rastro debe estar centralizado, protegido y retenido.

### AWS:

* **CloudTrail** habilitado en todas las cuentas, con logs centralizados en una cuenta de seguridad.
* **S3 Server Access Logs**, **VPC Flow Logs**, **Config**, **GuardDuty** → centralizados con Kinesis o SIEM.

### Azure:

* **Azure Monitor + Log Analytics Workspace** centralizado.
* Diagnósticos habilitados en Storage, Key Vault, App Service, etc.
* Integración con **Microsoft Sentinel** para correlación de eventos.

### Retención mínima recomendada:

* 1 año para logs de seguridad, 3–7 años para cumplimiento normativo.

---

## 🔐 Acceso mínimo y separación de entornos

> El cumplimiento también implica limitar el riesgo humano.

* Usar **principio de mínimo privilegio**: nadie tiene acceso por defecto.
* Separar entornos (dev, staging, prod) física o lógicamente (OU / Management Groups).
* Revisión periódica de accesos y roles.

### AWS:

* Revisión de permisos con IAM Access Analyzer.
* SCPs para bloquear acciones no autorizadas (ej. crear usuarios, desactivar CloudTrail).

### Azure:

* RBAC basado en grupos de Azure AD.
* Políticas para impedir despliegues fuera de entornos controlados.

---

## 🧪 Pruebas, documentación y control de excepciones

> Compliance ≠ todo perfecto. Pero lo no conforme debe estar documentado.

* Documentar excepciones: por qué se hace, con qué mitigación, cuándo se revisa.
* Mantener un **catálogo de controles** y su estado por entorno.
* Automatizar reporting para auditores (compliance dashboards).

### Herramientas útiles:

* **AWS Security Hub**, **AWS Audit Manager**.
* **Azure Compliance Manager**, **Workbook Reports**.
* Integración con **Power BI**, **Grafana**, etc.

---

## 🚫 Errores comunes a evitar

| Error                                          | Riesgo asociado                        |
| ---------------------------------------------- | -------------------------------------- |
| Políticas en modo `audit` que nunca se revisan | Falsa sensación de seguridad           |
| Regiones abiertas sin control                  | Pérdida de trazabilidad y cumplimiento |
| Uso de cuentas personales/admins globales      | Exposición innecesaria                 |
| Logging mal configurado o sin retención        | Falta de evidencia ante auditoría      |
| Excepciones sin control ni vencimiento         | Puertas traseras indefinidas           |

---

## ✅ Checklist práctico final

| Ítem                                                              | Estado |
| ----------------------------------------------------------------- | ------ |
| \[ ] Todos los recursos tienen etiquetas requeridas               |        |
| \[ ] Logging activado y centralizado en todas las cuentas         |        |
| \[ ] Cifrado obligatorio en datos en reposo y en tránsito         |        |
| \[ ] Políticas de compliance como código                          |        |
| \[ ] Roles de acceso revisados cada 90 días                       |        |
| \[ ] Regiones bloqueadas salvo las necesarias                     |        |
| \[ ] Reporting automatizado para auditoría                        |        |
| \[ ] Excepciones documentadas y con vencimiento                   |        |
| \[ ] Control Tower / Blueprints aplicados en entornos productivos |        |
| \[ ] Config / Policy con remediación automática habilitada        |        |

---

## 📚 Recursos recomendados

* [Compliance as Code – AWS](https://aws.amazon.com/blogs/security/how-to-implement-compliance-as-code/)
* [Azure Policy samples](https://github.com/Azure/azure-policy)
* [AWS Config managed rules](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
* [Microsoft Cloud Adoption Framework – Governance](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/govern/)
* [CIS Controls v8](https://www.cisecurity.org/controls/cis-controls-list/)