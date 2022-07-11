# Captstone-Project

--Making a copy of the data
SELECT *
INTO final_table
FROM Apr_2022_tripdata

--Altering the table to match column data types
ALTER TABLE 
	final_table
ALTER COLUMN 
	start_station_id nvarchar(max)

ALTER TABLE final_table
ALTER COLUMN end_station_id nvarchar(max)

--Continued combining all into one master
INSERT INTO final_table
SELECT *
FROM Aug_2021_tripdata

INSERT INTO final_table
SELECT *
FROM Dec_2021_tripdata

INSERT INTO final_table
SELECT *
FROM Feb_2022_tripdata

INSERT INTO final_table
SELECT *
FROM Jan_2022_tripdata

INSERT INTO final_table
SELECT *
FROM July_2021_tripdata

INSERT INTO final_table
SELECT *
FROM June_2021_tripdata

INSERT INTO final_table
SELECT *
FROM Mar_2022_tripdata

INSERT INTO final_table
SELECT *
FROM May_2022_tripdata

INSERT INTO final_table
SELECT *
FROM Nov_2021_tripdata

INSERT INTO final_table
SELECT *
FROM Oct_2021_tripdata

INSERT INTO final_table
SELECT *
FROM Sept_2021_tripdata


--Viewing the data without duplicate ride_id, user under 60 sec, user over 24 hours
SELECT *
INTO temp_final
FROM
	(SELECT
	DISTINCT ride_id,
	member_casual,
	rideable_type,
	start_station_name,
	end_station_name,
	started_at,
	ended_at,
	DATEDIFF(minute, started_at, ended_at) AS Duration
FROM
	final_table
WHERE
	(start_station_name IS NOT NULL 
	AND end_station_name IS NOT NULL)
	AND
	(start_station_name != 'WATSON TESTING - DIVVY'
	AND end_station_name != 'WATSON TESTING - DIVVY')
	AND
	(ended_at - started_at > 0)
	AND
	DATEDIFF(second, started_at, ended_at) < 86400
	AND 
	DATEDIFF(second, started_at, ended_at) > 59
)
 as TEMP


--Query to pull descriptive stats table
SELECT 
	DISTINCT member_casual,
	PERCENTILE_CONT(0)
		WITHIN GROUP(ORDER BY duration) OVER (PARTITION BY member_casual) AS min_drtn,
	PERCENTILE_CONT(0.25)
		WITHIN GROUP(ORDER BY duration) OVER (PARTITION BY member_casual) AS [25%],
	PERCENTILE_CONT(0.5)
		WITHIN GROUP(ORDER BY duration) OVER (PARTITION BY member_casual) AS median,
	AVG(DATEDIFF(MINUTE, started_at, ended_at)) OVER
		(PARTITION BY member_casual) AS mean_drtn,
	PERCENTILE_CONT(0.75)
		WITHIN GROUP(ORDER BY duration) OVER (PARTITION BY member_casual) AS [75%],
	PERCENTILE_CONT(1)
		WITHIN GROUP(ORDER BY duration) OVER (PARTITION BY member_casual) AS max_drtn
FROM temp_final


-- Number of rides throughout the week member vs casual
SELECT
	DATENAME(dw, started_at) AS day_of_week,
	member_casual,
	COUNT(*) AS no_of_rides
FROM 
	temp_final
GROUP BY
	DATENAME(dw, started_at),
	member_casual
ORDER BY
	member_casual,
	COUNT(*) DESC,
	DATENAME(dw, started_at)


--Temp table with rides per type
SELECT * INTO rides_per_type FROM (
SELECT
		DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, started_at), 0) AS year_month,
		rideable_type,
		member_casual,
		COUNT(*) AS no_of_rides
	FROM
		temp_final
	GROUP BY DATEADD(MONTH, DATEDIFF(MONTH, 0, started_at), 0),
		rideable_type,
		member_casual
) 
AS TEMP
ORDER BY DATEADD(MONTH, DATEDIFF(MONTH, 0, year_month), 0),
		rideable_type,
		member_casual

--percent per type
SELECT 
	year_month,
	member_casual,
	rideable_type,
	no_of_rides,
	SUM(no_of_rides) OVER
		(PARTITION BY year_month) AS rides_per_month,
	ROUND(CAST(no_of_rides AS float)/SUM(no_of_rides) OVER(PARTITION BY year_month) *100, 2) AS prcnt
FROM 
	rides_per_type
ORDER BY 
	year_month, 
	member_casual



--viewing the temp table
SELECT *
FROM rides_per_type
ORDER BY DATEADD(MONTH, DATEDIFF(MONTH, 0,year_month), 0),
		rideable_type,
		member_casual

