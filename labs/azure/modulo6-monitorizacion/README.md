# 🧪 Laboratorio 06 – Monitorizar una VM con Azure Monitor y Log Analytics

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Habilitar la monitorización avanzada de una VM en Azure.
* Enlazar una máquina virtual a un **workspace de Log Analytics**.
* Visualizar **métricas en tiempo real**, logs del sistema y registros de actividad.
* Crear una **alerta basada en condiciones de CPU o disco**.

---

## 🧵 Escenario práctico

Tu organización quiere comenzar a recopilar información operativa de sus VMs Linux y Windows. Como administrador, debes conectar una VM existente a Azure Monitor para ver su estado, uso de recursos, logs y generar alertas si el rendimiento se degrada.

---

## 🧰 Requisitos previos

* VM Linux o Windows creada en Azure (por ejemplo: `vm-monitor-test`).
* Permisos para crear un **Log Analytics Workspace** y editar la configuración de la VM.
* Azure CLI (opcional para validación por línea de comandos).

---

## 🧭 Pasos detallados

### 1️⃣ Crear un Log Analytics Workspace

1. En el portal, busca **"Log Analytics workspaces"** → **Create**
2. Configura:

   * **Nombre**: `workspace-monitoring-lab`
   * **Región**: misma que la VM
   * **Grupo de recursos**: `rg-monitorizacion`
3. Clic en **Review + Create** → **Create**

---

### 2️⃣ Enlazar la VM al Workspace

1. Ve a **Máquinas virtuales** → selecciona `vm-monitor-test`
2. En el menú lateral: **Monitoring → Insights**
3. Clic en **Enable**
4. Selecciona el **Log Analytics Workspace** que creaste (`workspace-monitoring-lab`)
5. Acepta la instalación de agentes (si es necesario)

✅ Esto habilita la recopilación de métricas y logs del sistema operativo.

---

### 3️⃣ Consultar métricas básicas desde Azure Monitor

1. Ve a **Monitor → Metrics**
2. En *Scope*, selecciona la VM
3. Métrica: `Percentage CPU`, `Network In`, `Available Memory`, etc.
4. Crea un gráfico de línea con período de 30 minutos

---

### 4️⃣ Consultar logs en Log Analytics

1. Ve a **Monitor → Logs**
2. En la parte superior: selecciona `workspace-monitoring-lab`
3. Ejecuta una consulta básica:

```kusto
Heartbeat
| summarize LastSeen = max(TimeGenerated) by Computer
```

4. Para ver uso de CPU y memoria:

```kusto
Perf
| where ObjectName == "Processor" or ObjectName == "Memory"
| summarize avg(CounterValue) by bin(TimeGenerated, 5m), CounterName
```

✅ Puedes ver los datos enviados por la VM en tiempo real.

---

### 5️⃣ Crear una alerta basada en CPU

1. Ve a **Monitor → Alerts** → **Create alert rule**
2. Scope: selecciona `vm-monitor-test`
3. Condition:

   * **Signal type**: Metric
   * **Signal name**: `Percentage CPU`
   * **Operator**: Greater than
   * **Threshold**: `70`
   * Period: 5 min
4. Action group: crea uno nuevo (puede ser sin email si es demo)
5. Clic en **Review + Create**

✅ Esta alerta se disparará cuando la CPU esté alta durante más de 5 minutos.

---

## ✅ Validación

* ¿La VM está conectada al workspace?
* ¿Puedes ver métricas en la pestaña “Metrics”?
* ¿Puedes consultar logs desde Log Analytics (KQL)?
* ¿La alerta aparece en estado “Active” si fuerzas un uso alto de CPU?

---

## ⚠️ Errores comunes

| Problema            | Causa común                                       | Solución                                       |
| ------------------- | ------------------------------------------------- | ---------------------------------------------- |
| No se ven logs      | Agente no instalado o no habilitado correctamente | Verifica que se haya completado la instalación |
| Métricas vacías     | El recurso no está vinculado al workspace         | Asegúrate de enlazarlo correctamente           |
| Alerta no se activa | Umbral mal configurado o sin tráfico              | Prueba generando carga o bajando el umbral     |

---

## 🧩 Extensiones opcionales

* Crea una **alerta basada en logs** (KQL + condición personalizada).
* Añade métricas de disco, red o procesos específicos.
* Visualiza las métricas en un **dashboard personalizado**.
* Usa Azure CLI para automatizar la conexión de nuevas VMs a Log Analytics.