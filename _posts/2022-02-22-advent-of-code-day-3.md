---
layout: post
title: Advent of Code - Day 3
date: "2022-02-22"
subtitle: Day 3 of the Advent of Code 2021 challenge
---

# Context

This post follows [this one](/2022-01-22-advent-of-code-day-2/) and is part of the series where I tackle Advent of Code 2021 challenges. It is dedicated to solving day 3 puzzle.

# Part one

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

# Part two

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

# Conclusion

![Easy](https://media.giphy.com/media/NaboQwhxK3gMU/giphy.gif)
