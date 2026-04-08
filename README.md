# boletines-md-corpus

El **corpus** de los boletines oficiales españoles, descargados, normalizados y publicados como Markdown plano con frontmatter YAML estructurado.

> **Estado:** fase inicial. Primera fuente publicada: **BOE** (Boletín Oficial del Estado). Más fuentes (DOGC, BOJA, BOPV, BOP provinciales) en fases posteriores.

---

## Qué hay aquí

Cada disposición publicada en un boletín oficial es un único fichero `.md` con metadatos completos en el frontmatter y el cuerpo normalizado a Markdown plano.

```
boletines-md-corpus/
├── README.md
├── SPEC.md                    ← especificación completa del formato
└── boe/                       ← prefijo por fuente
    └── 2025/
        └── 01/
            └── 01/
                ├── _sumario.md       ← índice del día
                ├── BOE-A-2025-1.md
                ├── BOE-A-2025-2.md
                └── ...
```

Cuando lleguen otras fuentes (DOGC, BOJA, …) se añadirán al mismo nivel:

```
├── boe/
├── dogc/
└── boja/
```

---

## Cómo se ve un fichero

```yaml
---
identificador: "BOE-A-2025-1"
titulo: "Ley 6/2024, de 5 de diciembre, de simplificación administrativa"
pais: "es"
fuente: "BOE"
fuente_tipo: "estatal"
jurisdiccion: "comunidad-valenciana"
emisor: "Comunitat Valenciana"
departamento: ""
rango: "ley"
numero_oficial: "6/2024"
fecha_disposicion: "2024-12-05"
fecha_publicacion: "2025-01-01"
fecha_vigencia: "2024-12-10"
ultima_actualizacion: "2025-01-01"
estado: "vigente"
seccion: "I. Disposiciones generales"
diario_numero: "1"
pagina_inicial: 1
pagina_final: 293
url_html: "https://www.boe.es/diario_boe/txt.php?id=BOE-A-2025-1"
url_pdf:  "https://www.boe.es/boe/dias/2025/01/01/pdfs/BOE-A-2025-1.pdf"
url_xml:  "https://www.boe.es/diario_boe/xml.php?id=BOE-A-2025-1"
url_eli:  "https://www.boe.es/eli/es-vc/l/2024/12/05/6"
parser_version: "boletines-md/0.1"
---

# Ley 6/2024, de 5 de diciembre, de simplificación administrativa

### TÍTULO I. Regulación general de la simplificación administrativa

### CAPÍTULO I. Disposiciones generales

## Artículo 1. Objeto y finalidad.

Esta ley tiene por objeto...
```

Ver [`SPEC.md`](SPEC.md) para la **especificación completa**: todos los campos del frontmatter, jerarquía de encabezados, mapeo de elementos XML→Markdown y reglas de normalización por fuente.

---

## Cómo usar el corpus

### Clonar y leer

```bash
git clone https://github.com/dcarrero/boletines-md-corpus.git
cd boletines-md-corpus
ls boe/2025/01/01/
```

### Búsqueda con grep / ripgrep

```bash
# Listar todas las leyes orgánicas
rg 'rango: "ley orgánica"' boe/

# Documentos que mencionan "inteligencia artificial"
rg -l "inteligencia artificial" boe/

# Disposiciones de un emisor concreto
rg -l 'emisor: "Ministerio de Hacienda"' boe/

# Todas las disposiciones derogadas
rg 'estado: "derogada"' boe/
```

### Como input de un LLM

Cada fichero es Markdown plano y, en la mayoría de los casos, cabe entero en el contexto de cualquier LLM moderno. Para documentos largos (leyes consolidadas), pásale solo los artículos relevantes:

```python
from pathlib import Path

doc = Path("boe/2025/01/01/BOE-A-2025-1.md").read_text()
# usar 'doc' como contexto del prompt
```

### Como base de datos

El frontmatter YAML hace trivial cargar el corpus en cualquier herramienta:

```python
import yaml
from pathlib import Path

for md_file in Path("boe").rglob("BOE-A-*.md"):
    text = md_file.read_text()
    if not text.startswith("---"):
        continue
    fm_end = text.index("---", 3)
    metadata = yaml.safe_load(text[3:fm_end])
    # metadata tiene 23 campos estructurados
```

---

## Frecuencia de actualización

El corpus se actualiza con el contenido del BOE de cada día laborable (el BOE no publica los domingos). La fecha del último día procesado se puede consultar en el último commit del repo.

---

## Licencia y atribución

- **El contenido** de los boletines oficiales es de **dominio público** en España (art. 13 de la Ley de Propiedad Intelectual). Los textos se reproducen tal y como aparecen en los boletines originales.
- **El formato, la normalización y la organización** del corpus se publican bajo licencia **MIT**.
- **Para citas oficiales**, consulta siempre la URL `url_html` o `url_pdf` del frontmatter del documento. Este corpus es una herramienta de acceso, no una fuente oficial.

---

## Cómo se genera

El código que descarga el BOE y normaliza el XML a Markdown vive en un repo separado. Este repositorio contiene únicamente los datos resultantes del proceso. Si encuentras un error de normalización o un campo de frontmatter incorrecto, abre un issue aquí indicando el `identificador` del documento afectado.
