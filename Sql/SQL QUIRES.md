Query a list of **CITY** names from **STATION** for cities that have an even **ID** number. Print the results in any order, but exclude duplicates from the answer.

SELECT DISTINCT CITY
FROM STATION
WHERE MOD(ID,2) = 0;


---
Find the difference between the total number of **CITY** entries in the table and the number of distinct **CITY** entries in the table.

SELECT 
COUNT(CITY)-COUNT(DISTINCT CITY)
FROM STATION;

---
Query the two cities in **STATION** with the shortest and longest _CITY_ names, as well as their respective lengths (i.e.: number of characters in the name). If there is more than one smallest or largest city, choose the one that comes first when ordered alphabetically.


SELECT CITY, LEN(CITY) AS name_length
FROM (
    SELECT TOP 1 CITY
    FROM STATION
    ORDER BY LEN(CITY) ASC, CITY ASC
) AS shortest
UNION
SELECT CITY, LEN(CITY) AS name_length
FROM (
    SELECT TOP 1 CITY
    FROM STATION
    ORDER BY LEN(CITY) DESC, CITY ASC
) AS longest;

----
https://www.hackerrank.com/challenges/weather-observation-station-6/problem?isFullScreen=true