# Tournament Information

The following models are based on alot of the early Tournament models from SDR/Crossover days.  They have been adopted to APIs and other use cases.  Tournaments in general represent any type of tournament including brackets, group play, etc.  The NCAA Tournament is the biggest example of this.  The following high level examples exist:

* Tournament : Represents a specific tournament and general information about it
* TournamentConfiguration : Represents the configuration such as bracket layout for the tournament
* TournamentMatchup : Represents a specific matchup between competitors within the configuration
* TournamentGroup : Represents a group layout of the configuration with a child/parent relationship

## Tournament

### Models

#### Tournament

The following model provides the base model for a tournament and is meant to be generic for a variety of subclasses.

`String id`
The id of the tournament

`TournamentType type`
The type of the tournament

`String displayName`
The name of the tournament

`String shortDisplayName`
The short name of the tournament

`String abbreviation`
The abbreviation of the tournament

`Boolean major`
Represents whether this tournament represents a major (such as Tennis or Golf)

#### TournamentSeason

The following model is the primary model used to represent a tournament from a specific season. It inherits from `Tournament`.

`ApiReference<Season> season`
The reference to the associated season

`Integer numberOfRounds`
The number of rounds within the tournament

`Integer currentRound`
The current round in progress

`TournamentState state`
The state as either pre, in, or post

`ApiReference<Competitor> winner`
The winning competitor (team, person, etc)

`ApiReference<TournamentConfiguration> configuration`
The configuration of the tournament

#### TournamentState

The following is just a simple enum of PRE, IN, POST following similar semantics
as CompetitionStatusType.

[NOTE] I'm not completely sold on this because it overlaps and yet it doesn't 
with CompetitionStatusType.  It feels like we should re-use that but it also
feels like we just need the enumerated values.

`PRE`
The tournament has not yet started

`IN`
The tournament is actively in progress

`POST`
The tournament has completed.

#### TournamentType

`String id`
The id of the tournament type

`String text`
The text describing the tournament

### Resources

`/sports/{sport}/leagues/{league}/tournaments?types={id|type}` : `ApiCollection<ApiReference<TournamentSeason>>`
This outputs a list of all available tournaments played in the league across all seasons.  For example, in NCAA it would list the various tournaments such as NCAA, NIT, CBI, etc.  For tennis, it would list the various tournaments (Wimbledon, etc). The `types` flag may be used to filter the list by tournament type.

`/sports/{sport}/leagues/{league}/tournaments/{id|type}` : `TournamentSeason`
This should be a helper endpoint that links to the last available (or next if applicable) season of the tournament.

`/sports/{sport}/leagues/{league}/seasons/{year}/tournaments?types={id|type}` : `ApiCollection<ApiReference<TournamentSeason>>`
This outputs a list of all available tournaments for the given season. The `types` flag may be used to filter the list by tournament type.

`/sports/{sport}/leagues/{league}/seasons/{year}/tournaments/{id|type}` : `TournamentSeason`
This outputs the specific tournament for the given season/id.

## TournamentConfiguration

### Models

#### TournamentConfiguration

`String id`
The id of the configuration

`String name`
The general name of the configuration

`String description`
The description of the configuration

`ApiReference<ApiCollection<TournamentMatchup>> matchups`
The list of matchups in the tournament

`ApiReference<TournamentGroup> group`
The root group of the tournament

### Resources

`/sports/{sport}/leagues/{league}/seasons/{year}/tournaments/{id}/configuration` : `TournamentConfiguration`
This outputs the configuration for the given tournament/season.

## TournamentMatchup

### Models

#### TournamentMatchup extends Series

The matchup is essentially a series of competitions and may use the existing
series object.

`String id`
The id of the matchup

`String type` (inherited)
The type of tournament matchup

`String title` (inherited)
The title describing the matchup (ie: matchup text)

`String summary` (inherited)
The summary of wins/losses, if relevant

`Integer totalCompetitions` (inherited)
The number of possible competitions in the matchup (ie: 7 for best of 7 series) 

`Integer roundNumber`
The round, if applicable, this matchup occurs in
NOTE: I'm surprised we don't have this on Series itself (maybe it should?)

`Integer bracketLocation`
The location within this bracket where this matchup occurs

`String winnerAdvancesTo`
The location where the winner advances

`String loserAdvancesTo`
The location where the loser advances

`Boolean active`
The active state of this matchup

