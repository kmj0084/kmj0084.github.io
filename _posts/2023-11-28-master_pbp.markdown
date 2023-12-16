---
layout: posts
title:  "Pitch-by-Pitch Fact Table"
date:   2023-11-28
categories: work
tags: sql
author_profile: true
author: Keegan Johnson
description: tSQL aggregations for a sabermetrics database primary fact table
---
# Building a sabermetrics pitch-by-pitch fact table

My goal with this short project is to build a pbp table using R and SQL for the 2021-2023 MLB regular seasons. This table can be subsequently used for all kinds of further analysis, and one could just as easily import a subset of total data to suite a specific need.

I used Bill Petti's *rbaseball* package, which scrapes data from MLB, Fangraphs, Baseball Reference, and other sources. This data can then easily be pulled into an RDBMS (I've used MSSQL) to denormalize for additional tables, or a reporting tool like Power BI directly for visualization and analysis.

#### Start by loading in R packages:
```
library('dplyr')
library('tidyverse')
library('lubridate')
library('plyr')
library('baseballr')
```
<br>
Next I'll create a date sequence that corresponds to the 2023 MLB Season and map the corresponding game_pk identifiers to each game date. The resulting dataframe is then mapped with the get_pbp_mlb function.  
#### 2023 Pitch-by-Pitch
```
x <- map_df(.x = seq.Date(as.Date('2023-03-30'),
                          as.Date('2023-10-02'),
                          'day'),
            ~get_game_pks_mlb(date = .x,
            level_ids = c(1)))

safe_mlb <- safely(get_pbp_mlb)

df.x <- map(.x = x |>
            filter(status.codedGameState == "F") |>
            pull(game_pk),
          ~safe_mlb(game_pk = .x)) |>
  map('result') |>
  bind_rows()

master_2023_pbp <- df.x
----

```
<br>
#### 2022 Pitch-by-Pitch
Repeating the same process for 2022
```
y <- map_df(.x = seq.Date(as.Date('2022-04-07'),
                          as.Date('2022-10-02'),
                          'day'),
            ~get_game_pks_mlb(date = .x,
            level_ids = c(1)))


safe_mlb <- safely(get_pbp_mlb)

df.y <- map(.x = y |>
            filter(status.codedGameState == "F") |>
            pull(game_pk),
          ~safe_mlb(game_pk = .x)) |>
  map('result') |>
  bind_rows()

master_2022_pbp <- df.y

# OPTIONAL: To save backup data images:
# save.image("data_output_saber_w2022.RData")
```

#### 2021 Pitch-by-Pitch
```
 z <- map_df(.x = seq.Date(as.Date('2021-04-01'),
                          as.Date('2021-10-03'),
                          'day'),
            ~get_game_pks_mlb(date = .x,
            level_ids = c(1)))

safe_mlb <- safely(get_pbp_mlb)

df.z <- map(.x = z |>
            filter(status.codedGameState == "F") |>
            pull(game_pk),
          ~safe_mlb(game_pk = .x)) |>
  map('result') |>
  bind_rows()

master_2021_pbp <- df.z
# save.image("data_output_saber_w2021.RData")
```

