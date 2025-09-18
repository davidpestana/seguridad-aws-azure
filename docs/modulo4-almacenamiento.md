## 💾 Módulo 4: Gestión de Datos y Almacenamiento Seguro (AWS y Azure)

---

## 🎯 Objetivos del módulo

Al terminar este módulo serás capaz de:

* Elegir el servicio de almacenamiento adecuado (objetos, bloques, archivos) en AWS y Azure según caso de uso, coste y requisitos de seguridad.
* Configurar **cifrado en reposo y en tránsito** con claves gestionadas por el proveedor o por el cliente (KMS / Key Vault).
* Aplicar **controles de acceso** (IAM/RBAC, políticas, SAS) y **aislamiento de red** (endpoints privados) para minimizar exposición.
* Diseñar **estrategias de resiliencia** (replicación, versiones, snapshots, inmutabilidad) alineadas con RPO/RTO.
* Asegurar cumplimiento y trazabilidad (logs de acceso, auditoría de claves, políticas de retención).
* Evitar errores frecuentes (público por defecto, claves incrustadas, HTTPS deshabilitado, permisos excesivos).

---

## 🧭 Índice detallado del módulo

---

### 1) Introducción al almacenamiento seguro en la nube

✅ Objetivo
Entender el mapa de opciones de almacenamiento (objetos, bloques, archivos) y cómo cambia la seguridad cuando los datos viven en servicios gestionados, multi-inquilino y con alcance global.

📌 Claves
Modelo compartido · Objetos/Bloques/Archivos · Coste vs riesgo · Clasificación de datos · Políticas de retención

🧩 Contenido
En cloud, almacenar datos no es “crear un disco y ya”. Debes elegir el **tipo** correcto:

* **Objetos**: S3 / Blob (ideal para estáticos, logs, backups, datasets).
* **Bloques**: EBS / Managed Disks (volúmenes para VMs y bases de datos autogestionadas).
* **Archivos**: EFS / Azure Files (NFS/SMB para compartidos).

Cada opción implica controles de seguridad distintos: políticas a nivel de objeto/contenedor, cifrado, endpoints privados, auditoría. Veremos cómo **clasificar datos** (sensibles, personales, regulados) y asociar **requisitos**: cifrado obligatorio, inmutabilidad, retención legal, **coste por acceso** (hot/cool/archive).

Errores típicos: exponer contenedores/“buckets” públicamente “para probar”, no forzar HTTPS, no activar versiones, no controlar claves KMS/Key Vault.
📚 docs/modulo4-almacenamiento/01-introduccion.md

---

### 2) Almacenamiento en AWS (S3, EBS, EFS)

✅ Objetivo
Dominar los mecanismos de seguridad de S3 (objetos), EBS (bloques) y EFS (archivos), y cuándo usar cada uno de forma segura.

📌 Claves
S3 Block Public Access · Bucket Policies vs IAM · SSE-S3 / SSE-KMS / CMK · Versioning/Object Lock · VPC Endpoints (Gateway/Interface)

🧩 Contenido
**S3 (objetos)**:

* **Bloqueo de acceso público** a nivel de cuenta/bucket para evitar exposiciones accidentales.
* Cifrado en reposo: **SSE-S3** (gestionado por AWS) o **SSE-KMS** con **CMK** (clave gestionada por cliente).
* **Policies** (bucket) vs **IAM** (identidad) y **ACLs** (legado): recomienda policies + IAM.
* **Versioning** + **Object Lock (WORM)** para inmutabilidad y protección ante ransomware.
* **VPC Endpoints** (Gateway para S3) para acceso privado sin Internet.
* **Classes**: Standard / IA / One Zone-IA / Glacier Instant / Flexible / Deep Archive → seguridad + costes.

**EBS (bloques)**:

* Cifrado por defecto con KMS; control de **snapshots** (encriptados, compartidos, cross-account).
* Gestión de **copias** y **lifecycle** con AWS Backup.
* Aislamiento: adjuntar solo a instancias dentro de la misma AZ; controlar IAM en operaciones de snapshot/restore.

**EFS (archivos)**:

