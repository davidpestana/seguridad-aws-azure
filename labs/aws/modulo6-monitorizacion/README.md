# 🧪 Laboratorio 06 – Monitorizar una instancia EC2 con Amazon CloudWatch

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Consultar **métricas de rendimiento** de una instancia EC2 desde CloudWatch.
* Configurar **alarmas de CPU** para detectar uso anómalo.
* Instalar el **agente de CloudWatch** para recolectar logs del sistema.
* Validar visualizaciones, alarmas y eventos desde la consola y la CLI.

---

## 🧵 Escenario práctico

Tienes una instancia EC2 de backend en producción. Tu objetivo es comenzar a monitorizar su uso de CPU, disco y memoria, y generar alertas automáticas si el rendimiento es anómalo. También quieres recolectar los logs del sistema (`/var/log/messages`) para analizarlos desde CloudWatch Logs.

---

## 🧰 Requisitos previos

* Instancia EC2 activa (Amazon Linux 2) en estado "running".
* Acceso por SSH a la instancia.
* Rol IAM asociado con permisos para usar CloudWatch (ver paso 1).
* AWS CLI instalado en tu máquina (opcional para validaciones).

---

## 🧭 Pasos detallados

### 1️⃣ Crear un rol IAM con permisos de monitorización

1. Ve a **IAM** → **Roles** → **Create role**
2. Tipo de entidad: **AWS service** → **EC2**
3. Adjunta estas políticas:

   * ✅ `CloudWatchAgentServerPolicy`
   * ✅ `AmazonEC2ReadOnlyAccess` (opcional para demo)
4. Nombre del rol: `ec2-monitoring-role`
5. Crear el rol

Luego, ve a **EC2** → selecciona tu instancia → pestaña **Security** → **Modify IAM role** → selecciona `ec2-monitoring-role`

---

### 2️⃣ Conectar por SSH a la instancia

```bash
ssh -i mi-clave.pem ec2-user@<ip-publica>
```

---

### 3️⃣ Instalar y configurar el agente de CloudWatch

```bash
# Descargar e instalar el agente
sudo yum install -y amazon-cloudwatch-agent

# Crear archivo de configuración básico
cat <<EOF | sudo tee /opt/aws/amazon-cloudwatch-agent/bin/config.json
{
  "metrics": {
    "append_dimensions": {
      "InstanceId": "\${aws:InstanceId}"
    },
    "metrics_collected": {
      "mem": {
        "measurement": [
          "mem_used_percent"
        ]
      },
      "disk": {
        "resources": ["/"],
        "measurement": [
          "used_percent"
        ]
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/messages",
            "log_group_name": "/ec2/lab/messages",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
EOF
```

---

### 4️⃣ Iniciar el agente de CloudWatch

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json \
  -s
```

✅ El agente enviará métricas de RAM, disco y logs a CloudWatch.

---

### 5️⃣ Crear una alarma de CPU desde la consola

1. Ve a [CloudWatch > Alarms](https://console.aws.amazon.com/cloudwatch/home#alarmsV2:)
2. Clic en **Create alarm**
3. Selecciona **EC2 → Per-Instance Metrics → CPUUtilization**
4. Elige la instancia deseada
5. Condición:

   * ≥ 70%
   * Period: 5 minutos
6. Notificación:

   * Puedes omitir notificación o crear una SNS Topic

✅ La alarma se activará si la CPU pasa del 70% sostenido durante 5 minutos.

---

### 6️⃣ Visualizar métricas y logs

1. Ve a **CloudWatch → Metrics** → `CWAgent` namespace
2. Explora métricas de:

   * CPU, RAM (`mem_used_percent`)
   * Disco (`disk_used_percent`)
3. Ve a **CloudWatch → Logs → Log groups**:

   * Debe existir `/ec2/lab/messages`
   * Explora eventos del sistema

---

## ✅ Validación

* ¿El agente de CloudWatch está corriendo?
* ¿Ves métricas de RAM y disco en CloudWatch?
* ¿Puedes ver logs en `/ec2/lab/messages`?
* ¿La alarma de CPU está configurada y visible?

---

## ⚠️ Errores comunes

| Problema                            | Causa común                                     | Solución                                                |
| ----------------------------------- | ----------------------------------------------- | ------------------------------------------------------- |
| No aparecen métricas personalizadas | Agente no instalado o sin permisos              | Verifica configuración y reinicia el agente             |
| No puedes asociar el rol            | Instancia lanzada sin IAM role                  | Modifica la instancia y asocia el rol desde consola     |
| Logs no visibles                    | Error de path o permisos en `/var/log/messages` | Verifica que el archivo existe y el agente puede leerlo |

---

## 🧩 Extensiones opcionales

* Agrega métricas de red (`netstat`, `tcp_connections`) al config.json
* Crea una **alarma de RAM** si supera el 80%
* Configura notificaciones con SNS (correo o Slack)
* Integra con **CloudWatch Dashboards** para una visualización completa