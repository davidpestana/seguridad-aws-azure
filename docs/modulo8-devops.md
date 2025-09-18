## ⚙️ Módulo 8: Seguridad en Aplicaciones y DevOps (AWS y Azure)

---

## 🎯 Objetivos del módulo

Este módulo conecta la **seguridad en el desarrollo de software** con la **operativa cloud**, aplicando principios de **DevSecOps**. Se centra en cómo asegurar pipelines de CI/CD, integrar análisis de seguridad automatizados y evitar vulnerabilidades antes de llegar a producción.

Al finalizar, serás capaz de:

* Comprender qué es DevSecOps y cómo se integra en CI/CD.
* Construir pipelines en AWS (CodePipeline/CodeBuild) y Azure (DevOps/GitHub Actions) con validaciones de seguridad.
* Integrar análisis estático (SAST), dinámico (DAST) y de dependencias en el ciclo de desarrollo.
* Configurar despliegues seguros y controlar secretos en pipelines.
* Adoptar buenas prácticas en integración continua, despliegue continuo y gestión de vulnerabilidades.

---

## 🧭 Índice detallado del módulo

---

### 1) Introducción a DevSecOps

✅ Objetivo
Entender la filosofía DevSecOps: seguridad como parte del ciclo de desarrollo, no como fase final.

📌 Claves
Shift Left · Cultura DevSecOps · Automación · Seguridad continua

🧩 Contenido

* **Shift Left**: mover controles de seguridad al inicio del ciclo (repositorio, código, dependencias).
* **Automatización**: seguridad integrada en pipelines, no tareas manuales aisladas.
* **Cultura**: seguridad como responsabilidad compartida, no solo del equipo de seguridad.
* Diferencia DevOps vs DevSecOps: pasar de “velocidad” a “velocidad + seguridad”.
  Ejemplo: detectar dependencias vulnerables en un `requirements.txt` antes de desplegar una Lambda o App Service.

📚 docs/modulo8-devops/01-introduccion.md

---

### 2) Integración continua segura

✅ Objetivo
Aplicar controles de seguridad en pipelines de integración continua.

📌 Claves
SAST · Dependencias · Code Review · Secrets scanning

🧩 Contenido

* **Análisis estático (SAST)**: detectar vulnerabilidades en código (ej. SonarQube, CodeQL).
* **Escaneo de dependencias**: comprobar librerías frente a CVEs (Dependabot, Trivy, pip-audit).
* **Secrets scanning**: evitar claves/API en repositorios (GitHub secret scanning, git-secrets).
* **Code review con checklist de seguridad**: obligatoriedad de PR revisados.

Caso práctico: pipeline en GitHub Actions que ejecuta linting, test y análisis de seguridad antes de merge.

📚 docs/modulo8-devops/02-integracion-continua.md

---

### 3) Despliegue seguro

✅ Objetivo
Configurar despliegues que minimicen exposición y eviten malas configuraciones.

📌 Claves
IaC seguro · Validaciones previas · Configuración mínima

🧩 Contenido

* **Infraestructura como Código (IaC)**: usar Terraform/ARM/CloudFormation con validación de seguridad (Checkov, tfsec).
* Validaciones de configuración antes de aplicar (ej. detectar SG con `0.0.0.0/0`).
* Deploy con **principio de menor privilegio**: credenciales mínimas en el pipeline.
* Segregar entornos: dev/test/prod con políticas diferentes.
  Ejemplo: CodePipeline en AWS que despliega solo si las validaciones tfsec son aprobadas.

📚 docs/modulo8-devops/03-despliegue-seguro.md

---

### 4) Seguridad en pipelines (credenciales y secretos)

✅ Objetivo
Proteger los pipelines de CI/CD contra fugas de credenciales y accesos indebidos.

📌 Claves
Secrets management · Vaults · IAM Roles / Managed Identities · GitHub OIDC

🧩 Contenido

* **Gestión de secretos**:

  * AWS Secrets Manager o SSM Parameter Store.
  * Azure Key Vault.
* Eliminar secretos en repositorios/pipelines → usar **OIDC + IAM roles** (GitHub → AWS) o **Federated Credentials** (GitHub → Azure AD).
* Rotación automática de credenciales.
* Casos reales: fugas de tokens de CI en repositorios públicos.

Ejemplo: GitHub Actions accediendo a AWS con OIDC sin usar claves estáticas.

📚 docs/modulo8-devops/04-seguridad-en-pipelines.md

---

### 5) Escaneo y validación de aplicaciones

✅ Objetivo
Automatizar pruebas de seguridad en el pipeline para detectar vulnerabilidades antes de desplegar.

📌 Claves
DAST · SCA · Contenedores · IaC scanning

🧩 Contenido

* **DAST (Dynamic Application Security Testing)**: testear la aplicación en ejecución (ej. OWASP ZAP).
* **SCA (Software Composition Analysis)**: auditar librerías de terceros.
* **Escaneo de contenedores**: usar Trivy, Grype, Dockle.
* **Escaneo de IaC**: detectar configuraciones inseguras en Terraform/ARM.

Caso práctico: en CodeBuild/Azure DevOps pipeline, ejecutar Trivy sobre la imagen antes de publicar en ECR/ACR.

📚 docs/modulo8-devops/05-escaneo-y-validacion.md

---

### 6) DevSecOps en práctica

✅ Objetivo
Integrar la seguridad en todo el ciclo DevOps como proceso continuo.

📌 Claves
Shift Left · Automación · Feedback continuo · KPIs

🧩 Contenido

* Seguridad como **pipeline continuo**, no tareas aisladas.
* Integración de análisis estáticos, dinámicos, de dependencias y contenedores.
* Feedback rápido al desarrollador.
* KPIs: % de builds fallidos por seguridad, tiempo de corrección de vulnerabilidades.
* Ejemplo: pipeline GitHub Actions → lint + test + análisis SAST + escaneo contenedor + despliegue seguro.

📚 docs/modulo8-devops/06-devsecops.md

---

### 7) Buenas prácticas en seguridad de aplicaciones y DevOps

✅ Objetivo
Consolidar prácticas que eviten errores comunes y aseguren el ciclo completo de desarrollo.

📌 Claves
Principio de mínimo privilegio · Segregación de entornos · Revisiones periódicas

🧩 Contenido

* Definir **políticas claras** de revisión de código y seguridad.
* Usar secretos rotativos y gestionados (no hardcodeados).
* Validar IaC con herramientas automáticas antes de aplicar.
* Escanear dependencias y contenedores de forma periódica.
* Segregar entornos (dev/test/prod) y aplicar políticas diferenciadas.
* Documentar flujos de seguridad y entrenar a los equipos.

Errores comunes:

* Claves estáticas en pipelines.
* Permisos excesivos para pipelines (admin global en vez de permisos mínimos).
* “Security scans” desactivados por ralentizar builds.

📚 docs/modulo8-devops/07-buenas-practicas.md