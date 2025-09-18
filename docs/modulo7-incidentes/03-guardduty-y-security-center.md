# 🛡️ AWS GuardDuty y Azure Defender: Detección Nativa de Amenazas

---

## 🧭 ¿Qué son y para qué sirven?

Tanto **AWS GuardDuty** como **Microsoft Defender for Cloud** (antes Azure Security Center) son **servicios nativos de detección de amenazas**, diseñados para analizar señales internas y externas y alertar sobre posibles ataques o configuraciones inseguras.

Ambos utilizan:

* **Machine Learning** y **Threat Intelligence** (listas negras, reputación de IPs, etc.).
* Análisis de logs nativos (CloudTrail, VPC Flow Logs, DNS, Activity Logs).
* Integración con servicios de alertado y respuesta automatizada.

---

## ☁️ AWS GuardDuty: análisis inteligente de actividad maliciosa

### ✅ ¿Qué analiza?

GuardDuty revisa de forma continua:

| Fuente            | Qué busca                                             |
| ----------------- | ----------------------------------------------------- |
| **CloudTrail**    | Escalado de privilegios, uso anómalo de APIs.         |
| **VPC Flow Logs** | Comunicaciones con IPs maliciosas o tráfico inusual.  |
| **DNS Logs**      | Resoluciones a dominios usados en malware o phishing. |

### 🧠 ¿Qué tipo de amenazas detecta?

Algunos ejemplos:

* **Uso de claves comprometidas**.
* **Comandos inusuales en EC2**.
* **Activación de instancias desde regiones sospechosas**.
* **Conexión con servidores de C\&C (Command and Control)**.

### 📦 Integración con Security Hub

GuardDuty puede integrarse con **AWS Security Hub**, lo que permite **centralizar findings** de múltiples servicios de seguridad (GuardDuty, Inspector, Macie, etc.).

### 🧪 Activar GuardDuty vía CLI

```bash
aws guardduty create-detector --enable
```

Ver findings:

```bash
aws guardduty list-findings
```

Simular findings:

```bash
aws guardduty create-sample-findings
```

---

## ☁️ Azure Defender (Security Center): evaluación + protección activa

### ✅ ¿Qué hace Defender?

Defender for Cloud combina:

| Funcionalidad       | Descripción                                            |
| ------------------- | ------------------------------------------------------ |
| **Secure Score**    | Evalúa el nivel de protección de tus recursos.         |
| **Recomendaciones** | Detecta configuraciones inseguras.                     |
| **Alertas**         | Lanza avisos en tiempo real ante actividad sospechosa. |
| **Planes Defender** | Protección específica por tipo de workload.            |

### 🧠 Tipos de amenazas que detecta

* Inicios de sesión desde ubicaciones sospechosas.
* Creación de roles con permisos excesivos.
* Malware en máquinas virtuales (con agente instalado).
* Acceso anómalo a bases de datos o almacenamiento.

### 🔧 Planes Defender (por recurso)

| Recurso            | Plan específico         |
| ------------------ | ----------------------- |
| Máquinas virtuales | Defender for Servers    |
| Bases de datos     | Defender for SQL        |
| Contenedores       | Defender for Kubernetes |
| Almacenamiento     | Defender for Storage    |

### 🧪 Activar Defender via CLI

```bash
az security pricing create \
  --name VirtualMachines \
  --tier Standard
```

Ver estado de Defender:

```bash
az security pricing list
```

---

## 🧑‍💼 Caso empresarial: ataque de minería en EC2/VM

### 🧨 Escenario

Una instancia EC2 (o VM de Azure) ha sido comprometida y se está usando para **minar criptomonedas**.

| Paso          | AWS                                                                | Azure                                                    |
| ------------- | ------------------------------------------------------------------ | -------------------------------------------------------- |
| 1️⃣ Detección | GuardDuty lanza un finding como `CryptoCurrency:EC2/BitcoinTool.B` | Defender detecta uso de CPU anómalo y proceso sospechoso |
| 2️⃣ Acción    | El finding se envía a Security Hub + EventBridge                   | Defender lanza una alerta crítica con recomendaciones    |
| 3️⃣ Respuesta | Lambda desactiva la instancia automáticamente                      | Logic App aísla la VM y notifica al SOC                  |

---

## 🧪 Validaciones en consola

| Acción                | AWS                            | Azure                                            |
| --------------------- | ------------------------------ | ------------------------------------------------ |
| Ver resultados        | GuardDuty > Findings           | Defender for Cloud > Alerts                      |
| Simular detecciones   | `create-sample-findings`       | Uso de Microsoft Attack Simulator (más limitado) |
| Revisar configuración | GuardDuty Settings > Detectors | Defender > Environment Settings                  |

---

## ⚠️ Errores comunes

* ❌ Activar GuardDuty o Defender pero **no conectar acciones posteriores** (SNS, Playbooks, etc.).
* ❌ Ignorar findings de tipo "bajo" cuando son parte de un patrón.
* ❌ No desplegar el agente Defender en VMs, lo que limita detección profunda.
* ❌ Desactivar alertas para reducir “ruido”, en vez de afinarlas correctamente.

---

## ✅ Buenas prácticas

✔ Activar detección en **todas las regiones** (por defecto solo se activa en una).
✔ Reforzar findings con eventos correlacionados desde otros servicios (CloudTrail + Inspector).
✔ Usar Security Hub (AWS) o Azure Monitor/Sentinel para **gestión centralizada de alertas**.
✔ Crear alertas custom para findings críticos + integración con Slack/Teams/SOC.

---

## 📚 Recursos recomendados

### AWS

* [AWS GuardDuty Overview](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)
* [Security Hub Integrations](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-integrations.html)

### Azure

* [Microsoft Defender for Cloud Overview](https://learn.microsoft.com/en-us/azure/defender-for-cloud/)
* [Defender for Servers Plan](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-servers-introduction)

