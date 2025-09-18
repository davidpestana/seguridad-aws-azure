# 🧪 Casos de Uso de Compliance en AWS y Azure

---

## 🎯 Introducción

Las regulaciones y estándares como **GDPR**, **HIPAA**, **PCI DSS**, **ISO 27001** o **NIS2** no son opcionales en muchos sectores. Este archivo analiza cómo se aplican estos marcos en entornos cloud (AWS y Azure), con ejemplos prácticos por industria.

La clave no es solo tener “servicios seguros”, sino demostrar que los controles están:

1. **Definidos**
2. **Aplicados**
3. **Auditados**

---

## 🏥 HIPAA – Sanidad

**Requisitos clave**:

* Protección de datos personales de salud (PHI).
* Registro de accesos.
* Cifrado en tránsito y en reposo.
* Retención de logs y eventos.

**AWS**:

* **S3 + SSE-KMS** para almacenar PHI cifrado.
* **CloudTrail** + **Config** para auditar accesos y configuración.
* **IAM Roles** para evitar credenciales estáticas.
* Acuerdo de BAA firmado con AWS.

**Azure**:

* **Blob Storage** + Key Vault con CMK para cifrado.
* **Azure Monitor + Log Analytics** para trazabilidad.
* Activar **Azure Policy** para exigir cifrado y regiones concretas.
* Acuerdo BAA disponible para clientes regulados.

📌 Ejemplo:

> Aplicación médica que almacena historiales clínicos → debe usar cifrado CMK, logging activado, acceso solo desde VPC/vNet privadas y política de retención mínima de 7 años.

---

## 💳 PCI DSS – Sector financiero (tarjetas)

**Requisitos clave**:

* Datos cifrados en todo momento.
* Segmentación de red.
* Gestión de identidades.
* Logs detallados e inmutables.

**AWS**:

* **IAM + MFA + SCPs** para limitar acceso.
* **S3 Object Lock** (modo WORM) para logs de transacciones.
* **Security Hub** con controles PCI DSS activados.
* **AWS WAF + Shield** en capas frontales.

**Azure**:

* **NSG + Azure Firewall** para segmentación.
* **Azure Policy** para prevenir configuraciones no conformes.
* **Defender for Cloud** con baseline PCI DSS.
* **Immutable blob** + Soft delete en almacenamiento.

📌 Ejemplo:

> Un PSP (proveedor de pagos) usa Azure → se exige cifrado CMK, Azure Policy para denegar acceso no cifrado, y centralización de logs en Log Analytics con 365 días de retención.

---

## 👥 GDPR – Protección de datos personales (UE)

**Requisitos clave**:

* Consentimiento explícito.
* Derecho al olvido.
* Acceso restringido.
* Geolocalización y residencia de datos.
* Registro de accesos y borrados.

**AWS**:

* Usar **regiones europeas** y bloquear cross-region.
* **Tags** para identificar datos personales.
* **Lambda/DataBrew** para anonimización.
* Logs de acceso y políticas de borrado programado.

**Azure**:

* **Compliance Manager** con informes GDPR.
* **Data Loss Prevention** (Purview) para detectar datos personales.
* Uso de **RBAC** para limitar accesos.
* Reglas de borrado y versioning en almacenamiento.

📌 Ejemplo:

> Startup europea con app B2C debe demostrar localización de datos en EU, tener logs de acceso a PHI, mecanismos de borrado (automático o manual) y protección con Azure RBAC.

---

## 🔐 ISO 27001 – Seguridad de la información

**Requisitos clave**:

* Políticas documentadas.
* Acceso mínimo necesario.
* Evaluación de riesgos.
* Control de cambios.

**AWS**:

* Uso de **AWS Config** para compliance continuo.
* **GuardDuty + CloudTrail** para detección y auditoría.
* Automatización con **AWS Systems Manager**.
* **Organizations** y OUs para separar entornos.

**Azure**:

* **Blueprints** con ISO 27001 preconfigurado.
* Activación de **Security Center** con recomendaciones.
* Uso de **Just-In-Time Access** (JIT) para minimizar exposición.
* Automatización con Azure Automation / Defender for Cloud.

📌 Ejemplo:

> Consultora que prepara su certificación ISO27001 debe demostrar políticas de control de acceso, rotación de claves, logs protegidos, y control de cambios en infraestructura.

---

## 🌐 NIS2 – Infraestructuras críticas (UE)

**Requisitos clave**:

* Alta disponibilidad y continuidad.
* Monitorización de incidentes.
* Notificación obligatoria de brechas.
* Evaluación continua de seguridad.

**AWS**:

* Replicación multi-región con S3 CRR.
* **CloudWatch + SNS + EventBridge** para alertas.
* Automatización con Runbooks y SSM.
* AWS Shield Advanced para protección DDoS.

**Azure**:

* Geo-replicación (RA-GRS) y Azure Backup.
* **Azure Sentinel** para correlación de eventos e incidentes.
* **Microsoft Defender for Cloud** + regulaciones NIS2.
* Políticas de gobierno centralizado con Management Groups.

📌 Ejemplo:

> Una empresa eléctrica de España, sujeta a NIS2, debe demostrar recuperación ante desastres en menos de X horas, logs en regiones UE, y alertas automatizadas ante eventos anómalos.

---

## 🧩 Comparativa rápida

| Regulación | AWS principal                 | Azure principal                     |
| ---------- | ----------------------------- | ----------------------------------- |
| HIPAA      | S3 + KMS + Config             | Blob + Key Vault + Policy           |
| PCI DSS    | Security Hub + SCP + WORM     | Defender + NSG + Immutable Blob     |
| GDPR       | Region lock + Tags + IAM      | Compliance Manager + RBAC + Purview |
| ISO 27001  | Config + GuardDuty + SSM      | Blueprints + JIT + Defender         |
| NIS2       | S3 CRR + EventBridge + Shield | Sentinel + RA-GRS + Policy          |

---

## ✅ Buenas prácticas comunes

* Definir **políticas por industria** (no genéricas).
* Automatizar controles y remediaciones.
* Auditar continuamente con herramientas nativas.
* Usar entornos separados (desarrollo/test/producción).
* Documentar excepciones justificadas a políticas.

---

## 🔗 Recursos recomendados

* [AWS Compliance Programs](https://aws.amazon.com/compliance/programs/)
* [Azure Compliance Offerings](https://learn.microsoft.com/en-us/compliance/regulatory/)
* [Microsoft Compliance Manager](https://learn.microsoft.com/en-us/microsoft-365/compliance/compliance-manager)
* [AWS Artifact](https://aws.amazon.com/artifact/)