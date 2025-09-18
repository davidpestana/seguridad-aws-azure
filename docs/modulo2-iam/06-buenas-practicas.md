# 📄 `06-buenas-practicas.md`

## 🧭 Introducción – Buenas prácticas en la gestión de identidades y accesos

La mayoría de los incidentes de seguridad en la nube tienen una causa común: **errores humanos en la configuración de identidades y permisos**.

Adoptar buenas prácticas desde el inicio permite:

* Reducir drásticamente el riesgo de accesos no autorizados.
* Prevenir filtraciones de claves o privilegios excesivos.
* Cumplir requisitos de auditoría y normativas (ISO 27001, NIST, CIS…).

Este archivo recoge un conjunto de prácticas esenciales para configurar un entorno **seguro, mantenible y auditable** en AWS y Azure.

---

## 🔐 1. MFA obligatorio en todas las cuentas críticas

**Multi-Factor Authentication (MFA)** debe ser activado para:

* Todos los usuarios con privilegios elevados.
* Cuentas root o administradores globales.
* Accesos externos o federados.

| Plataforma | Configuración                                                              |
| ---------- | -------------------------------------------------------------------------- |
| **AWS**    | Activar MFA para usuarios IAM y cuenta root. Usar políticas que lo exijan. |
| **Azure**  | Configurar políticas de acceso condicional que requieran MFA.              |

✔️ En Azure, se puede activar MFA automáticamente con “Security Defaults”.

---

## 👥 2. Uso de grupos para asignación de permisos

No asignes permisos directamente a usuarios individuales. Usa **grupos**:

* Grupos por función (`devops`, `lectores-s3`, `auditores`).
* Grupos por entorno (`prod-admins`, `test-readers`).
* Asignaciones de roles o políticas a nivel de grupo.

✔️ Esto facilita la gestión, escalabilidad y trazabilidad.

---

## 🔐 3. Aplicar el principio de **mínimo privilegio**

Cada identidad debe tener **solo los permisos necesarios** para su tarea, y nada más.

* Evita políticas amplias como `*:*` en AWS.
* En Azure, evita usar el rol `Owner` por defecto.
* Revisa regularmente las asignaciones con:

  * **IAM Access Analyzer** (AWS)
  * **PIM** – Privileged Identity Management (Azure)

---

## 🔑 4. Prohibir claves de acceso estáticas

Las claves IAM o tokens secretos nunca deben:

* Usarse en producción.
* Estar en código fuente o archivos `.env`.
* Subirse a repositorios (¡ni privados!).

✔️ Usa **roles temporales**, federación de identidad o acceso mediante credenciales efímeras.

| Alternativas seguras                  |
| ------------------------------------- |
| AWS STS + IAM Roles                   |
| Azure Managed Identities              |
| Secrets Manager / Key Vault para apps |

---

## 🧪 5. Rotación de claves y contraseñas

* **Rotar claves IAM** cada 90 días (o eliminarlas si no se usan).
* **Desactivar usuarios inactivos** tras un periodo (30/60/90 días).
* En Azure, aplicar **políticas de expiración de contraseñas**.

✔️ Automatiza auditorías para detectar credenciales obsoletas.

---

## 🔎 6. Supervisar actividades sospechosas

Activa auditoría y monitorización desde el día 1:

* **AWS**:

  * CloudTrail (auditoría API)
  * GuardDuty (detección de amenazas)
  * IAM Access Analyzer (revisión de políticas)

* **Azure**:

  * Azure AD Sign-In Logs
  * Defender for Cloud
  * Conditional Access + Alertas

✔️ Configura alertas ante actividades anómalas:

* Inicio de sesión desde IP desconocida
* Acceso a recursos sensibles fuera de horario
* Cambios de políticas o roles

---

## 🧱 7. Revisiones y gobernanza periódicas

Al menos una vez al mes:

* Revisa usuarios, roles y grupos activos.
* Elimina identidades no usadas.
* Documenta cambios relevantes en IAM o Azure AD.
* Usa herramientas de evaluación como:

  * **AWS Security Hub**
  * **Azure Secure Score**

✔️ Esto mejora la postura de seguridad y prepara para auditorías.

---

## 🏢 Caso empresarial – Auditoría y refuerzo de seguridad IAM

La empresa **MediCloud**, que opera en AWS y Azure, se somete a una auditoría de seguridad.

✅ Medidas tomadas:

* Elimina 27 claves IAM no usadas hace más de 90 días.
* Obliga MFA en todos los administradores de ambas plataformas.
* Reduce los usuarios con permisos `AdministratorAccess` en AWS de 9 a 3.
* Crea grupos separados por entorno y función.
* Usa Access Analyzer y PIM para revisar asignaciones.

🔐 Resultado: aprobación sin observaciones, y recomendaciones formales para otras unidades de negocio.

---

## 💻 Configuración – Snippets útiles

### ✅ AWS – Política que requiere MFA para todas las acciones

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

### ✅ Azure – Activar Security Defaults para forzar MFA

```bash
az rest --method PATCH \
  --uri "https://graph.microsoft.com/v1.0/policies/identitySecurityDefaultsEnforcementPolicy" \
  --body '{"isEnabled": true}'
```

---

## ❌ Errores comunes

| Error                                                | Consecuencia                                 | Solución                                |
| ---------------------------------------------------- | -------------------------------------------- | --------------------------------------- |
| Usar cuenta root o Global Admin para tareas normales | Alto riesgo de compromiso                    | Usar cuentas de administración delegada |
| No revisar roles heredados o anidados                | Permisos excesivos sin saberlo               | Revisar jerarquías y scopes asignados   |
| No eliminar usuarios inactivos                       | Ataques oportunistas por cuentas abandonadas | Auditorías automáticas de actividad     |

---

## ✅ Checklist general de buenas prácticas IAM

* [ ] MFA habilitado en cuentas sensibles.
* [ ] Grupos usados para asignación de permisos.
* [ ] Prohibidas claves estáticas en producción.
* [ ] Revisiones mensuales de roles, políticas y accesos.
* [ ] Uso de herramientas nativas (PIM, Analyzer, Secure Score).
* [ ] Auditoría y logging activo desde el inicio.

---

## 📚 Recursos recomendados

| Recurso                                                                                                                                | Descripción                                      |
| -------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)                                         | Guía oficial de buenas prácticas                 |
| [Azure AD Security Best Practices](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/security-best-practices)      | Lista de prácticas recomendadas por Microsoft    |
| [AWS Access Analyzer](https://docs.aws.amazon.com/IAM/latest/UserGuide/access-analyzer.html)                                           | Análisis de accesos externos a recursos AWS      |
| [Azure Privileged Identity Management (PIM)](https://learn.microsoft.com/en-us/azure/active-directory/privileged-identity-management/) | Gestión de permisos con acceso temporal en Azure |
