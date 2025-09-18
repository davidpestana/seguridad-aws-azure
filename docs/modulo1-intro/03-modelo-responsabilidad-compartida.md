# 📄 `03-modelo-responsabilidad-compartida.md`


## 🧭 Introducción – ¿Qué es el modelo de responsabilidad compartida?

Uno de los conceptos más importantes –y más malinterpretados– en seguridad cloud es el **modelo de responsabilidad compartida**.
En él se define qué elementos son responsabilidad del **proveedor cloud** (AWS, Azure) y cuáles recaen sobre el **cliente/usuario**.

Este modelo no es fijo: **varía en función del tipo de servicio** que estés utilizando (IaaS, PaaS o SaaS). Entenderlo correctamente evita errores graves como dejar datos expuestos, malconfigurar redes, o no aplicar cifrado.

---

## 🔎 Explicación detallada – ¿Cómo se distribuyen las responsabilidades?

### 🧩 1. ¿Quién hace qué?

Ambos proveedores, AWS y Azure, utilizan un modelo similar:

| Área de seguridad                        | Responsable en IaaS | Responsable en PaaS | Responsable en SaaS |
| ---------------------------------------- | ------------------- | ------------------- | ------------------- |
| Seguridad física del datacenter          | Cloud Provider      | Cloud Provider      | Cloud Provider      |
| Infraestructura de red y hardware        | Cloud Provider      | Cloud Provider      | Cloud Provider      |
| Virtualización (hipervisores, nodos)     | Cloud Provider      | Cloud Provider      | Cloud Provider      |
| Sistema operativo y parches              | Cliente             | Cloud Provider      | Cloud Provider      |
| Configuración de red / firewalls         | Cliente             | Parcial             | Cloud Provider      |
| Cuentas, usuarios, contraseñas           | Cliente             | Cliente             | Cliente             |
| Cifrado de datos (en tránsito/en reposo) | Cliente             | Cliente/Compartido  | Compartido          |
| Gestión de identidades y roles           | Cliente             | Cliente             | Cliente             |
| Código de la aplicación / lógica         | Cliente             | Cliente             | No aplica           |
| Datos y protección de datos              | Cliente             | Cliente             | Cliente             |

> ⚠️ Importante: **el proveedor te da las herramientas, pero tú debes activarlas y configurarlas correctamente.**

---

### 🧭 2. Visualización del modelo

#### 📌 AWS – Shared Responsibility Model

![AWS Shared Responsibility](https://d1.awsstatic.com/security-center/AWS_Security_Shared_Responsibility_Model.a18153b0917891f19f956b0e1523aa346a51b03e.png)
📎 Fuente: [AWS Security Documentation](https://aws.amazon.com/compliance/shared-responsibility-model/)

#### 📌 Azure – Shared Responsibility Matrix

![Azure Shared Responsibility](https://learn.microsoft.com/en-us/media/azure-shared-responsibility-model.png)
📎 Fuente: [Microsoft Azure Trust Center](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility)

---

### 📊 3. Comparativa práctica entre proveedores

| Aspecto                             | AWS                                                  | Azure                                        |
| ----------------------------------- | ---------------------------------------------------- | -------------------------------------------- |
| Modelo visual                       | Diagrama por niveles (infra → software)              | Matriz por capas y tipo de servicio          |
| Enfoque                             | Seguridad **del** cloud vs seguridad **en** el cloud | Idéntico concepto                            |
| Recursos de ayuda                   | Security Hub, Well-Architected Tool                  | Microsoft Defender for Cloud, CAF (Security) |
| Modo de alerta ante incumplimientos | AWS Config Rules, Security Hub                       | Azure Policy, Compliance Insights            |

---

## 🏢 Ejemplo empresarial – Confusión en una migración

La empresa **DataNova** migra su CRM a una máquina virtual en Azure (modelo IaaS).

El equipo técnico asume que “Azure se encarga de la seguridad” y no configura:

* Firewall de red
* Autenticación multifactor
* Cifrado en disco
* Actualizaciones del sistema operativo

🔍 Resultado: en 2 semanas, la VM es atacada con fuerza bruta desde IPs externas, obteniendo acceso administrador.
El incidente se debió a que **el cliente era responsable de esas configuraciones**, pero no lo sabía.

🔒 Lecciones aprendidas:

* El proveedor da herramientas, pero **la responsabilidad final del uso es del cliente**.
* Es esencial **leer y entender las matrices de responsabilidad** antes de desplegar un servicio.

---

## 💻 Código y configuración

### ✅ AWS – Activar alertas en Security Hub por configuraciones inseguras

```bash
aws securityhub enable-security-hub
aws securityhub batch-enable-standards \
  --standards-subscription-requests \
  '[{"StandardsArn":"arn:aws:securityhub:::standards/aws-foundational-security-best-practices/v/1.0.0"}]'
```

### ✅ Azure – Aplicar políticas de cumplimiento con Azure Policy

```bash
az policy assignment create \
  --name "enforce-vm-encryption" \
  --scope "/subscriptions/ID_SUBSCRIPCION" \
  --policy "/providers/Microsoft.Authorization/policyDefinitions/enforce-vm-encryption"
```

---

## 🧪 Tips para probar

### 🧰 En consola web

* **AWS**:

  * Accede a Security Hub → Activar estándar “AWS Foundational Best Practices”
  * Accede a IAM Access Analyzer → Verifica configuraciones peligrosas.

* **Azure**:

  * Visita “Microsoft Defender for Cloud” → Revisa recomendaciones.
  * Ir a “Políticas” → Asignar definiciones a un grupo de recursos.

### 🧪 En CLI

* Ver si tu Security Hub está activo:

  ```bash
  aws securityhub get-enabled-standards
  ```

* Listar políticas en Azure:

  ```bash
  az policy definition list --query "[].{Name:displayName, Mode:mode}" -o table
  ```

---

## ❌ Errores comunes

| Error                                                     | Consecuencia                                          | Solución                                                         |
| --------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------------- |
| Pensar que todo está protegido por el proveedor           | Exposición de datos, brechas, malware                 | Estudiar el modelo según servicio usado                          |
| Usar IaaS como PaaS sin asegurar OS, red, actualizaciones | Ataques de día cero o escaladas de privilegio         | Automatizar hardening inicial                                    |
| No aplicar políticas organizativas ni auditoría           | Entornos desordenados, difícil cumplimiento normativo | Usar herramientas nativas de gobierno (Azure Policy, AWS Config) |

---

## ✅ Buenas prácticas

✔️ Checklist recomendado:

* [ ] Antes de desplegar, identificar el tipo de servicio: IaaS / PaaS / SaaS.
* [ ] Revisar la matriz de responsabilidad oficial (AWS o Azure).
* [ ] Activar monitoreo de cumplimiento de buenas prácticas.
* [ ] Aplicar políticas restrictivas por defecto (deny-all, luego whitelisting).
* [ ] Educar a todos los equipos (DevOps, SecOps, etc.) en sus responsabilidades.

---

## 📚 Recursos recomendados

| Recurso                                                                                                                              | Descripción                                      |
| ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------ |
| [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)                                    | Página oficial del modelo de AWS                 |
| [Azure Shared Responsibility in Cloud](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility)          | Modelo de responsabilidad en Microsoft Azure     |
| [AWS Foundational Security Best Practices](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-standards-fsbp.html) | Estándar de referencia en Security Hub           |
| [Microsoft Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction)           | Herramienta de seguridad y cumplimiento en Azure |
