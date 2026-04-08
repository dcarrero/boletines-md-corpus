# Reglas de Normalización — boletines-md

Documento canónico de cómo se transforman los boletines oficiales españoles (XML, HTML o PDF según fuente) en ficheros Markdown con frontmatter YAML estructurado.

> Esta especificación es **propia del proyecto**. Toma inspiración del [Legalize Format Spec](https://github.com/legalize-dev/legalize/blob/main/SPEC.md) pero no está sujeta a él.

---

## 1. Modelo de salida

Cada documento de cualquier boletín se materializa como **un único fichero `.md`** con dos partes:

```
---
{frontmatter YAML estructurado}
---

# Título oficial del documento

{cuerpo Markdown normalizado}
```

Una excepción: cada día se genera además un fichero índice `_sumario.md` con la lista de documentos publicados ese día, agrupados por sección y emisor.

---

## 2. Frontmatter — contrato común

Todos los ficheros `.md` del corpus comparten el siguiente esquema. Los campos están agrupados temáticamente y todos son obligatorios; si el dato no existe en la fuente, el valor es la cadena vacía `""` o `0` para enteros — **no se omiten campos**.

### 2.1 Identidad

| Campo | Tipo | Descripción |
|---|---|---|
| `identificador` | string | ID oficial único en la fuente. Ej: `BOE-A-2025-1` |
| `titulo` | string | Título oficial completo de la disposición |

### 2.2 Procedencia

| Campo | Tipo | Valores |
|---|---|---|
| `pais` | string (ISO 3166-1 α2) | `es` |
| `fuente` | string | Código corto del boletín: `BOE`, `DOGC`, `BOJA`, `BOPV`, `BOP-SE`, ... |
| `fuente_tipo` | string | `estatal`, `autonomico`, `provincial`, `local` |
| `jurisdiccion` | string | Slug del territorio: `estatal`, `cataluna`, `comunidad-valenciana`, ... |
| `emisor` | string | Nombre del organismo emisor (ministerio, conselleria, parlamento, ayuntamiento, juzgado...) |
| `departamento` | string | Subdivisión opcional dentro del emisor |

### 2.3 Tipo de norma

| Campo | Tipo | Descripción |
|---|---|---|
| `rango` | string (lowercase) | Tipo legal: `ley`, `real decreto`, `orden`, `resolución`, `acuerdo`, `decreto`, `circular`, ... — libre, dependiente de cada ordenamiento |
| `numero_oficial` | string | Numeración oficial. Ej: `6/2024`, `RD 123/2025` |

### 2.4 Fechas

Todas en formato **ISO 8601** (`YYYY-MM-DD`).

| Campo | Descripción |
|---|---|
| `fecha_disposicion` | Fecha en que se firmó/aprobó la norma |
| `fecha_publicacion` | Fecha en que apareció en el boletín |
| `fecha_vigencia` | Fecha de entrada en vigor |
| `ultima_actualizacion` | Última reforma reflejada en este texto. Por defecto = `fecha_publicacion` |

### 2.5 Estado legal

| Campo | Valores |
|---|---|
| `estado` | `vigente`, `derogada`, `parcialmente_derogada`, `anulada` |

### 2.6 Ubicación en el boletín

| Campo | Tipo | Descripción |
|---|---|---|
| `seccion` | string | Sección del boletín (legible). Ej: `I. Disposiciones generales` |
| `diario_numero` | string | Número del boletín del día |
| `pagina_inicial` | int | Página de inicio en el PDF oficial |
| `pagina_final` | int | Página final |

### 2.7 Enlaces a originales

| Campo | Descripción |
|---|---|
| `url_html` | URL de la versión HTML oficial |
| `url_pdf` | URL del PDF oficial |
| `url_xml` | URL del XML oficial (si existe) |
| `url_eli` | URL ELI (European Legislation Identifier) si la fuente la publica |

### 2.8 Trazabilidad

| Campo | Descripción |
|---|---|
| `fetched_at` | Timestamp ISO 8601 con `Z` (UTC) en que se descargó |
| `parser_version` | Versión del normalizador. Permite reprocesar ficheros antiguos al mejorar el normalizador |

---

## 3. Normalización del cuerpo Markdown

### 3.1 Reglas comunes a todas las fuentes

1. **H1 único:** el primer encabezado del fichero es siempre el `# título oficial`. No hay otros H1.
2. **Codificación:** UTF-8, sin entidades HTML residuales (`&amp;` → `&`, `&lt;` → `<`, etc.).
3. **Espacios:** se colapsan espacios y tabs múltiples a uno solo. Se eliminan trailing spaces.
4. **Saltos de línea en blanco:** máximo dos consecutivos (un párrafo vacío).
5. **Inline:**

   | Origen | Markdown |
   |---|---|
   | `<em>`, `<i>` | `*texto*` |
   | `<strong>`, `<b>` | `**texto**` |
   | `<sup>` | `^texto^` |
   | `<sub>` | `~texto~` |
   | `<a href="url">texto</a>` | `[texto](url)` |
   | `<br/>` | doble espacio + newline |
   | `<img>` (base64) | `[imagen]` (en esta fase; futuras: extraer a `img/`) |

6. **Listas:**
   - `<ul>/<li>` → `- item`
   - `<ol>/<li>` → `1. item`
7. **Citas:** `<blockquote>` → líneas con prefijo `> `.
8. **Tablas:** `<table>` → tabla Markdown estándar. Si la tabla tiene celdas combinadas u otra complejidad que el formato Markdown no soporta, se incluye como bloque HTML literal.
9. **Notas al pie:** `[^N]` con definición al final del fichero.
10. **Enlaces internos al propio boletín:** se preservan como URL absoluta. En fases futuras se podrán reescribir como enlaces relativos al fichero del repo cuando el destino esté disponible.
11. **Versiones consolidadas:** cuando un documento del corpus consolidado incluye múltiples `<version>`, se procesan todas las versiones en orden — la última es la vigente.

### 3.2 Jerarquía de encabezados

La jerarquía estructural se proyecta sobre niveles Markdown del siguiente modo:

| Nivel MD | Elementos |
|---|---|
| H1 | Título oficial del documento (único) |
| H2 | Libro, parte, anexo, disposición (adicional, transitoria, derogatoria, final), preámbulo, **artículo** |
| H3 | Título, capítulo |
| H4 | Sección |

**Decisión de diseño:** los **artículos** son siempre H2 aunque estén anidados bajo título/capítulo/sección. Razón: el artículo es la unidad citable de cualquier norma jurídica, y mantenerlos al mismo nivel permite generar índices navegables planos y simplifica las referencias cruzadas.

### 3.3 Reglas específicas por fuente

Cada fuente se traduce a la jerarquía común aplicando un mapeo desde su marcado nativo. El mapeo de cada fuente vive en `sources/{fuente}/normalizer.py`.

#### 3.3.1 BOE (Boletín Oficial del Estado)

El XML del BOE marca cada párrafo con un atributo `class` que indica su rol semántico. Los encabezados estructurales se publican como **dos `<p>` consecutivos**: uno con el número (`*_num`) y otro con el título (`*_tit`). El normalizador los **fusiona** en un único heading.

| Marcado XML | Markdown resultante |
|---|---|
| `<p class="titulo_num">TÍTULO I</p>` + `<p class="titulo_tit">Régimen general</p>` | `### TÍTULO I. Régimen general` |
| `<p class="capitulo_num">CAPÍTULO I</p>` + `<p class="capitulo_tit">Disposiciones generales</p>` | `### CAPÍTULO I. Disposiciones generales` |
| `<p class="anexo_num">ANEXO I</p>` + `<p class="anexo_tit">Tablas</p>` | `## ANEXO I. Tablas` |
| `<p class="seccion">Sección 1.ª Impulso de proyectos</p>` | `#### Sección 1.ª Impulso de proyectos` |
| `<p class="articulo">Artículo 1. Objeto.</p>` | `## Artículo 1. Objeto.` |
| `<p class="preambulo">...</p>` | `## ...` |
| `<p class="parrafo">...</p>` | Texto normal |
| `<p class="parrafo_2">...</p>` | Texto normal (sangría se pierde) |
| `<p class="centro_cursiva">...</p>` | Texto normal en cursiva |
| `<p class="centro_redonda">...</p>` | Texto normal |
| `<p class="publicado">Madrid, ...</p>` | Texto normal (firma final) |
| `<table>...</table>` | Tabla Markdown |

**Selección del cuerpo:** el cuerpo principal está en `<documento>/<texto>` (hijo directo del root). Hay otros nodos `<texto>` profundos dentro de `<analisis>/<referencias>` que son referencias a normas derogadas y deben **ignorarse**. El normalizador toma siempre el `<texto>` que es hijo directo del root.

---

## 4. Estructura de directorios

```
data/
├── boe/                                  ← prefijo por fuente
│   └── 2025/                             ← año
│       └── 01/                           ← mes (con cero)
│           └── 01/                       ← día (con cero)
│               ├── _sumario.md           ← índice del día
│               └── BOE-A-2025-1.md       ← documento individual
└── (futuras fuentes: dogc/, boja/, ...)
```

**Convenciones:**
- Año/mes/día siempre con ceros (`2025/01/01`, no `2025/1/1`).
- Nombre de fichero = `{identificador}.md` literal de la fuente.
- Sumario con prefijo `_` para que aparezca primero en orden alfabético.
- BOE no publica los domingos: el directorio del domingo simplemente no existe.

---

## 5. Sumario diario (`_sumario.md`)

```yaml
---
fuente: "BOE"
fecha: "2025-01-01"
diario_numero: "1"
total_documentos: 247
---

# BOE núm. 1 — 2025-01-01

## I. Disposiciones generales

### MINISTERIO DE HACIENDA

- [Real Decreto-ley 9/2024, de 23 de diciembre, ...](BOE-A-2025-1.md)
- [Real Decreto 1234/2024, de 23 de diciembre, ...](BOE-A-2025-2.md)

### MINISTERIO DE JUSTICIA

- [Orden JUS/1500/2024, ...](BOE-A-2025-3.md)
```

Los enlaces son **relativos al propio directorio del día**. Esto permite navegar el corpus como un sitio estático sin reescribir nada.

---

## 6. Versionado del normalizador

El campo `parser_version` del frontmatter (`boletines-md/0.1`) se incrementa cuando hay un cambio incompatible en las reglas de normalización. Esto permite, en el futuro, **reprocesar** todos los ficheros antiguos al actualizar el normalizador y detectar cuáles aún están en una versión obsoleta.

Reglas de cambio:
- **Patch** (`0.1` → `0.1.1`): bugfix, no cambia salida en el caso correcto.
- **Minor** (`0.1` → `0.2`): añade campos al frontmatter o nuevos mapeos sin romper los existentes.
- **Major** (`0.1` → `1.0`): cambio incompatible — requiere reprocesar el corpus.
