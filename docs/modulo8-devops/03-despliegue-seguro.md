# 🚀 Despliegue Seguro

---

## 🧭 Introducción – Qué es y por qué importa

El proceso de despliegue (CD: *Continuous Deployment*) es un punto crítico de seguridad.
Aunque el código esté auditado, un mal despliegue puede:

* Exponer puertos o rutas no deseadas.
* Otorgar permisos excesivos.
* Filtrar secretos o variables sensibles.

**Desplegar no es solo mover artefactos, es validar que lo que llega a producción cumple las políticas de seguridad.**

---

## 📚 Explicación detallada

### 🔐 ¿Qué implica un despliegue seguro?

#### 1. Validación previa a aplicar

Antes de aplicar un `terraform apply` o desplegar con un pipeline, es necesario **auditar y validar** lo que se va a desplegar.

Ejemplo de validación:

* ¿Se expone `0.0.0.0/0` sin restricciones?
* ¿Se están creando roles IAM o identidades con permisos amplios?

---

#### 2. Infraestructura como Código (IaC) con validación

La IaC permite que la seguridad sea **reproducible y auditable**.

* **AWS**: Terraform, CloudFormation, CDK.
* **Azure**: Bicep, ARM templates, Terraform.

Validadores comunes:

* [`tfsec`](https://aquasecurity.github.io/tfsec/)
* [`checkov`](https://www.checkov.io/)
* \[`cfn-lint`, `cfn-guard`] (para CloudFormation)
* \[`PSRule`, `ARM-TTK`] (para ARM templates)

---

#### 3. Principio de menor privilegio

El pipeline debe ejecutar con **credenciales mínimas necesarias**, no con privilegios de administrador global.

* En **AWS**: usar roles IAM con políticas acotadas al entorno (`dev`, `prod`).
* En **Azure**: usar identidades administradas (Managed Identities) con permisos limitados.

---

#### 4. Segregación de entornos

Separar claramente los despliegues:

* `dev`: sandbox para pruebas.
* `test`: entorno validado automáticamente.
* `prod`: protegido con revisiones adicionales.

Cada entorno debe tener políticas, secretos y roles **diferenciados**.

---

#### 5. Validaciones automáticas en el pipeline

Antes de aplicar cambios:

* Escanear la plantilla IaC.
* Validar variables y configuración.
* Revisar cambios con `terraform plan` o equivalente.
* Aplicar solo si pasa el escaneo y la revisión.

---

## 💼 Ejemplo empresarial

### Contexto

Una empresa de logística despliega recursos en AWS mediante Terraform y CodePipeline.

### Problema

Un ingeniero despliega accidentalmente un Security Group con acceso abierto (`0.0.0.0/0`) a la base de datos RDS.

### Solución DevSecOps

1. Se integra `tfsec` en el pipeline de CodePipeline.
2. El `terraform plan` se revisa automáticamente.
3. Se bloquea el `apply` si hay findings de seguridad.
4. Se crean entornos diferenciados (`dev`, `qa`, `prod`) con políticas IAM distintas.

Resultado:
✔️ Prevención de errores humanos comunes
✔️ Separación clara entre entornos
✔️ Auditoría reproducible del despliegue

---

## ⚙️ Código / Configuración

### 🔍 Pipeline de despliegue seguro con `tfsec` en GitHub Actions

```yaml
# .github/workflows/deploy-secure.yml
name: Secure Deployment

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.5.7

      - name: Init Terraform
        run: terraform init

      - name: Validate Terraform
        run: terraform validate

      - name: Run tfsec scan
        uses: aquasecurity/tfsec-action@v1.0.0

      - name: Terraform plan
        run: terraform plan -out=tfplan.binary

      - name: Manual approval gate (optional)
        if: github.ref == 'refs/heads/main'
        run: echo "✅ Revisar tfsec report y aprobar manualmente antes de aplicar"

      # - name: Apply (solo si es seguro)
      #   run: terraform apply tfplan.binary
```

> 💡 Recomendado: usar aprobación manual en producción.

---

## 🧪 Tips para probar en consola y CLI

### En CLI

```bash
# Instala tfsec
brew install tfsec  # o `go install` desde GitHub

# Escanea tu carpeta Terraform
tfsec ./infra
```

Resultado: advertencias como:

```
[aws-sg-open-all] Security group allows ingress from 0.0.0.0/0 on port 5432
```

---

### En consola (GitHub)

1. Sube un `main.tf` con un recurso inseguro.
2. Observa cómo `tfsec` lo detecta y el pipeline falla.
3. Revisa los findings en el log de GitHub Actions.

---

## ⚠️ Errores comunes

| Error                                       | Riesgo asociado                             |
| ------------------------------------------- | ------------------------------------------- |
| Desplegar sin `terraform plan` o validación | Cambios inesperados en producción           |
| No escanear SGs o IAM                       | Permisos abiertos, datos expuestos          |
| Usar claves estáticas en CI/CD              | Riesgo de compromiso si se filtran          |
| No separar entornos                         | Pruebas pueden afectar producción           |
| Repos compartidos sin revisión              | Cualquier commit puede comprometer recursos |

---

## ✅ Buenas prácticas

✔️ Validar toda IaC antes de aplicar.
✔️ Escanear con `tfsec`, `checkov` o similares.
✔️ Revisar el `terraform plan` antes de aplicar.
✔️ Usar entornos segregados (`dev`, `test`, `prod`).
✔️ Usar variables seguras y secretos gestionados.
✔️ Aplicar el principio de menor privilegio en el pipeline.

---

## 🔗 Recursos recomendados

* [🔒 tfsec (validador de Terraform)](https://aquasecurity.github.io/tfsec/)
* [🛡️ checkov (IaC security scanner)](https://www.checkov.io/)
* [📘 Terraform Security Best Practices](https://developer.hashicorp.com/terraform/tutorials/security/security-best-practices)
* [🔐 Azure Bicep security guide](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/security-baselines)
* [📚 GitHub Actions manual approvals](https://docs.github.com/en/actions/using-jobs/using-environments-for-deployment#required-reviewers)
