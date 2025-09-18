# 🔍 Escaneo y Validación de Aplicaciones

---

## 🧭 Introducción – Qué es y por qué importa

Una aplicación puede estar correctamente desarrollada y desplegada, pero seguir siendo **vulnerable** si:

* Utiliza librerías de terceros con fallos.
* Tiene configuraciones inseguras en contenedores o IaC.
* Presenta fallos lógicos que solo se detectan en tiempo de ejecución.

Por eso, los pipelines modernos deben incluir **escaneos automáticos**:

* Antes de empaquetar (SAST, SCA).
* Al generar artefactos (escaneo de contenedores).
* Después del despliegue en staging (DAST).

---

## 📚 Explicación detallada

### 🧰 Tipos de escaneo y validación

#### 1. **DAST – Análisis Dinámico de Aplicaciones**

> Simula ataques desde el exterior mientras la app está en ejecución.

* Ideal para detectar XSS, CSRF, inyecciones, errores en cabeceras, etc.
* Herramientas: **OWASP ZAP**, **Burp Suite**, **Nikto**.
* Se ejecuta contra entornos de staging o test.

---

#### 2. **SCA – Software Composition Analysis**

> Audita las dependencias (npm, pip, Maven…) en busca de CVEs conocidas.

* Herramientas: **Dependabot**, **pip-audit**, **npm audit**, **Snyk**, **GitHub Advisory Database**.
* Detecta vulnerabilidades **antes de construir** o desplegar.

---

#### 3. **Escaneo de contenedores**

> Verifica que las imágenes Docker no incluyan:

* Paquetes obsoletos o vulnerables.

* Ficheros innecesarios o secretos.

* Configuraciones inseguras.

* Herramientas: **Trivy**, **Grype**, **Dockle**, **Clair**.

---

#### 4. **Escaneo de Infraestructura como Código (IaC)**

> Audita ficheros Terraform, Bicep, CloudFormation…

* Detección de:

  * Recursos mal configurados (SGs abiertos, roles excesivos).
  * Falta de encriptación.
  * Exposición pública no deseada.

* Herramientas: **tfsec**, **checkov**, **cfn-lint**, **PSRule**, **tflint**.

---

## 💼 Ejemplo empresarial

### Contexto

Una empresa SaaS despliega contenedores en EKS mediante CodeBuild. Al no escanear sus imágenes, suben una versión con `log4j` vulnerable y comandos `curl` instalados.

### Solución DevSecOps

* Se integra **Trivy** y **Dockle** en el pipeline de CodeBuild.
* Antes de publicar la imagen en ECR:

  * Se escanea la imagen.
  * Si contiene vulnerabilidades críticas → **build fallido**.
* Se automatiza análisis DAST en staging con **OWASP ZAP**.
* Se hace obligatorio el análisis SCA con Dependabot + CLI.

Resultado:
✔️ Imágenes libres de CVEs conocidos.
✔️ Validación IaC + contenedores + código en cada PR.
✔️ Análisis en tiempo de ejecución antes de ir a producción.

---

## ⚙️ Código / Configuración

### 🐳 Escaneo de imágenes Docker con Trivy en GitHub Actions

```yaml
# .github/workflows/container-scan.yml
name: Scan Docker Image

on:
  push:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Build image
        run: docker build -t my-app:latest .

      - name: Scan with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: my-app:latest
          format: table
          exit-code: 1  # Falla el job si hay CVEs de alto nivel
          severity: CRITICAL,HIGH
```

---

### ☁️ Escaneo DAST con OWASP ZAP CLI (en staging)

```bash
zap-cli start
zap-cli open-url http://staging.miapp.com
zap-cli spider http://staging.miapp.com
zap-cli active-scan http://staging.miapp.com
zap-cli report -o zap-report.html -f html
```

> 💡 Se puede automatizar en un pipeline y bloquear el despliegue si hay findings críticos.

---

### 🌐 SCA con pip-audit en el pipeline

```bash
pip install pip-audit
pip-audit -r requirements.txt
```

> Detecta CVEs en librerías como `requests`, `urllib3`, `Flask`, etc.

---

### 🔐 Validación IaC con tfsec

```bash
tfsec ./infra

# o en GitHub Actions
- name: Run tfsec
  uses: aquasecurity/tfsec-action@v1.0.0
```

---

## 🧪 Tips para probar

* Intenta construir una imagen con `openssl` desactualizado → Trivy lo detectará.
* Añade una librería vulnerable (`urllib3==1.25`) y ejecuta `pip-audit`.
* Despliega una app en staging con formulario sin validación y escanea con ZAP.
* En IaC, crea un `aws_security_group` con `0.0.0.0/0` y escanéalo con tfsec.

---

## ⚠️ Errores comunes

| Error                                   | Riesgo asociado                          |
| --------------------------------------- | ---------------------------------------- |
| Construir imágenes sin escaneo          | CVEs en producción                       |
| Ignorar findings de SCA                 | Vulnerabilidades conocidas sin parche    |
| No escanear IaC antes de aplicar        | Recursos inseguros en la nube            |
| Saltarse DAST                           | Fallos en runtime no detectados          |
| Ejecutar scans sin bloquear el pipeline | Análisis “decorativos”, sin impacto real |

---

## ✅ Buenas prácticas

✔️ Automatiza escaneos de contenedores, código, IaC y apps.
✔️ Integra Trivy, ZAP, tfsec o similares **en el pipeline**.
✔️ Falla el build si hay vulnerabilidades críticas.
✔️ Escanea también en staging antes de ir a producción.
✔️ Usa formatos de reporte (HTML, SARIF) para integrarlos en dashboards.

---

## 🔗 Recursos recomendados

* [🔍 Trivy – Container & IaC scanner](https://aquasecurity.github.io/trivy/)
* [🛠️ OWASP ZAP](https://www.zaproxy.org/)
* [📦 pip-audit](https://pypi.org/project/pip-audit/)
* [🛡️ tfsec](https://aquasecurity.github.io/tfsec/)
* [📘 GitHub Container Scanning](https://docs.github.com/en/code-security/supply-chain-security/keeping-your-dependencies-updated/automatically-scanning-packages-for-vulnerabilities)
* [🔒 Dockle – Dockerfile linter](https://github.com/goodwithtech/dockle)