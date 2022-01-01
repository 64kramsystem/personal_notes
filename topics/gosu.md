# Gosu

- [Gosu](#gosu)
  - [References](#references)
  - [Hello world](#hello-world)
  - [Window](#window)
  - [Images](#images)
  - [Shapes/transformations](#shapestransformations)
  - [Input](#input)
  - [Text](#text)
  - [Sound](#sound)
  - [Other APIs](#other-apis)
  - [Math](#math)

## References

- [API](https://www.rubydoc.info/github/gosu/gosu)
- [Wiki/Tutorials](https://github.com/gosu/gosu/wiki/Ruby-Tutorial)

## Hello world

```rb
require 'gosu'

class Tutorial < Gosu::Window
  def initialize
    super(640, 480)
    self.caption = "Tutorial Game"
  end
  
  # There's no node graph management, so the entities draw() must be manually invoked.
  def update; end
  
  def draw; end
end

Tutorial.new.show
```

## Window

```rb
Gosu::Window.new.close() # closes the window
Gosu::screen_width()
Gosu::screen_height()
```

## Images

```rb
# When loading BMP images files, Gosu replaces #ff00ff with transparent pixels.
# options:
# - :tileable: hard edges on scaling (false)
# - :retro:    no scaling; overrides :tileable (false)
# - :rect:     source rectangle (x, y, w, h) (defaults to whole image)
#
image = Gosu::Image.new("assets/mad_scientist.png", options)

# Draw (/rotated)
#
image.draw(x, y, z)
image.draw_rot(x, y, z, angle)

# Preset color constants
#
Gosu::Color::BLACK.dup

# Load tiles
# options: :tileable, :retro
#
[images] = Gosu::Image.load_tiles("media/star.png", tile_width, tile_height, options)

# Amusing/convenient way to define Z ordering
#
module ZOrder
  BACKGROUND, STARS, PLAYER, UI = *0..3
end
```

There are no APIs for animations!

## Shapes/transformations

Shapes:

```rb
# It seems there is no point API; use draw_rect.
# :color1 is the start, :color2 the end.
#
Gosu::draw_rect(x, y, width, height, color, z=0, mode=:default)
Gosu::draw_line(x1, y1, color1, x2, y2, color2, z=0, mode=:default)
Gosu::draw_quad(x1, y1, color1, x2, y2, color2, x3, y3, c3, x4, y4, c4, z=0, mode=:default)
Gosu::draw_triangle(x1, y1, color1, x2, y2, color2, x3, y3, c3, z=0, mode=:default)
```

Transformations (applied to the block):

```rb
Gosu::translate(x, y) { ... }
Gosu::rotate(angle, around_x=0, around_y=0) { ... }
Gosu::scale(scale_x, scale_y, around_x, around_y) { ... }
Gosu::transform(m0, m1, m2, m3, m4, m5, m6, m7, m8, m9, m10, m11, m12, m13, m14, m15) { ... } # free-form matrix

Gosu::render(width, height) { ... } # Returns a Gosu::Image
```

## Input

The is no tap testing API.

```rb
class Tutorial < Gosu::Window
  def update
    # Inline testing
    do_action if Gosu.button_down?(mykeycode)
  end

  # Event-based testing
  def button_down(id)
    id == mykeycode ? close : super
  end
end

# (Some) Keycodes
# See: https://www.rubydoc.info/github/gosu/gosu/Gosu
#
Gosu::KB_LEFT
Gosu::KB_RIGHT
Gosu::KB_UP
Gosu::KB_ESCAPE
Gosu::GP_LEFT
Gosu::GP_RIGHT
Gosu::GP_BUTTON_0
```

## Text

```rb
# options: :name (system font), :bold, :italic, :underline
font = Gosu::Font.new(height, options)
font.draw_text("Hello world!", x, y, z, scale_x, scale_y, Gosu::Color::YELLOW)
```

## Sound

```rb
sound = Gosu::Sample.new("media/beep.wav")
sound.play

# Songs are like sounds, without some differences:
# - only one at a time can be played
# - they have other functions, like pausing
#
song = Gosu::Song.new("../media/game_over.wav")
song.play
song.pause
```

## Other APIs

```rb
Gosu.milliseconds # Game time
Gosu.random(min, max) # Float in [min, max)
```

## Math

```rb
# x offset of a vector
#
Gosu.offset_x(angle, magnitude)

# distance between two points
#
Gosu.distance(x1, y1, x2, y2)
```
