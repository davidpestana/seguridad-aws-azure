## 🔐 Módulo 2: Gestión de Identidades y Accesos (IAM y Azure AD)

---

## 🎯 Objetivos del módulo

Este módulo aborda el pilar más importante de la seguridad en la nube: **la identidad**. En un entorno sin perímetros físicos, las identidades, roles y permisos se convierten en la **primera línea de defensa**.

Al finalizar el módulo serás capaz de:

* Explicar por qué la identidad es el nuevo perímetro en cloud.
* Entender cómo funcionan IAM en AWS y Azure AD en Microsoft Azure.
* Diferenciar autenticación y autorización en entornos distribuidos.
* Crear usuarios, grupos, roles y políticas en AWS y Azure.
* Configurar MFA, SSO y accesos federados de forma segura.
* Comparar equivalencias de conceptos entre ambos proveedores.
* Aplicar buenas prácticas para reducir riesgos como credenciales expuestas o privilegios excesivos.

---

## 🧭 Índice detallado del módulo

---

### 1. Introducción a la gestión de identidades

* ✅ **Objetivo**: Comprender por qué la identidad se ha convertido en el nuevo perímetro de seguridad en cloud.
* 📌 **Claves**: Zero Trust · Identidad como perímetro · Accesos distribuidos · Multi-cloud.
* 🧩 **Contenido**:
  En entornos on-premise, el perímetro estaba protegido por firewalls y segmentación de red. En cloud, **el perímetro es la identidad**: si un atacante consigue credenciales válidas, puede entrar directamente en la consola y gestionar recursos sin necesidad de explotar vulnerabilidades técnicas.

  * El 80% de las brechas en cloud comienzan con credenciales comprometidas.
  * Las identidades no son solo personas: también hay identidades de máquinas, aplicaciones y servicios.
  * La gestión centralizada de identidades es crítica en entornos híbridos o multi-cloud.
    **Ejemplo**: claves IAM subidas accidentalmente a GitHub → acceso inmediato a toda la infraestructura AWS.
* 📚 [`01-introduccion.md`](modulo2-iam/01-introduccion.md)

---

### 2. IAM en AWS

* ✅ **Objetivo**: Conocer la arquitectura y componentes principales de AWS Identity and Access Management (IAM).
* 📌 **Claves**: Usuarios · Grupos · Roles · Policies JSON · STS.
* 🧩 **Contenido**:

  * **Usuarios y grupos**: para identidades humanas, con permisos heredados de grupos.
  * **Roles**: identidades sin credenciales permanentes, asociadas a servicios o cuentas.
  * **Policies JSON**: el corazón de IAM, permiten describir permisos con acciones (`s3:ListBucket`), recursos y condiciones.
  * **STS (Security Token Service)**: permite credenciales temporales para sesiones seguras.
    **Casos de uso**:
  * Dar a un servicio de EC2 acceso a un bucket S3 sin usar claves estáticas.
  * Crear un rol de solo lectura para auditores externos.
    **Error común**: usar la cuenta root de AWS para tareas diarias → riesgo crítico.
* 📚 [`02-iam-en-aws.md`](modulo2-iam/02-iam-en-aws.md)

---

### 3. Azure Active Directory (Azure AD)

* ✅ **Objetivo**: Comprender cómo Azure AD gestiona identidades, autenticación y autorización.
* 📌 **Claves**: Tenant · Usuarios · Grupos · Roles RBAC · Enterprise Apps.
* 🧩 **Contenido**:

  * **Azure AD vs Active Directory clásico**: Azure AD es un directorio basado en la nube, orientado a identidades modernas, no a dominios on-premise.
  * **Tenant**: contenedor raíz de identidades en Azure.
  * **Usuarios y grupos**: identidad centralizada para aplicaciones y recursos.
  * **RBAC (Role-Based Access Control)**: define permisos en diferentes niveles de scope (suscripción, grupo de recursos, recurso).
  * **Enterprise Apps**: integración con aplicaciones SaaS vía SSO (ej. Salesforce, Office 365).
    **Ejemplo**: asignar el rol "Reader" a un grupo de usuarios para toda una suscripción de Azure.
