---
cssclass: dashboard
---
---
cssclass: dashboard
---

# 📚 Literature Hub

## 🎨 Annotation Color Guide

Use these colors consistently in Zotero to categorize your highlights:

| Color | Type | Purpose |
|-------|------|---------|
| 🟡 **Yellow** (`#ffd400`) | Highlight | General important passages |
| 🔴 **Red** (`#ff6666`) | Important | Critical concepts or key arguments |
| 🟢 **Green** (`#5fb236`) | Reference | Citations or references to follow up |
| 🔵 **Blue** (`#2ea8e5`) | Question | Raises questions or needs clarification |
| 🟣 **Purple** (`#a28ae5`) | Definition | Definitions or theoretical frameworks |

---

## Quick Stats

```dataviewjs
const pages = dv.pages('"Literature_Notes"').where(p => p.type);
const totalNotes = pages.length;
const totalAuthors = new Set(pages.flatMap(p => p.authors || [])).size;
const unreadCount = pages.where(p => p["reading-status"] === "unread").length;

dv.paragraph(`
**Total Literature Notes**: ${totalNotes} | 
**Unique Authors**: ${totalAuthors} | 
**Unread**: ${unreadCount}
`);
```

---

## 📋 All Literature Notes

```dataviewjs
const pages = dv.pages('"Literature_Notes"')
    .where(p => p.type)
    .sort(p => p.date, 'desc');

dv.table(
    ["Title", "Authors", "Type", "Date", "Status"],
    pages.map(p => [
        p.file.link,
        p["author-display"] || (p.authors ? (p.authors.length > 1 ? p.authors[0] + " et al." : p.authors[0]) : "Unknown"),
        p.type || "N/A",
        p.date || "N/A",
        p["reading-status"] || "N/A"
    ])
);
```

---

## 👤 Browse by Author

### Author Search

```dataviewjs
const input = dv.container.createEl("input", {
    type: "text",
    placeholder: "Search author name...",
    attr: { style: "width: 100%; padding: 8px; margin-bottom: 20px; border: 1px solid var(--background-modifier-border); border-radius: 4px;" }
});

const resultsDiv = dv.container.createEl("div");

function updateResults() {
    const searchTerm = input.value.toLowerCase();
    resultsDiv.innerHTML = "";
    
    if (searchTerm.length < 2) {
        resultsDiv.innerHTML = "<em>Enter at least 2 characters to search...</em>";
        return;
    }
    
    const pages = dv.pages('"Literature_Notes"')
        .where(p => p.authors && p.authors.some(a => 
            a.toLowerCase().includes(searchTerm)
        ));
    
    if (pages.length === 0) {
        resultsDiv.innerHTML = "<em>No authors found</em>";
        return;
    }
    
    // Group by author
    const authorMap = {};
    pages.forEach(page => {
        (page.authors || []).forEach(author => {
            if (author.toLowerCase().includes(searchTerm)) {
                if (!authorMap[author]) {
                    authorMap[author] = [];
                }
                authorMap[author].push(page);
            }
        });
    });
    
    // Display results
    Object.keys(authorMap).sort().forEach(author => {
        const authorSection = dv.el("div", "", {
            attr: { style: "margin-bottom: 30px; padding: 15px; background: var(--background-secondary); border-radius: 8px;" }
        });
        
        dv.el("h3", `${author} (${authorMap[author].length} works)`, {
            container: authorSection
        });
        
        authorMap[author].forEach(work => {
            const workDiv = dv.el("div", "", {
                container: authorSection,
                attr: { style: "margin-left: 20px; margin-bottom: 10px;" }
            });
            
            const authorDisplay = work["author-display"] || (work.authors.length > 1 ? work.authors[0] + " et al." : work.authors[0]);
            
            dv.el("strong", "", {
                container: workDiv
            }).innerHTML = `<a href="${work.file.path}">${work.title || work.file.name}</a>`;
            
            dv.el("span", ` (${work.date || "No date"}) - ${authorDisplay}`, {
                container: workDiv,
                attr: { style: "color: var(--text-muted);" }
            });
        });
        
        resultsDiv.appendChild(authorSection);
    });
}

input.addEventListener("input", updateResults);
updateResults();
```

---

## 🏷️ Browse by Categories & Concepts

### Category/Concept Search (Document Level)

