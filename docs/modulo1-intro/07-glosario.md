# 📄 `07-glosario.md`

📁 Ruta sugerida: `modulo1-intro/07-glosario.md`

---

## 🧭 Introducción – Glosario de términos clave en seguridad cloud

Este glosario recopila los **conceptos, acrónimos y términos esenciales** para comprender y trabajar con seguridad en la nube de forma efectiva.

Muchos de estos términos son comunes a AWS y Azure, pero con **nombres diferentes o enfoques particulares** según la plataforma.
Por ello, este documento incluye:

* Definiciones claras y breves.
* Equivalencias AWS ↔ Azure cuando apliquen.
* Clasificación temática para facilitar su consulta.
* Ejemplos contextualizados para su aplicación.

Este glosario servirá como **referencia rápida** durante todo el curso y también para futuras auditorías, proyectos o certificaciones.

---

## 🔐 Gestión de identidades y acceso

| Término        | Definición                                                      | AWS                    | Azure                                         |
| -------------- | --------------------------------------------------------------- | ---------------------- | --------------------------------------------- |
| **IAM**        | Sistema de gestión de identidades y accesos basado en políticas | IAM                    | Azure AD + RBAC                               |
| **RBAC**       | Control de acceso basado en roles                               | IAM roles/policies     | Role-Based Access Control (RBAC)              |
| **MFA**        | Autenticación multifactor: añade una capa adicional al login    | IAM MFA                | Conditional Access + MFA                      |
| **STS**        | Servicio de generación de credenciales temporales               | Security Token Service | Privileged Identity Management (PIM)          |
| **Federación** | Permitir acceso desde identidad externa (ej. Google, SAML, AD)  | SAML/OIDC Federation   | Azure AD Enterprise Apps, Identity Federation |

---

## 🔒 Cifrado y protección de datos

| Término                 | Definición                                            | AWS                                             | Azure                                |
| ----------------------- | ----------------------------------------------------- | ----------------------------------------------- | ------------------------------------ |
| **Cifrado en tránsito** | Protege datos durante su transmisión (ej. HTTPS, TLS) | Activado por defecto en la mayoría de servicios | Activado por defecto                 |
| **Cifrado en reposo**   | Protege datos almacenados en disco o base de datos    | KMS / SSE-S3 / SSE-KMS                          | Azure Storage Encryption / TDE / CMK |
| **KMS**                 | Servicio de gestión de claves y cifrado               | AWS KMS                                         | Azure Key Vault                      |
| **CMK**                 | Claves gestionadas por el cliente para cifrado        | Customer Managed Keys (KMS)                     | Customer Managed Keys (Key Vault)    |
| **Secrets Management**  | Almacenamiento seguro de claves, tokens, secretos     | AWS Secrets Manager                             | Azure Key Vault Secrets              |

---

## 📊 Auditoría, monitoreo y trazabilidad

| Término       | Definición                                                     | AWS                             | Azure                                 |
| ------------- | -------------------------------------------------------------- | ------------------------------- | ------------------------------------- |
| **Auditoría** | Registro de acciones y eventos relevantes para la seguridad    | CloudTrail                      | Activity Logs                         |
| **Logs**      | Registros estructurados de eventos (aplicación, red, SO, etc.) | CloudWatch Logs                 | Azure Monitor Logs                    |
| **SIEM**      | Sistema de gestión de eventos e información de seguridad       | Integración con Security Hub    | Microsoft Sentinel                    |
| **Config**    | Seguimiento de configuraciones, cambios y cumplimiento         | AWS Config                      | Azure Resource Graph / Change History |
| **Alertas**   | Notificaciones automáticas ante eventos críticos o anómalos    | CloudWatch Alarms / EventBridge | Azure Monitor Alerts                  |

---

## 🛡️ Protección activa y amenazas

| Término                 | Definición                                           | AWS                          | Azure                               |
| ----------------------- | ---------------------------------------------------- | ---------------------------- | ----------------------------------- |
| **DDoS**                | Ataque de denegación de servicio distribuido         | AWS Shield                   | Azure DDoS Protection               |
| **WAF**                 | Firewall de aplicaciones web                         | AWS WAF                      | Azure WAF                           |
| **IDS/IPS**             | Sistemas de detección/prevención de intrusos         | GuardDuty / Network Firewall | Defender for Cloud / Azure Firewall |
| **Threat Intelligence** | Base de datos de IPs y patrones maliciosos conocidos | GuardDuty / Macie            | Microsoft Threat Intelligence       |

