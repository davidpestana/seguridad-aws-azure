# 🛡️ Curso de Seguridad en AWS y Azure

Este repositorio contiene todos los materiales, laboratorios y configuraciones necesarias para seguir el curso de **Seguridad en la Nube con AWS y Azure**, orientado a profesionales que desean comprender, aplicar y administrar buenas prácticas de seguridad en entornos cloud reales.

## 📚 Contenidos del curso

El curso está organizado en los siguientes módulos temáticos:

1. **Introducción a la Seguridad en la Nube**
2. **Gestión de Identidades y Accesos**
3. **Seguridad en la Red**
4. **Gestión de Datos y Almacenamiento Seguro**
5. **Seguridad en Servicios de Cómputo**
6. **Monitorización y Registro de Actividades**
7. **Gestión de Incidentes y Respuesta**
8. **Seguridad en Aplicaciones y DevOps**
9. **Compliance y Gobernanza**

Cada módulo incluye documentación resumida y uno o más laboratorios prácticos aplicables a AWS y Azure.

## 🧪 Laboratorios

Los laboratorios están pensados para ser ejecutados en un entorno real con cuentas activas de AWS y Azure. Instrucciones específicas se encuentran dentro de cada módulo en la carpeta `labs/`.

Ejemplos de prácticas:

- Gestión de usuarios y políticas IAM
- Configuración de redes seguras (VPC, NSG)
- Almacenamiento cifrado con control de acceso
- Monitorización con CloudWatch y Azure Monitor
- Simulación de ataques con GuardDuty y Sentinel
- Pipelines CI/CD seguros

## 💻 Entorno de desarrollo

El repositorio está preparado para usarse de forma inmediata mediante:

### 🔹 GitHub Codespaces

Puedes abrir el repositorio directamente en Codespaces para contar con:

- CLI de AWS (`aws`)
- CLI de Azure (`az`)
- Herramientas auxiliares (`jq`, `curl`, `terraform`, `bicep`, etc.)
- Autenticación fácil mediante navegador

Solo necesitas hacer clic en **Code > Open with Codespaces** (requiere habilitar Codespaces en tu cuenta).

### 🔸 Uso local con DevContainer

También puedes clonar el repositorio y abrirlo en VS Code con Docker:

```bash
git clone https://github.com/<ORGANIZACION>/seguridad-aws-azure.git
cd seguridad-aws-azure
code .
````

Asegúrate de tener:

* Docker Desktop
* Visual Studio Code
* Extensión de **Remote - Containers**

> El entorno se cargará automáticamente con todas las herramientas necesarias.

## 📝 Requisitos previos

* Cuenta de AWS (Free Tier)
* Cuenta de Azure (puede ser [Free Trial](https://azure.microsoft.com/es-es/free/), o gestionada por el formador)
* Conocimientos básicos de línea de comandos
* Familiaridad con conceptos cloud generales

## 📁 Estructura del repositorio

```
.
├── docs/                   # Material teórico y de apoyo
├── labs/                   # Carpeta principal de laboratorios por módulo
│   ├── aws/
│   └── azure/
├── .devcontainer/          # Entorno de desarrollo con Docker
├── scripts/                # Scripts auxiliares para labs o setup
└── README.md