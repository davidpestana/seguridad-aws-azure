# 📄 `04-autenticacion.md`

## 🧭 Introducción – Autenticación, MFA y federación en la nube

En la seguridad cloud, uno de los pilares más críticos es **la autenticación**: el proceso de verificar **quién eres** antes de otorgarte acceso a los recursos.
En este proceso participan:

* Métodos de autenticación (contraseñas, tokens, certificados).
* Segundo factor o MFA (Multi-Factor Authentication).
* Proveedores de identidad externos (federación).

Tanto AWS como Azure proporcionan mecanismos robustos de autenticación, pero **la configuración adecuada es clave**. Muchos incidentes de seguridad ocurren porque se omite el uso de MFA o se acepta autenticación sin restricciones geográficas ni de contexto.

---

## 🔎 Explicación detallada – Autenticación ≠ Autorización

| Concepto          | ¿Qué significa?                         | Ejemplo                                                          |
| ----------------- | --------------------------------------- | ---------------------------------------------------------------- |
| **Autenticación** | Verificar tu identidad                  | Iniciar sesión con usuario + contraseña + MFA                    |
| **Autorización**  | Decidir qué puedes hacer una vez dentro | Acceder a un bucket S3, modificar una VM, leer una base de datos |

> ❗Una buena autenticación **sin una buena autorización** sigue siendo un problema… y viceversa.

---

## 🔐 MFA (Multi-Factor Authentication)

**MFA añade una capa de seguridad extra**, obligando a verificar tu identidad con un segundo factor:

| Factor          | Ejemplos                                                                |
| --------------- | ----------------------------------------------------------------------- |
| Algo que sabes  | Contraseña                                                              |
| Algo que tienes | Token físico, app móvil (Microsoft Authenticator, Google Authenticator) |
| Algo que eres   | Huella dactilar, rostro, voz                                            |

🔍 En entornos cloud, **activar MFA para todos los usuarios administrativos debería ser obligatorio**.

---

## 🔁 Federación de identidades

La federación permite a los usuarios **iniciar sesión usando su identidad corporativa o externa**, sin necesidad de crear cuentas adicionales en el proveedor cloud.

| Plataforma | Soporta federación con…                                  |
| ---------- | -------------------------------------------------------- |
| **AWS**    | Azure AD, Google Workspace, ADFS, SAML 2.0, OIDC         |
| **Azure**  | Microsoft Entra ID, Google, Facebook, SAML, OAuth2, OIDC |

✔️ Ideal para empresas que ya tienen un directorio corporativo (como Microsoft 365, LDAP, etc.).

---

## 🏢 Caso empresarial – MFA y SSO para consultores externos

La consultora **SecuraTech** gestiona entornos AWS y Azure para varios clientes. Necesita:

* Otorgar acceso temporal a usuarios externos.
* Evitar gestionar credenciales manualmente.
* Asegurar MFA en todos los accesos.

Solución aplicada:

* **AWS**: configura federación con Azure AD, y crea roles IAM específicos para los usuarios externos.
* **Azure**: activa **SSO** desde Azure AD con políticas de acceso condicional (MFA y ubicación geográfica).

Resultado: los usuarios acceden con sus credenciales corporativas y cumplen los requisitos de seguridad desde el primer inicio de sesión.

---

## 💻 Configuración – Ejemplos prácticos

### ✅ AWS – Activar MFA para un usuario IAM (CLI parcial)

> **Nota**: activar MFA completamente requiere usar consola web o scripts con autenticación interactiva.

1. Crear dispositivo virtual MFA:

```bash
aws iam create-virtual-mfa-device \
  --virtual-mfa-device-name mfa-usuario1 \
  --outfile /tmp/qrcode.png \
  --bootstrap-method QRCodePNG
```

2. Asociar el dispositivo (requiere códigos del usuario):

```bash
aws iam enable-mfa-device \
  --user-name usuario1 \
  --serial-number arn:aws:iam::123456789012:mfa/mfa-usuario1 \
  --authentication-code1 123456 \
  --authentication-code2 654321
```

---

### ✅ Azure – Crear política de acceso condicional para requerir MFA

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

  * Ir a IAM → Usuarios → Verifica columna “MFA”.
  * Intenta iniciar sesión sin segundo factor → será denegado si MFA está habilitado.

* **Azure**:

  * Azure AD → Acceso condicional → Crear nueva política → Simular acceso con usuario de prueba.
  * Revisa logs de inicio de sesión en Azure Monitor.

### En CLI

* Ver si un usuario tiene MFA en AWS:

```bash
aws iam list-mfa-devices --user-name <nombre>
```

* Ver políticas de acceso condicional activas en Azure:

```bash
az ad conditional-access policy list --output table
```

---

## ❌ Errores comunes

| Error                               | Consecuencia                                      | Solución                                                               |
| ----------------------------------- | ------------------------------------------------- | ---------------------------------------------------------------------- |
| No activar MFA en cuentas críticas  | Robo de cuenta con solo la contraseña             | MFA obligatorio para admins y usuarios con acceso a datos sensibles    |
| Compartir cuentas sin federación    | Pérdida de trazabilidad y control                 | Configurar SSO con identidad corporativa (SAML/OIDC)                   |
| Permitir accesos desde cualquier IP | Acceso malicioso desde ubicaciones no controladas | Aplicar filtros geográficos o por red mediante políticas condicionales |

---

## ✅ Buenas prácticas

✔️ Checklist de autenticación segura:

* [ ] MFA activado para todos los usuarios con roles críticos.
* [ ] Federar identidades externas (Google, Azure AD, etc.) en vez de crear cuentas duplicadas.
* [ ] No usar claves de acceso largas → preferir sesiones temporales.
* [ ] Auditar intentos de inicio de sesión y aplicar alertas.
* [ ] Aplicar acceso condicional basado en ubicación y tipo de dispositivo.

---

## 📚 Recursos recomendados

| Recurso                                                                                                          | Descripción                                    |
| ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| [AWS – Enforcing MFA](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa.html)                  | Guía oficial de activación y uso de MFA en AWS |
| [Azure – Conditional Access](https://learn.microsoft.com/en-us/azure/active-directory/conditional-access/)       | Acceso condicional en Azure AD                 |
| [AWS Federation with SAML](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_saml.html)        | Integración de SAML SSO con AWS                |
| [Azure – Identity Federation](https://learn.microsoft.com/en-us/entra/fundamentals/identity-federation-overview) | Cómo federar Azure AD con otras plataformas    |