`Boolean completed` (inherited)
The state of whether this matchup is completed

`ApiReference<TournamentMatchupCompetitor> winner`
The winner of the matchup
NOTE: I'm surprised this is not on the core object...maybe it should

`List<TournamentMatchupCompetitor> competitors` (inherited)
The competitors in the matchup

`List<ApiReference<Event>> events` (inherited)
The list of competitions in the matchup

#### TournamentMatchupCompetitor extends SeriesCompetitor

This provides information about a specific competitor within a matchup including relevant information such as seeding, wins/losses, etc.

`Integer position`
The position of the competitor such as top or bottom

`ApiReference<Team> team` (inherited)
The reference to the team/person associated with this competitor
NOTE: this ideally should point to a competitor type to work for person and team

`String seed`
The seeding of the competitor

`Integer wins` (inherited)
The number of wins in this matchup

`Integer losses` (inherited)
The number of losses in this matchup

`Integer ties` (inherited)
The number of ties in this matchup

`Boolean isWinner`
The state of whether the competitor won or not
NOTE: I'm surprised this is not on the core object...maybe it should

### Resources

`/sports/{sport}/leagues/{league}/seasons/{year}/tournaments/{id}/matchups` : `ApiCollection<ApiReference<TournamentMatchup>>`
Get the list of all matchups for the given tournament.

`/sports/{sport}/leagues/{league}/seasons/{year}/tournaments/{id}/matchups/{id}` : TournamentMatchup
The matchup of the specific id.

`/sports/{sport}/leagues/{league}/seasons/{year}/tournaments/{id}/groups/{id}/matchups` : `ApiCollection<ApiReference<TournamentMatchup>>`
Get the list of all matchups for the given tournament group.

## TournamentGroup

### Models

#### TournamentGroup

This should mimic the standard sports grouping metaphor to a degree.

`String id`
The id of the group

`String displayName`
The name of the group

`String shortDisplayName`
The name of the group 

`String abbreviation`
The abbreviation of the group

`String location`
The general location of the city (often the noted location of the group)
NOTE: alternatively, if we had the data we could use `City city`

`ApiReference<Venue> venue`
The venue the group plays at

`Date startDate`
The starting date play begins within this group

`Date endDate`
The ending date of play within this group

`Integer displayOrder`
The order of display precedence in the parent group

`ApiReference<TournamentGroup> parent`
The parent group

`ApiReference<ApiCollection<TournamentGroup>> children`
The children groups

`ApiReference<ApiCollection<TournamentMatchup>> matchups`
The matchups within the group

### Resources

`/sports/{sport}/leagues/{league}/seasons/{year}/tournaments/{id}/groups` : `ApiCollection<ApiReference<TournamentGroup>>`
The list of top-level groups within the tournament

`/sports/{sport}/leagues/{league}/seasons/{year}/tournaments/{id}/groups/{id}` : `TournamentGroup`
The specific tournament group

`/sports/{sport}/leagues/{league}/seasons/{year}/tournaments/{id}/groups/{id}/children` : `ApiCollection<ApiReference<TournamentGroup>>`
The list of children groups

# Team Additions

In order to easily provide historical information on tournaments, we will expose helper endpoints and models to the Team.

## Team

### Models

#### Team

The following properties should be added to team.

`ApiReference<ApiCollection<TeamHistory>> history`
The various histories of the team

`ApiReference<ApiCollection<TournamentSeason>> tournaments`
The list of tournaments this team has been played in in the given year

### Resources

`/sports/{sport}/leagues/{league}/seasons/{year}/teams/{id}/tournaments?types={type}` : `ApiCollection<ApiReference<TournamentSeason>>`
The list of tournaments for the given year played in by this team. The `types` flag may be used to filter the list by tournament type.

`/sports/{sport}/leagues/{league}/teams/{id}/tournaments?types={types}` : `ApiCollection<ApiReference<TournamentSeason>>`
The list of all tournament types the team has played in over all seasons.  The `types` flag may be used to filter the list by tournament type.

`/sports/{sport}/leagues/{league}/teams/{id}/tournaments/{id}` : `TournamentSeason`
[optional] Outputs the most recent trip to that specific tournament

## Team History

### Models

#### TeamHistory

The base model for all historical contexts such as tournament histories.

`String type`
The type of historical information

`String displayName`
The name of the historical information