```dataviewjs
const input = dv.container.createEl("input", {
    type: "text",
    placeholder: "Search categories or concepts (e.g., epistemology, methodology)...",
    attr: { style: "width: 100%; padding: 8px; margin-bottom: 20px; border: 1px solid var(--background-modifier-border); border-radius: 4px;" }
});

const resultsDiv = dv.container.createEl("div");

async function updateCategoryResults() {
    const searchTerm = input.value.toLowerCase();
    resultsDiv.innerHTML = "";
    
    if (searchTerm.length < 2) {
        resultsDiv.innerHTML = "<em>Enter at least 2 characters to search...</em>";
        return;
    }
    
    resultsDiv.innerHTML = "<em>Searching...</em>";
    
    const pages = dv.pages('"Literature_Notes"').where(p => p.type);
    const matchingPages = [];
    
    for (const page of pages) {
        const generalCategories = page["general-categories"] || [];
        
        const matchingTags = generalCategories.filter(tag => 
            tag.toLowerCase().includes(searchTerm)
        );
        
        if (matchingTags.length > 0) {
            matchingPages.push({
                page: page,
                tags: [...new Set(matchingTags)]
            });
        }
    }
    
    resultsDiv.innerHTML = "";
    
    if (matchingPages.length === 0) {
        resultsDiv.innerHTML = "<em>No matching categories or concepts found</em>";
        return;
    }
    
    dv.el("div", `Found ${matchingPages.length} works`, {
        container: resultsDiv,
        attr: { style: "margin-bottom: 15px; font-weight: bold;" }
    });
    
    matchingPages.forEach(({page, tags}) => {
        const workDiv = dv.el("div", "", {
            container: resultsDiv,
            attr: { style: "margin-bottom: 20px; padding: 15px; background: var(--background-secondary); border-radius: 8px;" }
        });
        
        dv.el("div", "", {
            container: workDiv
        }).innerHTML = `<strong><a href="${page.file.path}">${page.title || page.file.name}</a></strong>`;
        
        const authorDisplay = page["author-display"] || (page.authors ? (page.authors.length > 1 ? page.authors[0] + " et al." : page.authors[0]) : "Unknown");
        
        dv.el("div", `By: ${authorDisplay}`, {
            container: workDiv,
            attr: { style: "color: var(--text-muted); font-size: 0.9em; margin-top: 5px;" }
        });
        
        dv.el("div", `Matching tags: ${tags.join(", ")}`, {
            container: workDiv,
            attr: { style: "color: var(--text-accent); font-size: 0.9em; margin-top: 5px;" }
        });
    });
}

input.addEventListener("input", () => {
    clearTimeout(input.searchTimeout);
    input.searchTimeout = setTimeout(updateCategoryResults, 300);
});
updateCategoryResults();
```

---

## 🔍 Browse Annotations by Category

### Search Annotations by Their Individual Categories

```dataviewjs
const input = dv.container.createEl("input", {
    type: "text",
    placeholder: "Search categories in annotations (e.g., methodology, findings)...",
    attr: { style: "width: 100%; padding: 8px; margin-bottom: 20px; border: 1px solid var(--background-modifier-border); border-radius: 4px;" }
});

const resultsDiv = dv.container.createEl("div");

async function extractAnnotationsWithCategories(page, searchTerm) {
    const file = app.vault.getAbstractFileByPath(page.file.path);
    if (!file) return [];
    
    const content = await app.vault.read(file);
    const annotations = [];
    
    const lines = content.split('\n');
    let i = 0;
    
    while (i < lines.length) {
        const line = lines[i];
        
        if (line.match(/^>\[!quote/)) {
            let annotationText = '';
            let pageNum = '';
            let link = '';
            let comment = '';
            let color = '';
            let categories = '';
            
            const colorMatch = line.match(/>\[!quote\|([^\]]+)\]/);
            if (colorMatch) {
                color = colorMatch[1];
            }
            
            i++;
            while (i < lines.length && (lines[i].startsWith('>') || lines[i].trim() === '')) {
                const quoteLine = lines[i];
                
                const textMatch = quoteLine.match(/>([^[\(]+)\s*\[\(p\.\s*(\d+)\)\]/);
                if (textMatch) {
                    annotationText = textMatch[1].trim();
                    pageNum = textMatch[2];
                    
                    const linkMatch = quoteLine.match(/\(zotero:\/\/[^\)]+\)/);
                    if (linkMatch) {
                        link = linkMatch[0].slice(1, -1);
                    }
                }
                
                const commentMatch = quoteLine.match(/>\*\*Comment\*\*:\s*(.+)/);
                if (commentMatch) {
                    comment = commentMatch[1].trim();
                }
                
                const categoriesMatch = quoteLine.match(/>\*\*Categories\*\*::\s*(.+)/);
                if (categoriesMatch) {
                    categories = categoriesMatch[1].trim();
                }
                
                i++;
            }
            
            // Check if categories match search term
            if (annotationText && categories && categories.toLowerCase().includes(searchTerm)) {
                annotations.push({
                    text: annotationText,
                    page: pageNum,
                    link: link,
                    comment: comment,
                    color: color,
                    categories: categories
                });
            }
        } else {
            i++;
        }
    }
    
    return annotations;
}

function getColorLabel(color) {
    const labels = {
        "#ffd400": "🟡 Highlight",
        "#ff6666": "🔴 Important",
        "#5fb236": "🟢 Reference",
        "#2ea8e5": "🔵 Question",
        "#a28ae5": "🟣 Definition"
    };
    return labels[color] || "Note";
}

async function searchAnnotationsByCategory() {
    const searchTerm = input.value.toLowerCase();
    resultsDiv.innerHTML = "";
    
    if (searchTerm.length < 2) {
        resultsDiv.innerHTML = "<em>Enter at least 2 characters to search...</em>";
        return;
    }
    
    resultsDiv.innerHTML = "<em>Searching annotations...</em>";
    
    const pages = dv.pages('"Literature_Notes"').where(p => p.type);
    const matchingAnnotations = [];
    
    for (const page of pages) {
        const annotations = await extractAnnotationsWithCategories(page, searchTerm);
        
        if (annotations.length > 0) {
            matchingAnnotations.push({
                page: page,
                annotations: annotations
            });
        }
    }
    
    resultsDiv.innerHTML = "";
    
    if (matchingAnnotations.length === 0) {
        resultsDiv.innerHTML = "<em>No annotations found with matching categories</em>";
        return;
    }
    
    const totalCount = matchingAnnotations.reduce((sum, item) => sum + item.annotations.length, 0);
    
    dv.el("div", `Found ${totalCount} annotations across ${matchingAnnotations.length} works`, {
        container: resultsDiv,
        attr: { style: "margin-bottom: 20px; font-weight: bold; font-size: 1.1em;" }
    });
    
    matchingAnnotations.forEach(({ page, annotations }) => {
        const workDiv = dv.el("div", "", {
            container: resultsDiv,
            attr: { style: "margin-bottom: 30px; padding: 15px; background: var(--background-secondary); border-radius: 8px;" }
        });
        
        const authorDisplay = page["author-display"] || (page.authors ? (page.authors.length > 1 ? page.authors[0] + " et al." : page.authors[0]) : "Unknown");
        
        dv.el("div", "", {
            container: workDiv
        }).innerHTML = `<h3><a href="${page.file.path}">${page.title || page.file.name}</a></h3>`;
        
        dv.el("div", `${authorDisplay} | ${annotations.length} annotation${annotations.length > 1 ? 's' : ''}`, {
            container: workDiv,
            attr: { style: "color: var(--text-muted); margin-bottom: 15px;" }
        });
        
        annotations.forEach((annotation) => {
            const annotDiv = dv.el("div", "", {
                container: workDiv,
                attr: { style: "margin-left: 20px; margin-bottom: 15px; padding: 10px; background: var(--background-primary); border-left: 3px solid var(--text-accent); border-radius: 4px;" }
            });
            
            // Type label
            dv.el("div", getColorLabel(annotation.color), {
                container: annotDiv,
                attr: { style: "font-size: 0.8em; color: var(--text-muted); margin-bottom: 5px; font-weight: bold;" }
            });
            
            // Quote text
            dv.el("div", `"${annotation.text}"`, {
                container: annotDiv,
                attr: { style: "font-style: italic; margin-bottom: 8px;" }
            });
            
            // Comment
            if (annotation.comment) {
                dv.el("div", `💭 ${annotation.comment}`, {
                    container: annotDiv,
                    attr: { style: "font-size: 0.9em; color: var(--text-accent); margin-bottom: 8px; padding: 5px; background: var(--background-secondary); border-radius: 3px;" }
                });
            }
            
            // Categories
            dv.el("div", `🏷️ Categories: ${annotation.categories}`, {
                container: annotDiv,
                attr: { style: "font-size: 0.85em; color: var(--text-accent); margin-bottom: 8px; font-weight: bold;" }
            });
            
            // Link
            const linkDiv = dv.el("div", "", {
                container: annotDiv,
                attr: { style: "font-size: 0.85em; color: var(--text-muted);" }
            });
            
            linkDiv.innerHTML = `<a href="${page.file.path}">View in note</a> | Page ${annotation.page}`;
        });
    });
}

input.addEventListener("input", () => {
    clearTimeout(input.searchTimeout);
    input.searchTimeout = setTimeout(searchAnnotationsByCategory, 300);
});
searchAnnotationsByCategory();
```

