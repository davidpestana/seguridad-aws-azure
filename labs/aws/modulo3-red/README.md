# 🧪 Laboratorio 03 – Configurar Security Groups y NACLs para controlar el tráfico

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Crear y aplicar **Security Groups** (SG) para controlar el tráfico a instancias EC2.
* Configurar **Network ACLs (NACLs)** para definir reglas a nivel de subred.
* Entender las diferencias entre **SG y NACL** en AWS.
* Validar el bloqueo o autorización de tráfico entrante/saliente.

---

## 🧵 Escenario práctico

Tu empresa necesita asegurar que solo el tráfico permitido alcance sus instancias EC2. Vas a desplegar una pequeña infraestructura con una instancia en subred pública, y protegerás el acceso:

* Usarás un **Security Group** que permita solo SSH desde tu IP.
* Aplicarás un **NACL** para bloquear tráfico no deseado en la subred.

---

## 🧰 Requisitos previos

* Haber completado un laboratorio de VPC con subred pública (como `lab01.md`).
* Una VPC activa (`vpc-segura-lab`) con una subred pública (`subnet-publica-a`).
* AWS CLI instalada (opcional para validación).
* IP pública desde la que harás SSH (puedes obtenerla en [https://ifconfig.me](https://ifconfig.me)).

---

## 🧭 Pasos detallados

### 1️⃣ Crear un Security Group para SSH limitado

1. En la consola, accede a **EC2** → **Security Groups** → **Create security group**

2. Configura:

   * **Nombre**: `sg-ssh-limitado`
   * **VPC**: `vpc-segura-lab`

3. En **Inbound rules**, añade:

   * Tipo: SSH
   * Protocolo: TCP
   * Puerto: 22
   * Origen: `Mi IP` (automáticamente detectado)

4. En **Outbound rules**, deja la regla por defecto (`Allow all`)

---

### 2️⃣ Lanzar una instancia EC2 con ese SG

1. Ve a **EC2** → **Launch Instance**
2. Nombre: `instancia-sg-test`
3. Amazon Linux 2, t2.micro
4. Red: `vpc-segura-lab`
5. Subred: `subnet-publica-a`
6. IP pública: ✅ habilitada
7. **Security Group**: usar `sg-ssh-limitado`
8. Clave: selecciona o crea una (`.pem`)

✅ Al lanzar la instancia, anota su IP pública.

---

### 3️⃣ Probar acceso SSH a la instancia

```bash
ssh -i mi-clave.pem ec2-user@<ip-publica>
```

✅ Deberías poder acceder **solo desde tu IP actual**.
Prueba desde otro sitio (VPN o proxy) y verás que falla.

---

### 4️⃣ Ver y editar el NACL de la subred pública

1. Ve a **VPC** → **Network ACLs**
2. Busca el NACL asociado a `subnet-publica-a`
3. Observa sus reglas:

#### Por defecto:

* **Entrante**: `100 ALLOW 0.0.0.0/0` para todos los puertos
* **Saliente**: `100 ALLOW 0.0.0.0/0` para todos los puertos

4. Edita las reglas **entrantes**:

| Rule # | Type | Protocol | Port Range | Source        | Allow/Deny |
| ------ | ---- | -------- | ---------- | ------------- | ---------- |
| 100    | SSH  | TCP      | 22         | tu IP pública | ALLOW      |
| 110    | ALL  | ALL      | ALL        | 0.0.0.0/0     | DENY       |

5. Borra cualquier regla `ALLOW` para `0.0.0.0/0` que esté antes.

✅ Con esto, el NACL también filtra a nivel de subred.

---

### 5️⃣ Validar bloqueo y acceso

* Intenta conectar por SSH desde tu IP ✅ → Accede
* Intenta desde otra IP ❌ → Denegado

Opcionalmente, revisa logs de conexión con:

```bash
sudo yum install tcpdump
sudo tcpdump -i eth0 port 22
```

---

## ✅ Validación

* ¿Puedes acceder por SSH solo desde tu IP?
* ¿Está el Security Group correctamente limitado?
* ¿El NACL bloquea otros orígenes?
* ¿Puedes comprobar tráfico denegado desde otra red?

---

## ⚠️ Errores comunes

| Problema                          | Causa                                           | Solución                                   |
| --------------------------------- | ----------------------------------------------- | ------------------------------------------ |
| Conexión rechazada                | IP no permitida en SG o NACL                    | Verifica origen en ambos                   |
| Conexión silenciosamente ignorada | NACL bloquea antes que SG                       | NACL evalúa primero, SG después            |
| Reglas en conflicto               | Reglas que se solapan o tienen orden incorrecto | Asegura que `DENY` esté después de `ALLOW` |

---

## 🧩 Extensiones opcionales

* Crea una subred privada y aplica NACL aún más restrictivo.
* Usa `CloudWatch` o `VPC Flow Logs` para auditar tráfico.
* Crea un bastion host en la subred pública para acceder a instancias privadas.