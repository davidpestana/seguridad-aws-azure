## 📑 Módulo 9: Compliance y Gobernanza (AWS y Azure)

---

## 🎯 Objetivos del módulo

La seguridad no se limita a la parte técnica: también hay que cumplir con **normativas legales y estándares de la industria**. Este módulo cubre cómo implementar políticas de compliance y gobernanza en AWS y Azure para asegurar que las configuraciones cumplen regulaciones (ISO, GDPR, PCI DSS, HIPAA, etc.) y que se pueden auditar fácilmente.

Al finalizar serás capaz de:

* Entender el rol de compliance y gobernanza en entornos cloud.
* Acceder a documentación oficial de cumplimiento (AWS Artifact, Microsoft Trust Center).
* Configurar políticas de control y compliance automático (AWS Config, Azure Policy).
* Gestionar entornos multi-cuenta (AWS Organizations, Azure Management Groups).
* Implementar **Blueprints** y **Control Tower** para gobernanza a gran escala.
* Adoptar buenas prácticas de auditoría, reporting y cumplimiento normativo.

---

## 🧭 Índice detallado del módulo

---

### 1) Introducción a compliance y gobernanza

✅ Objetivo
Comprender la importancia de compliance y gobernanza en cloud como marco que une seguridad, legalidad y auditoría.

📌 Claves
Compliance vs Gobernanza · Regulaciones · Riesgo reputacional · Auditoría continua

🧩 Contenido

* **Compliance** = cumplir con normativas externas (GDPR, PCI DSS, HIPAA, ISO 27001).
* **Gobernanza** = asegurar que la organización aplica políticas internas coherentes (segregación, tagging, mínimos de seguridad).
* La nube acelera despliegues → riesgo de incumplimiento por configuraciones rápidas y sin control.
* Ejemplo: desplegar un bucket S3 público en una empresa de salud = incumplimiento HIPAA inmediato.

📚 docs/modulo9-compliance/01-introduccion.md

---

### 2) Compliance en AWS

✅ Objetivo
Usar servicios y herramientas nativas de AWS para cumplir normativas.

📌 Claves
AWS Artifact · AWS Config · Security Hub · Organizations

🧩 Contenido

* **AWS Artifact**: repositorio de documentación de compliance (ISO, PCI DSS, informes de auditoría de AWS).
* **AWS Config**: define reglas de configuración (ej. todos los buckets deben estar cifrados). Permite remediación automática.
* **AWS Security Hub**: consolida hallazgos de seguridad y compliance (CIS Benchmarks, PCI DSS).
* **AWS Organizations/Control Tower**: aplicar políticas de seguridad en múltiples cuentas.

Ejemplo: usar AWS Config para detectar buckets sin cifrado y remediarlos automáticamente.

📚 docs/modulo9-compliance/02-compliance-en-aws.md

---

### 3) Compliance en Azure

✅ Objetivo
Conocer las herramientas nativas de Azure para cumplimiento y auditoría.

📌 Claves
Azure Policy · Blueprints · Compliance Manager · Trust Center

🧩 Contenido

* **Azure Trust Center**: documentación oficial de certificaciones y normativas.
* **Azure Policy**: crear reglas que impiden configuraciones no conformes (ej. denegar VMs sin cifrado de discos).
* **Azure Blueprints**: plantillas que combinan políticas, RBAC y recursos preconfigurados.
* **Compliance Manager**: panel con el estado de cumplimiento frente a normativas.

Ejemplo: denegar el despliegue de recursos en regiones no autorizadas mediante Azure Policy.

📚 docs/modulo9-compliance/03-compliance-en-azure.md

---

### 4) Políticas y Blueprints

✅ Objetivo
Automatizar cumplimiento y gobernanza en el despliegue de recursos.

📌 Claves
Policies (deny/append/audit) · Blueprints como plantillas · Automación

🧩 Contenido

* **AWS Config rules** vs **Azure Policy**: ambas permiten “auditar” y “denegar”.
* **Blueprints en Azure**: definen paquetes completos (ej. red, NSG, políticas de tagging).
* **Control Tower en AWS**: despliega cuentas seguras por defecto con guardrails.

Ejemplo comparativo:

* AWS Config Rule → “los buckets deben tener versioning habilitado”.
* Azure Policy → “storage accounts deben tener secure transfer enabled”.

📚 docs/modulo9-compliance/04-politicas-y-blueprints.md

---

### 5) AWS Organizations y Control Tower

✅ Objetivo
Gestionar múltiples cuentas AWS con políticas centralizadas.

📌 Claves
OU (Organizational Units) · SCP (Service Control Policies) · Guardrails

🧩 Contenido

* **AWS Organizations**: agrupar cuentas en **OUs** y aplicar **SCPs** (ej. impedir que nadie cree recursos fuera de la región autorizada).
* **Control Tower**: configuración guiada de múltiples cuentas con seguridad y compliance habilitados por defecto.
* **Centralización de logging y billing**.

Ejemplo: usar SCP para prohibir la creación de IAM users en lugar de roles en toda la organización.

📚 docs/modulo9-compliance/05-organizations-y-control-tower.md

---

### 6) Casos de uso de compliance

✅ Objetivo
Analizar cómo se aplican compliance y gobernanza en distintos sectores.

📌 Claves
PCI DSS · HIPAA · GDPR · ISO 27001 · NIS2

🧩 Contenido

* **PCI DSS** (banca, tarjetas): cifrado obligatorio, auditoría de accesos, logs detallados.
* **HIPAA** (sanidad): protección de datos sensibles, registros de acceso.
* **GDPR** (Europa): privacidad por diseño, derecho al olvido, localización de datos.
* **ISO 27001**: marco de gestión de seguridad global.
* **NIS2**: requisitos de ciberseguridad en infraestructuras críticas.

Ejemplo: una fintech debe aplicar PCI DSS → en AWS usar Config rules + KMS; en Azure habilitar cifrado y monitorización obligatoria.

📚 docs/modulo9-compliance/06-casos-de-uso.md

---

### 7) Buenas prácticas en compliance y gobernanza

✅ Objetivo
Establecer prácticas comunes que permitan cumplir normativas y mantener control en despliegues multi-cloud.

📌 Claves
“Compliance as Code” · Auditoría continua · Etiquetado obligatorio · Remediación automática

🧩 Contenido

* Usar **compliance as code** (Config/Policy) → reglas auditables en repositorios.
* Automatizar **remediaciones** en lugar de solo alertar.
* **Tagging obligatorio** para identificar propietarios de recursos.
* Revisiones periódicas de cumplimiento en SOC.
* Mantener logs centralizados y con retención adecuada (años, no días).
* Evitar “shadow IT”: políticas que bloqueen recursos no aprobados.

Errores comunes:

* Políticas en “audit only” nunca revisadas.
* Falta de control en entornos de prueba (mayor riesgo de fuga).
* Confundir documentación de compliance del proveedor con cumplimiento del cliente.

📚 docs/modulo9-compliance/07-buenas-practicas.md