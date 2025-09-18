# 🔐 Claves, contraseñas y autenticación en instancias de cómputo (AWS y Azure)

---

## 🗂️ Introducción

La **autenticación segura** es uno de los pilares de la protección de instancias de cómputo. Aun cuando el firewall y la imagen de base sean seguros, una **clave SSH mal gestionada** o una **contraseña débil** pueden poner en riesgo toda la infraestructura.

Este archivo explora las mejores prácticas y herramientas para **gestionar el acceso seguro** a instancias en AWS y Azure.

---

## 🧩 Principales mecanismos de autenticación

| Mecanismo              | AWS EC2                       | Azure VM                     |
| ---------------------- | ----------------------------- | ---------------------------- |
| Clave SSH              | ✅ Recomendado                 | ✅ Recomendado                |
| Contraseña             | ❌ (deshabilitado por defecto) | ✅ (opcional, no recomendado) |
| Certificados AAD       | ❌                             | ✅ (Azure AD login para VMs)  |
| MFA (factor adicional) | Opcional (bastion, jumpbox)   | Opcional (via Bastion/AAD)   |

---

## 🔐 Claves SSH: gestión y buenas prácticas

### ✅ En AWS EC2

* Al lanzar una instancia, puedes asociar un **Key Pair (clave pública SSH)**.
* La clave privada debe ser almacenada **localmente** y **nunca compartida ni versionada**.
* No se puede recuperar una clave perdida: habría que **crear una nueva instancia o usar SSM**.

```bash
aws ec2 create-key-pair --key-name mi-clave-segura \
  --query 'KeyMaterial' --output text > ~/.ssh/mi-clave.pem
chmod 400 ~/.ssh/mi-clave.pem
```

🔐 **Buenas prácticas:**

* Usar **una clave diferente por entorno o equipo**.
* **Eliminar claves no usadas** en AWS (evita pares obsoletos).
* Controlar acceso mediante **roles IAM**, no solo por clave.

---

### ✅ En Azure VMs

* Se puede inyectar una **clave SSH pública** al crear la VM.
* Alternativamente, puede configurarse autenticación por contraseña (desaconsejado en producción).

```bash
az vm create \
  --resource-group mi-rg \
  --name securevm \
  --image Ubuntu2204 \
  --admin-username devops \
  --ssh-key-values ~/.ssh/id_rsa.pub
```

🔐 **Buenas prácticas:**

* Desactivar el acceso por contraseña (`PasswordAuthentication no` en SSHD).
* Inyectar claves desde **Azure Key Vault** para entornos automatizados.
* Usar **Azure Bastion** para accesos administrativos (sin exponer puertos).

---

## 🔑 Gestión de secretos y contraseñas

### 🔒 AWS

| Servicio                | Uso principal                            |
| ----------------------- | ---------------------------------------- |
| **Secrets Manager**     | Contraseñas, tokens, claves API          |
| **SSM Parameter Store** | Configuración, claves, variables seguras |

Ejemplo: guardar una contraseña cifrada en Parameter Store

```bash
aws ssm put-parameter \
  --name "/db/admin/password" \
  --type SecureString \
  --value "MyS3cretPwd123" \
  --key-id alias/aws/ssm
```

---

### 🔒 Azure

| Servicio             | Uso principal                             |
| -------------------- | ----------------------------------------- |
| **Azure Key Vault**  | Almacén de secretos, claves, certificados |
| **Managed Identity** | Acceso controlado a Key Vault sin claves  |

Ejemplo: guardar un secreto

```bash
az keyvault secret set \
  --vault-name mi-kv \
  --name dbPassword \
  --value MyS3cretPwd123
```

---

## 👤 Autenticación con Azure Active Directory

En Azure es posible permitir el **inicio de sesión en la VM usando identidad corporativa de AAD**, tanto para Windows como Linux.

```bash
az vm extension set \
  --resource-group mi-rg \
  --vm-name mi-vm \
  --name AADLoginForLinux \
  --publisher Microsoft.Azure.ActiveDirectory.LinuxSSH
```

✅ Ventajas:

* Integración con **MFA y políticas de acceso condicional**.
* Control centralizado de quién puede acceder.
* Auditoría de inicios de sesión mediante Azure AD logs.

---

## 🧠 Buenas prácticas generales

| Recomendación                                   | Justificación                                               |
| ----------------------------------------------- | ----------------------------------------------------------- |
| ❌ **No usar contraseñas débiles o compartidas** | Fácil de vulnerar por fuerza bruta o filtraciones           |
| ✅ **Rotar claves periódicamente**               | Reduce el riesgo si se filtra una clave                     |
| ✅ **Acceso remoto restringido por IP**          | Menor superficie de ataque (evita escáneres globales)       |
| ✅ **Deshabilitar usuario root o admin**         | Obliga al uso de usuarios controlados con sudo              |
| ✅ **Almacenar secretos en un servicio seguro**  | Evita hardcoding en código o config                         |
| ✅ **Revisar logs de acceso periódicamente**     | Permite detectar accesos no autorizados o intentos fallidos |

---

## 🏢 Caso empresarial

**Escenario**: una empresa de desarrollo necesita que su equipo acceda a instancias EC2 y Azure VMs para mantenimiento.

| Requisito                                 | Implementación segura                        |
| ----------------------------------------- | -------------------------------------------- |
| Acceso remoto controlado                  | Solo desde IP de oficina / VPN               |
| Claves personalizadas por usuario         | SSH key pair distinta por dev, sin compartir |
| Acceso temporal para proveedores externos | SAS con expiración (Azure) / STS en AWS      |
| Auditoría de accesos                      | CloudTrail + Auth Logs en Azure              |
| Almacenamiento de contraseñas de DB       | Secrets Manager (AWS) / Key Vault (Azure)    |

---

## ❌ Errores comunes

1. Usar una **única clave SSH compartida** entre todo el equipo.
2. Activar acceso SSH sin restricciones (`0.0.0.0/0`).
3. Usar **contraseñas por defecto o sin complejidad**.
4. Incluir claves o contraseñas **en scripts versionados (GitHub)**.
5. No eliminar claves o usuarios de ex-empleados.
6. No rotar secretos tras eventos críticos o cambios de personal.

---

## ✅ Checklist rápido

| Elemento                            | AWS                   | Azure                         |
| ----------------------------------- | --------------------- | ----------------------------- |
| Claves SSH seguras                  | ✅                     | ✅                             |
| Acceso por contraseña deshabilitado | ✅ (por defecto)       | ❌ (se debe forzar)            |
| Almacenamiento de secretos          | Secrets Manager / SSM | Key Vault                     |
| Autenticación federada              | No nativo en EC2      | ✅ Azure AD login              |
| MFA para acceso                     | Vía bastion/jumpbox   | Con AAD o Azure Bastion       |
| Logs de acceso remoto               | CloudTrail / SSM      | Azure Monitor / Activity Logs |

---

## 🔗 Recursos recomendados

* [AWS: Connect to your Linux instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)
* [Managing Secrets with AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
* [Azure: Secure Linux VMs using SSH keys](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/create-ssh-keys-detailed)
* [Azure Key Vault Overview](https://learn.microsoft.com/en-us/azure/key-vault/general/overview)