#### Choose fields
```
# Select fields:
master_2023_pbp_to_sql <- master_2023_pbp |>
  select (game_pk, game_date, startTime, endTime, isPitch, type,
	playId, pitchNumber, details.description, details.event,
	details.awayScore, details.homeScore, details.isScoringPlay,
	details.hasReview, details.code, details.ballColor, 
	details.isInPlay, details.isStrike, details.isBall, 
	details.call.code, details.call.description, 
	count.balls.start, count.strikes.start, count.outs.start,
	player.id, player.link, pitchData.strikeZoneTop, 
	pitchData.strikeZoneBottom, details.fromCatcher, 
	pitchData.coordinates.x, pitchData.coordinates.y, 
	hitData.trajectory, hitData.hardness, hitData.location,
	hitData.coordinates.coordX, hitData.coordinates.coordY,
	actionPlayId, details.eventType, details.runnerGoing,
	position.code, position.name, position.type,
	position.abbreviation, battingOrder, atBatIndex, result.type,
	result.event, result.eventType result.description, 
	result.rbi, result.awayScore, result.homeScore,
	about.atBatIndex, about.halfInning, about.inning,
	about.startTime, about.endTime, about.isComplete,
	about.isScoringPlay, about.hasReview, about.hasOut,
	about.captivatingIndex,	count.balls.end, count.strikes.end,
	count.outs.end, matchup.batter.id, matchup.batter.fullName,
	matchup.batter.link, matchup.batSide.code,
	matchup.batSide.description, matchup.pitcher.id,
	matchup.pitcher.fullName, matchup.pitcher.link,
	matchup.pitchHand.code, matchup.pitchHand.description,
	matchup.splits.batter, matchup.splits.pitcher, 
	matchup.splits.menOnBase, batted.ball.result,
	home_team, home_level_id, home_level_name,	
	home_parentOrg_id, home_parentOrg_name, home_league_id,
	home_league_name, away_team, away_level_id,
	away_level_name, away_parentOrg_id,	away_parentOrg_name,
	away_league_id, away_league_name, batting_team,
	fielding_team, last.pitch.of.ab, pfxId, 
	details.trailColor, details.type.code,
	details.type.description, pitchData.startSpeed, 
	pitchData.endSpeed, pitchData.zone,
	pitchData.typeConfidence, pitchData.plateTime,
	pitchData.extension, pitchData.coordinates.aY, 
	pitchData.coordinates.aZ, pitchData.coordinates.pfxX, 
	pitchData.coordinates.pfxZ, pitchData.coordinates.pX,
	pitchData.coordinates.pZ, pitchData.coordinates.vX0,
	pitchData.coordinates.vY0, pitchData.coordinates.vZ0, 
	pitchData.coordinates.x0, pitchData.coordinates.y0, 
	pitchData.coordinates.z0,  pitchData.coordinates.aX, 
	pitchData.breaks.breakAngle, pitchData.breaks.breakLength, 
	pitchData.breaks.breakY, pitchData.breaks.spinRate,
	pitchData.breaks.spinDirection, hitData.launchSpeed,
	hitData.launchAngle, hitData.totalDistance, injuryType,
	umpire.id, umpire.link, details.isOut,
	pitchData.breaks.breakVertical, 
	pitchData.breaks.breakVerticalInduced,
	pitchData.breaks.breakHorizontal, isBaseRunningPlay,
	details.disengagementNum, isSubstitution, 
	replacedPlayer.id, replacedPlayer.link,
	details.violation.type, details.violation.description,
	details.violation.player.id, 
	details.violation.player.fullName, result.isOut, 
	about.isTopInning, matchup.postOnFirst.id,
	matchup.postOnFirst.fullName, matchup.postOnFirst.link,
	matchup.postOnSecond.id, matchup.postOnSecond.fullName,
	matchup.postOnSecond.link, matchup.postOnThird.id, 
	matchup.postOnThird.fullName, matchup.postOnThird.link, 
	base, reviewDetails.isOverturned.x, 
	reviewDetails.inProgress.x, reviewDetails.reviewType.x,
	reviewDetails.challengeTeamId.x, reviewDetails.isOverturned.y,
	reviewDetails.inProgress.y, reviewDetails.reviewType.y, 
	reviewDetails.challengeTeamId.y, reviewDetails.isOverturned, 
	reviewDetails.inProgress, reviewDetails.reviewType, 
	reviewDetails.challengeTeamId)

```

## Repeat the process for each year of desired data.
```
# Write individual seasons pbp 2021 - 2023 to disk
write.csv(master_2023_pbp_to_sql
	, "D:/R_stuff/master_2023_pbp_to_sql.csv")
write.csv(master_2022_pbp_to_sql
	, "D:/R_stuff/master_2022_pbp_to_sql.csv")
write.csv(master_2021_pbp_to_sql
	, "D:/R_stuff/master_2021_pbp_to_sql.csv")
```

<br>
# In SSMS
```
Using CTES on slightly modified partitioned SQL tables
---------
with proxy8 as (
	select cast(substring(rindex, 2, len(rindex)-2)
		as int) as rindex_new
		, *
FROM [saber].[dbo].[pbp_sql_build_8]
),
proxy9 as (
	select cast(substring(rindex, 2, len(rindex)-2) as int)
		as rindex_new
		, *
FROM [saber].[dbo].[pbp_sql_build_9]
),
---------
cte_master_pbp as (
	select one.rindex, *
	from pbp_sql_build_1 one
	join pbp_sql_build_2 two on one.rindex = two.rindex
	join pbp_sql_build_3 three on one.rindex = three.rindex
	join pbp_sql_build_4 four on one.rindex = four.rindex
	join pbp_sql_build_5 five on one.rindex = five.rindex
	join pbp_sql_build_6 six on one.rindex = six.rindex
	join pbp_sql_build_7 seven on one.rindex = seven.rindex
	join eight on one.rindex = eight.rindex_new
	join nine on one.rindex = nine.rindex_new
	join pbp_sql_build_10 ten on one.rindex = ten.rindex
	join pbp_sql_build_11 eleven on one.rindex = eleven.rindex
	)

```

<br>
#### Write partitioned tables into master pbp table. Subsequent views can be created from this master table
```
select pbp.*, info.game_date as new_game_date
into master_pbp
from cte_master_pbp pbp
join game_info_supplement info on pbp.game_pk = info.game_pk
order by rindex asc
```
