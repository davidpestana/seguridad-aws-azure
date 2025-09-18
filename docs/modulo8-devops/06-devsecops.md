# 🔄 DevSecOps en Práctica

---

## 🧭 Introducción – Qué es y por qué importa

Hasta ahora hemos visto piezas individuales del enfoque DevSecOps (SAST, DAST, SCA, gestión de secretos…).
En este archivo vamos a **conectar todas esas piezas** para construir una **cadena de seguridad continua**, totalmente integrada en los pipelines.

> DevSecOps no es una herramienta, sino una **práctica continua de seguridad integrada** en el desarrollo y la entrega.

---

## 📚 Explicación detallada

### 🔁 DevSecOps ≠ Seguridad al final

DevSecOps propone que la seguridad:

* Se **automatice** desde la primera línea de código.
* Se **valide continuamente** a lo largo del pipeline.
* Se convierta en **responsabilidad compartida**.

---

### 🔄 Fases típicas en un pipeline DevSecOps

1. **Commit**

   * Escaneo de código (SAST).
   * Secret scanning.
   * Linting.

2. **Build**

   * Escaneo de dependencias (SCA).
   * Validación de IaC.
   * Build de contenedor.

3. **Test**

   * Test unitarios.
   * Escaneo de contenedores.
   * Análisis de configuración.

4. **Deploy**

   * Validación del plan IaC (`terraform plan`, `bicep what-if`).
   * Deploy en entorno de staging.
   * DAST (análisis en ejecución).

5. **Monitorización**

   * Recolección de métricas de seguridad.
   * Alertas por findings de pipelines.
   * KPIs de cumplimiento.

---

### 🧱 Componentes clave del ciclo DevSecOps

| Componente     | Herramientas comunes               |
| -------------- | ---------------------------------- |
| SAST           | SonarQube, CodeQL, Bandit          |
| SCA            | pip-audit, npm audit, Dependabot   |
| IaC Scan       | tfsec, checkov                     |
| Secret scan    | GitHub Secret Scanning, truffleHog |
| Container scan | Trivy, Dockle, Grype               |
| DAST           | OWASP ZAP, Burp Suite              |
| Monitorización | Grafana, ELK, AWS Security Hub     |

---

### 📈 KPIs para medir la madurez DevSecOps

* % de pipelines que ejecutan al menos 1 control de seguridad.
* Tiempo medio de respuesta a vulnerabilidades (`time to patch`).
* Nº de builds fallidos por findings críticos.
* Tasa de exposición de secretos (medido por detecciones en PRs).
* Cumplimiento por entorno (dev/test/prod).

---

## 💼 Ejemplo empresarial

### Contexto

Una organización pública quiere garantizar que todo el software que despliega cumple requisitos mínimos de seguridad. Los equipos trabajan con distintos lenguajes y pipelines (GitHub Actions, Azure DevOps, Jenkins...).

### Implementación DevSecOps:

1. Se define una **plantilla mínima** de pipeline DevSecOps con pasos obligatorios:

   * SCA (Dependabot + CLI).
   * SAST (SonarQube o CodeQL).
   * Escaneo de contenedores (Trivy).
   * Validación IaC (tfsec).
   * Secret scanning (activado por defecto).

2. Se automatiza:

   * Notificación por Slack/Teams al detectar findings críticos.
   * Registro centralizado de resultados en Elastic o CloudWatch.

3. Se miden KPIs trimestralmente:

   * Builds seguros.
   * Tiempo de resolución.
   * Cumplimiento por equipo.

Resultado:
✔️ Menor carga operativa en equipos de seguridad.
✔️ Entregas más rápidas, seguras y auditables.
✔️ Cultura de seguridad integrada en cada equipo.

---

## ⚙️ Código / Configuración

### 🚦 Ejemplo de pipeline DevSecOps con GitHub Actions

```yaml
# .github/workflows/devsecops.yml
name: DevSecOps Pipeline

on:
  pull_request:
    branches: [main]

jobs:
  devsecops:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install bandit pip-audit

      - name: Lint
        run: pylint app/

      - name: Run unit tests
        run: pytest

      - name: SAST (Bandit)
        run: bandit -r app/ -ll

      - name: SCA (pip-audit)
        run: pip-audit -r requirements.txt

      - name: Build Docker image
        run: docker build -t myapp:latest .

      - name: Container Scan (Trivy)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:latest
          format: table
          exit-code: 1
          severity: CRITICAL,HIGH

      - name: IaC Scan (tfsec)
        uses: aquasecurity/tfsec-action@v1.0.0
```

> 🔐 Resultado: cualquier paso crítico fallará el pipeline.

---

## 🧪 Tips para probar y validar

* Ejecuta el pipeline con dependencias vulnerables → verifica que el build se bloquea.
* Substituye `myapp` por una imagen con CVEs conocidos → observa el resultado de Trivy.
* Añade un `aws_security_group` con apertura total → tfsec lo detectará.
* Simula una app con XSS y ejecuta OWASP ZAP → comprueba detección.

---

## ⚠️ Errores comunes

| Error                                       | Impacto                                 |
| ------------------------------------------- | --------------------------------------- |
| Añadir “scans” pero no bloquear el pipeline | Findings ignorados                      |
| Usar scans decorativos sin informes ni logs | Falta trazabilidad                      |
| No escanear en staging/prod                 | Exposición fuera del entorno de pruebas |
| No usar alertas o KPIs                      | Imposible evaluar eficacia              |
| Falta de consenso en los equipos            | Cada equipo aplica lo que quiere        |

---

## ✅ Buenas prácticas

✔️ Define una plantilla de pipeline DevSecOps mínima.
✔️ Aplica los mismos controles en todas las ramas importantes.
✔️ Recolecta logs y resultados de escaneo de forma centralizada.
✔️ Implementa alertas por findings críticos.
✔️ Mide el éxito con métricas claras.
✔️ Promueve la cultura DevSecOps en todos los equipos (no imponerla desde seguridad).

---

## 🔗 Recursos recomendados

* [📘 OWASP DevSecOps Maturity Model](https://owasp.org/www-project-devsecops-maturity-model/)
* [🛠️ GitHub Actions hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
* [📚 AWS DevSecOps Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/principles-devsecops/principles-devsecops.pdf)
* [🧠 Azure DevOps Security Guidance](https://learn.microsoft.com/en-us/azure/devops/learn/devops-at-microsoft/security)
* [🧩 Trivy, tfsec, ZAP GitHub integrations](https://github.com/aquasecurity)