# 🧪 Laboratorio 05 – Crear una máquina virtual segura con clave SSH y acceso basado en roles (RBAC)

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Crear una máquina virtual en Azure con autenticación por **clave SSH**.
* Evitar el uso de contraseñas, aumentando la seguridad del acceso.
* Asignar un **rol RBAC** a la VM para permitirle acceder a otros recursos (por ejemplo, almacenamiento).
* Validar el acceso desde la terminal local mediante SSH.
* Verificar el uso de **Managed Identity** desde la VM.

---

## 🧵 Escenario práctico

Necesitas desplegar una VM Linux para ejecutar tareas internas, pero sin comprometer la seguridad. La organización prohíbe el uso de contraseñas, y además quiere que esta VM acceda de forma segura a una cuenta de almacenamiento sin almacenar claves: para ello usarás **Managed Identity + RBAC**.

---

## 🧰 Requisitos previos

* Suscripción de Azure activa.
* Azure CLI instalada (recomendado).
* Una clave SSH local (`id_rsa.pub`) o generarla durante el laboratorio.
* Permisos para crear máquinas virtuales, Managed Identity y asignar roles.

---

## 🧭 Pasos detallados

### 1️⃣ Crear un grupo de recursos

```bash
az group create \
  --name rg-computo-seguro \
  --location westeurope
```

---

### 2️⃣ (Opcional) Generar una clave SSH local

Si no tienes clave SSH aún:

```bash
ssh-keygen -t rsa -b 4096
```

Usa la clave pública generada en el siguiente paso: `~/.ssh/id_rsa.pub`

---

### 3️⃣ Crear la máquina virtual

```bash
az vm create \
  --resource-group rg-computo-seguro \
  --name vm-segura \
  --image UbuntuLTS \
  --admin-username azureuser \
  --authentication-type ssh \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --public-ip-sku Standard \
  --assign-identity
```

📌 Nota: El flag `--assign-identity` crea automáticamente una **Managed Identity** para la VM.

---

### 4️⃣ Crear una cuenta de almacenamiento

```bash
az storage account create \
  --name almacenadoseguro<tuusuario> \
  --resource-group rg-computo-seguro \
  --location westeurope \
  --sku Standard_LRS \
  --kind StorageV2
```

---

### 5️⃣ Asignar rol RBAC a la VM para acceder al almacenamiento

Primero, obtener el ID de la identidad:

```bash
az vm show \
  --resource-group rg-computo-seguro \
  --name vm-segura \
  --query identity.principalId \
  --output tsv
```

Luego, asignar el rol:

```bash
az role assignment create \
  --assignee <principal-id> \
  --role "Storage Blob Data Reader" \
  --scope $(az storage account show \
              --name almacenadoseguro<tuusuario> \
              --resource-group rg-computo-seguro \
              --query id -o tsv)
```

---

### 6️⃣ Acceder por SSH a la VM

```bash
ssh azureuser@<ip-publica>
```

✅ Si configuraste bien la clave, deberías poder entrar sin contraseña.

---

### 7️⃣ Validar acceso a la cuenta de almacenamiento desde la VM

1. Instalar Azure CLI (si no está):

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

2. Obtener token como Managed Identity:

```bash
az login --identity
```

3. Acceder a blobs (si hay alguno cargado):

```bash
az storage blob list \
  --account-name almacenadoseguro<tuusuario> \
  --container-name <nombre-del-contenedor> \
  --auth-mode login
```

✅ Si la asignación de rol fue correcta, la VM podrá acceder **sin claves**.

---

## ✅ Validación

* ¿Puedes acceder a la VM solo con clave SSH?
* ¿La VM tiene una Managed Identity activa?
* ¿Tiene permisos RBAC sobre el recurso de almacenamiento?
* ¿Puedes listar blobs desde dentro de la VM sin configurar claves?

---

## ⚠️ Errores comunes

| Problema                                    | Causa común                             | Solución                                                |
| ------------------------------------------- | --------------------------------------- | ------------------------------------------------------- |
| SSH falla                                   | IP bloqueada o clave incorrecta         | Revisa reglas NSG y asegúrate de usar la clave correcta |
| az login --identity falla                   | No se asignó la identidad correctamente | Revisa con `az vm show` si tiene identity               |
| “Authorization failed” al acceder a Storage | No se asignó el rol correctamente       | Revisa con `az role assignment list`                    |

---

## 🧩 Extensiones opcionales

* Asocia un **NSG** a la red de la VM y restringe el puerto 22 a tu IP.
* Crea un contenedor en Storage y sube archivos desde la VM.
* Usa `az keyvault` para probar acceso a secretos desde la VM con su identidad.
* Automatiza la creación con Terraform o Bicep.