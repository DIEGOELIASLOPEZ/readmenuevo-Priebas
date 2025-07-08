# 🏛 Think-Tank  

**Extracción, procesamiento y almacenamiento estructurado de datos médicos desde un archivo PDF.**

![](https://www.iimas.unam.mx/wp-content/uploads/2023/11/Logo-pagina-ok.png)

---

## 📑 Tabla de Contenidos
*Haz clic en la sección que deseas consultar*

1. [🧾 Información del Proyecto](#-información-del-proyecto)
2. [📋 Requisitos](#-requisitos)
3. [📂 Archivos](#-archivos)
   - [📗 header_footer_to_df.py](#-header_footer_to_dfpy)
   - [📘 extract_pdf.py](#-extract_pdfpy)
   - [📕 main.py](#-mainpy)
4. [🚀 Configuración de la ruta del PDF](#-Configuración-de-la-ruta-del-pdf)
5. [📜 Ejemplo de salida](#Ejemplo-de-salida)
---
## 🧾 Información del Proyecto

Este repositorio resuelve una necesidad específica del **Hospital Infantil de México Federico Gómez**: la **extracción automatizada de información clínica contenida en archivos PDF** generados por su sistema interno. Estos documentos siguen un formato y contienen datos médicos relevantes que actualmente no pueden ser aprovechados fácilmente para su análisis o visualización.

El principal objetivo de este proyecto es **transformar las notas clínicas médicas en formato PDF en estructuras de datos limpias, organizadas y utilizables**. Esto permite que la información contenida en estos documentos pueda ser:

- **Consultada rápidamente** por usuarios clínicos o administrativos.
- **Visualizada gráficamente** en paneles o tableros informativos.
- **Analizada de forma automatizada** con herramientas estadísticas o de inteligencia artificial.
- **Integrada en otras plataformas** como expedientes clínicos electrónicos, sistemas hospitalarios o páginas web.

A través del uso de técnicas de procesamiento de texto posicional (mediante `pdfplumber`) y lógica de segmentación basada en el diseño visual del documento, el sistema es capaz de:

- Detectar secciones clave incluidas en los expedientes como: “Signos Vitales”, “Diagnóstico”, “Órdenes Médicas”, “Evolución”, entre otras.
- Extraer tanto **texto libre** (narrativo, descripciones clínicas) como **tablas de datos** (por ejemplo, mediciones biométricas o medicamentos prescritos).
- Organizar esta información en un formato JSON estructurado, validado y estandarizado.

Además, se busca que la solución sea:

- **Reproducible**: cualquier desarrollador podrá ejecutarlo fácilmente con solo instalar las dependencias.
- **Dinámico**: el código está modularizado para facilitar la extracción de las distintas notas clínicas existentes del Hospital.
---

## 📋 Requisitos

### Versión de Python
- **Python 3.12.3** (mínimo recomendado)

###  Librerías estándar de Python
Estas vienen instaladas por defecto con Python:

| Librería | Propósito |
|----------|-----------|
| `datetime` | Manejo de fechas: nacimiento, ingreso, alta, creación de nota. |
| `typing` | Anotaciones de tipo (`List`, `Dict`, etc.). Mejora la claridad y documentación del código. |
| `re` | Uso de expresiones regulares para búsqueda y extracción de patrones de texto. |

---

###  Librerías externas (requieren instalación con `pip`)

| Librería | Descripción / Propósito |
|----------|--------------------------|
| `pandas==2.2.3` | Manipulación de datos en estructuras tipo `DataFrame`. Se usa para almacenar la información extraída del PDF de manera estructurada. |
| `numpy` | Librería numérica usada principalmente para compatibilidad o cálculos matriciales (uso mínimo en este proyecto). |
| `pdfplumber==0.11.5` | Extracción de texto desde PDFs con información posicional (x, y, tamaño de fuente). Fundamental para leer contenido mal estructurado. |
| `Levenshtein==3.0.1` | Calcula la distancia de edición entre cadenas. Se usa para identificar etiquetas similares en secciones del PDF, incluso con errores tipográficos. |

---

## 📂 Archivos

### 🧾 `header_footer_to_df.py`

Este módulo se encarga de **extraer y estructurar los datos generales** que aparecen en el encabezado y pie de página de las notas médicas extraídas de un PDF. Su objetivo principal es convertir esa información desorganizada en un `DataFrame` limpio, estructurado y listo para su posterior análisis o integración.

Incluye funciones que permiten detectar automáticamente:

- El nombre completo del paciente, separando correctamente los apellidos, incluso si son compuestos o contienen partículas como “de la”, “del”, “y”, etc.
- Fecha de nacimiento, sexo y edad del paciente.
- Número de expediente, número de nota, tipo de nota y código HIM.
- Fechas clave: ingreso, alta y creación de la nota médica.
- Nombre completo y cédula profesional del médico firmante.
- Nombre del hospital donde se generó la nota.

#### 🔧 Funciones principales

- `get_head(text, df)`  
  Extrae información administrativa como el número de nota, tipo de nota, número de expediente y código HIM.

- `get_patient_data(text, df)`  
  Detecta y separa los apellidos paterno y materno, así como los nombres del paciente. También obtiene su fecha de nacimiento, sexo y edad. Es compatible con nombres largos o con partículas compuestas.

- `get_medical_data(text, df)`  
  Extrae datos del contexto médico: fechas de ingreso, alta y creación de la nota, así como el nombre completo del médico firmante y su cédula profesional.

- `convert_to_df(text_list)`  
  Función integradora que toma una lista de líneas de texto del encabezado/pie y devuelve un `DataFrame` unificado con toda la información anterior de forma estructurada.


---

### 📘 `extract_pdf.py`


Este módulo representa el **núcleo del motor de extracción estructurada** para documentos PDF médicos. Se basa en una clase principal (`PDF`) que encapsula toda la lógica necesaria para interpretar el contenido posicional de las notas clínicas, con un enfoque adaptable a documentos complejos o mal estructurados.

#### ✅ Funcionalidad principal

`extract_pdf.py` convierte un archivo PDF en un modelo estructurado del contenido médico mediante las siguientes etapas:

1. **Lectura posicional del PDF**  
   Utiliza la librería `pdfplumber` para leer texto, posiciones (x, y), tamaños de fuente y otras características visuales de cada palabra. Esta lectura se guarda en un `DataFrame` que permite analizar el contenido línea por línea.

2. **Segmentación por secciones**  
   Emplea patrones repetitivos del layout médico (como títulos en mayúsculas, alineaciones específicas o palabras clave) para identificar y clasificar secciones del documento, por ejemplo:  
   - Encabezado (header)  
   - Pie de página (footer)  
   - Cuerpo clínico (e.g., "Signos Vitales", "Diagnóstico", "Evolución")

3. **Extracción inteligente de contenido**  
   A través de sus métodos, permite:
   - **Obtener texto estructurado** por sección (`get_subsections`)
   - **Extraer tablas** a partir de regiones con datos tabulares (`get_table`)
   - **Detectar bloques clínicos** dentro del texto continuo (`extract_blocks_from_text`), diferenciando párrafos, indicaciones o ítems por su espaciado vertical o estilo tipográfico.

4. **Flexibilidad para distintos tipos de nota médica**  
   El sistema no depende de una plantilla fija, sino de la lógica visual del documento, por lo que puede adaptarse a diferentes tipos de notas: evolución, alta, triage, interconsulta, enfermería, etc.

#### 🔠 Uso de la librería `Levenshtein`

En este módulo se emplea `Levenshtein` para calcular la **distancia de edición** entre cadenas de texto, lo cual es crucial cuando se necesita:

- Identificar **secciones clínicas aunque estén mal escritas**, abreviadas o con errores tipográficos (por ejemplo, "Dx Activo" vs. "Diagnósticos Activos")
- Detectar **similitudes aproximadas** en encabezados entre distintos PDFs médicos, que pueden variar según el sistema o plantilla utilizada.

Esto permite una **segmentación robusta y tolerante a errores** del contenido, sin depender de coincidencias exactas. Gracias a esto, el motor puede identificar correctamente secciones incluso en documentos con formatos inconsistentes o escaneos defectuosos.


---

### 📕 `main.py`

Este script actúa como el **punto de entrada principal** del proyecto. Es el encargado de orquestar todo el flujo de trabajo, desde la lectura del PDF hasta la construcción final de un archivo JSON estructurado con los datos clínicos extraídos.

Su principal objetivo es integrar todos los módulos del sistema (`extract_pdf.py` y `header_footer_to_df.py`), ejecutar sus funciones clave y generar una salida coherente, útil y estandarizada que pueda ser utilizada para visualización, análisis o carga en una base de datos clínica.

---

#### ✅ Funciones principales:

- `df_to_dict(df)`  
  Convierte un `DataFrame` en una lista de diccionarios, lista para serializarse como JSON. También reemplaza espacios en los nombres de las columnas por guiones bajos para evitar errores.

- `main()`  
  Ejecuta el flujo completo del sistema en los siguientes pasos:

  1. **Carga del PDF**  
     Se inicializa la clase `PDF` con la ruta definida al principio del script. Esto permite cargar todo el contenido posicional del PDF.

  2. **Extracción del encabezado y pie de página**  
     Utiliza la función `extract_header_footer_text()` para obtener los textos de las secciones administrativa y médica. Luego, los datos se estructuran mediante funciones del módulo `header_footer_to_df.py`:
     - `get_head()` para información administrativa.
     - `get_patient_data()` para información del paciente.
     - `get_medical_data()` para datos médicos y del profesional de salud.

  3. **Organización de datos individuales**  
     Se crean tres diccionarios:  
     - `informacion_nota`: contiene número de nota, tipo, expediente, HIM, fechas clave y hospital.  
     - `informacion_paciente`: contiene los datos del paciente como nombre, apellidos, fecha de nacimiento, edad y sexo.  
     - `informacion_medico`: contiene datos del médico firmante como nombre completo, cédula profesional, fecha de creación de la nota, etc.

  4. **Extracción de secciones clínicas**  
     Se itera sobre todas las secciones detectadas en el PDF (excepto encabezado y pie) para extraer:
     - Tablas con información clínica (`get_table`)
     - Texto libre o estructurado (`get_subsections`)  
     Cada sección se almacena en el diccionario `note_sections`.

  5. **Construcción del JSON final**  
     Se crea una estructura JSON con todos los datos organizados por categorías:
     - Información de la nota (`

### 🚀 Configuración de la ruta del PDF

Antes de ejecutar el script, es fundamental configurar correctamente la ruta del archivo PDF en la variable global `ruta_pdf`. Una vez que la ruta esté definida, el programa estará listo para ejecutarse.

**Ejemplo de cómo especificar la ruta dentro del script:**
Por defecto, la variable `ruta_pdf` está configurada para apuntar a la carpeta EJEMPLO NOTAS DE UN PACIENTE 7 DIAS y al archivo nota1.pdf. El valor predeterminado es el siguiente:
`ruta_pdf = "./data/EJEMPLO NOTAS DE UN PACIENTE 7 DIAS/nota1.pdf"`

Si deseas utilizar una ruta diferente, modifica la variable ruta_pdf según el sistema operativo de la siguiente manera:

    ruta_pdf = "C:/Users/Usuario/Escritorio/think-tank/data/archivo.pdf"  # Windows
    ruta_pdf = "/home/usuario/Escritorio/think-tank/data/archivo.pdf"  # Linux

### 📜 Ejemplo de salida

```json
{
    "note_info": {
        "No_nota": "Numero del Nota del Expediente",
        "Tipo_nota": "Tipo de Nota",
        "No_Expediente": "Numero del expediente",
        "HIM": "Identificador HIM",
        "Hospital": "Nombre del Hospital",
        "Fecha_ingreso": "Fecha en que se ingreso al hospotal en formato Datetime (AAAA-MM-DDTHH:MM:SSZ)",
        "Fecha_alta": "Fecha en que se dio de alta al hospotal en formato Datetime (AAAA-MM-DDTHH:MM:SSZ)"
    },
    "patient_info": {
        "Apellido_paterno": "Apelldio Paterno del paciente",
        "Apellido_materno": "Apelldio Materno del paciente",
        "Nombres": "Nombre(s) del paciente",
        "Fecha_nacimiento": "Fecha de nacimiento formato Datetime (AAAA-MM-DDTHH:MM:SSZ)."  ,
        "Sexo": "Sexo del paciente"
    },
    "medical_info": {
        "Nombre_Completo": "Nombre del medico que firmo",
        "Firmado_por": "Firma/Nombre del medico que firmo",
        "Supervisor_nombre": "Nombre del Supervison",
        "Cedula_profesional": "Cedula Profesional",
        "Fecha_creacion": "Fecha de creacion del expediente en formato Datetime (AAAA-MM-DDTHH:MM:SSZ)"
    },
    "sign_info": {
        "Tabla": [
            {
                "Fecha/Hora": "Fecha y Hora en formato Datetime (AAAA-MM-DDTHH:MM:SSZ)",
                "FR": "Frecuencia Respiratoria",
                "FC": "Frecuencia Cardíaca",
                "PAS": "Presión Arterial Sistólica",
                "PAD": "Presión Arterial Diastólica",
                "SAT_O2": "Saturacion",
                "Temp_°C": "Temperatura",
                "Peso": "Peso",
                "Talla": "Talla"
            }
        ],
        "Subjetivo": "Nota con descripcion del diagnostico "
    },
{
    "diagnostic_info": {
        "Examen_Físico": "Descripción del examen físico realizado al paciente, incluyendo hallazgos relevantes.",
        "Diagnósticos_Activos_Tabla": [
            {
                "Fecha_Ingresada": "Fecha y hora en que el diagnóstico fue registrado.",
                "Descripción": "Descripción detallada del diagnóstico del paciente.",
                "Tipo": "Tipo de diagnóstico, por ejemplo, diagnóstico principal o secundario.",
                "Médico": "Nombre del médico que realizó el diagnóstico."
            }
        ],
        "Análisis_Condición": "Descripción del análisis y estado clínico actual del paciente, incluyendo antecedentes relevantes, tratamiento actual y recomendaciones.",
        "Estudios": "Descripción de los estudios realizados o pendientes al paciente, incluyendo resultados previos y planes de estudios adicionales.",
        "Plan_de_Tratamiento": "Plan detallado de tratamiento propuesto para el paciente, con indicaciones específicas para su seguimiento."
    },
    "diet_info": {
        "Tabla": [
            {
                "Fecha_Ingresada": "Fecha en que se ingresó la información dietética del paciente en formato Datetime (AAAA-MM-DDTHH:MM:SSZ)",
                "Tipo": "Tipo de dieta indicada para el paciente, por ejemplo, NPO (Nada por boca), líquida, etc.",
                "Tipo_terapéutico": "Especificación del tipo de dieta terapéutica si aplica, como dieta baja en sal, controlada, etc."
            }
        ]
    },
    "nursing_info": {
        "Tabla": [
            {
                "Fecha_Ingresada": "Fecha en que se ingresó la información dietética del paciente en formato Datetime (AAAA-MM-DDTHH:MM:SSZ)",
                "Orden": " acción específica de enfermería o procedimiento realizado",
                "Médico": "Nombre del médico que realizó el diagnóstico."
            }
        ]
    },
    "medication_info": {
        "Tabla": [
            {
                "Inicio": "Fecha y hora de inicio del tratamiento con el medicamento.",
                "Medicamento": "Nombre y presentación del medicamento administrado al paciente.",
                "Frecuencia": "Frecuencia con la que se administra el medicamento, por ejemplo, cada 8 horas, continua, etc.",
                "Via": "Vía de administración del medicamento, como intravenosa, oral, etc.",
                "Dosis": "Cantidad de medicamento administrado por cada dosis.",
                "UDM": "Unidad de medida utilizada para la dosis, como miligramos, mililitros, etc.",
                "Cantidad": "Cantidad total del medicamento administrado.",
                "Tipo": "Tipo de unidad medida utilizada, como ml, mg, etc.",
                "Médico": "Nombre del médico que prescribió el medicamento."
            }
        ]
    }
}


```


```python
ruta_pdf = "./data/EJEMPLO NOTAS DE UN PACIENTE 7 DIAS/nota1.pdf"
