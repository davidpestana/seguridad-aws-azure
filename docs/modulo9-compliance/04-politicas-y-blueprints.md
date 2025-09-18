# 🎯 Políticas y Blueprints

---

## 🗂️ Introducción

En entornos cloud, asegurar cumplimiento y coherencia en la configuración de recursos requiere más que buenas intenciones. Tanto **AWS** como **Azure** permiten implementar **políticas automatizadas y auditables** para prevenir configuraciones erróneas y aplicar estándares organizativos.

Las **políticas** permiten imponer reglas claras ("los buckets deben estar cifrados"), mientras que los **blueprints** agrupan múltiples políticas, controles de acceso y plantillas de despliegue.

---

## 📌 Comparativa general

| Característica        | AWS                                     | Azure                                          |
| --------------------- | --------------------------------------- | ---------------------------------------------- |
| Motor de políticas    | AWS Config Rules                        | Azure Policy                                   |
| Plantillas gobernadas | AWS Control Tower (guardrails)          | Azure Blueprints                               |
| Efectos de las reglas | `Compliance`, `Remediation`, `Alerting` | `Deny`, `Audit`, `Append`, `DeployIfNotExists` |
| Agrupación de reglas  | Custom Conformance Packs                | Policy Initiatives                             |

---

## 🛠️ Azure Policy y Blueprints

### ✅ Azure Policy

* Permite crear y aplicar reglas para:

  * Evitar despliegue de recursos no conformes.
  * Auditar el estado de los recursos existentes.
  * Forzar configuraciones (DeployIfNotExists).

Ejemplo: denegar creación de recursos en regiones fuera de Europa.

```json
{
  "if": {
    "not": {
      "field": "location",
      "in": ["northeurope", "westeurope"]
    }
  },
  "then": {
    "effect": "deny"
  }
}
```

---

### 🧩 Azure Blueprints

* Agrupan:

  * **Policies** (como la anterior),
  * **Roles (RBAC)** para equipos,
  * **Recursos estándar** (nombres, tagging, NSGs, etc.).
* Usados para desplegar entornos con gobernanza preconfigurada.

Ejemplo: Blueprint de “entorno regulado GDPR” que incluye:

* Policy para cifrado de discos.
* RBAC para equipo de auditoría.
* Tags obligatorios en cada recurso.

---

## 🛠️ AWS Config Rules y Control Tower

### ✅ AWS Config Rules

* Permiten definir reglas que validan configuraciones y generan alertas.
* Soportan **remediación automática**.

Ejemplo: requerir cifrado en buckets S3:

```json
{
  "ConfigRuleName": "s3-bucket-server-side-encryption-enabled",
  "Source": {
    "Owner": "AWS",
    "SourceIdentifier": "S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"
  }
}
```

---

### 🏗️ AWS Control Tower + Guardrails

* Framework para gestión multi-cuenta con gobernanza.
* Usa **Guardrails**: políticas predefinidas como “no permitir recursos fuera de la región”.
* Incluye **landing zone** inicial con:

  * Cuentas segregadas (auditoría, facturación, seguridad).
  * Logging centralizado.
  * SCPs (Service Control Policies) sobre cuentas hijas.

---

## 🧪 Tips para probar

### ▶️ Azure CLI

```bash
# Crear una iniciativa de políticas (Policy Initiative)
az policy set-definition create \
  --name gdpr-compliance \
  --display-name "GDPR Compliance Pack" \
  --definitions '[{"policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/..." }]'
```

### ▶️ AWS CLI

```bash
# Activar regla de AWS Config
aws configservice put-config-rule \
  --config-rule file://s3_encryption_rule.json
```

---

## 🧑‍💼 Caso de uso realista

**Empresa**: Consultora legal con clientes bancarios.
**Objetivo**: Cumplir con PCI DSS y evitar configuraciones inseguras.

### Implementación:

| Requisito                             | Azure                     | AWS                             |
| ------------------------------------- | ------------------------- | ------------------------------- |
| Cifrado obligatorio en almacenamiento | Policy + Blueprint        | Config Rule + Remediación       |
| Regiones autorizadas                  | Policy (location check)   | SCP (Service Control Policy)    |
| Logging obligatorio                   | Blueprint (Log Analytics) | Control Tower + Central logging |
| Auditoría de cumplimiento             | Compliance Manager        | Security Hub + Config Reports   |

---

## ❌ Errores frecuentes

| Error                                                       | Impacto                                        |
| ----------------------------------------------------------- | ---------------------------------------------- |
| Aplicar políticas en modo "audit" y no revisarlas nunca     | Configuraciones inseguras pasan sin corrección |
| No vincular blueprints a suscripciones nuevas               | Entornos sin control ni trazabilidad           |
| No mantener versiones de políticas                          | Cambios sin trazabilidad                       |
| Implementar políticas sin probarlas en entornos de staging  | Bloqueos inesperados o fallos masivos          |
| Pensar que los "guardrails" reemplazan el hardening interno | Es complementario, no sustitutivo              |

---

## ✅ Buenas prácticas

* Crear políticas primero en **modo audit**, luego en `deny` tras validación.
* Usar **nombres consistentes** en políticas, roles y recursos.
* Versionar tus políticas/blueprints con Git.
* Etiquetar recursos críticos para seguimiento en informes de cumplimiento.
* Documentar excepciones justificadas (ex: región especial, cuenta legacy).
* Revisar políticas tras actualizaciones regulatorias (ej: NIS2, DORA).

---

## 🔗 Recursos recomendados

* [Azure Policy samples (GitHub)](https://github.com/Azure/azure-policy)
* [Documentación Azure Blueprints](https://learn.microsoft.com/en-us/azure/governance/blueprints/)
* [AWS Config Rule Reference](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
* [AWS Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/)