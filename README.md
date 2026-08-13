# 📊 Documentación de Datasets y Metodología - ConnectaTel

## 💾 Descripción de los Datasets

### 1. `plans.csv` (Información de Planes)
Contiene las tarifas base y los costos adicionales de los planes ofrecidos (**Básico** y **Premium**):

* `plan_name`: Nombre del plan (`Basico` / `Premium`).
* `messages_included`: Cantidad de mensajes de texto incluidos al mes.
* `gb_per_month`: Gigabytes de datos incluidos al mes.
* `minutes_included`: Minutos de llamadas incluidos al mes.
* `usd_monthly_pay`: Tarifa fija mensual (USD).
* `usd_per_gb`: Costo por GB adicional (USD).
* `usd_per_message`: Costo por mensaje adicional (USD).
* `usd_per_minute`: Costo por minuto adicional (USD).

### 2. `users_latam.csv` (Información de Clientes)
Registro demográfico y suscripción de 4,000 usuarios:

* `user_id`: Identificador único del usuario.
* `first_name` / `last_name`: Nombre y apellido del cliente.
* `age`: Edad del usuario.
* `city`: Ciudad de residencia (Medellín, CDMX, Bogotá, GDL, MTY, Cali, etc.).
* `reg_date`: Fecha de alta en el servicio.
* `plan`: Plan contratado (`Basico` o `Premium`).
* `churn_date`: Fecha de cancelación del servicio (nulo si el cliente sigue activo).

### 3. `usage.csv` (Uso Real de Servicios)
Registro de 40,000 eventos de comunicación durante 2024:

* `id`: Identificador del evento.
* `user_id`: Identificador del usuario que realizó la acción.
* `type`: Tipo de interacción (`call` para llamada, `text` para mensaje).
* `date`: Fecha y hora de la interacción.
* `duration`: Duración de la llamada en minutos (solo aplica a `type == 'call'`).
* `length`: Longitud/caracteres del mensaje (solo aplica a `type == 'text'`).

---

## 🛠️ Metodología y Flujo de Trabajo

### Paso 1: Carga y Exploración Inicial
* Carga masiva de datasets utilizando `pandas`.
* Evaluación preliminar de dimensiones (`.shape`), tipos de datos y estructuras (`.info()`, `.head()`).

### Paso 2: Diagnóstico de Calidad de Datos
* **Valores Ausentes:**
  * `churn_date`: ~88.35% de nulos (corresponde a clientes activos, no requiere eliminación).
  * `city`: ~11.7% de nulos e inconsistencias.
  * `duration` y `length`: Presentan valores nulos faltantes en patrón **MAR (Missing At Random)** dependientes del tipo de servicio (`length` es nulo para llamadas y `duration` es nulo para SMS).
* **Valores Sentinels e Invalidez:**
  * Presencia del valor centinela `-999` en la columna `age`.
  * Presencia de caracteres `'?'` en la columna `city`.
* **Inconsistencia en Fechas:**
  * Identificación de registros futuros (año 2026) en `reg_date`.

### Paso 3: Limpieza y Tratamiento de Datos
* Reemplazo de sentinels `-999` en `age` por la mediana (47 años).
* Conversión de `'?'` a valores nulos estandarizados (`pd.NA`) en `city`.
* Imputación/Coerción de fechas imposibles en `reg_date` (fechas > 2025 convertidas a `pd.NaT`).
* Manejo adecuado de nulos por estructura de evento en `usage`.

### Paso 4: Agregación de Métricas por Usuario
* Construcción de variables agregadas en `usage`:
  * `cant_mensajes`: Total de mensajes enviados por el usuario.
  * `cant_llamadas`: Total de llamadas realizadas.
  * `cant_minutos_llamada`: Suma de minutos consumidos.
* Integración mediante `merge` con `users_latam` para crear la vista consolidada `user_profile`.
* Análisis de distribución del tipo de plan: **64.88% Básico** vs **35.12% Premium**.

### Paso 5: Análisis Exploratorio y Visualización (EDA)
* Generación de histogramas comparativos por plan tarifario mediante `seaborn` y `matplotlib`:
  * Distribución de edad por plan.
  * Patrones de envío de mensajes.
  * Frecuencia y duración acumulada de llamadas.
