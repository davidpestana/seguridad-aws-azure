## 🌐 Módulo 3: Seguridad en la Red (AWS y Azure)

---

## 🎯 Objetivos del módulo

Este módulo tiene como finalidad que el alumno adquiera la capacidad de **diseñar, desplegar y asegurar redes en entornos cloud**, entendiendo cómo se trasladan los principios de la seguridad de red tradicional a la nube y qué nuevas herramientas existen.

Al finalizar el módulo podrás:

* Diseñar arquitecturas de red seguras en AWS (VPC) y Azure (VNet).
* Diferenciar subredes públicas, privadas y de servicios, y aplicar segmentación.
* Configurar y auditar reglas de tráfico entrante/saliente con SG, NACLs y NSG.
* Usar firewalls gestionados, NAT Gateways y endpoints privados para proteger tráfico interno y externo.
* Implementar controles de acceso avanzados integrados con identidades y roles.
* Monitorizar tráfico y establecer alertas frente a anomalías.
* Aplicar buenas prácticas de hardening y defensa en capas para redes cloud.

---

## 🧭 Índice detallado del módulo

---

### 1. Introducción a la seguridad de red en cloud

* ✅ **Objetivo**: Entender cómo ha cambiado la seguridad de red con la nube y por qué ya no existe un perímetro físico único.
* 📌 **Claves**: Perímetro lógico · Redes definidas por software · Zero Trust · Defensa en profundidad.
* 🧩 **Contenido**:
  En la era on-premise, la seguridad dependía de dispositivos físicos (firewalls perimetrales, routers, switches, VLANs). En cloud, todo esto se abstrae en software y queda bajo tu responsabilidad de configuración.

  * **El perímetro ha desaparecido**: cada recurso (máquina, contenedor, función serverless) puede exponerse a internet si no se controla.
  * **La red es programable**: definimos topologías mediante código (Terraform, ARM, CloudFormation). Esto agiliza despliegues, pero también multiplica la posibilidad de errores masivos.
  * **El nuevo enfoque es Zero Trust**: no se confía ni en redes internas ni en conexiones privadas, todo debe autenticarse y auditarse.
    **Ejemplo**: una máquina en una subred mal configurada en Azure puede abrir un túnel a internet y comprometer toda la aplicación. En AWS, un SG con `0.0.0.0/0` habilitado en el puerto 22 deja toda la red expuesta a bots de escaneo en minutos.

📚 [`01-introduccion.md`](modulo3-red/01-introduccion.md)

---

### 2. VPC y subredes en AWS / VNets en Azure

* ✅ **Objetivo**: Comprender cómo se construyen redes privadas virtuales en cada proveedor y cómo segmentarlas de forma segura.
* 📌 **Claves**: VPC · VNet · Subredes públicas/privadas · Tablas de rutas · Address Space.
* 🧩 **Contenido**:

  * **AWS**: una VPC es un rango de direcciones IP privadas dividido en subredes. Las subredes pueden ser públicas (con acceso a Internet Gateway) o privadas (aisladas, con NAT Gateway opcional). Tablas de rutas definen cómo fluye el tráfico.
  * **Azure**: una VNet funciona de forma similar: rango de direcciones IP, subredes asignadas y tablas de rutas. Azure añade flexibilidad con “peering” entre VNets y address spaces superpuestos.
    **Caso práctico**: desplegar una aplicación 3-tier (web, app, db).
  * Web → en subred pública con acceso a internet controlado.
  * App y DB → en subred privada, solo accesibles internamente.
  * Comunicación entre capas restringida por NSG/SG.
    **Error frecuente**: crear todos los recursos en la misma subred y abrir puertos directamente al exterior (muy común en pruebas rápidas).

📚 [`02-vpc-y-subredes.md`](modulo3-red/02-vpc-y-subredes.md)

---

### 3. Network Security Groups (NSG) y Security Groups

* ✅ **Objetivo**: Configurar reglas de red que controlen el tráfico en recursos cloud.
* 📌 **Claves**: Reglas inbound/outbound · Stateful vs Stateless · NACLs.
* 🧩 **Contenido**:

  * **AWS**:

    * **Security Groups (SG)** → aplican a instancias (EC2, RDS, etc.), son *stateful* (si permites tráfico entrante, la respuesta está permitida automáticamente).
    * **Network ACLs (NACLs)** → aplican a nivel de subred, son *stateless* (hay que configurar entrada y salida por separado).
  * **Azure**:

    * **NSG** → conjunto de reglas aplicables a NICs o subredes, con prioridad numérica y soporte de inbound/outbound.
      **Ejemplo comparativo**:
  * En AWS: crear un SG que permita HTTP/HTTPS desde `0.0.0.0/0` y SSH solo desde la IP corporativa.
  * En Azure: crear un NSG con reglas que bloqueen todo el tráfico entrante salvo puertos 80 y 443, y permitan RDP solo desde una subnet corporativa.
    **Error común**: abrir todo el tráfico saliente sin restricciones; esto facilita exfiltración de datos en caso de intrusión.

