## sampling query: sampling 1 timestamp per day most closed to designated day time 

```sql
-- base_daily_points : generate criteria timstamptz timeline
WITH base_daily_points AS (
    SELECT generate_series(
        '2025-05-01 00:00:00'::timestamp AT TIME ZONE 'Asia/Seoul',
        '2025-06-01 00:00:00'::timestamp AT TIME ZONE 'Asia/Seoul',
        INTERVAL '1 day'
    ) AS target_utc_tstz
),
ranked_data AS (
    SELECT
        bdp.target_utc_tstz,
        ett.*,
        ROW_NUMBER() OVER (
            PARTITION BY bdp.target_utc_tstz
            ORDER BY ett.timeline DESC
        ) AS rn
    FROM
        base_daily_points bdp
    JOIN
        tb_exam_timestamp_timeline ett
    ON
        ett.timeline > (bdp.target_utc_tstz - make_interval(days => 1))
        AND
        ett.timeline <= bdp.target_utc_tstz
    ),
selected_daily_points AS (
    SELECT
        (target_utc_tstz AT TIME ZONE 'Asia/Seoul')::date AS "date",
        timeline
    FROM ranked_data
    WHERE
        rn = 1
)
SELECT * FROM selected_daily_points sdp;
```