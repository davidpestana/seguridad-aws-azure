# 📄 `02-vpc-y-subredes.md`

## 🧩 VPC y subredes en AWS / VNets en Azure

### 🎯 Objetivo

Comprender cómo se diseñan redes virtuales en AWS y Azure, aplicando segmentación lógica para separar servicios y controlar la exposición de recursos.

---

## 🌐 ¿Qué es una red virtual?

Una **red virtual privada** (VPC en AWS / VNet en Azure) es un entorno lógico y aislado que simula una red física dentro del proveedor cloud. Dentro de estas redes se crean **subredes** (subnets) que segmentan recursos según su función, nivel de exposición o requisitos de seguridad.

---

## 🧠 Comparativa AWS ↔ Azure

| Concepto        | AWS (VPC)                                 | Azure (VNet)                                 |
| --------------- | ----------------------------------------- | -------------------------------------------- |
| Nombre          | Virtual Private Cloud (VPC)               | Virtual Network (VNet)                       |
| Rango IP        | RFC1918 / personalizado                   | RFC1918 / personalizado                      |
| Subredes        | Obligatorias, asociadas a zonas AZ        | Obligatorias, sin AZ específicas por defecto |
| Tablas de rutas | Customizables, una por subred             | Customizables, compartidas entre subredes    |
| Segmentación    | Pública vs privada (por Internet Gateway) | NSG + UDR + subredes                         |
| DNS interno     | Integrado (AmazonProvidedDNS)             | Integrado (Azure-provided name resolution)   |

---

## 🛠 Estructura típica: arquitectura en 3 capas (3-tier)

Una arquitectura segura suele dividir la red en tres capas o zonas:

1. **Web / Pública**: recursos que deben ser accesibles desde internet (ej. Load Balancer).
2. **Aplicación**: lógica de negocio, backend, servicios internos.
3. **Base de datos**: recursos que no deben tener contacto directo con internet.

### 📌 Ejemplo en AWS

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.2.0/24"
}
```

* Subred `10.0.1.0/24`: pública, asociada a una ruta con Internet Gateway.
* Subred `10.0.2.0/24`: privada, sin acceso directo a Internet.

### 📌 Ejemplo en Azure (Bicep)

```bicep
resource vnet 'Microsoft.Network/virtualNetworks@2022-07-01' = {
  name: 'myVNet'
  location: resourceGroup().location
  properties: {
    addressSpace: {
      addressPrefixes: ['10.1.0.0/16']
    }
    subnets: [
      {
        name: 'web-subnet'
        properties: {
          addressPrefix: '10.1.1.0/24'
        }
      }
      {
        name: 'app-subnet'
        properties: {
          addressPrefix: '10.1.2.0/24'
        }
      }
    ]
  }
}
```

* `web-subnet`: se puede asociar a un Application Gateway o firewall con acceso público.
* `app-subnet`: protegida con NSG y sin IP pública.

---

## 🔁 Reglas de conectividad

| Elemento          | AWS                           | Azure                                |
| ----------------- | ----------------------------- | ------------------------------------ |
| Enrutamiento      | Route Tables por subred       | UDR (User Defined Routes) opcionales |
| Acceso a internet | Internet Gateway (IGW)        | NAT Gateway + IP pública opcional    |
| Salida controlada | NAT Gateway / VPC Endpoint    | Azure Firewall / NAT Gateway         |
| Acceso interno    | VPC Peering / Transit Gateway | VNet Peering / Private Link          |

---

## ⚠️ Errores comunes

| Error                                       | Consecuencia                 | Solución                                                |
| ------------------------------------------- | ---------------------------- | ------------------------------------------------------- |
| Crear todos los recursos en la misma subred | Exposición total de la red   | Dividir en subredes y aplicar NSG/SG                    |
| No definir tabla de rutas personalizada     | Tráfico saliente sin control | Definir rutas explícitas y bloquear tráfico innecesario |
| IP públicas asignadas por defecto           | Recursos internos expuestos  | Desactivar `map_public_ip_on_launch` y usar NAT         |
| Usar VNet sin NSG ni firewall               | Sin control de tráfico       | Asociar NSG a cada subred o interfaz                    |

---

## 🔎 Tips para validar en consola

### En AWS:

1. Ve a **VPC > Subnets**: comprueba que las subredes estén en diferentes zonas de disponibilidad (AZs).
2. En **Route Tables**, asegúrate de que solo las subredes públicas tienen ruta `0.0.0.0/0` hacia un Internet Gateway.

### En Azure:

1. Ve a **Virtual Networks > Subnets**: verifica que estén divididas y tengan asociadas sus NSG.
2. Comprueba que las subredes privadas no tengan IP pública ni asociadas a recursos con interfaces expuestas.

---

## 🧪 Validación en CLI

### AWS CLI

```bash
aws ec2 describe-subnets \
  --query "Subnets[*].{SubnetId:SubnetId,CIDR:CidrBlock,Public:MapPublicIpOnLaunch}"
```

### Azure CLI

```bash
az network vnet subnet list \
  --resource-group MiRG \
  --vnet-name myVNet \
  --query "[].{name:name,addressPrefix:addressPrefix}"
```

---

## ✅ Buenas prácticas

* Crea **al menos dos AZs** por VPC para alta disponibilidad.
* Asocia **una tabla de rutas por subred** si necesitas control granular.
* Usa **VPC endpoints** o **Private Endpoints** para acceder a servicios internos sin salir a Internet.
* Nombra claramente cada subred según su función: `web-subnet`, `db-subnet`, etc.
* No dejes direcciones `0.0.0.0/0` abiertas si no es estrictamente necesario.

---

## 📚 Recursos oficiales

* [AWS – VPC Overview](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)
* [Azure – Virtual Network concepts](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)
* [Terraform VPC module](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest)
* [Bicep VNet module](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/virtual-network)
