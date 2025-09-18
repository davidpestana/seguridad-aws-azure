# 📄 `01-que-es-seguridad-cloud.md`


## 🧭 Introducción – ¿Qué es la seguridad en la nube y por qué importa?

La seguridad en la nube no es simplemente trasladar las buenas prácticas del datacenter al entorno virtualizado del proveedor cloud. Implica **repensar** los controles, **redefinir** los perímetros y **adoptar una nueva mentalidad** centrada en la identidad, la automatización y la visibilidad constante.

En los entornos tradicionales (on-premise), la seguridad estaba asociada a **muros físicos**: firewalls perimetrales, control de acceso al datacenter, cableado estructurado y segmentación LAN. Pero en la nube, los activos están expuestos por diseño: **cualquier recurso mal configurado puede ser accesible desde cualquier parte del mundo**.

Conocer estas diferencias no es opcional: es la **primera línea de defensa** para evitar errores de configuración, fugas de datos, escaladas de privilegios y vulnerabilidades silenciosas.

---

## 🔎 Explicación detallada: seguridad en entornos cloud vs on-premise

### 🔄 Cambio de paradigma

| Elemento clave         | On-Premise                         | Cloud                                      |
| ---------------------- | ---------------------------------- | ------------------------------------------ |
| Perímetro de seguridad | Físico / Red (firewall, switches)  | Lógico (identidades, políticas, etiquetas) |
| Control de acceso      | Local (sala, tarjeta, red privada) | Federado, remoto, basado en rol            |
| Escalado               | Manual, lento                      | Automático, instantáneo                    |
| Aislamiento            | VLANs, firewalls                   | Seguridad por diseño (VPC, NSG, SG, etc.)  |
| Auditoría              | Dispersa, logs en local            | Centralizada, estructurada, en tiempo real |

---

### ⚠️ Nuevas amenazas: configuración como vector de ataque

La **configuración incorrecta** es la causa más frecuente de incidentes en la nube. Algunos ejemplos reales:

* Buckets S3 con permisos públicos abiertos accidentalmente.
* Bases de datos Azure SQL sin firewall habilitado.
* Roles IAM sobreprivilegiados expuestos a actores maliciosos.
* Dispositivos IoT conectados sin autenticación fuerte.

El **problema no es la nube**, sino **cómo se usa**. El enfoque tradicional de “proteger el perímetro” se diluye; ahora la **configuración incorrecta es el nuevo agujero de seguridad**.

---

### 🔐 La identidad como nuevo perímetro

AWS y Azure convergen en un principio clave: **la identidad y el control de acceso son el núcleo de la seguridad**.

| Componente                      | AWS                                | Azure                                 |
| ------------------------------- | ---------------------------------- | ------------------------------------- |
| Gestión de identidades          | IAM (Identity & Access Management) | Azure Active Directory (AAD)          |
| Autenticación multifactor (MFA) | Integrado y extensible             | Integrado con políticas condicionales |
| Control basado en rol           | IAM Policies, Roles                | RBAC (Role-Based Access Control)      |

Cada acción en la nube se valida a través de **tokens y políticas**. No hay “detrás del firewall”: todo se define por **quién eres y qué puedes hacer**.

---

## 🏢 Ejemplo empresarial: la empresa "CloudlyTech"

**CloudlyTech**, una startup tecnológica que migra su stack de desarrollo a AWS, decide exponer su API REST como servicio gestionado en una instancia EC2 detrás de un Application Load Balancer (ALB).

Un desarrollador configura un bucket S3 con los archivos estáticos del frontend... **y habilita accidentalmente el acceso público** para facilitar las pruebas.

Resultado: el sitio web y sus recursos están disponibles **globalmente**, incluyendo archivos de configuración con credenciales en texto plano que se habían subido por error.

🔍 **Causa raíz**: falta de comprensión de los mecanismos de seguridad cloud, ausencia de políticas preventivas y nula validación automática de configuraciones.

🔒 **Solución**:

* Activar **AWS Config** para auditar configuraciones no seguras.
* Aplicar **políticas de bucket estrictas** y bloqueos de acceso público.
* Establecer flujos de revisión antes del despliegue.

---

## 💻 Código y configuración

### 📍 AWS CLI – ejemplo de comprobación de acceso a un bucket S3

```bash
# Verifica si un bucket tiene acceso público habilitado
aws s3api get-bucket-policy-status --bucket NOMBRE_DEL_BUCKET
```

### 📍 Azure CLI – comprobar configuración de firewall en una base SQL

```bash
# Listar las reglas de firewall configuradas en una base de datos SQL
az sql server firewall-rule list --resource-group RG-NOMBRE --server SERVER-NOMBRE
```

---

## 🧪 Tips para probar

### 🔍 Desde la consola web

* En **AWS**:

  * Ve a S3 → Configura un bucket → Habilita/deshabilita acceso público → Observa alertas.
  * IAM → Usuarios → Revisa políticas asignadas y prueba acceso simulado.

* En **Azure**:

  * Accede a **Storage Accounts** → Configura firewalls y redes virtuales.
  * Prueba políticas RBAC en Azure AD → Asigna roles mínimos a un usuario.

### 💡 En CLI:

* Usa `aws iam simulate-principal-policy` para comprobar si un usuario puede acceder a un recurso.
* Usa `az role assignment list` para verificar permisos de usuarios en Azure.

---

## ❌ Errores comunes

| Error                                       | Descripción                                                                                | Cómo evitarlo                                                                   |
| ------------------------------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------- |
| ✅ Asumir que “todo está seguro por defecto” | Muchos servicios cloud se crean con configuraciones abiertas para facilitar el desarrollo. | Revisar configuraciones antes de producción. Aplicar seguridad desde el diseño. |
| ✅ Usar una sola cuenta para todo            | Mezclar desarrollo y producción en un mismo tenant es peligroso.                           | Separar entornos y aplicar restricciones por entorno.                           |
| ✅ Exponer claves en repositorios            | Subir `.env`, `config.json` o claves a GitHub es muy frecuente.                            | Usar gestores de secretos (Secrets Manager, Key Vault).                         |

---

## ✅ Buenas prácticas

✔️ Aplica estas reglas desde el primer proyecto:

* Crea cuentas separadas por entorno (dev/test/prod).
* Obliga al uso de **MFA** para usuarios con permisos elevados.
* Usa **nombres de recursos que indiquen propósito y sensibilidad**.
* Automatiza validaciones de seguridad con AWS Config o Azure Policy.
* Revisa los logs de acceso desde el primer día.

---

## 📚 Recursos recomendados

| Recurso                                                                                                       | Descripción                                        |
| ------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| [AWS Security Documentation](https://docs.aws.amazon.com/security/)                                           | Portal oficial de seguridad de AWS                 |
| [Microsoft Azure Security Documentation](https://learn.microsoft.com/en-us/security/azure-security/)          | Guía centralizada de seguridad en Azure            |
| [CIS Benchmarks Cloud](https://www.cisecurity.org/benchmarks)                                                 | Estándares y configuraciones seguras por proveedor |
| [AWS Well-Architected – Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/) | Guía de diseño seguro en la nube AWS               |
