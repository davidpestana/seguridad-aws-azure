# 🧪 Laboratorio 08 – Crear un pipeline CI/CD seguro con AWS CodePipeline y análisis de seguridad

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Configurar un pipeline en AWS CodePipeline que automatice el build y despliegue.
* Usar CodeBuild para compilar el código y ejecutar análisis de seguridad.
* Integrar herramientas como **Trivy** para escanear imágenes o dependencias.
* Proteger el pipeline con roles IAM adecuados y secretos cifrados.
* Validar que los resultados del análisis afecten la ejecución del pipeline.

---

## 🧵 Escenario práctico

Tu equipo despliega una aplicación Dockerizada en ECS. Quieres crear un pipeline que construya la imagen, la escanee automáticamente con Trivy antes del despliegue, y falle si hay vulnerabilidades críticas.

---

## 🧰 Requisitos previos

* Cuenta AWS con permisos sobre IAM, CodePipeline, CodeBuild, ECR.
* Un repositorio en GitHub con una app Docker simple (puedes usar [https://github.com/aquasecurity/trivy-java-sample](https://github.com/aquasecurity/trivy-java-sample) como demo).
* Un ECR repository ya creado (ej: `secure-app`).
* Dockerfile presente en el repo.

---

## 🧭 Pasos detallados

### 1️⃣ Crear roles IAM para CodePipeline y CodeBuild

#### A. Rol para CodePipeline

1. Ve a **IAM → Roles → Create role**
2. Servicio: **CodePipeline**
3. Adjunta estas políticas:

   * `AWSCodePipelineFullAccess`
   * `AmazonEC2ContainerRegistryPowerUser`
4. Nombre del rol: `codepipeline-secure-role`

#### B. Rol para CodeBuild

1. Servicio: **CodeBuild**
2. Adjunta:

   * `AmazonEC2ContainerRegistryPowerUser`
   * `AmazonS3FullAccess` (para artefactos)
   * `CloudWatchLogsFullAccess`
3. Nombre del rol: `codebuild-scan-role`

---

### 2️⃣ Crear un proyecto de CodeBuild con Trivy

1. Ve a **CodeBuild → Create project**

2. Nombre: `build-scan-app`

3. Fuente: GitHub (conecta tu repo)

4. Entorno:

   * Tipo: Managed image
   * Imagen: Ubuntu estándar con Docker
   * Privileged: ✅ Habilitado (para Docker builds)
   * Rol: `codebuild-scan-role`

5. Fichero de build (`buildspec.yml`) en el repositorio raíz:

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      docker: 20
    commands:
      - echo "Instalando Trivy..."
      - curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh
  pre_build:
    commands:
      - echo "Login en ECR"
      - aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
  build:
    commands:
      - echo "Construyendo imagen..."
      - docker build -t secure-app .
      - echo "Escaneando con Trivy..."
      - ./trivy image --severity CRITICAL secure-app || exit 1
  post_build:
    commands:
      - echo "Empujando imagen..."
      - docker tag secure-app <aws_account_id>.dkr.ecr.<region>.amazonaws.com/secure-app:latest
      - docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/secure-app:latest

artifacts:
  files:
    - '**/*'
```

🔒 Nota: el `exit 1` fuerza que el pipeline falle si se detectan vulnerabilidades **CRÍTICAS**.

---

### 3️⃣ Crear el pipeline en CodePipeline

1. Ve a **CodePipeline → Create pipeline**
2. Nombre: `pipeline-segura`
3. Service role: usar `codepipeline-secure-role`

#### Fases:

* **Source**: GitHub → repositorio con Dockerfile y buildspec
* **Build**: CodeBuild → `build-scan-app`
* **Deploy**: puede omitirse o apuntar a ECS si tienes setup

---

### 4️⃣ Probar el pipeline

1. Realiza un commit en el repo (puede ser una modificación simple).
2. Observa la ejecución del pipeline:

   * Fase de build ejecutará Trivy
   * Si hay vulnerabilidades críticas, fallará.
3. Puedes editar el `buildspec.yml` para cambiar la severidad (`--severity HIGH,CRITICAL`) o hacer que pase el build aunque haya findings (`|| true`).

---

## ✅ Validación

* ¿El pipeline se activa con un commit en GitHub?
* ¿CodeBuild construye la imagen y ejecuta Trivy?
* ¿Se detiene la ejecución si hay fallos de seguridad?
* ¿Se publica la imagen solo si pasa el escaneo?

---

## ⚠️ Errores comunes

| Problema                   | Causa común                   | Solución                                        |
| -------------------------- | ----------------------------- | ----------------------------------------------- |
| `trivy: command not found` | Fallo en instalación de Trivy | Asegúrate de ejecutar el script correctamente   |
| Error al hacer push a ECR  | Rol sin permisos              | Añadir `AmazonEC2ContainerRegistryPowerUser`    |
| Build falla siempre        | Trivy detecta findings        | Ajusta `--severity` o cambia la lógica de fallo |

---

## 🧩 Extensiones opcionales

* Añade notificaciones con **SNS** al fallar el pipeline.
* Usa **Secrets Manager** para almacenar claves del repo o API keys.
* Agrega una etapa de despliegue a ECS/Fargate.
* Integra con **CloudWatch dashboards** para visualizar estado de builds.