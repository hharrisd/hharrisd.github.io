---
title: "Second"
date: 2022-09-27T19:23:32-05:00
draft: true
cover:
    image: img/bells.png
    alt: 'Esta es una imagen divertida'
    caption: 'Las aventuras de Bells'
tags: ["python", "coding"]
categories: ["tech"]
ShowToc: true
---

## Este es mi segundo post de prueba

### Tiene un parrafo

Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.

### Y otro más:

It is a long established fact that a reader will be distracted by the readable content of a page when looking at its layout. The point of using Lorem Ipsum is that it has a more-or-less normal distribution of letters, as opposed to using 'Content here, content here', making it look like readable English. Many desktop publishing packages and web page editors now use Lorem Ipsum as their default model text, and a search for 'lorem ipsum' will uncover many web sites still in their infancy. Various versions have evolved over the years, sometimes by accident, sometimes on purpose (injected humour and the like).

### Ahora algo de código

```python
def _write_row_items_for_table(self, data, table):
    with DBManager(**self.db_credentials['destiny']).db_cursor() as self.db_cursor:
        headers = self.__get_headers_for_table(table)
    self._create_or_update_items(table, headers, data)
    self._retry_failed_items(table, headers)
```