* NFS con **encriptación en tránsito** (TLS) y en **reposo**.
* **Access Points** para aislar paths por aplicación/usuario.
* Montaje a través de **mount targets** en subredes privadas; combinar con **Security Groups** restrictivos.

Casos de uso:

* Estático web/logs → S3 con CloudFront y OAC (sin público directo).
* Datos de BD en EC2 → EBS con snapshots y KMS-CMK.
* Carpeta compartida multi-instancia → EFS con Access Points y SG.
  📚 docs/modulo4-almacenamiento/02-almacenamiento-en-aws.md

---

### 3) Almacenamiento en Azure (Blob, Files, Managed Disks)

✅ Objetivo
Configurar seguridad en Azure Blob (objetos), Files (SMB/NFS) y Managed Disks (bloques), con RBAC, SAS y Private Endpoints.

📌 Claves
Private Endpoints · RBAC vs SAS · Encryption at Rest (MS-managed vs CMK en Key Vault) · Immutability Policies · Replicación LRS/ZRS/GRS/RA-GRS

🧩 Contenido
**Blob Storage (objetos)**:

* **HTTPS Only** y **requerir TLS**.
* **RBAC** para acceso vía Azure AD (preferible) y **SAS** de tiempo limitado para accesos delegados (firma con **user delegation key**).
* **Encryption at rest** gestionado por Microsoft o **CMK** con **Key Vault** (BYOK).
* **Immutability Policies** (legal hold / time-based) a nivel de contenedor para WORM.
* **Private Endpoints** para que el tráfico no salga a Internet.
* Niveles de acceso **Hot/Cool/Archive** para optimizar costes.

**Azure Files (archivos)**:

* SMB/NFS con **AD DS/Azure AD DS** para identidad.
* **Private Endpoints**, ACLs y **RBAC** para controlar acceso.
* Cifrado en reposo y en tránsito.

**Managed Disks (bloques)**:

* Cifrado en reposo por defecto; **CMK** opcional con Key Vault.
* **Snapshots** y **Image Gallery** para estandarizar despliegues.
* **Disk Encryption Sets** para asociar discos a CMKs concretas.

Casos de uso:

* Data lake/logs → Blob con Private Endpoint y lifecycle a Archive.
* Compartido Windows → Azure Files con identidad AD y Private Endpoint.
* Discos de VM sensibles → Managed Disks con CMK y snapshots protegidos.
  📚 docs/modulo4-almacenamiento/03-almacenamiento-en-azure.md

---

### 4) Cifrado en tránsito y en reposo

✅ Objetivo
Asegurar que los datos están cifrados **siempre**: cuando viajan (TLS) y cuando se guardan (SSE y claves CMK).

📌 Claves
TLS 1.2+ · HTTPS Only · SSE-S3/SSE-KMS · CMK/Customer-Managed Keys · Key rotation · Auditoría de uso de claves

🧩 Contenido
**En tránsito**:

* Forzar **HTTPS/TLS 1.2+** en endpoints (Azure: “Secure transfer required”; AWS: **aws\:SecureTransport=true** en bucket policy).
* Evitar clientes antiguos; validar certificados, HSTS en frontales públicos.

**En reposo**:

* **AWS**: S3 (SSE-S3 o **SSE-KMS** con **CMK**), EBS/EFS cifrados por defecto con KMS.
* **Azure**: Storage Service Encryption gestionado por Microsoft o **CMK** desde **Key Vault**.
* Diseñar **rotación** de claves, **separación de roles** (quién usa vs quién administra claves), **logs de acceso** a KMS/Key Vault.
* **Cifrado doble** cuando lo exija la normativa (algunas verticales).

Casos de examen/entrevista: ¿cuándo usar CMK frente a claves del proveedor? → cuando necesitas **control del ciclo de vida**, **auditoría fina**, **revocación** o **escrow**.
📚 docs/modulo4-almacenamiento/04-cifrado-y-proteccion.md

---

### 5) Control de acceso a datos

✅ Objetivo
Aplicar controles de acceso robustos: identidad, políticas, condiciones (IP, VPC/subred), tokens temporales y aislamiento de red.

