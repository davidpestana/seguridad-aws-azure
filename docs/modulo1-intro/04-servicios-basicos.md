# 📄 `04-servicios-basicos.md`


## 🧭 Introducción – Servicios básicos de seguridad en AWS y Azure

Tanto **AWS como Azure ofrecen un conjunto de servicios nativos** diseñados específicamente para facilitar la implementación de arquitecturas seguras desde el inicio. Estos servicios cubren áreas como:

* Gestión de identidades y accesos
* Auditoría y trazabilidad
* Protección activa contra amenazas
* Cumplimiento normativo y evaluación continua

Conocer y dominar estos servicios es esencial para construir soluciones cloud que no solo funcionen bien, sino que además estén **alineadas con las mejores prácticas de seguridad**.

---

## 🔎 Explicación detallada – Categorías clave de servicios de seguridad

### 🔐 1. Gestión de identidades y accesos (IAM)

Este es el punto de partida de toda estrategia de seguridad cloud.

| Característica                  | AWS (IAM)                                        | Azure (Azure AD + RBAC)                        |
| ------------------------------- | ------------------------------------------------ | ---------------------------------------------- |
| Autenticación                   | Usuarios IAM, federación con SAML/OIDC           | Azure AD, Microsoft Entra, SSO empresarial     |
| Autorización basada en roles    | Policies (JSON), roles, grupos                   | Role-Based Access Control (RBAC)               |
| MFA (autenticación multifactor) | Activable por usuario y obligatorio por política | Activable por condiciones (Conditional Access) |
| Accesos temporales              | Roles asumibles (STS)                            | Roles asignados por tiempo limitado            |

🔍 **Importancia**: si se gestiona mal, un usuario IAM o un App Registration puede tener acceso completo a toda la cuenta.

---

### 📜 2. Auditoría y trazabilidad

Permite **saber quién hizo qué, cuándo y desde dónde**, lo que resulta esencial para detectar comportamientos anómalos, responder a incidentes y cumplir normativas.

| Funcionalidad                | AWS                    | Azure                                 |
| ---------------------------- | ---------------------- | ------------------------------------- |
| Auditoría de acciones API    | AWS CloudTrail         | Azure Activity Logs                   |
| Logs de recursos y servicios | Amazon CloudWatch Logs | Azure Monitor + Log Analytics         |
| Seguimiento de cambios       | AWS Config             | Azure Resource Graph / Change History |
| Visualización de eventos     | CloudWatch Dashboards  | Azure Workbooks                       |

🔍 **Consejo**: habilita logs en cuentas nuevas y establece destinos de almacenamiento seguros desde el primer momento.

---

### 🛡️ 3. Protección activa contra amenazas

Estos servicios **analizan comportamientos sospechosos**, detectan patrones de ataque y pueden generar alertas o bloquear eventos automáticamente.

| Funcionalidad                | AWS                                    | Azure                                        |
| ---------------------------- | -------------------------------------- | -------------------------------------------- |
| Detección de amenazas        | Amazon GuardDuty                       | Microsoft Defender for Cloud                 |
| Protección contra DDoS       | AWS Shield                             | Azure DDoS Protection                        |
| Protección de red            | AWS Network Firewall / Security Groups | Azure Firewall / NSG                         |
| Análisis de vulnerabilidades | Amazon Inspector                       | Defender para VM, contenedores, App Services |

🔍 **Importancia**: habilita estas protecciones especialmente en entornos expuestos (públicos, críticos, APIs).

---

### ⚖️ 4. Cumplimiento normativo y evaluación de configuración

Permiten **evaluar tu entorno contra estándares de seguridad**, alertar sobre desviaciones y automatizar correcciones.

| Funcionalidad              | AWS                                 | Azure                                    |
| -------------------------- | ----------------------------------- | ---------------------------------------- |
| Evaluación de seguridad    | AWS Security Hub + AWS Config Rules | Defender for Cloud (Compliance Insights) |
| Aplicación de políticas    | AWS Organizations SCP + Config      | Azure Policy + Blueprints                |
| Integración con estándares | CIS, PCI, HIPAA, ISO 27001, NIST    | CIS, ISO, NIST, FedRAMP                  |
| Remediación automática     | Lambda + Config / SSM Automation    | Azure Policy Remediation Tasks           |

