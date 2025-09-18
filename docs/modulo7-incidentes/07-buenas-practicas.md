# ✅ Buenas Prácticas en Gestión de Incidentes en la Nube

---

## 🧭 ¿Por qué importa tener buenas prácticas?

Una gestión de incidentes **reactiva, desorganizada o improvisada** puede:

* **Aumentar el impacto** de los ataques.
* Generar **confusión y pánico** en el equipo.
* Poner en riesgo la **reputación empresarial**.
* Incumplir normativas como **ISO 27035**, **PCI DSS**, **NIS2** o **ENS**.

Adoptar un enfoque proactivo, entrenado y documentado garantiza una **respuesta efectiva**, alineada con objetivos de negocio y compliance.

---

## 🧩 Elementos clave de una buena estrategia

| Elemento                        | Descripción                                                     |
| ------------------------------- | --------------------------------------------------------------- |
| 🎯 **Runbooks**                 | Guías paso a paso para responder a tipos de incidentes comunes. |
| 👥 **Roles claros**             | Saber quién analiza, quién actúa, quién comunica.               |
| 📢 **Comunicación efectiva**    | Con equipo técnico, directivos y clientes.                      |
| 🔄 **Post-mortems útiles**      | Aprender de cada incidente y mejorar el plan.                   |
| 🧪 **Simulaciones regulares**   | Validar procesos y entrenar al equipo.                          |
| 🔏 **Auditoría y trazabilidad** | Toda acción debe quedar registrada.                             |

---

## 📘 ¿Qué debe incluir un runbook?

Un buen runbook debe ser **concreto, accionable y fácil de seguir**, incluso bajo presión.

**Ejemplo: “Detección de uso indebido de clave IAM” (AWS)**

```yaml
- Paso 1: Confirmar alerta en GuardDuty
- Paso 2: Deshabilitar clave con script IAM
- Paso 3: Revisar logs de CloudTrail
- Paso 4: Notificar por Teams + Email
- Paso 5: Rotar clave y documentar incidente
```

**Ejemplo: “Ataque RDP en Azure VM”**

```yaml
- Paso 1: Verificar alerta en Defender
- Paso 2: Ejecutar Playbook: apagar VM + snapshot
- Paso 3: Revisar tráfico en Log Analytics
- Paso 4: Escalar a CISO si implica datos personales
```

---

## 👥 Roles y responsabilidades

| Rol                   | Responsabilidad                              |
| --------------------- | -------------------------------------------- |
| **SOC Analyst**       | Monitoreo y primera respuesta.               |
| **Ingeniero Cloud**   | Ejecutar contención y mitigación.            |
| **Team Lead/Manager** | Escalado y toma de decisiones.               |
| **CISO**              | Evaluación del impacto y comunicación legal. |
| **DevSecOps**         | Mejorar reglas, detección y automatización.  |

📌 Si no hay responsables definidos, las alertas se ignoran o duplican esfuerzos.

---

## 📢 Comunicación durante incidentes

Es vital establecer:

* Canales dedicados (ej: Teams “🔥 INC-SOC”).
* Plantillas de notificación interna y externa.
* Protocolos para contactar con soporte del proveedor (AWS/Azure).
* Un “comunicador oficial” hacia negocio y clientes.

**Ejemplo de mensaje interno:**

> 🚨 Incidente detectado: Acceso no autorizado a StorageAccount.
> ⏱️ Detectado a las 02:34 UTC.
> 🛠️ Acción: acceso bloqueado, logs extraídos, snapshot creado.
> 👤 Responsable: Juan Pérez (SOC).
> 📝 Lecciones en marcha para incluir IP ranges seguros en NSG.

---

## 🔄 Lecciones aprendidas (post-mortems)

Después de cada incidente (real o simulado), haz una revisión:

* ¿Qué se detectó bien? ¿Qué falló?
* ¿Hubo retrasos? ¿Falsos positivos?
* ¿Se ejecutaron los protocolos correctamente?
* ¿Qué alertas no estaban afinadas?
* ¿Qué playbooks no se ejecutaron por permisos, errores o límites?

🛠️ Documenta y **mejora las reglas, scripts, permisos y alertas**.

---

## 🧪 Buenas prácticas operativas

| Práctica                                   | Valor                                            |
| ------------------------------------------ | ------------------------------------------------ |
| ✅ Entrenar al equipo cada trimestre        | Reduce errores humanos bajo presión.             |
| ✅ Usar detecciones simuladas               | Permite validar sin comprometer entornos reales. |
| ✅ Automatizar tareas repetitivas           | Reduce MTTR y carga del SOC.                     |
| ✅ Tener un “libro rojo” por tipo de ataque | Mejora velocidad y consistencia en la respuesta. |
| ✅ Medir y publicar KPIs de seguridad       | Ayuda a justificar mejoras y recursos.           |

---

## ⚠️ Errores comunes

* ❌ Confiar solo en herramientas → el proceso humano también falla.
* ❌ Alertas sin dueños → nadie actúa.
* ❌ No simular → equipo entra en pánico ante el primer ataque real.
* ❌ No versionar los runbooks → documentación obsoleta en un ataque.

---

## 📚 Normativas y referencias

* **ISO 27035**: Gestión de incidentes de seguridad de la información.
* **PCI DSS Req. 12.10**: Plan de respuesta a incidentes.
* **NIST SP 800-61**: Computer Security Incident Handling Guide.
* **ENS (España)**: Requiere plan de respuesta y roles definidos.

---

## 📚 Recursos adicionales

* [AWS Security Incident Response Guide](https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/aws-security-incident-response-guide.html)
* [Microsoft Sentinel: Best Practices](https://learn.microsoft.com/en-us/azure/sentinel/best-practices)
* [NIST Guide to Incident Response](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)
* [ISO 27035 Overview](https://www.iso.org/standard/44379.html)

---

✅ **Checklist final del módulo:**

* [x] ¿Tienes definidos roles y responsables?
* [x] ¿Cuentas con runbooks actualizados?
* [x] ¿Has hecho simulaciones este trimestre?
* [x] ¿Hay métricas y KPIs definidos?
* [x] ¿El negocio sabe cómo se le informa durante un incidente?