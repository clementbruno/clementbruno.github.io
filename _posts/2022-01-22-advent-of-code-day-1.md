---
layout: post
title: A few days of Advent of Code
date: "2022-01-22"
subtitle: Days 1 to 3 of the Advent of Code 2021 challenge
---

# Context

Every year I do a few days of advent of code but after a week or so I always get lazy and stop.

![Shame on me](https://media.giphy.com/media/dJtDZzyjLF66I/giphy.gif)

In any case I think those are usually fun exercises and I thought it would be interesting to share some of them.

For those who don't know what that is, [Advent of Code](https://adventofcode.com/2021) is a challenge which takes place every year. It starts on Dec. 1st, and ends on Dec. 25th. Each day a challenge is unlocked and each exercise is composed of two distinct parts. You are provided with a dataset input and have to write code following the explanations provided. The code should exploit the data to calculate a number. This number is then submitted on the platform and if the answer is valid, you earn a star and unlock the next exercise.

---

# Day 1

The [challenge for the first day](https://adventofcode.com/2021/day/1) was pretty simple.
The puzzle input was a series of measurements which looked something like this:

```
180
152
159
171
178
169
212
213
...
```

## Part 1

The first part of the exercise was pretty trivial. You have to count the number of times a measurement increases from the previous measurement.

Since I wanted to have a structure that I could reuse each day I started by creating a module which contained a constant with the provided input:

```ruby
# input.rb
module Input
  DATA = <<~DATA
    180
    152
    159
    171
    178
    169
    ...
  DATA
end
```

And then another file with just a class for each day and a method for each part of the challenge:

```ruby
require_relative "input.rb"

input = Input::DATA

class DayOne
  def initialize(input)
    @input = input
  end

  def part_one
    # TODO
  end

  def part_two
    # TODO
  end
end

solution = DayOne.new(input)
p solution.part_one
p solution.part_two
```

The solution that I opted for was quite straightforward:

1. Parse the input to have a list of numbers
2. Iterate on each element of the list
3. Check if the current element is greater than the previous one
4. If it is greater then increase the count by one

The code for part 1 ended up looking something like this:

```ruby
require_relative "input.rb"

input = Input::DATA.split("\n").map(&:to_i)

class DayOne
  def initialize(input)
    @input = input
  end

  def part_one
    @input.each_with_index.reduce(0) do |increases, (depth, idx)|
      increases if idx == 0

      if depth > @input[idx - 1]
        increases += 1
      end
      increases
    end
  end
end

solution = DayOne.new(input)
p solution.part_one
```

A few remarks on the above code

- I've parsed the input to convert it to a list of numbers outside of the class which is an arbitrary decision. I could have instantiated a `DayOne` object with the raw data and then have a method to parse it. This would also be completely acceptable.
- I've used a combination of `each_with_index` and `reduce` which may seem a bit uncommon to people coming from other programming languages and who are not used to ruby. `each_with_index` allows us to keep track of the index of the current element we are iterating on, so using `idx - 1` allows me to check the previous element in the list. And `reduce` (sometimes you'll see `inject` instead but it is the exact same thing since these methods are aliases) is a method which allows us to iterate through a list and instanciating an object, here it is the variable called `increases` which we create with a value of `0` that is passed as a parameter to the `reduce` method. In documentation you'll often see `memo` used for the variable name. Then we have a block with the `increases` object that we've created and the current element and its index in between brackets. If the textual explanation above is unclear, hereafter is a more pythonic looking version performing exactly the same thing:

```ruby
increases_count = 0
idx = 1
while idx < input.length - 1
  if input[idx] > input[idx - 1]
    increases count += 1
  end
  idx += 1
end
puts increases_count
```

## Part 2

The second part was a bit more fun since it involved counting the increases of the sum of sliding windows of 3 elements.

For instance, if we look at our data:

```
180
152
159
171
178
169
...
```

The first sum here would be `180 + 152 + 159 = 491`, and the second sum would be `152 + 159 + 171 = 482`.
This means that the second sum is smaller than the first one and would not increase our count.
The third sum though `159 + 171 + 178 = 508` is greater than the previous one and would cause our count to increase.

The approach I took was:

1. Calculate all 3-elements sums and store them in a list
2. Iterate on this list as we did in part one to count increases

The code ended up looking something like that:

```ruby
require_relative "input.rb"

input = Input::DATA.split("\n").map(&:to_i)

class DayOne
  def initialize(input)
    @input = input
  end

  ...

  def part_two
    rolling_sums.each_with_index.reduce(0) do |increases, (rolling_sum, idx)|
      increases if idx == 0

      if rolling_sum > rolling_sums[idx - 1]
        increases += 1
      end
      increases
    end
  end

  private

  def rolling_sums
    @input.slice(0, @input.length-2).each_with_index.reduce([]) do |sums, (depth, idx)|
      sums << (depth + @input[idx+1] + @input[idx+2])
      sums
    end
  end
end

solution = DayOne.new(input)
p solution.part_two
```

I've added a private method called `rolling_sums` which aimed at creating the list containing all sums of 3-elements.
I called `slice` on the `@input` attribute in order not to iterate on all elements of the input because, since I would sum one element and the following two, I knew that I would not be able to do that once I would reach the last two elements of the list.

The complete code looks like this:

```ruby
require_relative "input.rb"

input = Input::DATA.split("\n").map(&:to_i)

class DayOne
  def initialize(input)
    @input = input
  end

  def part_one
    @input.each_with_index.reduce(0) do |increases, (depth, idx)|
      increases if idx == 0

      if depth > @input[idx - 1]
        increases += 1
      end
      increases
    end
  end

  def part_two
    rolling_sums.each_with_index.reduce(0) do |increases, (rolling_sum, idx)|
      increases if idx == 0

      if rolling_sum > rolling_sums[idx - 1]
        increases += 1
      end
      increases
    end
  end

  private

  def rolling_sums
    @input.slice(0, @input.length-2).each_with_index.reduce([]) do |sums, (depth, idx)|
      sums << (depth + @input[idx+1] + @input[idx+2])
      sums
    end
  end
end

solution = DayOne.new(input)
p solution.part_one
p solution.part_two
```

I could have factored the increased count into its own method taking a list as a parameter which would have saved me the logic repetition in `part_two` but I guess it's fine like that.

---

# Day 2

## Part one

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

## Part two

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

---

# Day 3

## Part one

So today, the input consisted in a list of binary strings such as:<br />

```
000001100010
100111011010
001100011001
011010001010
011010101011
001001110101
100110001101
100010010011
011110000110
011000110110
011111111110
...
```

What was required for part one was the calculation of two distinct rates:

- `gamma`: obtained by looking at the most frequent bit at each index of the sequence.
- `epsilon`: obtained by looking at the least frequent bit at each index of the sequence.

Taking the most and least frequent bit at each index allows us to obtain 2 new bit sequences which we need to convert to base 10 and then multiply with each other to obtain the desired output.

Based on this requirement the code was pretty easy to come up with:

```ruby
require "./input.rb"

input = Input::DATA.split("\n")

class DayThree
  def initialize(input)
    @input = input
  end

  def part_one
    rates = (0...@input[0].length).reduce({
      gamma: "",
      epsilon: ""
    }) do |memo, idx|
      bits = @input.map { |binary_num| binary_num[idx].to_i }
      memo[:gamma] += most_frequent(bits)
      memo[:epsilon] += least_frequent(bits)

      memo
    end

    rates[:gamma].to_i(2) * rates[:epsilon].to_i(2)
  end

  private

  def most_frequent(bits)
    bits.sum >= bits.length.to_f / 2 ? "1" : "0"
  end

  def least_frequent(bits)
    most_frequent(bits) == "1" ? "0" : "1"
  end
end

solution = DayThree.new(input)
p solution.part_one #3958484
```

The interesting chunk of code here is the `part_one` method which contains the logic.<br /> In this method we iterate on a range of `n` length where `n` is the length of each bits sequence (NB: In the input provided, all sequences have the same number of bits).
The loop is built using `reduce` which allows us to instantiate a `Hash` object with the following structure at the beginning of our loop:

```ruby
{
  gamma: "",
  epsilon: ""
}
```

Each iteration in the loop allows us to count the most frequent bit at a given index and populate `gamma` and `epsilon`.<br />
Each time, to determine whether "1" or "0" is the most frequent we perform a sum of all bits at the given index. if this sum is superior to the number of elements in the input divided by 2 then it means that the most frequent bit is 1. Otherwise it is 0.

Once this is done we just have to convert both bit sequences to a base 10 number and multiply them by each other.

## Part two

In part two, we have to multiply two new rates which are obtained with a different method.
We are asked to iterate through our list of bits sequences and each time we iterate on an index we have to keep in the list of bits sequences for the next iteration only the elements which are the most frequents for the oxygen multiplier and the least frequents for the CO2 multiplier. By doing so, at each stage, the number of remaining elements is reduced until there is only one left.<br />

The remaining elements are our multipliers, they only needs to be converted to base 10 numbers and then multiplied to get the answer.

```ruby
require "./input.rb"

input = Input::DATA.split("\n")

class DayThree
  def initialize(input)
    @input = input
  end

  def part_one
    rates = (0...@input[0].length).reduce({
      gamma: "",
      epsilon: ""
    }) do |memo, idx|
      bits = @input.map { |binary_num| binary_num[idx].to_i }
      memo[:gamma] += most_frequent(bits)
      memo[:epsilon] += least_frequent(bits)

      memo
    end

    rates[:gamma].to_i(2) * rates[:epsilon].to_i(2)
  end

  def part_two
    get_oxygen * get_co2
  end

  private

  def get_oxygen
    remaining = @input
    idx = 0
    while remaining.length > 1
      remaining = most_frequents(remaining, idx)
      idx += 1
    end
    remaining.first.to_i(2)
  end

  def get_co2
    remaining = @input
    idx = 0
    while remaining.length > 1
      remaining = least_frequents(remaining, idx)
      idx += 1
    end
    remaining.first.to_i(2)
  end

  def most_frequents(list, idx)
    first_bits = list.map { |el| el[idx].to_i }

    list.select { |bin_num| bin_num[idx] == most_frequent(first_bits) }
  end

  def least_frequents(list, idx)
    list - most_frequents(list, idx)
  end

  def most_frequent(bits)
    bits.sum >= bits.length.to_f / 2 ? "1" : "0"
  end

  def least_frequent(bits)
    most_frequent(bits) == "1" ? "0" : "1"
  end
end

solution = DayThree.new(input)
p solution.part_one #3958484
p solution.part_two #1613181
```

In the above code we've added a few new methods:

- `part_two` is pretty trivial and is just the public interface allowing us to retrieve the answer. It just performs the multiplication between the 2 values that we need to retrieve. Not much to see there.
- `get_oxygen` and `get_co2` are very similar methods. They are used to find the values that we need to multiply in order to solve the exercise. Basically they start with a copy of the input and use a `while` loop which runs until the length of the copy reaches one. In other terms the loop runs and eliminates elements until only one bit sequence is left.
- `most_frequents` and `least_frequents` are called at each iteration of the loop in `get_oxygen` and `get_co2`. These methods leverage on the previously created `most_frequent` and `least_frequent` methods that were created in part one. They are used to filter the bits sequences and keep only the most or least frequent ones.

![Easy](https://media.giphy.com/media/NaboQwhxK3gMU/giphy.gif)
