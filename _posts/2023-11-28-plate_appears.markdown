---
layout: posts
title:  "Saber Main Fact Table"
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

# Plate Appearances Sabermetrics View (tSQL)

```

use saber
go

--drop view dbo.at_bat_log;
--go

CREATE VIEW dbo.at_bat_log AS

with proxy as (
	select
		game_date
		, batting_team
		, matchup_batter_fullName
		, concat(atBatIndex,'-',game_pk) as pa_index
		, result_eventType
		, result_rbi
		, about_inning
		, about_startTime
		, about_endTime
		, max(count_outs_start) as outs_start
		, max(count_balls_end) as balls_end
		, max(count_strikes_end) as strikes_end
		, max(count_outs_end) as outs_end
		, max(pitchNumber) as total_pitches
		, matchup_pitcher_fullName
		, matchup_pitchHand_description
		, matchup_splits_batter
		, matchup_splits_menOnBase
		, fielding_team
	--	, matchup_postOnFirst_fullName
	--	, matchup_postOnSecond_fullName
	--	, matchup_postOnThird_fullName
	from 
		master_pbp
	--where batting_team = 'Houston Astros' and game_date > '2023-03-29'
	group by 
		game_date
		, batting_team
		, matchup_batter_fullName
		, concat(atBatIndex,'-',game_pk)
		, result_eventType
		, result_rbi
		, about_inning
		, about_startTime
		, about_endTime
		, matchup_pitcher_fullName
		, matchup_pitchHand_description
		, matchup_splits_batter
		, matchup_splits_menOnBase
		, fielding_team
	--	, matchup_postOnFirst_fullName
	--	, matchup_postOnSecond_fullName
	--	, matchup_postOnThird_fullName
		) 

select 
	*
	, case when result_eventType = 'single' then 1 else 0 end as single
	, case when result_eventType = 'double' then 1 else 0 end as 'double'
	, case when result_eventType = 'triple' then 1 else 0 end as triple
	, case when result_eventType = 'home_run' then 1 else 0 end as home_run
	, case when result_eventType = 'strikeout' then 1 else 0 end as strikeouts
	, case when result_eventType = 'walk' then 1 else 0 end as walks
	, case when result_eventType = 'double_play' then 1 else 0 end as double_play
	, case when result_eventType = 'grounded_into_double_play' then 1 else 0 end as grounded_dp
	, case when result_eventType = 'hit_by_pitch' then 1 else 0 end as hit_by_pitch
	, case when result_eventType = 'sac_fly' then 1 else 0 end as sac_fly
--	, dense_rank() over ( partition by batting_team, game_date order by about_startTime asc ) as batting_order
	, case when result_eventType in ('field_out', 'strikeout', 'single', 'double', 'home_run', 'force_out', 'grounded_into_double_play', 
		'field_error', 'triple', 'double_play', 'fielder_choice', 'fielders_choice_out', 'strikeout_double_play', 'other_out', 'triple_play') then 1 else 0 end as at_bat
--	, sum(case when result_eventType = 'single' then 1 else 0 end) over (partition by matchup_batter_fullName) as season_singles
--	, sum(case when result_eventType = 'double' then 1 else 0 end) over (partition by matchup_batter_fullName) as season_doubles
--	, sum(case when result_eventType = 'triple' then 1 else 0 end) over (partition by matchup_batter_fullName) as season_triples
--	, sum(case when result_eventType = 'home_run' then 1 else 0 end) over (partition by matchup_batter_fullName) as season_homers
-- 	, sum(result_rbi) over (partition by matchup_batter_fullName) as season_rbis
from 
	proxy
-- where batting_team = 'Houston Astros' and game_date > '2023-03-29'
group by 
	game_date
	, batting_team
	, matchup_batter_fullName
	, pa_index
	, result_eventType
	, result_rbi
	, about_inning
	, about_startTime
	, about_endTime
	, outs_start
	, balls_end
	, strikes_end
	, outs_end
	, total_pitches
	, matchup_pitcher_fullName
	, matchup_pitchHand_description
	, matchup_splits_batter
	, matchup_splits_menOnBase
	, fielding_team
--	, matchup_postOnFirst_fullName
--	, matchup_postOnSecond_fullName
--	, matchup_postOnThird_fullName

/*
order by
	game_date
	, about_StartTime asc
	, matchup_batter_fullName
*/

```
	