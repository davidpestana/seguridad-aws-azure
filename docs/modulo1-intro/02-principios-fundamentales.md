# 📄 `02-principios-fundamentales.md`


## 🧭 Introducción – Principios fundamentales de la seguridad en la nube

Aunque el entorno cloud introduce nuevas herramientas, servicios y modelos operativos, **los principios fundamentales de la seguridad informática siguen plenamente vigentes**. Sin embargo, deben reinterpretarse para adaptarse al carácter dinámico, distribuido y automatizado de la nube.

Este archivo profundiza en cómo se aplican pilares como la **Confidencialidad, Integridad y Disponibilidad (CIA)** en entornos cloud, y añade principios clave como la **trazabilidad** y la **resiliencia**, esenciales para proteger recursos y datos en plataformas como AWS y Azure.

Comprender estos principios es vital para diseñar arquitecturas seguras, establecer controles eficaces y responder ante incidentes.

---

## 🔎 Explicación detallada: principios clásicos y ampliados

### 🔐 1. Confidencialidad

> Evitar el acceso no autorizado a la información.

En cloud, este principio se aplica mediante:

* **Control de acceso basado en roles**:

  * **AWS**: IAM Policies y Roles.
  * **Azure**: RBAC (Role-Based Access Control).
* **Cifrado de datos**:

  * **En tránsito**: TLS por defecto en la mayoría de servicios gestionados.
  * **En reposo**: cifrado automático con claves gestionadas (KMS en AWS, Key Vault en Azure).
* **Segmentación lógica de recursos**:

  * VPCs, subredes privadas, grupos de seguridad, NSG (Azure), etc.

> Ejemplo: una base de datos en AWS con cifrado KMS y accesible solo desde un role IAM asociado a una Lambda específica.

---

### ✅ 2. Integridad

> Garantizar que los datos no han sido alterados de forma no autorizada.

En la nube, se aplica a través de:

* **Control de versiones y auditoría**:

  * Versionado de objetos (S3, Azure Blob Storage).
  * Logs de actividad (CloudTrail, Azure Activity Logs).
* **Validaciones automáticas**:

  * Funciones Lambda / Azure Functions que verifican hashes, firmas, integridad de archivos.
* **Protección contra cambios manuales**:

  * Infraestructura como código (IaC) + políticas inmutables.

> Ejemplo: almacenar evidencias firmadas digitalmente en un blob storage con versionado habilitado y monitorización continua.

---

### ♻️ 3. Disponibilidad

> Asegurar el acceso a los sistemas y datos cuando se necesiten.

Esto implica diseñar para:

* **Alta disponibilidad (HA)**:

  * Distribuir instancias entre zonas de disponibilidad (AZ) o regiones.
* **Escalabilidad automática**:

  * Auto Scaling Groups (AWS), Azure Scale Sets.
* **Tolerancia a fallos**:

  * Backups automatizados, failover (RDS/Azure SQL Geo-replication).

> Ejemplo: un sitio web desplegado con AWS Elastic Beanstalk o Azure App Service configurado para múltiples zonas.

---

### 🛰️ 4. Trazabilidad

> Conocer **quién hizo qué, cuándo y desde dónde**.

En la nube, la trazabilidad es una ventaja clave:

* **Auditoría centralizada y continua**:

  * **AWS**: CloudTrail (actividad API), Config (historial de recursos), CloudWatch Logs.
  * **Azure**: Azure Monitor, Activity Logs, Log Analytics.
* **Integración con SIEM y alertas**:

  * Exportar eventos a servicios como AWS Security Hub, Microsoft Sentinel.
* **Etiquetado y seguimiento de recursos**:

  * Etiquetas (`tags`) para trazar propietarios, costes, propósitos.

> Ejemplo: detectar una acción de borrado en una base de datos por parte de un usuario no autorizado gracias a un log de CloudTrail o Activity Log.

---

### 🧱 5. Resiliencia

> Capacidad de un sistema para **recuperarse rápidamente** ante fallos o ataques.

