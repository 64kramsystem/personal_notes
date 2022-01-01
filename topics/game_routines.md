# Game routines
- [Game routines](#game-routines)
  - [Notes](#notes)
  - [Bidimensional scrolling](#bidimensional-scrolling)
  - [Player Animations/Acceleration](#player-animationsacceleration)

## Notes

All the routines focus on the subject, and don't include the context; they're generally assumed to be inside a `Gosu::Window` subclass.

## Bidimensional scrolling

```rb
# World coordinates
attr_accessor :player_x, :player_y
# Bidimensional array
attr_accessor :tiles
# The reference tile is the top left visible one
attr_accessor :reference_tile_row, :reference_tile_col
# Screen coordinates
attr_accessor :reference_tile_x, :reference_tile_y

def update
  self.player_x, self.player_y = compute_player_position_from_input
  self.reference_tile_row, self.reference_tile_col, self.reference_tile_x, self.reference_tile_y = compute_tile_position
end

def compute_player_position_from_input
  player_x, player_y = self.player_x, self.player_y

  player_x = (player_x - PLAYER_SPEED).clamp(0, TILES_X_COUNT * SCREEN_WIDTH - 1) if button_down?(KB_A)
  player_x = (player_x + PLAYER_SPEED).clamp(0, TILES_X_COUNT * SCREEN_WIDTH - 1) if button_down?(KB_D)
  player_y = (player_y - PLAYER_SPEED).clamp(0, self.tiles.size * SCREEN_HEIGHT - 1) if button_down?(KB_W)
  player_y = (player_y + PLAYER_SPEED).clamp(0, self.tiles.size * SCREEN_HEIGHT - 1) if button_down?(KB_S)

  [player_x, player_y]
end

def compute_tile_position
  # There are slightly different references that can be used; this is just an option.

  # Adjust for the player being in the middle of the screen.
  world_reference_x = (self.player_x - SCREEN_WIDTH / 2).clamp(0, SCREEN_WIDTH * (TILES_X_COUNT - 1))

  tile_col = world_reference_x / SCREEN_WIDTH

  # The distance of the player from the reference is the start of the tile drawing, on the opposite direction.
  tile_x = -(world_reference_x % SCREEN_WIDTH)

  world_reference_y = (self.player_y - SCREEN_HEIGHT / 2).clamp(0, SCREEN_HEIGHT * (self.tiles.size - 1))
  tile_row = world_reference_y / SCREEN_HEIGHT
  tile_y = -(world_reference_y % SCREEN_HEIGHT)

  [tile_row, tile_col, tile_x, tile_y]
end

def draw_player
  # When the player is in the outermost half screens, don't anchor it to the center anymore.
  player_screen_x = if self.player_x <= SCREEN_WIDTH / 2 || self.player_x >= TILES_X_COUNT * SCREEN_WIDTH - SCREEN_WIDTH / 2
    player_x % SCREEN_WIDTH
  else
    SCREEN_WIDTH / 2
  end

  player_screen_y = if self.player_y <= SCREEN_HEIGHT / 2 || self.player_y >= self.tiles.size * SCREEN_HEIGHT - SCREEN_HEIGHT / 2
    player_y % SCREEN_HEIGHT
  else
    SCREEN_HEIGHT / 2
  end

  draw_rect(player_screen_x, player_screen_y, 1, 1, Color::YELLOW, 1)
end

def draw_background
  self.tiles.dig(self.reference_tile_row, self.reference_tile_col).draw(self.reference_tile_x, self.reference_tile_y, 0)

  # For simplicity, draw also if outside the screen.

  self.tiles.dig(self.reference_tile_row, self.reference_tile_col + 1)&.draw(self.reference_tile_x + SCREEN_WIDTH, self.reference_tile_y, 0)
  self.tiles.dig(self.reference_tile_row + 1, self.reference_tile_col)&.draw(self.reference_tile_x, self.reference_tile_y + SCREEN_HEIGHT, 0)
  self.tiles.dig(self.reference_tile_row + 1, self.reference_tile_col + 1)&.draw(self.reference_tile_x + SCREEN_WIDTH, self.reference_tile_y + SCREEN_HEIGHT, 0)
end
```

## Player Animations/Acceleration

Assumes a Tetra-like animation class

```rb
attr_accessor :velocity
attr_accessor :state      # IDLE/RUNNING
attr_accessor :animations # {state => animation}

# Change only if different, and if so, restart!
def set_state(state)
  if self.state != state
    self.state = state
    self.animations[self.state].restart
  end
end

def update
  if button_down?(KB_A)
    self.velocity = (self.velocity.x - 0.5).clamp(-5.0, -0.5)
  elsif ... # opposite for the other direction
  else
    # decrease the velocity of a max between ±0.5, and the velocity itself
    self.velocity.x -= self.velocity.clamp(-0.5, 0.5)
  end

  # ... handle pos

  if self.velocity.x != 0.0 {
    self.animations[self.state].set_state(RUNNING)
  else {
    self.animations[self.state].set_state(IDLE)
  end
end

def draw
  self.animations[self.state].advance
  self.animations[self.state].draw
  # ...
end
```
