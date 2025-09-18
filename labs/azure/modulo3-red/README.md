# 🧪 Laboratorio 03 – Configurar Network Security Groups (NSG) en Azure

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Crear un **Network Security Group (NSG)** en Azure.
* Asociar el NSG a una subred o a una máquina virtual.
* Crear reglas de seguridad de entrada y salida.
* Validar el tráfico permitido/bloqueado mediante conexión SSH o HTTP.

---

## 🧵 Escenario práctico

Estás configurando la red de una pequeña aplicación web en Azure. Quieres asegurarte de que solo se permita tráfico HTTP (puerto 80) y SSH (puerto 22) desde tu IP, y bloquear cualquier otro tráfico no deseado. Usarás **NSG (Network Security Group)** para controlar el acceso.

---

## 🧰 Requisitos previos

* Suscripción activa de Azure.
* Acceso al portal de Azure: [https://portal.azure.com](https://portal.azure.com)
* IP pública desde la que harás pruebas (puedes obtenerla desde [https://ifconfig.me](https://ifconfig.me))
* Conocimientos básicos de creación de máquinas virtuales (VMs) en Azure.

---

## 🧭 Pasos detallados

### 1️⃣ Crear un Network Security Group (NSG)

1. En el portal, busca y selecciona **“Network Security Groups”**.
2. Haz clic en **“Create”**.
3. Configura:

   * **Nombre**: `nsg-web-seguro`
   * **Región**: misma que tu red virtual
   * **Grupo de recursos**: crea uno o usa uno existente (ej. `lab-red`)
4. Haz clic en **Review + Create** → **Create**

---

### 2️⃣ Crear reglas de seguridad de entrada

1. Accede al NSG creado → pestaña **Inbound security rules**
2. Borra cualquier regla genérica como `AllowAnyAny`
3. Agrega las siguientes reglas (una por una):

#### ✅ Permitir SSH desde tu IP:

* Nombre: `allow-ssh`
* Prioridad: `100`
* Puerto: `22`
* Protocolo: `TCP`
* Origen: `Mi IP`
* Acción: **Allow**

#### ✅ Permitir HTTP desde cualquier IP:

* Nombre: `allow-http`
* Prioridad: `200`
* Puerto: `80`
* Protocolo: `TCP`
* Origen: `Any`
* Acción: **Allow**

#### ❌ Bloquear todo lo demás:

* Nombre: `deny-rest`
* Prioridad: `300`
* Puerto: `*`
* Protocolo: `Any`
* Origen: `Any`
* Acción: **Deny**

✅ *Estas reglas aseguran que solo SSH y HTTP estén permitidos.*

---

### 3️⃣ Asociar el NSG a una subred o una NIC

#### Opción A – Asociar a una **subred** (afecta a todas las VMs dentro)

1. En el NSG → pestaña **Subnets**
2. Haz clic en **Associate**
3. Elige:

   * **Virtual network**: red existente
   * **Subred**: por ejemplo, `default` o `web-subnet`
4. Asociar

#### Opción B – Asociar a una **interfaz de red (NIC)** de una VM

1. Ve a **Máquinas virtuales** → selecciona tu VM
2. Pestaña **Redes** → **Network interface**
3. En “Security group”, haz clic en **Editar**
4. Selecciona `nsg-web-seguro` y guarda

---

### 4️⃣ Validar el acceso desde fuera

1. Lanza una VM en la subred o NIC protegida:

   * Nombre: `vm-nsg-test`
   * SO: Ubuntu o Amazon Linux
   * Puerto: habilita solo 22 en el asistente (o ninguno si usas NSG)
   * IP pública habilitada
   * Usuario: `azureuser` y clave o SSH

2. Desde tu máquina local, intenta:

```bash
ssh azureuser@<ip-publica>
```

✅ Debe funcionar solo desde tu IP

3. Instala un servidor web en la VM:

```bash
sudo apt update && sudo apt install apache2 -y
```

4. Abre el navegador y accede a `http://<ip-publica>`
   ✅ Debe cargarse la página de bienvenida de Apache

5. Intenta conectarte por otro puerto (ej. 3389, 5000) → debe fallar

---

## ✅ Validación

* ¿Puedes conectar por SSH desde tu IP?
* ¿Puedes acceder al puerto 80 (HTTP) desde el navegador?
* ¿Otros puertos están bloqueados?
* ¿El NSG está correctamente asociado a la red o VM?

---

## ⚠️ Errores comunes

| Problema                  | Causa común                          | Solución                                              |
| ------------------------- | ------------------------------------ | ----------------------------------------------------- |
| No puedes acceder por SSH | NSG no permite tu IP, o falta regla  | Añade regla `allow-ssh` y verifica origen             |
| HTTP no responde          | Apache no instalado o NSG lo bloquea | Asegura que puerto 80 esté permitido                  |
| El NSG no tiene efecto    | No está asociado correctamente       | Verifica si está vinculado a la subred o NIC correcta |

---

## 🧩 Extensiones opcionales

* Crea un **NSG separado para tráfico interno** entre subredes.
* Asocia NSG a múltiples VMs con distintos roles (web, base de datos).
* Usa Azure CLI para crear el NSG y las reglas (`az network nsg rule create`).
* Crea una regla de **alta prioridad que bloquee ICMP (ping)**.