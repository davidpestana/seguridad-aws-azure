# 📄 `03-nsg-y-security-groups.md`

## 🛡 Network Security Groups (NSG) y Security Groups (SG)

### 🎯 Objetivo

Configurar reglas de red que controlen el tráfico entrante y saliente de los recursos desplegados en AWS y Azure, entendiendo las diferencias entre NSG, SG y NACLs, y aplicando filtros efectivos según el caso de uso.

---

## 🧠 Fundamentos de control de tráfico

En entornos cloud, los filtros de tráfico se implementan **en el borde del recurso** (NIC o subred), no mediante dispositivos físicos como firewalls. Estos filtros:

* **Controlan tráfico entrante y saliente.**
* **Pueden aplicarse a máquinas, bases de datos, balanceadores, etc.**
* **Definen protocolos, puertos, rangos IP y prioridades.**

---

## ⚖ Comparativa rápida

| Concepto               | AWS: Security Groups (SG)            | Azure: Network Security Groups (NSG)     |
| ---------------------- | ------------------------------------ | ---------------------------------------- |
| Nivel de aplicación    | Instancia EC2 / RDS                  | NICs / Subredes                          |
| Tipo de reglas         | Solo reglas permitidas               | Reglas de permitir y denegar             |
| Estado                 | *Stateful* (respuesta permitida)     | *Stateful* (respuesta permitida)         |
| Dirección del tráfico  | Inbound / Outbound                   | Inbound / Outbound                       |
| Prioridad de reglas    | No aplica                            | Sí (más baja = mayor prioridad)          |
| Asociaciones múltiples | Sí (mismo SG para muchas instancias) | Sí (mismo NSG para muchas NICs/subredes) |
| Evaluación             | Solo allow                           | allow y deny                             |

---

## 🔐 ¿Qué significa “stateful”?

Ambos sistemas recuerdan el estado de las conexiones.
Ejemplo:

* Si permites entrada por el puerto 443 (HTTPS), **la respuesta automáticamente está permitida**.
* No necesitas configurar explícitamente la regla de salida correspondiente.

---

## 🛠 Ejemplo práctico – AWS SG

```hcl
resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  description = "Allow HTTP, HTTPS and SSH"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "Allow HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Allow HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Allow SSH from corporate IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["192.168.10.0/24"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

---

## 🛠 Ejemplo práctico – Azure NSG (Bicep)

```bicep
resource nsg 'Microsoft.Network/networkSecurityGroups@2020-11-01' = {
  name: 'web-nsg'
  location: resourceGroup().location
  properties: {
    securityRules: [
      {
        name: 'Allow_HTTP'
        properties: {
          priority: 100
          direction: 'Inbound'
          access: 'Allow'
          protocol: 'Tcp'
          sourceAddressPrefix: '*'
          sourcePortRange: '*'
          destinationAddressPrefix: '*'
          destinationPortRange: '80'
        }
      }
      {
        name: 'Allow_HTTPS'
        properties: {
          priority: 110
          direction: 'Inbound'
          access: 'Allow'
          protocol: 'Tcp'
          sourceAddressPrefix: '*'
          destinationPortRange: '443'
        }
      }
      {
        name: 'Allow_SSH_Corp'
        properties: {
          priority: 120
          direction: 'Inbound'
          access: 'Allow'
          protocol: 'Tcp'
          sourceAddressPrefix: '192.168.10.0/24'
          destinationPortRange: '22'
        }
      }
    ]
  }
}
```

---

## ⚠ Errores comunes

| Error                                  | Consecuencia                            | Recomendación                                        |
| -------------------------------------- | --------------------------------------- | ---------------------------------------------------- |
| Abrir `0.0.0.0/0` en puertos sensibles | Acceso global desde Internet            | Limitar por rango IP corporativo                     |
| No usar NSG en subredes internas       | Tráfico no auditado entre recursos      | Asociar NSG incluso en entornos internos             |
| Egresos sin restricciones              | Riesgo de exfiltración                  | Restringir puertos salientes en servidores sensibles |
| Reglas NSG mal priorizadas             | Reglas permitidas sobrescritas por deny | Revisar prioridades y pruebas funcionales            |

---

## 🔎 Validación en consola

### En AWS:

1. Ve a **EC2 > Security Groups**.
2. Filtra por VPC y revisa reglas `Inbound` y `Outbound`.
3. Comprueba qué instancias están asociadas a cada SG.

### En Azure:

1. Ve a **Redes > Network Security Groups**.
2. Selecciona el NSG y ve a **Reglas de seguridad**.
3. Ordena por prioridad y revisa rangos y protocolos.

---

## 🧪 Comprobaciones CLI

### AWS

```bash
aws ec2 describe-security-groups \
  --query "SecurityGroups[*].{Name:GroupName,ID:GroupId,Inbound:IpPermissions[*].FromPort}"
```

### Azure

```bash
az network nsg rule list \
  --resource-group MiRG \
  --nsg-name web-nsg \
  --output table
```

---

## ✅ Buenas prácticas

* Siempre **cerrar todo por defecto** y luego abrir lo necesario.
* Usar **rangos de IP específicos**, no `0.0.0.0/0`, excepto para HTTP/HTTPS públicos.
* Asignar **SG/NSG por función**: `web-sg`, `db-sg`, etc.
* Revisar periódicamente reglas de acceso con herramientas de auditoría.
* Aplicar **defensa en profundidad**: SG + NACLs (AWS), NSG + Firewall (Azure).

---

## 📚 Recursos oficiales

* [AWS – Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
* [Azure – NSG overview](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
* [AWS – Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
* [Terraform AWS SG module](https://registry.terraform.io/modules/terraform-aws-modules/security-group/aws/latest)
