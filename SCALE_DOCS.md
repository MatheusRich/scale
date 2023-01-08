# Scale Docs

Everything you need to get started with using [Scale](https://github.com/DragonRidersUnite/scale).

## Quickstart

Once you've got your game from the Scale template dropped into your `mygame` directory in your DragonRuby GTK game, here's what you'll want to do next.

### Configure

Start by specifying your name and game title in `metadata/game_metadata.txt`. The following properties are useful:

- `devid` — your itch.io username, no spaces
- `devtitle` — your name, spaces are okay
- `gameid` — the URL slug of your game on itch.io that you must already create, no spaces
- `gametitle` — the name of your game, spaces are okay!

Now when you start the DragonRuby engine for your game, you'll see what you've set.

### Lay of the Land

Because all of Scale is included with your new game, you can just browse through the source code to get the lay of the land. You'll notice in the `app/` directory a bunch of files! It's okay, don't worry. It's pretty sensibly organized, and there are plenty of comments.

Explore, be curious, and change things to suit your needs.

Scale is structured to follow the default DragonRuby GTK way of working with methods and `args.state`. That drives how much of it works. Most of Scale is organized into modules so that there aren't method name conflicts in the global space.

### Gameplay

Open up `app/scenes/gameplay.rb`, as that's where you'll actually add the fun parts of your game. `Scene.tick_gameplay` is just like the regular ole `#tick` in DragonRuby GTK. It takes args, and you go from there. It does include some handy stuff though, but just let that be.

You'll code your game just as you normally would. If you're new to DragonRuby GTK, [check out the book](https://book.dragonriders.community/) to get started.

### Input

Scale comes with some helper methods to make checking for input from multiple sources (multiple keyboard keys and gamepad buttons) easy. That code lives in `app/input.rb`, and out of the box, you get:

- `primary_down?` — check if J, Z, Space, or Gamepad A button was just pressed
- `primary_down_or_held?` — check if J, Z, Space, or Gamepad A button was just pressed or is being held
- `pause_down?` — check if Escape, P, or Gamepad Start button was just pressed, which pauses the game

You could add more methods for various inputs in your game. Maybe there's a secondary button you use. Or any number of them! These input methods make it really easy to support various input methods without worrying about the keys/buttons that are being pressed. Want to add a new key for a given layout? Just change it.

Here's an example of how you'd use it if you wanted to have your player swing their sword:

```
if primary_down?(args.inputs)
  swing_sword(args, args.state.player)
end
```

### Text

You'll very likely need to display text in your game to show health or the player's level or to give instruction. It's a fundamental part of game user interfaces. `app/text.rb` contains a few helpful constructs to make working with text easier.

First is the `TEXT` Hash constant. It contains key values of the text to display in your game. This is preferable to putting strings everywhere because you can more easily review the text for typos and eventually translate your game much more easily. It takes a little time to get used to putting your text here first, but it'll become second nature quick enough. You'll see text for what already exists in Scale.

`#text` is a method you call to access the values in `TEXT`. Example:

``` ruby
text(:start)
```

returns `"Start"`.

Then we've got the `#label` method. It contains sensible defaults for outputting labels quickly with DragonRuby GTK. It's got a friendlier API than the default data structure in DRGTK. There are multiple ways to use it.

In its simplest form, you can pass in a symbol key for `TEXT` and specify the position to render it:

``` ruby
args.outputs.labels << label(:start, x: 100, y: 100)
```

You can also pass in a value, like a number that will display:

``` ruby
args.outputs.labels << label(args.state.player.level, x: 100, y: 100)
```

Additional parameters are supported too, all the way up to something like this:

``` ruby
args.outputs.labels << label(
  :restart, x: 100, y: 100, align: ALIGN_CENTER,
  size: SIZE_SM, color: RED, font: FONT_ITALIC
)
```

In `app/text.rb` you'll find the constants for text size and the various fonts. Change them as you'd like!

### Sprites

A lot of games need sprites, and when making them, it's so helpful to see them in the context of your game. DragonRuby GTK allows you to reload sprites in your game, but you need to specify which ones to reload. To get around this, in Scale you track your sprites in `app/sprite.rb`'s `SPRITES` hash. It's a little tedious, but by having a dictionary of sprites, they can easily be reloaded with <kbd>i</kbd> when developing your game.

Get the path for your sprite with `Sprite.for(:key)`:

``` ruby
args.outputs.sprites << {
  x: 100, y: 100, w: 32, h: 32, path: Sprite.for(:player),
}
```

### Scenes

The `app/scenes/` directory is where different scenes of your game go. There are scenes for Paused, Main Menu, Settings, and Gameplay. Scale uses `args.state.scene` to know which scene the game is in.

You can switch scenes by using `Scene.switch(args, :gameplay)` where the parameter is the symbol of the scene you want. It must match the name of your scene method after the `tick_` prefix. So if you wanted a `:credits` scene, you'd define it like this in `app/scenes/credits.rb`:

``` ruby
module Scene
  class << self
    def tick_credits(args)
      args.outputs.labels << label(:credits, x: args.grid.w / 2, y: args.grid.top - 200, align: ALIGN_CENTER, size: SIZE_LG, font: FONT_BOLD)
    end
  end
end
```

(Don't forget to require it in `app/main.rb` too!)

Then when you want to show the Credits scene, call `Scene.switch(args, :credits)`.

### Colors

A simple color palette is provided in `app/constants.rb`. Most of the primary colors are present, along with some variants. They return Hashes with `r`, `g`, and `b` keys. If you have a sprite that you want to change red, you'd do this:

``` ruby
args.outputs.sprites << {
  x: 100, y: 100, w: 32, h: 32, path: Sprite.for(:player)
}.merge(RED)
```

### Adding New Files

When you add new code files to `app/`, just be sure to require them in `app/main.rb`.

### Debug Tick & Development Shortcuts

When you're making a game, you often want to have easy shortcuts to toggle settings. Maybe you want to make your player invincible so you can easily test changes out. `app/tick.rb` contains a method called `#debug_tick` that's only called when you're game is in debug mode (a.k.a. not the version shipped to players). Anything you put in there will run every tick but only while you make the game.

You'll see there are already three shortcuts Scale gives you:

- <kbd>i</kbd> — reloads sprites from disk
- <kbd>r</kbd> — resets game state
- <kbd>0</kbd> — displays framerate and other debug details

Use those as templates to add your own development shortcuts.

You can check for Debug mode in your game anywhere with the `debug?` method.

### `#debug_label` Method

It's common to need to display debug-only information about entities in your game. Maybe you want to see a value that changes over time. This is what `#debug_label` is for. It putputs text that's shown when you toggle on debug details with the <kbd>0</kbd> key.

Here's an example of how to use it to track the player's current coordinates:

``` ruby
# assume we have args.state.player that can move
player = args.state.player
debug_label(args, player.x, player.y - 4, "#{player.x}, #{player.y}")
```

## Make Scale Yours!

This is your game now. Scale is just here to help you out. Change Scale to meet your game's needs.

## Scripts

- `run_tests` — test runner for Unix-like environments with a shell that has proper exit codes on success and fail