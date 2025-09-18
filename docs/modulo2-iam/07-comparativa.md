# 📄 `07-comparativa.md`

📁 Ruta sugerida: `modulo2-iam/07-comparativa.md`

---

## 🧭 Introducción – IAM (AWS) vs Azure AD + RBAC (Azure)

Aunque tanto **AWS** como **Azure** ofrecen soluciones avanzadas para la gestión de identidades y accesos, **su arquitectura y modelos de control de permisos son diferentes**.
Comprender estas diferencias es crucial para:

* Diseñar estrategias multi-cloud coherentes.
* Evitar errores al migrar o gestionar entornos híbridos.
* Traducir mentalmente conceptos entre plataformas sin perder seguridad.

Este archivo ofrece una comparativa directa entre **IAM en AWS** y **Azure Active Directory (Azure AD) + RBAC**, destacando similitudes, diferencias y casos de equivalencia funcional.

---

## 🔎 Comparativa general de conceptos clave

| Concepto funcional           | AWS – IAM                              | Azure – Azure AD + RBAC                      |
| ---------------------------- | -------------------------------------- | -------------------------------------------- |
| Contenedor raíz de identidad | Cuenta AWS                             | Tenant de Azure AD                           |
| Gestión de usuarios          | Usuarios IAM                           | Usuarios en Azure AD                         |
| Autenticación multifactor    | MFA (IAM o root)                       | Acceso condicional / Security Defaults       |
| Grupos                       | Grupos IAM (poco usados)               | Grupos de seguridad / Microsoft 365          |
| Control de acceso            | Políticas JSON aplicadas a identidades | Roles RBAC asignados por scope               |
| Roles para servicios         | IAM Roles                              | Managed Identities / Principales de servicio |
| Identidades externas         | Federadas (SAML, OIDC)                 | Invitados, federación, Entra ID              |
| Evaluación de permisos       | IAM Policy Simulator                   | Azure "Modo de prueba" + PIM                 |

✔️ En AWS se trabaja principalmente **por política explícita JSON**, mientras que en Azure **se asignan roles RBAC a scopes jerárquicos**.

---

## 🔐 Control de permisos: JSON vs RBAC

### 🔸 AWS IAM

* Basado en **políticas JSON**.
* Alto grado de granularidad:

  * Permite restringir por IP, por etiquetas, por hora, por condición...
* Permisos aplicados directamente a usuarios, grupos o roles.

🧩 Ejemplo:

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::bucket-seguro/*"
}
```

---

### 🔹 Azure RBAC

* Basado en **roles predefinidos o personalizados**.
* Aplicados a **scopes jerárquicos** (suscripción → grupo de recursos → recurso).
* Menos flexible que IAM JSON, pero más fácil de gestionar visualmente.

🧩 Ejemplo:

```bash
az role assignment create \
  --assignee usuario@empresa.com \
  --role Reader \
  --scope /subscriptions/<id>
```

---

## 🔁 Equivalencias funcionales

| Función                 | AWS                           | Azure                                    |
| ----------------------- | ----------------------------- | ---------------------------------------- |
| Usuario humano          | IAM User                      | Usuario Azure AD                         |
| Grupo de usuarios       | IAM Group                     | Grupo de seguridad                       |
| Rol para apps           | IAM Role + Assume Role Policy | App Registration + Principal de servicio |
| Permisos temporales     | STS – Temporary Credentials   | PIM – Just-In-Time Access                |
| Permisos personalizados | JSON Policy                   | Custom RBAC Role                         |
| Revisión de permisos    | IAM Access Analyzer           | Azure PIM / Access Reviews               |
| Autenticación MFA       | IAM MFA / Policy enforcement  | Azure MFA / Conditional Access           |
| Federación de identidad | SAML/OIDC → AssumeRole        | Azure B2B / External ID / SAML           |

---

## 🧱 Jerarquía y scopes

### En AWS:

* No hay scopes jerárquicos como en Azure.
* El control se realiza **por recurso individual** (ej. por ARN).
* Si quieres segmentar entornos, usas **múltiples cuentas** (con AWS Organizations).

### En Azure:

* Jerarquía clara:

  1. Tenant (identidad)
  2. Suscripción
  3. Grupo de recursos
  4. Recurso
* Los roles asignados se **heredan hacia abajo**.

---

## 🏢 Caso empresarial – Migración funcional entre plataformas

La empresa **TechBridge** quiere replicar una política de acceso de AWS en Azure:

* En AWS:

  * IAM Role con política que permite a ciertos usuarios leer solo un bucket S3.

* En Azure:

  * Crea un **grupo de seguridad** `storage-readers`.
  * Crea un **rol RBAC personalizado** con acceso solo a blobs.
  * Asigna el rol a nivel de recurso (solo ese Storage Account).

✔️ Mismo objetivo alcanzado, pero con enfoques distintos:

* En AWS: recurso explícito por ARN.
* En Azure: scope jerárquico + rol + grupo.

---

## ❌ Errores comunes al comparar plataformas

| Error                                                      | Explicación                                                         | Solución                                                        |
| ---------------------------------------------------------- | ------------------------------------------------------------------- | --------------------------------------------------------------- |
| Asumir que Azure AD y Azure RBAC son lo mismo              | Azure AD gestiona identidades; RBAC gestiona acceso a recursos      | Estudiar los dos modelos por separado                           |
| Buscar scopes jerárquicos en AWS                           | AWS no tiene scopes → se compensa con múltiples cuentas y etiquetas | Usar AWS Organizations y políticas SCP                          |
| Asumir que roles RBAC de Azure son tan granulares como IAM | RBAC es más simple, no tiene condiciones complejas                  | Usar políticas de Azure Policy para restricciones más avanzadas |

---

## ✅ Buenas prácticas comparadas

| Práctica recomendada   | AWS                                        | Azure                                          |
| ---------------------- | ------------------------------------------ | ---------------------------------------------- |
| MFA obligatorio        | IAM Policies + MFA root                    | Conditional Access + Security Defaults         |
| Mínimo privilegio      | Políticas específicas por acción + recurso | Asignar solo roles necesarios + scopes mínimos |
| Revisión periódica     | IAM Access Analyzer / CloudTrail           | PIM / Secure Score / Sign-In Logs              |
| Separación de entornos | Múltiples cuentas + SCP                    | Múltiples suscripciones o grupos de recursos   |

---

## 📚 Recursos recomendados

| Recurso                                                                                                               | Descripción                                |
| --------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| [AWS IAM vs Azure AD – Microsoft Docs](https://learn.microsoft.com/en-us/security/compass/identity-aws-vs-azure)      | Comparativa oficial Microsoft              |
| [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/)                                            | Documentación oficial IAM                  |
| [Azure RBAC Overview](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)                     | Guía de control de acceso en Azure         |
| [AWS Organizations & SCPs](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html) | Segmentación y control multi-cuenta en AWS |

---

## 🧠 Conclusión

IAM y Azure AD abordan los mismos objetivos (gestión de identidades y control de accesos), pero lo hacen con **paradigmas distintos**:

* AWS apuesta por **políticas JSON altamente configurables**, sin jerarquía de scopes.
* Azure se basa en **roles RBAC aplicados jerárquicamente**, con integración profunda en todo el ecosistema Microsoft.

Conocer ambas aproximaciones permite a los profesionales de seguridad adaptarse a entornos multi-cloud con eficacia y evitar errores de configuración al traducir conceptos entre plataformas.