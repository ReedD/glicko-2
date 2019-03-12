# Glicko 2

The Glicko-2 rating system is a method for assessing a player's strength in games of skill, such as chess and Go. It was invented by [Mark Glickman](http://www.glicko.net) as an improvement of the Elo rating system, and initially intended for the primary use as a chess rating system. Glickman's principal contribution to measurement is "ratings reliability", called RD, for ratings deviation. You can read the full Glicko-2 documentation [here](http://glicko.net/glicko/glicko2.pdf).

## Basic Usage

The following illustrates the most basic low-level usage of this module.

```typescript
import Player, { Outcome } from 'glicko-2';

const mike = new Player({
  defaultRating: 1500,
  rating: 1600,
  ratingDeviation: 350,
  tau: 0.5,
  volatility: 0.06,
});

const kyle = new Player({
  defaultRating: 1500,
  rating: 1500,
  ratingDeviation: 350,
  tau: 0.5,
  volatility: 0.06,
});

// Add results to mike
mike.addResult(kyle, Outcome.Win);
mike.addResult(kyle, Outcome.Loss);
mike.addResult(kyle, Outcome.Tie);
// Add results to kyle
kyle.addResult(mike, Outcome.Loss);
kyle.addResult(mike, Outcome.Win);
kyle.addResult(mike, Outcome.Tie);

// Update their ratings
mike.updateRating();
kyle.updateRating();

// mike -> {
//   "rating": 1542,
//   "ratingDeviation": 230.15,
//   "volatility": 0.059998
// }
// kyle -> {
//   "rating": 1558,
//   "ratingDeviation": 230.15,
//   "volatility": 0.059998
// }
```

To simplify the usage of the above code we can do the following.

```typescript
import { createPlayerFactory, Match } from 'glicko-2';

// First we can create a player factory with the
// defaults we want set for each of our players
// (all option values below are the defaults)
const createPlayer = createPlayerFactory({
  defaultRating: 1500,
  defaultRatingDeviation: 350,
  defaultVolatility: 0.06,
  tau: 0.5,
});

// We can use the factory to generate players
const mike = createPlayer({ rating: 1600 });
const kyle = createPlayer();

const match = new Match(mike, kyle);
match.reportTeamAWon(); // mike won
match.reportTeamBWon(); // kyle won
match.reportTie();      // tie
match.updatePlayerRatings();
// The results are the same as in the first example above
// mike -> {
//   "rating": 1542,
//   "ratingDeviation": 230.15,
//   "volatility": 0.059998
// }
// kyle -> {
//   "rating": 1558,
//   "ratingDeviation": 230.15,
//   "volatility": 0.059998
// }
```

## Advanced Usage

The `Match` class also supports team ranking, by using a "composite player" update method.
> The Composite Opponent Update Method considers each outcome as a match against a single player possessing the average rating and deviation of the opposing team players. In effect, the update method creates a composite player out of the opposing team and uses this player’s resulting rating and deviation when updating a player. - [Garrick J. Williams](http://rhetoricstudios.com/downloads/AbstractingGlicko2ForTeamGames.pdf)

```typescript
import { createPlayerFactory, Match } from 'glicko-2';

const createPlayer = createPlayerFactory();

const aTeam = [
  createPlayer({ rating: 1800 }),
  createPlayer({ rating: 1500 }),
  createPlayer({ rating: 1600 }),
];

const bTeam = [
  createPlayer({ rating: 1700 }),
  createPlayer({ rating: 900 }),
  createPlayer({ rating: 1550 }),
];

const match = new Match(aTeam, bTeam);
match.reportTeamAWon();
match.reportTeamAWon();
match.reportTeamBWon();
match.reportTeamAWon();
match.updatePlayerRatings();
```

## When to update rankings

Generally speaking, the system was designed to allow for a series of zero-many games played within a set rating period interval. At the end of the rating period, all players ratings would be updated. The rating interval length is up to the administrator.

> The Glicko-2 system works best when the number of games in a rating period is moderate to large, say an average of at least 10-15 games per player in a rating period. [Mark E. Glickman](http://www.glicko.net/glicko/glicko2.pdf)

However, this is not always the case [lichess](https://lichess.org/qa/888/what-is-glicko-2-rating-volatility#answer-5619) for example updates after every single match.

