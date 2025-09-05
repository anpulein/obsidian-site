---
title: Copy - массовая вставка в PostgreSQL
aliases:
  - Copy - массовая вставка в PostgreSQL
linter-yaml-title-alias: Copy - массовая вставка в PostgreSQL
date created: Friday, September 5th 2025, 7:37:41 am
date modified: Friday, September 5th 2025, 7:56:22 am
tags:
  - type/permanent
  - tech/csharp
  - status/completed
  - tech/postgresql
  - media/telegram
---
## 🏷️ Tags

 #media/telegram #type/permanent #tech/csharp #tech/postgresql  #status/completed 

---

![[photo_701090458_363 - 20250723091539589.jpg]]

> [!note] **Эта фича показала мне, насколько PostgreSQL крут.**
> В одном проекте мне нужно было за одну операцию вставить 5 тысяч записей. Хранилищем был Postgres, и, естественно, всё должно было работать быстро. После небольшого ресёрча я наткнулся на команду `COPY`. Эта команда позволяет выполнять массовую вставку данных напрямую в базу.
> 
> - Метод `BINARY COPY` отправляет данные в бинарном формате, который `Npgsql` декодирует, и после вызова` Complete()` данные сохраняются в БД.
>   
>   Так как при этом сохраняются типы данных без конвертации, лучше использовать `NpgsqlDbType` — он позволяет явно указать нужный тип без неоднозначностей.
