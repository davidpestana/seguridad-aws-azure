# 📄 `05-autorizacion.md`

## 🧭 Introducción – ¿Qué es la autorización en la nube?

Una vez autenticada una identidad (usuario, servicio, aplicación), el siguiente paso es **decidir qué puede hacer** dentro del entorno cloud. Esto es lo que llamamos **autorización**.

En la nube, la autorización se basa en:

* **Políticas (AWS)**: documentos JSON que definen permisos detallados sobre recursos.
* **Roles RBAC (Azure)**: asignaciones de rol sobre recursos en diferentes niveles (scopes).

El diseño de permisos es fundamental para aplicar el **principio de mínimo privilegio** y evitar sobreexposición de recursos.
Un error aquí puede permitir, por ejemplo, que un desarrollador borre una base de datos de producción sin querer.

---

## 🔎 Explicación detallada – Modelos de control de acceso

### ✅ AWS – Control basado en **políticas JSON**

* **Policies**: documento JSON que define:

  * Acciones permitidas/denegadas: `s3:GetObject`, `ec2:StopInstances`, etc.
  * Recursos afectados: buckets, instancias, funciones Lambda…
  * Condiciones opcionales: IP, etiquetas, horario, MFA activado…

* **Asignación**:

  * A usuarios IAM, grupos o roles.

#### 🧩 Ejemplo: política de lectura en un bucket S3

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::mi-bucket/*"
    }
  ]
}
```

✔️ Políticas pueden ser **administradas (AWS o cliente)** o **inline**.

---

### ✅ Azure – Control basado en **RBAC (Role-Based Access Control)**

* **Rol RBAC**: conjunto de permisos definidos (ej. `Reader`, `Contributor`, `Owner`).
* **Asignación de rol**: vínculo entre un rol, una identidad (usuario, grupo o app), y un scope.

#### Jerarquía de scopes en Azure:

1. **Suscripción**
2. **Grupo de recursos**
3. **Recurso individual**

✔️ El **rol se hereda hacia abajo**, es decir, si asignas un rol en una suscripción, se aplica a todos los recursos dentro.

#### 🧩 Ejemplo: asignar `Reader` a un grupo para una suscripción completa

```bash
az role assignment create \
  --assignee <ID_GRUPO> \
  --role "Reader" \
  --scope /subscriptions/<ID_SUBSCRIPCION>
```

---

## 🧰 Comparativa práctica: AWS vs Azure

| Aspecto                 | AWS IAM                                   | Azure RBAC                                       |
| ----------------------- | ----------------------------------------- | ------------------------------------------------ |
| Unidad de control       | Política JSON                             | Rol RBAC                                         |
| Nivel de detalle        | Muy granular (acción, recurso, condición) | Más abstracto (conjunto predefinido de permisos) |
| Scope                   | Por recurso (vía ARN)                     | Scope jerárquico (subscripción → RG → recurso)   |
| Permisos personalizados | Policies personalizadas                   | Custom Roles                                     |
| Delegación temporal     | Roles con STS                             | Roles con duración limitada (PIM)                |

---

## 🏢 Caso empresarial – Acceso limitado a base de datos en cada plataforma

La empresa **InnovaFin** tiene dos equipos:

* Equipo A (AWS): necesita **solo lectura** en un bucket S3.
* Equipo B (Azure): necesita **modificar datos** en una base de datos SQL sin acceder al resto de recursos.

### 🟨 En AWS:

1. Crear una política JSON con `s3:GetObject`.
2. Asignarla a un grupo `lectores-s3`.

### 🟦 En Azure:

1. Crear un grupo `sql-editors`.
2. Asignar el rol `SQL DB Contributor` **solo en el scope del recurso SQL**.

Resultado: cada equipo tiene los permisos exactos requeridos, sin riesgo de sobreprivilegio.

---

## 💻 Configuración – Ejemplos prácticos

### ✅ AWS – Política inline con condición de MFA

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

✔️ Esta política **deniega todo acceso si el usuario no tiene MFA activado**.

---

### ✅ Azure – Crear un rol personalizado con acceso de solo lectura a blobs

```bash
az role definition create --role-definition '{
  "Name": "BlobReadOnlyCustom",
  "IsCustom": true,
  "Description": "Lectura de blobs únicamente",
  "Actions": [
    "Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"
  ],
  "NotActions": [],
  "AssignableScopes": ["/subscriptions/<ID_SUBSCRIPCION>"]
}'
```

---

## 🧪 Tips para probar

### En consola web

* **AWS**:

  * IAM → Simulador de políticas → Probar acceso a recursos antes de aplicar.
  * Asociar un rol a una Lambda → Ver si accede correctamente a S3.

* **Azure**:

  * Azure Portal → IAM de un recurso → Asignar rol → Usar modo “Modo de prueba”.
  * Azure Monitor → Activar alerta si alguien intenta acceder fuera de scope.

### En CLI

* Ver políticas aplicadas en AWS:

```bash
aws iam list-attached-user-policies --user-name usuarioX
```

* Ver roles RBAC aplicados en Azure:

```bash
az role assignment list --assignee <usuario> --output table
```

---

## ❌ Errores comunes

| Error                                                 | Consecuencia                               | Solución                                            |
| ----------------------------------------------------- | ------------------------------------------ | --------------------------------------------------- |
| Asignar permisos directamente a usuarios individuales | Gestión caótica y difícil de escalar       | Usar grupos para centralizar permisos               |
| Usar `*:*` en políticas IAM                           | Acceso total no controlado                 | Aplicar principio de mínimo privilegio              |
| Asignar `Owner` a todos los recursos por comodidad    | Riesgo de borrado, escalado de privilegios | Usar roles específicos como `Reader`, `Contributor` |

---

## ✅ Buenas prácticas

✔️ Checklist para autorización segura:

* [ ] Evitar asignaciones individuales → usar grupos.
* [ ] Preferir políticas y roles predefinidos, salvo que sea estrictamente necesario.
* [ ] Limitar el scope de permisos al recurso mínimo posible.
* [ ] Revisar asignaciones mensualmente con herramientas nativas.
* [ ] Activar alertas ante nuevas asignaciones de rol o cambios de política.

---

## 📚 Recursos recomendados

| Recurso                                                                                                   | Descripción                                      |
| --------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| [AWS IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html)         | Guía completa sobre creación de políticas JSON   |
| [Azure RBAC Overview](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)         | Explicación oficial del modelo de roles y scopes |
| [Policy Simulator – AWS](https://policysim.aws.amazon.com/)                                               | Herramienta online para simular permisos IAM     |
| [Azure RBAC Custom Roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/custom-roles) | Cómo crear roles RBAC personalizados             |