# 🔐 Almacenamiento seguro en Azure: Blob, Files y Managed Disks

---

## 🗂️ Introducción

Microsoft Azure ofrece múltiples servicios de almacenamiento, cada uno optimizado para distintos escenarios técnicos y de seguridad. En este archivo nos enfocamos en los tres más importantes:

| Tipo         | Servicio      | Uso principal                                   |
| ------------ | ------------- | ----------------------------------------------- |
| **Objetos**  | Blob Storage  | Datos no estructurados, backups, data lakes     |
| **Archivos** | Azure Files   | Compartidos NFS/SMB para aplicaciones           |
| **Bloques**  | Managed Disks | Discos de VM, alta disponibilidad y rendimiento |

Cada uno ofrece mecanismos robustos de **cifrado**, **control de acceso**, **aislamiento de red** y **resiliencia**.

---

## 📦 Azure Blob Storage (almacenamiento de objetos)

### 🔐 Seguridad en tránsito y en reposo

* **HTTPS obligatorio**: las cuentas deben forzar conexiones seguras.
* **TLS 1.2+**: obligatorio en clientes modernos; puede requerir configuración en SDKs.

```bash
az storage account update \
  --name miaccount \
  --resource-group mi-rg \
  --enable-https-traffic-only true
```

* **Cifrado en reposo**:

  * Por defecto: claves gestionadas por Microsoft.
  * Alternativa: **CMK** (Customer-Managed Key) en **Key Vault** (BYOK).
  * Soporta cifrado doble (Double Encryption) en escenarios específicos.

```bash
az storage account update \
  --name miaccount \
  --resource-group mi-rg \
  --encryption-key-source Microsoft.Keyvault \
  --keyvault-encryption-key-uri https://mi-keyvault.vault.azure.net/keys/clave/versión
```

### 👮 Control de acceso

| Mecanismo                         | Uso recomendado                            |
| --------------------------------- | ------------------------------------------ |
| **RBAC (Azure AD)**               | Ideal para acceso interno y por roles      |
| **SAS (Shared Access Signature)** | Acceso delegado y temporal (API, terceros) |

**SAS Tokens**:

* Siempre con **expiración**.
* Preferentemente firmados con **user delegation key** para mayor trazabilidad.
* Pueden revocarse desde el portal o CLI.

```bash
az storage blob generate-sas \
  --account-name miaccount \
  --container-name datos \
  --name archivo.txt \
  --permissions r \
  --expiry 2025-12-31T23:59Z \
  --as-user
```

### 🧱 Inmutabilidad

* **Políticas de retención (WORM)**:

  * `Time-based retention`: define un período en días.
  * `Legal hold`: bloquea hasta que se libera manualmente.

```bash
az storage container immutability-policy set \
  --account-name miaccount \
  --container-name datos \
  --period 30 \
  --policy-mode Unlocked
```

### 🌐 Aislamiento de red

* Habilitar **Private Endpoint** para evitar exposición pública.
* Definir reglas de firewall para permitir sólo rangos IP o subredes específicas.

```bash
az network private-endpoint create \
  --name blob-pe \
  --resource-group mi-rg \
  --vnet-name mi-vnet \
  --subnet subnet-privada \
  --private-connection-resource-id /subscriptions/xxx/resourceGroups/mi-rg/providers/Microsoft.Storage/storageAccounts/miaccount \
  --group-id blob
```

---

## 📁 Azure Files (almacenamiento de archivos)

### 🔐 Seguridad y autenticación

* Protocolos soportados:

  * **SMB** (Windows/Linux): con **Azure AD DS** o **AD DS on-prem**.
  * **NFS 4.1** (preview): para entornos Linux sin necesidad de dominio.

* **Control de acceso**:

  * ACLs sobre recursos compartidos.
  * RBAC a nivel de cuenta de almacenamiento.
  * Requiere cifrado en tránsito y en reposo.

```bash
az storage share-rm create \
  --resource-group mi-rg \
  --storage-account miaccount \
  --name mi-share \
  --access-tier Hot
```

### 🌐 Aislamiento

* Montaje a través de **Private Endpoint**.
* Uso combinado con **Azure Firewall** y grupos de seguridad para limitar accesos.

---

## 💽 Managed Disks (almacenamiento de bloques)

### 🔐 Cifrado en reposo

* Cifrado por defecto con claves gestionadas por Microsoft.
* Para control total, usa **Disk Encryption Sets (DES)** con una **CMK** en **Azure Key Vault**.

```bash
az disk create \
  --resource-group mi-rg \
  --name disco-cifrado \
  --size-gb 128 \
  --sku Premium_LRS \
  --encryption-type EncryptionAtRestWithCustomerKey \
  --disk-encryption-set <DES_ID>
```

> 💡 La clave (CMK) puede tener políticas de rotación, acceso restringido, y auditoría habilitada.

### 🧩 Snapshots y plantillas

* Snapshots cifrados automáticamente.
* Las imágenes pueden almacenarse en **Shared Image Gallery** para estandarizar y reutilizar.

```bash
az snapshot create \
  --resource-group mi-rg \
  --name snap-vm \
  --source /subscriptions/.../discos/mi-vm-disk
```

---

## 🏢 Ejemplo empresarial

**Escenario**: una empresa de salud requiere alta confidencialidad y cumplimiento legal (GDPR, ENS).

| Requisito                        | Servicio      | Configuración recomendada                                              |
| -------------------------------- | ------------- | ---------------------------------------------------------------------- |
| Repositorio de imágenes médicas  | Azure Blob    | Cifrado con CMK, Private Endpoint, SAS firmados con expiración         |
| Carpeta compartida entre equipos | Azure Files   | Integración con Azure AD, ACLs, SMB sobre red privada                  |
| Discos de VM con datos sensibles | Managed Disks | CMK con Key Vault, snapshots protegidos, backups a vault con retención |

---

## ❌ Errores comunes

1. **SAS tokens sin expiración** o compartidos por canales inseguros.
2. **Key Vault sin política de auditoría ni separación de roles.**
3. **Acceso público habilitado por defecto** en cuentas de almacenamiento.
4. **No usar Private Endpoint** para servicios críticos, exponiendo tráfico.
5. **Discos sin cifrado CMK** en entornos con requisitos regulatorios.

---

## ✅ Checklist rápido

| Seguridad                 | Blob             | Files       | Managed Disks         |
| ------------------------- | ---------------- | ----------- | --------------------- |
| Cifrado en reposo         | ✅ (CMK opcional) | ✅           | ✅ (CMK opcional)      |
| Cifrado en tránsito       | ✅                | ✅ (SMB/NFS) | ❌ (no aplica)         |
| RBAC                      | ✅                | ✅           | ✅                     |
| SAS / tokens temporales   | ✅                | ❌           | ❌                     |
| Private Endpoint          | ✅                | ✅           | ✅                     |
| Snapshots / backup        | ✅ (blobs)        | ✅ (files)   | ✅                     |
| Inmutabilidad / retención | ✅                | ❌           | ✅ (a través de vault) |

---

## 🔗 Recursos oficiales

* [Azure Blob Storage Security](https://learn.microsoft.com/en-us/azure/storage/blobs/security-recommendations)
* [Azure Files Identity-based Access](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-identity-auth-active-directory-enable)
* [Azure Disk Encryption](https://learn.microsoft.com/en-us/azure/virtual-machines/disks-enable-customer-managed-keys)
* [Shared Access Signature Best Practices](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)

