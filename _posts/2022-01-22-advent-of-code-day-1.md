---
layout: post
title: Advent of Code - Day 1
date: "2022-01-22"
subtitle: Day 1 of the Advent of Code 2021 challenge
---

# Context

When December 2021 started I was determined to take the advent of code challenge and finish it all...

Unfortunately after a bit more than a week I got lazy and stopped.

![Shame on me](https://media.giphy.com/media/dJtDZzyjLF66I/giphy.gif)

This series of posts aims at fixing that. I will do all 25 challenges and finally finish that Advent of Code thing.

For those who don't know what that is, [Advent of Code](https://adventofcode.com/2021) is a challenge which takes place every year. It starts on Dec. 1st, and ends on Dec. 25th. Each day a challenge is unlocked and each exercise is composed of two distinct parts. You are provided with a dataset input and have to write code following the explanations provided. The code should exploit the data to calculate a number. This number is then submitted on the platform and if the answer is valid, you earn a star and unlock the next exercise.

![Let's go](https://media.giphy.com/media/YiJnzo66IFcZTAf3Qt/giphy.gif)

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

# Conclusion

Day 1 was pretty easy and straightforward. The complete code looks like this and is accessible [here](https://github.com/clementbruno/advent_of_code_2021/tree/main/day_1) on Github:

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
