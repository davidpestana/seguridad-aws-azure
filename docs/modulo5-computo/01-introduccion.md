# 🧠 Introducción al cómputo seguro en la nube

---

## 🗂️ ¿Qué significa “asegurar” el cómputo en la nube?

En la nube, lanzar una máquina virtual o instancia EC2 puede hacerse en **segundos**. Pero con esa facilidad también llega un gran riesgo: **cada nueva instancia es un vector de exposición potencial** si no está bien configurada desde el inicio.

A diferencia de entornos on-premise, donde las máquinas suelen ser estáticas y duraderas, en la nube:

* Las instancias son **efímeras** y pueden escalar dinámicamente.
* La configuración y seguridad deben ser **automatizadas desde el origen**.
* Se espera que el entorno cumpla con principios de **Zero Trust** y **mínimo privilegio** desde el primer arranque.

---

## 🏗️ Principales diferencias respecto al entorno tradicional

| Elemento                     | On-Premise                                  | Cloud (AWS / Azure)                             |
| ---------------------------- | ------------------------------------------- | ----------------------------------------------- |
| Creación de instancias       | Manual, costosa y lenta                     | Instantánea, vía consola, CLI o IaC             |
| Control de acceso físico     | Muy restringido                             | Inexistente: todo depende de red y credenciales |
| Tiempo de vida de la máquina | Larga duración (“pets”)                     | Corta duración, reemplazables (“cattle”)        |
| Seguridad inicial            | Se configura manualmente tras el despliegue | Debe venir preconfigurada en la imagen base     |
| Rol del sistema operativo    | Control completo                            | A menudo subordinado a roles / políticas de IAM |

---

## 📦 Imágenes base: AMIs y Marketplace

Uno de los principales vectores de riesgo son las **imágenes que usamos para crear instancias**:

| En AWS                       | En Azure                           |
| ---------------------------- | ---------------------------------- |
| AMIs (Amazon Machine Images) | Marketplace Images / Custom Images |

⚠️ Problemas comunes:

* Imágenes con software desactualizado o sin parches.
* AMIs compartidas por terceros sin validación.
* Imágenes comprometidas con puertas traseras (backdoors).
* Permisos amplios concedidos a imágenes con configuraciones inseguras.

🔐 Buenas prácticas:

* Usar imágenes **oficiales del proveedor o de partners verificados**.
* Versionar y validar las imágenes internas.
* Escanear las imágenes en busca de vulnerabilidades (Inspector en AWS, Defender en Azure).
* Usar **“golden images”** como estándar corporativo.

---

## 🧩 ¿Dónde se traslada la seguridad en la nube?

La seguridad deja de depender del “entorno físico” y pasa a gestionarse en:

### 🔑 1. Configuración inicial

* **Usuarios por defecto** (root, admin).
* **Claves SSH** / contraseñas.
* Puertos abiertos en **Security Groups / NSG**.
* Software preinstalado.

### 👤 2. Identidad y acceso

* ¿Quién puede lanzar o detener instancias?
* ¿Qué permisos tiene la instancia?
* ¿Cómo se accede remotamente? (RDP, SSH, Bastion)

### 🛡️ 3. Endurecimiento y monitoreo

* Servicios innecesarios deshabilitados.
* Firewall interno (iptables, ufw, Windows Firewall).
* Logging, syslog, antivirus, IDS.
* Envío de logs a sistemas externos: CloudWatch / Log Analytics.

---

## ⚡ Ejemplo realista: malas prácticas

Una empresa contrata a un pentester para auditar su entorno. Lanza 100 EC2 públicas en 5 minutos con:

* **SSH abierto a todo el mundo (`0.0.0.0/0`)**.
* **Contraseña por defecto** no cambiada.
* Sin patch management ni logs.

📉 Resultado:

* En menos de 10 minutos, múltiples intentos automatizados de login.
* Detección de malware y cryptominers en instancias vulneradas.
* Pérdida de confianza en la imagen base interna.

🔁 Moraleja: en cloud, **“rápido” ≠ “seguro”**.

---

## 🧪 Cómo empezar con buen pie

| Recomendación                     | Cómo hacerlo en AWS                  | Cómo hacerlo en Azure                     |
| --------------------------------- | ------------------------------------ | ----------------------------------------- |
| Usar imágenes endurecidas         | AMIs oficiales + Inspector           | Imágenes validadas Marketplace + Defender |
| Controlar acceso por red          | Security Groups con IPs restringidas | NSG + reglas por subnet/vNet              |
| Usar claves en vez de contraseñas | SSH key pair por instancia           | SSH key injectada en despliegue           |
| Asignar roles/identidades         | IAM Role a EC2                       | Managed Identity a VM                     |
| Automatizar configuración segura  | CloudInit, Ansible, IaC (Terraform)  | Custom Script Extension, IaC              |

---

## 🧠 Conceptos clave a dominar

| Concepto                      | Qué es                                                                                       |
| ----------------------------- | -------------------------------------------------------------------------------------------- |
| **Zero Trust**                | Nunca confiar por defecto; todo acceso debe estar autenticado y autorizado                   |
| **Hardening**                 | El proceso de minimizar la superficie de ataque de un sistema                                |
| **DevSecOps**                 | Integrar seguridad desde el inicio del ciclo de desarrollo / CI/CD                           |
| **Infraestructura inmutable** | No modificar instancias activas; reemplazarlas por nuevas instancias configuradas desde cero |

---

## 🔗 Recursos recomendados

* [AWS EC2 Security Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-best-practices.html)
* [Azure VM Security Overview](https://learn.microsoft.com/en-us/azure/virtual-machines/security-overview)
* [CIS Benchmarks for AWS and Azure](https://www.cisecurity.org/cis-benchmarks)
* [NIST SP 800-123: Guide to General Server Security](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-123.pdf)
