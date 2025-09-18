# 🧭 Buenas Prácticas en Seguridad de Aplicaciones y DevOps

---

## 🧭 Introducción – Qué es y por qué importa

La práctica constante de DevSecOps requiere más que herramientas: requiere **disciplina, cultura y normas claras**.

> En este último archivo del módulo veremos las **buenas prácticas** fundamentales que aseguran todo el ciclo DevOps, desde el commit hasta el despliegue, pasando por la gestión de secretos, IaC y contenedores.

Aplicarlas **no es opcional** si queremos:

* Minimizar errores humanos.
* Reducir la superficie de ataque.
* Mejorar la confianza y velocidad de despliegue.

---

## ✅ Reglas esenciales de seguridad en DevOps

---

### 1. 🔑 Nunca usar claves estáticas en código ni pipelines

* Usa OIDC o identidades gestionadas.
* Mueve secretos a AWS Secrets Manager o Azure Key Vault.
* Usa políticas de acceso específicas por entorno.

**Mal ejemplo**:

```yaml
env:
  AWS_SECRET_KEY: my-super-secret
```

**Buen ejemplo**:

```yaml
permissions:
  id-token: write  # GitHub OIDC
```

---

### 2. 📦 Escanea todas las dependencias, siempre

* Escaneo automático con Dependabot, `pip-audit`, `npm audit`, etc.
* Bloquear despliegue si hay CVEs críticas.
* Mantener los lockfiles (`package-lock.json`, `poetry.lock`, etc.) versionados.

---

### 3. 🐳 Validar todas las imágenes de contenedor

* Integrar **Trivy**, **Grype**, **Dockle** en el pipeline.
* Evitar imágenes como `latest` o sin origen controlado.
* Usar imágenes mínimas (`distroless`, `alpine`) y firmadas.

**Mal ejemplo**:

```Dockerfile
FROM ubuntu:latest
```

**Buen ejemplo**:

```Dockerfile
FROM python:3.11-slim
```

---

### 4. 🛡️ Validar IaC antes de aplicar

* Escaneo con `tfsec`, `checkov`, `tflint`.
* Revisar `terraform plan` o `bicep what-if`.
* Revisar outputs y permisos creados por IaC.

**Checklist mínimo**:

* ¿Hay recursos públicos no deseados?
* ¿Se expone `0.0.0.0/0`?
* ¿Se usan nombres, secretos o etiquetas sensibles?

---

### 5. 🔍 Revisión obligatoria de código (con checklist de seguridad)

En cada PR:

✅ ¿El código valida inputs del usuario?
✅ ¿Evita dependencias innecesarias?
✅ ¿Hay hardcodeos de secretos?
✅ ¿Las rutas públicas están protegidas?

---

### 6. 🚫 Falla el pipeline si algo no pasa

* Todos los análisis de seguridad deben poder **frenar** el despliegue.
* La visibilidad no sirve si no se actúa.

> 🧨 *"Un análisis de seguridad que no puede fallar el pipeline es solo decoración."*

---

### 7. 🌍 Segregar entornos (dev, test, prod)

* No usar los mismos secretos en todos los entornos.
* Aplicar **políticas y permisos distintos**.
* Los entornos deben poder caer sin afectar a los demás.

---

### 8. 🔐 Implementar rotación automática de secretos

* Secrets Manager (AWS) y Key Vault (Azure) lo permiten.
* Los tokens o credenciales deben tener caducidad y renovación controlada.
* Auditar el acceso a los secretos.

---

### 9. 📊 Medir KPIs de seguridad en DevOps

* % de PRs escaneadas por seguridad.
* % de builds que fallan por seguridad.
* Tiempo medio de remediación.
* Tasa de rotación de secretos.
* Nº de secretos expuestos evitados.

---

### 10. 📚 Documentar y formar a los equipos

* Checklist de revisión de código.
* Guías de seguridad por lenguaje/framework.
* Formación básica en SAST, DAST, SCA y CI/CD.
* Crear cultura de “security champions” en cada equipo.

---

## 💼 Casos de errores comunes

| Error                                           | Riesgo asociado                           |
| ----------------------------------------------- | ----------------------------------------- |
| Dejar secretos en `.env`, `config.py`, etc.     | Robo inmediato de credenciales            |
| Reusar claves entre entornos                    | Escalado lateral de ataques               |
| Aceptar findings de seguridad “por velocidad”   | Introducción voluntaria de riesgo         |
| Tener scans, pero no bloquear nada              | Seguridad ignorada por diseño             |
| PRs sin revisión o con merges directos a `main` | Código malicioso o inseguro en producción |

---

## 📋 Checklist práctico final

### 🔒 Seguridad del código

* [ ] Secret scanning activado
* [ ] Análisis estático (SAST)
* [ ] PR con revisión obligatoria

### 📦 Dependencias

* [ ] SCA automático (Dependabot / CLI)
* [ ] Lockfiles versionados
* [ ] CVEs críticas bloquean build

### 🐳 Contenedores

* [ ] Escaneo con Trivy o Grype
* [ ] No usar imágenes `latest`
* [ ] Imágenes mínimas y firmadas

### 🛠️ Infraestructura como Código

* [ ] Validación previa (`plan`, `what-if`)
* [ ] Escaneo con tfsec / checkov
* [ ] Separación clara de entornos

### 🔐 Secretos y CI/CD

* [ ] No hay claves estáticas
* [ ] Uso de OIDC / MI
* [ ] Rotación activa
* [ ] Accesos auditables

---

## 🔗 Recursos recomendados

* [🔐 DevOps Security Best Practices (AWS)](https://docs.aws.amazon.com/whitepapers/latest/principles-devsecops/principles-devsecops.pdf)
* [🛡️ Azure DevOps Security Overview](https://learn.microsoft.com/en-us/azure/devops/learn/devops-at-microsoft/security)
* [🔍 Trivy – Scanner unificado](https://aquasecurity.github.io/trivy/)
* [📦 GitHub Advanced Security](https://github.com/features/security)
* [📘 OWASP Top 10 DevSecOps Risks](https://owasp.org/www-project-devsecops-guideline/latest/)
* [✅ DevSecOps Maturity Model](https://owasp.org/www-project-devsecops-maturity-model/)