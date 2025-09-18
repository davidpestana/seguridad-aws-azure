# 📄 `01-introduccion.md`


## 🧭 Introducción – ¿Por qué la identidad es el nuevo perímetro en la nube?

En los entornos tradicionales (on-premise), la seguridad se construía alrededor de un **perímetro físico**: firewalls, switches, VLANs y acceso restringido al datacenter.
Pero en la nube, ese perímetro ya no existe.

Hoy, **la identidad es el nuevo perímetro de seguridad**: quien tenga las credenciales adecuadas puede entrar desde cualquier lugar del mundo, sin necesidad de vulnerar la red.

### 🌐 Realidad actual

* El 80% de las brechas en la nube comienzan con **credenciales comprometidas**.
* Las identidades no son solo personas: también incluyen **máquinas, servicios, contenedores y funciones sin servidor**.
* La gestión de identidades **es crítica** para entornos **multi-cloud** y distribuidos.

---

## 🔎 Explicación detallada – Qué implica gestionar la identidad en cloud

### 🔐 De perímetros físicos a identidades distribuidas

| Enfoque tradicional           | Enfoque en la nube                 |
| ----------------------------- | ---------------------------------- |
| Firewall de red               | Política de permisos basada en rol |
| Segmentación LAN              | Segmentación por recursos y scopes |
| Control de acceso físico      | Autenticación multifactor (MFA)    |
| Protección de infraestructura | Protección de identidad y tokens   |

> ⚠️ La nube elimina muchas barreras de acceso, pero **multiplica los vectores si no se gestiona bien la identidad**.

---

### 🧱 Tipos de identidades

En cloud, las identidades no se limitan a usuarios humanos. Debemos considerar:

| Tipo de identidad | Descripción                                              | Ejemplo                                       |
| ----------------- | -------------------------------------------------------- | --------------------------------------------- |
| Usuario humano    | Persona que accede a la consola o APIs                   | [admin@empresa.com](mailto:admin@empresa.com) |
| Servicio          | Aplicación o proceso que requiere acceso a recursos      | función Lambda, contenedor, VM                |
| Aplicación        | Cliente OAuth2 que accede en nombre propio o de usuarios | API web integrada con Azure AD                |
| Federada          | Identidad externa autenticada vía SAML/OIDC              | Usuario de Google o Microsoft 365             |

---

## 🏢 Caso empresarial – Claves filtradas en GitHub

Una startup sube su repositorio privado a GitHub sin excluir el archivo `.env`.
El archivo contiene **claves IAM de AWS con permisos de administrador**.

El repositorio es clonado por un bot, que usa las credenciales para:

* Crear máquinas virtuales para minería de criptomonedas.
* Borrar copias de seguridad en S3.
* Exfiltrar datos de usuarios.

🔍 **Causa raíz**: identidad mal gestionada, sin MFA ni limitación de permisos, y claves estáticas sin rotación.

🔒 **Medidas correctivas**:

* Eliminar claves estáticas y usar **roles temporales**.
* Habilitar MFA para todos los usuarios IAM.
* Supervisar accesos con **CloudTrail** o **Defender for Cloud**.
* Implementar **gestión de secretos** (Secrets Manager, Key Vault).

---

## 💻 Configuración – Buenas prácticas iniciales

### ✅ AWS – Crear usuario IAM con MFA y sin acceso programático

```bash
aws iam create-user --user-name auditor-externo

aws iam attach-user-policy \
  --user-name auditor-externo \
  --policy-arn arn:aws:iam::aws:policy/SecurityAudit

# Asignar MFA (debe hacerse vía consola o script adicional con códigos de validación)
```

### ✅ Azure – Crear usuario y asignar rol “Reader” en suscripción

```bash
az ad user create \
  --display-name "auditor externo" \
  --user-principal-name auditor@empresa.onmicrosoft.com \
  --password ContraseñaSegura123!

az role assignment create \
  --assignee auditor@empresa.onmicrosoft.com \
  --role Reader \
  --scope /subscriptions/<ID_SUSCRIPCION>
```

---

## 🧪 Tips para probar

### En consola web

* **AWS**:

  * Ir a IAM → Ver todos los usuarios → Revisar MFA y claves activas.
  * Activar **IAM Access Analyzer** para ver qué recursos son accesibles externamente.

* **Azure**:

  * Ir a Azure AD → Usuarios → Revisar método de autenticación (MFA activado).
  * Revisar actividad de inicio de sesión desde Azure Monitor.

### En CLI

* Ver claves activas de un usuario en AWS:

  ```bash
  aws iam list-access-keys --user-name <nombre>
  ```

* Ver si un usuario tiene MFA en Azure:

  ```bash
  az ad user list --query "[].{user: userPrincipalName, mfa: strongAuthenticationRequirements}" -o table
  ```

---

## ❌ Errores comunes

| Error                                        | Consecuencia                       | Solución                                    |
| -------------------------------------------- | ---------------------------------- | ------------------------------------------- |
| Usar claves de acceso estáticas sin rotación | Acceso ilimitado si se filtran     | Usar roles temporales o federación          |
| No habilitar MFA en cuentas críticas         | Alto riesgo de secuestro de sesión | Política obligatoria de MFA desde el inicio |
| Compartir credenciales entre servicios       | Falta de trazabilidad y auditoría  | Crear identidades separadas por servicio    |

---

## ✅ Buenas prácticas esenciales

✔️ Desde el primer día:

* [ ] Crea identidades separadas por persona, servicio o aplicación.
* [ ] Habilita MFA en usuarios administrativos y críticos.
* [ ] Revisa accesos y permisos al menos una vez al mes.
* [ ] Usa políticas de acceso de mínimo privilegio.
* [ ] Prohíbe claves de acceso estáticas en producción.

---

## 📚 Recursos recomendados

| Recurso                                                                                                                                   | Descripción                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)                                            | Guía oficial de buenas prácticas de identidad |
| [Microsoft Identity Security Overview](https://learn.microsoft.com/en-us/security/zero-trust/deploy/identity-security-overview)           | Base para seguridad de identidades en Azure   |
| [OWASP Cloud Top 10 – 2023](https://owasp.org/www-project-cloud-native-application-security-top-10/)                                      | Principales riesgos en entornos cloud         |
| [Azure AD Authentication Methods](https://learn.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-methods) | Métodos de autenticación soportados por Azure |
