---
cssclass: literature-note
type: "{{itemType}}"
citekey: "{{citekey}}"
title: "{{title}}"
publication: "{{publicationTitle}}"
date: {% if date %}{{date | format("YYYY-MM-DD")}}{% endif %}
archive: "{{archive}}"
archive-location: "{{archiveLocation}}"
authors: [{% for type, creators in creators | groupby("creatorType") %}{% for creator in creators %}{% if creator.name %}"{{creator.name}}"{% else %}"{{creator.lastName}}, {{creator.firstName}}"{% endif %}{% if not loop.last %}, {% endif %}{% endfor %}{% endfor %}]
first-author: "{% for type, creators in creators | groupby("creatorType") %}{% for creator in creators %}{% if loop.first %}{% if creator.name %}{{creator.name}}{% else %}{{creator.lastName}}, {{creator.firstName}}{% endif %}{% endif %}{% endfor %}{% endfor %}"
author-display: "{% for type, creators in creators | groupby("creatorType") %}{% for creator in creators %}{% if loop.first %}{% if creator.name %}{{creator.name}}{% else %}{{creator.lastName}}{% endif %}{% if loop.length > 1 %} et al.{% endif %}{% endif %}{% endfor %}{% endfor %}"
tags-zotero: [{% for t in tags %}"{{t.tag}}"{% if not loop.last %}, {% endif %}{% endfor %}]
reading-status: unread
general-categories:  # categorías generales de la nota

---

[online]({{uri}}) [local]({{desktopURI}})
{% for attachment in attachments | filterby("path", "endswith", ".pdf") %}
[pdf](file://{{attachment.path | replace(" ", "%20")}})
{% endfor %}

### Authors / Creators
{% for type, creators in creators | groupby("creatorType") %}
{% for creator in creators %}
- **{{type | replace("interviewee", "Author") | replace("director","Author") | replace("presenter","Author") | replace("podcaster","Author") | replace("programmer","Author") | replace("cartographer","Author") | replace("inventor","Author") | replace("sponsor","Author") | replace("performer","Author") | replace("artist","Author")}}**: {% if creator.name %}{{creator.name}}{% else %}{{creator.lastName}}, {{creator.firstName}}{% endif %}
{% endfor %}
{% endfor %}

### Tags Zotero
{% for t in tags %}
- {{t.tag}}
{% endfor %}

---

## Index
start-date:: {% if date %}{{date | format("YYYY-MM-DD")}}{% endif %}
end-date::
page-no:: {% for annotation in annotations %}{% if loop.first %}{{annotation.pageLabel}}{% endif %}{% endfor %}

---

## Notes / Highlights Zotero

{% macro calloutHeader(color) -%}
{% if color == "#ffd400" %}🟡 Highlight
{% elif color == "#ff6666" %}🔴 Important
{% elif color == "#5fb236" %}🟢 Reference
{% elif color == "#2ea8e5" %}🔵 Question
{% elif color == "#a28ae5" %}🟣 Definition
{% else %}Note
{% endif %}
{% endmacro %}

{% persist "annotations" %}
{% set newAnnotations = annotations | filterby("date", "dateafter", lastImportDate) %}
{% if newAnnotations.length > 0 %}
### Imported on {{importDate | format("YYYY-MM-DD h:mm a")}}

{% for annotation in newAnnotations %}
>[!quote{% if annotation.color %}|{{annotation.color}}{% endif %}] 
>{{calloutHeader(annotation.color)}}
>{% if annotation.imageRelativePath %}![[{{annotation.imageRelativePath}}]]{% endif %}
>{% if annotation.annotatedText %} 
>{{annotation.annotatedText}} [(p. {{annotation.pageLabel}})](zotero://open-pdf/library/items/{{annotation.attachment.itemKey}}?page={{annotation.pageLabel}}&annotation={{annotation.id}})
>{% endif %}
>**Categories:** _add here your categories using tags_
>**💬 Comment:** {{annotation.comment}}
{% endfor %}
{% endif %}
{% endpersist %}


---

### Color Legend

Use these colors consistently in Zotero to categorize your highlights:

| Color                     | Type           | Purpose                                 |
| :------------------------ | :------------- | :-------------------------------------- |
| 🟡 **Yellow** (`#ffd400`) | **Highlight**  | General important passages              |
| 🔴 **Red** (`#ff6666`)    | **Important**  | Critical concepts or key arguments      |
| 🟢 **Green** (`#5fb236`)  | **Reference**  | Citations or references to follow up    |
| 🔵 **Blue** (`#2ea8e5`)   | **Question**   | Raises questions or needs clarification |
| 🟣 **Purple** (`#a28ae5`) | **Definition** | Definitions or theoretical frameworks   |

---

## Own Observations
_Write here your own analytical or interpretive observations._

## Thoughts
_Use this section for theoretical connections or emerging ideas._

## Questions
_List conceptual, methodological, or interpretative questions._

## Other Related Texts
- [[ ]]
- [[ ]]
