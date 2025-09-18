# ✅ Compliance en AWS

---

## 🗂️ Introducción

AWS proporciona un ecosistema maduro y robusto de herramientas que facilitan la **gestión del cumplimiento normativo** en entornos cloud. Desde repositorios de documentación legal hasta herramientas que verifican automáticamente configuraciones, es posible auditar, corregir y demostrar cumplimiento frente a regulaciones como **GDPR**, **ISO 27001**, **HIPAA** o **PCI DSS**.

---

## 🔍 Servicios clave para compliance

### 1. **AWS Artifact**

> Tu portal oficial de compliance y auditoría.

* Proporciona acceso a **informes de auditoría**, **certificados**, y **documentación legal**.
* Permite a las empresas descargar documentos oficiales sobre cumplimiento de AWS con:

  * ISO 27001, ISO 27701, ISO 9001
  * PCI DSS
  * SOC 1, SOC 2, SOC 3
  * HIPAA, FedRAMP, etc.

📌 Acceso desde consola: **AWS Artifact → View Reports**
📎 [https://artifact.aws.amazon.com](https://artifact.aws.amazon.com)

---

### 2. **AWS Config**

> Para auditar configuraciones y automatizar cumplimiento.

* Revisa continuamente el estado de los recursos (por ejemplo: "¿están los buckets cifrados?").
* Soporta **reglas predefinidas** y **personalizadas (Lambda)**.
* Permite acciones de **remediación automática** si se detecta desviación.

✅ **Ejemplo real**:

```json
{
  "ConfigRuleName": "s3-bucket-server-side-encryption-enabled",
  "SourceIdentifier": "S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED",
  "Scope": {
    "ComplianceResourceTypes": ["AWS::S3::Bucket"]
  }
}
```

📌 Puedes usar **Conformance Packs** para agrupar reglas relacionadas con marcos como PCI DSS o CIS Benchmarks.

---

### 3. **AWS Security Hub**

> Consolida hallazgos de múltiples servicios y frameworks.

* Agrega datos desde **GuardDuty**, **Config**, **Inspector**, etc.
* Evalúa cumplimiento con **CIS AWS Foundations Benchmark** y otros estándares.
* Crea puntuaciones de cumplimiento y listas de hallazgos priorizados.

✅ **Uso típico**:

* Ver qué instancias EC2 no tienen logs activados.
* Detectar buckets sin cifrado.
* Analizar tendencias de cumplimiento en múltiples regiones/cuentas.

📌 Integración con **AWS Organizations** para visión centralizada.

---

### 4. **AWS Organizations + Control Tower**

> Gobierno multi-cuenta con políticas centralizadas.

* Agrupa cuentas en **Organizational Units (OU)**.
* Aplica **SCP (Service Control Policies)** para restringir uso de servicios o regiones.
* Con **Control Tower**, puedes desplegar nuevas cuentas con:

  * Cifrado forzado
  * Logging habilitado
  * Tags obligatorios

✅ Ejemplo de **SCP** que impide usar servicios fuera de `eu-west-1`:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "*",
    "Resource": "*",
    "Condition": {
      "StringNotEquals": {
        "aws:RequestedRegion": "eu-west-1"
      }
    }
  }]
}
```

---

## 🧑‍💼 Caso empresarial

**Empresa**: Startup Fintech con presencia en Europa y EE.UU.
**Regulación aplicable**: GDPR + PCI DSS
**Desafío**: Necesitan demostrar que sus servicios en AWS cumplen con requisitos regulatorios sin construir un sistema desde cero.

**Solución con AWS**:

| Necesidad                         | Servicio AWS Usado            |
| --------------------------------- | ----------------------------- |
| Acceso a documentación PCI        | AWS Artifact                  |
| Cifrado obligatorio en S3         | AWS Config Rule + Remediación |
| Supervisión de seguridad continua | AWS Security Hub + GuardDuty  |
| Restricción de regiones           | AWS Organizations + SCP       |
| Gobernanza y cuentas seguras      | AWS Control Tower             |

---

## 🧪 Tips para probar en consola y CLI

### ▶️ Probar desde consola:

1. Ve a **AWS Config** → Activa reglas para recursos como S3, EC2, RDS.
2. Crea una regla predefinida: `s3-bucket-server-side-encryption-enabled`.
3. Lanza un bucket sin cifrado → verifica el hallazgo de no cumplimiento.
4. Aplica una **acción de remediación automática**.

### ▶️ Probar desde CLI:

```bash
aws configservice describe-config-rules
aws configservice get-compliance-summary-by-config-rule
```

---

## ❌ Errores comunes

| Error                                                  | Consecuencia                                            |
| ------------------------------------------------------ | ------------------------------------------------------- |
| Activar Config sin remediación automática              | Se detectan errores, pero no se corrigen                |
| No revisar hallazgos de Security Hub                   | Fallos de seguridad/compliance ignorados                |
| No usar SCP en Organizations                           | Cuentas hijas pueden crear recursos peligrosos          |
| Usar cuentas sueltas sin Control Tower                 | Imposibilidad de aplicar gobernanza consistente         |
| Creer que Artifact asegura el cumplimiento del cliente | AWS cumple con su parte; tú debes asegurar tus recursos |

---

## ✅ Buenas prácticas

* Usar **Conformance Packs** para marcos como PCI, NIST, CIS.
* Activar **Config** en todas las regiones, incluso si no se usan.
* Definir **acciones automáticas** cuando se detecte incumplimiento.
* Supervisar el dashboard de **Security Hub** al menos semanalmente.
* Usar **Control Tower** para onboarding automatizado de nuevas cuentas.

---

## 🔗 Recursos recomendados

* [AWS Artifact](https://aws.amazon.com/artifact/)
* [AWS Config Rules Repository](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
* [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html)
* [AWS Control Tower](https://docs.aws.amazon.com/controltower/)
* [Service Control Policies (SCP)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)