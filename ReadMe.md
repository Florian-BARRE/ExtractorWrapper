# Extractor Wrapper

---

## English

**extractor_wrapper** is a lightweight Python package that provides a unified way to extract text (and basic metadata) from common document formats. Instead of learning multiple libraries, you can use one interface to work with:

- **PDF** files (`.pdf`)
- **Word** documents (`.doc` and `.docx`)
- **Excel** spreadsheets (`.xlsx`)
- **PowerPoint** presentations (`.pptx`)
- **Plain text** files (`.txt`)
- **Outlook message** files (`.msg`)

### Principle

1. Each format has its own extractor class under `extractor_wrapper.ext`, for example:
   - `PDFExtractor`
   - `DOCExtractor`
   - `DOCXExtractor`
   - `XLSXExtractor`
   - `PPTXExtractor`
   - `TXTExtractor`
   - `MSGExtractor`

2. If a required third-party library for an extractor is missing at runtime, that extractor is marked as unavailable rather than causing an import error.

3. You can either:
   - Use an extractor class directly, e.g. `PDFExtractor("file.pdf")`.
   - Use the factory method `ExtractorFactory.auto_extract(path)` to automatically pick the right extractor based on file extension.

### Simple Usage Examples

#### 1. Using the factory (`ExtractorFactory.auto_extract`)

```python
import os
from extractor_wrapper import ExtractorFactory

path = "documents/report.pdf"

try:
    content = ExtractorFactory.auto_extract(path)
    print("Extracted content (first 500 chars):")
    print(content[:500])
except Exception as e:
    print(f"Extraction failed: {e}")
````

* **What happens here**:

  * `auto_extract(path)` inspects the file extension (`.pdf`) and instantiates `PDFExtractor` behind the scenes.
  * It returns the full text content (as a single string).
  * If the required library (`pdfminer.six` or `PyPDF2`) is not installed, it raises an error.

#### 2. Instantiating an extractor class directly

```python
from extractor_wrapper.ext.docx import DOCXExtractor

path = "notes/project.docx"

docx_ext = DOCXExtractor()
# No need to check availability explicitly; if a required library is missing, initialization will raise.
try:
    content = docx_ext.extract(path)
    print("DOCX content (first 300 chars):")
    print(content[:300])
except Exception as e:
    print(f"Cannot extract DOCX: {e}")
```

* **What happens here**:

  * You import `DOCXExtractor` and instantiate it with a `.docx` path.
  * The `extract()` method returns a single string containing all paragraphs from the document.
  * If `python-docx` is missing, instantiation or `extract()` will fail with a clear message.

#### 3. Looping over a folder of mixed files

```python
import os
from extractor_wrapper import ExtractorFactory
from loggerplusplus import Logger

logger = Logger(identifier="BatchTest")

INPUT_DIR = "test_files"
OUTPUT_DIR = "extracted_texts"

os.makedirs(OUTPUT_DIR, exist_ok=True)

for filename in os.listdir(INPUT_DIR):
    full_path = os.path.join(INPUT_DIR, filename)
    if not os.path.isfile(full_path):
        continue

    logger.info(f"Testing {filename}")
    try:
        content = ExtractorFactory.auto_extract(full_path)
        out_file = os.path.join(OUTPUT_DIR, filename + ".txt")
        with open(out_file, "w", encoding="utf-8") as f:
            f.write(content)
        logger.info(f"Saved extracted text to {out_file}")
    except Exception as e:
        logger.error(f"Failed to extract {filename}: {e}")
