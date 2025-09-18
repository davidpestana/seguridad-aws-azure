## 📘 Módulo 1: Introducción a la Seguridad en la Nube (AWS y Azure)

---

## 🎯 Objetivos del módulo

Este módulo establece los fundamentos sobre los que construiremos todo el enfoque de seguridad cloud a lo largo del curso. Al completarlo, serás capaz de:

* Comprender las diferencias clave entre la seguridad en la nube y en entornos tradicionales.
* Identificar los principios fundamentales de la seguridad cloud y cómo se aplican en la práctica.
* Entender el modelo de responsabilidad compartida en AWS y Azure y sus implicaciones reales.
* Reconocer y ubicar los servicios de seguridad nativos en ambas plataformas.
* Adoptar buenas prácticas desde el primer despliegue para reducir riesgos comunes.

---

## 🧭 Índice detallado del módulo

---

### 1. ¿Qué es la seguridad en la nube?

✅ **Objetivo**
Distinguir los conceptos fundamentales de seguridad en entornos cloud frente a entornos tradicionales on-premise. Comprender cómo la naturaleza de los riesgos y los mecanismos de protección se transforman al operar en la nube.

📌 **Claves**
Cambio de paradigma · Superficies de ataque lógicas · Configuración como amenaza · Elasticidad y automatización

🧩 **Contenido**
La seguridad cloud no es una simple extensión de la ciberseguridad clásica. En un entorno local, la seguridad se basa en perímetros físicos, firewalls de red, segmentación LAN y control físico del hardware. En la nube, el perímetro se disuelve: la seguridad depende de configuraciones, políticas, identidad y automatización.

Veremos cómo:

* La **configuración incorrecta** se convierte en el vector de ataque más frecuente.
* La **exposición accidental** de datos y servicios (ej. buckets S3 públicos o bases SQL sin firewall) puede escalar rápidamente por la conectividad global.
* La **gestión de identidades** se convierte en el nuevo “firewall” de la arquitectura cloud.

Ejemplos y noticias recientes demostrarán cómo pequeñas decisiones de configuración han llevado a brechas masivas, y por qué entender este nuevo terreno es la base para diseñar cualquier solución segura.

📚 [`01-que-es-seguridad-cloud.md`](modulo1-intro/01-que-es-seguridad-cloud.md)

---

### 2. Principios fundamentales

✅ **Objetivo**
Aplicar los principios clásicos de la seguridad informática a entornos en la nube, entendiendo cómo se adaptan al nuevo contexto distribuido, automatizado y altamente expuesto.

📌 **Claves**
CIA (Confidencialidad, Integridad, Disponibilidad) · Trazabilidad · Resiliencia · Seguridad como proceso continuo

🧩 **Contenido**
Los principios que sustentan la seguridad informática siguen siendo válidos en la nube, pero deben reinterpretarse:

* **Confidencialidad**: control de acceso basado en roles, cifrado en tránsito y en reposo, segregación lógica de datos.
* **Integridad**: validación de datos, control de versiones, protección contra modificaciones no autorizadas.
* **Disponibilidad**: tolerancia a fallos, escalabilidad automática, diseño multi-zona.

A esto se suman principios especialmente importantes en cloud:

* **Trazabilidad**: gracias a logs y auditorías centralizadas (CloudTrail, Azure Monitor).
* **Resiliencia**: capacidad de recuperación ante fallos o ataques, apoyada por la arquitectura elástica.

Estos principios se materializan en herramientas y prácticas concretas que se aplicarán en todo el curso.

📚 [`02-principios-fundamentales.md`](modulo1-intro/02-principios-fundamentales.md)

---

### 3. Modelo de responsabilidad compartida

✅ **Objetivo**
Comprender qué aspectos de la seguridad son responsabilidad del cliente y cuáles del proveedor cloud, dependiendo del tipo de servicio (IaaS, PaaS, SaaS).

📌 **Claves**
Separación de responsabilidades · Error humano · Modelo dinámico · Confusión común

🧩 **Contenido**
Uno de los errores más comunes en entornos cloud es asumir que el proveedor se encarga de todo. Este módulo expone con claridad quién es responsable de qué, y cómo varía esa división en función del modelo de servicio:

* **IaaS**: tú gestionas el sistema operativo, las actualizaciones, la configuración de red, el firewall lógico, etc.
* **PaaS**: tú gestionas la lógica de negocio, las claves, los usuarios, y el código que subes.
* **SaaS**: tú gestionas los usuarios, los datos, las configuraciones del producto.

