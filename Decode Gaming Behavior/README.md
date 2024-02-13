# Decode Gaming Behavior

In this internship, I had worked with a dataset related to a game. The dataset includes two tables: `Player Details` and `Level Details`. Below is a brief description of the dataset.

# Dataset Description:
## Player Details Table:

1. `P_ID`: Player ID
2. `PName`: Player Name
3. `L1_status`: Level 1 Status
4. `L2_status`: Level 2 Status
5. `L1_code`: Systemgenerated Level 1 Code
6. `L2_code`: Systemgenerated Level 2 Code

## Level Details Table:

1. `P_ID`: Player ID
2. `Dev_ID`: Device ID
3. `start_time`: Start Time
4. `stages_crossed`: Stages Crossed
5. `level`: Game Level
6. `difficulty`: Difficulty Level
7. `kill_count`: Kill Count
8. `headshots_count`: Headshots Count
9. `score`: Player Score
10. `lives_earned`: Extra Lives Earned

------------------------------------------------------------------------------------------------------------------------------------------

#### Question 1: Extract `P_ID`, `Dev_ID`, `PName`, and `Difficulty_level` of all players at Level 0.

```sql
SELECT a.P_ID, a.Dev_ID, b.PName, a.Difficulty

FROM dbo.level_details2 a 

INNER JOIN dbo.player_details b on a.P_ID = B.P_ID

WHERE Level = 0

```

#### Question 2: Find `Level1_code`wise average `Kill_Count` where `lives_earned` is 2, and at least 3 stages are crossed.

```sql
SELECT b.L1_code,AVG(a.kill_count) AS avg_kill_count

FROM dbo.level_details2 a

JOIN dbo.player_details b on a.P_ID = B.P_ID

WHERE lives_earned = 2 AND Stages_crossed >=3

GROUP BY b.L1_Code

```

#### Question 3: Find the total number of stages crossed at each difficulty level for Level 2 with players using `zm_series` devices. Arrange the result in decreasing order of the total number of stages crossed.

```sql
SELECT a.Difficulty ,SUM(a.Stages_crossed) AS tlt_cross

FROM dbo.level_details2 a

JOIN dbo.player_details b on a.P_ID = B.P_ID

WHERE Level = 2 AND Dev_ID = 'zm_015'

GROUP BY a.Difficulty

ORDER BY tlt_cross DESC

```

#### Question 4: Extract `P_ID` and the total number of unique dates for those players who have played games on multiple days.

```sql
SELECT P_ID,COUNT(DISTINCT CONVERT(DATE,TimeStamp)) AS uniqe_date

FROM dbo.level_details2

GROUP BY P_ID

HAVING COUNT(DISTINCT CONVERT(DATE,TimeStamp)) >1

```

#### Question 5: Find `P_ID` and levelwise sum of `kill_counts` where `kill_count` is greater than the average kill count for Medium difficulty.

```sql
SELECT P_ID,Level,SUM(Kill_Count) AS tlt_count

FROM dbo.level_details2

WHERE Difficulty = 'Medium'

GROUP BY P_ID, Level

HAVING SUM(Kill_Count) > (SELECT AVG(Kill_Count)

                            FROM dbo.level_details2

                             WHERE Difficulty = 'Medium')

```

#### Question 6: Find `Level` and its corresponding `Level_code`wise sum of lives earned, excluding Level 0. Arrange in ascending order of level.

```sql
SELECT a.Level,b.L2_Code,SUM(a.Lives_Earned) AS tlt_earned

FROM dbo.level_details2 a

JOIN dbo.player_details b on a.P_ID = B.P_ID

WHERE Level <> 0

GROUP BY a.Level,b.L2_Code

ORDER BY a.Level DESC

```

#### Question 7: Find the top 3 scores based on each `Dev_ID` and rank them in increasing order using `Row_Number`. Display the difficulty as well.

```sql
SELECT Dev_ID,Score,Difficulty

FROM ( SELECT Dev_ID,Score,Difficulty ,

ROW_NUMBER() OVER(PARTITION BY Dev_ID ORDER BY Score DESC) AS r

FROM Internship.dbo.level_details2) AS e

WHERE r <=3

```
                                 OR 

