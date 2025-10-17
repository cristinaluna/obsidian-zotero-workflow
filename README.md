# 📘 Obsidian Literature Notes Dashboard

An interactive **Obsidian dashboard** for managing and exploring literature notes imported from **Zotero**.  
It combines automatic annotation import, structured templates for analytical reflection, and a dynamic overview for browsing by author, concept, or category.

---

## 🧩 Features

- Automatic import of **highlights and annotations** from Zotero  
- Unified literature note template with sections for:
  - Zotero metadata and comments  
  - Personal observations and reflections  
  - Questions and analytical categories  
  - Related texts and interlinked references  
- Dynamic Dataview-based dashboard:
  - Displays notes grouped by **author**, **concept**, or **category**
  - Supports variable-based filtering (`authorFilter`, `categoryFilter`)
  - No code editing required — filters update automatically  

---

## 🧠 Folder Structure

Vault/
├─ Literature_Notes/
│ ├─ AuthorYear_Title.md
│ ├─ ...
├─ Templates/
│ ├─ my_template.md
│ ├─ ...
└─ Knowledge Management Plaza.md


- `Literature_Notes/` → stores automatically created notes via Zotero Integration  
- `Templates/` → contains the literature note template(s)  
- `Knowledge Management Plaza.md` → main dashboard for filtering and visualising all notes  

---

## ⚙️ Requirements

- [Obsidian](https://obsidian.md)
- [Dataview](https://github.com/blacksmithgu/obsidian-dataview)
- [Zotero Integration](https://github.com/mgmeyers/obsidian-zotero-integration)
- *(Optional)* [Templater](https://github.com/SilentVoid13/Templater) for enhanced automation

---

## 🚀 Usage

1. Download the code
2. Open the folder in Obsidian as a Vault. t will configure automatically the different plugins.
3. Create or update your literature notes through the Zotero Integration plugin. Add your notes by opening the command palette: Zotero Integration: Create research note (you should open Zotero first). It will open the Zotero pop-up for selecting the reference.
4. Include any relevant information in the Literature Note, add thoughs, questions and categories.
6. Open **Knowledge Management Plaza.md** to explore, search, and filter your notes dynamically.  
7. Refine your view by adjusting inline variables (e.g. `authorFilter`, `categoryFilter`).  

---

## 🧭 Example Workflow

1. Read and annotate a paper in Zotero.  
2. Sync with Obsidian — the note appears in `Literature_Notes/`.  
3. Add personal reflections, analytical tags, and related references.  
4. Browse all notes via the dashboard to connect concepts and authors.

---

## 🪶 License

This repository is released under the [MIT License](LICENSE).