---

## ⚖️ Cumplimiento y gobierno

| Término              | Definición                                                   | AWS                            | Azure                           |
| -------------------- | ------------------------------------------------------------ | ------------------------------ | ------------------------------- |
| **Compliance**       | Cumplimiento de normativas (PCI, ISO, GDPR…)                 | AWS Compliance Center          | Microsoft Trust Center          |
| **Security Score**   | Métrica para evaluar postura de seguridad                    | Security Hub / Trusted Advisor | Defender for Cloud Secure Score |
| **Políticas**        | Reglas que limitan acciones o configuran comportamiento      | IAM SCP / AWS Config Rules     | Azure Policy                    |
| **Blueprints**       | Paquetes de recursos preconfigurados con seguridad integrada | -                              | Azure Blueprints                |
| **Well-Architected** | Framework para evaluar buenas prácticas en la nube           | Well-Architected Framework     | CAF (Cloud Adoption Framework)  |

---

## 🧱 Arquitectura segura

| Término                               | Definición                                                    | AWS                              | Azure                                      |
| ------------------------------------- | ------------------------------------------------------------- | -------------------------------- | ------------------------------------------ |
| **Zero Trust**                        | Modelo de seguridad que no confía en ningún actor por defecto | IAM + SCP + Logging + Encryption | Identity + Conditional Access + Monitoring |
| **Principio de mínimo privilegio**    | Dar solo los permisos necesarios, nada más                    | IAM Policy Granular              | RBAC personalizado                         |
| **Segmentación de red**               | Separar y aislar recursos sensibles en subredes distintas     | VPC, subnets, NACL, SG           | VNet, NSG, subnets                         |
| **Multi-account**                     | Uso de varias cuentas cloud para aislar entornos o dominios   | AWS Organizations                | Azure Management Groups + Subs             |
| **Infraestructura como código (IaC)** | Declarar recursos y configuración en archivos versionables    | CloudFormation / Terraform       | Bicep / Terraform                          |

---

## 🧠 Otros conceptos clave

| Término                                     | Definición                                                                                    |
| ------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Postura de seguridad (Security posture)** | Nivel actual de seguridad de una organización o entorno cloud.                                |
| **DevSecOps**                               | Integración de la seguridad en el ciclo de vida DevOps, desde el diseño hasta la operación.   |
| **Drift**                                   | Divergencia entre la configuración real de los recursos y la declarada en código o políticas. |
| **Hardening**                               | Proceso de reforzar sistemas, servicios y configuraciones para minimizar riesgos.             |
| **Misconfiguración**                        | Configuración incorrecta o incompleta de recursos que puede exponer vulnerabilidades.         |

---

## 🔍 Recomendaciones para el uso del glosario

✔️ **Durante el curso**:

* Consulta este glosario cada vez que aparezca un concepto nuevo o siglas poco conocidas.
* Compara los enfoques AWS ↔ Azure para entender cómo varían las implementaciones.

✔️ **Después del curso**:

* Usa este glosario como parte de documentación interna de equipos técnicos.
* Añade términos nuevos según se incorporen nuevas tecnologías (ej. IA generativa, confidential computing…).

---

## 📚 Recursos complementarios

| Recurso                                                                                    | Descripción                                                       |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| [AWS Glossary](https://docs.aws.amazon.com/general/latest/gr/glos-chap.html)               | Glosario oficial de términos AWS                                  |
| [Azure Glossary](https://learn.microsoft.com/en-us/azure/azure-glossary-cloud-terminology) | Glosario oficial de términos Azure                                |
| [OWASP Glossary](https://owasp.org/www-project-glossary/)                                  | Glosario centrado en seguridad de aplicaciones y vulnerabilidades |
| [Microsoft Security Concepts](https://learn.microsoft.com/en-us/security/)                 | Conceptos clave de seguridad en el ecosistema Microsoft           |