```sql
WITH RankedScores AS (

    SELECT ld.Dev_ID, ld.score, ld.difficulty,

           ROW_NUMBER() OVER (PARTITION BY ld.Dev_ID ORDER BY ld.score DESC) AS rank

    FROM Internship.dbo.level_details2 ld
)
SELECT Dev_ID, score, difficulty

FROM RankedScores

WHERE rank <= 3;

```

#### Question 8: Find the `first_login` datetime for each device ID.

```sql
SELECT Dev_ID,MIN(TimeStamp) AS first_loign

FROM Internship.dbo.level_details2

GROUP BY Dev_ID

```

#### Question 9: Find the top 5 scores based on each difficulty level and rank them in increasing orderusing `Rank`. Display `Dev_ID` as well.

```sql
WITH RankedScores AS (

    SELECT Dev_ID, difficulty, score,

           RANK() OVER (PARTITION BY difficulty ORDER BY score ASC) AS score_rank

    FROM Internship.dbo.level_details2)

SELECT Dev_ID, difficulty, score, score_rank

FROM RankedScores

WHERE score_rank <= 5;

```
#### Question 10:Find the device ID that is first logged in (based on `start_datetime`) for each player (`P_ID`). Output should contain player ID, device ID, and first login datetime.

```sql
SELECT ld.P_ID, ld.Dev_ID, ld.start_datetime AS first_login_datetime

FROM Level_Details ld

INNER JOIN (

    SELECT P_ID, MIN(start_datetime) AS min_start_datetime

    FROM Level_Details

    GROUP BY P_ID

) AS min_login ON ld.P_ID = min_login.P_ID AND ld.start_datetime = min_login.min_start_datetime;

```
#### Question 11:For each player and date, determine how many `kill_counts` were played by the player so far.
a) Using window functions

b) Without window functions

a) Using window functions

```sql
SELECT P_ID, start_date, kill_count,

       SUM(kill_count) OVER (PARTITION BY P_ID ORDER BY start_date) AS cumulative_kill_counts
FROM (

    SELECT P_ID, CONVERT(date, start_time) AS start_date, kill_count

    FROM Level_Details

) AS subquery;

```

b) Without window functions

```sql
SELECT
    ld.P_ID,
    
    CONVERT(date, ld.TimeStamp) AS play_date,
    
    ld.kill_count,
    
    SUM(ld_inner.kill_count) AS cumulative_kill_counts
    
FROM Internship.dbo.level_details2 ld

JOIN Internship.dbo.level_details2 ld_inner

     ON ld.P_ID = ld_inner.P_ID

AND CONVERT(date, ld.TimeStamp) >= CONVERT(date, ld_inner.TimeStamp)
                            
GROUP BY

    ld.P_ID, CONVERT(date, ld.TimeStamp), ld.kill_count
    
ORDER BY

    ld.P_ID, CONVERT(date, ld.TimeStamp);

```

### Question 12:Find the cumulative sum of stages crossed over `start_datetime` for each `P_ID`, excluding the most recent `start_datetime`.

```sql
WITH RankedLevels AS (

    SELECT P_ID, TimeStamp, stages_crossed,

           ROW_NUMBER() OVER (PARTITION BY P_ID ORDER BY TimeStamp DESC) AS rn

    FROM Internship.dbo.level_details2
)

SELECT P_ID, TimeStamp, stages_crossed,

       SUM(stages_crossed) OVER (PARTITION BY P_ID ORDER BY TimeStamp ASC ROWS BETWEEN

	   UNBOUNDED PRECEDING AND 1 PRECEDING) AS cumulative_stages_crossed

FROM RankedLevels

WHERE rn > 1;

```

### Question 13: Extract the top 3 highest sums of scores for each `Dev_ID` and the corresponding `P_ID`.

```sql
WITH RankedScores AS (

    SELECT P_ID, Dev_ID, SUM(score) AS total_score,

           ROW_NUMBER() OVER (PARTITION BY Dev_ID ORDER BY SUM(score) DESC) AS rank

    FROM Internship.dbo.level_details2

    GROUP BY P_ID, Dev_ID
)

SELECT P_ID, Dev_ID, total_score

FROM RankedScores

WHERE rank <= 3;

```

### Question 14:Find players who scored more than 50% of the average score, scored by the sum of scores for each `P_ID`.

