# Análisis rápido · mahabrain-obsidian-vault · 24-nov-2026

Diagnóstico, sin cambios al código ni al vault. No se ejecutó el scraper ni se recorrió el vault: se muestreó con la API de GitHub (listado de raíz y de `tao/`, y las páginas `cnote.md` y `tos.md`).

## 1. Qué es

- Copia de `jonnyg23/mahabrain` (código MIT): scrape de sacred-texts.com (marzo 2023, ISTA 9.0) guardado como `.md`. Unos 2,2 GB y más de 120 000 archivos, según el README.
- El vault tiene 77 carpetas por sección (`bib`, `bud`, `hin`, `sbe`, `tao`, `tarot`, `grim`...) y 29 `.md` sueltos en la raíz.
- Código mínimo: un spider Scrapy (`books`) y `format_vault_files.py`. No hay tests ni CI.
- En este checkout el vault **no está** (sparse checkout: solo `docs/` y `sacred_texts_scraper/`; el `.git` pesa ~5 MB). Está en GitHub y se puede muestrear por API.
- Memoria (Graphiti): solo hay la descripción del repo del 3-nov. No hay decisiones previas sobre el vault. No hay registro de `impulse-publishing-studio`; no pude verificar esa línea.
- Qdrant: sin resultados sobre este repo.

## 2. Nemotron (primer avance)

- No disponible: `opencode_status` dice *online*, pero `opencode_run` falla dos veces con `Unrecognized flag: --dir in command opencode run`. Es un fallo del MCP, no del mensaje. Se reintentó con el modelo explícito y falló igual.
- Por tanto no hay nada que contrastar. Conviene arreglar el MCP antes de usarlo en otra tarea.

## 3. Lo que verifiqué yo

| Hallazgo | Evidencia |
|---|---|
| Cada `.md` es el `<BODY>` HTML crudo, no markdown | `cnote.md` y `tos.md`: tablas de navegación, menú de ~100 enlaces, scripts de anuncios de Google, widgets de FeedBlitz y AddThis |
| Mucho ruido por página (estimación a partir de 3 muestras) | `abuse.md` pesa 12,7 KB y su contenido útil es ~1 KB. Si la proporción se repite en 120 mil páginas, más de la mitad de los 2,2 GB es plantilla |
| El texto útil está en `<span class="ista-content">` | Se ve en `cnote.md` y `tos.md`; permite extraerlo con un selector |
| Los libros grandes son un solo archivo grande | `tao/taote.md` pesa 66 KB y `stbib.md` 254 KB |
| El código es frágil | El spider no tiene throttling ni `DOWNLOAD_DELAY`. `apply_func_to_files` abre en `r+` y reescribe en sitio. `format_vault_files.py` usa ruta `../sacred_obsidian_vault/` |
| El scrape no sirve tal cual para RAG | Hace falta limpiar el HTML antes de trocear |

## 4. Valor para Impulse Labs (Ethea / devocionales / RAG)

**Sirve, pero como materia prima y no como producto.**

- **RAG interno**: útil como corpus de consulta (tradiciones, textos clásicos, simbología) para Ethea y para ideación editorial. Requiere limpiar el HTML y conservar la ruta de origen y el estado de derechos de cada página.
- **Devocionales publicables (KDP)**: el valor está en los textos de dominio público concretos, no en el vault entero. Esos textos ya existen en fuentes más limpias (Project Gutenberg, Wikisource, CCEL, archive.org).
- **Valor diferencial**: el trabajo editorial propio (selección, traducción al español, comentario, estructura devocional). El vault no lo aporta.

### Derechos (criterio: buscar salidas, dentro de la ley)

Lo que dicen las páginas del propio vault (`tos.md`, `cnote.md`; son de 2023 y el sitio hoy tiene otro diseño, así que hay que confirmar los términos vigentes):

1. **Dominio público (DP) en EE. UU.**: la mayoría de los textos, con criterio conservador (publicados hasta 1922, o 1923–1963 sin renovación). Los DP pueden usarse para cualquier fin, incluido comercial, sin pedir permiso. ISTA pide atribución por cortesía.
2. **Capa que sí protege ISTA**: derecho de colección (selección y arreglo), páginas `index.htm`, introducciones con firma, gráficos, JavaScript (§7). Copiar los archivos de nivel superior a otro sitio está prohibido sin permiso.
3. **Textos "producidos por ISTA"** (listados en `stbib.htm`): uso comercial **en su totalidad** prohibido sin licencia escrita y tarifa, que puede condonarse (§5–6).
4. **Textos modernos con copyright** (p. ej. Internet Book of Shadows): solo no comercial (§8). Los marca el ícono `cnote`.
5. **Robots** (§10): solo para indexar o para bajar **un texto por día**. Un scrape masivo como el de 2023 queda fuera de lo permitido. El riesgo directo recae en quien raspó (upstream). Para nosotros el riesgo es **seguir redistribuyendo** esa copia y **usarla comercialmente** como si estuviera limpia.
6. **México** (a verificar con abogado de PI): el plazo es vida del autor + 100 años, más largo que el criterio de EE. UU. Un texto DP en EE. UU. puede no serlo en México (autores fallecidos después de 1926: Waite, Crowley, etc.). Además, las traducciones tienen derecho propio, y los derechos morales (atribución e integridad) no caducan para obras DP.
7. **KDP**: Amazon exige valor diferenciado en obras de dominio público. No basta con reempaquetar.

**Salidas viables (riesgo calculado):**

- **Whitelist de obras**: elegir, obra por obra, textos con autor y traductor fallecidos hace más de 100 años (cubre EE. UU. y México) y sin ícono `cnote`. Documentar la verificación en una tabla.
- **Fuente primaria**: tomar el texto de Gutenberg, Wikisource o CCEL (el §12 de ISTA los exime de restricciones) y usar sacred-texts solo como índice para localizar obras. Así se evita el derecho de colección y de transcripción de ISTA.
- **Uso interno vs. público**: RAG privado con atribución = riesgo bajo. Citas textuales largas en un producto de pago = solo sobre la whitelist.
- **Si se quiere el corpus ISTA tal cual para uso comercial**: pedir licencia escrita (el propio §6 prevé que la tarifa se puede condonar), o comprar la edición oficial (USB ~127 USD en el sitio actual). Ambas son salidas baratas frente a un reclamo.
- **No** volver a ejecutar el scraper. **No** publicar el vault en un repo público ni exponerlo como descarga.

## 5. Recomendación: mover a almacenamiento y dejar el repo ligero

**Mover (no mantener en git, no archivar a ciegas).**

1. Empaquetar el vault actual (`tar.zst`) con un manifiesto de hashes y subirlo a almacenamiento privado. Es un respaldo, no una fuente de verdad.
2. Dejar en el repo solo código, `docs/` y el manifiesto. Esto resuelve el peso del `.git` (~600 MB según CLAUDE.md).
3. Construir un **corpus derivado y curado** (solo `ista-content`, markdown limpio, una fila por obra con estado de derechos). Ese es el activo que usarán Ethea y el RAG.
4. Si la línea devocional no avanza en ~90 días, archivar el respaldo en frío y cerrar el repo.

Riesgos de la recomendación: el tamaño del vault y el estado incierto de derechos de algunas obras. No se midió el porcentaje real de páginas con ícono `cnote`.

## 6. Conectores (1–2 usos cada uno, sin activar)

- **Qdrant (RAG)**:
  1. Indexar una muestra de ~300 páginas ya limpias (p. ej. `tao`, `hin`, `sbe`) en una colección aparte, con metadata `ruta`, `seccion` y `estado_derechos`, y medir si recupera pasajes útiles para prompts devocionales.
  2. Usar la misma colección como banco de pruebas para comparar con el texto de Gutenberg de las mismas obras.
- **Almacenamiento**:
  1. **AWS S3**: respaldo del `tar.zst` en un bucket privado con versionado y paso a almacenamiento frío; es la opción adecuada para el archivo grande (el conector entrega URL prefirmada).
  2. **Google Drive o Dropbox**: compartir con el equipo editorial solo el corpus curado y la tabla de derechos (archivos pequeños). Dropbox no permite subir binarios por el conector, así que el respaldo grande no iría por ahí.

## 7. Siguientes pasos (a decidir por Giovanni)

1. Aprobar mover el vault a S3 y limpiar el repo.
2. Elegir 10–20 obras piloto para la whitelist y verificar su estado de derechos.
3. Decidir si se pide licencia a ISTA o se va por fuentes primarias.
4. Arreglar el MCP de OpenCode antes de reintentar el análisis con Nemotron.