📌 Claves
IAM/Bucket Policy vs ACL · Azure RBAC · SAS con expiración · Condiciones por red (VPC Endpoint/Private Endpoint) · Secure Transfer

🧩 Contenido
**AWS**:

* Preferir **Bucket Policies + IAM**; evitar **ACLs** salvo interoperabilidad heredada.
* Condiciones: **aws\:SourceVpce**, **aws\:SecureTransport**, rangos IP, etiquetas de recurso.
* **S3 Access Points** para aislar accesos por aplicación.
* **STS** para credenciales temporales (menor superficie de ataque).

**Azure**:

* **RBAC** con Azure AD para acceso de apps/usuarios; **SAS** para compartir temporalmente (revocables).
* **Private Endpoints** para acceso privado; reglas de firewall por **red/vnet**.
* Políticas para **bloquear público** y exigir cifrado.

Casos de uso:

* Compartir objetos a terceros: SAS (Azure) o URL firmadas vía aplicación (AWS).
* Limitar acceso a datos solo desde workload interno: VPC Endpoint/Private Endpoint obligatorio + política que bloquee Internet.
  📚 docs/modulo4-almacenamiento/05-control-de-acceso.md

---

### 6) Redundancia, replicación y disponibilidad

✅ Objetivo
Diseñar resiliencia según RPO/RTO: versión, replicación/geo-replicación, snapshots, políticas de retención y recuperación.

📌 Claves
Versioning · Object Lock · CRR/SRR (S3) · LRS/ZRS/GRS/RA-GRS (Azure) · Snapshots · Backup cross-account

🧩 Contenido
**AWS**:

* **S3**: Versioning + **Replication** (SRR intra-región / **CRR** cross-región), **Object Lock** para WORM y retención legal.
* **EBS**: snapshots (mejor **cross-account** para aislar de compromisos).
* **EFS**: replication y backups con AWS Backup.

**Azure**:

* **Redundancia**: **LRS** (intra-DC), **ZRS** (multi-zona), **GRS/RA-GRS** (geo-replicación).
* **Snapshots**: Managed Disks, File Shares, Blob versioning + soft delete.
* **Backup**: Azure Backup, políticas por workload, vaults separados.

Diseño: elegir la **redundancia mínima** para cumplir el **RPO/RTO** y presupuesto. Combinar con **inmutabilidad** para ransomware.
📚 docs/modulo4-almacenamiento/06-redundancia-y-disponibilidad.md

---

### 7) Buenas prácticas en gestión de almacenamiento

✅ Objetivo
Consolidar un checklist de implementación segura y sostenible (coste/control), con auditoría continua y automatización.

📌 Claves
Bloqueo de público · HTTPS Only · CMK/Key Vault con rotación · Versioning/Soft delete · Lifecycle/Archivado · Logging/Auditoría · Policy/Blueprint

🧩 Contenido

* **Bloquear acceso público** por defecto (S3 Block Public Access, “Public access level: Private” en Azure).
* **Forzar HTTPS** y negar peticiones no cifradas.
* **Cifrado** siempre, con **CMK/Key Vault** donde aplique; **rotación** y **separación de deberes**.
* **Versioning/Soft Delete** + **inmutabilidad** para proteger ante borrados/alteraciones.
* **Lifecycle** hacia clases frías (S3 IA/Glacier; Blob Cool/Archive) con controles de acceso equivalentes.
* **Logging y auditoría**: S3 Server Access Logs / CloudTrail data events; Azure Monitor / Diagnostic settings.
* **Automatizar cumplimiento**: AWS Config Rules / Azure Policy & Blueprints.
* **Detectar datos sensibles**: Macie (AWS) / Microsoft Purview (Azure).
* **Backups** aislados (cross-account/segregated vaults), pruebas periódicas de restore.

Errores frecuentes:

* Contenedores/buckets públicos “para pruebas” que permanecen años.
* SAS sin expiración, o compartidos vía chat.
* KMS/Key Vault sin logs ni rotación.
* Desactivar TLS “temporalmente” en entornos internos.
  📚 docs/modulo4-almacenamiento/07-buenas-practicas.md
