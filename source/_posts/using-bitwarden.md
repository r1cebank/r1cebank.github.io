---
title: Using Bitwarden as password manager
copyright_author_href: 'https://github.com/r1cebank'
date: 2021-03-03 01:12:59
categories:
- Review
- Software
tags:
- security
- software
cover: /covers/bitwarden.png
---
When I was in high school and starting to know about the danger of password being stolen, I started to change my password every 6 month. It got really crazy as I started to get more and more online accounts. During that time I have no list of all the online accounts and I often forget one of the account and I ended up forgot which pass password that account is using.

The I realized using the same password for everything is not a good idea, also there are asshole websites that only allow you to use password with only numbers or only alphanumeric.
{% asset_img image.png your email password shouldn't be the same as your drawbridge password %}
## Lastpass
{% asset_img lastpass.png lastpass logo %}
Lastpass was the first password manager I used, at that time it was recommended by one of my friend and given its free price and great browser integration I use it for a while. Until the couple security incident in 2011 and 2015.
[Website](https://www.lastpass.com/)

## 1Password
{% asset_img 1password.png 1password logo %}
1Password was the replacement after Lastpass, I really enjoyed its app syncing and later family sharing features. The app was great and I am able to do have all my passwords sync accross all my devices (iPhone, iPad, Laptop, Desktop, anything with a web browser).

It all changed after I started to create a secure server running locally on my network that hosts all my private services (DNS, git, seafile, VPN, etc). Then I realized, I should maybe have all my password hosted on that private server as well. After some investigation, I discovered Bitwarde, a open source password manager that I am able to run it anywhere.
[Website](https://1password.com/)
## Bitwarden
{% asset_img bitwarden.png bitwarden logo %}
After used 1Password for more than 7 years, here are some difference I found between the two:
[Website](https://bitwarden.com/)

|   App   | Self Hosted | Open Source | iOS App | iOS Autofill extension | Syncing | Family Sharing | Browser Extension | Linux Support |
|:---------:|:-----------:|:-----------:|:-------:|:----------------------:|:-------:|:--------------:|:-----------------:|---------------|
| 1Password |      O      |      O      |    X    |            X           |    X    |        O       |         X         | X             |
| Bitwarden |      X      |      X      |    X    |            X           |    X    |        X       |         X         | X             |

As you can see the features between both solutions are pretty similar, but one thing stood out for Bitwarden is its open source and able to self host. Then I am able to control my own data and take it anywhere. If I wanted I can have Bitwarden running on a USB thumbdrive shaped computer.
