---
layout: post
title: "Major update to model-wrangler"
author: "Brenton McMenamin"
tags: [model-wrangler, releases]
---

I hadn't looked at [model_wrangler](http://github.com/bmcmenamin/modelwrangler) for a while, but I recent had a chance to use it for rapid prototyping on a text-based convnet. Everything worked, but it was *very* confusing. I had built such a complex tangle of config files and meta-config files that it was really hard to tweak the model away from defaults. So, I knew what to focus on for the first major update.

I removed all the custom 'config' object (e.g. `LayerConfig`) and model-specific default values and decided that you can just pass in a giant dict when you initialize the model. I'd rather have you looking over parameters in a giant dict of YAML file that get passed to a model on init than having to dive into the code base and figure out where default values are going to come from.

And as long as I was in there redoing all this config stuff, I made a few other changes:
* The default for all models is to have multiple inputs/outputs (i.e., lists of layers). This makes it far easier to build fun things like siamese/triplet networks or have multiple objective functions.

* Models now treat 'embedding' layers with the same level of respect as an output layers. You can now define an `embeds` attribute during layer setup to points at embedding layers in the model (just like the `outputs` attribute points at output layers), and then later call `ModelWrangler.embed()` to get the embedded values for a particular model input.

* DatasetManagers can handle streaming from disk to handle big out-of-memory datasets. This feature may have some bugs, so I'm going to do more testing on it.