---

## 📖 Extract Quotes & Highlights

### Search Quotes by Author or Category

```dataviewjs
const searchTypeDiv = dv.container.createEl("div", {
    attr: { style: "margin-bottom: 15px;" }
});

const authorRadio = dv.el("input", "", {
    container: searchTypeDiv,
    attr: { type: "radio", name: "searchType", value: "author", id: "authorRadio", checked: true }
});
dv.el("label", " By Author ", {
    container: searchTypeDiv,
    attr: { for: "authorRadio", style: "margin-right: 20px;" }
});

const categoryRadio = dv.el("input", "", {
    container: searchTypeDiv,
    attr: { type: "radio", name: "searchType", value: "category", id: "categoryRadio" }
});
dv.el("label", " By Document Category", {
    container: searchTypeDiv,
    attr: { for: "categoryRadio" }
});

const input = dv.container.createEl("input", {
    type: "text",
    placeholder: "Search for quotes...",
    attr: { style: "width: 100%; padding: 8px; margin-bottom: 10px; border: 1px solid var(--background-modifier-border); border-radius: 4px;" }
});

// Color filter
const colorFilterDiv = dv.container.createEl("div", {
    attr: { style: "margin-bottom: 20px; padding: 10px; background: var(--background-secondary); border-radius: 4px;" }
});

dv.el("label", "Filter by annotation type: ", {
    container: colorFilterDiv,
    attr: { style: "margin-right: 10px; font-weight: bold;" }
});

const colorSelect = dv.el("select", "", {
    container: colorFilterDiv,
    attr: { style: "padding: 5px; border-radius: 4px; border: 1px solid var(--background-modifier-border);" }
});

const colorOptions = [
    { value: "all", label: "All Types" },
    { value: "#ffd400", label: "🟡 Highlight" },
    { value: "#ff6666", label: "🔴 Important" },
    { value: "#5fb236", label: "🟢 Reference" },
    { value: "#2ea8e5", label: "🔵 Question" },
    { value: "#a28ae5", label: "🟣 Definition" }
];

colorOptions.forEach(opt => {
    const option = dv.el("option", opt.label, {
        container: colorSelect,
        attr: { value: opt.value }
    });
});

const resultsDiv = dv.container.createEl("div");

async function extractQuotes(page, filterColor = "all") {
    const file = app.vault.getAbstractFileByPath(page.file.path);
    if (!file) return [];
    
    const content = await app.vault.read(file);
    const quotes = [];
    
    const lines = content.split('\n');
    let i = 0;
    
    while (i < lines.length) {
        const line = lines[i];
        
        if (line.match(/^>\[!quote/)) {
            let quoteText = '';
            let pageNum = '';
            let link = '';
            let comment = '';
            let color = '';
            
            const colorMatch = line.match(/>\[!quote\|([^\]]+)\]/);
            if (colorMatch) {
                color = colorMatch[1];
            }
            
            if (filterColor !== "all" && color !== filterColor) {
                i++;
                continue;
            }
            
            i++;
            while (i < lines.length && (lines[i].startsWith('>') || lines[i].trim() === '')) {
                const quoteLine = lines[i];
                
                const textMatch = quoteLine.match(/>([^[\(]+)\s*\[\(p\.\s*(\d+)\)\]/);
                if (textMatch) {
                    quoteText = textMatch[1].trim();
                    pageNum = textMatch[2];
                    
                    const linkMatch = quoteLine.match(/\(zotero:\/\/[^\)]+\)/);
                    if (linkMatch) {
                        link = linkMatch[0].slice(1, -1);
                    }
                }
                
                const commentMatch = quoteLine.match(/>\*\*Comment\*\*:\s*(.+)/);
                if (commentMatch) {
                    comment = commentMatch[1].trim();
                }
                
                i++;
            }
            
            if (quoteText) {
                quotes.push({
                    text: quoteText,
                    page: pageNum,
                    link: link,
                    comment: comment,
                    color: color
                });
            }
        } else {
            i++;
        }
    }
    
    return quotes;
}

function getColorLabel(color) {
    const labels = {
        "#ffd400": "🟡 Highlight",
        "#ff6666": "🔴 Important",
        "#5fb236": "🟢 Reference",
        "#2ea8e5": "🔵 Question",
        "#a28ae5": "🟣 Definition"
    };
    return labels[color] || "Note";
}

async function updateQuoteResults() {
    const searchTerm = input.value.toLowerCase();
    const searchType = authorRadio.checked ? "author" : "category";
    const colorFilter = colorSelect.value;
    resultsDiv.innerHTML = "";
    
    if (searchTerm.length < 2) {
        resultsDiv.innerHTML = "<em>Enter at least 2 characters to search...</em>";
        return;
    }
    
    resultsDiv.innerHTML = "<em>Loading quotes...</em>";
    
    const pages = dv.pages('"Literature_Notes"').where(p => p.type);
    let matchingPages = [];
    
    if (searchType === "author") {
        matchingPages = pages.where(p => 
            p.authors && p.authors.some(a => 
                a.toLowerCase().includes(searchTerm)
            )
        );
    } else {
        matchingPages = pages.where(p => {
            const generalCategories = p["general-categories"] || [];
            return generalCategories.some(tag => tag.toLowerCase().includes(searchTerm));
        });
    }
    
    if (matchingPages.length === 0) {
        resultsDiv.innerHTML = "<em>No matching works found</em>";
        return;
    }
    
    resultsDiv.innerHTML = "";
    let totalQuotes = 0;
    
    for (const page of matchingPages) {
        const quotes = await extractQuotes(page, colorFilter);
        
        if (quotes.length > 0) {
            totalQuotes += quotes.length;
            
            const workDiv = dv.el("div", "", {
                container: resultsDiv,
                attr: { style: "margin-bottom: 30px; padding: 15px; background: var(--background-secondary); border-radius: 8px;" }
            });
            
            const authorDisplay = page["author-display"] || (page.authors ? (page.authors.length > 1 ? page.authors[0] + " et al." : page.authors[0]) : "Unknown");
            
            dv.el("div", "", {
                container: workDiv
            }).innerHTML = `<h3><a href="${page.file.path}">${page.title || page.file.name}</a></h3>`;
            
            dv.el("div", `${authorDisplay} | ${quotes.length} quote${quotes.length > 1 ? 's' : ''}`, {
                container: workDiv,
                attr: { style: "color: var(--text-muted); margin-bottom: 15px;" }
            });
            
            quotes.forEach((quote) => {
                const quoteDiv = dv.el("div", "", {
                    container: workDiv,
                    attr: { style: "margin-left: 20px; margin-bottom: 15px; padding: 10px; background: var(--background-primary); border-left: 3px solid var(--text-accent); border-radius: 4px;" }
                });
                
                dv.el("div", getColorLabel(quote.color), {
                    container: quoteDiv,
                    attr: { style: "font-size: 0.8em; color: var(--text-muted); margin-bottom: 5px; font-weight: bold;" }
                });
                
                dv.el("div", `"${quote.text}"`, {
                    container: quoteDiv,
                    attr: { style: "font-style: italic; margin-bottom: 8px;" }
                });
                
                if (quote.comment) {
                    dv.el("div", `💭 ${quote.comment}`, {
                        container: quoteDiv,
                        attr: { style: "font-size: 0.9em; color: var(--text-accent); margin-bottom: 8px; padding: 5px; background: var(--background-secondary); border-radius: 3px;" }
                    });
                }
                
                const linkDiv = dv.el("div", "", {
                    container: quoteDiv,
                    attr: { style: "font-size: 0.85em; color: var(--text-muted);" }
                });
                
                linkDiv.innerHTML = `<a href="${page.file.path}">View in note</a> | Page ${quote.page}`;
            });
        }
    }
    
    if (totalQuotes === 0) {
        const filterMsg = colorFilter !== "all" ? ` with the selected color filter` : ``;
        resultsDiv.innerHTML = `<em>No quotes found${filterMsg} in matching works.</em>`;
    }
}

authorRadio.addEventListener("change", updateQuoteResults);
categoryRadio.addEventListener("change", updateQuoteResults);
colorSelect.addEventListener("change", updateQuoteResults);
input.addEventListener("input", () => {
    clearTimeout(input.searchTimeout);
    input.searchTimeout = setTimeout(updateQuoteResults, 300);
});
updateQuoteResults();
```