Verás ejemplos de cómo clientes han fallado en su parte de la responsabilidad por desconocimiento o malentendidos, y cómo evitarlo con buenas prácticas y referencias visuales del modelo (matrices AWS y Azure).

📚 [`03-modelo-responsabilidad-compartida.md`](modulo1-intro/03-modelo-responsabilidad-compartida.md)

---

### 4. Servicios básicos de seguridad (AWS/Azure)

✅ **Objetivo**
Identificar los servicios clave integrados en AWS y Azure que permiten implementar seguridad desde el diseño, con foco en gestión de identidades, auditoría y protección activa.

📌 **Claves**
IAM · Monitorización · Configuración auditada · Detección de amenazas · Políticas y cumplimiento

🧩 **Contenido**
Tanto AWS como Azure ofrecen servicios específicos para construir arquitecturas seguras sin depender de herramientas externas:

* **Identidad y acceso**: IAM (AWS), Azure AD, políticas de acceso.
* **Auditoría**: CloudTrail, Azure Activity Logs.
* **Protección activa**: GuardDuty, Azure Defender.
* **Cumplimiento**: AWS Config, Azure Policy, Blueprints.

No solo aprenderás a identificar estos servicios, sino a entender **cuándo y cómo activarlos**, y qué errores comunes los inutilizan (por ejemplo, tener logging habilitado pero sin destino ni alertas).

📚 [`04-servicios-basicos.md`](modulo1-intro/04-servicios-basicos.md)

---

### 5. Buenas prácticas iniciales

✅ **Objetivo**
Adoptar desde el primer momento un conjunto de prácticas esenciales que previenen la mayoría de errores y mejoran radicalmente la postura de seguridad general.

📌 **Claves**
MFA · Mínimo privilegio · Cuentas separadas · Gestión de claves · Control de cambios

🧩 **Contenido**
No necesitas conocimientos avanzados para mejorar significativamente tu seguridad cloud. Aquí sentaremos una serie de prácticas que aplican a todo entorno nuevo:

* Forzar **MFA** para todos los usuarios con permisos sensibles.
* Aplicar el principio de **mínimo privilegio** desde la creación de roles y grupos.
* Separar cuentas por entorno: desarrollo, producción, pruebas.
* Limitar y rotar **claves de acceso** a API y usuarios IAM.
* Supervisar cambios con alertas automatizadas.

Estas buenas prácticas formarán parte de los checklists recurrentes a lo largo del curso.

📚 [`05-buenas-practicas.md`](modulo1-intro/05-buenas-practicas.md)

---

### 6. Recursos oficiales y lecturas recomendadas

✅ **Objetivo**
Conocer fuentes fiables de referencia para continuar aprendiendo, respaldar decisiones y mantenerte actualizado en seguridad cloud.

📌 **Claves**
Documentación oficial · Benchmarks · Whitepapers · Normativas y marcos

🧩 **Contenido**
Incluiremos enlaces clasificados y comentados a:

* AWS Security Docs, Azure Security Fundamentals
* [CIS Benchmarks](https://www.cisecurity.org/benchmarks)
* [AWS Well-Architected Framework – Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)
* NIST, ISO 27001, Microsoft CAF

Este repositorio de recursos será útil no solo durante el curso, sino también en entornos laborales o certificaciones oficiales.

📚 [`06-recursos-oficiales.md`](modulo1-intro/06-recursos-oficiales.md)

---

### 7. Glosario de términos clave

✅ **Objetivo**
Consolidar la comprensión del vocabulario técnico y funcional más frecuente en el ámbito de la seguridad cloud, con equivalencias entre AWS y Azure.

📌 **Claves**
Terminología base · Acrónimos · Traducción cruzada · Conceptos de arquitectura

🧩 **Contenido**
Desde conceptos básicos (IAM, MFA, Zero Trust) hasta términos avanzados (Postura de seguridad, DevSecOps), este glosario funcionará como guía de referencia rápida durante el curso.

* Incluye equivalencias terminológicas entre proveedores.
* Organizado por temas: identidad, red, auditoría, compliance.
* Incluye ejemplos de uso para cada término en contexto real.

📚 [`07-glosario.md`](modulo1-intro/07-glosario.md)
