# 📄 `01-introduccion.md`

## 🌐 Introducción a la seguridad de red en entornos cloud (AWS y Azure)

### 🧩 ¿Qué ha cambiado respecto al modelo tradicional?

En los entornos tradicionales (on-premise), la seguridad de red se estructuraba alrededor de **un perímetro físico definido**:

* Cortafuegos (firewalls) físicos en los bordes de la red.
* Segmentación mediante VLANs.
* Control de acceso físico a racks, switches y routers.

Sin embargo, en la nube **ese perímetro ha desaparecido**.

Hoy, cualquier máquina virtual, contenedor o función puede exponerse a internet con una **simple mala configuración de red o permisos**.
En lugar de proteger “el borde” del datacenter, ahora debemos asegurar **cada recurso de forma individual**, bajo el principio de **Zero Trust**.

---

## 🧠 Comparativa conceptual: perímetro físico vs perímetro lógico

| Aspecto                 | Modelo tradicional (on-premise)    | Modelo cloud (AWS / Azure)                       |
| ----------------------- | ---------------------------------- | ------------------------------------------------ |
| Perímetro de red        | Físico (firewall perimetral)       | Lógico (SG/NSG, políticas, roles)                |
| Topología de red        | Estática, cableada                 | Definida por software (SDN)                      |
| Cambios y despliegues   | Manuales                           | Automatizados (Terraform, CloudFormation, etc.)  |
| Visibilidad del tráfico | Limitada, con appliances dedicados | Alta, vía flow logs, SIEM y analítica            |
| Exposición a Internet   | Centralizada, controlada           | Distribuida, propensa a errores de configuración |
| Segmentación interna    | VLANs, ACLs                        | Subredes privadas/públicas, SG/NSG               |

---

## 📌 Conceptos clave en seguridad de red cloud

### 🔐 1. El perímetro es ahora lógico

No existe “la red interna segura” por defecto.
Todo recurso debe ser **aislado, auditado y validado**, incluso si está en la misma red virtual.

Ejemplo:
Un atacante que compromete una sola instancia mal configurada puede moverse lateralmente y escalar privilegios si no se segmentó correctamente la red.

---

### 🛠 2. La red se define como código

Hoy en día, las redes se despliegan y modifican usando herramientas como:

* **AWS**: CloudFormation, CDK, Terraform.
* **Azure**: ARM templates, Bicep, Terraform.

Esto **acelera despliegues** pero también **multiplica el riesgo** si no se revisan correctamente los scripts o se reutilizan plantillas inseguras.

---

### ⚔️ 3. El nuevo enfoque: Zero Trust

Zero Trust asume que **ninguna red es segura**, ni siquiera la interna.
Por tanto, cada conexión debe ser:

* Autenticada (identidad verificada).
* Autorizada (permisos mínimos necesarios).
* Auditada (registro de actividad).

Este enfoque se traduce en prácticas como:

* No abrir puertos innecesarios.
* Controlar el tráfico entre capas (app → db).
* Evitar conexiones salientes abiertas (`0.0.0.0/0`).

---

## 🏢 Caso empresarial – Consecuencias de mala segmentación

La empresa ficticia **FinCloud** desplegó una aplicación en Azure que incluía:

* Frontend (web pública)
* Backend (API)
* Base de datos SQL

Todos los recursos se crearon en la misma VNet y subred, sin NSG restrictivos.

Resultado:

1. Una vulnerabilidad en la web permitió acceso RCE.
2. El atacante exploró la red y accedió directamente a la base de datos.
3. No había logging ni alertas configuradas.

✔️ En una arquitectura segura:

* La base de datos estaría en una subred privada.
* Solo el backend tendría acceso a ella (mediante NSG).
* El acceso desde Internet se restringiría por Azure Firewall.

---

## 💻 Código/configuración: errores comunes

### AWS – Mala práctica

```hcl
resource "aws_security_group" "inseguro" {
  name_prefix = "open-to-world"
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # SSH abierto al mundo
  }
}
```

### Azure – Mala práctica

```json
{
  "name": "AllowAllInbound",
  "properties": {
    "priority": 100,
    "access": "Allow",
    "direction": "Inbound",
    "sourceAddressPrefix": "*",   // cualquier origen
    "destinationPortRange": "*",  // cualquier puerto
    "protocol": "*"
  }
}
```

---

## 🧪 Tips para probar y validar

### En consola web:

* AWS: entra a VPC > Security Groups > revisa reglas abiertas al mundo (`0.0.0.0/0`).
* Azure: entra a NSG > Reglas de entrada > busca reglas amplias con prioridad alta.

### En CLI:

#### AWS CLI

```bash
aws ec2 describe-security-groups \
  --query "SecurityGroups[*].IpPermissions[*].IpRanges[*].CidrIp" \
  --output text | grep 0.0.0.0/0
```

#### Azure CLI

```bash
az network nsg rule list \
  --resource-group mi-grupo \
  --nsg-name mi-nsg \
  --query "[?access=='Allow' && sourceAddressPrefix=='*']"
```

---

## ❌ Errores comunes

| Error                               | Consecuencia                | Solución                   |
| ----------------------------------- | --------------------------- | -------------------------- |
| Subred pública sin control          | Acceso total desde internet | Usar NSG/SG restrictivos   |
| Mismo NSG para todos los recursos   | Segmentación ineficaz       | NSG por capa (web/app/db)  |
| Salida a internet sin restricciones | Exfiltración de datos       | Configurar NAT con logging |
| No activar Flow Logs                | Ceguera operativa           | Activar logs y SIEM        |

---

## ✅ Buenas prácticas clave

* Usa subredes privadas por defecto.
* No abras puertos “por si acaso”.
* Usa Bastion Host o SSM en lugar de SSH directo.
* Revisa y elimina reglas de red no utilizadas.
* Aplica tagging y documentación a recursos de red.
* Automatiza validación con herramientas como **Checkov**, **Prowler**, **Azure Policy**.

---

## 📚 Recursos recomendados

* [AWS Security Best Practices for VPC](https://docs.aws.amazon.com/vpc/latest/userguide/security-best-practices.html)
* [Microsoft – Security considerations for Azure Virtual Network](https://learn.microsoft.com/en-us/azure/virtual-network/security-overview)
* [Zero Trust Architecture – NIST 800-207](https://csrc.nist.gov/publications/detail/sp/800-207/final)
* [Checkov – Infra as Code scanner](https://www.checkov.io/)