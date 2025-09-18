# 🔐 Seguridad en Pipelines (Credenciales y Secretos)

---

## 🧭 Introducción – Qué es y por qué importa

Los pipelines de CI/CD son tan seguros como lo son sus **credenciales**.

Un solo descuido (como una clave expuesta en un repo) puede comprometer:

* Tu cuenta cloud entera.
* Repositorios privados.
* Datos sensibles en producción.

Este archivo aborda cómo **asegurar pipelines** frente a fugas de secretos, **gestionar credenciales de forma segura** y aplicar principios de seguridad mínima en entornos AWS y Azure.

---

## 📚 Explicación detallada

### 🎯 ¿Por qué es crítico?

Los ataques más frecuentes en la cadena de suministro comienzan por:

* Secretos hardcodeados en código o pipelines.
* Tokens de acceso expuestos en GitHub.
* Repositorios públicos con historiales inseguros.

El objetivo de un pipeline seguro es:
✅ Evitar el uso de claves estáticas.
✅ Asegurar el acceso cloud desde el pipeline sin exponer secretos.
✅ Implementar políticas de rotación y aislamiento de secretos.

---

### 🔑 Gestión de secretos en AWS y Azure

#### En **AWS**

* [Secrets Manager](https://aws.amazon.com/secrets-manager/): para credenciales, tokens y claves.
* [SSM Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html): alternativa para parámetros y secretos.
* **IAM Roles**: acceso a servicios sin claves.

#### En **Azure**

* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/): almacén seguro de secretos, certificados, claves.
* **Managed Identity (MI)**: acceso a servicios sin claves explícitas.

---

### 🛡️ GitHub Actions: acceso sin claves usando OIDC

GitHub permite a un pipeline autenticarse contra AWS o Azure **sin usar claves** mediante OIDC (*OpenID Connect*).

#### 🔁 GitHub → AWS (con OIDC)

1. Configuras un **OIDC Identity Provider** en AWS IAM.
2. Creas un **IAM Role** con `Condition` que permita asumirlo solo si viene de GitHub.
3. GitHub usa el token OIDC para obtener **credenciales temporales**.

#### 🔁 GitHub → Azure (Federated Credentials)

1. Creas un **App Registration** en Azure AD.
2. Añades un **Federated Credential** vinculado a GitHub.
3. GitHub asume esa identidad para usar servicios Azure con permisos mínimos.

✅ Sin claves.
✅ Tokens efímeros.
✅ Acceso restringido por `repo`, rama o workflow.

---

### 🔁 Rotación de secretos

Las herramientas como Secrets Manager y Key Vault permiten:

* Programar rotaciones automáticas.
* Emitir nuevos secretos y revocar los antiguos.
* Auditar accesos y uso de cada secreto.

---

## 💼 Ejemplo empresarial

### Contexto

Una empresa fintech almacena sus claves AWS como variables en GitHub Actions. Un empleado accidentalmente imprime `AWS_SECRET_ACCESS_KEY` en un log.

### Incidente

Horas después, la cuenta AWS recibe llamadas desde una IP desconocida que lanza instancias EC2 para minería.

### Solución

1. Se elimina el uso de claves estáticas.
2. Se implementa autenticación OIDC GitHub → AWS.
3. Los secretos reales se mueven a Secrets Manager, accesibles solo desde producción.
4. Se habilita auditoría y rotación automática.

Resultado:
✔️ Cero secretos en el código.
✔️ Accesos controlados y trazables.
✔️ Aislamiento por entorno (dev/test/prod).

---

## ⚙️ Código / Configuración

### 🔐 Ejemplo: Acceder a AWS desde GitHub Actions usando OIDC

#### 1. GitHub Actions: configuración mínima

```yaml
# .github/workflows/deploy.yml
name: Deploy with OIDC

permissions:
  id-token: write
  contents: read

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-deploy-role
          aws-region: eu-west-1

      - name: Deploy (example)
        run: aws s3 ls
```

#### 2. AWS: crear `OIDC provider` y `IAM role` limitado

```json
{
  "Effect": "Allow",
  "Principal": {
    "Federated": "arn:aws:iam::<account_id>:oidc-provider/token.actions.githubusercontent.com"
  },
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:sub": "repo:mi-org/mi-repo:ref:refs/heads/main"
    }
  }
}
```

---

## 🧪 Tips para probar en consola y CLI

### En consola (AWS)

* Crea IAM Role con confianza OIDC.
* Usa `sts:AssumeRoleWithWebIdentity` para simular acceso GitHub.

### En CLI (local)

```bash
aws sts assume-role-with-web-identity \
  --role-arn arn:aws:iam::123456789012:role/github-deploy-role \
  --role-session-name test \
  --web-identity-token file://token.jwt
```

### En GitHub

* Revisa la ejecución del step `configure-aws-credentials` y asegúrate de que no hay claves visibles.
* Verifica que la identidad temporal se genera correctamente.

---

## ⚠️ Errores comunes

| Error                                | Riesgo                                 |
| ------------------------------------ | -------------------------------------- |
| Usar claves estáticas en GitHub      | Claves filtradas = cuenta comprometida |
| Reutilizar secretos en dev/test/prod | Un fallo afecta a todos los entornos   |
| Guardar secretos en texto plano      | Accesibles por cualquier desarrollador |
| Rotación manual (o inexistente)      | Secretos antiguos no se invalidan      |

---

## ✅ Buenas prácticas

✔️ Elimina claves estáticas de los pipelines.
✔️ Usa OIDC o Managed Identities para acceso sin secretos.
✔️ Gestiona secretos con AWS Secrets Manager o Azure Key Vault.
✔️ Segrega secretos por entorno (dev/test/prod).
✔️ Activa la rotación automática y auditoría de acceso.
✔️ Revisa periódicamente la política de confianza (`Condition`) en OIDC/Federated Credentials.

---

## 🔗 Recursos recomendados

* [🔐 GitHub Actions OIDC → AWS](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
* [🛡️ GitHub OIDC → Azure](https://learn.microsoft.com/en-us/azure/active-directory/workload-identities/workload-identity-federation-create-trust-github)
* [🔐 AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
* [🔐 Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/overview)
* [📜 GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning/about-secret-scanning)
* [🛠️ git-secrets](https://github.com/awslabs/git-secrets)