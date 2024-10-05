---
layout: post
title: Building a birthday bot
date: "2022-01-17"
subtitle: How I built a bot in charge of reminding me everyone's birthday everyday
---

# Context

It's been a while since I wanted to remove the facebook app from my phone! Currently I use facebook for mainly two things:

1. Messenger: To interact with friends who don't want to move to Signal...
2. Birthday reminders: To remind me of everyone's birthday

Since messenger has a dedicated app I realized that I could start by getting rid of the facebook app if I was able to find an alternative for birthday reminders.

Since I like to automate things (and reinvent the wheel because, I am sure, this service already exists) I wondered: What if I built a bot which would be in charge of sending me an email everyday to tell me whose birthday it is.

Enter **1337, the friendly Birthday Bot** :tada:

In order to build it, I just needed to:

1. Retrieve the birthdays of my friends and relatives
2. Identify if there are birthdays today by scanning the data and selecting the rows matching the current date
3. If there is at least one birthday then send myself an email with the name of the person(s)
4. Automate this process to run once everyday

![Let's do this](https://media.giphy.com/media/BpGWitbFZflfSUYuZ9/giphy.gif)
<br/>

# Approach

## Step 1 - Retrieve the bithdays

The first step was to retrieve my friends' birthday dates from facebook.

It seems that a while ago facebook provided their users with the ability to export all their friends birth dates to a CSV or calendar events. They've since removed this feature probably because they realized that birthday reminders was a functionality that retained a portion of their user base.<br/>
Long story short you cannot do that anymore and I didn't want to write a scraper to do it because there was a much easier alternative. There is a small chrome/brave extension that you can install which retrieves them for you:<br/>
[Click here for the extension](https://chrome.google.com/webstore/detail/birthday-calendar-exporte/imielmggcccenhgncmpjlehemlinhjjo?hl=en)

Using the extension to retrieve the birthdates was a pretty straightforward process. You just need to connect to your facebook account and follow the steps detailed by the extension. At the end, you are provided with the option to export the information retrieved as a CSV or as calendar objects. I opted for the CSV format because I wanted my bot to parse it and notify me by email everyday if it is anyone's birthday.

## Step 2 - Parse the CSV

The CSV data retrieved was dead simple. It had 5 columns and was saved in a dedicated file `birthdays.csv` in my bot directory:

- Name
- Year
- Month
- Day
- Link to Profile

```
Name,Year,Month,Day,Link to Profile
John Doe,1984,12,1,https://facebook.com/some_facebook_user_id
Maria Doe,1984,1,15,https://facebook.com/some_facebook_user_id
...
```

Ruby is excellent for data manipulation and has a dedicated `CSV` module with a very simple API which makes it very easy to parse CSV data. Thus, I created a `birthday_robot` file inside the `/bin` directory which would contain the logic for parsing the data and triggering the email sending:

```ruby
#!/usr/bin/env ruby

require 'csv'
require 'date'

all_birthdays = CSV.read("birthdays.csv", headers: true)
current_date = Date.today
birthdays = all_birthdays.select do |birthday|
  current_date.month == birthday["Month"].to_i && current_date.day == birthday["Day"].to_i
end

if birthdays.any?
  # Send email
end
```

There are 3 small things worth noting in the above code:

1. We use `CSV.read` which works fine in our case because the file size is small and has a small number of rows. But imagine you had 10M friends !!!<br/>
   If it were the case then using `CSV.read` and storing the result in a variable would be a bad idea because it would mean building the entire CSV object in memory... <br /><br />
   ![Noooo](https://media.giphy.com/media/d10dMmzqCYqQ0/giphy.gif)<br />
   A much better alternative would be to use the `foreach` method provided by the `CSV` module because it would be way less memory intensive as it would iterate on the file line by line.

2. Same thing goes for the `select` method call that I've used. It is really simple and worked well for our use case because there are so few rows in the file. However, it is worth noting that we could easily improve the performance here because the rows retrieved are ordered by month and day. This implies that we could just check the values up until the `Month` field is superior to the current month and stop afterwards.

3. The data type of the parsed CSV values is `string`. So we need to call `to_i` on the `Month` and `Day` fields to convert them to the `integer` format which is needed in order for us to be able to perform a comparison with the data types returned when calling `current_date.month` or `current_date.day`. Otherwise we are returned an `ArgumentError` which makes total sense:

```ruby
ArgumentError: comparison of Integer with String failed
```

In other terms, an optimized version of the above code which would work much better for a large CSV dataset would look like this:

```ruby
#!/usr/bin/env ruby

require 'csv'
require 'date'

current_date = Date.today
birthdays = []
CSV.foreach("birthdays.csv", headers: true) do |row|
  break if current_date.month < row["Month"].to_i # This relies on the hypothesis that the rows are ordered in ascending order by Month/Day

  if current_date.month == row["Month"].to_i && current_date.day == row["Day"].to_i
    birthdays << row
  end
end

if birthdays.any?
  # Send email
end
```

## Step 3 - Send email

There starts the interesting part. I had never sent emails from ruby directly before. I had used Rails' `ActionMailer` and its associated abstractions but I wanted to use the `SMTP` module included in ruby's standard library which provides a lower level interface which seemed pretty interesting.

The [doc](https://ruby-doc.org/stdlib-3.0.0/libdoc/net/smtp/rdoc/Net/SMTP.html) was quite clear and I started with a basic implementation which worked well in my local environment.

```ruby
#!/usr/bin/env ruby

require 'net/smtp'
...
if birthdays.any?
  names = birthdays.map { |bday| bday["Name"] }.join(" and ")
  message = <<~MESSAGE
  From: Birthday Bot <#{BIRTHDAY_BOT_EMAIL_ADDRESS}>
  To: #{TARGET_EMAIL_ADDRESS}
  Subject: Birthday Reminder
  Date: #{Date.today.strftime('%a %d %b %Y')}
  Greetings !
  I am 1337, the Birthday Bot 🤖
  I wanted to remind you that today is #{names}'s birthday !
  Don't forget to send them a nice message
  MESSAGE

  Net::SMTP.start(
    'smtp.gmail.com',
    25,
    'mail.from.domain',
    BIRTHDAY_BOT_EMAIL_ADDRESS,
    BIRTHDAY_BOT_EMAIL_ADDRESS_PASSWORD,
    :login
  ) do |smtp|
    smtp.send_message message,
                      BIRTHDAY_BOT_EMAIL_ADDRESS,
                      TARGET_EMAIL_ADDRESS
  end
end
```

Apparently there are two ways to proceed. You can either:

- Explicitly create an `SMTP` instance and start a connection. Then send your message and manually call `finish` to close the `SMTP` session:

```ruby
smtp = Net::SMTP.start('your.smtp.server', 25)
smtp.send_message msgstr, 'from@address', 'to@address'
smtp.finish
```

- Or, opt for the version I chose above where I use the result of the `start` method on `Net::SMTP` which apparently instantiate an `SMTP` object and yields it in the block which allows us to call the `send_message` instance method afterward. The main advantage of this approach is that it closes the `SMTP` session automatically when the block ends. No need to call `finish` on `smtp`.

Everything was fine in my local environment. The email were successfully sent. So I thought that everything was fine and decided to push my code to Heroku -> `git push heroku main`<br />
Once deployed I opened an Heroku console and tried to run my command:

```bash
heroku run bash
$ birthday_bot
```

And then... :boom:

```ruby
SMTPAuthenticationError: ..., 'Must issue a STARTTLS command first')
```

Mmh ... Interesting. I googled the issue a dozen of times, tried several different solutions but did not find much useful information...

One of the [most upvoted solution](https://stackoverflow.com/a/10509882/16336179) suggested to use a port different from port `25` which is the one used in the ruby documentation because apparently it is an open relay which means that it is unauthenticated and that it was used for spamming which would explain why Gmail would not want me to use it.

Well, I tried to change the port to `587` which was the one recommended and the one most commonly seen in the `ActionMailer` config I've seen posted to StackOverflow. Unfortunately this did not work...

![Rage1](https://media.giphy.com/media/KliqvuoABugsp3eJI4/giphy.gif)

I looked at the documentation again and tried to find a solution from there by looking at anything related to `TLS`. After a few minutes of research I've found the `enable_starttls_auto` command which sounded like it could solve my problems :tada: <br />
I amended my code just slightly to make sure to call the `enable_starttls_auto` method on my `smtp` object before trying to send the message:

```ruby
...
Net::SMTP.start(
    'smtp.gmail.com',
    587,
    'mail.from.domain',
    BIRTHDAY_BOT_EMAIL_ADDRESS,
    BIRTHDAY_BOT_EMAIL_ADDRESS_PASSWORD,
    :login
  ) do |smtp|
    smtp.enable_starttls_auto # Added here to hopefully solve the issue
    smtp.send_message message,
                      BIRTHDAY_BOT_EMAIL_ADDRESS,
                      TARGET_EMAIL_ADDRESS
  end
```

But, again, it was not working

![Rage2](https://media.giphy.com/media/3ohs81rDuEz9ioJzAA/giphy.gif)

I then remembered that when I began I chose to use one syntax over another to start the SMTP connection.
Remember, when I used `Net::SMTP.start(something, something).{ |smtp| ... }` instead of explicitly instantiating an `SMTP` object, explicitly starting and explicitly finishing the connection ?
Well, I tried changing that:

```ruby
...
smtp = Net::SMTP.new('smtp.gmail.com', 587)
smtp.enable_starttls_auto
smtp.start('mail.from.domain', BIRTHDAY_BOT_EMAIL_ADDRESS, BIRTHDAY_BOT_EMAIL_ADDRESS_PASSWORD, :login)
smtp.send_message message,
                  BIRTHDAY_BOT_EMAIL_ADDRESS,
                  TARGET_EMAIL
smtp.finish
```

And guess what it freaking worked !!! I guess that `enable_starttls_auto` needed to be called on the instance before the `start` method.

![Happy](https://media.giphy.com/media/KYElw07kzDspaBOwf9/giphy.gif)

## Step 4 - Automate the daily check

Finally, the last thing I needed to handle was how I would be able to make sure that the script would run everyday without having to trigger it manually.
My code was hosted on Heroku and I started looking at solutions involving cron tables when I came across [this article](https://www.clairecodes.com/blog/2016-01-06-scheduling-ruby-and-javascript-jobs-with-heroku/) about the [scheduler add-on for Heroku](https://devcenter.heroku.com/articles/scheduler).

With very little config I could make sure that my script would run everyday at a given time. By running these 2 small commands:

```bash
heroku addons:create scheduler:standard
heroku addons:open scheduler
```

It then opened Heroku's web interface and I have been able to set the frequency of execution of my `birthday_bot` command by just filling out the form.

![Heroku scheduler](/../../assets/img/heroku_scheduler.png)

I decided to have it run everyday early in the morning so that I would wake up with an email from my bot telling me whose birthday it is.

![Birthday](https://media.giphy.com/media/eDXcHIdARrCv9xjqZ9/giphy.gif)
