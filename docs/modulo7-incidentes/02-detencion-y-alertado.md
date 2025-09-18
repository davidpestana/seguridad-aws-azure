# 🔍 Detección Temprana y Alertado en la Nube

---

## 🧭 ¿Por qué importa la detección temprana?

La **detección temprana de amenazas** es clave para evitar que un incidente menor evolucione en una brecha mayor. Identificar patrones inusuales, accesos anómalos o configuraciones peligrosas permite **responder rápidamente** y minimizar el daño.

En entornos cloud, donde los recursos son accesibles globalmente y el aprovisionamiento es ágil, **la superficie de ataque cambia constantemente**, lo que exige:

* Monitoreo continuo.
* Alertas inteligentes, no solo técnicas.
* Correlación entre múltiples señales.

---

## 📊 ¿Qué señales de alerta debemos observar?

En AWS y Azure, hay múltiples **fuentes de señales de posible compromiso**, conocidas como **IoC (Indicators of Compromise)**. Algunos ejemplos:

| Fuente                | Señales de Alerta                                                      |
| --------------------- | ---------------------------------------------------------------------- |
| Logs de autenticación | Inicios de sesión desde países inusuales, múltiples intentos fallidos. |
| Logs de red           | Conexiones salientes a dominios maliciosos, escaneo de puertos.        |
| Logs de configuración | Cambios súbitos en políticas IAM, apertura de puertos en firewall.     |
| Actividad de usuarios | Acceso fuera de horario laboral, uso de API no habitual.               |

---

## ☁️ AWS: Detección con CloudTrail, CloudWatch y GuardDuty

### 🛠️ Servicios implicados

| Servicio       | Función                                                                                |
| -------------- | -------------------------------------------------------------------------------------- |
| **CloudTrail** | Registro detallado de todas las llamadas a la API (por usuarios, servicios o scripts). |
| **CloudWatch** | Permite crear métricas personalizadas y alertas a partir de logs.                      |
| **GuardDuty**  | Motor de detección basado en inteligencia de amenazas y machine learning.              |

### 🧪 Ejemplo de detección en AWS

**Escenario**: un atacante intenta escalar privilegios en una cuenta usando la API `CreatePolicyVersion`.

**Paso a paso**:

1. **CloudTrail** registra el evento API.
2. **GuardDuty** lo analiza y detecta como actividad sospechosa.
3. **CloudWatch** puede activar una alerta si se cumple una métrica específica (ej. creación de más de 2 políticas en 1 minuto).
4. **SNS** notifica al equipo de seguridad.

```bash
# Activar alertas desde métricas de CloudWatch
aws cloudwatch put-metric-alarm \
  --alarm-name "Exceso-Politicas-IAM" \
  --metric-name "PutPolicy" \
  --namespace "AWS/CloudTrail" \
  --statistic Sum \
  --period 60 \
  --threshold 2 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:REGION:ACCOUNT_ID:TopicName
```

---

## ☁️ Azure: Monitorización con Activity Logs, Azure Monitor y Defender

### 🛠️ Servicios implicados

| Servicio                         | Función                                                         |
| -------------------------------- | --------------------------------------------------------------- |
| **Activity Logs**                | Registro de operaciones en recursos y servicios.                |
| **Azure Monitor**                | Centraliza métricas, logs y alertas (equivalente a CloudWatch). |
| **Microsoft Defender for Cloud** | Detecta amenazas en tiempo real y malas configuraciones.        |

### 🧪 Ejemplo de detección en Azure

**Escenario**: un usuario crea una regla de red que permite tráfico desde cualquier IP (`0.0.0.0/0`) al puerto 3389 (RDP).

**Paso a paso**:

1. **Activity Log** detecta la creación de la regla.
2. **Defender for Cloud** lanza una recomendación crítica.
3. **Azure Monitor** puede tener configurada una **alerta personalizada** para ese evento.
4. **Email/Teams** notifican al equipo.

```bash
# Crear una alerta en Azure Monitor vía CLI
az monitor metrics alert create \
  --name "Alerta-RDP-Abierto" \
  --resource-group SEGURIDAD \
  --scopes /subscriptions/<SUB_ID>/resourceGroups/SEGURIDAD/providers/Microsoft.Network/networkSecurityGroups/NSG-Ejemplo \
  --condition "count Requests > 1" \
  --description "Detectar apertura de RDP a Internet"
```

---

## 🧑‍💼 Ejemplo empresarial: alerta por acceso desde IP anómala

Una empresa tecnológica ha desplegado una plataforma SaaS en AWS. El equipo de seguridad desea recibir una alerta **si hay un inicio de sesión desde un país donde no operan**, por ejemplo, Rusia o Corea del Norte.

| Plataforma | Configuración                                                                                              |
| ---------- | ---------------------------------------------------------------------------------------------------------- |
| **AWS**    | Activar GuardDuty, que incluye detecciones como `UnauthorizedAccess:IAMUser/ConsoleLoginFromSuspiciousIP`. |
| **Azure**  | Defender for Cloud y Azure Monitor, con reglas KQL en Log Analytics para geografías sospechosas.           |

💡 Ambas plataformas permiten integrar con sistemas externos como Slack, Teams, o ServiceNow para alertado y ticketing.

---

## 🧪 Tips para probar y visualizar alertas

| Acción                      | AWS                         | Azure                               |
| --------------------------- | --------------------------- | ----------------------------------- |
| Ver eventos sospechosos     | GuardDuty > Findings        | Defender > Recommendations / Alerts |
| Crear alerta manual         | CloudWatch Alarms           | Azure Monitor Alerts                |
| Ver logs crudos             | CloudTrail > Event History  | Azure Activity Logs                 |
| Simular actividad maliciosa | Sample Findings (GuardDuty) | Microsoft Attack Simulator          |

---

## ⚠️ Errores comunes

* ❌ Crear alertas genéricas que generan muchos falsos positivos.
* ❌ No integrar las alertas con herramientas de comunicación o ticketing.
* ❌ Configurar métricas pero no activar acciones (SNS, Logic App, etc.).
* ❌ Omitir la validación periódica de que los sistemas de alertado **realmente funcionan**.

---

## ✅ Buenas prácticas

✔ Diseñar alertas **contextuales**, basadas en el comportamiento esperado de tu organización.
✔ Agrupar señales similares para evitar alertas duplicadas.
✔ Asegurar que cada alerta tenga **un responsable y un protocolo** de respuesta.
✔ Usar múltiples fuentes (logs, actividad, métricas, seguridad) para detectar incidentes con mejor precisión.

---

## 📚 Recursos recomendados

### AWS

* [Guía de Amazon GuardDuty](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)
* [Crear métricas de CloudWatch desde CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudwatch-alarms-for-cloudtrail.html)

### Azure

* [Microsoft Defender for Cloud – Alertas y recomendaciones](https://learn.microsoft.com/en-us/azure/defender-for-cloud/recommendations-reference)
* [Azure Monitor – Alertas personalizadas](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview)