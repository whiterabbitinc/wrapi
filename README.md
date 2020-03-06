Welcome to the WHITE RABBIT API (WRAPI, pronounced "wrappy") documentation.

# What is it?
With WRAPI, you can experience the rabbit hole firsthand, using your favorite programming language to play a super awesome game.

# Is the game fun?
Yes, it's super fun.

# How does the game work?
You play the game by interacting with our game server via the API described in this document.

First you will create a new game. To do this, you'll need a password for the level you want to play.

Next, you move your rabbit towards the exit. Watch out for snakes! If you get hit by a snake, the game ends, but you can always start a new one.

Once you reach the exit, you'll get the password for the next level. Then you can start a new game with the new password to play the new level.

# More game mechanics
The game is played on a 2D board. (You can think of the board as a 2D array, with the top left at coordinates `[0, 0]`).

Each square of the board can be one of 5 types:
* `Player`: this is you, the rabbit!
* `Enemy`: oh no, snaaaaaaaake!
* `Wall`: a barrier that you cannot cross
* `Exit`: a portal to the next level!
* `Empty`: free space that you can walk over

Games will begin 6 seconds after they are created. During this first 6 seconds, you can look at the game board, but the game is essentially frozen.

After 6 seconds, the game starts. Every 1.5 seconds, a "tick" happens. You can think of a "tick" as a turn. Each "tick", your rabbit will advance one square in the direction that you last sent to the game (even if you sent the direction several ticks ago).

You can move up, down, left, and right, provided that the square you are moving to is not a wall. You can also tell the game that you wish to stop moving. If you have yet to send a direction for the game since creating it, or if you've sent the stop direction as your last move, your rabbit will remain stationary.

If you try to move into a wall, you simply won't move. If you move into the exit, you move into the snake, or the snake moves into you, the game will end. If you make it to the exit, you've won. If not, you've lost.

Snakes can move too, but they must follow the same movement mechanics outlined above. They may choose to give you a head start towards the exit if they're feeling nice.

You can have many games running in parallel if you want, but there's no advantage to doing so, unless you've lost or know you will lose and you want to start over before your current game is over.

There is currently no time limit on a game. So if there aren't any snakes, you have as long as you want to reach the exit.

When you reach the exit, you'll get the password to the next level (see below for details). You need to start a new game with the new password under a new game ID to play the next level that you have just unlocked. You can always re-use a password to start again at a certain level.

# How do I use WRAPI?
WRAPI works entirely over HTTP GET requests.

Simply send an HTTP GET request to the WRAPI server's appropriate endpoint.

The server will respond to you using JSON, as per the documentation below.

Your favorite programming language should already provide libraries for sending HTTP requests, reading the responses, and parsing JSON. We will include some examples below.

# API Methods
The API exposes 3 functions:
* `newgame`: This creates a new game.
* `gamestatus`: At any time, this lets you view the details of your game.
* `move`: This is how you control your rabbit.

Details for each follow.

## newgame

All you need to create a new game is the password for the level, which you provide as part of your request. For example:

> http://example.com/newgame/mypassword/

Here, `example.com` is the WRAPI server, and `mypassword` is the password for the level you'd like to play.

If unsuccessful, the response will contain an error message.

```json
{"success": 0, "err_reason": "Bad request"}
```

If successful, the response will contain the `ID` of your new game, which you will need to look up its status or make moves.

```json
{"success": 1, "id": "cw2PPWnfwW"}
```

The response will NOT contain any information about the game. That is what the `gamestatus` method is for.

## gamestatus

This method will tell you everything you need to know about the game, including the game status and the position of your rabbit, the enemy snake(s) (if any), and the exit. It will also tell you the password for the next level, but only once (and if) you have reached the exit.

To look up a game's status, you just need the `ID` that you obtained from the `newgame` method. For example:

> http://example.com/gamestatus/cw2PPWnfwW/

If unsuccessful, you'll get an error similar to the one documented above for newgame.

If successful, you will get detailed information about the game:

```json
{
  "success": 1,
  "game_info": {
    "level_num": 1,
    "started": 1,
    "tick": 23,
    "game_over": 1,
    "game_won": 1,
    "id": "cw2PPWnfwW",
    "board_info": {
      "board_width": 5,
      "board_height": 5,
      "player_pos": {
        "x": 2,
        "y": 3
      },
      "exit_pos": {
        "x": 2,
        "y": 3
      },
      "wall_pos": [
        {
          "x": 0,
          "y": 0
        },
        {
          "x": 1,
          "y": 0
        },
        {
          "x": 2,
          "y": 0
        },
      ],
      "enemy_pos": []
    }
  },
  "next_level_password": "example"
}
```

We trust the names of the keys in the JSON dictionary above are mostly self-explanatory. `pos` stands for position, and positions will always have `x` and `y` keys available. In this contrived example, the game has been won, and the next level password is available. There were no snakes in this example, but if there were, their positions would have been provided just like in `wall_pos` for walls.

Of course all of the positions will remain up-to-date as the game progresses.

## move

You control your rabbit by sending the `direction` that you want to move, and of course the `ID` of the game. Your rabbit will move this direction each tick, until the game ends or until you send a new move position. The valid positions are:
* `u`: Up
* `d`: Down
* `r`: Right
* `l`: Left
* `s`: Stop (remain stationary)

For example, to move up in the game from the previous examples:
> http://example.com/move/cw2PPWnfwW/u

Or to stop moving:
> http://example.com/move/cw2PPWnfwW/s

When you stop, the game does not pause. The snakes (if any) will still be free to move on each tick.

If you try to move in a game that has ended, nothing happens.

If you try to move before the game starts, your move will be registered, but won't actually happen until the first tick.

If you send multiple moves between ticks, the one that the server received last will be the one applied on the next tick and all following ticks.

# Examples

Briefly, in Python, to view the level number of a game, you might do this:

```python
import urllib
import json
req = urllib.urlopen("http://example.com/gamestatus/cw2PPWnfwW/")
raw_response = req.read()
decoded_json = json.loads(raw_response)
if "success" not in decoded_json or not decoded_json["success"]:
    raise RuntimeError("Error!")

print "I am on level ", str(decoded_json["game_info"]["level_num"])
```
