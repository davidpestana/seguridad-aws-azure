# 🖥️ EC2 en AWS y Máquinas Virtuales en Azure: Lanzamiento y Configuración Segura

---

## 🗂️ Introducción

Tanto **Amazon EC2** como las **Azure Virtual Machines** ofrecen servicios de cómputo bajo demanda, pero difieren en su enfoque, configuraciones predeterminadas y mecanismos de seguridad. En ambos casos, lanzar una instancia sin considerar la seguridad puede provocar **exposición inmediata a Internet** o acceso indebido a datos internos.

Este archivo compara las configuraciones críticas de seguridad en el despliegue de instancias en ambos proveedores.

---

## 🧱 1. Lanzamiento básico de instancias

### 🟡 AWS – EC2

| Elemento           | Recomendación de seguridad                                     |
| ------------------ | -------------------------------------------------------------- |
| **AMI**            | Usar imágenes oficiales o endurecidas (CIS, STIG, etc.)        |
| **VPC + Subnet**   | Subred privada por defecto, acceso restringido                 |
| **Security Group** | Permitir solo puertos necesarios (HTTP/HTTPS, SSH restringido) |
| **Key Pair (SSH)** | Generar nueva clave, nunca reutilizar                          |
| **EBS**            | Activar cifrado con KMS (por defecto desde 2021)               |
| **IAM Role**       | Asignar un rol mínimo para acceso a otros servicios            |

```bash
aws ec2 run-instances \
  --image-id ami-abc123 \
  --instance-type t3.micro \
  --key-name mi-clave-ssh \
  --security-groups sg-seguro \
  --subnet-id subnet-privada \
  --iam-instance-profile Name=rol-webapp \
  --block-device-mappings '[{"DeviceName":"/dev/xvda","Ebs":{"VolumeSize":20,"Encrypted":true}}]'
```

---

### 🔵 Azure – Virtual Machines

| Elemento                 | Recomendación de seguridad                                   |
| ------------------------ | ------------------------------------------------------------ |
| **Imagen base**          | Usar imágenes oficiales del Marketplace                      |
| **VNet + Subred**        | Subred privada; evitar exponer IP pública                    |
| **NSG (Security Group)** | Permitir solo tráfico necesario; evitar `Any`                |
| **Autenticación**        | Preferir claves SSH sobre contraseñas                        |
| **Disco OS + datos**     | Cifrado activado por defecto; puede usarse CMK con Key Vault |
| **Managed Identity**     | Activar identidad administrada para acceso a recursos        |

```bash
az vm create \
  --resource-group mi-rg \
  --name webappvm \
  --image Ubuntu2204 \
  --admin-username adminuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --vnet-name mi-vnet \
  --subnet subnet-privada \
  --nsg mi-nsg \
  --assign-identity
```

---

## 🔐 2. Seguridad de red

### ✅ Reglas mínimas en entornos productivos

| Servicio      | Reglas de red recomendadas                             |
| ------------- | ------------------------------------------------------ |
| HTTP/HTTPS    | Permitir desde el mundo (0.0.0.0/0), si sirve público  |
| SSH/RDP       | Solo desde IPs concretas (oficina, VPN, bastion)       |
| Otros puertos | Denegados por defecto; permitir solo con justificación |

### 🧱 AWS – Security Groups

* Stateless, aplicados por instancia.
* Ejemplo: permitir SSH solo desde IP concreta.

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-123456 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.10/32
```

### 🧱 Azure – Network Security Groups (NSG)

* Asociados a la NIC o subred.
* Reglas explícitas para cada dirección y puerto.

```bash
az network nsg rule create \
  --resource-group mi-rg \
  --nsg-name mi-nsg \
  --name AllowSSH \
  --priority 1000 \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --source-address-prefixes 203.0.113.10/32 \
  --destination-port-ranges 22