```

* **What happens here**:

  * We walk through each file in `test_files/`.
  * For each file, call `ExtractorFactory.auto_extract`, then write the returned string to a `.txt` file under `extracted_texts/`.
  * If an extractor is unavailable (missing dependency) or any error occurs, it’s caught and logged.

### Project Structure (simplified)

```
extractor_wrapper/                ← repository root
├── extractor_wrapper/            ← Python package
│   ├── __init__.py               ← may define `ExtractorFactory.auto_extract`
│   └── ext/
│       ├── base.py               ← BaseExtractor (shared interface)
│       ├── pdf.py                ← PDFExtractor implementation
│       ├── doc.py                ← DOCExtractor implementation
│       ├── docx.py               ← DOCXExtractor implementation
│       ├── xlsx.py               ← XLSXExtractor implementation
│       ├── pptx.py               ← PPTXExtractor implementation
│       ├── txt.py                ← TXTExtractor implementation
│       └── msg.py                ← MSGExtractor implementation
├── README.md                     ← this file
├── LICENSE                       ← GNU GPL v3 license text
└── setup.py                      ← package installation script
```

### How It Works

* **Extractor Classes** (`extractor_wrapper/ext/*.py`) each implement an `extract()` method:

  * For text-based formats (PDF, DOC, DOCX, TXT, MSG), `extract()` returns a single string of the full text.
  * For spreadsheets (`.xlsx`), you might choose to return a dictionary of sheet names → rows (but the factory returns just the raw content as a string or structured data).
  * For presentations (`.pptx`), `extract()` returns a list of slide strings (the factory might join them internally before returning).

* **Availability**:

  * If the underlying required library is missing, instantiating the extractor or calling `extract()` raises an informative exception instead of silently failing.

* **Factory (`ExtractorFactory.auto_extract`)**:

  * Checks the file extension, picks the right extractor class, calls `extract()`, and returns its result.
  * Simplifies your code so you don’t need to write `if-elif` blocks on extensions.

---

### Author

Project created and maintained by **Florian BARRE**.
For questions or contributions, feel free to contact me.
[My Website](https://florianbarre.fr/) | [LinkedIn](https://www.linkedin.com/in/barre-florian) | [GitHub](https://github.com/Florian-BARRE)

---

## Français

**extractor\_wrapper** est un package Python léger qui fournit une interface unique pour extraire du texte (et des métadonnées basiques) depuis des formats de documents courants. Au lieu de gérer plusieurs bibliothèques, vous pouvez utiliser une seule interface pour :

* **PDF** (`.pdf`)
* **Word** (`.doc` et `.docx`)
* **Excel** (`.xlsx`)
* **PowerPoint** (`.pptx`)
* **Texte brut** (`.txt`)
* **Message Outlook** (`.msg`)

### Principe

1. Chaque format possède sa propre classe d’extracteur dans `extractor_wrapper.ext`, par exemple :

   * `PDFExtractor`
   * `DOCExtractor`
   * `DOCXExtractor`
   * `XLSXExtractor`
   * `PPTXExtractor`
   * `TXTExtractor`
   * `MSGExtractor`

2. Si la bibliothèque tierce nécessaire n’est pas installée, l’extracteur est marqué comme indisponible à l’exécution (aucune erreur d’importation).

3. Vous pouvez :

   * Utiliser directement une classe d’extracteur, par exemple `PDFExtractor("fichier.pdf")`.
   * Utiliser la méthode factory `ExtractorFactory.auto_extract(path)` pour choisir automatiquement l’extracteur en fonction de l’extension du fichier.

### Exemples Simples

#### 1. Avec la factory (`ExtractorFactory.auto_extract`)

```python
import os
from extractor_wrapper import ExtractorFactory

chemin = "documents/rapport.pdf"

try:
    contenu = ExtractorFactory.auto_extract(chemin)
    print("Contenu extrait (500 premiers caractères) :")
    print(contenu[:500])
except Exception as e:
    print(f"Échec de l’extraction : {e}")
```

* **Explication** :

  * `auto_extract(chemin)` regarde l’extension (`.pdf`) et utilise `PDFExtractor`.
  * Renvoie tout le texte du PDF sous forme d’une chaîne.
  * Si `pdfminer.six` ou `PyPDF2` n’est pas installé, une exception est levée.

#### 2. Instancier directement un extracteur

```python
from extractor_wrapper.ext.docx import DOCXExtractor

chemin = "notes/projet.docx"

docx_ext = DOCXExtractor()
# Si la librairie python-docx manque, l’init ou extract() lèvera une erreur.
try:
    contenu = docx_ext.extract(chemin)
    print("Contenu DOCX (300 premiers caractères) :")
    print(contenu[:300])
except Exception as e:
    print(f"Impossible d’extraire le DOCX : {e}")
```

* **Explication** :

  * On importe `DOCXExtractor`, on l’instancie avec un chemin `.docx`.
  * `extract()` renvoie une chaîne contenant tous les paragraphes.
  * Si `python-docx` manque, une erreur claire est produite.

#### 3. Parcourir un dossier de fichiers mixtes

```python
import os
from extractor_wrapper import ExtractorFactory
from loggerplusplus import Logger

logger = Logger(identifier="BatchTest")

REPERTOIRE_ENTREE = "test_files"
REPERTOIRE_SORTIE = "extracted_texts"

os.makedirs(REPERTOIRE_SORTIE, exist_ok=True)

for nom_fichier in os.listdir(REPERTOIRE_ENTREE):
    chemin = os.path.join(REPERTOIRE_ENTREE, nom_fichier)
    if not os.path.isfile(chemin):
        continue

    logger.info(f"Traitement de {nom_fichier}")
    try:
        contenu = ExtractorFactory.auto_extract(chemin)
        sortie = os.path.join(REPERTOIRE_SORTIE, nom_fichier + ".txt")
        with open(sortie, "w", encoding="utf-8") as f:
            f.write(contenu)
        logger.info(f"Texte sauvegardé dans {sortie}")
    except Exception as e:
        logger.error(f"Échec pour {nom_fichier} : {e}")
```

* **Explication** :

  * On parcourt chaque fichier dans `test_files/`.
  * Appel à `auto_extract` → on obtient une chaîne de texte, qu’on écrit dans un `.txt` dans `extracted_texts/`.
  * Si l’extracteur est indisponible ou s’il y a une erreur, on la consigne dans les logs.

---

### Auteur

Projet créé et maintenu par **Florian BARRE**.
Pour toute question ou contribution, n’hésitez pas à me contacter.
[Mon Site](https://florianbarre.fr/) | [Mon LinkedIn](https://www.linkedin.com/in/barre-florian) | [Mon GitHub](https://github.com/Florian-BARRE)
