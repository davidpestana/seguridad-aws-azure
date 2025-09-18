# 🧪 Laboratorio 02 – Crear un usuario en Azure AD y asignarle roles de acceso

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, el alumno será capaz de:

* Crear un nuevo usuario en Azure Active Directory (Azure AD).
* Asignar un rol predefinido al usuario para controlar su nivel de acceso.
* Probar el acceso desde una sesión separada.
* Entender los conceptos de **asignación de roles por suscripción**, **RBAC** y **scopes** en Azure.

---

## 🧵 Escenario práctico

Tu empresa ha incorporado a un nuevo miembro al equipo de IT. Necesitas crearle una cuenta de acceso limitada en Azure para que pueda consultar recursos, pero sin poder modificarlos. Harás esto usando **Azure Active Directory** y la **asignación de roles con control basado en roles (RBAC)**.

---

## 🧰 Requisitos previos

* Cuenta de Azure con permisos de **Administrador de usuarios** y **Administrador de acceso**.
* Acceso al portal de Azure: [https://portal.azure.com](https://portal.azure.com)
* Acceso a una segunda ventana privada o navegador para simular el login del nuevo usuario.

---

## 🧭 Pasos detallados

### 1️⃣ Crear un nuevo usuario en Azure Active Directory

1. En el portal, busca y entra en **Azure Active Directory**.
2. En el menú lateral, selecciona **Usuarios** → **Nuevo usuario**.
3. Configura los siguientes campos:

   * **Nombre**: `usuario-lab`
   * **Nombre de usuario**: `usuario-lab@<tudominio>.onmicrosoft.com`
   * **Contraseña inicial**: genera automáticamente
4. Anota el nombre de usuario y contraseña para la prueba de login.
5. Haz clic en **Crear**.

---

### 2️⃣ Asignar un rol de acceso a la suscripción (lectura)

1. En el portal, busca y accede a **Suscripciones**.
2. Selecciona tu suscripción activa.
3. En el menú izquierdo, entra a **Control de acceso (IAM)**.
4. Haz clic en **Agregar** → **Agregar asignación de roles**.
5. Configura:

   * **Rol**: *Lector (Reader)*
   * **Asignar acceso a**: *Usuario, grupo o entidad de servicio*
   * **Seleccionar**: busca y selecciona `usuario-lab`
6. Haz clic en **Revisar y asignar**.

✅ *Esto otorga permisos de solo lectura a nivel de suscripción.*

---

### 3️⃣ Validar acceso con el nuevo usuario

1. Abre una ventana de incógnito o nuevo navegador.
2. Accede a [https://portal.azure.com](https://portal.azure.com).
3. Inicia sesión con el nuevo usuario: `usuario-lab@<tudominio>.onmicrosoft.com`
4. Cambia la contraseña cuando se solicite.
5. Verifica:

   * Puedes ver recursos en el portal.
   * No puedes crear, borrar ni modificar recursos.

---

### 4️⃣ (Opcional) Probar acceso desde CLI

1. Instala la CLI de Azure si no la tienes:

```bash
# Linux/macOS
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# Windows
winget install -e --id Microsoft.AzureCLI
```

2. Inicia sesión:

```bash
az login
```

3. Selecciona el nuevo usuario y verifica que puedes listar recursos:

```bash
az resource list
```

✅ *Deberías poder listar pero no modificar recursos.*

---

## ✅ Validación

* ¿Se ha creado correctamente el usuario en Azure AD?
* ¿Tiene asignado el rol de “Lector” en la suscripción?
* ¿Puede iniciar sesión en el portal?
* ¿Se respetan las restricciones de solo lectura?

---

## ⚠️ Errores comunes

| Problema                         | Causa común                                  | Solución                                                    |
| -------------------------------- | -------------------------------------------- | ----------------------------------------------------------- |
| El usuario no puede ver recursos | Rol no asignado o asignado en otro scope     | Verifica la asignación y la suscripción correcta            |
| No se puede iniciar sesión       | Contraseña incorrecta o cuenta mal creada    | Restablece contraseña desde Azure AD                        |
| Acceso denegado en CLI           | No se ha hecho login con el usuario adecuado | Usa `az logout` y repite el `az login` con el nuevo usuario |

---

## 🧩 Extensiones opcionales

* Asigna el rol “Colaborador” solo a un grupo de recursos, no a toda la suscripción.
* Crea un grupo de Azure AD y prueba asignar permisos al grupo en lugar de al usuario.
* Establece una política de **MFA obligatoria** para usuarios con acceso a la suscripción.
