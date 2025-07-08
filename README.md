# 🏛 Think-Tank  

**Extracción, procesamiento y almacenamiento estructurado de datos médicos desde un archivo PDF.**

![](https://www.iimas.unam.mx/wp-content/uploads/2023/11/Logo-pagina-ok.png)

---

## 📑 Tabla de Contenidos
*Haz clic en la sección que deseas consultar*

1. [🧾 Información del Proyecto](#🧾-información-del-proyecto)
2. [📋 Requisitos](#📋-requisitos)
3. [📂 Archivos](#📂-archivos)
   - [📗 header_footer_to_df.py](#-header_footer_to_dfpy)
   - [📘 extract_pdf.py](#-extract_pdfpy)
   - [📕 main.py](#-mainpy)
4. [🚀 Configuración de la ruta del PDF](#Configuración de la ruta del PDF)
5. [📜 Ejemplo de Salida](#📜-ejemplo-de-salida)

---

## 🧾 Información del Proyecto

Este repositorio resuelve un problema común en hospitales: la extracción automatizada de datos clínicos desde archivos PDF que no siguen un formato fijo.

Está diseñado para transformar documentos médicos en papel o en PDF (escaneos o generados digitalmente) en datos estructurados y útiles, que puedan analizarse, visualizarse o integrarse en otros sistemas como bases de datos o expedientes clínicos electrónicos.

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

### 📗 `header_footer_to_df.py`

Se encarga de **extraer los datos administrativos** del encabezado y pie de página de las notas clínicas.

#### Funciones principales:
- `get_head()`: Número de nota, expediente, tipo, HIM.
- `get_patient_data()`: Nombres, apellidos (incluyendo partículas), edad, sexo, fecha de nacimiento.
- `get_medical_data()`: Fechas de ingreso, alta, creación de nota, médico firmante y cédula.
- `convert_to_df()`: Convierte todos los datos extraídos en un `DataFrame` organizado.

---

### 📘 `extract_pdf.py`

Este módulo es el **motor de extracción estructurada**. A partir de texto posicionado y características visuales del PDF, convierte el documento en una estructura de datos organizada.

#### Funcionalidad principal:
1. **Lectura posicional** usando `pdfplumber`.
2. **Segmentación por secciones** mediante lógica de formato (títulos, mayúsculas, alineación).
3. **Extracción de contenido** por bloques, texto plano o tablas.
4. **Flexibilidad** para distintos tipos de notas (evolución, alta, interconsulta, etc.).

#### Uso de `Levenshtein`:
Permite identificar secciones o etiquetas mal escritas o variables. Por ejemplo:
- "Diagnóstico" vs "Dx Activo"
- "Estudios" vs "Laboratorio/Imagen"

Así, el motor es **tolerante a errores** y adaptable a múltiples plantillas de hospital.

#### Métodos clave:
- `extract_header_footer_text()`
- `get_table(section_name)`
- `get_subsections(section_name)`
- `extract_blocks_from_text()`

---

### 📕 `main.py`

Este script es el **punto de entrada del proyecto**. Ejecuta el proceso completo desde la lectura del PDF hasta la generación del JSON final con los datos clínicos.

#### Funciones:
- `df_to_dict(df)`: Convierte un `DataFrame` en lista de diccionarios para exportación.
- `main()`: Orquesta todo el flujo:
  - Carga el PDF.
  - Extrae encabezado y pie de página.
  - Extrae secciones clínicas.
  - Construye el JSON estructurado.

---

### Configuración de la ruta del PDF

Antes de ejecutar el script, es fundamental configurar correctamente la ruta del archivo PDF en la variable global `ruta_pdf`. Una vez que la ruta esté definida, el programa estará listo para ejecutarse.

**Ejemplo de cómo especificar la ruta dentro del script:**
Por defecto, la variable `ruta_pdf` está configurada para apuntar a la carpeta EJEMPLO NOTAS DE UN PACIENTE 7 DIAS y al archivo nota1.pdf. El valor predeterminado es el siguiente:
`ruta_pdf = "./data/EJEMPLO NOTAS DE UN PACIENTE 7 DIAS/nota1.pdf"`

Si deseas utilizar una ruta diferente, modifica la variable ruta_pdf según el sistema operativo de la siguiente manera:

    ruta_pdf = "C:/Users/Usuario/Escritorio/think-tank/data/archivo.pdf"  # Windows
    ruta_pdf = "/home/usuario/Escritorio/think-tank/data/archivo.pdf"  # Linux

# Ejemplo de Salida.

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
