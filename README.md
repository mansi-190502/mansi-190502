To get data of active user  

------------------------------------------------------------------------------
```
CREATE TABLE IF NOT EXISTS matomo.live_user_counts (

    name VARCHAR(255),

    time DATETIME,

    live_user_count INT,

    PRIMARY KEY (name, time)

);
 
 
-- Drop the event if it already exists

DROP EVENT IF EXISTS update_live_user_counts;

DELIMITER $$
 
-- Create the event to update live_user_counts table every minute

CREATE EVENT update_live_user_counts

ON SCHEDULE EVERY 1 MINUTE

DO

BEGIN

    INSERT INTO matomo.live_user_counts (name, time, live_user_count)

    SELECT

        s.name,

        DATE_FORMAT(lv.visit_last_action_time, '%Y-%m-%d %H:%i:00') AS time,

        COUNT(DISTINCT lv.idvisitor) AS live_user_count

    FROM

        matomo.log_visit lv

    JOIN 

        matomo.site s ON lv.idsite = s.idsite

    WHERE 

        lv.visit_last_action_time >= NOW() - INTERVAL 24 HOUR

        AND NOT EXISTS (

            SELECT 1

            FROM matomo.live_user_counts luc

            WHERE luc.name = s.name

              AND luc.time = DATE_FORMAT(lv.visit_last_action_time, '%Y-%m-%d %H:%i:00')

        )

    GROUP BY 

        time, s.name

    ORDER BY 

        time

    ON DUPLICATE KEY UPDATE

        live_user_count = VALUES(live_user_count);

END$$
 
DELIMITER ;
 
 
------------------------------------------------------------------------------

CREATE TABLE IF NOT EXISTS rum_report (

    date DATE,

    name VARCHAR(255),

    unique_user_count INT,

    total_visitor_count INT,

    bounce_rate DECIMAL(10, 2),

    average_visit_time_seconds DECIMAL(10, 2),

    average_visit_interaction DECIMAL(10, 2),

    PRIMARY KEY (date, name)

);
 
 
DELIMITER $$
 
-- Enable event scheduling if it's not already enabled

SET GLOBAL event_scheduler = ON$$
 
-- Create the event

CREATE EVENT IF NOT EXISTS daily_aggregation_event

ON SCHEDULE EVERY 1 DAY

STARTS TIMESTAMP('2024-07-12', '00:00:00')

DO

BEGIN

    -- Your SQL script here

    INSERT INTO rum_report(date, name, unique_user_count, total_visitor_count, bounce_rate, average_visit_time_seconds, average_visit_interaction)

    SELECT 

        DATE(visit_last_action_time) AS date,

        name,

        SUM(unique_user_count) AS unique_user_count,

        SUM(total_visitor_on_site) AS total_visitor_count,

        AVG(bounce_rate_percentage) AS bounce_rate,

        AVG(average_visit_time_seconds) AS average_visit_time_seconds,

        AVG(average_visit_interaction) AS average_visit_interaction

    FROM 

        matomo.RUM_view

    WHERE 

        visit_last_action_time >= NOW() - INTERVAL 1 DAY

    GROUP BY 

        date, name;
```
END$$
 
DELIMITER ;

 

<!---
mansi-190502/mansi-190502 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
