---
title: "Notion: calcular cuántos días hay entre dos fechas"
published: true
disqus_identifier: 8fa19c0b-ee2b-4e29-aa41-acc25b5cc7bb
comments: true
excerpt_separator: <!-- more -->
categories:
- Notion
---

Tengo una tabla en Notion con un campo opcional de tipo `Date` 
y quiero calcular la cantidad de días que han pasado desde esta 
fecha hasta hoy. 

<!-- more -->

Para esto creo un nuevo campo de tipo `Formula` con el siguiente contenido:

```
if(
    empty(
        prop("Fecha Aplicación")
    ), 
    0, 
    dateBetween(
        now(), 
        prop("Fecha Aplicación"), 
        "days"
    )
)
```

donde `Fecha Aplicación` es el nombre de mi campo de tipo `Date`.