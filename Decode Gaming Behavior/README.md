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
