---
layout: post
title: Advent of Code - Day 2
date: "2022-01-22"
subtitle: Day 2 of the Advent of Code 2021 challenge
---

# Context

This post follows [this one](/2022-01-22-advent-of-code-day-1) and is part of the series where I will tackle all Advent of Code 2021 challenges.

# Part one

The input provided for the day contains a series of instructions and looks like this:

```
forward 7
down 8
forward 5
down 3
forward 6
down 9
down 2
forward 1
down 2
down 9
forward 8
down 8
up 6
forward 8
down 3
forward 9
up 3
...
```

The goal of the exercise is to parse these instructions to calculate a position composed of 2 distinct elements: depth and horizontal position. Then, we should multiply these 2 elements to get the answer.

The depth and horizontal position are impacted by the instructions as follows:

> forward X increases the horizontal position by X units.<br />
> down X increases the depth by X units.<br />
> up X decreases the depth by X units.

As for the previous day, we reuse the structure we've put in place with the input data stored in a constant in an `Input` module and a dedicated class to handle the logic. Please check the [previous blog post](/2022-01-22/advent-of-code-day-1) if you want more details about it.

The way I've approached the problem was:

1. Parse the input to have a list of operations
2. Calculated coordinates in a `Hash` data structure with a starting state <br />`{ horizontal: 0, depth: 0 }`
3. Iterate through the operations list and change the coordinates state accordingly

The resulting code ended up looking like that:

```ruby
require_relative "input.rb"

input = Input::DATA

operations = input.split("\n")

class DayTwo
  def initialize(operations)
    @operations = operations
  end

  def part_one
    coordinates[:horizontal] * coordinates[:depth]
  end

  private

  def coordinates
    @coordinates ||= @operations.reduce({
      horizontal: 0,
      depth: 0
    }) do |position, operation|
      direction, moves = operation.split(" ")

      case direction
      when "up"
        position[:depth] -= moves.to_i
      when "down"
        position[:depth] += moves.to_i
      when "forward"
        position[:horizontal] += moves.to_i
      end
      position
    end
  end
end

solution = DayTwo.new(operations)
p solution.part_one
```

So, what is happening in the above code? Nothing fancy actually :sweat_smile:<br />
The `part_one` method performs the multiplication of the horizontal and depth positions.<br/>
The `coordinates` method contains the core of the logic since it iterates through all the operations and mutates the coordinates to calculate the final position.

# Part two

Part two introduces a new component in the coordinates: `aim`.

The operations impact each component of the coordinates as follows:

> down X increases your aim by X units.<br />
> up X decreases your aim by X units.<br />
> forward X does two things:<br />
> 1.It increases your horizontal position by X units.<br />
> 2.It increases your depth by your aim multiplied by X.

To solve the problem we just need to modify the coordinates calculation to take into account the new component. We also have the opportunity to refactor some of the code that we wrote in part one to share a little bit of the logic:

```ruby
require_relative "input.rb"

input = Input::DATA

operations = input.split("\n")

class DayTwo
  def initialize(operations)
    @operations = operations
  end

  def part_one
    calculation_position(coordinates_part_one)
  end

  def part_two
    calculation_position(coordinates_part_two)
  end

  private

  def calculation_position(coords)
    coords[:horizontal] * coords[:depth]
  end

  def coordinates_part_one
    @operations.reduce({
      horizontal: 0,
      depth: 0
    }) do |position, operation|
      direction, moves = operation.split(" ")

      case direction
      when "up"
        position[:depth] -= moves.to_i
      when "down"
        position[:depth] += moves.to_i
      when "forward"
        position[:horizontal] += moves.to_i
      end
      position
    end
  end

  def coordinates_part_two
    @operations.reduce({
      aim: 0,
      horizontal: 0,
      depth: 0
    }) do |position, operation|
      direction, moves = operation.split(" ")
      moves = moves.to_i

      case direction
      when "up"
        position[:aim] -= moves
      when "down"
        position[:aim] += moves
      when "forward"
        position[:horizontal] += moves
        position[:depth] += position[:aim] * moves
      end
      position
    end
  end
end

solution = DayTwo.new(operations)
p solution.part_one
p solution.part_two
```

Boom ! Done 💥 <br />
That's it for day 2

![Done](https://media.giphy.com/media/l0Iyl55kTeh71nTXy/giphy.gif)
