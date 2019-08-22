---
title: "Fsduplicates"
date: 2016-06-26T19:49:00-04:00
draft: false
description: "Help duplicate songs with audio fingerprinting"
showcomments: false
showpagemeta: false
categories:
 - projects
tags:
 - macos
 - open source
---

![fsduplicates](/img/fsduplicates.png)

*Discontinued project. Developed in 2016*

Command line tool to detect duplicate songs in OS X/macOS based on their Audio Fingerprints.

fsduplicates helps you find song duplicates recursively in any specific directory and output its findings in a different directory. With the generated results you can later analyse and take action on duplicate songs. The results are in plain text files, which you can analyse using any tool, like Unix standard commands, or fsduplicates’ tools.

fsduplicates works by contacting AcoustID (acoustid.org) for each song generating an unique ID for each song. If two songs have the same ID, they are the same song. Internally, fsduplicates uses fpcalc, a command line tool developed by Chromaprint to generate fingerprints.

<hr>

Available source code (Warning - it's bad): [Github](https://github.com/AndyIbanez/fsduplicates)