---

## 🎨 Browse Annotations by Type

### All Annotations by Color

```dataviewjs
const colorFilterDiv = dv.container.createEl("div", {
    attr: { style: "margin-bottom: 20px; padding: 15px; background: var(--background-secondary); border-radius: 8px;" }
});

dv.el("h4", "Select annotation type:", {
    container: colorFilterDiv
});

const colorSelect = dv.el("select", "", {
    container: colorFilterDiv,
    attr: { style: "width: 100%; padding: 8px; border-radius: 4px; border: 1px solid var(--background-modifier-border); margin-top: 10px;" }
});

const colorOptions = [
    { value: "#ffd400", label: "🟡 Highlights - General important passages" },
    { value: "#ff6666", label: "🔴 Important - Critical concepts or key arguments" },
    { value: "#5fb236", label: "🟢 References - Citations to follow up" },
    { value: "#2ea8e5", label: "🔵 Questions - Needs clarification" },
    { value: "#a28ae5", label: "🟣 Definitions - Theoretical frameworks" }
];

colorOptions.forEach(opt => {
    dv.el("option", opt.label, {
        container: colorSelect,
        attr: { value: opt.value }
    });
});

const resultsDiv = dv.container.createEl("div");

async function extractAnnotationsByColor(page, targetColor) {
    const file = app.vault.getAbstractFileByPath(page.file.path);
    if (!file) return [];
    
    const content = await app.vault.read(file);
    const annotations = [];
    const lines = content.split('\n');
    let i = 0;
    
    while (i < lines.length) {
        const line = lines[i];
        
        if (line.match(/^>\[!quote/)) {
            let annotationText = '';
            let pageNum = '';
            let comment = '';
            let color = '';
            
            const colorMatch = line.match(/>\[!quote\|([^\]]+)\]/);
            if (colorMatch) {
                color = colorMatch[1];
            }
            
            if (color === targetColor) {
                i++;
                while (i < lines.length && (lines[i].startsWith('>') || lines[i].trim() === '')) {
                    const quoteLine = lines[i];
                    
                    const textMatch = quoteLine.match(/>([^[\(]+)\s*\[\(p\.\s*(\d+)\)\]/);
                    if (textMatch) {
                        annotationText = textMatch[1].trim();
                        pageNum = textMatch[2];
                    }
                    
                    const commentMatch = quoteLine.match(/>\*\*Comment\*\*:\s*(.+)/);
                    if (commentMatch) {
                        comment = commentMatch[1].trim();
                    }
                    
                    i++;
                }
                
                if (annotationText) {
                    annotations.push({
                        text: annotationText,
                        page: pageNum,
                        comment: comment
                    });
                }
            } else {
                i++;
            }
        } else {
            i++;
        }
    }
    
    return annotations;
}

function getColorLabel(color) {
    const labels = {
        "#ffd400": "🟡 Highlight",
        "#ff6666": "🔴 Important",
        "#5fb236": "🟢 Reference",
        "#2ea8e5": "🔵 Question",
        "#a28ae5": "🟣 Definition"
    };
    return labels[color] || "Note";
}

async function displayAnnotationsByColor() {
    const selectedColor = colorSelect.value;
    resultsDiv.innerHTML = "<em>Loading annotations...</em>";
    
    const pages = dv.pages('"Literature_Notes"').where(p => p.type);
    const allAnnotations = [];
    
    for (const page of pages) {
        const annotations = await extractAnnotationsByColor(page, selectedColor);
        
        if (annotations.length > 0) {
            allAnnotations.push({
                page: page,
                annotations: annotations
            });
        }
    }
    
    resultsDiv.innerHTML = "";
    
    if (allAnnotations.length === 0) {
        resultsDiv.innerHTML = `<em>No annotations found with type ${getColorLabel(selectedColor)}</em>`;
        return;
    }
    
    const totalCount = allAnnotations.reduce((sum, item) => sum + item.annotations.length, 0);
    
    dv.el("div", `Found ${totalCount} ${getColorLabel(selectedColor)} annotations across ${allAnnotations.length} works`, {
        container: resultsDiv,
        attr: { style: "margin-bottom: 20px; font-weight: bold; font-size: 1.1em;" }
    });
    
    allAnnotations.forEach(({ page, annotations }) => {
        const workDiv = dv.el("div", "", {
            container: resultsDiv,
            attr: { style: "margin-bottom: 30px; padding: 15px; background: var(--background-secondary); border-radius: 8px;" }
        });
        
        const authorDisplay = page["author-display"] || (page.authors ? (page.authors.length > 1 ? page.authors[0] + " et al." : page.authors[0]) : "Unknown");
        
        dv.el("div", "", {
            container: workDiv
        }).innerHTML = `<h3><a href="${page.file.path}">${page.title || page.file.name}</a></h3>`;
        
        dv.el("div", `${authorDisplay} | ${annotations.length} annotation${annotations.length > 1 ? 's' : ''}`, {
            container: workDiv,
            attr: { style: "color: var(--text-muted); margin-bottom: 15px;" }
        });
        
        annotations.forEach((annotation) => {
            const annotDiv = dv.el("div", "", {
                container: workDiv,
                attr: { style: "margin-left: 20px; margin-bottom: 15px; padding: 10px; background: var(--background-primary); border-left: 3px solid var(--text-accent); border-radius: 4px;" }
            });
            
            dv.el("div", `"${annotation.text}"`, {
                container: annotDiv,
                attr: { style: "font-style: italic; margin-bottom: 8px;" }
            });
            
            if (annotation.comment) {
                dv.el("div", `💭 ${annotation.comment}`, {
                    container: annotDiv,
                    attr: { style: "font-size: 0.9em; color: var(--text-accent); margin-bottom: 8px; padding: 5px; background: var(--background-secondary); border-radius: 3px;" }
                });
            }
            
            const linkDiv = dv.el("div", "", {
                container: annotDiv,
                attr: { style: "font-size: 0.85em; color: var(--text-muted);" }
            });
            
            linkDiv.innerHTML = `<a href="${page.file.path}">View in note</a> | Page ${annotation.page}`;
        });
    });
}

colorSelect.addEventListener("change", displayAnnotationsByColor);
displayAnnotationsByColor();
```

