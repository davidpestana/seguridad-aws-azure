# 🧪 Laboratorio 08 – Crear un pipeline seguro con GitHub Actions y análisis de seguridad en Azure

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Crear un **pipeline CI/CD** usando GitHub Actions en un repositorio de Azure App.
* Configurar el pipeline para **construir una imagen Docker**.
* Integrar una herramienta de escaneo como **Trivy** o **CodeQL**.
* Publicar la imagen en **Azure Container Registry (ACR)** solo si pasa el análisis.
* Proteger los secretos usados en el pipeline.

---

## 🧵 Escenario práctico

Estás desarrollando una aplicación web en Azure. Quieres implementar un pipeline en GitHub que construya y analice la aplicación cada vez que se hace un push. Si todo está bien, publica la imagen en ACR.

---

## 🧰 Requisitos previos

* Cuenta de Azure con permisos sobre ACR y App Service (o Azure Container Apps).
* Un repositorio en GitHub con Dockerfile y aplicación de ejemplo.
* Azure CLI instalada y autenticada (`az login`).
* Docker instalado en local para pruebas (opcional).
* ACR ya creado (ej: `acrseguro.azurecr.io`).

---

## 🧭 Pasos detallados

### 1️⃣ Crear o vincular un Azure Container Registry

```bash
az acr create \
  --name acrseguro \
  --resource-group rg-devops \
  --sku Basic \
  --admin-enabled true
```

> 🔐 Anota el `loginServer`, `username`, y `password` (los usarás en GitHub).

---

### 2️⃣ Guardar secretos en GitHub

En tu repositorio de GitHub:

1. Ve a **Settings → Secrets and variables → Actions**
2. Agrega los siguientes secretos:

| Nombre             | Valor                           |
| ------------------ | ------------------------------- |
| `ACR_USERNAME`     | nombre de usuario del ACR       |
| `ACR_PASSWORD`     | contraseña del ACR              |
| `ACR_LOGIN_SERVER` | ejemplo: `acrseguro.azurecr.io` |

---

### 3️⃣ Crear archivo `.github/workflows/pipeline.yml`

```yaml
name: Build and Scan

on:
  push:
    branches:
      - main

jobs:
  build-and-scan:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker
      uses: docker/setup-buildx-action@v2

    - name: Build Docker image
      run: docker build -t myapp .

    - name: Install Trivy
      run: |
        sudo apt-get update
        sudo apt-get install -y wget
        wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.53.0_Linux-64bit.deb
        sudo dpkg -i trivy_0.53.0_Linux-64bit.deb

    - name: Run Trivy scan
      run: |
        trivy image --exit-code 1 --severity CRITICAL,HIGH myapp

    - name: Login to ACR
      run: echo "${{ secrets.ACR_PASSWORD }}" | docker login ${{ secrets.ACR_LOGIN_SERVER }} -u ${{ secrets.ACR_USERNAME }} --password-stdin

    - name: Tag and Push to ACR
      run: |
        docker tag myapp ${{ secrets.ACR_LOGIN_SERVER }}/myapp:latest
        docker push ${{ secrets.ACR_LOGIN_SERVER }}/myapp:latest
```

✅ Este pipeline **fallará automáticamente** si Trivy detecta vulnerabilidades críticas o altas.

---

### 4️⃣ Realizar un commit para probar el pipeline

1. Haz un pequeño cambio en el `README.md` o código.
2. Realiza `git commit` y `git push`.
3. Ve a **GitHub → Actions** → selecciona el workflow.
4. Verifica:

   * El build y escaneo se completan correctamente.
   * Solo se sube la imagen si pasa el análisis.

---

## ✅ Validación

* ¿Trivy detiene el pipeline si hay vulnerabilidades críticas?
* ¿La imagen solo se publica si el análisis es exitoso?
* ¿Los secretos están protegidos en GitHub?
* ¿El pipeline se ejecuta automáticamente con cada push a `main`?

---

## ⚠️ Errores comunes

| Problema                                | Causa común          | Solución                                    |
| --------------------------------------- | -------------------- | ------------------------------------------- |
| `trivy: command not found`              | Falla en instalación | Verifica la descarga e instalación manual   |
| `unauthorized: authentication required` | Fallo en login a ACR | Revisa `ACR_USERNAME` y `ACR_PASSWORD`      |
| Imagen vulnerable, push bloqueado       | Trivy detecta fallos | Corrige vulnerabilidades o ajusta severidad |

---

## 🧩 Extensiones opcionales

* Añade **CodeQL** como segunda capa de análisis de código:

  ```yaml
  - uses: github/codeql-action/init@v2
  ```
* Integra una etapa de despliegue a **Azure Web App for Containers**.
* Crea un dashboard en **Azure Monitor** para trackear builds y alertas.
* Usa **Dependabot** para mantener las dependencias del repo actualizadas.