```

---

## 🔒 3. Acceso remoto seguro

| Mecanismo       | AWS                              | Azure                                   |
| --------------- | -------------------------------- | --------------------------------------- |
| Claves SSH      | ✅ Recomendado                    | ✅ Recomendado                           |
| Contraseñas     | ❌ No recomendado                 | ❌ Solo si se fuerza seguridad adicional |
| Bastion Host    | EC2 + Security Group restringido | Azure Bastion                           |
| JumpBox con MFA | Compatible vía SSM + IAM         | Compatible con AAD login + MFA          |

> 🔐 Consejo: deshabilitar acceso directo a root/administrador y limitar el acceso SSH/RDP a través de salto (bastion o gateway).

---

## 📦 4. Almacenamiento seguro

| Elemento            | AWS (EBS)                    | Azure (Managed Disks)                           |
| ------------------- | ---------------------------- | ----------------------------------------------- |
| Cifrado por defecto | ✅ desde 2021 con AWS KMS     | ✅ por defecto con clave gestionada por Azure    |
| Uso de CMK propia   | ✅ opcional via KMS           | ✅ opcional con Disk Encryption Sets + Key Vault |
| Snapshots           | ✅ automáticos vía AWS Backup | ✅ snapshots + vault                             |

---

## 🏢 Caso práctico: Servidor web seguro en ambas plataformas

**Escenario**: lanzar un servidor web en producción con acceso HTTP/HTTPS, SSH restringido, y almacenamiento cifrado.

| Requisito     | AWS                                      | Azure                                     |
| ------------- | ---------------------------------------- | ----------------------------------------- |
| Lanzamiento   | `aws ec2 run-instances`                  | `az vm create`                            |
| Imagen        | AMI oficial de Amazon Linux 2            | Ubuntu 22.04 desde Marketplace            |
| Red           | VPC privada + SG con puertos 22, 80, 443 | VNet privada + NSG restringido            |
| Cifrado       | EBS cifrado con KMS                      | Disco cifrado con Key Vault (opcional)    |
| Identidad     | IAM Role para acceder a S3               | Managed Identity para Storage / Key Vault |
| Acceso remoto | SSH solo desde IP fija                   | SSH con Azure Bastion                     |

---

## ❌ Errores comunes

1. **Permitir SSH o RDP desde “Any” (`0.0.0.0/0`)** sin justificación.
2. **Usar imágenes de terceros sin validación** (Marketplace o AMI desconocida).
3. **Reutilizar claves SSH entre múltiples instancias**.
4. **No asignar un rol o identidad administrada**, y luego incrustar credenciales.
5. **Dejar puertos abiertos tras pruebas** (ej. `tcp/3306`, `tcp/22`, etc.).
6. **No aplicar tags o etiquetas de seguridad** que permitan automatización de políticas.

---

## ✅ Checklist rápido de lanzamiento seguro

| Elemento                          | AWS                       | Azure                         |
| --------------------------------- | ------------------------- | ----------------------------- |
| Imagen base oficial o validada    | ✅                         | ✅                             |
| Cifrado de disco                  | ✅ (EBS + KMS)             | ✅ (Disk + Key Vault opcional) |
| Red privada                       | ✅ (VPC + SG)              | ✅ (VNet + NSG)                |
| Acceso remoto restringido         | ✅ (IP concreta o bastion) | ✅ (Bastion + NSG)             |
| Roles / identidades administradas | ✅ (IAM Role)              | ✅ (Managed Identity)          |
| Logs y monitoreo                  | CloudWatch Logs           | Azure Monitor + Log Analytics |

---

## 🔗 Recursos oficiales

* [Launch and Secure EC2 - AWS Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-instance_linux.html)
* [Azure VM Deployment Best Practices](https://learn.microsoft.com/en-us/azure/virtual-machines/best-practices)
* [Security Groups – AWS](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
* [NSG and Azure Firewall – Azure Docs](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-overview)
