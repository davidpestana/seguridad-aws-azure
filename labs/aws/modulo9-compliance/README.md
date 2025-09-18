# 🧪 Laboratorio 09 – Evaluar el cumplimiento normativo con AWS Config y conformance packs

## 🎯 Objetivos del laboratorio

Al finalizar este laboratorio, serás capaz de:

* Activar **AWS Config** para auditar configuraciones de recursos.
* Usar **reglas gestionadas** para comprobar cumplimiento (por ejemplo, cifrado en S3, restricciones de acceso, etc.).
* Aplicar un **conformance pack** predefinido (por ejemplo, CIS AWS Foundations Benchmark).
* Generar un informe de cumplimiento para una región o cuenta.
* Entender cómo usar AWS Config como base de auditoría continua.

---

## 🧵 Escenario práctico

Tu empresa necesita demostrar que su entorno AWS cumple con políticas de seguridad como el **Benchmark CIS**, **GDPR**, o controles internos de auditoría. Para ello, vas a habilitar **AWS Config**, activar reglas de cumplimiento y aplicar un **conformance pack preconfigurado**.

---

## 🧰 Requisitos previos

* Cuenta de AWS con permisos de administrador.
* Región: puedes usar `us-east-1` o la que uses por defecto.
* Permisos sobre AWS Config, S3 (para logs), IAM (para roles de servicio).

---

## 🧭 Pasos detallados

### 1️⃣ Crear un bucket S3 para almacenar los logs de Config

```bash
aws s3 mb s3://config-logs-<tutoken> --region us-east-1
```

⚠️ Usa un nombre único y reemplaza `<tutoken>` por tu nombre o alias.

---

### 2️⃣ Activar AWS Config desde la consola

1. Ve a [https://console.aws.amazon.com/config](https://console.aws.amazon.com/config)
2. Clic en **Get started** (si es la primera vez)
3. Configura:

   * **Bucket S3**: selecciona el que acabas de crear (`config-logs-<tutoken>`)
   * **Record all resources**: ✅ sí
   * **Include global resources**: ✅ sí
   * **AWS Config role**: crear automáticamente
4. Habilita **SNS** (opcional) para notificaciones

Clic en **Confirm** → Se activará la recopilación de configuraciones.

---

### 3️⃣ Habilitar reglas gestionadas

1. Ve a **Rules → Add rule**
2. Activa algunas reglas como:

   * `s3-bucket-server-side-encryption-enabled`
   * `iam-user-mfa-enabled`
   * `ec2-instance-no-public-ip`
3. Ejecuta el análisis.

✅ Verás el estado de cumplimiento para cada recurso.

---

### 4️⃣ Aplicar un Conformance Pack (Benchmark CIS)

1. Ve a **Conformance packs → Deploy conformance pack**
2. Nombre: `cis-aws-foundations-benchmark`
3. Template source: **AWS managed**
4. Pack: selecciona `CIS AWS Foundations Benchmark v1.2.0`
5. S3 bucket para resultados: puedes usar el mismo que el de logs
6. Crear

✅ Se desplegará un conjunto completo de reglas CIS.

---

### 5️⃣ Ver resultados del cumplimiento

1. Ve a **Conformance packs**

2. Haz clic sobre `cis-aws-foundations-benchmark`

3. Revisa las reglas y estado de cada una:

   * **Compliant**
   * **Noncompliant**
   * **Not applicable**

4. Puedes exportar los resultados o consultarlos por CLI:

```bash
aws config describe-compliance-by-config-rule
```

---

## ✅ Validación

* ¿Se ha activado AWS Config con éxito?
* ¿Las reglas gestionadas muestran resultados (compliant/noncompliant)?
* ¿El conformance pack CIS se ha aplicado correctamente?
* ¿Puedes ver un resumen del cumplimiento por recurso?

---

## ⚠️ Errores comunes

| Problema                     | Causa común                                       | Solución                                           |
| ---------------------------- | ------------------------------------------------- | -------------------------------------------------- |
| Bucket S3 no válido          | Nombre duplicado                                  | Usa un nombre globalmente único                    |
| Reglas no se ejecutan        | Config no completamente activado                  | Espera unos minutos o verifica IAM role            |
| Muchos recursos no compliant | Configuración real no alinea con buenas prácticas | Ajusta configuraciones para cumplir con las reglas |

---

## 🧩 Extensiones opcionales

* Crea un **conformance pack personalizado** con reglas propias.
* Automatiza la respuesta con **AWS Systems Manager** o **EventBridge**.
* Exporta resultados a **Athena** o **QuickSight** para análisis visual.
* Integra Config con **Security Hub** para correlación de alertas.