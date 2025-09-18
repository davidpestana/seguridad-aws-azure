## 🚨 Módulo 7: Gestión de Incidentes y Respuesta (AWS y Azure)

---

## 🎯 Objetivos del módulo

La prevención nunca es suficiente: hay que **detectar, responder y aprender** de los incidentes de seguridad. Este módulo cubre las herramientas nativas de AWS y Azure para detección de amenazas, automatización de respuesta y simulación de ataques.

Al finalizar, serás capaz de:

* Configurar servicios nativos de detección de amenazas (AWS GuardDuty, Azure Defender/Security Center).
* Integrar logs y señales de seguridad en servicios de monitoreo avanzado (AWS Config, Azure Sentinel).
* Diseñar flujos de **respuesta automatizada** a incidentes.
* Simular ataques comunes (phishing, malware, accesos no autorizados) para entrenar la detección.
* Adoptar buenas prácticas de gestión de incidentes y establecer un plan de respuesta.

---

## 🧭 Índice detallado del módulo

---

### 1) Introducción a la gestión de incidentes en la nube

✅ Objetivo
Entender qué es un incidente de seguridad en cloud y por qué el tiempo de detección/respuesta es crítico.

📌 Claves
MTTD (Mean Time to Detect) · MTTR (Mean Time to Respond) · NIST IR lifecycle · Shared responsibility

🧩 Contenido

* Un incidente es cualquier evento que comprometa confidencialidad, integridad o disponibilidad.
* El ciclo según NIST: **Preparación → Detección → Contención → Erradicación → Recuperación → Lecciones aprendidas**.
* En cloud, la **responsabilidad es compartida**: AWS/Azure protegen la infraestructura, tú debes detectar y responder en tus cuentas.
* Ejemplo: una clave IAM expuesta = incidente inmediato, aunque no haya aún explotación.
  📚 docs/modulo7-incidentes/01-introduccion.md

---

### 2) Detección temprana y alertado

✅ Objetivo
Conocer cómo identificar señales tempranas de ataque en AWS y Azure.

📌 Claves
Indicadores de Compromiso (IoC) · Logs de acceso · Alertas proactivas

🧩 Contenido

* Fuentes de señales: logs de autenticación, tráfico de red, cambios de configuración, patrones anómalos.
* **AWS**: integración CloudTrail + CloudWatch + GuardDuty para detectar accesos sospechosos.
* **Azure**: Activity Logs + Azure Monitor + Defender para detectar intentos de ataque.
* Ejemplo: alertar si hay inicios de sesión desde países inesperados o si se crea una policy IAM demasiado permisiva.
  📚 docs/modulo7-incidentes/02-detencion-y-alertado.md

---

### 3) AWS GuardDuty y Azure Security Center/Defender

✅ Objetivo
Configurar servicios nativos de threat detection y comprender sus capacidades.

📌 Claves
Threat Intelligence · Malware detection · IAM misuse · Defender plans

🧩 Contenido

* **AWS GuardDuty**:

  * Analiza logs (CloudTrail, VPC Flow, DNS) para encontrar actividad sospechosa.
  * Detecta accesos desde IPs maliciosas, escalado de privilegios, anomalías de comportamiento.
  * Se integra con Security Hub para gestión centralizada.

* **Azure Security Center (Defender for Cloud)**:

  * Proporciona **Secure Score** de recursos.
  * Detecta configuraciones inseguras y ataques activos.
  * Defender plans para workloads específicos (VMs, DB, Kubernetes, Storage).

Caso práctico: configurar alertas de GuardDuty y Defender y responder a un ataque simulado de minería de criptomonedas.
📚 docs/modulo7-incidentes/03-guardduty-y-security-center.md

---

### 4) Azure Sentinel (SIEM/SOAR)

✅ Objetivo
Usar Azure Sentinel como plataforma SIEM/SOAR para correlación avanzada y respuesta automatizada.

📌 Claves
SIEM · SOAR · KQL queries · Playbooks

🧩 Contenido

* **SIEM**: recopila y correlaciona logs de múltiples fuentes (Azure, AWS, on-prem).
* **SOAR**: automatiza respuesta con Playbooks (Logic Apps).
* **KQL**: consultas para detectar patrones complejos (ej. intentos de fuerza bruta distribuidos).
* **Integración multi-cloud**: Sentinel puede consumir logs de AWS (CloudTrail, GuardDuty).
  Ejemplo: detectar logins fallidos masivos desde distintas IPs y ejecutar un playbook que bloquee la cuenta afectada automáticamente.
  📚 docs/modulo7-incidentes/04-azure-sentinel.md

---

### 5) Respuesta automatizada a incidentes

✅ Objetivo
Diseñar flujos de respuesta automáticos para contener incidentes y reducir MTTR.

📌 Claves
Lambda functions · EventBridge · Logic Apps · Automation Rules

🧩 Contenido

* **AWS**:

  * GuardDuty + EventBridge + Lambda para automatizar respuestas.
  * Ejemplo: al detectar clave comprometida → deshabilitar usuario IAM, notificar por SNS.

* **Azure**:

  * Sentinel Playbooks con Logic Apps → aislamiento de VM, reseteo de credenciales, envío de alertas.
  * Ejemplo: al detectar ataque RDP → apagar VM, crear snapshot, notificar SOC.

Buenas prácticas: respuestas deben ser **reversibles y auditadas** para evitar “auto-sabotajes”.
📚 docs/modulo7-incidentes/05-respuesta-automatizada.md

---

### 6) Simulación de ataques

✅ Objetivo
Entrenar al equipo y validar controles de seguridad mediante simulaciones controladas.

📌 Claves
Red Team · Pen Testing · Simulación de credenciales expuestas · Chaos Engineering

🧩 Contenido

* Simulación de ataques comunes:

  * Uso de credenciales filtradas.
  * Escaneo de puertos internos.
  * Movimiento lateral.
  * Despliegue de malware/minería en instancias.
* **AWS**: simular alertas en GuardDuty (feature “Sample Findings”).
* **Azure**: usar Microsoft Attack Simulator o pruebas de Red Team controladas.
* Valor: medir tiempo de detección y efectividad de playbooks.
  📚 docs/modulo7-incidentes/06-simulacion-de-ataques.md

---

### 7) Buenas prácticas en gestión de incidentes

✅ Objetivo
Adoptar un plan estructurado de respuesta a incidentes que se pueda aplicar en cualquier organización.

📌 Claves
Runbooks · Roles claros · Comunicación · Post-mortem

🧩 Contenido

* Definir **runbooks** claros: qué hacer ante cada tipo de incidente.
* Asignar roles: quién detecta, quién comunica, quién ejecuta.
* Mantener comunicación clara con negocio y clientes.
* Realizar **post-mortems** para aprender de cada incidente.
* Documentar y automatizar lo aprendido en nuevas políticas.
* Normativas: PCI DSS, ISO 27035 exigen un plan de respuesta documentado.

Errores comunes:

* No entrenar al equipo → pánico en incidentes reales.
* Alertas sin responsables → nadie actúa.
* “False positives” ignorados → atacantes pasan desapercibidos.

📚 docs/modulo7-incidentes/07-buenas-practicas.md