# 📡 CloudWatch en AWS: Métricas, Logs, Alarmas y Dashboards

## 🧭 Introducción

Amazon CloudWatch es el servicio de **monitorización centralizada** de AWS. Permite recolectar y visualizar **métricas**, **logs**, **eventos** y generar **alarmas**, convirtiéndose en el núcleo de la observabilidad operativa y de seguridad dentro de AWS.

Si bien comenzó como un sistema de métricas básicas (CPU, red), hoy CloudWatch permite **crear dashboards**, **consultar logs en tiempo real**, **analizar trazas**, y generar **alertas automatizadas** que pueden integrarse con flujos de acción como escalado automático o envío de notificaciones.

---

## 🔧 Componentes clave de CloudWatch

### 1. **Métricas**

* AWS ofrece métricas **por defecto** para la mayoría de servicios.
* Puedes crear **métricas personalizadas** desde scripts, aplicaciones o agentes.
* Pueden agregarse por dimensiones como región, instancia, recurso, etc.

📌 Ejemplos comunes:

* CPUUtilization (EC2)
* RequestCount (ALB)
* Throttles (Lambda)

### 2. **Logs (CloudWatch Logs)**

* Puedes enviar logs desde:

  * EC2 (vía agente CloudWatch)
  * Lambda
  * ECS/EKS
  * Aplicaciones propias
* Permite **filtrar, buscar, y crear métricas a partir de logs**.

📌 Ejemplo: crear una alerta si en un log aparece `"Access Denied"`.

### 3. **Alarmas (CloudWatch Alarms)**

* Se basan en **métricas o expresiones matemáticas**.
* Permiten definir umbrales (thresholds) y generar acciones:

  * Enviar notificaciones (SNS)
  * Escalar instancias (Auto Scaling)
  * Ejecutar Lambdas

📌 Ejemplo: CPU > 75% durante 5 minutos → activar autoescalado.

### 4. **Eventos (EventBridge)**

* Antes “CloudWatch Events”.
* Detecta eventos del plano de control (ej. creación de recurso, cambio de IAM).
* Puede lanzar acciones: notificaciones, funciones, automatizaciones.

### 5. **Dashboards**

* Visualización gráfica de métricas y logs.
* Totalmente personalizables (gráficos, widgets de texto, mapas, etc.)
* Útiles para seguridad, salud operativa y auditoría.

---

## 🧪 Caso de uso: Monitorizar un clúster web y escalar si CPU > 75%

### Escenario

Un clúster de EC2 detrás de un ALB atiende tráfico web. Se desea:

1. Medir uso de CPU por instancia.
2. Alertar si supera el 75% durante más de 5 minutos.
3. Escalar automáticamente el número de instancias.

---

## 🛠️ Configuración paso a paso

### 1. Ver métricas en consola

* Ir a **CloudWatch > Metrics > EC2 > Per-Instance Metrics**.
* Seleccionar `CPUUtilization`.
* Crear gráfico o alarma desde ahí.

---

### 2. Crear una alarma (CLI)

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPU" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 75 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:MyTopic \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0
```

---

### 3. Activar escalado automático (ejemplo en ASG)

```bash
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name web-asg \
  --policy-name scale-up-cpu \
  --scaling-adjustment 1 \
  --adjustment-type ChangeInCapacity
```

---

### 4. Enviar logs desde una instancia EC2

Instalar el agente CloudWatch:

```bash
sudo yum install -y amazon-cloudwatch-agent
```

Configurar:

```json
// /opt/aws/amazon-cloudwatch-agent/bin/config.json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [{
          "file_path": "/var/log/messages",
          "log_group_name": "webserver-logs",
          "log_stream_name": "{instance_id}"
        }]
      }
    }
  }
}
```

Lanzar:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

---

### 5. Crear un Dashboard

Desde consola:

* **CloudWatch > Dashboards > Create**
* Añadir widgets de:

  * CPU (gráfico de líneas)
  * Logs de errores (filtro por mensaje)
  * Texto con instrucciones
  * Mapa geográfico de peticiones (si aplica)

---

## 📊 Tips para probar

| Acción                | Descripción                                              |
| --------------------- | -------------------------------------------------------- |
| Forzar carga alta     | Usa `stress` en EC2: `sudo stress --cpu 2 --timeout 300` |
| Validar envío de logs | Añadir líneas manualmente en `/var/log/messages`         |
| Simular eventos       | Crear recurso vía CLI/API y observar EventBridge         |
| Probar escalado       | Ajustar umbrales para forzar políticas de escalado       |
| Exportar métricas     | Desde consola, se pueden descargar o consultar vía API   |

---

## ❌ Errores comunes

* **No instalar el agente de logs** en EC2 → sin logs en CloudWatch Logs.
* Crear alarmas sin **acciones asociadas** → se dispara pero no notifica.
* **No borrar logs antiguos** → coste innecesario.
* Usar solo métricas por defecto → poca visibilidad real de la aplicación.
* Alarmas demasiado sensibles → alert fatigue (ruido excesivo).

---

## ✅ Buenas prácticas

✔️ Enviar **métricas personalizadas** de aplicaciones críticas.
✔️ Separar **log groups por tipo de servicio**.
✔️ Activar **retención personalizada** en logs.
✔️ Usar dashboards como **visibilidad ejecutiva o SOC**.
✔️ Comprimir y exportar logs antiguos a S3.
✔️ Correlacionar logs + métricas para detección de anomalías.

---

## 📚 Recursos oficiales

* [CloudWatch Overview](https://docs.aws.amazon.com/cloudwatch/)
* [Using CloudWatch Logs](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html)
* [CloudWatch Agent Setup](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent.html)
* [EventBridge](https://docs.aws.amazon.com/eventbridge/)
* [Auto Scaling with Alarms](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-instance-monitoring.html)