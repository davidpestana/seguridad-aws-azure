# 📘 Introducción a la Gestión de Incidentes en la Nube

---

## 🧭 ¿Qué es un incidente de seguridad y por qué importa?

Un **incidente de seguridad** en la nube es cualquier evento que afecte negativamente a la **confidencialidad**, **integridad** o **disponibilidad** de los activos digitales.
Esto incluye accesos no autorizados, robo de credenciales, modificaciones maliciosas, ataques DDoS, entre otros.

En entornos cloud como AWS y Azure, los incidentes no solo son posibles, sino que **pueden propagarse rápidamente** si no se detectan y gestionan de forma oportuna.

💡 **La velocidad de detección y respuesta es crítica.**
Por eso trabajamos con dos métricas clave:

| Métrica | Significado                                                                |
| ------- | -------------------------------------------------------------------------- |
| `MTTD`  | Mean Time to Detect – Tiempo promedio en detectar un incidente.            |
| `MTTR`  | Mean Time to Respond – Tiempo promedio en contener y mitigar un incidente. |

---

## 🔄 Ciclo de vida de la respuesta a incidentes (según NIST)

La **guía NIST 800-61r2** define un ciclo estructurado para afrontar incidentes de seguridad, aplicable tanto a entornos on-prem como cloud:

```
1️⃣ Preparación
2️⃣ Detección y Análisis
3️⃣ Contención, Erradicación y Recuperación
4️⃣ Actividades post-incidente
```

| Etapa                    | Descripción                                                   |
| ------------------------ | ------------------------------------------------------------- |
| **Preparación**          | Definir roles, herramientas, políticas y entrenamientos.      |
| **Detección**            | Identificar eventos sospechosos y convertirlos en incidentes. |
| **Contención**           | Limitar el alcance y evitar la propagación.                   |
| **Erradicación**         | Eliminar vectores de ataque.                                  |
| **Recuperación**         | Restaurar sistemas y servicios.                               |
| **Lecciones aprendidas** | Revisar, documentar y mejorar el plan de respuesta.           |

---

## 🔐 Responsabilidad compartida: ¿quién responde?

Uno de los principios clave en la nube es el **modelo de responsabilidad compartida**:

| Elemento                   | Responsable en AWS/Azure | Responsable del Cliente |
| -------------------------- | ------------------------ | ----------------------- |
| Seguridad física           | ✅                        | ❌                       |
| Infraestructura y hardware | ✅                        | ❌                       |
| Hipervisores y red core    | ✅                        | ❌                       |
| Configuración de cuentas   | ❌                        | ✅                       |
| Seguridad de recursos      | ❌                        | ✅                       |
| Detección y respuesta      | ❌                        | ✅                       |

👉 Esto significa que **el cliente es responsable de detectar y gestionar los incidentes en su cuenta**, aunque la infraestructura sea gestionada por el proveedor.

---

## 🧪 Ejemplo empresarial: clave IAM expuesta

Supongamos que una empresa expone por error una **clave de acceso AWS IAM** en un repositorio público.

| Fase             | Acción esperada                                                           |
| ---------------- | ------------------------------------------------------------------------- |
| **Detección**    | GuardDuty detecta uso anómalo desde una IP rusa.                          |
| **Contención**   | Deshabilitar clave IAM, rotar credenciales.                               |
| **Erradicación** | Auditar repositorios, buscar más claves filtradas.                        |
| **Recuperación** | Reasignar permisos temporales y revisar logs.                             |
| **Lecciones**    | Activar políticas de prevención de fuga (ex. GitHub Actions, pre-commit). |

Este tipo de incidente puede tener **impacto grave** incluso si aún no se ha explotado: basta con que la clave esté disponible públicamente para activar los protocolos de respuesta.

---

## ⚙️ Código y configuración inicial

Aunque esta sección es introductoria, puedes preparar tu entorno para los laboratorios siguientes:

### 🔧 AWS CLI – Activar GuardDuty

```bash
aws guardduty create-detector --enable
```

### 🔧 Azure CLI – Activar Defender

```bash
az security auto-provisioning-setting update \
  --name default \
  --auto-provision "On"
```

---

## 🧪 Tips para validar en consola

| Acción                         | AWS                   | Azure                               |
| ------------------------------ | --------------------- | ----------------------------------- |
| Ver activación de servicio     | GuardDuty > Detectors | Microsoft Defender for Cloud        |
| Ver alertas recientes          | GuardDuty > Findings  | Defender > Recommendations / Alerts |
| Simular alertas (más adelante) | Sample Findings       | Simulación de ataque                |

---

## ⚠️ Errores comunes

* ❌ Confiar en detección automática sin configurarla explícitamente.
* ❌ No revisar logs de seguridad regularmente.
* ❌ Pensar que “como no pasó nada”, no es necesario responder.

---

## ✅ Buenas prácticas

✔ Establecer un **plan formal de gestión de incidentes**, visible para todos los equipos.
✔ Entrenar periódicamente con **simulaciones de incidentes**.
✔ Activar servicios nativos de detección desde el primer día.
✔ Documentar cada incidente para **retroalimentar el plan**.

---

## 📚 Recursos recomendados

### AWS

* [AWS GuardDuty – documentación oficial](https://docs.aws.amazon.com/guardduty/latest/ug/what-is-guardduty.html)
* [AWS Security Incident Response Guide](https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/aws-security-incident-response-guide.html)

### Azure

* [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/)
* [Azure Incident Response](https://learn.microsoft.com/en-us/security/incident-response/overview)