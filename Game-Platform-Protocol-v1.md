# WFHomie/GPP Game Platform Protocol 

* Name: WFHomie/GPP
* Version: 1
* Full Name: WFHomie Game Platform Protocol
* Status: stable
* Editor: Isa Hekmatizadeh isa@wfhomie.com

The WFHomie Game Platform Protocol governs how different independent, web based group games could
communicate with the WFHomie Game server via its APIs. From now on for the simplicity, each
independent web based game called GAME and the WFHomie Game server called SERVER.

## License
Copyright (c) 2021 WFHomie Corp.

This Specification is free software; you can redistribute it and/or modify it under the terms of 
the GNU General Public License as published by the Free Software Foundation; either version 3 of 
the License, or (at your option) any later version.

This Specification is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; 
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.  

You should have received a copy of the GNU General Public License along with this program; if 
not, see <http://www.gnu.org/licenses>.

## Language
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", 
"SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this 
document are to be interpreted as described in RFC 2119 (see "[Key words for use in RFCs to 
Indicate Requirement Levels](http://tools.ietf.org/html/rfc2119)").

## Goals
The goal of the whole system is to provide a useful and efficient way for each GAME to host a group of
authorized people so they can play with each other.

## Architecture
Each Game and the SERVER is a web-based application. they communicate via REST APIs.
Assigning people to each authorized group is outside the scope of this protocol. we can assume that
SERVER already know about people in each group.
The GAME should be web based, and each group should playing independently from other groups.
Each GAME could be completly independent from other GAMEs, it means that they could hosted in different
machines and they do not need to have any connection with each other.
SERVER provide several REST API, for authentication and indicating the group of a person as well as gathering
information about each round in the game.
Each person, when he/she accessed the GAME address, provides a HTTP header named "Authentication" which contains
a token string. the GAME could use Authenticate API of the SERVER to validate the token and get related information
about the group and any game configuration. after finishing each round of game, game may use Wrap up API to provide
any kind of useful informatioun about the game and players. for example the winner and loser list, their points, etc.
Each sesson of game could consist of several round of game.

## Authenticate API
``` POST /authenticate/{token} ```

The authenticate API must be called when a player reach out to the GAME website. The response contains information
about the player, his/her group, and may contains game configuration.

Returns:
```
{
	player_id: 24,
	player_name: "Isa Hekmatizadeh",
	group_id: 71200
	group_name: "wfhomie engineering team",
	valid_from: "2021-03-12T09:59:51.322Z",
	valid_to: "2021-03-12T10:59:51.322Z",
	game_config: {
		     team_size: 10,
		     mafia_no: 2,
		     citizen_no: 8,
		     has_doctor: true,
		     has_inspector: flase
	}
	
}
```
* 200 - in the case of token is valid
* 403 - in the case of token is invalid
* 412 - in the case of request before valid_from or after valid_to

Where:
* player_id - the unique id of the player
* player_name - display name of the player
* group_id - the unique id of the group
* group_name - display name of the group
* valid_from - start of the valid duration in ISO 8601 format in UTC timezone
* valid_to - end of the valid duration in ISO 8601 format in UTC timezone
* game_config - all of the game related configuarion, these fields could be communicated while registering a GAME in the system.


## Wrap up API

POST /wrapup/{group_id}

After each round of game, GAME may call this API to provide information about the round.
Example of body:
```
{
	winner_ids: [32,12,433,13],
	loser_ids: [55,43,67]
}
```
The body of the request is completly optional, GAME could provide any kind of game related information in the body.

Returns:
* 200 - in case of success
* 400 - in case of group_id is invalid

## Security
All API calls should have "Authorization" http header, with the value provided by WFHomie.