Incluye:

* **Diseño redundante**:

  * Uso de múltiples zonas/regiones.
  * Copias en caliente o frías.
* **Plan de recuperación ante desastres (DRP)**:

  * Recuperación automatizada de infraestructura usando Terraform, CloudFormation o Bicep.
* **Protección activa**:

  * **AWS**: Shield, WAF, GuardDuty.
  * **Azure**: Azure Firewall, DDoS Protection, Defender.

> Ejemplo: despliegue de backups de configuración de red con Terraform en otra región tras un fallo regional.

---

## 🏢 Caso empresarial – Aplicación de los principios en una aseguradora digital

La aseguradora digital **PoliCloud** despliega su core transaccional en Azure y sus APIs públicas en AWS por razones estratégicas.

* Implementan **IAM y RBAC** con privilegios mínimos.
* Activan cifrado con claves gestionadas por cliente en ambos entornos.
* Automatizan despliegues con Terraform y configuran alertas en Azure Monitor y CloudWatch.
* Prueban periódicamente su **plan de recuperación**, restaurando la infraestructura en una región secundaria.

Resultado: cumplimiento de requisitos de seguridad exigidos por normativas ISO/NIST y reducción significativa del tiempo de respuesta ante incidentes.

---

## 💻 Código y configuración

### 🔐 AWS – Política de cifrado en reposo para S3 con KMS

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ForceS3Encryption",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::mi-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "aws:kms"
        }
      }
    }
  ]
}
```

### 📑 Azure – Activar cifrado en reposo en Azure SQL

```bash
az sql server tde set \
  --resource-group MiRG \
  --server MiServidorSQL \
  --status Enabled
```

---

## 🧪 Tips para probar

### En consola web

* **AWS**:

  * Ir a S3 → habilitar el versionado → subir archivos → probar recuperación de versiones.
  * Acceder a CloudTrail → ejecutar acción y ver en el log.

* **Azure**:

  * Ver actividad reciente desde Azure Monitor.
  * Configurar alertas en Log Analytics con consultas Kusto (KQL).

### En CLI

* Probar recuperación de backups:

  ```bash
  aws rds restore-db-instance-from-db-snapshot \
    --db-instance-identifier restaurada \
    --db-snapshot-identifier mi-snapshot
  ```

* Ver logs de actividad:

  ```bash
  az monitor activity-log list --max-events 5
  ```

---

## ❌ Errores comunes

| Error                              | Descripción                             | Cómo prevenirlo                                    |
| ---------------------------------- | --------------------------------------- | -------------------------------------------------- |
| ❌ Confiar solo en firewalls        | No cubrir identidades y configuraciones | Aplicar seguridad multinivel                       |
| ❌ Desactivar logging por coste     | Perder trazabilidad clave               | Revisar costes y activar almacenamiento optimizado |
| ❌ No probar planes de recuperación | Falsa sensación de seguridad            | Automatizar pruebas de DR                          |

---

## ✅ Buenas prácticas

✔️ Checklist recomendado:

* [ ] Activar el versionado en buckets y blobs.
* [ ] Configurar cifrado con claves propias si es necesario (KMS, Key Vault).
* [ ] Registrar y almacenar logs de actividad desde el primer día.
* [ ] Etiquetar todos los recursos críticos.
* [ ] Diseñar entornos tolerantes a fallos en múltiples zonas.

---

## 📚 Recursos recomendados

| Recurso                                                                                                                                | Descripción                                               |
| -------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| [AWS Security Pillar – Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/)                | Guía base para seguridad en AWS                           |
| [Azure Security Best Practices](https://learn.microsoft.com/en-us/security/azure-security/fundamentals/best-practices)                 | Recomendaciones de Microsoft para seguridad en Azure      |
| [NIST SP 800-53](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)                                                      | Controles de seguridad recomendados a nivel federal (USA) |
| [Microsoft Azure CAF – Security](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/security) | Framework de adopción cloud seguro en Azure               |