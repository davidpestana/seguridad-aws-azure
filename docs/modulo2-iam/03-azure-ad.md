# 📄 `03-azure-ad.md`

## 🧭 Introducción – ¿Qué es Azure Active Directory (Azure AD)?

**Azure Active Directory (Azure AD)** es el servicio de **identidad y control de acceso basado en la nube** de Microsoft.
Es el núcleo de la seguridad en Azure, y se integra también con **Microsoft 365**, **SaaS externas**, y otros entornos cloud.

> ❗️Nota: desde julio de 2023, Microsoft ha comenzado a renombrar **Azure AD** como parte de **Microsoft Entra ID**, pero en este curso mantendremos el término "Azure AD" por claridad.

Azure AD gestiona **usuarios, grupos, autenticación, roles, SSO y federación**.
Además, permite aplicar controles avanzados como **acceso condicional, MFA, protección de identidades** y más.

---

## 🔎 Explicación detallada – Arquitectura y componentes clave

### 🏢 1. Tenant

Un **tenant** es el contenedor raíz donde se crean y gestionan todas las identidades y recursos de Azure AD.

* Es único por organización.
* Tiene su propio dominio (`empresa.onmicrosoft.com`).
* No debe confundirse con suscripciones de Azure (una organización puede tener varias).

✔️ Todo usuario, grupo o app vive dentro de un tenant.

---

### 👤 2. Usuarios y grupos

* **Usuarios**: pueden ser internos (miembros del tenant) o externos (invitados).
  Ejemplo: `ana@empresa.com` o `proveedor1@gmail.com`.

* **Grupos**: sirven para agrupar usuarios y asignarles roles y permisos fácilmente.

  * **Grupos de seguridad**: para asignación de roles y control de acceso.
  * **Grupos de Microsoft 365**: para colaboración (correo, Teams, SharePoint...).

✔️ Buenas prácticas:

* Usar **grupos para asignar permisos**, no usuarios individuales.
* Separar grupos por función y entorno (`dev-rg-lec`, `prod-sql-admin`, etc.).

---

### 🔐 3. Roles RBAC (Role-Based Access Control)

Azure permite definir **quién puede hacer qué** en cada recurso mediante RBAC.

| Concepto              | Descripción                                                                           |
| --------------------- | ------------------------------------------------------------------------------------- |
| **Rol**               | Conjunto de permisos (ej. “Reader”, “Contributor”, “Owner”).                          |
| **Asignación de rol** | Vínculo entre un rol, una identidad y un scope.                                       |
| **Scope**             | Ámbito donde se aplica el rol (ej. suscripción, grupo de recursos, recurso concreto). |

✔️ Roles predefinidos:

* `Reader` (solo lectura)
* `Contributor` (lectura + escritura sin borrar)
* `Owner` (control total)

---

### 🧩 4. Aplicaciones y Enterprise Apps

Azure AD permite gestionar identidades de **aplicaciones** (no solo personas):

* **App registrations**: representan apps personalizadas que necesitan autenticarse.
* **Enterprise Applications**: representan apps SaaS o integraciones SSO (ej. Google Workspace, Salesforce, etc.).

🧠 Cada app puede tener:

* Su propia identidad (principal de servicio).
* Roles personalizados y permisos delegados.
* Políticas de acceso condicional.

✔️ Se puede controlar el acceso de usuarios a aplicaciones SaaS desde Azure AD.

---

## 🏢 Caso empresarial – Azure AD para gestionar un equipo de desarrollo

La empresa **CloudCoders** quiere otorgar a su equipo de desarrollo acceso de solo lectura a toda la infraestructura de Azure.

Pasos realizados:

1. Crea un **grupo de seguridad** llamado `dev-readers`.
2. Agrega a los usuarios `ana@cloudcoders.com` y `luis@cloudcoders.com` al grupo.
3. Asigna el **rol `Reader`** a ese grupo a nivel de **suscripción**.

Resultado:

* Los desarrolladores pueden revisar recursos sin modificarlos.
* El acceso se gestiona centralizadamente por grupo.
* Si un miembro cambia de rol, basta con quitarlo del grupo.

---

## 💻 Configuración – Ejemplos prácticos

### ✅ Crear un usuario en Azure AD

```bash
az ad user create \
  --display-name "Luis Fernández" \
  --user-principal-name luis@empresa.onmicrosoft.com \
  --password 'ClaveSegura123!' \
  --force-change-password-next-login true
```

---

### ✅ Crear grupo y asignar rol `Reader` a nivel de suscripción

```bash
# Crear grupo
az ad group create --display-name "dev-readers" --mail-nickname "dev-readers"

# Obtener ID del grupo
az ad group show --group "dev-readers" --query objectId -o tsv

# Asignar rol Reader a todo el grupo
az role assignment create \
  --assignee <ID_GRUPO> \
  --role "Reader" \
  --scope /subscriptions/<ID_SUSCRIPCION>
```

---

## 🧪 Tips para probar

### En portal web

* Ir a **Azure Active Directory → Usuarios** → Ver métodos de autenticación (MFA activado o no).
* Ir a **Azure AD → Grupos** → Ver miembros y roles asignados.
* Acceder a una suscripción y revisar en **Control de acceso (IAM)** las asignaciones activas.

### En CLI

* Listar miembros de un grupo:

```bash
az ad group member list --group "dev-readers" --output table
```

* Ver roles de un usuario:

```bash
az role assignment list \
  --assignee luis@empresa.onmicrosoft.com \
  --output table
```

---

## ❌ Errores comunes

| Error                                              | Consecuencia                               | Solución                                       |
| -------------------------------------------------- | ------------------------------------------ | ---------------------------------------------- |
| Asignar roles directamente a usuarios individuales | Dificulta la gestión a largo plazo         | Usar grupos para asignaciones                  |
| No activar MFA                                     | Aumenta el riesgo de secuestro de cuenta   | Política obligatoria de MFA para admins        |
| Confundir suscripción con tenant                   | Malas prácticas de gobierno y segmentación | Entender jerarquía tenant → subscripción → RGs |

---

## ✅ Buenas prácticas en Azure AD

✔️ Checklist inicial:

* [ ] MFA obligatorio para todos los administradores.
* [ ] Usuarios agrupados por función o equipo.
* [ ] Uso de grupos para asignar permisos, no usuarios individuales.
* [ ] Separar apps internas de apps externas.
* [ ] Asignar roles con el **menor scope posible**.

---

## 📚 Recursos recomendados

| Recurso                                                                                                            | Descripción                                     |
| ------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------- |
| [Azure AD Overview](https://learn.microsoft.com/en-us/entra/fundamentals/whatis)                                   | Introducción oficial a Azure AD                 |
| [Azure AD Roles vs Azure RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)         | Comparativa de sistemas de roles                |
| [Microsoft Learn – Azure Identity](https://learn.microsoft.com/en-us/training/paths/secure-azure-resources/)       | Ruta de aprendizaje gratuita                    |
| [Enterprise Apps SSO](https://learn.microsoft.com/en-us/azure/active-directory/manage-apps/what-is-single-sign-on) | Guía para configurar SSO con apps empresariales |

