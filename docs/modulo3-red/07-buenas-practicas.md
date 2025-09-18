# 📄 `07-buenas-practicas.md`

## 🛡️ Buenas prácticas en redes cloud (AWS y Azure)

### 🎯 Objetivo

Aplicar un conjunto sólido y accionable de **buenas prácticas** que refuercen la seguridad de red en arquitecturas cloud desde el primer despliegue, minimizando la exposición y aumentando la capacidad de auditoría y defensa.

---

## 🧠 Fundamentos

Una red mal segmentada o expuesta innecesariamente sigue siendo una de las causas más frecuentes de brechas de seguridad. Estas buenas prácticas son el resultado de años de incidentes reales, auditorías y recomendaciones oficiales de AWS y Azure.

---

## ✅ Checklist de buenas prácticas

| Categoría                   | Buenas prácticas clave                                                                                                                                                   |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Segmentación**            | - Divide recursos en subredes públicas y privadas según necesidad real.<br>- Usa VNets/VPCs diferentes por entorno (dev, prod, test).                                    |
| **Reglas de acceso**        | - Evita `0.0.0.0/0` para acceso a puertos administrativos (SSH, RDP).<br>- Usa SG/NSG restrictivos por defecto ("deny all").                                             |
| **Hardening de exposición** | - Usa NAT Gateways para salida a internet desde subredes privadas.<br>- Implementa Private Endpoints o VPC Endpoints para recursos PaaS.                                 |
| **Control de identidad**    | - Integra control de red con identidades: SGs en AWS o RBAC + NSG en Azure.<br>- Usa reglas de acceso que permitan tráfico solo entre identidades conocidas.             |
| **Monitorización**          | - Activa Flow Logs desde el inicio.<br>- Configura alertas para conexiones inusuales o cambios de reglas.<br>- Usa soluciones SIEM si es posible.                        |
| **Automatización**          | - Define la topología de red con Terraform, CloudFormation o ARM.<br>- Usa plantillas versionadas con validaciones.<br>- Impón compliance con AWS Config o Azure Policy. |
| **Auditoría continua**      | - Revisa SG/NSG y reglas de routing al menos una vez al mes.<br>- Elimina reglas no utilizadas o innecesarias.<br>- Mantén una política de revisión de logs.             |

---

## 🧪 Caso empresarial – Red segura para eCommerce

**Escenario**: Una empresa despliega un sitio de comercio electrónico con front-end público, lógica de negocio interna y base de datos en entorno cloud multi-región.

**Mal diseño**:

* Todos los recursos están en una única subred pública.
* El puerto 22 abierto desde cualquier IP.
* No hay monitorización activa ni Flow Logs.

**Rediseño seguro**:

* VPC/VNet con subred pública para balanceadores y subredes privadas para app y BD.
* NAT Gateway para salida desde las subredes privadas.
* SGs que solo permiten tráfico desde otras SGs específicas (app ↔ BD).
* NSG con reglas explícitas y denegación por defecto.
* Flow Logs habilitados + alertas con CloudWatch / Azure Monitor.
* Acceso SSH restringido mediante Bastion Host + IPs autorizadas.

---

## 💻 Código de referencia – Terraform fragmentos

### AWS – Subred pública y SG seguro

```hcl
resource "aws_security_group" "web_sg" {
  name        = "web-sg"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["203.0.113.0/24"] # solo IP empresa
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Azure – NSG con reglas explícitas

```json
{
  "name": "nsg-web",
  "properties": {
    "securityRules": [
      {
        "name": "Allow_HTTP",
        "properties": {
          "priority": 100,
          "direction": "Inbound",
          "access": "Allow",
          "protocol": "Tcp",
          "sourcePortRange": "*",
          "destinationPortRange": "80",
          "sourceAddressPrefix": "*",
          "destinationAddressPrefix": "*"
        }
      },
      {
        "name": "Deny_All_Others",
        "properties": {
          "priority": 4096,
          "direction": "Inbound",
          "access": "Deny",
          "protocol": "*",
          "sourcePortRange": "*",
          "destinationPortRange": "*",
          "sourceAddressPrefix": "*",
          "destinationAddressPrefix": "*"
        }
      }
    ]
  }
}
```

---

## ❌ Errores frecuentes a evitar

| Error                                                | Consecuencia                                     |
| ---------------------------------------------------- | ------------------------------------------------ |
| Dejar puertos abiertos a cualquier IP (ej. SSH, RDP) | Ataques de fuerza bruta, secuestro de sesión     |
| No separar redes por entorno (dev/prod)              | Riesgo de contaminación cruzada y fuga de datos  |
| No revisar reglas SG/NSG en meses                    | Reglas obsoletas que permiten acceso innecesario |
| Usar IP públicas innecesarias                        | Aumenta superficie de ataque                     |
| Desplegar sin automatización ni validación           | Inconsistencias de seguridad entre entornos      |
| No eliminar reglas “temporales” después de pruebas   | Puertas traseras inadvertidas                    |

---

## 📊 Integración con cumplimiento y auditoría

* AWS:

  * Usa **AWS Config** para verificar que no haya reglas abiertas al mundo.
  * Crea **Conformance Packs** para evaluar tu postura de seguridad de red.

* Azure:

  * Usa **Azure Policy** para imponer restricciones como “no permitir NSG con `Allow *`”.
  * Activa **Azure Defender for Network** para recibir recomendaciones automáticas.

---

## 📚 Recursos oficiales

* [AWS – Best practices for VPC security](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html)
* [Azure – Network security best practices](https://learn.microsoft.com/en-us/azure/security/fundamentals/network-best-practices)
* [AWS – AWS Config conformance packs](https://docs.aws.amazon.com/config/latest/developerguide/conformance-packs.html)
* [Azure – Azure Policy samples](https://github.com/Azure/azure-policy)

