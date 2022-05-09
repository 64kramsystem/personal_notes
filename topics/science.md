# Science

- [Science](#science)
  - [Computer Science](#computer-science)
    - [Logic](#logic)
  - [Math](#math)
    - [Arithmetic](#arithmetic)
      - [Properties](#properties)
    - [Number theory](#number-theory)
    - [Trigonometry](#trigonometry)
    - [Geometry](#geometry)
    - [Statistics](#statistics)
    - [Algebra](#algebra)
      - [Finding the square root of a number (without exponentiation functions)](#finding-the-square-root-of-a-number-without-exponentiation-functions)
    - [Linear Algebra](#linear-algebra)
      - [Vectors](#vectors)
      - [Matrices](#matrices)
    - [Convenient stuff](#convenient-stuff)
      - [Test if a positive number is a power of two](#test-if-a-positive-number-is-a-power-of-two)
      - [Check if a number is close within the two ends of an interval](#check-if-a-number-is-close-within-the-two-ends-of-an-interval)
  - [Physics](#physics)
    - [Falling object (gravity)](#falling-object-gravity)

## Computer Science

### Logic

"Complete boolean evaluation": when the conditions are always evaluated (as opposed to Short-Circuit).

## Math

### Arithmetic

- "Left associative": evaluates from left, e.g. (1 • 2 • 3) = ((1 • 2) • 3)

#### Properties

- Commutative: (A • B) = (B • A)

### Number theory

- positive numbers: integers > 0
- natural numbers: positive numbers, plus, sometimes, 0
  - use `nonnegative integer` to avoid ambiguities
- rational numbers; result of a fraction

### Trigonometry

- tan(θ) = opposite/adjacent
- θ = arctan(opposite/adjacent)

### Geometry

Collision test between rectangles aligned with the axes. Essentially, r1.top_left < r2.bottom_right, and r1.bottom_right > r2.top_left.

```
r2.left > r1.right && r2.right < r1.left && r2.top > r1.bottom && r2.bottom < r1.top
```

### Statistics

Stddev: sqrt( Ε( (xᵢ-avg)² ) / n) )

In words: root of (the mean of (the square of difference)).

### Algebra

Solve a quadratic equation:

```
ax² + bx + c = 0

x = (-b ± √(b² - 4ac)) / 2a
```

#### Finding the square root of a number (without exponentiation functions)

Use the [Babylonian method](https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Babylonian_method); `x` is positive:

```rb
def my_sqrt(x)
  approximation = x.to_f

  while approximation - x / approximation > SIGMA             # set sigma as appropriate
    approximation = (approximation + x / approximation) / 2
  end

  return approximation
end
```

If one needs an approximated integer:

```rb
def my_sqrt(x)
  approximation = x.to_f

  while true
    new_approximation = (approximation + x / approximation) / 2

    if approximation.to_i == new_approximation.to_i
      return approximation.to_i
    else
      approximation = new_approximation
    end
  end
end
```

### Linear Algebra

#### Vectors

Vector length is called "magnitude":

  $$ |\vec{V}| = \sqrt{x^2 + y^2 + z^2} $$

The dot product is often used to indicate the angle between two vectors; algebraic formula:

  $$ \langle \vec{V},\vec{W} \rangle = V_x * W_x + V_y * W_y + V_z * W_z $$

Dot product, geometic formula:

  $$ |\vec{V}| \cdot |\vec{W}| \cdot cos(\alpha) $$

The cross product is used to find the normal vector of a surface (two vectors); it's complex.

#### Matrices

The product of two matrices V*W = X is (informal):

  $$ X_{r,c} = |Vrow_r| \cdot |Wcol_c| $$

Vcols must equal Wcols; result dimensions are (Vrows, Wcols).

Since the operation matrix * vector = vector, it transforms the vector.

### Convenient stuff

#### Test if a positive number is a power of two

`n & (n - 1) == 0`

#### Check if a number is close within the two ends of an interval

Example: Check if on a field 10 units wide, an element is within (0, 1), or (9, 10).

Intuitive explanation: x is far from the center more than (half width - interval).

```sh
$width - x < $interval ||
         x < $interval

# Subtract $width/2:

$width/2 - x < $interval - $width/2 ||
x - $width/2 < $interval - $width/2

# We can't make absolute here, otherwise it's always false, as (interval - width/2) is negative, which
# would make the equation always false.

# Change the sign:

x - $width/2 > $width/2 - $interval ||
$width/2 - x > $width/2 - $interval

# Make absolute!

|x - $width/2| > $width/2 - $interval
```

## Physics

### Falling object (gravity)

(`ᵢ` = initial)

Velocity(`t`)     : `vₜ = vᵢ + tg`
Displacement(`t`) : `y = gt²/2 + vᵢt`
Velocity(`y`)     : `vₜ² = vᵢ² - 2gy`

```rs
const JUMP_SPEED: f32 = -500.;
const GRAVITY: f32 = 700.;
const ABSOLUTE_MAX_SPEED: f32 = 300.;

if input.jump_tap {
    player.current_speed += JUMP_SPEED;
}

player.current_speed =
    (player.current_speed + frame_time * GRAVITY)
    .clamp(-ABSOLUTE_MAX_SPEED, ABSOLUTE_MAX_SPEED);

let fall_displacement = GRAVITY * frame_time.powi(2) / 2. + player.current_speed * frame_time;
player.current_pos += vec2(0., fall_displacement);
```
