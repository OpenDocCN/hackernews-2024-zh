<!--yml
category: 未分类
date: 2024-05-27 14:35:09
-->

# Useful utilities and toys over DNS

> 来源：[https://www.dns.toys/](https://www.dns.toys/)

# Useful utilities and services over DNS

dns.toys is a DNS server that takes creative liberties with the DNS protocol to offer handy utilities and services that are easily accessible via the command line.

Copy and run the below commands to try it out.

## World time

 `dig mumbai.time @dns.toys

dig newyork.time @dns.toys

dig paris/fr.time @dns.toys` 

Pass city names without spaces suffixed with `.time`. Pass two letter country codes separated by slash optionally.

### Timezone conversion

Pass YYYY-MM-DD**T**HH:MM-$fromCity-$toCity (two letter country codes separated by slash optionally).

 `dig 2023-05-28T14:00-mumbai-paris/fr.time @dns.toys` 

## Weather

 `dig mumbai.weather @dns.toys

dig newyork.weather @dns.toys

dig amsterdam/nl.weather @dns.toys` 

Pass city names without spaces suffixed with `.weather`. Pass two letter country codes optionally. This service is powered by [yr.no](https://www.yr.no/en)

## Unit conversion

 `dig 42km-mi.unit @dns.toys

dig 32GB-MB.unit @dns.toys` 

$Value$FromUnit-$ToUnit. To see all 70 available units, `dig unit @dns.toys`

## Currency conversion (forex)

 `dig 100USD-INR.fx @dns.toys

dig 50CAD-AUD.fx @dns.toys` 

$Value$FromCurrency-$ToCurrency. Daily rates are from [exchangerate.host](https://exchangerate.host).

## IP echo

 `dig -4 ip @dns.toys` 

Echo your IPv4 address.

 `dig -6 ip @dns.toys` 

Echo your IPv6 address.

## Number to words

 `dig 987654321.words @dns.toys` 

Convert numbers to English words.

## Usable CIDR Range

 `dig 10.0.0.0/24.cidr @dns.toys

dig 2001:db8::/108.cidr @dns.toys` 

Parse CIDR notation to find out first and last usable IP address in the subnet.

## Number base conversion

 `dig 100dec-hex.base @dns.toys

dig 755oct-bin.base @dns.toys` 

Converts a number from one base to another. Supported bases are hex, dec, oct and bin.

## Pi

 `dig pi @dns.toys

dig pi -t txt @dns.toys

dig pi -t aaaa @dns.toys` 

Print digits of Pi. Yep.

## English dictionary

 `dig fun.dict @dns.toys

dig big-time.dict @dns.toys` 

Get dictionary definitions for English words. Powered by [WordNet®](https://wordnet.princeton.edu). Replace spaces with dashes.

## Rolling dice

 `dig 1d6.dice @dns.toys

dig 3d20/2.dice @dns.toys` 

The number of dice to roll, followed by `d`, followed by the number of sides for each dice, like in tabletop RPG games.

Optionally suffix by `/$number` to add it to the total. For example, a DnD roll like 2d20+3 is written as 2d20/3.

## Tossing coin

 `dig coin @dns.toys

dig 2.coin @dns.toys` 

Number of coins to toss.

## Random number generation

 `dig 1-100.rand @dns.toys

dig 30-200.rand @dns.toys` 

Generate a random number in a specified range (inclusive of the range values).

## Epoch/Unix timestamp conversion

 `dig 784783800.epoch @dns.toys` 

Convert an epoch/unix timestamp into a human readable date. Supports Unix timestamps in s, ms, µs and ns.

## Calculate aerial distance

 `dig A12.9352,77.6245/12.9698,77.7500.aerial @dns.toys` 

Calculate aerial distance between a lat-long pair

## Generate UUIDs

 `dig 5.uuid @dns.toys` 

Generate N UUIDs (v4).

## Help

 `dig help @dns.toys` 

Lists available services.

# Shortcut function

### Bash

Add this bash alias to your `~/.bashrc` file. The `+` args show cleaner output from dig.

 `alias dy="dig +short @dns.toys"` 

### Fish

Add this to your fish config file.

 `alias dy="dig +noall +answer +additional $argv @dns.toys"` 

### Zsh

Add this zsh alias to your `~/.zshrc` file. The `+` args show cleaner output from dig.

 `alias dy="dig +short @dns.toys"` 

Then, use the dy command as a shortcut.

 `dy berlin.time

dy mumbai.weather

dy 100USD-INR.fx` 

# Why?

Why not? For fun. I spend a lot of time on the terminal and doing quick unit conversions, weather checks etc. without having to open a clunky search page is useful.