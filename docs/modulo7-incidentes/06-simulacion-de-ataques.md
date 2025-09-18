# 🧨 Simulación de Ataques en Entornos Cloud

---

## 🧭 ¿Por qué simular ataques?

Simular ataques reales permite:

* Validar que tus sistemas de detección y alertado **realmente funcionan**.
* Entrenar a los equipos de seguridad y operaciones para responder bajo presión.
* Evaluar la cobertura de los controles actuales.
* Medir métricas clave como el **MTTD/MTTR** en la práctica.

⚠️ **Importante**: estas pruebas deben hacerse de forma **controlada, autorizada y documentada**, preferiblemente en entornos no productivos.

---

## 🎯 Objetivos de una simulación

Una buena simulación debe permitir:

* Generar **eventos anómalos reales**.
* Comprobar que se generan findings, alertas o incidentes.
* Activar flujos de respuesta automáticos o manuales.
* Registrar **evidencias y tiempos** para análisis posterior.

---

## 🧪 Tipos de ataques simulables

| Categoría                  | Ejemplos                                                            |
| -------------------------- | ------------------------------------------------------------------- |
| Credenciales comprometidas | Uso de claves IAM filtradas, inicio de sesión con tokens de acceso. |
| Malware                    | Despliegue de binarios maliciosos en VMs.                           |
| Movimiento lateral         | Escaneo interno de puertos y redes.                                 |
| Exfiltración de datos      | Acceso masivo a buckets, bases de datos o volcado de logs.          |
| Minería                    | Uso de CPU inusual por malware de criptomonedas.                    |

---

## ☁️ Simulación en AWS

### 🧩 GuardDuty – Sample Findings

AWS permite generar detecciones ficticias con un solo comando. Estas no afectan a tus recursos, pero se comportan como findings reales.

```bash
aws guardduty create-sample-findings
```

Esto generará ejemplos como:

* `CryptoCurrency:EC2/BitcoinTool.B`
* `Recon:IAMUser/UserEnumeration`
* `UnauthorizedAccess:EC2/TorIPCaller`

Ver resultados en consola:

> **GuardDuty > Findings > Resource Type: Simulated**

💡 Ideal para probar:

* Conexión con **EventBridge**.
* Respuesta automática con **Lambda**.
* Integración con **Security Hub**.

---

## ☁️ Simulación en Azure

### 🔧 Microsoft Defender Attack Simulator (Microsoft 365 Defender)

Desde el portal de Microsoft 365 Defender, puedes simular ataques como:

* **Phishing de credenciales**
* **Malware adjunto**
* **Ransomware**
* **Ataques de contraseña (spraying, brute-force)**

Requiere:

* Tener licencias E5 o equivalentes.
* Acceso a Microsoft Defender for Office o Endpoint.

➡️ Portal: `https://security.microsoft.com/attacksimulator`

### ⚙️ Alternativas en Azure (si no se tiene M365 Defender)

Puedes simular actividad anómala manualmente:

1. Conectarte a una VM desde una IP de fuera de Europa (usando VPN).
2. Intentar múltiples fallos de inicio de sesión.
3. Descargar muchas veces un archivo desde un blob.
4. Ejecutar scripts sospechosos en máquinas monitorizadas por Defender.

💡 Defender y Sentinel deberían:

* Lanzar alertas.
* Activar recomendaciones.
* Ejecutar playbooks si están configurados.

---

## 🧑‍💼 Caso empresarial: prueba de minería en EC2

**Contexto**: el equipo de seguridad quiere verificar si GuardDuty detectaría actividad de minería en una instancia.

### Plan:

1. Lanzar instancia EC2 tipo `t2.micro` en entorno aislado.
2. Instalar un proceso de CPU intensivo (simulación de minero).
3. Observar si GuardDuty detecta `BitcoinTool.B`.
4. Activar la respuesta automática: apagar la instancia + notificar al equipo.

Opcional: repetir en Azure con Defender for Servers habilitado.

---

## 📈 Métricas clave a medir

| Métrica           | Cómo se mide                                              |
| ----------------- | --------------------------------------------------------- |
| **MTTD**          | Tiempo desde inicio de ataque hasta generación de alerta. |
| **MTTR**          | Tiempo desde alerta hasta contención automática o manual. |
| **Alert Fatigue** | ¿Cuántas alertas hubo sin acción ni utilidad?             |
| **Cobertura**     | ¿Qué simulaciones fueron **no detectadas**?               |

---

## ⚠️ Errores comunes

* ❌ Hacer simulaciones en producción sin autorización.
* ❌ No avisar al equipo SOC → falsas alarmas o acciones destructivas.
* ❌ Usar siempre el mismo tipo de ataque → “entrenamiento sobreentrenado”.
* ❌ Simular solo detección, sin medir respuesta.

---

## ✅ Buenas prácticas

✔ Documentar cada simulación como si fuera un **incidente real**.
✔ Acompañar cada ataque simulado con un **checklist de verificación**.
✔ Realizar **post-mortem técnico** aunque no sea un incidente real.
✔ Automatizar la simulación mensual de ciertos ataques básicos.
✔ Incluir al equipo de negocio si hay implicaciones reputacionales o de proceso.

---

## 📚 Recursos recomendados

### AWS

* [Simulate Findings – GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_sample_findings.html)
* [Security Incident Response Simulation](https://aws.amazon.com/blogs/security/simulate-security-incidents/)

### Azure

* [Attack Simulation Training](https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-overview)
* [Create custom detections in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/tutorial-detect-threats-custom)