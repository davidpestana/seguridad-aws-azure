# 📄 `05-control-de-acceso.md`

## 🛡️ Control de Acceso a Nivel de Red

### 🎯 Objetivo

Aprender a **integrar la seguridad de red con la gestión de identidades y permisos**, aplicando políticas que restrinjan no solo el tráfico por IP o puerto, sino también **quién puede acceder** a qué, desde dónde y bajo qué condiciones.

---

## 🧠 Fundamentos

La seguridad de red moderna no se basa únicamente en reglas estáticas (ej. “permitir puerto 443”), sino que combina **identidad + red + contexto**:

* ¿Quién intenta conectarse?
* ¿Desde dónde?
* ¿A qué servicio y en qué momento?

Este enfoque está alineado con el modelo **Zero Trust**: *no se confía en nadie por defecto*, incluso si ya está dentro de la red privada.

---

## 🔄 Comparativa AWS ↔ Azure

| Funcionalidad                         | AWS                                          | Azure                                               |
| ------------------------------------- | -------------------------------------------- | --------------------------------------------------- |
| Filtrado por grupos de seguridad      | Security Group a Security Group (SG ↔ SG)    | NSG con restricciones a subredes o NICs específicas |
| Aislamiento de recursos               | SGs aplicados a instancias EC2, RDS, Lambda  | NSG por subred o por NIC de VM                      |
| Vinculación a identidades específicas | IAM Roles asociados a recursos de red        | Azure RBAC + NSG + Service Endpoints                |
| Control cruzado entre servicios       | Ej. permitir solo acceso entre SGs concretos | Ej. restringir acceso a SQL solo desde App Service  |
| Service Endpoints / Private Link      | VPC Endpoints para servicios gestionados     | Private Endpoints con acceso restringido a VNets    |

---

## 🧱 Caso práctico – acceso de microservicios a una base de datos

**Escenario:**
Una aplicación está compuesta por dos microservicios (`auth`, `orders`) y una base de datos (`db`). Todos están en la misma VNet/VPC, pero:

* Solo `auth` y `orders` deben comunicarse con `db`.
* Otros servicios no deben acceder a `db`, incluso si están en la misma red.

### 🛠 En AWS

```hcl
# Crear SG para la base de datos
resource "aws_security_group" "sg_db" {
  name        = "sg-db"
  description = "Permitir tráfico solo desde SGs de servicios autorizados"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [
      aws_security_group.sg_auth.id,
      aws_security_group.sg_orders.id
    ]
  }
}
```

✅ **Resultado:** Solo instancias con el SG `auth` o `orders` pueden acceder a la DB por el puerto 5432.

---

### 🛠 En Azure

```bicep
resource nsg_db 'Microsoft.Network/networkSecurityGroups@2020-06-01' = {
  name: 'nsg-db'
  location: resourceGroup().location
  properties: {
    securityRules: [
      {
        name: 'allow-auth-orders'
        properties: {
          priority: 100
          direction: 'Inbound'
          access: 'Allow'
          protocol: 'Tcp'
          sourceAddressPrefix: '10.0.1.0/24'  // Subred de auth
          sourcePortRange: '*'
          destinationAddressPrefix: '*'
          destinationPortRange: '1433'
        }
      }
    ]
  }
}
```

✅ **Resultado:** Solo las IPs de la subred `auth` (por ejemplo, `10.0.1.0/24`) podrán conectar con el puerto 1433 (SQL Server).

---

## 🧪 Tips para probar

### AWS

* Aplica un SG a la base de datos que solo permita acceso desde un SG específico.
* Lanza una instancia EC2 con otro SG → intenta conectar (debe fallar).
* Cambia el SG por uno autorizado → debe funcionar.

### Azure

* Aplica un NSG restrictivo a la subred de la base de datos.
* Intenta acceder desde distintas subredes o VM con/sin rol asignado.
* Usa `Network Watcher` para verificar rutas y seguridad efectiva.

---

## ❌ Errores comunes

| Error                                                 | Consecuencia                                           | Solución recomendada                                |
| ----------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------------- |
| Permitir acceso desde `0.0.0.0/0` a puertos sensibles | Exposición total de servicios críticos                 | Usar rangos internos o SGs con referencia explícita |
| No aislar recursos por SG/NSG                         | Movimiento lateral posible si se compromete un recurso | Aplicar microsegmentación con grupos de seguridad   |
| No validar tráfico cruzado entre entornos             | Un entorno de pruebas podría acceder a producción      | Separar redes + reglas de acceso estrictas          |
| Usar IPs estáticas sin identificar origen real        | Difícil de auditar, cambiar o revocar                  | Usar SG ↔ SG o roles con identidades seguras        |

---

## ✅ Buenas prácticas

* Implementar **microsegmentación** real: no todos los recursos de la misma red deben comunicarse entre sí.
* Usar **referencias de SG a SG** (AWS) o **NSG + subredes bien definidas** (Azure).
* Integrar el control de red con el control de identidad y roles (IAM/RBAC).
* **Separar entornos** (dev, test, prod) incluso a nivel de red.
* Aplicar principios de **mínimo acceso efectivo** también en la red.

---

## 🔍 Validación en CLI

### AWS – inspeccionar reglas de SG

```bash
aws ec2 describe-security-groups \
  --group-ids sg-xxxxxxxx \
  --query 'SecurityGroups[0].IpPermissions'
```

### Azure – mostrar reglas efectivas en NSG

```bash
az network nic list-effective-nsg \
  --name <nombre-nic> \
  --resource-group <nombre-rg>
```

---

## 📚 Recursos oficiales

* [AWS – Control access to resources using security groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
* [Azure – NSG Overview](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
* [AWS – VPC Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-best-practices.html)
* [Azure – Zero Trust Security Model](https://learn.microsoft.com/en-us/security/zero-trust/)