* 📚 [`03-azure-ad.md`](modulo2-iam/03-azure-ad.md)

---

### 4. Autenticación y MFA

* ✅ **Objetivo**: Comprender cómo se verifica la identidad de un usuario o servicio en AWS y Azure, y reforzarla con MFA.
* 📌 **Claves**: Autenticación vs Autorización · MFA · SSO · Federation.
* 🧩 **Contenido**:

  * **Autenticación** = demostrar quién eres (password, token, certificado).
  * **Autorización** = qué puedes hacer una vez autenticado (políticas, roles, RBAC).
  * **MFA**: segundo factor (app móvil, token físico, SMS).
  * **Federación**: confiar en otro proveedor de identidad (ej. Azure AD ↔ AWS).
    **Ejemplos**:
  * AWS: activar MFA para usuarios IAM mediante aplicaciones como Authenticator.
  * Azure: activar MFA condicional para accesos administrativos o desde ubicaciones sospechosas.
    **Error común**: tener usuarios con acceso de administrador sin MFA → uno de los fallos más graves en seguridad cloud.
* 📚 [`04-autenticacion.md`](modulo2-iam/04-autenticacion.md)

---

### 5. Autorización: políticas, roles y scopes

* ✅ **Objetivo**: Comprender cómo se aplican permisos en AWS y Azure mediante distintos modelos de control de acceso.
* 📌 **Claves**: Policies JSON · RBAC · Custom Roles · Scopes jerárquicos.
* 🧩 **Contenido**:

  * **AWS IAM**: permisos detallados a través de políticas JSON (acciones, recursos, condiciones). Roles para delegar acceso.
  * **Azure RBAC**: permisos basados en roles aplicados en distintos scopes: suscripción, grupo de recursos, recurso individual.
  * **Custom roles**: tanto AWS como Azure permiten crear roles a medida con permisos específicos.
    **Ejemplo comparativo**:
  * AWS: política JSON que da `s3:GetObject` solo en un bucket concreto.
  * Azure: rol RBAC personalizado que da acceso de lectura solo a un storage account específico.
* 📚 [`05-autorizacion.md`](modulo2-iam/05-autorizacion.md)

---

### 6. Buenas prácticas en IAM

* ✅ **Objetivo**: Establecer un conjunto de reglas básicas para proteger la gestión de identidades y accesos.
* 📌 **Claves**: MFA obligatorio · Mínimo privilegio · Rotación de claves · Monitorización.
* 🧩 **Contenido**:

  * **No usar la cuenta root (AWS)** o la cuenta de administrador global (Azure) para tareas diarias.
  * Aplicar **principio de mínimo privilegio** siempre (asignar el rol más bajo que cumpla el objetivo).
  * Obligar a **MFA** en todos los accesos críticos.
  * Rotar y auditar credenciales periódicamente.
  * Revisar permisos con herramientas nativas: AWS IAM Access Analyzer / Azure PIM (Privileged Identity Management).
    **Ejemplo**: desactivar claves IAM que no se usan hace más de 90 días.
* 📚 [`06-buenas-practicas.md`](modulo2-iam/06-buenas-practicas.md)

---

### 7. Comparativa AWS vs Azure

* ✅ **Objetivo**: Entender las similitudes y diferencias entre IAM (AWS) y Azure AD + RBAC (Azure).
* 📌 **Claves**: IAM vs Azure AD · Policies vs RBAC · Roles vs Scopes · Usuarios vs Tenants.
* 🧩 **Contenido**:

  * **Equivalencias**:

    * Usuarios IAM ↔ Usuarios Azure AD.
    * Roles IAM ↔ Roles RBAC.
    * Policies JSON ↔ Definiciones RBAC + condiciones.
  * Diferencias clave:

    * En AWS, todo gira en torno a políticas JSON.
    * En Azure, el control está fuertemente jerarquizado en scopes (subscripción, RG, recurso).
      **Caso de uso**: crear un usuario de solo lectura en AWS (política JSON “ReadOnlyAccess”) vs en Azure (asignar rol “Reader” en un scope concreto).
* 📚 [`07-comparativa.md`](modulo2-iam/07-comparativa.md)