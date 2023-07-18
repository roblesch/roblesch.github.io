---
layout: post
title:  "HyperVideo Authoring With Qt6"
date:   2021-12-08
author: Christian Robles
category: blog
published: true
---

For the end of term project in my Multimedia Systems Engineering course, we were tasked with implementing a GUI application for authoring and editing Hyper Videos. The concept for Hyper Video is straightforward - users should be able to open a video file, and create interactable links that when clicked will change playback to the target video at the specified time.

Most of the heavy lifting was in building out the [Qt6](https://www.qt.io/product/qt6) GUI, which was fairly straightforward if a bit tedious. A more tricky challenge was how to sync video playback with audio - a constraint of this assignment was that we were not allowed to zip frames and audio into a convenient media format like mp4, so we had to draw the correct frame on demand. I achieved this by polling every 30ms on a [QTimer](https://doc.qt.io/qt-6/qtimer.html). The timer checks the audio playback position of the [QMediaPlayer](https://doc-snapshots.qt.io/qt6-dev/qmediaplayer.html) and loads the nearest frame, calculated from the target framerate of 30fps.

<img src="/assets/images/hypervideo/hyperauthor.png" alt="author" width="80%"/>
<img src="/assets/images/hypervideo/hyperviewer.png" alt="viewer" width="80%"/>
