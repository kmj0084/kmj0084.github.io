---
layout: posts
title:  "Pitch-by-Pitch Fact Table"
date:   2023-11-28
categories: work
tags: sql
author_profile: true
author: Keegan Johnson
description: tSQL aggregations for a sabermetrics database primary fact table
header:
 overlay_image: https://media.istockphoto.com/id/173010399/hu/fot%C3%B3/baseball-park-eredm%C3%A9nyjelz%C5%91.jpg?s=612x612&w=0&k=20&c=W9X-qgTw00wCm389IW9F43mXMk3QP0uHZ9n6qaZM__o=
 teaser: https://media.istockphoto.com/id/173010399/hu/fot%C3%B3/baseball-park-eredm%C3%A9nyjelz%C5%91.jpg?s=612x612&w=0&k=20&c=W9X-qgTw00wCm389IW9F43mXMk3QP0uHZ9n6qaZM__o=
---
# Building a sabermetrics master fact table

There are a lot of ways to build this table. My approach is to utilize Bill Petti's rbaseball package, which scrapes data from MLB, Fangraphs, Baseball Reference, etc. This data can easily be pulled into an RDBMS (I've used MSSQL) to build further tables (or views) which is more suitable for predictive modelling.

```

with proxy8 as (
select cast(substring(rindex, 2, len(rindex)-2) as int) 
as rindex_new, *
FROM [saber].[dbo].[pbp_sql_build_8]
), 

proxy9 as (
select cast(substring(rindex, 2, len(rindex)-2) as int) 
as rindex_new, *
FROM [saber].[dbo].[pbp_sql_build_9]
),

```

```

cte_master_pbp as (
	select one.rindex
		, game_pk
		, game_date
		, startTime
		, endTime
		, isPitch
		, type
		, playId
		, pitchNumber
		, details_event
		, details_awayScore
		, details_homeScore
		, details_isScoringPlay
		, details_hasReview
		, details_code
		, matchup_splits_batter
		, matchup_splits_pitcher
		, matchup_splits_menOnBase
		, home_team
		, away_team
		, pitchData_extension
		, pitchData_coordinates_aY
		, pitchData_coordinates_aZ
		, pitchData_coordinates_pfxX
		, pitchData_coordinates_pfxZ
		, pitchData_coordinates_pX
		, pitchData_coordinates_pZ
		, pitchData_coordinates_vX0
		, pitchData_coordinates_vY0
		, pitchData_coordinates_vZ0
		, pitchData_coordinates_x0
		, pitchData_coordinates_y0
		, pitchData_coordinates_z0
		, pitchData_coordinates_aX
		, pitchData_breaks_breakAngle
		, pitchData_breaks_breakLength
		, pitchData_breaks_breakY
		, pitchData_breaks_spinRate
		, pitchData_breaks_spinDirection
		, hitData_launchSpeed
		, hitData_launchAngle
		, hitData_totalDistance
		, injuryType
		, umpire_id
		, umpire_link
		, details_isOut
		, pitchData_breaks_breakVertical
		, pitchData_breaks_breakVerticalInduced
		, pitchData_breaks_breakHorizontal
		, isBaseRunningPlay
		, details_ballColor
		, details_isInPlay
		, details_isStrike
		, details_isBall
		, details_call_code
		, details_call_description
		, count_balls_start
		, count_strikes_start
		, count_outs_start
		, player_id
		, player_link
		, pitchData_strikeZoneTop
		, pitchData_strikeZoneBottom
		, details_fromCatcher
		, pitchData_coordinates_x
		, pitchData_coordinates_y
		, hitData_trajectory
		, hitData_hardness
		, hitData_location
		, hitData_coordinates_coordX
		, hitData_coordinates_coordY
		, actionPlayId
		, details_eventType
		, details_runnerGoing
		, position_code
		, position_name
		, position_type
		, position_abbreviation
		, battingOrder
		, atBatIndex
		, result_type
		, result_event
		, result_eventType
		, result_description
		, result_rbi
		, result_awayScore
		, result_homeScore
		, about_atBatIndex
		, about_halfInning
		, about_inning
		, about_startTime
		, about_endTime
		, about_isComplete
		, about_isScoringPlay
		, about_hasReview
		, about_hasOut
		, about_captivatingIndex
		, count_balls_end
		, count_strikes_end
		, count_outs_end
		, matchup_batter_id
		, matchup_batter_fullName
		, matchup_batter_link
		, matchup_batSide_code
		, matchup_batSide_description
		, matchup_pitcher_id
		, matchup_pitcher_fullName
		, matchup_pitcher_link
		, matchup_pitchHand_code
		, matchup_pitchHand_description
		, away_parentOrg_name
		, away_league_id
		, away_league_name
		, batting_team
		, fielding_team
		, last_pitch_of_ab
		, pfxId
		, details_trailColor
		, details_type_code
		, details_type_description
		, pitchData_startSpeed
		, pitchData_endSpeed
		, pitchData_zone
		, pitchData_typeConfidence
		, pitchData_plateTime
		, details_disengagementNum
		, isSubstitution
		, replacedPlayer_id
		, replacedPlayer_link
		, details_violation_type
		, details_violation_description
		, details_violation_player_id
		, details_violation_player_fullName
		, result_isOut
		, about_isTopInning
		, matchup_postOnFirst_id
		, matchup_postOnFirst_fullName
		, matchup_postOnFirst_link
		, matchup_postOnSecond_id
		, matchup_postOnSecond_fullName
		, matchup_postOnSecond_link
		, matchup_postOnThird_id
		, matchup_postOnThird_fullName
		, matchup_postOnThird_link
		, base
		, reviewDetails_isOverturned_x
		, reviewDetails_inProgress_x
		, reviewDetails_reviewType_x
		, reviewDetails_challengeTeamId_x
		, reviewDetails_isOverturned_y
		, reviewDetails_inProgress_y
		, reviewDetails_reviewType_y
		, reviewDetails_challengeTeamId_y
		, reviewDetails_isOverturned
		, reviewDetails_inProgress
		, reviewDetails_reviewType
		, reviewDetails_challengeTeamId

	from [saber].[dbo].[pbp_sql_build_1] one
	join [saber].[dbo].[pbp_sql_build_2] two on one.rindex = two.rindex
	join [saber].[dbo].[pbp_sql_build_3] three on one.rindex = three.rindex
	join [saber].[dbo].[pbp_sql_build_4] four on one.rindex = four.rindex
	join [saber].[dbo].[pbp_sql_build_5] five on one.rindex = five.rindex
	join [saber].[dbo].[pbp_sql_build_6_revamp] six on one.rindex = six.rindex
	join [saber].[dbo].[pbp_sql_build_7] seven on one.rindex = seven.rindex
	join proxy8 eight on one.rindex = eight.rindex_new
	join proxy9 nine on one.rindex = nine.rindex_new
	join [saber].[dbo].[pbp_sql_build_10] ten on one.rindex = ten.rindex
	join [saber].[dbo].[pbp_sql_build_11] eleven on one.rindex = eleven.rindex
	)

```


```

select pbp.*, info.game_date as new_game_date
into [saber].[dbo].[master_pbp]
from cte_master_pbp pbp
join [saber].[dbo].[game_info_supplement] info on pbp.game_pk = info.game_pk
order by rindex asc

```
