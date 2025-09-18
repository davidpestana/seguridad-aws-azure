# 📅 Planificación del Curso: Seguridad en AWS y Azure

Este documento resume la estructura del curso intensivo de **Seguridad en la Nube con AWS y Azure**, impartido en **4 jornadas** de 4,5 horas efectivas cada una. El curso combina teoría y práctica sobre entornos reales de AWS y Azure, con apoyo de laboratorios guiados.

---

## 🧭 Estructura General

- 🗓️ **Duración**: 4 días (de 09:00 a 14:00, con 30 minutos de descanso)
- 🧪 **Formato**: Teoría breve + laboratorio práctico por módulo
- 💡 **Modalidad**: GitHub Codespaces o entorno local (DevContainer)

---

## ✅ Día 1 – Introducción + Gestión de Identidades

### Módulos cubiertos:
- Módulo 1: Introducción a la Seguridad en la Nube
- Módulo 2: Gestión de Identidades y Accesos

### Contenidos:
- Principios de seguridad en la nube
- Modelo de responsabilidad compartida
- IAM en AWS: usuarios, grupos, políticas
- Azure AD: roles, grupos, permisos

### Laboratorios:
- [ ] IAM en AWS: gestión de usuarios y políticas
- [ ] Azure AD: configuración de usuarios y control de acceso

---

## ✅ Día 2 – Seguridad en Red y Almacenamiento

### Módulos cubiertos:
- Módulo 3: Seguridad en la Red
- Módulo 4: Almacenamiento Seguro

### Contenidos:
- Diseño de redes seguras (VPC / VNet)
- Subredes públicas y privadas
- Firewalls, SG, NSG, puntos de conexión
- Control de acceso y cifrado en S3 / Blob

### Laboratorios:
- [ ] AWS: VPC con subredes y reglas de seguridad
- [ ] Azure: VNet con NSG por capas
- [ ] AWS: S3 bucket con políticas de acceso y cifrado
- [ ] Azure: Blob Storage seguro con redundancia

---

## ✅ Día 3 – Seguridad en Cómputo + Monitorización

### Módulos cubiertos:
- Módulo 5: Seguridad en Servicios de Cómputo
- Módulo 6: Monitorización y Registro de Actividades

### Contenidos:
- Lanzamiento y aseguramiento de instancias EC2 / VMs
- Acceso seguro: claves SSH, RDP, roles IAM
- Monitorización: métricas, alertas, dashboards
- Logs y eventos en CloudWatch y Azure Monitor

### Laboratorios:
- [ ] EC2 con rol, clave y configuración segura
- [ ] VM en Azure con acceso controlado
- [ ] CloudWatch: métricas y alarmas
- [ ] Azure Monitor: logs y alertas

---

## ✅ Día 4 – Incidentes, DevSecOps y Compliance

### Módulos cubiertos:
- Módulo 7: Gestión de Incidentes y Respuesta
- Módulo 8: Seguridad en Aplicaciones y DevOps
- Módulo 9: Compliance y Gobernanza

### Contenidos:
- Detección de amenazas: GuardDuty, Security Center
- Respuesta automatizada a incidentes
- Pipelines CI/CD seguros en AWS y Azure
- Políticas y cumplimiento en múltiples cuentas

### Laboratorios:
- [ ] AWS: GuardDuty + Config con simulación de amenaza
- [ ] Azure: Security Center + Sentinel con respuesta
- [ ] CI/CD seguro con pruebas en pipeline (CodePipeline o GitHub Actions)
- [ ] Azure Policy y Blueprint / AWS Organizations y Control Tower

---

## 📦 Recursos adicionales

- 📁 [docs/](./): resúmenes teóricos y material de apoyo
- 🧪 [labs/aws/](../labs/aws/) y [labs/azure/](../labs/azure/): prácticas guiadas paso a paso
- ⚙️ Compatible con GitHub Codespaces y entorno local con DevContainer

---

> 🎯 Esta planificación busca un equilibrio entre el dominio de conceptos clave y la aplicación práctica inmediata, adaptada a profesionales que trabajan con entornos cloud en producción.

