---
layout: post
title: "Illusory movement with ImageMagick"
author: "Brenton McMenamin"
tags: [Misc]
---

I wanted to get more familiar with ImageMagick, so I decided to use it to re-create one of my favorite visual illusions from back-in-the-day when I studied the visual system. [Illusory continuous-motion illusion](https://journals.sagepub.com/doi/abs/10.1068/p150627) is an illusion where you can create the illusion of perpetual, continuous movement from a single short clip. The trick is that on each repetition the contrast for the clip inverts.

An example [here](http://www.georgemather.com/MotionDemos/FourstrokeQT.html) on George Mather's website shows this by repeating two frames of video in the following sequence `Frame 1 -> Frame 2 -> Inverted Frame 1 -> Inversted Frame 2`. Below, I took a short video clip with 14 frames and accomplished the same thing. Using nothing but built-in ImageMagick commands it was pretty simple... so I decided to jazz it up with some stroboscopic neon colors while I was at it.

Here's the script:

```bash
#!/bin/bash

ROOTNAME=hypercat
INPUT_VIDEO_FILE=cat_trim.mp4

TEXT='HYPERCAT'

rm -rf /tmp/$ROOTNAME
mkdir /tmp/$ROOTNAME
mkdir /tmp/$ROOTNAME/raw/
mkdir /tmp/$ROOTNAME/hp/
mkdir /tmp/$ROOTNAME/grads/
mkdir /tmp/$ROOTNAME/hp_color/
mkdir /tmp/$ROOTNAME/raw_with_color/


# Convert movie clip to image series
ffmpeg -hide_banner -i $INPUT_VIDEO_FILE /tmp/$ROOTNAME/raw/thumb_%05d.png
for infile in /tmp/hypercat/raw/*.png; do
    convert $infile -normalize -auto-level $infile
done


# Create randomly colored plasma gradients
IMAGE_SIZE=`identify -format "%wx%h" /tmp/hypercat/raw/thumb_00001.png`
convert -size $IMAGE_SIZE plasma:fractal -modulate 1000,1000 -seed 1212 /tmp/$ROOTNAME/_gradient.png
convert -size $IMAGE_SIZE -blur 0x25 -emboss 2 -seed 1212 plasma:chartreuse-HotPink /tmp/$ROOTNAME/gradient.png
hue_rotation=100
for infile in /tmp/hypercat/raw/*.png; do
    outfile=${infile/raw/grads}
    convert /tmp/$ROOTNAME/gradient.png -modulate 100,100,$hue_rotation -alpha on $outfile
    hue_rotation=$((hue_rotation+20)) 
done


# Get highpasses of each frame
for infile in /tmp/hypercat/raw/*.png; do
    outfile=${infile/raw/hp}
    convert $infile \
        \( -clone 0 -blur 0x8 \) \
        \( -clone 0 -clone 1 -compose mathematics \
        -set option:compose:args "0,-1,1,0.5" -composite \) \
        -normalize \
        -alpha off \
        -morphology Open Octagon \
        -delete 0,1 \
        -font Silom -pointsize 80 -gravity South -fill white -annotate 0 $TEXT \
        $outfile
done


# Superimpose highpass with gradient to make brighly colored outlines
for infile in /tmp/hypercat/raw/*.png; do
    hp_file=${infile/raw/hp}
    color_file=${infile/raw/grads}
    out_file=${infile/raw/hp_color}

    convert $color_file $hp_file -alpha Off \
        -fx "v<0.6 ? 0 : u/v - u.p{0,0}/v + u.p{0,0}" \
        $hp_file -compose Copy_Opacity -composite \
        $out_file

    convert $out_file -transparent black $out_file
done


# Superimpose colored lines on raw images
for infile in /tmp/hypercat/raw/*.png; do
    raw_file=${infile/raw/raw}
    color_file=${infile/raw/hp_color}
    out_file=${infile/raw/raw_with_color}
    convert $raw_file $color_file \
        -compose hardlight -composite \
        $out_file
done


# Assemble frames into gif
convert -delay 2 -loop 0 /tmp/$ROOTNAME/raw_with_color/thumb_*.png /tmp/$ROOTNAME/raw_with_color.gif
convert /tmp/$ROOTNAME/raw_with_color.gif -negate /tmp/$ROOTNAME/seq_inv.gif
convert /tmp/$ROOTNAME/*.gif /tmp/$ROOTNAME/full.gif
convert -delay 2x100 /tmp/$ROOTNAME/full.gif $ROOTNAME.gif 

rm -rf /tmp/$ROOTNAME
```


[Hypnotoad](https://futurama.fandom.com/wiki/Hypnotoad), meet HYPERCAT:

<div align="center">
    <img alt="HYPERCAT" src="/figs/illusory-imagemagick/hypercat.gif" width="600px">
</div>
