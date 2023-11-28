---
layout: posts
title:  "Plate Appearances"
date:   2023-11-28
categories: work
tags: sql
description: this is an article about an intial sql data load for a sabermetrics database
header:
 overlay_image: https://plus.unsplash.com/premium_photo-1680700148924-4abdd12c89b5?q=80&w=1974&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
 teaser: https://plus.unsplash.com/premium_photo-1680700148924-4abdd12c89b5?q=80&w=1974&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---
# Plate Appearances Sabermetrics Table with tSQL

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

	