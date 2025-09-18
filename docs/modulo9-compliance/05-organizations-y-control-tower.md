# 🏢 AWS Organizations y Control Tower

---

## 🎯 Introducción

En entornos empresariales, una sola cuenta de AWS no es suficiente. Para una buena **gobernanza multi-cuenta**, AWS ofrece dos servicios clave:

* **AWS Organizations**: estructura jerárquica para agrupar cuentas.
* **AWS Control Tower**: entorno guiado para crear, asegurar y gobernar cuentas automáticamente.

Ambas herramientas permiten aplicar **políticas centralizadas**, auditar comportamientos y reducir riesgos operativos.

---

## 🧱 Arquitectura y componentes clave

| Componente                      | Descripción                                                            |
| ------------------------------- | ---------------------------------------------------------------------- |
| Cuenta raíz (management)        | Control central, donde se crea la organización.                        |
| Organizational Units (OUs)      | Agrupaciones lógicas (por equipo, entorno, país, etc.).                |
| Cuentas miembro                 | Cuentas individuales dentro de la organización.                        |
| Service Control Policies (SCPs) | Reglas que definen qué puede y no puede hacer una cuenta u OU.         |
| Guardrails                      | Políticas preconfiguradas de Control Tower (basadas en SCPs y Config). |

---

## 🧭 AWS Organizations

### ✅ ¿Qué es?

Permite gestionar múltiples cuentas como una sola organización, aplicando reglas y políticas globales.

### 📌 Funcionalidades:

* Crear OUs (por ejemplo: `producción`, `desarrollo`, `seguridad`, `auditoría`).
* Aplicar **SCPs** para limitar servicios, regiones, acciones.
* Consolidar facturación (coste unificado).
* Delegar administración por cuenta u OU.

### 🧩 Ejemplo: SCP que **impide uso de regiones fuera de Europa**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "eu-west-1",
            "eu-central-1"
          ]
        }
      }
    }
  ]
}
```

---

## 🏗️ AWS Control Tower

### ✅ ¿Qué es?

Es un servicio que simplifica la **creación y gobernanza de cuentas nuevas** dentro de AWS Organizations. Proporciona:

* **Landing Zone segura** por defecto.
* Creación automatizada de cuentas con configuración inicial (logging, auditoría, VPCs).
* Aplicación automática de **guardrails**.

### 📌 Guardrails:

* Son políticas predefinidas que aplican SCPs y AWS Config Rules.
* Dos tipos:

  * **Preventive**: deniegan (ej. impedir desactivación de CloudTrail).
  * **Detective**: auditan (ej. alertar si se crea un bucket sin cifrado).

---

## 🔄 Ejemplo completo: nueva cuenta segura

1. Admin lanza Control Tower.
2. Define OUs: `prod`, `dev`, `shared`.
3. Aplica guardrails como:

   * Impedir IAM users (solo roles).
   * Cifrado obligatorio en S3.
4. Crea nuevas cuentas para `Equipo A` y `Equipo B` dentro de `prod`.
5. Las cuentas nacen con:

   * CloudTrail habilitado.
   * Logging centralizado.
   * SCPs restrictivos.

---

## 💡 Tips para implementación

* Agrupa cuentas por **tipo de entorno** (ej: `OU-prod`, `OU-dev`) o **unidad organizativa**.
* Usa **nombres claros y consistentes**.
* Habilita **CloudTrail en todas las regiones**.
* Aplica **SCPs restrictivas por defecto**, luego abre excepciones justificadas.

---

## 🔍 Validación y prueba

### ▶️ AWS CLI – listar OUs y SCPs

```bash
# Listar Organizational Units
aws organizations list-organizational-units-for-parent \
  --parent-id <root-id>

# Ver políticas SCP aplicadas
aws organizations list-policies-for-target \
  --target-id <account-id> \
  --filter SERVICE_CONTROL_POLICY
```

### ▶️ Web console

1. Ir a "AWS Organizations" → ver jerarquía de OUs.
2. Navegar a "Control Tower" → verificar cuentas, landing zone, guardrails activos.
3. Validar aplicación de reglas desde AWS Config.

---

## ⚠️ Errores comunes

| Error                                                | Impacto                                            |
| ---------------------------------------------------- | -------------------------------------------------- |
| Crear cuentas fuera de Organizations                 | No aplican SCPs ni control centralizado            |
| Usar IAM users en vez de roles                       | Riesgo de credenciales estáticas, difícil de rotar |
| No habilitar guardrails o dejar en modo “audit only” | No hay enforcement; alertas sin acción             |
| No segregar cuentas por función/entorno              | Mezcla de permisos, auditoría y coste poco claros  |
| No centralizar logs en una cuenta de auditoría       | Riesgo de pérdida o manipulación de evidencias     |

---

## ✅ Buenas prácticas

* Crea al menos 3 cuentas base:

  * `Auditoría`: centraliza logs.
  * `Seguridad`: IAM, Config, CloudTrail.
  * `Sandbox`: pruebas y desarrollo.
* Aplica SCPs tipo *deny all* y habilita solo lo necesario.
* Documenta las políticas activas por OU.
* Realiza revisiones periódicas de compliance.
* Mantén un proceso de onboarding para nuevas cuentas.

---

## 🔗 Recursos recomendados

* [AWS Organizations – Docs](https://docs.aws.amazon.com/organizations/)
* [AWS Control Tower – Docs](https://docs.aws.amazon.com/controltower/)
* [Service Control Policies – Examples](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps_examples.html)
* [AWS Config Rules – Library](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)