---

## 📊 Analytics

### Most Cited Authors

```dataviewjs
const pages = dv.pages('"Literature_Notes"').where(p => p.type);
const authorCount = {};

pages.forEach(p => {
    (p.authors || []).forEach(author => {
        authorCount[author] = (authorCount[author] || 0) + 1;
    });
});

const sortedAuthors = Object.entries(authorCount)
    .sort((a, b) => b[1] - a[1])
    .slice(0, 15);

dv.table(
    ["Author", "Works"],
    sortedAuthors.map(([author, count]) => [author, count])
);
```

### Reading Status Distribution

```dataviewjs
const pages = dv.pages('"Literature_Notes"').where(p => p.type);
const statusCount = {};

pages.forEach(p => {
    const status = p["reading-status"] || "unread";
    statusCount[status] = (statusCount[status] || 0) + 1;
});

dv.table(
    ["Status", "Count"],
    Object.entries(statusCount).map(([status, count]) => [status, count])
);
```

### Publications by Year

```dataviewjs
const pages = dv.pages('"Literature_Notes"').where(p => p.type && p.date);
const yearCount = {};

pages.forEach(p => {
    const year = p.date.toString().substring(0, 4);
    yearCount[year] = (yearCount[year] || 0) + 1;
});

const sortedYears = Object.entries(yearCount)
    .sort((a, b) => b[0] - a[0]);

dv.table(
    ["Year", "Publications"],
    sortedYears.map(([year, count]) => [year, count])
);
```

---

## 🏷️ Categories & Concepts Index

### All General Categories Used

```dataviewjs
const pages = dv.pages('"Literature_Notes"').where(p => p.type);
const categoryCount = {};

for (const page of pages) {
    const generalCategories = page["general-categories"] || [];
    generalCategories.forEach(cat => {
        categoryCount[cat] = (categoryCount[cat] || 0) + 1;
    });
}

const sortedTags = Object.entries(categoryCount)
    .sort((a, b) => b[1] - a[1]);

if (sortedTags.length === 0) {
    dv.paragraph("*No general categories have been tagged yet. Add them to your literature notes in the 'general-categories' frontmatter field.*");
} else {
    dv.table(
        ["Category/Concept", "Count"],
        sortedTags.map(([tag, count]) => [tag, count])
    );
}
```

### Category Cloud

```dataviewjs
const pages = dv.pages('"Literature_Notes"').where(p => p.type);
const categoryCount = {};

for (const page of pages) {
    const generalCategories = page["general-categories"] || [];
    generalCategories.forEach(cat => {
        categoryCount[cat] = (categoryCount[cat] || 0) + 1;
    });
}

if (Object.keys(categoryCount).length === 0) {
    dv.paragraph("*Start adding general categories to see your tag cloud!*");
} else {
    const maxCount = Math.max(...Object.values(categoryCount), 1);
    const cloudDiv = dv.container.createEl("div", {
        attr: { style: "display: flex; flex-wrap: wrap; gap: 10px; margin-top: 20px;" }
    });
    
    Object.entries(categoryCount)
        .sort((a, b) => b[1] - a[1])
        .forEach(([tag, count]) => {
            const size = 0.8 + (count / maxCount) * 1.5;
            const opacity = 0.6 + (count / maxCount) * 0.4;
            
            const tagEl = dv.el("span", tag, {
                container: cloudDiv,
                attr: { 
                    style: `
                        font-size: ${size}em; 
                        opacity: ${opacity};
                        padding: 5px 10px;
                        background: var(--background-secondary);
                        border-radius: 4px;
                        cursor: pointer;
                        transition: all 0.2s;
                    `,
                    class: "category-tag"
                }
            });
            
            tagEl.addEventListener("mouseenter", (e) => {
                e.target.style.background = "var(--interactive-accent)";
                e.target.style.color = "var(--text-on-accent)";
            });
            
            tagEl.addEventListener("mouseleave", (e) => {
                e.target.style.background = "var(--background-secondary)";
                e.target.style.color = "";
            });
        });
}
```

---

## 📊 Annotation Statistics

### Distribution by Type

```dataviewjs
const pages = dv.pages('"Literature_Notes"').where(p => p.type);
const colorCount = {
    "#ffd400": { count: 0, label: "🟡 Highlights" },
    "#ff6666": { count: 0, label: "🔴 Important" },
    "#5fb236": { count: 0, label: "🟢 References" },
    "#2ea8e5": { count: 0, label: "🔵 Questions" },
    "#a28ae5": { count: 0, label: "🟣 Definitions" }
};

async function countAnnotationsByColor() {
    for (const page of pages) {
        const file = app.vault.getAbstractFileByPath(page.file.path);
        if (!file) continue;
        
        const content = await app.vault.read(file);
        const lines = content.split('\n');
        
        for (let line of lines) {
            if (line.match(/^>\[!quote/)) {
                const colorMatch = line.match(/>\[!quote\|([^\]]+)\]/);
                if (colorMatch) {
                    const color = colorMatch[1];
                    if (colorCount[color]) {
                        colorCount[color].count++;
                    }
                }
            }
        }
    }
    
    const sortedColors = Object.entries(colorCount)
        .sort((a, b) => b[1].count - a[1].count);
    
    if (sortedColors.every(([_, data]) => data.count === 0)) {
        dv.paragraph("*No annotations found yet. Start highlighting in Zotero!*");
    } else {
        dv.table(
            ["Annotation Type", "Count"],
            sortedColors.map(([color, data]) => [data.label, data.count])
        );
    }
}

countAnnotationsByColor();
```

---

## 🔧 Quick Actions

### Batch Update Reading Status

```dataviewjs
const buttonContainer = dv.container.createEl("div", {
    attr: { style: "display: flex; gap: 10px; flex-wrap: wrap; margin-bottom: 20px;" }
});

const markReadButton = dv.el("button", "Mark All as Read", {
    container: buttonContainer,
    attr: { style: "padding: 8px 16px; background: var(--interactive-accent); color: var(--text-on-accent); border: none; border-radius: 4px; cursor: pointer;" }
});

const markUnreadButton = dv.el("button", "Mark All as Unread", {
    container: buttonContainer,
    attr: { style: "padding: 8px 16px; background: var(--background-modifier-border); color: var(--text-normal); border: none; border-radius: 4px; cursor: pointer;" }
});

markReadButton.addEventListener("click", async () => {
    const pages = dv.pages('"Literature_Notes"').where(p => p["reading-status"] === "unread");
    
    if (pages.length === 0) {
        alert("No unread notes found!");
        return;
    }
    
    if (confirm(`Mark ${pages.length} notes as read?`)) {
        for (const page of pages) {
            const file = app.vault.getAbstractFileByPath(page.file.path);
            if (file) {
                await app.fileManager.processFrontMatter(file, (fm) => {
                    fm["reading-status"] = "read";
                });
            }
        }
        alert("Updated successfully!");
        window.location.reload();
    }
});

markUnreadButton.addEventListener("click", async () => {
    const pages = dv.pages('"Literature_Notes"').where(p => p["reading-status"] === "read");
    
    if (pages.length === 0) {
        alert("No read notes found!");
        return;
    }
    
    if (confirm(`Mark ${pages.length} notes as unread?`)) {
        for (const page of pages) {
            const file = app.vault.getAbstractFileByPath(page.file.path);
            if (file) {
                await app.fileManager.processFrontMatter(file, (fm) => {
                    fm["reading-status"] = "unread";
                });
            }
        }
        alert("Updated successfully!");
        window.location.reload();
    }
});
```

---

## 🔍 Advanced Search

### Combined Author + Category Search

