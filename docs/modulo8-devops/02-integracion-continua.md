# 🔄 Integración Continua Segura

---

## 🧭 Introducción – Qué es y por qué importa

En entornos DevOps, la **integración continua (CI)** permite validar automáticamente cada cambio de código antes de integrarlo al repositorio principal.
Sin embargo, si no se integran **controles de seguridad**, se pueden introducir vulnerabilidades directamente en la cadena de suministro.

Una **integración continua segura** aplica el principio de *"fail early, fail safe"*, detectando errores de seguridad **antes del despliegue**.

---

## 📚 Explicación detallada

### 🧪 ¿Qué controles de seguridad aplicar en CI?

#### 1. **SAST – Análisis Estático del Código**

Detecta fallos de seguridad directamente en el código fuente.

* **AWS**: CodeGuru Reviewer (para Java, Python).
* **Azure**: Integración con SonarCloud / GitHub CodeQL.
* Otras herramientas comunes: **SonarQube**, **Bandit (Python)**, **Brakeman (Ruby)**.

✅ Detecta patrones inseguros (SQLi, XSS, malas prácticas).

---

#### 2. **Escaneo de Dependencias (SCA)**

Las vulnerabilidades no solo están en tu código, sino en las librerías de terceros.

* **AWS**: CodePipeline puede integrar Dependabot o Trivy.
* **Azure**: GitHub Dependabot o Azure DevOps con `pip-audit`, `npm audit`, etc.

✅ Detecta paquetes con CVEs activos.

---

#### 3. **Secrets Scanning**

Evita que claves, tokens o contraseñas terminen en el repositorio.

* **GitHub**: Secret Scanning integrado (detecta más de 100 tipos de claves).
* **CLI**: `git-secrets`, `truffleHog`, `detect-secrets`.

✅ Previene fugas accidentales de credenciales.

---

#### 4. **Revisión de Código con Checklist de Seguridad**

Toda integración debe pasar una **pull request** revisada por al menos un par.
Checklist básico:

* ¿Se validan entradas del usuario?
* ¿Se evita la exposición de secretos?
* ¿Se siguen buenas prácticas de codificación segura?
* ¿El código modifica configuraciones críticas?

---

## 💼 Ejemplo empresarial

### Contexto

Una startup de movilidad usa Python y React. Sus desarrolladores hacen push a `main` directamente sin revisión ni escaneo.

### Problema

Un desarrollador sube por error su token de acceso a AWS en un `config.py`. Es detectado 2 semanas después tras un uso fraudulento.

### Solución DevSecOps

1. Se obliga a trabajar con ramas + PR.
2. Se integran workflows de GitHub Actions con:

   * Linter + test + SAST con `bandit`.
   * `pip-audit` para dependencias.
   * Secret scanning con `truffleHog`.
3. Se añade checklist de seguridad obligatorio en cada revisión.

Resultado:
✔️ Detección instantánea de tokens expuestos
✔️ Validación automatizada en cada commit
✔️ Cultura de revisión y seguridad compartida

---

## ⚙️ Código / Configuración

### 🔐 GitHub Actions – Pipeline seguro con análisis estático y dependencias

```yaml
# .github/workflows/ci-security.yml
name: CI + Security

on:
  pull_request:
    branches: [ main ]

jobs:
  ci-security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install bandit pip-audit

      - name: Lint code
        run: pylint app/

      - name: Run unit tests
        run: pytest

      - name: Static analysis (bandit)
        run: bandit -r app/ -ll

      - name: Dependency scan (pip-audit)
        run: pip-audit -r requirements.txt
```

---

## 🧪 Tips para probar en consola y CLI

### En consola (GitHub)

1. Crea un PR con código Python inseguro (por ejemplo, `eval(input())` o una librería vulnerable).
2. Observa cómo el pipeline:

   * Ejecuta `bandit` y `pip-audit`
   * Falla si detecta una vulnerabilidad o mala práctica

### En CLI

```bash
pip install bandit pip-audit

# Bandit: análisis estático
bandit -r app/ -ll

# pip-audit: escaneo de dependencias
pip-audit -r requirements.txt
```

---

## ⚠️ Errores comunes

| Error                                    | Riesgo asociado                           |
| ---------------------------------------- | ----------------------------------------- |
| Push directo a `main` sin PR             | Código inseguro sin revisión              |
| No escanear dependencias                 | Introducción de CVEs sin control          |
| Ignorar alertas por "no bloquear builds" | Peligro de normalizar fallos de seguridad |
| Dejar claves en commits                  | Compromiso inmediato de infraestructuras  |

---

## ✅ Buenas prácticas

✔️ Hacer PRs obligatorios con revisión de al menos una persona.
✔️ Automatizar tests + SAST + SCA en cada PR.
✔️ Fallar el pipeline ante vulnerabilidades **críticas** o **secretos expuestos**.
✔️ No permitir push directo a rama principal.
✔️ Documentar y versionar el workflow de CI/CD.

---

## 🔗 Recursos recomendados

* [📘 GitHub Actions Security Best Practices](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
* [🔍 Bandit (análisis estático Python)](https://bandit.readthedocs.io/)
* [📦 pip-audit (auditoría de dependencias)](https://github.com/pypa/pip-audit)
* [🔐 git-secrets](https://github.com/awslabs/git-secrets)
* [📚 SonarCloud para GitHub](https://sonarcloud.io/documentation/integrations/github/)
* [🛡️ CodeQL](https://codeql.github.com/docs/)