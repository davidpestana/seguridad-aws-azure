## 🖥️ Módulo 5: Seguridad en Servicios de Cómputo (AWS y Azure)

---

## 🎯 Objetivos del módulo

En este módulo aprenderás a **lanzar, configurar y proteger instancias de cómputo** en la nube, ya sean **EC2 en AWS** o **Máquinas Virtuales en Azure**. Se abordarán desde la creación inicial, el endurecimiento del sistema operativo (hardening), el acceso remoto seguro (SSH/RDP), hasta la gestión de roles y actualizaciones.

Al finalizar, serás capaz de:

* Lanzar instancias EC2 y VMs de Azure con configuraciones seguras por defecto.
* Gestionar **claves SSH, contraseñas, certificados y credenciales temporales**.
* Aplicar **roles, políticas e identidades administradas** en lugar de credenciales estáticas.
* Implementar **hardening y configuración segura** de sistemas operativos y aplicaciones.
* Planificar **actualizaciones y parches** automáticos.
* Detectar y evitar errores frecuentes en el despliegue de máquinas en cloud.

---

## 🧭 Índice detallado del módulo

---

### 1) Introducción al cómputo seguro en la nube

✅ Objetivo
Entender qué significa “asegurar” instancias de cómputo y cómo difiere respecto al mundo on-premise.

📌 Claves
Elasticidad · Imágenes base (AMIs/Marketplace) · Infraestructura efímera · Zero Trust · DevSecOps

🧩 Contenido

* En la nube, lanzar una VM es trivial, pero **el riesgo también escala rápido**.
* El ciclo de vida cambia: instancias efímeras, autoescalado, infraestructura como código.
* **Imágenes base** (AMIs en AWS, imágenes del Marketplace en Azure) → riesgo si no se controlan (ej. imágenes con backdoors).
* La seguridad se traslada a:

  * Configuración inicial (usuario root/admin, claves SSH, contraseñas).
  * Gestión de identidades (roles, identidades administradas).
  * Endurecimiento del sistema y parcheo continuo.

**Ejemplo**: un pentester lanza 100 instancias EC2 sin restricciones y en 5 minutos ya son atacadas vía bots automatizados (SSH/HTTP).

📚 docs/modulo5-computo/01-introduccion.md

---

### 2) EC2 en AWS y Máquinas Virtuales en Azure

✅ Objetivo
Comparar cómo se crean y configuran las instancias en AWS y Azure, con énfasis en opciones de seguridad.

📌 Claves
EC2 vs VM · Tipos de instancia · AMIs · Marketplace · Discos · Redes asociadas

🧩 Contenido

* **AWS EC2**:

  * Elección de AMI (Amazon Linux, Windows, Marketplace).
  * Asociar a una VPC y Security Group.
  * Almacenamiento: volúmenes EBS cifrados por defecto.
  * Roles de IAM para permisos de la instancia.

* **Azure VMs**:

  * Elección de imagen (Ubuntu, Windows Server, Marketplace).
  * Asociar a una VNet y NSG.
  * Discos gestionados (cifrados por defecto).
  * Identidades administradas para acceder a otros servicios.

**Caso práctico**: lanzar un servidor web en cada plataforma, conectarlo a la red, habilitar acceso HTTP/HTTPS, pero protegerlo de accesos SSH/RDP globales.

📚 docs/modulo5-computo/02-ec2-y-maquinas-virtuales.md

---

### 3) Claves, contraseñas y autenticación

✅ Objetivo
Asegurar el acceso a las instancias, evitando credenciales débiles o expuestas.

📌 Claves
SSH keys · Contraseñas seguras · Certificados · MFA en salto · Vaults

🧩 Contenido

* **AWS**:

  * Acceso por **pares de claves SSH** al crear instancias.
  * Almacenamiento seguro en **AWS Secrets Manager** o **SSM Parameter Store**.
  * No reutilizar claves en múltiples instancias.

* **Azure**:

  * Acceso con claves SSH o contraseñas (se recomienda SSH).
  * Integración con **Azure Key Vault** para gestionar secretos.
  * Soporte de login con identidad de Azure AD (en Windows/Linux).

**Buenas prácticas**:

* Deshabilitar usuario root/admin para acceso directo.
* Forzar certificados en lugar de contraseñas.
* Acceso remoto solo a través de jump hosts/Bastion.

📚 docs/modulo5-computo/03-claves-y-autenticacion.md

---

### 4) Roles, políticas e identidades administradas

✅ Objetivo
Eliminar credenciales incrustadas y reemplazarlas por identidades administradas y roles con permisos mínimos.

📌 Claves
IAM Roles · Instance Profiles · Managed Identity (Azure) · Principio de mínimo privilegio

🧩 Contenido

* **AWS**: asignar un **IAM Role** a la instancia, de modo que obtenga credenciales temporales vía STS para acceder a servicios (ej. S3, DynamoDB).
* **Azure**: usar **Managed Identity** (system-assigned o user-assigned) para que la VM acceda a recursos (ej. Storage Account, Key Vault) sin credenciales en código.
* Beneficio: **rotación automática**, menos superficie de ataque.
* Error frecuente: incrustar **keys en archivos de configuración** o código fuente.

📚 docs/modulo5-computo/04-roles-y-politicas.md

---

### 5) Hardening y configuración segura

✅ Objetivo
Aplicar endurecimiento del sistema operativo y servicios para minimizar riesgos.

📌 Claves
CIS Benchmarks · Deshabilitar servicios innecesarios · Firewalls internos · Logs y auditoría

🧩 Contenido

* Usar imágenes endurecidas (CIS AMIs, Azure Marketplace hardened images).
* Deshabilitar puertos y servicios que no se usan.
* Configurar firewalls internos (iptables, ufw, Windows Firewall).
* Habilitar logs del sistema y enviarlos a CloudWatch/Log Analytics.
* Separar usuarios y usar sudo en vez de root directo.

Ejemplo: aplicar un script de hardening inicial que deshabilite root login por SSH, fuerce cifrado fuerte y configure logging remoto.

📚 docs/modulo5-computo/05-hardening-y-configuracion.md

---

### 6) Actualizaciones y parcheo

✅ Objetivo
Mantener instancias seguras con actualizaciones automáticas y pruebas de compatibilidad.

📌 Claves
Patch Management · AWS SSM · Azure Update Management · Golden Images

🧩 Contenido

* **AWS**: Systems Manager (SSM) Patch Manager → parches programados por grupos de instancias.
* **Azure**: Update Management integrado en Automation → control de parches de VMs.
* Estrategia: crear **golden images** actualizadas y lanzar nuevas instancias en lugar de parchear manualmente.
* Evitar instancias “pet” sin gestión → usar infraestructura inmutable.

📚 docs/modulo5-computo/06-actualizaciones-y-parcheo.md

---

### 7) Buenas prácticas en cómputo seguro

✅ Objetivo
Consolidar un conjunto de prácticas mínimas que aseguren instancias en producción.

📌 Claves
Principio de mínimo privilegio · Acceso restringido · Logging centralizado · Automatización

🧩 Contenido

* Usar roles/identidades administradas en lugar de credenciales estáticas.
* Limitar acceso SSH/RDP a IPs concretas (no `0.0.0.0/0`).
* Usar Bastion Hosts/Azure Bastion para accesos administrativos.
* Configurar backups de discos y probar recuperación.
* Aplicar CIS Benchmarks y escáneres de vulnerabilidad (Inspector en AWS, Defender en Azure).
* Automatizar el ciclo de vida: lanzar, configurar y destruir con IaC.

📚 docs/modulo5-computo/07-buenas-practicas.md