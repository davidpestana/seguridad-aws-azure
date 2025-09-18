# ✅ Buenas prácticas en cómputo seguro (AWS y Azure)

---

## 🗂️ Introducción

A lo largo del módulo hemos cubierto los distintos aspectos que afectan a la **seguridad en servicios de cómputo en la nube**, desde el lanzamiento de instancias hasta su configuración, acceso y mantenimiento. Este archivo resume las **buenas prácticas clave** que deben aplicarse de forma sistemática en entornos productivos, previendo ataques, errores humanos y malas configuraciones.

---

## 🔐 Principios generales

| Principio                             | Aplicación práctica                                  |
| ------------------------------------- | ---------------------------------------------------- |
| **Mínimo privilegio**                 | Roles/identidades con permisos específicos por tarea |
| **Zero Trust**                        | No confiar en nadie por defecto (ni red interna)     |
| **Infraestructura inmutable**         | Reemplazar, no modificar instancias activas          |
| **Automatización de la seguridad**    | Hardening, parcheo, auditoría desde el inicio        |
| **Auditoría y trazabilidad completa** | Toda acción debe dejar registro                      |

---

## ✅ Buenas prácticas por categoría

### 🖥️ **Despliegue inicial**

| Acción                                  | AWS                                  | Azure                              |
| --------------------------------------- | ------------------------------------ | ---------------------------------- |
| Usar imágenes oficiales o endurecidas   | AMIs validadas o CIS Hardened Images | Marketplace + Imágenes verificadas |
| Lanzar en red privada por defecto       | VPC/Subnet privada                   | VNet/Subnet sin IP pública         |
| Asignar rol/identidad desde el arranque | IAM Role                             | Managed Identity                   |
| Cifrado de disco activado               | EBS cifrado por defecto o CMK        | Managed Disk + Key Vault opcional  |

---

### 🔐 **Acceso seguro**

| Recomendación                      | AWS                         | Azure                                |
| ---------------------------------- | --------------------------- | ------------------------------------ |
| Autenticación por clave SSH        | Key Pair EC2                | Clave pública + Azure Bastion        |
| Deshabilitar acceso por contraseña | `PasswordAuthentication no` | Desactivar login por password        |
| No compartir claves                | Clave única por usuario     | Clave única o AAD login              |
| Acceso limitado por IP             | Security Group restringido  | NSG con origen específico            |
| Autenticación federada / MFA       | IAM + SSO opcional          | AAD login + Conditional Access + MFA |

---

### 🔑 **Gestión de credenciales**

| Recomendación                          | AWS                             | Azure                                   |
| -------------------------------------- | ------------------------------- | --------------------------------------- |
| No incrustar claves o tokens en código | Usar IAM Role + Secrets Manager | Managed Identity + Key Vault            |
| Rotación automática                    | KMS/STSM                        | Azure Key Vault Policies                |
| Control de acceso a secretos           | IAM granular + audit logs       | RBAC por identidad + registro de acceso |

---

### 🛡️ **Hardening**

| Acción                              | Herramienta recomendada                   |
| ----------------------------------- | ----------------------------------------- |
| Deshabilitar servicios innecesarios | Cloud-init, Ansible, scripts iniciales    |
| Configurar firewall interno         | iptables/ufw (Linux), Windows Defender FW |
| Desactivar root/admin               | Bloqueo de cuenta o solo sudoers          |
| Activar logs del sistema            | CloudWatch / Log Analytics                |
| Evaluar contra estándares           | AWS Inspector / Defender for Cloud        |

---

### 🔁 **Parcheo y ciclo de vida**

| Recomendación                         | AWS                             | Azure                          |
| ------------------------------------- | ------------------------------- | ------------------------------ |
| Automatizar aplicación de parches     | Patch Manager (SSM)             | Update Management (Automation) |
| Evitar instancias “pet”               | Infraestructura inmutable + ASG | VMSS con Golden Image          |
| Crear imágenes parcheadas (“golden”)  | EC2 Image Builder               | Shared Image Gallery           |
| Parchear en ventanas de mantenimiento | Maintenance Windows             | Schedules + Monitor logs       |

---

### 📈 **Auditoría y cumplimiento**

| Elemento                              | AWS                          | Azure                               |
| ------------------------------------- | ---------------------------- | ----------------------------------- |
| Logging activado                      | CloudTrail, SSM, Inspector   | Azure Monitor, Defender for Cloud   |
| Alertas por acceso o cambios          | CloudWatch Alarms, GuardDuty | Log Analytics + Activity Alerts     |
| Revisión periódica de roles/políticas | IAM Access Analyzer          | Azure Policy / Compliance Dashboard |
| Detección de datos sensibles          | Amazon Macie                 | Microsoft Purview                   |

---

## 🧠 Prácticas recomendadas por perfil

| Perfil                   | Buenas prácticas específicas                                        |
| ------------------------ | ------------------------------------------------------------------- |
| **DevOps**               | Automatizar despliegue + hardening + parcheo                        |
| **SysAdmin**             | Configurar logs, firewall, backup, rotación de claves               |
| **Arquitecto Cloud**     | Diseñar entorno sin puntos únicos, con roles segregados             |
| **Seguridad/Compliance** | Definir políticas, alertas, ciclos de revisión y escaneos regulares |

---

## ❌ Errores comunes

1. **Permitir SSH/RDP desde 0.0.0.0/0** “temporalmente” → se olvida y queda expuesto.
2. **Usar la misma clave SSH en todas las máquinas**.
3. **Olvidar revocar acceso de ex-empleados o usuarios temporales**.
4. **No aplicar parches porque “funciona”** → vulnerabilidades conocidas siguen activas.
5. **No tener registros centralizados de actividad** → sin trazabilidad en caso de incidente.
6. **Asignar roles con `*:*` (full admin) “para que no falle”**.

---

## ✅ Checklist de cómputo seguro

| Elemento                                | ¿Aplicado? ✔ / ❌ |
| --------------------------------------- | ---------------- |
| Claves únicas por usuario               |                  |
| Roles/identidades mínimas por instancia |                  |
| Acceso restringido por red (SG/NSG)     |                  |
| Cifrado de disco con KMS / Key Vault    |                  |
| Actualización automática o golden image |                  |
| Logging centralizado activo             |                  |
| Auditoría de cambios                    |                  |
| Detección de malware / escaneos         |                  |

> ✅ Puedes usar esta tabla como **check de auditoría interna** para evaluar cada entorno desplegado.

---

## 🔗 Recursos recomendados

* [AWS: Security Best Practices for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-best-practices.html)
* [Azure: VM Security Overview](https://learn.microsoft.com/en-us/azure/virtual-machines/security-overview)
* [CIS Benchmarks for Cloud Workloads](https://www.cisecurity.org/cis-benchmarks)
* [Defender for Cloud Overview (Azure)](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction)
* [AWS Inspector Documentation](https://docs.aws.amazon.com/inspector/latest/user/inspector_introduction.html)
