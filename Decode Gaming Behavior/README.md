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

![Screenshot (116)](https://github.com/sanjanapaluri/Mentorness_Internship/assets/127730680/d8a27200-ec60-49de-9d9a-8568deaa828a)

#### Question 2: Find `Level1_code`wise average `Kill_Count` where `lives_earned` is 2, and at least 3 stages are crossed.

```sql
SELECT b.L1_code,AVG(a.kill_count) AS avg_kill_count

FROM dbo.level_details2 a

JOIN dbo.player_details b on a.P_ID = B.P_ID

WHERE lives_earned = 2 AND Stages_crossed >=3

GROUP BY b.L1_Code

```

![Screenshot (117)](https://github.com/sanjanapaluri/Mentorness_Internship/assets/127730680/143384bc-59ad-48df-bb96-9a212d1222f8)


#### Question 3: Find the total number of stages crossed at each difficulty level for Level 2 with players using `zm_series` devices. Arrange the result in decreasing order of the total number of stages crossed.

```sql
SELECT a.Difficulty ,SUM(a.Stages_crossed) AS tlt_cross

FROM dbo.level_details2 a

JOIN dbo.player_details b on a.P_ID = B.P_ID

WHERE Level = 2 AND Dev_ID = 'zm_015'

GROUP BY a.Difficulty

ORDER BY tlt_cross DESC

```

![Screenshot (118)](https://github.com/sanjanapaluri/Mentorness_Internship/assets/127730680/3bedc847-3f57-419f-9cc5-2f4fdd664f72)

#### Question 4: Extract `P_ID` and the total number of unique dates for those players who have played games on multiple days.

```sql
SELECT P_ID,COUNT(DISTINCT CONVERT(DATE,start_datetime)) AS uniqe_date

FROM dbo.level_details2

GROUP BY P_ID

HAVING COUNT(DISTINCT CONVERT(DATE,start_datetime)) >1

```

![Screenshot (119)](https://github.com/sanjanapaluri/Mentorness_Internship/assets/127730680/97e191a9-fe83-487b-ace3-aa8549a6183b)


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

![Screenshot (120)](https://github.com/sanjanapaluri/Mentorness_Internship/assets/127730680/759cb88c-c9d7-4918-b9a1-bd0cb262a50c)

#### Question 6: Find `Level` and its corresponding `Level_code`wise sum of lives earned, excluding Level 0. Arrange in ascending order of level.

```sql
SELECT a.Level,b.L2_Code,SUM(a.Lives_Earned) AS tlt_earned

FROM dbo.level_details2 a

JOIN dbo.player_details b on a.P_ID = B.P_ID

WHERE Level <> 0

GROUP BY a.Level,b.L2_Code

ORDER BY a.Level DESC

```

![Screenshot (121)](https://github.com/sanjanapaluri/Mentorness_Internship/assets/127730680/e1527caa-ab5f-4ea6-9776-507ef5b71de4)

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
SELECT Dev_ID,MIN(Start_datetime) AS first_loign

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

       SUM(kill_count) OVER (PARTITION BY P_ID ORDER BY Start_datetime) AS cumulative_kill_counts
FROM (

    SELECT P_ID, CONVERT(date, Start_datetime) AS start_date, kill_count

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

AND CONVERT(date, ld.Start_datetime) >= CONVERT(date, ld_inner.Start_datetime)
                            
GROUP BY

    ld.P_ID, CONVERT(date, ld.Start_datetime), ld.kill_count
    
ORDER BY

    ld.P_ID, CONVERT(date, ld.TimeStamp);

```

### Question 12:Find the cumulative sum of stages crossed over `start_datetime` for each `P_ID`, excluding the most recent `start_datetime`.

```sql
WITH RankedLevels AS (

    SELECT P_ID, Start_datetime, stages_crossed,

           ROW_NUMBER() OVER (PARTITION BY P_ID ORDER BY Start_datetime DESC) AS rn

    FROM Internship.dbo.level_details2
)

SELECT P_ID, Start_datetime, stages_crossed,

       SUM(stages_crossed) OVER (PARTITION BY P_ID ORDER BY Start_datetime ASC ROWS BETWEEN

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

```sql
SELECT l.P_ID,l.Score

FROM Internship.dbo.level_details2 l

WHERE l.Score > 0.5 *(

                     SELECT AVG(Score) AS avg_score

					 FROM Internship.dbo.level_details2

					 WHERE P_ID = l.P_ID

					 GROUP BY P_ID );

```

### Question 15:Create a stored procedure to find the top `n` `headshots_count` based on each `Dev_ID` and rank them in increasing order using `Row_Number`. Display the difficulty as well.

```sql
CREATE PROCEDURE GetTopHeadshots

    @n INT

AS

BEGIN

    SET NOCOUNT ON;

    WITH TopHeadshots AS (

        SELECT Dev_ID, difficulty, headshots_count,

               ROW_NUMBER() OVER (PARTITION BY Dev_ID ORDER BY headshots_count DESC) AS rank

        FROM Internship.dbo.level_details2

    )

    SELECT Dev_ID, difficulty, headshots_count

    FROM TopHeadshots

    WHERE rank <= @n

    ORDER BY Dev_ID, rank;

END;


EXEC GetTopHeadshots @n = 3;

```

![Screenshot (114)](https://github.com/sanjanapaluri/SQL_Projects/assets/127730680/2b4ec76b-66c9-4e30-87ce-b165acf29aa3)
