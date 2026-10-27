# P1. Que tempo fixo?

Práctica de **Sistemas de Big Data (UD1)**. Se construye un pequeño *pipeline* de datos meteorológicos: descargar, inspeccionar, limpiar, analizar, visualizar y almacenar datos diarios en PostgreSQL. Se trabaja con la estación de **A Coruña / Alvedro** durante **2021–2025**.

## 1. Datos y procedencia

| Campo | Información |
|---|---|
| Estación | A Coruña / Alvedro (aeropuerto) |
| Indicativo | `08002` |
| Coordenadas | Latitud `43.3000`, longitud `-8.3833` |
| Altitud | `97 m` |
| Período | Del 01/01/2021 al 31/12/2025 |
| Origen meteorológico | [Meteostat](https://meteostat.net/) |
| Archivo de esta práctica | [Repositorio de datos](https://github.com/bigdatawirtz/datasets/blob/main/datasets/meteo_alvedro_2021_2025.csv) |
| URL de descarga directa | [CSV original de la práctica](https://raw.githubusercontent.com/bigdatawirtz/datasets/main/datasets/meteo_alvedro_2021_2025.csv) |
| Fecha de descarga | **[COMPLETAR: fecha real de la primera descarga]** |

**Importante:** el CSV utilizado procede de datos meteorológicos de Meteostat, pero **ha sido modificado para esta actividad docente** con errores de calidad introducidos intencionadamente. Por ello, las anomalías que aparecen en el fichero no deben atribuirse directamente a Meteostat.

**Condiciones de uso:** la [documentación de licencia de Meteostat](https://dev.meteostat.net/license) indica que los datos redistribuidos por Meteostat se ofrecen, con carácter general, bajo **Creative Commons Attribution 4.0 (CC BY 4.0)**: se permite compartirlos y adaptarlos con atribución a Meteostat y a sus proveedores, e indicando los cambios. También deben respetarse las condiciones aplicables a la fuente original cuando corresponda. En este proyecto se cita a Meteostat y se indica que el conjunto empleado ha sido modificado para fines didácticos.

## 2. Requisitos

Para reproducir el proyecto se necesita:

- **Git**, para clonar el repositorio.
- **Python 3** y la posibilidad de crear un entorno virtual.
- **Visual Studio Code** con extensiones **Python** y **Jupyter** para ejecutar el notebook.
- **Docker Desktop** con Docker Compose. En Windows con WSL, iniciar Docker Desktop y habilitar la integración con la distribución WSL que se utilice.
- Conexión a Internet para instalar dependencias, descargar la imagen de PostgreSQL y obtener el CSV.

Los comandos siguientes están preparados para una **terminal Ubuntu/WSL**, como la utilizada en la práctica. Ejecutar los comandos de instalación desde la raíz del repositorio, salvo que se indique otra carpeta.

## 3. Reproducir la práctica desde cero

### Paso 1. Clonar el repositorio

```bash
git clone https://github.com/mma-wirtz/p1-que-tempo-fixo.git
cd p1-que-tempo-fixo
```

### Paso 2. Crear y activar el entorno virtual

Se crea en el directorio personal de WSL, fuera de la carpeta compartida de Windows, para evitar problemas de permisos.

```bash
python3 -m venv ~/.venvs/p1-que-tempo-fixo
source ~/.venvs/p1-que-tempo-fixo/bin/activate
```

La terminal debe mostrar `(p1-que-tempo-fixo)` al principio. Comprobar el intérprete con:

```bash
which python
```

### Paso 3. Instalar las dependencias de Python

```bash
python -m pip install -r requirements.txt
```

Esto instala pandas, NumPy, Matplotlib, requests, ipykernel, SQLAlchemy, psycopg2 y python-dotenv, según el archivo `requirements.txt`.

### Paso 4. Preparar las variables de entorno

```bash
cp .env.example .env
```

Abrir `.env` y rellenar las variables que aparecen en la plantilla:

```dotenv
POSTGRES_USER=meteo
POSTGRES_PASSWORD=elige_una_contrasena_local
POSTGRES_DB=meteo_db
POSTGRES_PORT=5433
```

El usuario, la contraseña, el nombre de la base de datos y el puerto deben corresponderse con la configuración que se desea utilizar. **No subir `.env` al repositorio**: Git lo ignora. `.env.example` sí se publica porque es solo una plantilla sin credenciales reales.

### Paso 5. Iniciar Docker Desktop y PostgreSQL

Asegurarse de que Docker Desktop está abierto y operativo. Desde la **raíz del proyecto**, ejecutar:

```bash
docker compose config --quiet
docker compose up -d
docker ps
```

`docker ps` debe mostrar un contenedor llamado `p1-postgres` en ejecución. Para verificar que PostgreSQL acepta conexiones, ejecutar:

```bash
docker compose exec postgres sh -c 'pg_isready -U "$POSTGRES_USER" -d "$POSTGRES_DB"'
```

La respuesta debe incluir `accepting connections`.

El archivo `compose.yaml` define el servicio de PostgreSQL y toma las variables del archivo `.env`. El volumen de Docker conserva los datos entre reinicios del contenedor.

### Paso 6. Abrir y ejecutar el notebook

Desde la **raíz del repositorio**, abrir VS Code:

```bash
code .
```

1. Abrir `notebooks/01_exploracion_datos.ipynb`.
2. Seleccionar como *kernel* el Python del entorno `~/.venvs/p1-que-tempo-fixo/bin/python`.
3. **Importante:** ejecutar el notebook con su directorio de trabajo en `notebooks/`, ya que utiliza rutas relativas como `../datos/` y `../.env`.
4. Elegir **Run All / Ejecutar todas** para ejecutar las celdas de arriba abajo.
5. Dejar Docker y PostgreSQL en funcionamiento hasta que terminen las consultas SQL.

El notebook debe realizar por sí mismo la descarga del CSV en `datos/raw/`, la limpieza y guardado en `datos/clean/`, los análisis y gráficas, y la carga/consultas de PostgreSQL. **No hay que descargar ni copiar manualmente los CSV.**

Si se vuelve a ejecutar, `to_sql(if_exists="replace")` reconstruye la tabla `meteo_alvedro` para no duplicar registros.

### Paso 7. Comprobar el resultado

Al finalizar sin errores, se debe poder verificar:

- `datos/raw/meteo_alvedro_2021_2025.csv`: archivo descargado sin modificar.
- `datos/clean/meteo_alvedro_2021_2025_clean.csv`: archivo limpio guardado en UTF-8.
- Tablas, estadísticas y **tres gráficas** mostradas en el notebook.
- **1826 fechas** en los datos limpios, entre el 01/01/2021 y el 31/12/2025.
- Tabla `meteo_alvedro` en PostgreSQL con **1826 filas**.
- Tres consultas SQL y sus comparaciones con pandas, todas con resultado `True`.

Los datos diarios ausentes que se incorporan al calendario se mantienen como valores nulos; por tanto, **1826 fechas no significa 1826 días con todas las mediciones disponibles**.

Para detener el servicio al terminar, desde la raíz del repositorio:

```bash
docker compose down
```

Esto detiene y elimina el contenedor sin eliminar el volumen de datos. **No utilizar `down -v`** si se desea conservar la base de datos.

## 4. Organización del repositorio

```text
p1-que-tempo-fixo/
├── README.md
├── requirements.txt
├── .gitignore
├── .env.example
├── compose.yaml
├── datos/
│   ├── raw/
│   └── clean/
├── notebooks/
│   └── 01_exploracion_datos.ipynb
└── output/
```

Las carpetas `datos/raw/` y `datos/clean/` se entregan **sin archivos CSV** en GitHub: el notebook los crea al ejecutarse. El archivo `.env` tampoco se publica. El notebook se entrega **ejecutado, con los resultados y las gráficas visibles**.

## 5. Limpieza y validación de los datos

El conjunto original se conserva en `datos/raw/` sin modificaciones. La exploración del formato y de los problemas de calidad se documenta paso a paso en el notebook.

| Problema encontrado | Cómo se detectó | Decisión tomada |
|---|---|---|
| Metadatos antes de la cabecera | Inspección de los primeros bytes y líneas | Omitir las 6 primeras líneas al leer con pandas, conservando intacto el archivo original. |
| Codificación y formato no estándar | Inspección del fichero y prueba de lectura | Leer con `latin-1`, separador `;`, coma decimal y punto de millares. Guardar el resultado limpio en UTF-8. |
| Valores `-9999` como ausentes | Revisión de valores extremos y códigos de ausencia | Convertir esos valores a `NaN` para evitar que alteren medias y gráficos. |
| Código `Ip` en precipitación | Inspección de valores no numéricos de lluvia | Tratarlo como precipitación inapreciable (`0 mm`), según el criterio documentado en el notebook. |
| Fechas duplicadas | Búsqueda de duplicados por fecha | Resolver las 13 fechas duplicadas conservando el primer registro; distinguir los duplicados idénticos de los contradictorios. |
| Fechas que faltan | Comparación con un calendario diario completo | Incorporar las 27 fechas ausentes como filas sin mediciones, sin inventar observaciones. |
| Temperaturas inverosímiles o picos aislados | Inspección de rangos, estadísticas y visualizaciones | Marcar como ausente el dato de temperatura considerado no fiable. |
| Temperatura media inconsistente con la mínima/máxima | Comparación entre variables del mismo día | Marcar como ausente la media inconsistente, preservando los valores considerados válidos. |
| Precipitaciones negativas u otras inconsistencias | Comprobación de rangos y coherencia entre columnas | Revisar y tratar los valores inválidos según las reglas del notebook. |
| Ausencias que permanecen tras limpiar | Conteo de nulos por columna | Mantener como `NaN` las mediciones que no se pueden reconstruir justificadamente; evitar rellenarlas con valores inventados. |

**Resultado:** el conjunto final contiene 1826 fechas diarias ordenadas, de 2021 a 2025, y no presenta fechas duplicadas. De esas fechas, **27 no tenían registro en el fichero inicial** (incluido el intervalo del 3 al 23 de julio de 2023). Esta ausencia debe tenerse en cuenta al interpretar los análisis, especialmente los acumulados de precipitación.

## 6. Análisis y resultados

El notebook incluye las cuatro preguntas del apartado 4.4, tres gráficas y la comparación de eficiencia del apartado 4.6. A modo de resumen:

| Año | Temperatura media anual |
|---|---:|
| 2021 | 14,12 °C |
| 2022 | 14,98 °C |
| 2023 | 14,91 °C |
| 2024 | 14,45 °C |
| 2025 | 14,45 °C |

- **Mes más lluvioso de media:** noviembre, con **152,12 mm** de acumulado mensual medio.
- **Mes más seco de media:** agosto, con **13,42 mm** de acumulado mensual medio.
- **Día lluvioso:** se considera aquel con **al menos 1 mm** de precipitación.
- El notebook también calcula los días lluviosos por mes y las fechas más calurosas y frías de cada año.
- La operación vectorizada de amplitud térmica (`tmax − tmin`) fue más rápida que el bucle `for`, y la diferencia aumentó al multiplicar por cinco el tamaño del DataFrame.

### PostgreSQL

PostgreSQL se ejecuta con Docker Compose. Se carga la tabla limpia `meteo_alvedro` con pandas `to_sql()` y se realizan tres consultas SQL, cotejadas con pandas:

1. Temperatura media anual.
2. Precipitación acumulada mensual media.
3. Días por año con temperatura máxima **≥ 30 °C**.

La tercera consulta devolvió **5, 8, 11, 3 y 6 días** para 2021, 2022, 2023, 2024 y 2025, respectivamente. Las tres comparaciones SQL/pandas coinciden. Como faltan algunas observaciones, estos recuentos son de días calurosos **confirmados**, no necesariamente todos los ocurridos.

## 7. Uso de inteligencia artificial

Se utilizó **ChatGPT (OpenAI)** como apoyo durante el desarrollo para comprender el enunciado, aclarar conceptos de pandas, SQL, Docker y entornos virtuales, revisar fragmentos de código y ayudar a estructurar la documentación. Las decisiones de limpieza, la ejecución del notebook y las comparaciones de resultados entre pandas y PostgreSQL se comprobaron en el entorno de trabajo de la autora.

La IA se utilizó como **herramienta de apoyo**, no como fuente de datos meteorológicos ni como sustituto de las verificaciones.

## 8. Entrega

La entrega se realiza mediante un **enlace al repositorio de GitHub** en Moodle. El repositorio debe mantener `datos/` sin los CSV, conservar `.env` fuera de Git y contener el notebook **ejecutado con salidas visibles**. La fecha límite es la indicada en Moodle.
