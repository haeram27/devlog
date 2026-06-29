## sampling query: sampling 1 timestamp per day most closed to designated day time 

`tb_exam_tymestamp_timeline`은 `tymestamp::timestamp` 컬럼을 가진 data table이다.

```sql
-- base_daily_points : generate criteria timstamptz tymestamp
WITH base_daily_points AS (
    SELECT generate_series(
        -- 2025-05-01 09:00:00.000+0000
        '2025-05-01 00:00:00'::timestamp AT TIME ZONE 'Asia/Seoul',
        -- 2025-06-01 09:00:00.000+0000
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
            ORDER BY ett.tymestamp DESC
        ) AS rn
    FROM
        base_daily_points bdp
    JOIN
        tb_exam_tymestamp_timeline ett
    ON
        ett.tymestamp > (bdp.target_utc_tstz - make_interval(days => 1))
        AND
        ett.tymestamp <= bdp.target_utc_tstz
),
selected_daily_points AS (
    SELECT
        (target_utc_tstz AT TIME ZONE 'Asia/Seoul')::date AS "date",
        tymestamp
    FROM ranked_data
    WHERE
        rn = 1
)
SELECT * FROM selected_daily_points sdp;
```
