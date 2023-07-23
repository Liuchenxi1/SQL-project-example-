-- #1 (1871-2016)
select year,
		count(*) as year_count
from homegames
group by year
order by year asc

-- #2 Find the name and height of the shortest player in the database (Eddie Gaedel). How many games did he play in (52)? What is the name of the team for which he played? (St.Louis Browns)
SELECT a.playerid,
		p.namefirst,
		p.namelast,
		p.height,
		a.teamid,
		SUM(distinct a.g_all) AS total_games,
		t.name
FROM appearances AS a
JOIN people AS p
ON a.playerid = p.playerid
join teams as t
on a.teamid = t.teamid
WHERE p.height in (select min(p.height)
				from people as p)
GROUP BY a.playerid, p.namefirst, p.namelast, p.height, a.teamid,t.name

-- #3 Find all players in the database who played at Vanderbilt University. 
-- Create a list showing each player’s first and last names as well as the total salary they earned in the major leagues. 
-- Sort this list in descending order by the total salary earned. Which Vanderbilt player earned the most money in the majors?

SELECT CONCAT(P.namefirst,' ',p.namelast) as play_name, 
		SUM(s.salary::numeric::money) as total_earned
FROM people AS p
JOIN salaries AS s
ON p.playerid = s.playerid
WHERE p.playerid IN (select cp.playerid
					FROM collegeplaying AS CP
					WHERE cp.schoolid = 'vandy')
GROUP BY play_name
order by total_earned desc

-- #4 Using the fielding table, group players into three groups based on their position: label players with position OF as "Outfield", 
-- those with position "SS", "1B", "2B", and "3B" as "Infield", and those with position "P" or "C" as "Battery". 
-- Determine the number of putouts made by each of these three groups in 2016.

SELECT CASE WHEN pos = 'OF' THEN 'Outfield'
		WHEN pos = 'SS' OR pos = '1B' OR pos = '2B' OR pos = '3B' THEN ' Infield'
		WHEN pos = 'P' OR pos = 'C' THEN ' Battery'
		END AS position,
		COUNT(po) AS putouts
FROM fielding
WHERE yearid = '2016'
GROUP BY position;


-- #5 Find the average number of strikeouts per game by decade since 1920. 
-- Round the numbers you report to 2 decimal places. Do the same for home runs per game. Do you see any trends?

select round(avg((so::numeric)/ (g::numeric)),2) as strikeouts_pergame, 
round(avg((hr::numeric)/ (g::numeric)),2) as homeruns_pergame,
CONCAT(
	(floor(yearid/10) * 10),
	'-' , 
	(floor(yearid/10) * 10)+9) as year
from teams
where yearid in (select yearid
	 from teams
	 where yearid >= 1920)
group by year
order by year desc

SELECT decade, 
		SUM(so) as so_batter, SUM(soa) as so_pitcher, 
		ROUND(CAST(SUM(so) as dec) / CAST(SUM(g) as dec), 2) as so_per_game,
		ROUND(CAST(SUM(hr) as dec) / CAST(SUM(g) as dec), 2) as hr_per_game
FROM (
	SELECT CASE 
			WHEN yearid >= 2010 THEN '2010s'
			WHEN yearid >= 2000 THEN '2000s'
			WHEN yearid >= 1990 THEN '1990s'
			WHEN yearid >= 1980 THEN '1980s'
			WHEN yearid >= 1970 THEN '1970s'
			WHEN yearid >= 1960 THEN '1960s'
			WHEN yearid >= 1950 THEN '1950s'
			WHEN yearid >= 1940 THEN '1940s'
			WHEN yearid >= 1930 THEN '1930s'
			WHEN yearid >= 1920 THEN '1920s'
			ELSE NULL
		END AS decade,
		so,
		soa,
		hr,
		g
	FROM teams
-- 	WHERE decade IS NOT NULL
) sub
WHERE decade IS NOT NULL
GROUP BY decade
ORDER BY decade DESC;

