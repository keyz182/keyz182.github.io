---
id: 14
title: 'It&#8217;s sound Jim, but not as we know it!'
date: 2008-10-23T10:07:00+00:00
author: Kieran Evans
layout: post
guid: 14
blogger_blog:
  - blagodoom.blogspot.com
blogger_author:
  - Keyzohttp://www.blogger.com/profile/13751255300238796056noreply@blogger.com
blogger_permalink:
  - /2008/10/its-sound-jim-but-not-as-we-know-it.html
categories:
  - 3d auditory pixels
  - c++
  - mocap
  - project
---
I can now play mono wav files with a sample rate of 44100Hz. Stereo should be relativley easy, but isn't really needed. There seems to be one snag, the sample buffers don't seem to be playing on cue, it's jumping to the next buffer far too early. This is leading to whatever song I play through, being turned into a bunch of babbling chipmunks :S <br />Anyone used the Steinberg ASIO SDK? Now any good forums or resources for info on using it? If so let me know.