🔍 **Recomendación**: activa revisiones automáticas y monitoriza alertas de cumplimiento periódicamente.

---

## 🏢 Caso empresarial – Hardening de una startup SaaS

La empresa **EduTechNow** lanza su producto SaaS en AWS. En su primera auditoría, detectan varias deficiencias:

* Usuarios IAM sin MFA
* Buckets S3 públicos
* Ausencia de logs centralizados
* Sin análisis de amenazas

Acciones correctivas aplicadas:

1. Activan **GuardDuty** para detección de amenazas.
2. Centralizan logs con **CloudWatch Logs** + S3.
3. Habilitan políticas de cifrado y bloqueo de acceso público en S3.
4. Imponen uso obligatorio de MFA en IAM.

Resultado: la empresa supera el benchmark CIS y obtiene una mejora del 67% en el score de Security Hub.

---

## 💻 Código y configuración

### ✅ AWS – Activar GuardDuty y centralizar logs

```bash
# Activar GuardDuty
aws guardduty create-detector --enable

# Activar CloudTrail y enviar logs a S3
aws cloudtrail create-trail \
  --name MiTrail \
  --s3-bucket-name bucket-logs-miempresa \
  --is-multi-region-trail

aws cloudtrail start-logging --name MiTrail
```

### ✅ Azure – Crear una política de auditoría y asociarla

```bash
# Asignar política de auditoría básica en un grupo de recursos
az policy assignment create \
  --name "politica-auditoria" \
  --scope "/subscriptions/<id>/resourceGroups/<grupo>" \
  --policy "/providers/Microsoft.Authorization/policyDefinitions/audit-vm-insecure-passwords"
```

---

## 🧪 Tips para probar

### En consola web

* **AWS**:

  * Ve a Security Hub → Activa el estándar “AWS Foundational Best Practices”.
  * Revisa alertas de GuardDuty con detalles sobre IPs sospechosas.
  * Abre CloudTrail → Visualiza eventos de login y cambios de configuración.

* **Azure**:

  * Defender for Cloud → Revisa el score de seguridad y los controles no cumplidos.
  * Azure Monitor → Accede a Workbooks o crea reglas de alerta.
  * Azure Policy → Aplica una definición y observa las evaluaciones.

### En CLI

* Ver estado de GuardDuty:

  ```bash
  aws guardduty list-detectors
  ```

* Ver recomendaciones de Defender:

  ```bash
  az security task list --output table
  ```

---

## ❌ Errores comunes

| Error                                | Consecuencia                          | Solución recomendada                                       |
| ------------------------------------ | ------------------------------------- | ---------------------------------------------------------- |
| No habilitar logs de auditoría       | Falta de trazabilidad ante incidentes | Activar CloudTrail o Activity Logs desde el inicio         |
| Ignorar recomendaciones de seguridad | Superficie de ataque más amplia       | Automatizar revisión periódica con Security Hub o Defender |
| Accesos sin control centralizado     | Riesgo de escaladas de privilegio     | Implementar SSO y RBAC desde Azure AD o IAM                |

---

## ✅ Buenas prácticas

✔️ Checklist práctico:

* [ ] Activar auditoría en todas las regiones y cuentas.
* [ ] Forzar MFA y mínimo privilegio en identidades.
* [ ] Habilitar protección activa (Defender, GuardDuty).
* [ ] Aplicar políticas que prevengan errores comunes.
* [ ] Monitorizar periódicamente el score de seguridad.

---

## 📚 Recursos recomendados

| Recurso                                                                                               | Descripción                                           |
| ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| [AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/what-is-securityhub.html) | Agrega hallazgos de seguridad y compliance            |
| [Azure Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/)               | Plataforma centralizada de seguridad en Azure         |
| [AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/)                               | Revisión y auditoría de configuración de recursos AWS |
| [Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/overview)                    | Motor de cumplimiento y control de recursos           |
| [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)        | Recomendaciones oficiales de seguridad de identidades |