-- #6 Find the player who had the most success stealing bases in 2016, where success is measured as the percentage of stolen base attempts which are successful. 
-- (A stolen base attempt results either in a stolen base or being caught stealing.) Consider only players who attempted at least 20 stolen bases.

select CONCAT(P.namefirst,' ',p.namelast) as play_name,b.playerid, Round((cast(b.sb as numeric)/(cast(b.sb as numeric)+cast(b.cs as numeric))*100),2) as sucessful_stealing
from batting as b
join people as p
on b.playerid = p.playerid
where yearid = '2016' and sb+cs >=20 
order by sucessful_stealing desc

-- #7 From 1970 – 2016, what is the largest number of wins for a team that did not win the world series? (2001, SEA, 116)
-- What is the smallest number of wins for a team that did win the world series? (1981, LAN, 63)
-- Doing this will probably result in an unusually small number of wins for a world series champion – determine why this is the case. 
-- /*The season had a players' strike, which lasted from June 12 to July 31, and split the season into two halves.
-- Then redo your query, excluding the problem year. 
-- How often from 1970 – 2016 was it the case that a team with the most wins also won the world series? What percentage of the time?

select yearid,teamid,wswin,w
from teams 
where wswin is not null and wswin ='Y' and yearid in (select yearid
									  from teams
									  where yearid between 1970 and 2016 and yearid not in ('1981'))


with win_1 as (select max(w) as max,yearid as year, yearid,teamid
from teams
group by yearid,teamid
order by yearid asc)
select year, max, win_1.teamid, wswin
from win_1
inner join (select sum (distinct yearid), teamid, wswin
from teams 
where wswin is not null and yearid in (select yearid
									  from teams
									  where yearid between 1970 and 2016 and yearid not in ('1981'))
		   group by yearid, teamid, wswin) as win_2
on win_1.teamid = win_2.teamid
where wswin ='Y'



-- #8 Using the attendance figures from the homegames table, 
-- find the teams and parks which had the top 5 average attendance per game in 2016 (where average attendance is defined as total attendance divided by number of games). 
-- Only consider parks where there were at least 10 games played. 
-- Report the park name, team name, and average attendance. 
-- Repeat for the lowest 5 average attendance.

select p.park_name,h.team,round((h.attendance::numeric)/(h.games::numeric),2) as avg_attendance
from homegames as h
left join parks as p
using(park)
where h.year = '2016' and h.games >= '10'
order by avg_attendance asc
limit 5

-- #9 Which managers have won the TSN Manager of the Year award in both the National League (NL) and the American League (AL)? 
-- Give their full name and the teams that they were managing when they won the award.

WITH al_tsn AS
(SELECT a.playerid, a.yearid, m.teamid
FROM awardsmanagers AS a
INNER JOIN people AS p
ON a.playerid = p.playerid
INNER JOIN managers AS m
ON a.playerid = m.playerid
WHERE a.awardid = 'TSN Manager of the Year'
AND a.lgid = 'AL'
AND a.yearid = m.yearid
GROUP BY a.playerid, a.yearid, m.teamid),

nl_tsn AS
(SELECT a.playerid, a.yearid, m.teamid
FROM awardsmanagers AS a
INNER JOIN people AS p
ON a.playerid = p.playerid
INNER JOIN managers AS m
ON a.playerid = m.playerid
WHERE a.awardid = 'TSN Manager of the Year'
AND a.lgid = 'NL'
AND a.yearid = m.yearid
GROUP BY a.playerid, a.yearid, m.teamid)

SELECT CONCAT(p.namefirst, ' ', p.namelast) AS fullname, n.yearid, n.teamid, a.yearid, a.teamid
FROM nl_tsn AS n
INNER JOIN al_tsn AS a
ON n.playerid = a.playerid
INNER JOIN people AS p
ON n.playerid = p.playerid
GROUP BY fullname, n.yearid, n.teamid, a.yearid, a.teamid



