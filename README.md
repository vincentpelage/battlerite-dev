# battlerite-api
a javascript wrapper for the battlerite API

## Contents
* [About](#about)
* [Install](#install)
* [Usage Examples](#usage-examples)
* [Documentation](#documetation)
* [Todo/Wishlist](#todowishlist)
* [License](#license)

## About
A modular API wrapper for the [Battlerite](https://www.battlerite.com) [Developer API](https://developer.battlerite.com). Currently there is a module that provides a simple `endpoint querystring` => `json object` function. (`lib/node-api.js`) I plan to write a function that provides the same interface for the browser. A second module (`lib/lib.js`) consumes the first and provides an fully featured JavaScript API interface. An entrypoint for inclusion in node.js programs is provided in `index.js`  

Last known to be compatible with API `v7.5.1`

### Prior Art
* https://github.com/carlerikjohan/battlerite-api
* https://github.com/sime1/battlerite-node

My approach is decidedly more minimal than either of those, using much less code and no dependencies. I don't think this is better, just different.

* https://github.com/jlajoie/battlerite-api/

This is a tool which provides a wrapper around the internal API as used by the game itself, unfortunately that API is about a million times more useful than the official one (go figure)

## Install
You're going to want to have a recent node.
```bash
npm i -s battlerite-dev
```

## Usage Examples
Initiialize the API like so
```javascript
const Battlerite_Dev = require('battlerite-dev')
const api = new Battlerite_Dev({key: "your_very_long_api_key"}) 
```
a function exists corresponding to each API endpoint, consuming an `options` object and returning a promise that resolves to the requested JSON, for example:
```javascript
api.status().then(r => console.log(r))
```
or
```javascript
let options = {
  filter: {
    gameMode: ['ranked','casual']
  }
} 
api.matches(options).then(r => console.log(r))
```
or
```javascript
api.match('id_hash_as_string').then(r => console.log(r))
```
## Documentation
### Battlerite API Notes
The [matches](http://battlerite-docs.readthedocs.io/en/latest/matches/matches.html#get-a-collection-of-matches) endpoint doesn't quite behave as the docs state.  
Not only that, but there's currently no way of getting a playerId <=> playerName relationship using the official api, if you hit the unofficial API though you can get the data

`filter[gameMode]` returns a 404 if you use `ranked` or `casual` as the filterstring, but it does work if you use `1733162751`, which is sent as the `data.attributes.gameMode` with every(?) match right now  
`filter[teamName]` returns a 404 regardless of query? i've tried the team name as a string, and also the long id hash that is provided in roster objects  

### Telemetry API Notes
```typescript
type TelemetryType =
  | 'Structures.UserRoundSpell'
  | 'Structures.RoundEvent'
  | 'Structures.DeathEvent'
  | 'Structures.MatchReservedUser'
  | 'Structures.RoundFinishedEvent'
  | 'Structures.MatchFinishedEvent'
  | 'Structures.MatchStart'
  | 'Structures.ServerShutdown'
  | 'com.stunlock.battlerite.team.TeamUpdateEvent'
  | 'com.stunlock.service.matchmaking.avro.QueueEven'

interface TelemetryObject {
  cursor: number //not sure
  type: TelemetryType 
  dataObject : TelemetryData
}

interface TelemetryData {
  matchID: string
  time: number
  [prop: string]: any
}

type Telemetry = TememetryObject[]

```
#### Structures.MatchStart
```typescript
interface MatchStart extends TelemetryData {
  gameMode: number
  mapID: string
  matchID: string 
  region: string
  teamSize: number
  type: string //same as servertype in reserveduser
  version: string //"1.0"
}
```
#### Structures.MatchFinishedEvent
```typescript
interface MatchFinishedEvent extends TelemetryData {
  region: string // "us_southwest", "us_southeast", "us_west"
  matchLength: number
  teamOneScore: number
  teamTwoScore: number
  leavers: array
}
```
#### Structures.RoundFinishedEvent
```typescript
interface RoundFinishedEvent extends TelemetryData {

}
```
#### Structures.RoundEvent
```typescript
interface RoundEvent extends TelemetryData {
  character: number
  round: number
  timeIntoRound: number
  type: string //"MOUNT_DURATION", "ENERGY_ABILITY_USED", "ENERGY_SHARD_PICKUP", "HEALTH_SHARD_PICKUP", "ULTIMATE_USED", "MOUNTS_AFTER_ELEVATOR", "RUNE_LASTHIT", "ALLY_DEATH_ENERGY_PICKUP"
  value: number
}
```
#### Structures.MatchReservedUser
some data about each user, there's one of these for every user in a match
```typescript
interface MatchReservedUser extends TelemetryData {
  accountId: string //same as userID except with inconsistent caps
  attachment: number 
  character: number //character id same as actor?
  characterLevel: number
  characterTimePlayed: number
  division: number
  divisionRating: number
  emote: number
  league: number
  mount: number
  outfit: number
  rankingType: string //"RANKED" | "UNRANKED" | "NONE" maybe one more?
  seasonId: number
  serverType: string // "QUICK2V2" | "PRIVATE" | "QUICK3V3" there may also be additional ones
  team: number
  teamId: string 
  totalTimePlayed: number
}
```
#### Structures.DeathEvent
gives the death times of users, curiously it doesn't mention in which round the death occured
```typescript
interface DeathEvent extends TelemetryData {
  userID: string
}
```
#### Structures.ServerShutdown
```typescript
interface ServerShutdown extends TelemetryData {

}
```
#### Structures.UserRoundSpell
this one is incomplete so i'm not going to bother trying to figure out what it means
```typescript
interface UserRoundSpell extends TelemetryData {
  scoreType: string //"USES", "DAMAGE_RECEIVED", "CONTROL_DONE", "CONTROL_RECEIVED", "DAMAGE_DONE", "ENERGY_RECEIVED", "HEALING_DONE", "HEALING_RECEIVED", "ENERGY_USED"
}
```
#### com.stunlock.battlerite.team.TeamUpdateEvent
this has how much MMR you won or lost
#### com.stunlock.service.matchmaking.avro.QueueEvent
this is the matchmaker data, some of it overkaps with reserved user


### Methods
```javascript

```


## Todo/Wishlist

* Proper code documentation (JSDoc?)
* Rewrite the whole thing in typescript/purescript
* Tests?

## License

MIT
