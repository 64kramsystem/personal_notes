## Table of contents

- [Built-in functions](#built-in-functions)
  - [String functions](#string-functions)
    - [Character conversions](#character-conversions)
- [Window functions](#window-functions)

## Built-in functions

### String functions

```sql
TRIM(<expr>)                                             # Strips only spaces by default!
TRIM([BOTH|LEADING|TRAILING] <str> FROM <expr>)          # Strip characters; usable for newlines
```

#### Character conversions

Convert from code point to character and viceversa:

```sql
SELECT CHAR(0xF09F91B8 USING utf8mb4);

-- +--------------------------------+
-- | CHAR(0xF09F91B8 USING utf8mb4) |
-- +--------------------------------+
-- | 👸                             |
-- +--------------------------------+

SELECT HEX(ORD('👸'));

-- +---------------+
-- | HEX(ORD('?')) |
-- +---------------+
-- | F09F91B8      |
-- +---------------+
```

## Window functions

Basic window functions example: compute the difference of a week of sales compared to the previous year.

`LAG(_expression_{, _N_})` is the value of _expression_ applied to the row preceding `N` positions (`N` defaults to 1).

It's crucial to keep in mind the [window functions processing order](https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html):

> Query result rows are determined from the FROM clause, after WHERE, GROUP BY, and HAVING processing, and windowing execution occurs before ORDER BY, LIMIT, and SELECT DISTINCT.

as a consequence, it's not possible to filter out via `HAVING` (with this given structure) rows whose expression returns `NULL`.

In this (simplistic) example, can't use `YEAR()`/`WEEK()` separately, due to the week 0 being part of the previous year's last week.

```sql
SELECT
  YEARWEEK(created_at) `sales_year_week`,
  MIN(DATE(created_at)) `sales_week_start`,
  SUM(quantity) `tot_quantity`,
  SUM(quantity) - LAG(SUM(quantity), 52) OVER (ORDER BY created_at) `diff_quantity_52_weeks`
FROM sales
WHERE DATE(created_at) BETWEEN CURDATE() - INTERVAL (DAYOFWEEK(CURDATE()) - 1 ) DAY - INTERVAL 1 WEEK - INTERVAL 52 WEEK - INTERVAL 10 WEEK
                           AND CURDATE() - INTERVAL (DAYOFWEEK(CURDATE()) - 1 ) DAY - INTERVAL 1 WEEK
GROUP BY sales_year_week
ORDER BY sales_year_week;
```

When there are multiple equal windows, use named windows, otherwise, each window is evaluated separately:

```sql
SELECT
  id,
  LAG(id) OVER w                       `reference_sale_id`,
  updated_at,
  LAG(updated_at) OVER w               `reference_sale_updated_at`,
  updated_at >= LAG(updated_at) OVER w `in_order`
FROM sales
WHERE special_customer
WINDOW w AS (ORDER BY id DESC)
```
