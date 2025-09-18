**⚙️ Entorno DevContainer**

Esta carpeta contiene la configuración del entorno de desarrollo preconfigurado para el curso **Seguridad en AWS y Azure**, usando Visual Studio Code DevContainers o GitHub Codespaces.

---

**💡 ¿Qué incluye?**

El entorno ya viene preparado con:

* **AWS CLI**: Para gestionar IAM, EC2, S3, etc.
* **Azure CLI**: Para gestionar Azure AD, VMs, redes, etc.
* **Terraform**: (opcional) Para infraestructura como código
* **jq**: Para manipulación de JSON en terminal
* **bash**: Shell por defecto

---

**🚀 Usar en GitHub Codespaces**

1. Asegúrate de tener GitHub Codespaces habilitado en tu cuenta.
2. Abre el repositorio y haz clic en el botón verde "Code".
3. Selecciona "Open with Codespaces" > "New codespace".
4. Espera mientras se construye el entorno.
5. Una vez abierto, ya puedes usar `aws`, `az`, `terraform`, etc.

---

**🧑‍💻 Usar en VS Code (local)**

1. Instala:

   * Docker Desktop
   * Visual Studio Code
   * Extensión **Remote - Containers**

2. Clona el repositorio:

   `git clone https://github.com/<usuario>/seguridad-aws-azure.git`

   `cd seguridad-aws-azure`

3. Abre el proyecto en VS Code:

   `code .`

4. VS Code detectará el DevContainer y mostrará:

   > "Reopen in Container"

   Acepta y el entorno se preparará automáticamente.

---

**📝 Personalización**

* Puedes modificar el archivo `Dockerfile` para añadir más herramientas.
* Las extensiones y configuraciones están en `devcontainer.json`.
