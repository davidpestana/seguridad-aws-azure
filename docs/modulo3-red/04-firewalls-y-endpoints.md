# 📄 `04-firewalls-y-endpoints.md`

## 🔥 Firewalls, Endpoints y NAT Gateways

### 🎯 Objetivo

Usar servicios gestionados de red para reforzar el control sobre el tráfico de red saliente y entrante, reducir la exposición pública de recursos y asegurar la conectividad privada entre servicios internos.

---

## 🧠 Fundamentos

En la nube, la protección de red no se limita a reglas básicas. Para entornos sensibles o de producción se requiere:

* **Control fino del tráfico externo** (salida hacia internet).
* **Protección frente a accesos no autorizados** (tanto entrantes como laterales).
* **Evitar exposición pública innecesaria**.
* **Tener logging, inspección y validación de tráfico**.

Servicios como **Azure Firewall**, **AWS NAT Gateway** o **Private Endpoints** permiten implementar una arquitectura de red **defensiva, segmentada y trazable**.

---

## 🧰 Componentes clave en cada cloud

| Función                           | AWS                               | Azure                            |
| --------------------------------- | --------------------------------- | -------------------------------- |
| Salida controlada a Internet      | NAT Gateway                       | NAT Gateway / Firewall           |
| Entrada controlada                | Firewall Manager / SG / WAF       | Azure Firewall / NSG / Azure WAF |
| Conexión privada a servicios PaaS | VPC Endpoints (Gateway/Interface) | Private Endpoint                 |
| Acceso seguro a máquinas          | Bastion Host                      | Azure Bastion                    |

---

## 🛠 Ejemplo práctico 1 – NAT Gateway (AWS)

```hcl
resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public_subnet.id
}

resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }
}
```

🔎 Este NAT Gateway:

* Permite que instancias en subred privada **salgan a internet** (ej. para descargar paquetes).
* **No permite acceso desde fuera** → protege contra escaneos o ataques externos.

---

## 🛠 Ejemplo práctico 2 – Private Endpoint (Azure)

```bicep
resource pe 'Microsoft.Network/privateEndpoints@2021-03-01' = {
  name: 'pe-storage'
  location: resourceGroup().location
  properties: {
    subnet: {
      id: subnet.id
    }
    privateLinkServiceConnections: [
      {
        name: 'conn1'
        properties: {
          privateLinkServiceId: storageAccount.id
          groupIds: ['blob']
        }
      }
    ]
  }
}
```

🔎 Este Private Endpoint:

* Crea una **IP privada local** para acceder a un Storage Account.
* **El tráfico no pasa por internet**.
* Puede restringirse solo a subredes específicas.

---

## 💡 Caso empresarial: aplicación web con backend seguro

**Escenario típico**:

* Frontend accesible vía HTTPS.
* Backend (API + DB) en subred privada.
* Acceso a servicios como S3 o Azure Blob.

🧱 Solución:

* Backend accede a S3/Blob mediante **VPC Endpoint o Private Endpoint**.
* No se expone ninguna IP pública.
* Se activa logging de tráfico (Flow Logs).
* Se accede a servidores solo desde Bastion Host con MFA.

---

## 🧪 Tips para probar

### En AWS

* Accede a la consola VPC → Endpoints → crea uno tipo "Interface" para S3.
* Usa EC2 en subred privada y prueba `aws s3 ls`.
* Verifica que no hay conexión sin el endpoint.

### En Azure

* Crea un Storage Account + Private Endpoint.
* Intenta acceder al blob desde fuera de la VNet → debe fallar.
* Accede desde una VM interna → debe funcionar.

---

## ❌ Errores comunes

| Error                                                  | Consecuencia                          | Solución                                        |
| ------------------------------------------------------ | ------------------------------------- | ----------------------------------------------- |
| No usar NAT Gateway en subred privada                  | Las instancias no pueden actualizarse | Añadir NAT Gateway y asociarlo en route table   |
| Usar endpoints sin restringir acceso                   | Posible acceso lateral no deseado     | Aplicar políticas de acceso a endpoint y subred |
| No activar logs de firewall                            | No hay trazabilidad de conexiones     | Habilitar diagnóstico en Firewall y NSG         |
| Acceder a recursos con IP pública desde redes internas | Exposición innecesaria y más coste    | Usar Private Link / Endpoints internos          |

---

## ✅ Buenas prácticas

* Usar **Private Endpoints** siempre que sea posible para servicios gestionados.
* Configurar **NAT Gateway** solo en subredes públicas y revisar reglas de ruta.
* Auditar accesos a través de **flow logs** y herramientas de análisis.
* Usar **Bastion Host** para acceso a VMs → evita abrir puertos como SSH o RDP.
* Asegurar firewalls con reglas de denegación explícita por defecto.

---

## 🧭 Validación CLI

### AWS – comprobar endpoints

```bash
aws ec2 describe-vpc-endpoints \
  --query "VpcEndpoints[*].{Service:ServiceName,State:State}"
```

### Azure – listar endpoints privados

```bash
az network private-endpoint list \
  --output table
```

---

## 📚 Recursos oficiales

* [AWS – NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
* [AWS – VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/)
* [Azure – Private Endpoint Overview](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview)
* [Azure – Azure Firewall](https://learn.microsoft.com/en-us/azure/firewall/overview)
