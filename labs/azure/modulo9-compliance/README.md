# 🧪 Laboratorio 09 – Evaluar y reforzar el cumplimiento normativo con Azure Policy y Blueprints

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Evaluar recursos en Azure frente a políticas de cumplimiento.
* Aplicar políticas predefinidas como **Audit VMs without encryption** o **Disallow public IP**.
* Crear una **iniciativa de políticas** para agrupar controles.
* Asignar un **Blueprint** de gobernanza para asegurar que todas las suscripciones siguen las mismas normas.
* Consultar el estado de cumplimiento desde el portal.

---

## 🧵 Escenario práctico

Tu empresa quiere asegurarse de que todos los recursos que se crean en Azure cumplan con las políticas corporativas, como el cifrado obligatorio o la prohibición de IPs públicas. Como responsable técnico, vas a aplicar políticas usando **Azure Policy**, agruparlas en una iniciativa, y luego aplicar un **Blueprint** con esas políticas a una suscripción.

---

## 🧰 Requisitos previos

* Suscripción de Azure activa.
* Permisos de **Owner** o **Policy Contributor** en la suscripción.
* Azure CLI (opcional para validaciones).

---

## 🧭 Pasos detallados

### 1️⃣ Acceder a Azure Policy

1. Ve al portal de Azure → busca y entra en **"Policy"**.
2. En el menú lateral, haz clic en **Definitions**.
3. Filtra por **Built-in** para ver políticas predefinidas.

---

### 2️⃣ Asignar una política individual

1. En el buscador, localiza:

   * **“Audit VMs that do not use managed disks”**
2. Haz clic en la política → **Assign**
3. Configura:

   * **Scope**: tu suscripción o grupo de recursos
   * **Exclusions**: opcional
   * **Assignment name**: `auditar-disks-vm`
4. Clic en **Review + Create**

✅ Esto auditará cualquier VM que no use discos administrados (una mala práctica común).

---

### 3️⃣ Crear una iniciativa de políticas (grupo de políticas)

1. Ve a **Definitions → Initiative definitions → + Initiative definition**

2. Configura:

   * **Name**: `iniciativa-seguridad-básica`
   * **Definition location**: tu suscripción

3. Añade las siguientes políticas:

   * **Audit VMs that do not use managed disks**
   * **Audit virtual machines without disaster recovery configured**
   * **Audit network interfaces that aren't associated with a NSG**

4. Crear la iniciativa

---

### 4️⃣ Asignar la iniciativa a la suscripción

1. Ve a **Initiatives → selecciona `iniciativa-seguridad-básica`**
2. Haz clic en **Assign**
3. Configura:

   * **Scope**: tu suscripción
   * **Assignment name**: `iniciativa-global`
   * Acepta los valores por defecto
4. Crear

✅ Ahora puedes auditar múltiples aspectos en un solo lugar.

---

### 5️⃣ Revisar el cumplimiento

1. Ve a **Policy → Compliance**
2. Filtra por:

   * Nombre de la iniciativa o política
   * Scope (grupo de recursos o suscripción)
3. Verás:

   * Total de recursos evaluados
   * Cuántos cumplen y cuántos no

✅ Puedes hacer clic para ver los recursos no conformes.

---

### 6️⃣ (Opcional) Aplicar un Azure Blueprint

1. Ve a **Blueprints → Definitions → Create blueprint**

2. Nombre: `blueprint-cumplimiento`

3. Location: Management group o suscripción

4. Añade artefactos:

   * **Policy assignment**: asigna la iniciativa creada antes
   * (Opcional) **Resource group**, **role assignments**, etc.

5. **Publish** el blueprint y luego **Assign** a una suscripción.

✅ Esto garantiza que cada nueva implementación cumple con tus políticas.

---

## ✅ Validación

* ¿Se asignaron correctamente las políticas individuales?
* ¿La iniciativa agrupa varias políticas de cumplimiento?
* ¿El estado de cumplimiento muestra recursos no conformes?
* ¿El blueprint aplica correctamente la iniciativa en nuevas suscripciones?

---

## ⚠️ Errores comunes

| Problema                         | Causa común                                 | Solución                                                            |
| -------------------------------- | ------------------------------------------- | ------------------------------------------------------------------- |
| Estado de cumplimiento en blanco | Recursos aún no evaluados                   | Espera unos minutos o fuerza evaluación                             |
| No se pueden asignar políticas   | Permisos insuficientes                      | Asegúrate de ser *Owner* o *Policy Contributor*                     |
| Blueprint no aplica nada         | No se publicaron o asignaron los artefactos | Revisa que el blueprint esté en estado **Published** y **Assigned** |

---

## 🧩 Extensiones opcionales

* Exporta los resultados de cumplimiento a **Log Analytics** o **Power BI**.
* Usa **Azure Policy as Code** con Bicep o Terraform.
* Crea políticas **personalizadas** (por ejemplo: naming convention obligatoria).
* Habilita **Defender for Cloud** para complementar con detección de amenazas.