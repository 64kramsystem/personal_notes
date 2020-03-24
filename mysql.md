## Table of contents

- [Character conversions](#character-conversions)
- [Window functions](#window-functions)

## Character conversions

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

It's not possible to filter out (with this given structure) rows whose expression returns `NULL`, because `HAVING` applies before window functions.

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