```dataviewjs
const searchContainer = dv.container.createEl("div", {
    attr: { style: "margin-bottom: 20px; padding: 15px; background: var(--background-secondary); border-radius: 8px;" }
});

dv.el("h4", "Search by multiple criteria", {
    container: searchContainer
});

const authorInput = dv.el("input", "", {
    container: searchContainer,
    attr: { 
        type: "text", 
        placeholder: "Author name (optional)...",
        style: "width: 100%; padding: 8px; margin-bottom: 10px; border: 1px solid var(--background-modifier-border); border-radius: 4px;"
    }
});

const categoryInput = dv.el("input", "", {
    container: searchContainer,
    attr: { 
        type: "text", 
        placeholder: "Category/concept (optional)...",
        style: "width: 100%; padding: 8px; margin-bottom: 10px; border: 1px solid var(--background-modifier-border); border-radius: 4px;"
    }
});

const yearInput = dv.el("input", "", {
    container: searchContainer,
    attr: { 
        type: "text", 
        placeholder: "Year (optional)...",
        style: "width: 100%; padding: 8px; margin-bottom: 10px; border: 1px solid var(--background-modifier-border); border-radius: 4px;"
    }
});

const resultsDiv = dv.container.createEl("div");

async function performAdvancedSearch() {
    const authorTerm = authorInput.value.toLowerCase();
    const categoryTerm = categoryInput.value.toLowerCase();
    const yearTerm = yearInput.value;
    
    if (!authorTerm && !categoryTerm && !yearTerm) {
        resultsDiv.innerHTML = "<em>Enter at least one search criterion...</em>";
        return;
    }
    
    resultsDiv.innerHTML = "<em>Searching...</em>";
    
    let pages = dv.pages('"Literature_Notes"').where(p => p.type);
    const matchingPages = [];
    
    for (const page of pages) {
        let matches = true;
        
        if (authorTerm && matches) {
            matches = page.authors && page.authors.some(a => 
                a.toLowerCase().includes(authorTerm)
            );
        }
        
        if (categoryTerm && matches) {
            const generalCategories = page["general-categories"] || [];
            matches = generalCategories.some(tag => tag.toLowerCase().includes(categoryTerm));
        }
        
        if (yearTerm && matches) {
            matches = page.date && page.date.toString().includes(yearTerm);
        }
        
        if (matches) {
            matchingPages.push(page);
        }
    }
    
    resultsDiv.innerHTML = "";
    
    if (matchingPages.length === 0) {
        resultsDiv.innerHTML = "<em>No results found matching all criteria</em>";
        return;
    }
    
    dv.el("div", `Found ${matchingPages.length} matching works`, {
        container: resultsDiv,
        attr: { style: "margin-bottom: 15px; font-weight: bold;" }
    });
    
    matchingPages.forEach(page => {
        const workDiv = dv.el("div", "", {
            container: resultsDiv,
            attr: { style: "margin-bottom: 15px; padding: 12px; background: var(--background-secondary); border-radius: 8px;" }
        });
        
        const authorDisplay = page["author-display"] || (page.authors ? (page.authors.length > 1 ? page.authors[0] + " et al." : page.authors[0]) : "Unknown");
        
        dv.el("div", "", {
            container: workDiv
        }).innerHTML = `<strong><a href="${page.file.path}">${page.title || page.file.name}</a></strong>`;
        
        dv.el("div", `${authorDisplay} (${page.date || "No date"})`, {
            container: workDiv,
            attr: { style: "color: var(--text-muted); font-size: 0.9em; margin-top: 5px;" }
        });
    });
}

authorInput.addEventListener("input", () => {
    clearTimeout(authorInput.searchTimeout);
    authorInput.searchTimeout = setTimeout(performAdvancedSearch, 500);
});

categoryInput.addEventListener("input", () => {
    clearTimeout(categoryInput.searchTimeout);
    categoryInput.searchTimeout = setTimeout(performAdvancedSearch, 500);
});

yearInput.addEventListener("input", () => {
    clearTimeout(yearInput.searchTimeout);
    yearInput.searchTimeout = setTimeout(performAdvancedSearch, 500);
});

performAdvancedSearch();
```

---

## 📚 Reading List Manager

### Current Reading List

```dataviewjs
const pages = dv.pages('"Literature_Notes"')
    .where(p => p["reading-status"] === "reading")
    .sort(p => p.date, 'desc');

if (pages.length === 0) {
    dv.paragraph("*No works currently being read. Update the 'reading-status' field in your notes to 'reading'.*");
} else {
    dv.table(
        ["Title", "Authors", "Date", "Progress"],
        pages.map(p => [
            p.file.link,
            p["author-display"] || (p.authors ? (p.authors.length > 1 ? p.authors[0] + " et al." : p.authors[0]) : "Unknown"),
            p.date || "N/A",
            "📖 Reading"
        ])
    );
}
```

### To Read

```dataviewjs
const pages = dv.pages('"Literature_Notes"')
    .where(p => p["reading-status"] === "unread")
    .sort(p => p.date, 'desc')
    .limit(10);

if (pages.length === 0) {
    dv.paragraph("*Great! No unread works in the queue.*");
} else {
    dv.paragraph(`Showing 10 of ${dv.pages('"Literature_Notes"').where(p => p["reading-status"] === "unread").length} unread works`);
    
    dv.table(
        ["Title", "Authors", "Date"],
        pages.map(p => [
            p.file.link,
            p["author-display"] || (p.authors ? (p.authors.length > 1 ? p.authors[0] + " et al." : p.authors[0]) : "Unknown"),
            p.date || "N/A"
        ])
    );
}
```

---

## 🔗 Quick Links & Tips

### Navigation
- Jump to [[#📋 All Literature Notes|All Notes]]
- Jump to [[#👤 Browse by Author|Browse by Author]]
- Jump to [[#🏷️ Browse by Categories & Concepts|Browse by Document Category]]
- Jump to [[#🔍 Browse Annotations by Category|Browse Annotations by Category]]
- Jump to [[#📖 Extract Quotes & Highlights|Extract Quotes]]

### Usage Tips

**💡 Getting Started:**
1. Import your Zotero references using Zotero Integration plugin
2. Add general categories in the frontmatter field `general-categories: [methodology, empirical-study]`
3. Add specific categories to each annotation using the `**Categories**::` field
4. Use the search functions above to explore your library by document or annotation categories

**💡 Best Practices:**
- Update `reading-status` field as you progress through works
- Add general categories at the document level for broad categorization
- Add specific categories to each annotation for granular searchability
- Use consistent tag naming (lowercase, hyphens for spaces)
- Review the Categories & Concepts Index regularly to maintain consistency

**💡 Pro Tips:**
- Use "Browse Annotations by Category" to find specific insights across all your readings
- Yellow highlights from Zotero are now included as "Highlight" callouts
- Use the Advanced Search to find works at the intersection of topics
- The Category Cloud shows your most-used general concepts at a glance
- Each annotation can have its own categories for precise filtering

---

*Last updated: October 2025*