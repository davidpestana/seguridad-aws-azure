# 📄 `06-monitorizacion-de-red.md`

## 📡 Monitorización del tráfico y alertas en la red cloud

### 🎯 Objetivo

Aprender a **observar, registrar y analizar el tráfico de red** en AWS y Azure para:

* Detectar patrones sospechosos.
* Auditar accesos no autorizados.
* Generar alertas automáticas ante anomalías.

---

## 🧠 Fundamentos

En la nube, los ataques no siempre implican exploits complejos: muchas veces comienzan con **acceso no detectado** o **movimiento lateral silencioso**.

La monitorización de red cumple un rol esencial en la estrategia de seguridad:

* Visibilidad sobre quién se conecta, cuándo y a qué.
* Correlación con eventos de seguridad (login sospechoso, escalado de privilegios).
* Prevención mediante alertas, SIEMs y respuesta automatizada.

---

## 🔄 Comparativa AWS ↔ Azure

| Capacidad                    | AWS                                             | Azure                                         |
| ---------------------------- | ----------------------------------------------- | --------------------------------------------- |
| Logs de tráfico de red       | **VPC Flow Logs**                               | **NSG Flow Logs**                             |
| Herramienta de visualización | CloudWatch Logs, Athena                         | Network Watcher, Azure Monitor                |
| Detección de anomalías       | GuardDuty (detección inteligente sobre tráfico) | Azure Defender for Networks                   |
| Integración con SIEM         | CloudWatch Logs ↔ AWS OpenSearch / Splunk / ELK | Azure Monitor ↔ Azure Sentinel / Splunk / ELK |
| Alertas automáticas          | CloudWatch Alarms / EventBridge                 | Azure Monitor Alerts / Action Groups          |

---

## 🧪 Caso práctico – detectar escaneo de puertos

**Escenario:**
Una aplicación web en Azure comienza a responder de forma lenta. Se sospecha que hay un escaneo de puertos o ataque de fuerza bruta en curso.

### 🔎 En Azure

1. Activar **NSG Flow Logs** con `Network Watcher`.
2. Revisar logs en Log Analytics:

```kusto
AzureNetworkAnalytics_CL
| where Direction_s == "Inbound"
| summarize count() by SrcIp_s, DestPort_d
| order by count_ desc
```

3. Detectar múltiples intentos desde una IP única en muchos puertos → posible escaneo.

4. Crear alerta con Azure Monitor:

```bash
az monitor metrics alert create \
  --name "PortScanAlert" \
  --resource-group myRG \
  --scopes <resource_id> \
  --condition "count > 100" \
  --description "Posible escaneo de puertos"
```

---

### 🔎 En AWS

1. Activar **VPC Flow Logs** en la subred o VPC sospechosa.
2. Analizar con Athena:

```sql
SELECT srcaddr, dstport, COUNT(*) as attempts
FROM vpc_flow_logs
WHERE action = 'ACCEPT'
GROUP BY srcaddr, dstport
ORDER BY attempts DESC;
```

3. Detectar IPs con muchas conexiones a puertos distintos.

4. Crear alerta en CloudWatch:

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "ScanDetected" \
  --metric-name NetworkPacketsIn \
  --namespace AWS/EC2 \
  --statistic Sum \
  --period 60 \
  --threshold 1000 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions <SNS_ARN>
```

---

## 💡 Consejos para testear

* Simula actividad con herramientas como `nmap`, `curl` o `ab` (Apache Benchmark).
* Crea reglas de tráfico restrictivas y observa los rechazos (acciones `REJECT` o `DROP` en logs).
* Reproduce escenarios de tráfico legítimo e ilegítimo para entrenar tu detección.

---

## ❌ Errores comunes

| Error                                     | Consecuencia                                 | Solución                                                     |
| ----------------------------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| Habilitar logs pero no revisarlos         | Terabytes de datos inútiles                  | Automatiza análisis y define alertas                         |
| No tener retención adecuada               | Imposibilidad de auditar un incidente pasado | Define políticas de retención según normativa (ej. 90 días)  |
| No activar Flow Logs en subredes críticas | Ceguera ante actividad interna               | Habilita logs en todas las subredes relevantes               |
| Falta de integración con SIEM o alertas   | Respuesta reactiva y tardía ante incidentes  | Usa Azure Sentinel o integra con sistemas externos           |
| Permitir tráfico saliente sin auditarlo   | Exfiltración de datos sin ser detectada      | Revisa tráfico saliente, sobre todo desde recursos sensibles |

---

## ✅ Buenas prácticas

* **Habilita Flow Logs** desde el primer despliegue (NSG o VPC).
* Define **alertas automáticas** para picos de conexiones, tráfico inusual o nuevas IPs.
* Usa **dashboards** en Azure Monitor o CloudWatch para seguimiento en tiempo real.
* Integra con **SIEMs** como Azure Sentinel, Splunk o ELK para correlación avanzada.
* Usa herramientas como **AWS GuardDuty** o **Azure Defender** para detección inteligente basada en machine learning.

---

## 🧪 CLI – Activación de logs

### AWS – Activar VPC Flow Logs

```bash
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-abc123 \
  --traffic-type ALL \
  --log-group-name vpc-logs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/FlowLogRole
```

### Azure – Activar NSG Flow Logs

```bash
az network watcher flow-log configure \
  --resource-group myRG \
  --nsg myNSG \
  --enabled true \
  --storage-account myStorage \
  --retention 90
```

---

## 📚 Recursos oficiales

* [AWS – VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html)
* [Azure – NSG Flow Logs](https://learn.microsoft.com/en-us/azure/network-watcher/network-watcher-nsg-flow-logging-overview)
* [AWS – Athena para análisis de logs](https://docs.aws.amazon.com/athena/latest/ug/what-is.html)
* [Azure – Alertas con Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview)