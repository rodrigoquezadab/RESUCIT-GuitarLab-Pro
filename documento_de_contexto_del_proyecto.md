# 🎸 Resucitó GuitarLab Pro — Documento de Contexto y Arquitectura

## 1. Visión y Propósito del Proyecto
**Resucitó GuitarLab Pro** es una plataforma web interactiva y pedagógica inspirada en el catálogo digital de cantos del Camino Neocatecumenal ([app.resucito.es](https://app.resucito.es/home)). 

Su objetivo principal es evolucionar el formato de cantoral tradicional hacia un **laboratorio de aprendizaje para guitarristas y salmistas**, unificando en una misma experiencia:
1. **El catálogo completo de cantos** con su clasificación litúrgica y catequética original.
2. **Formación litúrgica y musical integrada:** cada filtro explica al usuario el significado de la etapa o momento litúrgico y cómo debe interpretarse musicalmente en la guitarra.
3. **Vista de ejecución en pantalla dividida (Split-View):** muestra la letra con acordes alineados a la izquierda y, de forma sincronizada a la derecha, los diagramas de mástil (SVG) y las tablaturas (TAB) exclusivos de los acordes que utiliza ese canto.
4. **Academia de Acordes, Rasgueos y Arpegios:** diccionarios visuales e interactivos con notación europea (`Do, Re, Mi, Fa, Sol, La, Si`), cifrado americano y lectura en tablatura de 6 cuerdas.

---

## 2. Referencias del Dominio y Reglas de Negocio
* **Sitio de referencia base:** `https://app.resucito.es/home` (Cantoral *Resucitó*).
* **Sistema de Notación Musical:**
  * Se utiliza como estándar principal la **notación latina/europea** (`Do, Re, Mi, Fa, Sol, La, Si`), respetando la tradición de los cantorales impresos del Camino.
  * Cada tarjeta de acorde incluye su equivalencia en **cifrado internacional/americano** (`C, D, E, F, G, A, B`) para facilitar el estudio cruzado.
  * Se contemplan digitaciones características del estilo mediterráneo/flamenco propias de estos cantos (por ejemplo, el uso del acorde de **Fa Maj7 abierto** `x-3-3-2-1-0` en las cadencias andaluzas `La m - Sol - Fa - Mi`, y el uso frecuente de la cejilla o *capo*).

---

## 3. Taxonomía de Filtros y Contexto Pedagógico

### 3.1. Etapas del Camino (Código de Colores de Cartillas)
| Identificador (`stage`) | Etapa | Color | Significado Catequético y Enfoque en Guitarra |
| :--- | :--- | :--- | :--- |
| `PRECATECUMENADO` | Precatecumenado | Blanco | Etapa inicial del Kerigma. Acordes abiertos fundamentales (`La m, Mi, Fa, Sol, Do`) y ritmos base marcados. Ideal para principiantes. |
| `CATECUMENADO` | Catecumenado | Azul | Etapa del combate espiritual y las entregas (*Shemá*, Salterio, Padre Nuestro). Introduce cejillas frecuentes, cambios de compás entre recitativo y asamblea, y arpegios más elaborados. |
| `ELECCION` | Elección | Verde | Última etapa de iniciación centrada en la unión mística (*Cantar de los Cantares*, *Sermón de la Montaña*). Exige dominio de arpegios delicados (como 6/8), dinámicas suaves y acordes con tensiones. |
| `LITURGIA` | Liturgia | Amarillo | Cantos fijos de la estructura sacramental (Santo, Plegarias Eucarísticas, Aclamaciones, Fracción del Pan, Bendición del Agua, Pregón Pascual). |

### 3.2. Momentos Litúrgicos (`moments`)
* `ENTRADA`: Cantos festivos de convocatoria; requieren ataque firme y tempo constante.
* `PAZ`: Cantos fraternales breves (ej. *Salmo 133*), a menudo en ritmos de danza hebrea (2/4).
* `FRACCION`: Cantos solemnes y contemplativos durante la fracción del Pan consagrado.
* `COMUNION`: Cantos extensos de estructura responsorial para el Cuerpo y la Sangre.
* `FINAL`: Cantos de envío misionero, alabanza o antífonas marianas al cierre de la celebración.
* `LAUDES_VISPERAS`: Salmos, himnos y cánticos evangélicos (*Benedictus, Magnificat, Nunc Dimittis*) de la Liturgia de las Horas.
* `ORDINARIO_MISA`: Respuestas dialogadas, Santo, Credo, Cordero de Dios y Padre Nuestro.

### 3.3. Tiempos Litúrgicos (`seasons`)
* `ADVIENTO`: Cantos mesiánicos (profeta Isaías) en tonalidades menores expectantes.
* `NAVIDAD`: Cantos luminosos de la encarnación.
* `CUARESMA`: Cantos penitenciales, de desierto y los *Cantos del Siervo de Yahvé* e *Improperios*.
* `PASCUA`: Cantos de victoria sobre la muerte, Vigilia Pascual y Pentecostés con rasgueo vigoroso.

---

## 4. Arquitectura Técnica y Stack Tecnológico

El prototipo actual sigue una arquitectura **Single-File Web Application (SPA ligera)** sin dependencias de compilación externas, lo que permite portabilidad inmediata o migración directa a frameworks modernos (React, Vue, Astro o Next.js).

* **Estructura y Maquetación:** HTML5 semántico.
* **Estilos y Diseño UI:** **Tailwind CSS** (vía CDN o CLI), utilizando una paleta inspirada en el cantoral físico (`red-800` institucional, fondos `stone-50` de alta legibilidad, y etiquetas cromáticas por etapa).
* **Tipografía:**
  * `Inter`: para interfaz de usuario, filtros y explicaciones pedagógicas.
  * `JetBrains Mono` (Monoespaciada): crítica para garantizar la alineación milimétrica entre sílabas y acordes, así como para la renderización de tablaturas ASCII.
* **Lógica y Motores Internos (Vanilla JavaScript ES6+):**
  1. **Motor Parser ChordPro (`parseChordProLine`):** procesa cadenas con acordes incrustados entre corchetes (ej. `"[La m]RESUCI[Sol]TÓ"`) y genera dinámicamente dos líneas paralelas (`chord-line` y `lyric-line`) perfectamente espaciadas.
  2. **Extractor Automático de Acordes (`extractUniqueChords`):** analiza mediante expresiones regulares (`/\[([^\]]+)\]/g`) el arreglo `chordPro` del canto para detectar qué acordes únicos utiliza, evitando declararlos manualmente.
  3. **Motor Transpositor Cromático (`transposeChordName`):** rota cualquier acorde sobre la escala cromática europea de 12 semitonos (`["Do", "Do#", "Re", "Re#", "Mi", "Fa", "Fa#", "Sol", "Sol#", "La", "Si b", "Si"]`), conservando sus sufijos (`m`, `7`, `maj7`) y actualizando en vivo tanto el canto como las tablaturas laterales.
  4. **Generador Gráfico de Mástil (`generateChordCardHTML`):** dibuja en tiempo real un gráfico vectorial **SVG** de 6 cuerdas por 4 trastes para cada acorde, incluyendo cejuelas, indicador de traste base (`fr`), barras de cejilla (*barre*), numeración de dedos (`1, 2, 3, 4`), cuerdas al aire (`O`) y cuerdas muteadas (`✕`), acompañado de su tablatura numérica de 6 líneas.

---

## 5. Estructuras de Datos (Schemas)

### 5.1. Esquema de un Canto (`SONGS_DATABASE`)
```json
{
  "id": "resucito",
  "title": "Resucitó",
  "citation": "1 Corintios 15",
  "stage": "PRECATECUMENADO",
  "moments": ["ENTRADA", "FINAL", "COMUNION"],
  "seasons": ["PASCUA"],
  "capo": "Sin cejilla",
  "rhythm": "Rasgueo Pascual (Cadencia andaluza: La m - Sol - Fa - Mi)",
  "introTab": "1ª|-----0---0---0---|...\n2ª|-----1---1---1---|...",
  "chordPro": [
    "[La m]RESUCITÓ, [Sol]RESUCITÓ, [Fa]RESUCITÓ, [Mi]ALELUYA.",
    "[La m]La muerte, ¿dónde [Sol]está la muerte?"
  ]
}
```

### 5.2. Esquema de un Acorde (`CHORD_DATABASE`)
* El arreglo `frets` representa las 6 cuerdas desde la más grave a la más aguda: `[6ª Mi, 5ª La, 4ª Re, 3ª Sol, 2ª Si, 1ª mi]`.
* Valores especiales en `frets`: `-1` = cuerda muteada (`x`), `0` = cuerda al aire (`0`), `>=1` = número de traste pisado.

```json
{
  "La m": {
    "name": "La m",
    "intl": "Am",
    "frets": [-1, 0, 2, 2, 1, 0],
    "fingers": ["", "", "2", "3", "1", ""],
    "barre": null,
    "tip": "Tónica principal del cantoral. Toca desde la 5ª cuerda."
  }
}
```

### 5.3. Esquema de Patrones Rítmicos (`RHYTHMS_DATABASE`)
```json
{
  "name": "1. Rasgueo Pascual / Rumba Litúrgica (4/4)",
  "usage": "Resucitó, Hacia ti morada santa, Cantad a Yahvé",
  "arrows": "↓   ↓ ↑   ↑ ↓ ↑",
  "counts": "1   2 &   & 4 &",
  "description": "Explicación técnica del movimiento de mano derecha.",
  "tab": "Representación ASCII en 6 cuerdas del patrón."
}
```

---

## 6. Módulos y Vistas de la Interfaz
1. **`#view-catalog` (Catálogo, Buscador Multicriterio y Selector de Vistas):**
   * Catálogo completo oficial integrado: **215 cantos** del Camino Neocatecumenal.
   * **Diseño Ultra-Compacto con Acordeón:**
     * Barra unificada de altura mínima (~40px) que integra buscador en vivo, selector de modo de vista (Lista / Tarjetas), chip rápido «Solo Salmos», selector de orden y botón de acordeón para filtros avanzados.
     * **Cajón Acordeón Desplegable:** Almacena los selectores de Etapa, Momento Litúrgico, Tiempo Litúrgico y la guía pedagógica de aprendizaje, desplegándose bajo demanda y dejando el 100% del área vertical útil para la exploración inmediata de los cantos.
   * **Selector de Modo de Visualización:**
     * **☰ Modo Lista (Fila Única):** Vista compacta y optimizada donde cada canto se visualiza en una sola fila horizontal (no tarjeta), con píldora cromática de etapa, insignia de salmo (`📖 Salmo X`), título, cita bíblica, cejilla, acordes base, disponibilidad multimedia (🎧/📄) y botón directo de apertura.
     * **🗂️ Modo Tarjetas (Grid Tradicional):** Cuadrícula de 3 columnas para visualización amplia.
     * Persistencia automática de la preferencia del usuario en `localStorage`.
   * **Motor de Ordenación Flexible:**
     * **📖 Por número de Salmo (1 → 150):** Ordenación numérica precisa que agrupa y lista todos los salmos bíblicos del cantoral en orden ascendente (Salmo 1, Salmo 2, Salmo 6... hasta el Salmo 150), seguidos de los cantos e himnos bíblicos en orden alfabético.
     * **🔤 Alfabético (A → Z) y (Z → A):** Ordenación alfabética insensible a tildes y mayúsculas.
     * **📋 Orden Original:** Secuencia original del archivo del cantoral.
   * **Filtro Rápido «Solo Salmos»:** Botón interactivo que aísla de inmediato los 62 salmos del cantoral con un solo clic.
   * Contador dinámico en vivo de cantos filtrados y total del cantoral.
   * Búsqueda instantánea en vivo por título, cita bíblica, número de salmo (ej. «Salmo 23», «23»), letra completa o contenido.
2. **`#view-song` (Vista de Canto + Split-View de Tablaturas + Herramientas de Salmista):**
   * **Barra de Herramientas Ultra-Compacta (Sticky Slim) con Acordeón:**
     * Barra fija de perfil bajo (~38px) con controles directos: botón de retorno, título del canto, transpositor cromático en vivo (`-1` / `+1`), botón de Auto-Scroll manos libres, botón de acceso al Modo Práctica y botón de acordeón `⚙️ ▼`.
     * **Cajón Acordeón Desplegable de Ajustes Secundarios:** Despliega selector de velocidad de scroll (1x, 2x, 3x), botón de reset de tono, formato de acordes (**🎸 Mástil**, **🎼 TAB** o **📑 Ambos**) y píldoras de acordes del canto.
   * Panel izquierdo (7 columnas):
     * Metadatos litúrgicos (Etapa, Cita bíblica, Cejilla/Capo, Ritmo de acompañamiento).
     * **Reproductor de Audio Oficial MP3:** escucha de la grabación original del canto.
     * **Reseña y Contexto Litúrgico/Catequético:** explicación de la solemnidad o momento.
     * **Visualizador de Cartilla Original Escaneada:** acceso a la partitura y dibujos originales de Kiko Argüello en alta definición.
     * Bloque de tablatura de punteo/arpegio de introducción.
     * Letra interactiva con acordes generada por el motor ChordPro: cada verso es clickeable para saltar de inmediato al momento correspondiente de la práctica.
   * **Estudio Interactivo de Práctica y Aprendizaje Exclusivo (`#practice-studio-box`):**
     * **Cabecera Ultra-Compacta con Acordeón de Atajos:** Muestra el título del canto, cejilla, ritmo, transposición en vivo, botón de salida y acordeón colapsable para atajos de teclado (`Espacio: Play`, `← →: Pasos`), maximizando el espacio vertical para el teleprompter y los diagramas de acordes.
      * **Reproducción de la melodía con control de velocidad lenta (Slow-Down):** selector de velocidad (`0.5x`, `0.75x`, `0.85x`, `1.0x`) con preservación de tono (`preservePitch`) para dar tiempo de sobra a colocar dedos y rasguear.
      * **Teleprompter de letra en tiempo real:** muestra la estrofa actual, iluminando la línea y sílaba interpretada.
      * **Navegador interactivo de versos:** Lista integrada dentro del estudio donde el guitarrista puede ver todas las estrofas y hacer clic en cualquiera de ellas para saltar de inmediato a esa parte del canto.
      * **Tablatura y Mástil del momento exacto:** renderiza el acorde activo con números de dedos, cejillas y cuerdas.
      * **Anticipación de dedos (Próximo Acorde):** tarjeta adyacente que anuncia el siguiente acorde y posición de mano antes del cambio de compás.
      * **Modos de acompañamiento:** sincronización con el audio oficial MP3, metrónomo interactivo Web Audio o avance manual paso a paso (flechas y barra espaciadora).
   * Panel derecho fijo/sticky (5 columnas - Espacio 100% optimizado):
     * Dedicado íntegramente a las tarjetas de acordes, eliminando encabezados redundantes.
     * Cuadrícula de 2 columnas de visualización inmediata de todos los acordes de la canción sin necesidad de desplazamiento vertical.
     * Tarjetas compactas con mástil SVG, números de dedos, resumen de trastes (`[x 0 2 2 1 0]`), tablatura de 6 cuerdas y consejo técnico, con actualización dinámica según la transposición activa.
     * Persistencia de preferencia en `localStorage`.
3. **`#view-chords` (Academia / Diccionario Completo de Acordes):**
   * Cuadrícula adaptable (de 4 a 6 columnas en modo compacto) con los 36 acordes naturales y alterados documentados gráficamente, con selector de formato (Mástil, TAB o Ambos).
4. **`#view-rhythms` (Escuela de Rasgueos y Arpegios):**
   * Fichas prácticas de mano derecha con dirección de golpes (`↓ ↑`), digitación de arpegio (`P, i, m, a`), conteo rítmico y tablatura de ejemplo.

---

## 7. Archivos de Datos y Exportación
* `songs.js`: Base de datos completa en JavaScript con los 215 cantos oficiales para carga directa sin restricciones CORS.
* `songs.json`: Archivo JSON estructurado descargable desde el botón **📥 Exportar JSON** de la barra de navegación.

---

## 8. Hoja de Ruta (Próximas Evoluciones Planeadas)
- [x] **Catálogo Completo Integrado:** 215 cantos oficiales del Camino Neocatecumenal integrados con ChordPro, audio y partituras.
- [x] **Reproductor de Audio Oficial:** integración del reproductor MP3 para escuchar cada canto mientras se ensaya.
- [x] **Visualizador de Cartillas Originales:** escaneos oficiales de alta resolución desplegables con un clic.
- [x] **Auto-Scroll Manos Libres:** desplazamiento vertical automático configurable para tocar sin soltar la guitarra.
- [x] **Exportador JSON:** descarga con un solo clic del catálogo completo `songs.json`.
- [x] **Sistema y Estudio de Práctica Interactiva:** modo ensayo con teleprompter de letra, velocidad lenta (0.5x, 0.75x, 1x), mástil dinámico en vivo y anticipación del siguiente acorde.
- [ ] **Síntesis de Audio Web (Web Audio API):** botón en cada tarjeta de acorde y tablatura para escuchar el sonido real cuerda por cuerda en el navegador.
- [ ] **Persistencia Local (`localStorage`) y Favoritos:** guardado de listas de cantos para preparar celebraciones (Palabra o Eucaristía) con el tono/transposición preferido de cada salmista.