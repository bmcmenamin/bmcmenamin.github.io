---
layout: post
title: "Scripts for building Tensorflow on Google Cloud"
author: "Brenton McMenamin"
tags: [math&data, releases]
---

I've been using the Google Cloud Platform to do my deep-learning side projects. However, one of the biggests pains is setting up the environment to actually run TensorFlow. It's
stressful chasing down all the steps for installing thse packages knowing that you're on the clock and paying for every moment you're wasting on the big machine.

You'd think Google would have easy-to-use premade disk images or containers to eploy on their machines to support their own software pacakges, but I haven't found them. Maybe it's a conspiracy to get data scientists to spend an extra couple hours configuring their GPU machines?

So I've made some [bash scripts](https://github.com/bmcmenamin/sundries/blob/master/gcloud_scripts/setup_instance_tensorflow.sh) that'll run to do it all for me. This script is set up to run on a Ubuntu 16.04 machine and build a custom version of Tensorflow from source. Setting the flags at the beginning will let you choose whether or not you want to set things up for a CPU machine or a GPU machine.

It assumes that you also have a Google Cloud Storage account (i.e., Google's S3) to store a few things, in particular:
- The compiled wheel (.whl) files for tensorflow after building. This can save time on future installs if you just pull the wheel to a new machine and pip-install that rather than building the whl from scratch.
- [For GPU installs] A .deb file for CUDA
- [For GPU installs] A .tgz file for CUDnn
