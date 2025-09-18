# ✅ Compliance en Azure

---

## 🗂️ Introducción

Microsoft Azure proporciona un conjunto de herramientas integradas que ayudan a las organizaciones a **verificar, mantener y demostrar el cumplimiento** de normativas como **GDPR**, **ISO 27001**, **PCI DSS**, **HIPAA**, y más. Estas herramientas permiten implementar una estrategia de **compliance como código**, donde las reglas y auditorías están automatizadas, centralizadas y trazables.

---

## 🧰 Servicios clave para cumplimiento en Azure

### 1. **Microsoft Trust Center**

> Portal oficial con documentación de cumplimiento de Microsoft.

* Acceso a reportes de cumplimiento de normativas como:

  * ISO 27001, ISO 27701
  * PCI DSS
  * HIPAA
  * SOC 1, SOC 2, SOC 3
* Detalla cómo Azure cumple y mantiene esas certificaciones.

📎 [https://www.microsoft.com/en-us/trust-center](https://www.microsoft.com/en-us/trust-center)

---

### 2. **Azure Policy**

> Motor de cumplimiento y control preventivo.

* Permite definir reglas que **auditan, niegan o modifican** recursos según condiciones específicas.
* Aplicables a suscripciones, grupos de recursos o recursos individuales.
* Soporta **iniciativas**: agrupaciones de políticas para normativas como **CIS**, **NIST**, **GDPR**.

✅ **Ejemplo de política** (en modo "Deny"):

```json
{
  "properties": {
    "displayName": "Denegar VMs sin discos cifrados",
    "policyRule": {
      "if": {
        "not": {
          "field": "Microsoft.Compute/virtualMachines/osDisk.encryptionSettings.enabled",
          "equals": true
        }
      },
      "then": {
        "effect": "deny"
      }
    }
  }
}
```

---

### 3. **Azure Blueprints**

> Plantillas de gobernanza para entornos completos.

* Combina políticas, control de acceso (RBAC), y recursos estándar.
* Ideal para organizaciones que despliegan entornos repetibles (por ejemplo, con requisitos ISO o PCI).
* Puede incluir:

  * Configuración de red
  * Políticas obligatorias
  * Grupos de recursos predefinidos
  * Roles asignados

📎 [Documentación oficial sobre Blueprints](https://learn.microsoft.com/en-us/azure/governance/blueprints/overview)

---

### 4. **Compliance Manager**

> Panel de cumplimiento normativo con puntuación y recomendaciones.

* Evalúa el estado de cumplimiento frente a más de 300 regulaciones globales.
* Ofrece:

  * Recomendaciones técnicas
  * Acciones pendientes
  * Evidencia necesaria para auditores

✅ Muy útil en entornos con auditorías recurrentes (como banca o salud).

📎 [https://compliance.microsoft.com](https://compliance.microsoft.com)

---

## 🧑‍💼 Caso empresarial

**Organización**: Clínica privada con sede en Europa.
**Normativa aplicable**: GDPR + ISO 27001 + HIPAA
**Requisito**: Demostrar que su infraestructura en Azure cumple estas normativas.

### Solución basada en herramientas Azure:

| Necesidad                     | Herramienta Azure Usada           |
| ----------------------------- | --------------------------------- |
| Acceso a documentación        | Trust Center                      |
| Evitar VMs sin cifrado        | Azure Policy (Deny)               |
| Plantilla de entornos seguros | Azure Blueprint (con RBAC+Policy) |
| Auditoría y plan de acción    | Compliance Manager                |

---

## 🧪 Tips para probar en consola y CLI

### ▶️ En el portal:

1. Accede a **Azure Policy** > Definir nueva política.
2. Usa una política existente (ej: `Audit VMs without encrypted disks`).
3. Asóciala a una suscripción y revisa el efecto (“audit” o “deny”).
4. Accede a **Compliance Manager** > Selecciona un marco (ej: GDPR).

### ▶️ En CLI:

```bash
# Listar definiciones de políticas
az policy definition list --query "[?contains(displayName, 'Encryption')]" -o table

# Crear asignación de política
az policy assignment create \
  --name "Enforce-VM-Encryption" \
  --policy "<policyDefinitionId>" \
  --scope "/subscriptions/<subscriptionId>"
```

---

## ❌ Errores comunes

| Error                                              | Consecuencia                                                    |
| -------------------------------------------------- | --------------------------------------------------------------- |
| Crear políticas sin aplicarlas (sin asignación)    | No se evalúan los recursos; no hay control real                 |
| Usar políticas solo en modo “audit”                | Se detectan errores, pero no se bloquean configuraciones        |
| Ignorar Blueprints y hacer despliegues manuales    | Configuraciones inconsistentes y alto riesgo de drift           |
| No actualizar Compliance Manager con evidencias    | Informes de cumplimiento incompletos                            |
| Confundir cumplimiento del proveedor con el propio | Microsoft cumple con su parte, pero tú debes configurar la tuya |

---

## ✅ Buenas prácticas

* Combinar **Azure Policy + Initiative** para cada marco regulatorio (GDPR, ISO, etc.).
* Crear **Blueprints reutilizables** para onboarding de nuevos proyectos.
* Establecer revisiones periódicas en **Compliance Manager**.
* Incluir reglas de tagging, localización de datos, cifrado, etc.
* Asegurar el control de acceso (RBAC) y aplicar privilegios mínimos.
* Automatizar la asignación de políticas con **Azure DevOps/Terraform/Bicep**.

---

## 🔗 Recursos recomendados

* [Azure Policy Docs](https://learn.microsoft.com/en-us/azure/governance/policy/overview)
* [Azure Blueprints Overview](https://learn.microsoft.com/en-us/azure/governance/blueprints/overview)
* [Microsoft Trust Center](https://www.microsoft.com/en-us/trust-center)
* [Compliance Manager](https://compliance.microsoft.com)
* [Azure Policy Samples (GitHub)](https://github.com/Azure/azure-policy)