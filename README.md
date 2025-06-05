# ass3DATABASES


Lets start.
Here is our first query generated Claude according to task requirements.

EXPLAIN ANALYZE
SELECT Country, athlete_count
FROM (
    SELECT a.Country, 
           (SELECT COUNT(*) 
            FROM Athletes a2 
            JOIN Teams t2 ON a2.Country = t2.Country AND a2.Discipline = t2.Discipline 
            WHERE a2.Country = a.Country) as athlete_count
    FROM Athletes a
    WHERE EXISTS (SELECT 1 FROM Teams t WHERE t.Country = a.Country)
) counts
ORDER BY athlete_count DESC
LIMIT 3;
￼
Result is: 24751ms.

———
Lets rewrite our code and create index for optimization.

Of course it has the sense to add indexes, cause each lesson we get the information, how useful they are and how well they optimize query. Lets check it:   add CREATE INDEX idx_athletes_country_discipline ON Athletes(Country, Discipline); CREATE INDEX idx_teams_country_discipline ON Teams(Country, Discipline):


1. ￼

Result is 22711ms.
—————
After rewriting query:  EXPLAIN ANALYZE SELECT a.Country, COUNT(*) as athlete_count FROM Athletes a JOIN Teams t ON a.Country =t.Country AND a.Discipline = t.Discipline GROUP BY a.Country ORDER BY COUNT(*) DESC LIMIT 3;

1. ￼

Result is 47.2ms

———— What if reading all rows is cheaper than using the indexes? Lets proof this case:
DROP INDEX idx_athletes_covering ON Athletes;
DROP INDEX idx_teams_covering ON Teams;
1. ￼

The result is 19.1ms
————


The same code, but after FLUSH TABLES;
1. ￼

The result is 14.9ms
———————

Lets change join order. Since Teams table looks smaller, make it the driving table:
EXPLAIN ANALYZE
SELECT t.Country, COUNT(*) as athlete_count
FROM Teams t
JOIN Athletes a ON t.Country = a.Country AND t.Discipline = a.Discipline
GROUP BY t.Country
ORDER BY COUNT(*) DESC
LIMIT 3;
￼
Success. Time executable is out off. 

The result is 13.7ms Go next step:
———————   Pre-filtered Approach
EXPLAIN ANALYZE
SELECT a.Country, COUNT(*) as athlete_count
FROM Athletes a
WHERE a.Country IN (SELECT DISTINCT Country FROM Teams)
  AND (a.Country, a.Discipline) IN (
    SELECT Country, Discipline FROM Teams
  )
GROUP BY a.Country
ORDER BY COUNT(*) DESC
LIMIT 3;
￼

Distinct deletes all unnecessary rows that helps us to avoid repeating reading. Using where is a pre-filtered approach and help us to reduce time executable.

THE FINALLY RESULT IS 13.4 ms