📚 [`03-nsg-y-security-groups.md`](modulo3-red/03-nsg-y-security-groups.md)

---

### 4. Firewalls, endpoints y NAT gateways

* ✅ **Objetivo**: Usar servicios gestionados para controlar el tráfico y minimizar exposición.
* 📌 **Claves**: Firewalls gestionados · NAT Gateway · Private Endpoints · Bastion Host.
* 🧩 **Contenido**:

  * **AWS**:

    * Firewall Manager para gestionar políticas en varias cuentas.
    * NAT Gateway para dar salida a internet a instancias privadas sin exponerlas.
    * VPC Endpoints para acceso privado a servicios como S3 o DynamoDB sin usar internet.
  * **Azure**:

    * Azure Firewall como servicio centralizado de filtrado y reglas.
    * Private Endpoints para conectar recursos a PaaS internos sin exposición pública.
    * Bastion Host para acceso seguro vía RDP/SSH sin abrir puertos directos.
      **Caso de uso**: una app que necesita acceder a un storage blob de Azure o a un bucket S3 → usar endpoints privados en vez de abrir acceso por IP pública.
      **Error típico**: usar NAT Gateway mal configurado que permite salida indiscriminada a internet sin logging.

📚 [`04-firewalls-y-endpoints.md`](modulo3-red/04-firewalls-y-endpoints.md)

---

### 5. Control de acceso a nivel de red

* ✅ **Objetivo**: Integrar seguridad de red con seguridad de identidad.
* 📌 **Claves**: Microsegmentación · Zero Trust Network Access · Service Endpoints.
* 🧩 **Contenido**:

  * El control no solo es “qué puerto abrir”, sino **quién puede usarlo**.
  * **AWS**: puedes restringir un SG a aceptar tráfico solo desde otro SG (ej. SG de una app → SG de la base de datos).
  * **Azure**: NSG + Service Endpoints permiten que solo un App Service acceda a un SQL Database, aunque compartan VNet.
    **Caso práctico**: garantizar que solo un conjunto de máquinas en subred privada acceda a una base de datos, incluso si hay otros recursos en la misma red.
    Esto reduce el riesgo de **movimiento lateral** en caso de intrusión.

📚 [`05-control-de-acceso.md`](modulo3-red/05-control-de-acceso.md)

---

### 6. Monitorización del tráfico y alertas

* ✅ **Objetivo**: Observar, registrar y analizar el tráfico de red para detectar anomalías.
* 📌 **Claves**: Flow Logs · Análisis de tráfico · Integración con SIEM · Alertas proactivas.
* 🧩 **Contenido**:

  * **AWS**: VPC Flow Logs (registros de conexiones entrantes/salientes) + CloudWatch para alertas.
  * **Azure**: NSG Flow Logs + Network Watcher, con integración en Azure Monitor y Sentinel.
    **Casos de uso**:
  * Detectar intentos de conexión desde IPs no habituales.
  * Ver patrones de escaneo de puertos.
  * Integrar con SIEM (Sentinel, Splunk, ELK) para correlación avanzada.
    **Error común**: habilitar logs pero no revisarlos ni establecer alertas → se acumulan terabytes de datos sin valor práctico.

📚 [`06-monitorizacion-de-red.md`](modulo3-red/06-monitorizacion-de-red.md)

---

### 7. Buenas prácticas en redes cloud

* ✅ **Objetivo**: Establecer un checklist sólido para asegurar redes en AWS y Azure.
* 📌 **Claves**: Mínima exposición · Defense in Depth · Auditoría continua.
* 🧩 **Contenido**:

  * Evitar reglas globales como `0.0.0.0/0` sin justificación.
  * Mantener entornos separados (prod/dev/test) en redes distintas.
  * Revisar periódicamente reglas de SG/NSG y eliminar las obsoletas.
  * Usar automatización (Terraform, Policy, Config) para imponer compliance.
  * Activar logging y retención suficiente para cumplir normativas.
    **Checklist de referencia**: un conjunto de “quick wins” aplicables a cualquier despliegue cloud para reforzar la seguridad de red desde el inicio.

📚 [`07-buenas-practicas.md`](modulo3-red/07-buenas-practicas.md)
