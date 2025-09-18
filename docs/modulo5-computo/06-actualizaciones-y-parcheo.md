# 🔄 Actualizaciones y parcheo automático de instancias (AWS y Azure)

---

## 🗂️ Introducción

El **parcheo de sistemas operativos y software** es una de las tareas más críticas —y a menudo olvidadas— en la gestión segura de máquinas virtuales. En cloud, donde las instancias son efímeras y escalan rápidamente, mantenerlas actualizadas requiere una **estrategia automatizada y coherente**.

Este archivo detalla cómo gestionar el parcheo en **AWS (EC2)** y **Azure (VMs)**, cubriendo desde herramientas nativas hasta buenas prácticas con imágenes inmutables.

---

## ⚠️ Riesgos de no aplicar parches

* Exposición a **vulnerabilidades conocidas** (exploits públicos).
* Infección con **malware o ransomware** automatizado.
* Compromiso de credenciales, escalada de privilegios o denegación de servicio.
* Incumplimiento de normativas (ENS, ISO27001, PCI DSS...).

Ejemplo real: *WannaCry explotó una vulnerabilidad de Windows parcheada semanas antes. Afectó a miles de servidores mal mantenidos.*

---

## 🧰 Estrategias generales de actualización

| Enfoque                     | Descripción                                               | Cuándo usarlo                          |
| --------------------------- | --------------------------------------------------------- | -------------------------------------- |
| **Parcheo automático**      | El SO se actualiza al arrancar o en segundo plano         | Casos simples, poco críticos           |
| **Parcheo por lotes**       | Ejecutado por herramientas de gestión centralizada        | Entornos con decenas de VMs            |
| **Golden Images**           | Se crean imágenes pre-parcheadas para lanzamientos nuevos | Infraestructura inmutable, autoscaling |
| **Infraestructura efímera** | En lugar de parchear, se reemplaza con nueva instancia    | En CI/CD o arquitectura serverless     |

---

## 🟡 AWS: Systems Manager Patch Manager

### 🔧 ¿Qué es?

**AWS Systems Manager Patch Manager** permite definir **parámetros, ventanas de mantenimiento y políticas** para aplicar parches a instancias EC2 compatibles (Linux y Windows), de forma automática y escalable.

### ✅ Requisitos

* EC2 debe tener:

  * **SSM Agent instalado y en ejecución.**
  * **IAM Role asociado** con permisos de SSM (ej. `AmazonSSMManagedInstanceCore`).
  * Acceso a Internet o VPC Endpoint para SSM.

### 🧪 Ejemplo: aplicar parches automáticamente

```bash
aws ssm create-patch-baseline \
  --name "BaselineLinux" \
  --operating-system AMAZON_LINUX_2 \
  --approval-rules '{
    "PatchRules": [{
      "PatchFilterGroup": {
        "PatchFilters": [{
          "Key": "CLASSIFICATION",
          "Values": ["Security"]
        }]
      },
      "ApproveAfterDays": 3
    }]
  }'
```

Luego puedes aplicar parches con:

```bash
aws ssm send-command \
  --document-name "AWS-RunPatchBaseline" \
  --targets Key=instanceIds,Values=i-abc123
```

---

## 🔵 Azure: Update Management (Azure Automation)

### 🔧 ¿Qué es?

**Azure Update Management** forma parte de **Azure Automation** y permite:

* Detectar qué actualizaciones están pendientes.
* Crear **grupos de máquinas** con diferentes políticas.
* Aplicar parches automáticamente o bajo aprobación manual.
* Generar reportes de cumplimiento.

### ✅ Requisitos

* Activar **Log Analytics workspace**.
* Registrar las VMs en Azure Automation.
* Permitir salida a Internet o configurar rutas privadas.

```bash
az automation software-update-configuration create \
  --automation-account-name mi-auto \
  --resource-group mi-rg \
  --name "Patch-Group-Web" \
  --update-configuration '{"operatingSystem": "Linux", "linux": {"includedPackageClassifications":["Security"]}}' \
  --schedule '{"frequency": "Week", "interval": 1, "startTime": "2025-09-30T02:00:00+00:00"}'
```

