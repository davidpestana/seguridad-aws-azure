# 🛡️ Hardening y configuración segura de instancias (AWS y Azure)

---

## 🗂️ Introducción

**Hardening** (o endurecimiento) es el proceso de **reducir la superficie de ataque** de una instancia eliminando configuraciones por defecto, desactivando servicios innecesarios y aplicando políticas de seguridad desde el arranque.

En cloud, el **riesgo escala tan rápido como la infraestructura**. Cada nueva instancia pública con SSH/RDP abierto y sin hardening es una puerta abierta para escáneres automáticos y atacantes.

---

## 🔐 ¿Por qué es crítico el hardening en la nube?

* Las instancias pueden estar **expuestas globalmente** en segundos.
* Muchas se crean desde **imágenes no validadas** (Marketplace, AMIs de terceros).
* A menudo se despliegan en pipelines **automatizados sin control manual**.
* El modelo de responsabilidad compartida indica que **la seguridad del sistema operativo es tuya**.

---

## 📏 Estándares y referencias

| Estándar           | Descripción                                                |
| ------------------ | ---------------------------------------------------------- |
| **CIS Benchmarks** | Guías de configuración segura específicas por SO y entorno |
| **DISA STIGs**     | Normas de seguridad del Departamento de Defensa (EEUU)     |
| **SCAP**           | Protocolo para escanear y validar configuraciones seguras  |

🔗 Puedes descargar benchmarks de CIS en: [https://www.cisecurity.org/cis-benchmarks/](https://www.cisecurity.org/cis-benchmarks/)

---

## 🧰 Opciones de hardening desde el inicio

### ✅ Usar imágenes endurecidas

| Plataforma | Opción recomendada                                             |
| ---------- | -------------------------------------------------------------- |
| AWS        | **CIS Hardened AMIs** desde AWS Marketplace                    |
| Azure      | **Hardened Images** en Azure Marketplace (Microsoft / Bitnami) |

> 🟡 Ventaja: incluyen configuraciones preestablecidas según CIS/DISA.
> 🔴 Reto: pueden requerir ajuste de compatibilidad con tus aplicaciones.

---

### 🧩 Configuración inicial (bootstrap / cloud-init)

Desde el arranque, puedes aplicar scripts de configuración segura para:

* Deshabilitar login root.
* Forzar autenticación por clave pública.
* Configurar firewall (iptables/ufw).
* Establecer parámetros de auditoría (auditd, syslog).
* Instalar antivirus o agentes de monitoreo.

#### Ejemplo: script de hardening para Ubuntu

```bash
#!/bin/bash
# Deshabilita login como root
passwd -l root
# Desactiva acceso por contraseña
sed -i 's/^#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
# Configura UFW
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp
ufw allow 443/tcp
ufw enable
# Reinicia SSH
systemctl restart sshd
```

---

## 🧱 Firewalls internos (host-based)

No dependas solo del SG/NSG del proveedor. **También debes configurar firewall dentro del sistema operativo**.

| OS             | Comando base                       |
| -------------- | ---------------------------------- |
| Linux (Ubuntu) | `ufw` o `iptables`                 |
| Windows        | `New-NetFirewallRule` (PowerShell) |

> 🎯 Objetivo: **defensa en profundidad**. Si alguien manipula las reglas externas, aún tienes control local.

---

## 📈 Logs y auditoría

Habilita y redirige los logs de seguridad para detección de incidentes y cumplimiento normativo:

| Tipo de log        | Ejemplos de herramientas                   |
| ------------------ | ------------------------------------------ |
| SSH / Auth         | `/var/log/auth.log`, `/var/log/secure`     |
| Firewall / Red     | `ufw`, `iptables`, Windows Firewall        |
| Cambios en sistema | `auditd`, `syslog`, `journalctl`           |
| Cloud-native       | AWS CloudWatch Agent / Azure Log Analytics |

---

## 🧪 Monitoreo del estado de hardening

### 🔍 En AWS

* **AWS Inspector** puede evaluar instancias EC2 contra benchmarks como **CIS**.
* **AWS Systems Manager** puede ejecutar scripts de comprobación y remediación.

```bash
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --instance-ids i-abc123 \
  --parameters 'commands=["/usr/bin/auditctl -l"]'
```

### 🔍 En Azure

* **Microsoft Defender for Servers** analiza VMs y detecta configuraciones débiles.
* **Azure Policy** permite aplicar reglas como “Deshabilitar cuenta root” o “Configurar antivirus”.

---

## 🏢 Caso empresarial

**Escenario**: una consultora legal despliega VMs para almacenar documentación sensible de clientes. Exige cumplimiento con ENS/CIS.

| Requisito                            | Solución recomendada                                   |
| ------------------------------------ | ------------------------------------------------------ |
| Desactivar root y contraseñas        | Script de cloud-init con SSH key only                  |
| Solo acceso desde IP interna         | NSG/SG + iptables con reglas explícitas                |
| Monitorear cambios en sistema        | Auditd + envío a CloudWatch o Log Analytics            |
| Antivirus / IDS                      | Amazon Inspector / Defender for Endpoint               |
| Validar configuración periódicamente | CIS Inspector (AWS) / Azure Security Center (Defender) |

---

## ❌ Errores comunes

1. Usar imágenes de terceros sin inspección ni validación.
2. Dejar habilitado el acceso SSH/RDP sin protección ni bastión.
3. No aplicar firewall local (ufw, iptables, Windows Defender).
4. No rotar claves ni eliminar usuarios innecesarios.
5. Configurar servicios de base de datos o API expuestos por defecto.
6. Ignorar logs: no recoger, no centralizar, no monitorear.

---

## ✅ Checklist rápido de hardening

| Componente                   | Acción recomendada                          |
| ---------------------------- | ------------------------------------------- |
| 🔐 Usuario root              | Deshabilitado, solo acceso vía `sudo`       |
| 🔑 Autenticación             | Solo clave SSH, sin contraseñas             |
| 🔥 Firewall interno          | Activado con reglas mínimas necesarias      |
| 📦 Servicios                 | Solo los imprescindibles habilitados        |
| 📊 Logs                      | Habilitados y enviados a plataforma externa |
| 🔍 Auditoría de cambios      | Monitorización activa (auditd, Defender)    |
| 🛡️ Antivirus / Endpoint Sec | Instalado y actualizado                     |
| 🧰 Hardening automatizado    | Scripts de bootstrap, Ansible, Terraform    |

---

## 🔗 Recursos recomendados

* [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
* [AWS EC2 Hardening Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/security-best-practices.html)
* [AWS Inspector Overview](https://docs.aws.amazon.com/inspector/latest/user/inspector_introduction.html)
* [Azure VM Security Best Practices](https://learn.microsoft.com/en-us/azure/virtual-machines/security-overview)
* [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction)