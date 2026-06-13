# CAT Extractor de Estadísticas del Catastro

**Notebook de Google Colab para descargar y estructurar automáticamente las estadísticas públicas de la Dirección General del Catastro de España.**

![Plataforma](https://img.shields.io/badge/plataforma-Google%20Colab-F9AB00?logo=googlecolab&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)
![Licencia](https://img.shields.io/badge/licencia-MIT-green)
![Estado](https://img.shields.io/badge/estado-estable-brightgreen)

---

## El problema: extracción manual, lenta y propensa a errores

El portal de estadísticas del Catastro ([catastro.hacienda.gob.es](https://www.catastro.hacienda.gob.es)) organiza su información en más de 25 tablas distribuidas en 7 bloques temáticos. Para obtener los datos de una sola provincia y un solo año, el flujo manual obliga a:

1. Navegar a cada URL de bloque.
2. Seleccionar manualmente provincia y año en menús desplegables.
3. Hacer clic en «Consultar todo» y esperar la ventana emergente.
4. Copiar o descargar la tabla resultante.
5. Repetir el proceso tabla por tabla.

Para un análisis multianual o multiprovincial, el número de operaciones manuales crece hasta hacer el proceso inviable. Este notebook automatiza los pasos 1–5 en un único clic.

---

## Características

- **Scraping con Playwright** — Controla un navegador Chromium en modo *headless*, selecciona todos los criterios y captura la tabla emergente, replicando exactamente la interacción humana.
- **Cobertura de 7 bloques de datos** — Catastro Urbano (B1), Anuarios de Distrito (B2), Titularidad (B4), Catastro Rústico (B5), IBI (B8), BICE (B9) y Ordenanzas Fiscales (B12); un total de 29 tablas por provincia y año.
- **Opciones extraídas en tiempo real** — Los menús de provincia y año se obtienen directamente del portal del Catastro en cada ejecución, garantizando que las opciones disponibles estén siempre actualizadas.
- **Limpieza numérica automática** — Elimina separadores de miles, normaliza la coma decimal a punto y convierte columnas a tipo numérico, dejando los CSV listos para análisis en pandas, R o Excel.
- **Soporte de tablas multinivel** — El bloque `05b_rust_porc_tipo_cultivo` tiene doble cabecera con `colspan`; el notebook lo gestiona con un parser especializado que genera encabezados combinados (p. ej. `Cítricos_Sup_%`).
- **Política de reintentos** — Cada descarga se reintenta hasta dos veces ante fallos transitorios de red o de la UI del portal.
- **Reanudación automática** — Si la ejecución se interrumpe, el proceso detecta los CSV ya generados y los salta, evitando trabajo duplicado.
- **Salida descargable** — Un botón comprime todos los CSV en un único archivo `.zip` con nombre normalizado (p. ej. `14_CORDOBA.zip`) e inicia la descarga en el navegador.

---

## Requisitos previos

| Requisito | Notas |
|-----------|-------|
| Cuenta de Google | Necesaria para ejecutar en Google Colab (gratuito). |
| Conexión a internet | El notebook descarga datos del portal del Catastro en tiempo real. |
| Ninguno más | Todas las dependencias se instalan en la primera celda. |

> **Ejecución local:** Es posible ejecutar el notebook fuera de Colab con Jupyter, pero `files.download()` (descarga automática del ZIP) requiere el entorno Colab. Fuera de él, el ZIP se genera en disco y debe copiarse manualmente.

---

## Instalación

No requiere instalación local. Basta con abrir el notebook en Google Colab:

1. Descarga `CAT_Extracc_Estadisticas.ipynb` desde este repositorio.
2. Ve a [colab.research.google.com](https://colab.research.google.com) → **Archivo → Subir notebook**.
3. Selecciona el archivo `.ipynb` descargado.
4. Ejecuta las celdas en orden (ver sección Uso).

Alternativamente, si el notebook está publicado con enlace directo:

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/TU_USUARIO/TU_REPO/blob/main/CAT_Extracc_Estadisticas.ipynb)

---

## Uso

El flujo completo se divide en 8 celdas que se ejecutan en orden. Las interacciones de usuario se concentran en las celdas 3 y 6.

| Celda | Título | Qué hace |
|-------|--------|----------|
| **1** | Instalación de dependencias | Instala `playwright`, `pandas`, `beautifulsoup4`, `ipywidgets` y `lxml`; descarga el navegador Chromium. |
| **2** | Constantes y URLs | Define las plantillas de URL para cada bloque y las funciones que las construyen dinámicamente según año y provincia. |
| **3** | Widgets de selección | ⚠️ **Interacción del usuario.** Muestra tres menús desplegables: provincia, año para bloques B1/B4/B5/B9, año para B2 y año para B8. Selecciona tus valores antes de continuar. |
| **4** | Funciones de descarga | Define la lógica de Playwright: navegar, seleccionar criterios, hacer clic en «Consultar todo» y guardar el HTML resultante. |
| **5** | Parsers HTML → DataFrame | Convierte el HTML descargado en DataFrames de pandas limpios; gestiona tablas con cabecera simple y cabecera multinivel. |
| **6** | Orquestador | ⚠️ **Interacción del usuario.** Pulsa el botón **«Iniciar Extracción Completa»**. El proceso itera sobre las 29 tablas, muestra barra de progreso y guarda un CSV por tabla. |
| **7** | Comprimir y descargar | Pulsa **«Comprimir y Descargar»** para empaquetar todos los CSV en un `.zip` e iniciar la descarga. |
| **8** | Limpieza | Pulsa **«Limpiar Resultados Anteriores»** para borrar la carpeta de resultados y el ZIP del entorno Colab. |

### Ejemplo de salida

Tras ejecutar para Córdoba (código `14`), año 2025 (B1/B4/B5/B9), 2025 (B2) y 2024 (IBI):

```
/content/catastro_Cordoba_2025_2025_2024/
├── 2025_14_01a_variables_catastro.csv
├── 2025_14_01b_parcelas_edificadas_propiedad.csv
├── 2025_14_01c_parcelas_por_superficie.csv
├── 2025_14_01d_superficies_parcelas_urbanas.csv
├── 2025_14_01e_bienes_inmuebles_usos.csv
├── 2025_14_01f_valor_catastral_usos.csv
├── 2025_14_02a_inmuebles_distrito_uso.csv
├── 2025_14_02b_superficie_media_distrito_uso.csv
├── 2025_14_02c_valor_catastral_medio_distrito_uso.csv
├── 2025_14_04a_titulares.csv
├── 2025_14_05a_rust_variables.csv
├── 2025_14_05b_rust_porc_tipo_cultivo.csv
├── 2024_14_08a_ibi_urbano.csv
├── 2024_14_08b_ibi_rustico.csv
├── 2024_14_08c_ibi_bice.csv
├── 2025_14_09a1_produc_energ_electrica.csv
├── ... (29 tablas en total)
└── proceso.log
```

El archivo ZIP descargable se denomina `14_CORDOBA.zip`.

### Ejecución parcial (un solo bloque)

Para procesar únicamente un bloque, edita la variable `SUBSET_PREFIX` en la celda 6 antes de pulsar el botón:

```python
SUBSET_PREFIX = "01"   # Solo el bloque B1 (Catastro Urbano)
SUBSET_PREFIX = "08"   # Solo el bloque B8 (IBI)
SUBSET_PREFIX = None   # Todos los bloques (comportamiento por defecto)
```

---

## Bloques de datos extraídos

| Bloque | Nombre | Contenido |
|:------:|--------|-----------|
| **B1** | Catastro Urbano | Variables generales, parcelas, superficies, usos y valor catastral por municipio. |
| **B2** | Urbana Anuales | Inmuebles, superficie media y valor catastral medio por distrito y uso. |
| **B4** | Titularidad | Número y tipo de titulares por municipio. |
| **B5** | Catastro Rústico | Variables generales y porcentajes por tipo de cultivo. |
| **B8** | IBI | Bases imponibles y cuotas líquidas para IBI urbano, rústico y BICE. |
| **B9** | Catastro BICE | Bienes Inmuebles de Características Especiales (centrales, aeropuertos, huertos solares…). |
| **B12**| Ordenanzas Fiscales | Tipos de gravamen y recargos sobre inmuebles desocupados. |

---

## Arquitectura del notebook

```
CAT_Extracc_Estadisticas.ipynb
│
├── Celda 1  — Entorno
│             Instalación de dependencias y configuración de nest_asyncio.
│
├── Celda 2  — URLs
│             BASE_B1…BASE_B12   Plantillas de URL por bloque.
│             PREFIJOS_B*        Mapas nombre_descriptivo → código_de_fichero.
│             build_urls_*()     Constructores dinámicos de URLs.
│             _prov()            Normaliza códigos de provincia a 2 dígitos.
│
├── Celda 3  — Widgets
│             extraer_opciones_catastro()   Obtiene años y provincias del portal en tiempo real.
│             selector_provincia            Dropdown con las 52 provincias.
│             selector_year_b1/b2/b8        Dropdowns de año por bloque.
│
├── Celda 4  — Scraper (Playwright)
│             _descargar_html_una_vez()              Un intento de descarga.
│             descargar_html_tabla_con_reintentos()  Envoltorio con política de reintentos.
│
├── Celda 5  — Parsers
│             _clean_num()                          Normaliza strings numéricos.
│             _dedupe_headers()                     Renombra cabeceras duplicadas.
│             extraer_datos_de_html()               Parser para tablas con cabecera simple.
│             extraer_datos_de_html_multinivel()    Parser para tablas con doble cabecera y colspan.
│
├── Celda 6  — Orquestador
│             _year_for()        Asigna el año correcto según el bloque.
│             _sanitize()        Normaliza texto para nombres de archivo.
│             run_extraction()   Bucle principal: descarga → parseo → CSV → log.
│
├── Celda 7  — Descarga
│             on_download_clicked()   Comprime la carpeta de resultados y descarga el ZIP.
│
└── Celda 8  — Limpieza
              on_cleanup_clicked()    Elimina carpeta de resultados y ZIP del entorno.
```

---

## Dependencias instaladas automáticamente

| Librería | Versión mínima | Uso |
|----------|---------------|-----|
| `playwright` | 1.40+ | Automatización del navegador Chromium. |
| `pandas` | 1.5+ | Estructuración y exportación de datos. |
| `beautifulsoup4` | 4.12+ | Extracción de datos de las tablas HTML. |
| `lxml` | 4.9+ | Parser HTML de alto rendimiento para BeautifulSoup. |
| `ipywidgets` | 7.0+ | Botones y menús desplegables interactivos en Colab. |
| `nest_asyncio` | 1.5+ | Compatibilidad de Playwright con el bucle de eventos de Jupyter. |
| `requests` | 2.28+ | Descarga de HTML para extracción de opciones de menú. |

---

## Notas y limitaciones conocidas

- **Dependencia del portal del Catastro** — El scraper depende de la estructura HTML del portal. Si la Dirección General del Catastro modifica sus páginas, puede ser necesario actualizar los selectores CSS o los IDs de los `<select>`.
- **Disponibilidad de datos por bloque** — No todas las tablas tienen datos para todas las provincias y años. El notebook lo detecta automáticamente (mensaje `⚠️ Tabla no válida o sin datos`) y continúa con la siguiente tabla sin interrumpir el proceso.
- **Sesión de Colab** — Google Colab puede desconectar la sesión tras un período de inactividad. Si la extracción de las 29 tablas supera ese límite, la reanudación automática (celda 6) permite continuar desde el último CSV guardado sin repetir trabajo ya completado.
- **Separador CSV** — Los archivos se generan con separador `;` y codificación `utf-8-sig` para compatibilidad directa con Excel en español.

---

## Autor

**José Carlos Rico** — [CITYLAB360, S.C.A.](https://citylab360.es)

¿Errores o sugerencias? Abre un [issue](../../issues) en este repositorio.