`List<ApiReference<?>> ocurrences`
The set of references to specific historical occurences 

`Stats statistics`
Various statistics about the historical context

#### TournamentHistory

The tournament history for a specific tournament that inherits from TeamHistory.  Note that we probably don't an actual model for this and just use the TeamHistory instance with the proper values.  The `occurences` points to each tournament season the team played that tournament in.  The `statistics` includes the following:

`wins`
The number of competition wins in all occurences

`losses`
The total number of competition losses in all occurences

`appearances`
The total number of apperances in the tournament

The following stats exist specifically for NCAA

`finalFours`
The number of final four appearances

`eliteEights`
The number of elite eight appearances

`finals`
The number of final appearances

`championships`
The number of times winning

### Resources

`/sports/{sport}/leagues/{league}/teams/{id}/history` : `ApiCollection<ApiReference<TeamHistory>>`
The list of historical context objects

`/sports/{sport}/leagues/{league}/teams/{id}/history/{type}` : `TeamHistory`
The specific team history for the given type.

# Examples

## List of Tournaments

Get a list of all tournaments for a given season

`GET /sports/basketball/leagues/mens-college-basketball/seasons/2015/tournaments`
```json
{
	"count" : 3,
	"items" : [
		{ "type" : "ncaa", "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2015/tournaments/1" },
		{ "type" : "nit", "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2015/tournaments/2" },
		{ "type" : "cbi", "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2015/tournaments/3" }
	]
}
```

## List of NCAA Tournaments

Get a list of all NCAA tournaments.

`GET /sports/basketball/leagues/mens-college-basketball/tournaments?types=ncaa`
```json
{
	"count" : 3,
	"items" : [
		{ "type" : "ncaa", "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2015/tournaments/1" },
		{ "type" : "ncaa", "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2014/tournaments/1" },
		{ "type" : "ncaa", "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2013/tournaments/1" }
	]
}
```

## Get a Specific Tournament

Get a specific instance of a tournament

`GET /sports/basketball/leagues/mens-college-basketball/seasons/2015/tournaments/1`
```json
{
	"id" : "1",
	"type" : {
		"id" : "1",
		"text" : "ncaa"
	},
	"name" : "NCAA Tournament",
	"shortName" : "NCAA",
	"abbreviation" : "NCAA",
	"major" : null,
	"season" : { "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2015" },
	"numberOfRounds" : 7,
	"currentRound" : 7,
	"state" : "post",
	"winner" : { "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2015/teams/1" },
	"configuration" : { "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2015/tournaments/1/configuration" }
}
```

## Get Team History

Get the team history including tournaments.

`GET /sports/basketball/leagues/mens-college-basketball/teams/1/history/tournament-ncaa`
```json
{
	"type" : "tournament-ncaa",
	"displayName" : "NCAA Tournament History",
	"occurences" : [
		{ "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2015/tournaments/1" },
		{ "$ref" : "/sports/basketball/leagues/mens-college-basketball/seasons/2014/tournaments/1" }
	],
	"statistics" : [
		{
			"name" : "wins",
			"displayName" : "Wins",
			"description" : "The number of total wins all-time in the tournament",
			"abbreviation" : "W",
			"value" : 0,
			"displayValue" : "0"
		},
		{
			"name" : "losses",
			"displayName" : "Losses",
			"description" : "The number of total losses all-time in the tournament",
			"abbreviation" : "L",
			"value" : 0,
			"displayValue" : "0"
		},
		{
			"name" : "appearances",
			"displayName" : "Appearances",
			"description" : "The number of total appearances all-time in the tournament",
			"abbreviation" : "A",
			"value" : 0,
			"displayValue" : "0"
		},
		{
			"name" : "championships",
			"displayName" : "Championships",
			"description" : "The number of total championships all-time in the tournament",
			"abbreviation" : "C",
			"value" : 0,
			"displayValue" : "0"
		}
	]
}
```

# Use Cases

Provide the following data for Tournament History module:

- # of Final Fours : use history.statistics.finalFours
- Tournament Record : use history.statistics.wins - history.statistics.losses
- Championship Seasons : use either /teams/1/tournaments?type=ncaa&winner=team.id or loop occurences finding winner = id
- Tournament Appearances : use history.statistics.appearances

# Open Questions

How do we handle disqualified championships?  Does data handle this or do we need a flag to say the team won but was stripped of it?
