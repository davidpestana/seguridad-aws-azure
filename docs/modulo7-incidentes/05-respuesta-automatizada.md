# ⚙️ Respuesta Automatizada a Incidentes en AWS y Azure

---

## 🧭 ¿Por qué automatizar la respuesta?

En un entorno cloud moderno, los ataques pueden escalar **en minutos**.
Automatizar las respuestas críticas permite:

* Reducir el **MTTR** (Mean Time to Respond).
* Evitar intervención humana en acciones repetitivas o urgentes.
* Minimizar el impacto de un incidente antes de que se propague.

⚠️ **Pero cuidado**: la automatización debe ser **controlada, reversible y auditada**. No se trata de “apagar fuegos automáticamente”, sino de responder con lógica y trazabilidad.

---

## 🔄 Flujo típico de respuesta automatizada

```plaintext
1️⃣ Detección del incidente
2️⃣ Evaluación del contexto
3️⃣ Ejecución de acciones (automáticas o supervisadas)
4️⃣ Notificación al equipo de seguridad
5️⃣ Registro y trazabilidad de la acción
```

---

## ☁️ AWS: Automatización con EventBridge + Lambda

### 🧩 Componentes clave

| Componente      | Función                                                                    |
| --------------- | -------------------------------------------------------------------------- |
| **GuardDuty**   | Detecta amenazas automáticamente.                                          |
| **EventBridge** | Captura eventos (findings) y los enruta según reglas.                      |
| **Lambda**      | Ejecuta funciones personalizadas (bloquear usuario, eliminar clave, etc.). |
| **SNS**         | Notifica por correo, SMS o sistemas externos.                              |

### 🧪 Ejemplo: deshabilitar clave IAM comprometida

#### 🔔 Paso 1: GuardDuty lanza finding

Ejemplo: `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`

#### 🔁 Paso 2: EventBridge detecta el evento

```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "type": ["UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration"]
  }
}
```

#### ⚙️ Paso 3: Lambda ejecuta la acción

```python
import boto3

def lambda_handler(event, context):
    iam = boto3.client('iam')
    username = event['detail']['resource']['accessKeyDetails']['userName']
    response = iam.delete_access_key(
        UserName=username,
        AccessKeyId=event['detail']['resource']['accessKeyDetails']['accessKeyId']
    )
    print(f"AccessKey eliminada: {response}")
```

#### 📣 Paso 4: SNS notifica

```bash
aws sns publish \
  --topic-arn arn:aws:sns:us-east-1:123456789012:AlertasSeguridad \
  --message "Clave IAM comprometida ha sido revocada automáticamente"
```

---

## ☁️ Azure: Automatización con Sentinel + Logic Apps

### 🧩 Componentes clave

| Componente                       | Función                                                  |
| -------------------------------- | -------------------------------------------------------- |
| **Microsoft Defender for Cloud** | Detecta anomalías y configuraciones inseguras.           |
| **Sentinel**                     | Crea incidentes y dispara automatizaciones.              |
| **Logic Apps**                   | Ejecuta flujos (playbooks) de forma automática o manual. |

### 🧪 Ejemplo: Aislar VM tras ataque RDP

#### 🔔 Paso 1: Defender detecta tráfico RDP inusual (puerto 3389)

#### 🔁 Paso 2: Sentinel lanza alerta con etiqueta “Alta gravedad”

#### ⚙️ Paso 3: Playbook ejecuta acción

```plaintext
1. Apagar la VM
2. Crear snapshot
3. Notificar a SOC por Teams
4. Registrar evento en Log Analytics
```

Código del playbook (resumen visual en portal):

```bash
az logicapp run \
  --name playbook-aislar-vm \
  --resource-group SOC-RG \
  --parameters '{"vmId": "/subscriptions/xxx/resourceGroups/yyy/providers/Microsoft.Compute/virtualMachines/vm-comprometida"}'
```

---

## 🧪 Validaciones prácticas

| Acción            | AWS                                    | Azure                                |
| ----------------- | -------------------------------------- | ------------------------------------ |
| Simular incidente | `aws guardduty create-sample-findings` | Microsoft Attack Simulator           |
| Ver ejecución     | CloudWatch Logs (Lambda)               | Logic App Run History                |
| Notificación      | SNS → Email, Teams, Slack              | Logic App → Email, Teams, ServiceNow |
| Trazabilidad      | CloudTrail                             | Log Analytics                        |

---

## ⚠️ Errores comunes

* ❌ Automatizar acciones destructivas sin validación previa.
* ❌ No incluir registros o logs de auditoría.
* ❌ Dejar lógica de respuesta en código “quemado”, sin parametrizar.
* ❌ No probar los flujos antes de usarlos en producción.
* ❌ Asumir que todo incidente debe tener respuesta automática: **a veces es mejor alertar y escalar manualmente**.

---

## ✅ Buenas prácticas

✔ Usar **etiquetas de severidad** para definir qué se automatiza y qué no.
✔ Añadir pasos de **reversibilidad** (crear snapshot antes de apagar, por ejemplo).
✔ Documentar cada playbook y su lógica.
✔ Incluir lógica de **excepciones** (ej: no apagar VMs críticas).
✔ Mantener flujos simples, modularizados y probados.

---

## 📚 Recursos recomendados

### AWS

* [Automated Response with GuardDuty + Lambda](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_lambda.html)
* [EventBridge Rules for Security](https://docs.aws.amazon.com/eventbridge/latest/userguide/what-is-amazon-eventbridge.html)

### Azure

* [Responding to threats with Azure Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/tutorial-respond-threats-playbook)
* [Create Logic Apps for Security](https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-overview)