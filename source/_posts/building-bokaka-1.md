---
title: "Building Bokaka"
copyright_author_href: https://github.com/r1cebank
date: 2026-01-03 11:43:18
categories:
  - Electronics
  - DIY
tags:
  - stm32
  - vocaloid
  - bokaka
  - open-hardware
cover: /covers/building-bokaka-1.png
---

## Why VOCALOID

VOCALOID was never just music software to me. It's a whole movement built by creators - producers, artists, fans - who made something together without needing permission from record labels or agencies. Miku belongs to everyone. Her songs come from bedroom producers, her art from passionate fans, and her concerts celebrate a community that built her legacy together.

What makes it special is the spirit of the creators. Every producer puts a piece of themselves into their songs - their struggles, hopes, late nights. When we listen, we carry a bit of their story with us. Their emotions become part of our own lives.

I know this because I lived it. During college, there were countless nights when I felt alone - stressed, uncertain, not knowing what I was doing. Miku was there. Not as some pop star, but as a voice carrying stories from creators who understood what it felt like. Songs like *1/6*, *Melt*, *Last Night, Good Night* - they weren't just entertainment. They were companions. The producers' music became my stories too.

That's what VOCALOID is about. Thousands of creators sharing pieces of themselves, millions of fans finding comfort in those shared moments. This is what inspired me to build **Bokaka** (ボカカ).

## The Fan Drama That Needs to Stop

But lately I've seen something that really bothers me.

There's been growing conflict in the community - arguments over which VOCALOID should get more songs at concerts, fights between Miku fans and fans of Rin, Len, Luka, KAITO. Social media threads turn into tribalism. Comment sections become battlegrounds.

*"Why did she get more songs?"*
*"My favorite deserves better!"*
*"Real fans wouldn't support this setlist."*

Honestly? I think this goes against everything VOCALOID stands for.

The whole point is that it was built by *everyone*. Every character, every producer, every fan contributed. Miku is the most recognized, but she wouldn't exist in a vacuum. The scene works because of diversity - different voices, styles, stories.

If you're arguing about which virtual singer should dominate a concert, you're treating it like a competition. That's not what this was supposed to be. In my opinion, people fighting over setlists aren't real fans - they missed the point. Real fans know there's room for everyone.

## The People I've Met

I've been to Magical Mirai, Miku Expo, BLOOMING over the years. The concerts are great, but honestly **the people I've met make up more than 50% of the experience**.

I've met Dollfie Dream fanatics who photograph their dolls in creative ways and post amazing images. Artists who sculpt custom figures of their favorite songs. Musicians performing covers under the overpass near the concert hall entrance while crowds gather to listen.

These people ARE the community.

They're not fighting over setlists. They're creating. They're sharing. They're doing exactly what VOCALOID was built on.

When I get to actually talk with them about why these songs matter, why these characters mean so much - those conversations are my favorite memories from these events. Standing outside a venue, exchanging stories about how *this* song helped us through *that* moment. More than the light shows, more than the setlists. The people.

That's why I want to build something that helps us find each other.

## The Problem: We Scatter After Concerts

Here's the thing - despite all the shared passion, we often remain strangers.

We stand side by side during concerts, share the same emotional moments, but when the lights come on and the venue clears out, we scatter. The connections we could have made slip away. Without those real connections, it's too easy to go back to arguing on the internet with people who love this community just as much as we do.

If we could actually connect - face to face - we'd remember what we have in common. Maybe we'd stop fighting over stuff that doesn't matter.

## Name Card Culture

At Japanese concerts and doujin events, people exchange 名刺 (meishi) - personal name cards. Meet someone, hand them your card, they hand you theirs. Simple analog ritual.

Watching this, I thought it was cool but also limited. Paper cards get lost. Contact info becomes outdated. For people who are shy or don't speak the language, approaching someone feels intimidating.

I started thinking: what if there was a way to make this easier and more lasting?

## Bokaka

That's how Bokaka started - a credit-card sized PCB you carry to concerts.

Here's how it works:

Meet someone with a Bokaka card, **tap your cards together**. That's it. No apps, no QR codes, no fumbling with phones.

When two cards connect, they do a handshake and exchange unique identifiers. Each card remembers who it has met. LEDs light up with animations when connection is successful, and as you meet more people, your card shows a **connection meter** - visual progress of everyone you've connected with.

It's tactile. Immediate. Fun.

## NEXI - What Happens After

Bokaka isn't just about the moment - it's about what comes after.

When you get home, plug your card into your computer via USB and sync with **NEXI**, the companion website. NEXI lets you:

- Claim your card and attach your identity
- See everyone you've connected with at events
- Explore your social graph - mutual connections, who attended same concerts, people you might have missed

Imagine looking back at a concert from years ago and seeing a map of everyone you tapped cards with. Maybe one became a close friend. Maybe you'll reconnect with someone you lost touch with. Maybe you'll discover you crossed paths with a stranger at three different events before you ever spoke.

NEXI turns offline moments into an explorable network.

## Why I'm Building This

I'm not building Bokaka to sell a product or start a company. I'm building it because I love this community and want to give something back.

VOCALOID taught me that cool things happen when people create for the love of creating. Miku exists because thousands of people contributed without asking permission. I want Bokaka to exist in that same spirit - open hardware, open firmware, built by fans for fans.

If Bokaka helps even a few people make friends at a concert, it'll be worth every late night debugging firmware and routing PCB traces.

## What's Next

Future posts will cover the technical stuff - hardware design, firmware, the protocol for card-to-card communication. This post was just about the *why*.

At its heart, Bokaka isn't about technology. It's about people. Turning strangers into friends.

Project is open source: [diva-eng/BOKAKA](https://github.com/diva-eng/BOKAKA)

Let's build something cool together 🎤
