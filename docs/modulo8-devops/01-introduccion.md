# 🔐 Introducción a DevSecOps

---

## 🧭 Introducción – Qué es y por qué importa

Tradicionalmente, la seguridad se ha tratado como una **etapa final** en el ciclo de desarrollo. Esto implicaba que las vulnerabilidades se descubrían **tarde**, justo antes de pasar a producción, lo cual ralentizaba la entrega o, peor aún, dejaba brechas abiertas.

**DevSecOps** propone un cambio de mentalidad:

> **"Integrar la seguridad desde el inicio del desarrollo (shift left), de forma automatizada, continua y compartida por todo el equipo."**

Este enfoque mejora:

* La **velocidad** de entrega segura.
* La **detención temprana de errores** de seguridad.
* La **responsabilidad colectiva** sobre la seguridad.

---

## 📚 Explicación detallada

### 🏗️ ¿Qué es DevSecOps?

DevSecOps es la evolución natural de DevOps, integrando la seguridad como una parte nativa del ciclo de desarrollo y entrega de software.
En lugar de ser un “freno”, la seguridad se convierte en un **acelerador confiable**, si se implementa con automatización.

| Enfoque tradicional         | Enfoque DevSecOps                         |
| --------------------------- | ----------------------------------------- |
| Seguridad al final          | Seguridad desde el inicio                 |
| Tareas manuales             | Automatización de análisis                |
| Equipo de seguridad aislado | Seguridad como responsabilidad compartida |
| Seguridad ralentiza         | Seguridad acelera (con confianza)         |

---

### 🔄 Principios clave

#### 1. **Shift Left**

> Mover los controles de seguridad al inicio del ciclo de desarrollo.

Aplicaciones prácticas:

* Escaneo de dependencias al hacer commit.
* Validación de código y secretos en los PRs.
* Análisis de configuración IaC antes del despliegue.

#### 2. **Automatización**

> La seguridad debe estar integrada en los pipelines de CI/CD.

Herramientas como SonarQube, Trivy, tfsec o GitHub secret scanning permiten detectar vulnerabilidades de forma automática **sin frenar al desarrollador**.

#### 3. **Cultura**

> Seguridad como responsabilidad compartida por desarrolladores, DevOps y seguridad.

Esto requiere:

* **Entrenamiento básico en seguridad** para todos.
* Definición de **checklists de seguridad** en las revisiones de código.
* Tolerancia cero a claves hardcodeadas o configuraciones inseguras.

#### 4. **Visibilidad y feedback**

> Cada paso debe dar visibilidad inmediata de posibles problemas.

Ejemplo: si el análisis de dependencias detecta una librería vulnerable, el pipeline debe fallar y notificar al desarrollador **antes del merge**.

---

## 💼 Ejemplo empresarial

### Contexto

Una empresa de e-commerce despliega microservicios en AWS Lambda y Azure App Services. El equipo DevOps implementa CI/CD con GitHub Actions.

### Problema

Detectan una vulnerabilidad crítica (Log4j) **después del despliegue**. No había escaneo de dependencias ni alertas.

### Solución DevSecOps

* Se integran herramientas como **Dependabot** y **pip-audit** en los PRs.
* Se añaden pasos en GitHub Actions que ejecutan análisis SAST, DAST y escaneo de contenedores.
* Los errores de seguridad bloquean automáticamente el merge o el despliegue.
* Se forma a los desarrolladores en lectura de alertas de seguridad.

Resultado:
✔️ Reducción del 80% en tiempos de respuesta a vulnerabilidades
✔️ Aumento de la confianza del equipo de seguridad en las entregas CI/CD

---

## ⚙️ Código / Configuración

### 🔍 Ejemplo de escaneo de dependencias con `pip-audit`

```yaml
# .github/workflows/security-audit.yml
name: Security Audit

on:
  pull_request:
    paths:
      - '**/requirements.txt'

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install pip-audit
        run: pip install pip-audit
      - name: Run pip-audit
        run: pip-audit -r requirements.txt
```

Este workflow escanea automáticamente el `requirements.txt` en cada PR, bloqueando el merge si hay dependencias con CVEs conocidas.

---

## 🧪 Tips para probar en consola y CLI

### En consola (GitHub)

1. Sube un `requirements.txt` vulnerable (ej. con `urllib3<1.26`).
2. Crea un PR en GitHub.
3. Observa cómo el **workflow falla** con alerta de seguridad.

### En CLI

```bash
pip install pip-audit
pip-audit -r requirements.txt
```

Resultado esperado: se muestran los paquetes con vulnerabilidades conocidas.

---

## ⚠️ Errores comunes

| Error                                        | Por qué es peligroso                          |
| -------------------------------------------- | --------------------------------------------- |
| Solo escanear en producción                  | Las vulnerabilidades ya estarán en el entorno |
| Subir claves en el repo                      | Riesgo de robo y abuso inmediato              |
| Ignorar análisis porque "ralentiza el build" | Compromete toda la cadena de suministro       |
| Permitir PR sin revisión de seguridad        | Facilita inyecciones de código inseguro       |

---

## ✅ Buenas prácticas

✔️ Aplicar “shift left”: escaneo desde el repositorio.
✔️ Automatizar los análisis (SAST, DAST, SCA) en cada build.
✔️ Entrenar a todos los desarrolladores en buenas prácticas de seguridad.
✔️ Reforzar el principio de menor privilegio (también en pipelines).
✔️ Fallar el pipeline si hay vulnerabilidades críticas.

---

## 🔗 Recursos recomendados

* [📘 DevSecOps – AWS Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/principles-devsecops/principles-devsecops.pdf)
* [🔐 Azure DevSecOps Guidance](https://learn.microsoft.com/en-us/azure/security/develop/devsecops-overview)
* [🔍 pip-audit – Python security tool](https://github.com/pypa/pip-audit)
* [📦 GitHub Actions security best practices](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
* [📚 OWASP DevSecOps Maturity Model](https://owasp.org/www-project-devsecops-maturity-model/)