---

## 🧪 Golden Images: parchear antes de lanzar

> 🔁 En entornos inmutables, la **mejor práctica no es “parchear” instancias activas, sino crear una nueva imagen actualizada y redeplegar.**

* Garantiza consistencia entre entornos.
* Reduce tiempo de recuperación (lanzamiento rápido).
* Compatible con **CI/CD pipelines** y **Auto Scaling Groups**.

### En AWS

* Usa EC2 Image Builder o `create-image` manualmente tras aplicar parches.

```bash
aws ec2 create-image \
  --instance-id i-abc123 \
  --name "webapp-golden-2025-09"
```

### En Azure

* Usa `az image create` o **Shared Image Gallery** para distribuir golden images.

```bash
az image create \
  --resource-group mi-rg \
  --name webimage-secure \
  --source mi-vm \
  --os-type Linux
```

---

## 🧠 Buenas prácticas de parcheo

| Recomendación                                 | Motivo                                                  |
| --------------------------------------------- | ------------------------------------------------------- |
| Parchear antes del lanzamiento                | Reduce riesgo desde el arranque                         |
| Automatizar ciclo de parches                  | Evita dependencias humanas o tareas olvidadas           |
| Validar compatibilidad en entornos de pruebas | Evita interrupciones en producción por incompatibilidad |
| Incluir parches en pipelines CI/CD            | Versiones siempre actualizadas                          |
| Revisar logs de errores tras parches          | Detecta fallos o parches no aplicados                   |

---

## 🏢 Caso empresarial

**Escenario**: una empresa de ecommerce con 20 servidores web Linux y 10 bases de datos en Windows.

| Requisito                                 | AWS                                   | Azure                             |
| ----------------------------------------- | ------------------------------------- | --------------------------------- |
| Aplicar parches de seguridad semanalmente | Patch Manager + Maintenance Window    | Update Management + Schedule      |
| Evitar downtime durante horario laboral   | Maintenance Windows configurados      | Ventana definida fuera de horario |
| Ver qué VMs están al día                  | Compliance report via Systems Manager | Azure Automation dashboard        |
| Automatizar golden image                  | EC2 Image Builder pipeline            | Packer + Shared Image Gallery     |

---

## ❌ Errores comunes

1. **No aplicar parches por meses** porque “la VM funciona bien”.
2. **Parchear manualmente cada instancia** → inconsistencia y olvido.
3. No validar parches antes en preproducción → incompatibilidades críticas.
4. No monitorear logs de post-parcheo → errores silenciosos.
5. No automatizar creación de imágenes actualizadas.
6. Usar versiones de SO obsoletas o sin soporte (fin de vida).

---

## ✅ Checklist rápido

| Componente                    | AWS                          | Azure                            |
| ----------------------------- | ---------------------------- | -------------------------------- |
| Parcheo automático            | ✅ Patch Manager (SSM)        | ✅ Update Management (Automation) |
| Parcheo programado            | ✅ Maintenance Windows        | ✅ Schedule en Update Management  |
| Reportes de cumplimiento      | ✅ Systems Manager Compliance | ✅ Azure Monitor / Update Logs    |
| Golden Images                 | ✅ EC2 Image Builder          | ✅ Shared Image Gallery           |
| Infraestructura inmutable     | Auto Scaling Groups + AMI    | VMSS + Golden Image              |
| Alertas ante fallo de parches | CloudWatch / SNS             | Log Analytics + Alerts           |

---

## 🔗 Recursos oficiales

* [AWS Systems Manager Patch Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-patch.html)
* [EC2 Image Builder](https://docs.aws.amazon.com/imagebuilder/latest/userguide/what-is-image-builder.html)
* [Azure Update Management Overview](https://learn.microsoft.com/en-us/azure/automation/update-management/overview)
* [Azure Image Management Best Practices](https://learn.microsoft.com/en-us/azure/virtual-machines/shared-image-galleries)