---
layout: post
title: "💾 Replacing Macbook HD with an SSD"
date: 2012-12-01 12:00:00 -0800
categories: technology
tags: laptop mac
---

My poor little laptop hard drive had been whining and whimpering, so I upgraded it to an SSD. Turned out to be inexpensive and very DIY friendly, so here are my cliffs notes.

## Step 1: Choose an SSD

![Mucho fasto SSD](/assets/images/mercury-extreme.jpg)

The consensus is that [Other World Computing (OWC)](http://www.nateware.com/macsales.com) makes the most Mac-compatible SSD's. I went with the [OWC Mercury Extreme Pro SSD](http://eshop.macsales.com/shop/SSD/OWC/Mercury_6G/). 120GB cost me $149. If you have an older Macbook (pre-2011), or just want to save money, you can go with the slightly slower [OWC Mercury Electra SSD](http://eshop.macsales.com/shop/SSD/OWC/Mercury_Electra_6G/) instead. I sprung for FedEx 2-day shipping for ~$10.

## Step 2: Buy a USB Drive Case

This is so you can attach the new drive to your laptop temporarily, to copy over your data. Needs to be a 2.5" SATA for the SSD, with a USB connection for the laptop. Amazon has the [Vantec NexStar 2.5-Inch SATA to USB 2.0 External Enclosure](http://www.amazon.com/gp/product/B002JQNXZC/ref=oh_details_o00_s00_i00) for $7.99. Done.

## Step 3: Put Drive in Case

Open the NexStar drive case, and plug the OWC SSD into the connector. Close it up and attach it to your laptop via the USB cable. This step should seem very simple. If not, rethink continuing w/o help.

## Step 4: (Optional) Grab a Beer

[Drake's Denogginizer](http://www.ratebeer.com/beer/drakes-denogginizer/30946/) goes well with upgrade-related tasks. Warning: With 22oz at 9.75%, the clock is now ticking.

## Step 5: Partition the Drive

![Disk Utility Window](/assets/images/disk-utility.png)

Once you attach the drive, a window will popup saying something like "Unrecognized drive format". Click the "Initialize" button to open up Disk Utility. You should see a screen like the one at right. Click the "Partition" button in the right pane, and do the following:

1. Create a partition with all the available space, named whatever you want. I called mine "SSD Boot HD".
2. Click "+" to add a partition named "Recovery HD" of at least 750 MB in size. This is required for OSX Lion, Mountain Lion, or later, or if you're using FileVault (disk encryption).

Both should be the default type of "Mac OSX Extended (Journaled)". It's important that the "Recovery HD" partition be second, because of restrictions on how Lion/Mountain Lion can and can't resize boot partitions.

## Step 6: Clone the Drive

![Carbon Copy Cloner](/assets/images/carbon-copy-cloner.png)

Download [Carbon Copy Cloner](http://www.bombich.com/download.html) and install it. There's a fully-functional 30-day trial so you can decide whether to purchase a license later. It's a great program and worth supporting if possible.

When it first starts up, it'll ask you if you want to see the "Quick Start Guide". Say yes. It opens up instructions telling you exactly how to copy your existing hard drive to a new external drive.

All you do is select your existing drive on the left, probably "Macintosh HD", and your new drive on the right (whatever you called it in Step 5), and click "Clone".

You may get a popup saying something like, "Recovery HD partition does not contain the correct OS." If so, follow the on-screen instructions to update it. I found CCC didn't properly reset itself after this, so I had to exit, re-launch, and then click "Clone" again to start the clone.

## Step 7: Wait

Sip on your beer from Step 4.

## Step 8: Shutdown Mac, Swap Drives

Once the clone is finished, shutdown and unplug the power cable. Pull the external drive out of the case, reversing Step 3. Then, follow [these excellent instructions](http://macinstruct.com/node/407) to physically install the SSD in your Macbook. Requires a teeny tiny midget screwdriver.

## Step 9: Boot Mac, Enjoy

Everything should Just Work, although I did notice that some programs like Dropbox required me to reenter my email/password the first time. For fun, try clicking on a beastly program like Photoshop or Word and it should open up unnervingly fast.
