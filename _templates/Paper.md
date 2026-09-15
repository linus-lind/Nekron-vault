---
type: paper
authors: "<% tp.system.prompt('Authors') %>"
year: <% tp.system.prompt('Year') %>
url: "<% tp.system.prompt('URL / arXiv link') %>"
rating: 0             # 1-5, relevance to Nekron
status: read          # to-read | reading | read
created: <% tp.date.now("YYYY-MM-DD") %>
tags: [paper, research]
---

# <% tp.file.title %>

*<% tp.system.prompt('Full paper title') %>* (<% tp.system.prompt('Source, e.g. arXiv:xxxx.xxxxx') %>).
Domain: 

## Method
- 

## Procedure
```
```

## Relevance
- 


