# Frontend Mentor - Pomodoro app solution

This is a solution to the [Pomodoro app challenge on Frontend Mentor](https://www.frontendmentor.io/challenges/pomodoro-app-KBFnycJ6G). Frontend Mentor challenges help you improve your coding skills by building realistic projects.

## Table of contents

- [Overview](#overview)
  - [The challenge](#the-challenge)
  - [Links](#links)
- [My process](#my-process)
  - [Built with](#built-with)
  - [What I learned](#what-i-learned)
  - [Continued development](#continued-development)
  - [Useful resources](#useful-resources)
- [Author](#author)

## Overview

### The challenge

Users should be able to:

- Set a pomodoro timer and short & long break timers
- Customize how long each timer runs for
- See a circular progress bar that updates every minute and represents how far through their timer they are
- Customize the appearance of the app with the ability to set preferences for colors and fonts

### Links

- Solution URL: [Add solution URL here](https://github.com/takahashiyb/pomodoro-app)
- Live Site URL: [Add live site URL here](https://takahashiyb.github.io/pomodoro-app/)

## My process

### Built with

- Semantic HTML5 markup
- CSS custom properties
- Flexbox
- CSS Grid
- Mobile-first workflow
- Vue JS Framework
- Typescript
- Sass CSS Preprocessor

### What I learned

It was quite a while since I used forms but I did use it here for the settings so the settings only apply when the apply button is clicked. It was difficult in Vue where the best point of it is seeing reactions right away, but in this case the reactivity should be delayed so it confused me a lot, which I ended up solving with form tag and submit button.

I also had a hard time with the progress bar, which I thought I solved with a div with conical gradient background then overlapping that, but when I saw that the ends were rounded that the solution was a mistake so I decided to look change to a svg circle. This solutions in itself was difficult as well until I found how to combine pathLength, dash-offset, dash-array attributes and some rotating it to place to make a convincing progress bar.

### Useful resources

- [icomoon.io](https://icomoon.io/app/#/select/library) - I got a checkmark icon from designer: Keyamoon

## Author

- Frontend Mentor - [@takahashiyb](https://www.frontendmentor.io/profile/takahashiyb)
