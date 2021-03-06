---
layout: post
title: "... and now for something completely different."
author: "Brenton McMenamin"
tags: [Misc]
---

This [script](https://github.com/bmcmenamin/sundries/blob/master/gilliam/youtube_in_terminal.sh) makes a simple CLI that will render any youtube video as dynamic ASCII art in your terminal. It uses `ffmpeg` to resample the video and run some edge detection to clean up the video. Then pipes the output through the very cool package [`jp2a`](https://csl.name/jp2a/) which convert each frame to ASCII. Seeing that I'm making animations by glueing together source material and assorted pieces of code, this is a fitting demo:

<div align="center">
    <img alt="And now for something completely different" src="/figs/something_different/mp_intro.gif" width="600px">
</div>

P.S. My only regret is that it is not called by typing as "`python -c 'import gilliam;'`"

P.P.S. I like the number of mediums and conversion this image has experience. It started out as paper image cutouts, photographed frame by frame, broadcast as an analog TV signal, recorded on VHS, digitized on YouTube, converted to JPG, render as text in a terminal window, and image cpatured into a GIF.