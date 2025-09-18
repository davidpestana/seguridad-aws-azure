# 📄 `06-recursos-oficiales.md`


## 🧭 Introducción – Recursos oficiales y lecturas recomendadas

En un entorno en constante evolución como la seguridad cloud, es fundamental contar con **fuentes fiables y actualizadas** para:

* Ampliar conocimientos más allá del curso.
* Sustentar decisiones de diseño y arquitectura.
* Prepararse para auditorías o certificaciones.
* Mantenerse al día con nuevas amenazas y actualizaciones de servicios.

Tanto **AWS como Azure** proporcionan extensas bibliotecas de documentación, whitepapers, herramientas de evaluación, benchmarks de seguridad y rutas de formación.
Este archivo recopila y categoriza los **recursos más relevantes**, con enlaces oficiales comentados y sugerencias de uso práctico.

---

## 📚 Documentación oficial de referencia

### 📘 AWS – Portales clave

| Recurso                                              | Descripción                                                         | Enlace                                                                          |
| ---------------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **AWS Security Documentation**                       | Guía principal de servicios y prácticas de seguridad en AWS.        | [→ Enlace](https://docs.aws.amazon.com/security/)                               |
| **AWS IAM Documentation**                            | Manual completo sobre gestión de identidades y accesos.             | [→ Enlace](https://docs.aws.amazon.com/IAM/latest/UserGuide/)                   |
| **AWS Well-Architected Framework – Security Pillar** | Pilares para diseñar infraestructuras seguras.                      | [→ Enlace](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/) |
| **AWS Compliance Center**                            | Certificaciones, estándares (ISO, PCI, NIST) y guías por industria. | [→ Enlace](https://aws.amazon.com/compliance/)                                  |

🔍 Úsalos para validar configuraciones, encontrar ejemplos detallados y consultar limitaciones de servicios.

---

### 📘 Azure – Portales clave

| Recurso                                 | Descripción                                                   | Enlace                                                                                                 |
| --------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Azure Security Documentation**        | Guía base para proteger recursos en Azure.                    | [→ Enlace](https://learn.microsoft.com/en-us/security/azure-security/)                                 |
| **Microsoft Entra ID (antes Azure AD)** | Documentación sobre identidad, autenticación y RBAC.          | [→ Enlace](https://learn.microsoft.com/en-us/entra/)                                                   |
| **Azure Security Benchmark (ASB)**      | Buenas prácticas organizadas por control (IAM, red, apps...). | [→ Enlace](https://learn.microsoft.com/en-us/security/benchmark/azure/)                                |
| **Azure Trust Center**                  | Certificaciones, cumplimiento normativo y gobierno.           | [→ Enlace](https://learn.microsoft.com/en-us/microsoft-365/compliance/microsoft-trust-center-overview) |

🔍 Incluye evaluaciones cruzadas con NIST, ISO, CIS y otras normativas globales.

---

## 🧪 Herramientas y benchmarks prácticos

### ✅ Estándares y guías independientes

| Recurso                      | Descripción                                                                               | Enlace                                                                 |
| ---------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **CIS Benchmarks**           | Configuraciones seguras validadas por la comunidad (VMs, servicios cloud, contenedores…). | [→ Enlace](https://www.cisecurity.org/benchmarks)                      |
| **NIST SP 800-53 / 800-171** | Estándares de seguridad para entornos críticos, adoptados por gobiernos y empresas.       | [→ Enlace](https://csrc.nist.gov/publications/sp)                      |
| **ISO 27001**                | Norma internacional para sistemas de gestión de seguridad de la información.              | [→ Enlace](https://www.iso.org/isoiec-27001-information-security.html) |

🔧 Muchos CSPM (Cloud Security Posture Management) y herramientas nativas como AWS Config o Defender for Cloud permiten auditar contra estos estándares automáticamente.

---

## 📈 Rutas de formación y autoaprendizaje

### 🎓 AWS Training y Learning Paths

| Recurso                          | Descripción                                                                       | Enlace                                                                                                    |
| -------------------------------- | --------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **AWS Security Learning Path**   | Ruta oficial de formación en seguridad (desde fundamentos hasta especialización). | [→ Enlace](https://www.aws.training/Details/Video?id=15861)                                               |
| **AWS Skill Builder – Security** | Cursos interactivos gratuitos y de pago, con laboratorios.                        | [→ Enlace](https://skillbuilder.aws/)                                                                     |
| **AWS Ramp-Up Guide – Security** | Guía PDF con cursos, whitepapers y certificaciones.                               | [→ Enlace](https://d1.awsstatic.com/training-and-certification/ramp-up-guides/Ramp-Up_Guide_Security.pdf) |

---

### 🎓 Microsoft Learn – Seguridad en Azure

| Recurso                                      | Descripción                                                                  | Enlace                                                                                                           |
| -------------------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **Microsoft Learn – Seguridad en Azure**     | Rutas de aprendizaje gratuitas con labs interactivos.                        | [→ Enlace](https://learn.microsoft.com/en-us/training/paths/secure-azure-resources/)                             |
| **Microsoft Security Certifications**        | Exámenes SC-900, SC-200, SC-300 (Microsoft Security, Identity & Compliance). | [→ Enlace](https://learn.microsoft.com/en-us/certifications/)                                                    |
| **Microsoft CAF (Cloud Adoption Framework)** | Buenas prácticas de seguridad organizativa y técnica.                        | [→ Enlace](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/security) |

---

## 🛡️ Recomendaciones por perfil

| Perfil                                 | Recursos recomendados                                                        |
| -------------------------------------- | ---------------------------------------------------------------------------- |
| **Desarrollador cloud**                | Azure Security Benchmark · IAM Best Practices (AWS) · Microsoft Learn Labs   |
| **Ingeniero DevSecOps**                | CIS Benchmarks · AWS Config · Azure Policy Docs · Terraform security modules |
| **CISO / Responsable de cumplimiento** | AWS Compliance Center · Azure Trust Center · NIST SP 800-53                  |
| **Alumno que quiere certificarse**     | Microsoft Learn: SC-900 · AWS Ramp-Up Guide · SkillBuilder AWS Security      |

---

## ✅ Buenas prácticas al usar estos recursos

✔️ Para aprovechar al máximo estas fuentes:

* 📌 Guarda los portales de AWS y Azure en marcadores por categoría (identidad, red, cumplimiento…).
* 🧭 Usa los benchmarks (CIS, ASB) como base para revisiones de seguridad periódicas.
* 📝 Mantén un documento de referencia viva con los enlaces clave para tu equipo.
* 🎓 Haz seguimiento de rutas de aprendizaje en función de tu perfil y necesidades.

---

## 📚 Repositorio de acceso rápido (recomendado)

Se sugiere crear un archivo de bookmarks o README centralizado con acceso rápido a:

```
📂 /docs/seguridad/
├── aws/
│   ├── aws-security-docs.md
│   ├── well-architected-security.md
│   └── compliance-standards.md
├── azure/
│   ├── azure-security-docs.md
│   ├── azure-security-benchmark.md
│   └── trust-center.md
├── estandares/
│   ├── cis.md
│   ├── nist.md
│   └── iso27001.md
├── formacion/
│   ├── aws-training.md
│   └── microsoft-learn.md
```

---

## 🧠 Conclusión

Tener una **biblioteca curada de recursos de seguridad cloud** no es opcional: es una parte activa de tu estrategia de protección.
Ya seas desarrollador, administrador o responsable de cumplimiento, estos recursos te permitirán tomar decisiones más seguras, justificadas y alineadas con las mejores prácticas globales.
