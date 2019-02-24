---
title: 2D Double Diffusive Convection
date: '2017-02-03 16:37:58 +0000'
published: true
---

## Part 1: An Introduction to Rayleigh-BÃ©nard Convection

This is the first in a series of post presenting in a simple way my masters thesis on simulating double diffusive convection in 2D. It will get reasonably technical at points, both in terms of mathematics and programming, but it is my aim to make this as accessible as possible, plus if you don't understand a single section, that's okay.

### What Is Convection?

If you've done high school level physics, or you google the word convection, you might know that one definition of convection is that it is the movement of temperature in a fluid because of temperature differences. On Earth, that means that hot fluid rises because hot fluid is lighter than cold fluid. Just to be clear here, by fluid I really mean anything liquid or gas, like water or air. Think about breathing out on a cold day. All the warm air leaves your lungs and is suddenly lighter than the cold air around it, causing it rise. The slightly more technical way of putting this is that air at a higher temperature is less dense than air at a lower temperature. When on Earth, gravity causes the less dense air to rise. For water, exactly the same thing happens, the hotter it gets the less dense (and lighter) it gets.

It gets more interesting when we think of more than just a breath or a bubble of hot fluid. What if we take a fluid that gradually changes temperature over a distance, say the water in a kettle after we've switched it on, but before it's started boiling. The water at the bottom in contact with the heating elements will be hotter than the water on top and as the 

### So What's This Double Diffusive Convection?

So usually convection is temperature moving around in fluid, right? What if you had a bowl of regular water and you poured a mug of salt water into it. Salt water is denser than the normal water, so it'll all sink to the bottom and hey, look at that, we've got another type of convection. Not only does the temperature of the water change the density but the concentration of salt does too. They're independent of each other, we can change the temperature without changing the amount of salt, so we could ask what happens if
