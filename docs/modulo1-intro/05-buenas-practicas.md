# 📄 `05-buenas-practicas.md`


## 🧭 Introducción – Buenas prácticas iniciales en seguridad cloud

La mayoría de los incidentes de seguridad en la nube **no se deben a vulnerabilidades técnicas**, sino a errores de configuración, accesos mal gestionados o prácticas inseguras desde el inicio.

Adoptar una serie de **buenas prácticas desde el primer despliegue** marca una diferencia crítica en la postura de seguridad. Tanto AWS como Azure proporcionan mecanismos para establecer estas prácticas de forma preventiva y automatizada.

Este archivo recoge un **conjunto de medidas esenciales** que puedes aplicar inmediatamente, sin necesidad de experiencia avanzada, y que cubren identidad, acceso, cifrado, auditoría y gestión del cambio.

---

## 🔎 Explicación detallada – Prácticas clave desde el inicio

### 🔐 1. Autenticación multifactor (MFA)

> Protege las cuentas más sensibles con un segundo factor obligatorio.

| Plataforma | Configuración recomendada                                                     |
| ---------- | ----------------------------------------------------------------------------- |
| **AWS**    | Activar MFA para root y usuarios IAM con permisos administrativos             |
| **Azure**  | Crear políticas de acceso condicional que obliguen MFA en roles privilegiados |

📌 **Ejemplo crítico**: comprometer una cuenta sin MFA puede otorgar control total del entorno en segundos.

---

### 👤 2. Principio de mínimo privilegio

> Cada usuario, servicio o aplicación debe tener **solo los permisos estrictamente necesarios**.

| En AWS   | Crear roles con políticas limitadas (scoped policies), evitar permisos amplios (`*`)   |
| -------- | -------------------------------------------------------------------------------------- |
| En Azure | Usar RBAC, roles personalizados, evitar asignar “Owner” o “Contributor” a todo recurso |

🔍 **Consejo**: usa herramientas como AWS IAM Access Analyzer o Azure PIM para revisar y optimizar permisos.

---

### 🧪 3. Separación de cuentas y entornos

> No mezcles desarrollo, pruebas y producción en la misma cuenta o suscripción.

* **AWS**: usar AWS Organizations para separar entornos (`dev`, `test`, `prod`) con cuentas diferentes.
* **Azure**: crear suscripciones independientes por entorno y agrupar por Resource Groups y etiquetas.

✔️ Ventaja: reduce el riesgo de errores humanos, mejora la trazabilidad y permite políticas distintas por entorno.

---

### 🔑 4. Gestión segura de claves y secretos

> Las claves, tokens y contraseñas **nunca deben estar en texto plano** o subidas a repositorios.

| Servicio recomendado | Plataforma | Descripción                                       |
| -------------------- | ---------- | ------------------------------------------------- |
| AWS Secrets Manager  | AWS        | Almacén cifrado con rotación automática           |
| AWS KMS              | AWS        | Gestión de claves para cifrado en S3, RDS, etc.   |
| Azure Key Vault      | Azure      | Gestor central de claves, certificados y secretos |

🔍 Incluye auditoría y control de acceso granular a cada secreto.

---

### 🕵️ 5. Control de cambios y trazabilidad

> Cada cambio debe quedar registrado y, si es posible, aprobado previamente.

* **Infraestructura como código (IaC)**: Terraform, CloudFormation, Bicep.
* **Registro de actividad**: CloudTrail (AWS), Activity Logs (Azure).
* **Alertas ante cambios críticos**: CloudWatch Events, Azure Monitor Alerts.

✔️ Usa flujos de GitOps (pull request + revisión + despliegue) para cualquier cambio estructural.

---

## 🏢 Caso empresarial – Prácticas iniciales en una fintech

**Finvergo**, una fintech regulada, empieza a operar en Azure. Durante el diseño inicial, aplican estas prácticas:

* Separan entornos en distintas suscripciones (`finvergo-dev`, `finvergo-prod`)
* Activan **MFA obligatoria** para todo el equipo técnico.
* Usan **Azure Key Vault** para API keys y cadenas de conexión.
* Automatizan despliegues con Terraform y Bicep.
* Habilitan alertas ante cambios de configuración en recursos críticos (SQL, redes, identidad).

Resultado: la empresa pasa una auditoría PCI-DSS sin hallazgos relevantes en la nube.

---

## 💻 Código y configuración

### ✅ AWS – Activar MFA para un usuario IAM

```bash
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name mfa-usuario1 \
  --outfile /tmp/mfa-qrcode.png \
  --bootstrap-method QRCodePNG

# Asociar dispositivo MFA
aws iam enable-mfa-device \
  --user-name usuario1 \
  --serial-number arn:aws:iam::123456789012:mfa/mfa-usuario1 \
  --authentication-code1 123456 \
  --authentication-code2 654321
```

### ✅ Azure – Política de acceso condicional para MFA

> Requiere licencias Entra ID P1 o superior (antes Azure AD Premium P1).

```bash
az ad conditional-access policy create \
  --display-name "MFA para administradores" \
  --conditions '{"users":{"includeRoles":["GlobalAdministrator"]}}' \
  --grant-controls '{"operator":"OR","builtInControls":["mfa"]}' \
  --state enabled
```

---

## 🧪 Tips para probar

### En consola web

* **AWS**:

  * Ve a IAM → Activa MFA en la cuenta raíz.
  * Simula acceso de usuarios con `IAM Policy Simulator`.
  * Crea alertas en CloudWatch ante cambios en políticas o creación de claves.

* **Azure**:

  * Entra a Azure AD → Acceso condicional → Crear nueva política.
  * Usa Azure Monitor → Activar alertas por cambios de rol o configuración de red.
  * Crea un secreto en Key Vault y aplica control RBAC o políticas de acceso.

---

## ❌ Errores comunes

| Error                                        | Consecuencia                                | Recomendación                                             |
| -------------------------------------------- | ------------------------------------------- | --------------------------------------------------------- |
| Usar usuarios root/admin para tareas diarias | Altísimo riesgo de brecha total             | Crear usuarios sin privilegios y usar roles con elevación |
| Subir claves o `.env` a GitHub               | Filtración de acceso a datos/producto       | Usar gestores de secretos y `.gitignore` correctamente    |
| Dar acceso “Owner” por comodidad             | Escalada de privilegios, borrado accidental | Aplicar mínimo privilegio y control de acceso granular    |

---

## ✅ Buenas prácticas – Checklist rápido

✔️ Aplica este checklist en cada nuevo proyecto:

* [ ] MFA activado en todas las cuentas críticas
* [ ] Accesos y permisos revisados y limitados por rol
* [ ] Entornos separados por cuenta/suscripción
* [ ] Secretos almacenados en Key Vault / Secrets Manager
* [ ] Despliegues realizados con IaC y revisados en Git
* [ ] Logging y alertas activadas desde el primer día

---

## 📚 Recursos recomendados

| Recurso                                                                                                                                 | Descripción                                              |
| --------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)                                          | Guía oficial de buenas prácticas de identidad            |
| [Azure Identity Security Best Practices](https://learn.microsoft.com/en-us/azure/active-directory/fundamentals/security-best-practices) | Prácticas clave para proteger cuentas y accesos en Azure |
| [Terraform + GitHub Actions](https://learn.hashicorp.com/tutorials/terraform/github-actions)                                            | Cómo automatizar flujos seguros de despliegue            |
| [Microsoft Learn – Azure Security](https://learn.microsoft.com/en-us/training/paths/secure-azure-resources/)                            | Ruta formativa gratuita sobre seguridad en Azure         |
