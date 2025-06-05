# find most closed timestamp with matching reqeusted timeline

```sql
WITH base_daily_points AS (
  SELECT generate_series(
    '2025-03-25 00:00:00'::timestamp AT TIME ZONE 'Asia/Seoul',
    '2025-06-05 00:00:00'::timestamp AT TIME ZONE 'Asia/Seoul',
    INTERVAL '1 day'
  ) AS target_utc
),
ranked_data AS (
  SELECT
    c.target_utc,
    t.*,
    ROW_NUMBER() OVER (
      PARTITION BY c.target_utc
      ORDER BY t.measuring_time DESC
    ) AS rn
  FROM
    base_daily_points c
  JOIN
    tb_meter_product_license_usage_hourly_timeline t
  ON
    t.measuring_time > (c.target_utc - make_interval(days => 1))
    AND
    t.measuring_time <= c.target_utc
),
selected_daily_points AS (
    SELECT
      (target_utc AT TIME ZONE 'Asia/Seoul')::date AS "date",
      measuring_time
    FROM ranked_data
    WHERE
      rn = 1
)
SELECT *
FROM tb_meter_product_license_usage_hourly_timeline t
WHERE t.measuring_time IN (SELECT measuring_time FROM selected_daily_points);
```
