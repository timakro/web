---
layout: software
title: Software and Projects
---

A selection of software I wrote over the years.

## Unique Clan

This is an ongoing project since 2014. A community all around the platformer game Teeworlds, find out more on the [official website](https://uniqueclan.net/). The source code for the website, game server modifications, scripts and database infrastructure can be found on [GitHub](https://github.com/unique-clan).

{% picture software/unique-clan.png --alt Unique Clan %}

<h3 class="date-section">2020</h3>

## Adobe Digital Editions Dockerized

If you own an e-book reader apart from Amazon's Kindle you have likely encountered Adobe's DRM format. You can't download e-books protected by Adobe's DRM directly from the bookstore. Instead you download a small text file with the ACSM extension which you then feed to the Adobe Digital Editions software to download the actual book. Adobe Digital Editions does fortunately run under Linux using Wine but I don't want to bloat my system with closed source DRM software. This project locks Adobe Digital Editions into a Docker container and fully automizes the process of downloading e-books. You will never have to deal with Adobe Digital Editions again instead just pass the ACSM file to a script on the command line to get your book. Find the project source on [GitHub](https://github.com/timakro/adobe_diged_docker).

After I wrote and tested this on Arch Linux it turned out it doesn't run on Debian 10. I went into this assuming with Docker my project would be fully portable. But one should always keep in mind that Docker containers use the hosts kernel. After I tested it and seen it work on Debian 11 my best guess is that it doesn't work with older kernels.

<h3 class="date-section">2019</h3>

## Teeworlds Machine Learning Project

The hype around AI, machine learning and deep neural networks is real. Do you just throw this technology at a real problem and get impressive results? No. I fell for the hype and tried to apply reinforcement learning to the 2D platformer & shooter [Teeworlds](https://www.teeworlds.com/), the results were awful. The process and ideas behind the project are still intersting.

Watch a 5 minute video over at [YouTube](https://youtu.be/sJ6R6zFMhE4) for a little bit of insight. The code with a short analysis of what might have went wrong is on [GitHub](https://github.com/timakro/tmlp). If you want to take a look, maybe there is still hope for the project after all.

{% picture software/tmlp.png mobile: software/tmlp-mobile.png --alt TMLP agent vision %}

<h3 class="date-section">2018</h3>

## superfsmon

A plugin for the Python [Supervisor](http://supervisord.org/) process control system. It restarts processes when files or directories change and should be used for hot loading. It is highly configurable with support for regex.

Installation instructions and source code can be found on [GitHub](https://github.com/timakro/superfsmon). The package is also on [PyPI](https://pypi.org/project/superfsmon/).

<h3 class="date-section">2017</h3>

## xssproxy

This C program listens to the `org.freedesktop.ScreenSaver` D-Bus interface used by programs like Firefox to disable the screensaver---for example when playing videos---and disables the [X11](https://en.wikipedia.org/wiki/X_Window_System) built-in screensaver on request. All the major full-featured desktop environments like GNOME come with an implementation of the D-Bus interface, so this program is meant to be used with lightweight window managers without this functionality.

The source code is on [GitHub](https://github.com/timakro/xssproxy) and I'm maintaining a [Debian Package](https://packages.debian.org/search?keywords=xssproxy).

<h3 class="date-section">2016</h3>

## searchant.vim

This [Vim](https://www.vim.org/) plugin improves the search function to not only highlight all results but also highlight the result last jumped to with a special color.

Installation instructions and the code are on [GitHub](https://github.com/timakro/vim-searchant). The plugin is also on [vim.org](https://www.vim.org/scripts/script.php?script_id=5404).

{% picture software/vim-searchant.png --alt vim-searchant demo %}

## TeeSmash

A mod for the 2D platformer [Teeworlds](https://www.teeworlds.com/), it's inspired by fighting games. I coded this together with my friend [Edgar](https://edgarluque.com/) in C++.

The code is on [GitHub](https://github.com/timazuki/TeeSmash) and there's a [post](https://www.teeworlds.com/forum/viewtopic.php?id=11878) on the Teeworlds forum. I'm hosting [game servers](https://uniqueclan.net/serverstatus/GER) in Germany and Canada if you'd like you give it a try.

<h3 class="date-section">2015</h3>

## DDNet Trashmap

The [DDNet Trashmap](https://trashmap.timakro.de/) service is popular around the map creation community of [DDNet](https://ddnet.tw/), a cooperative 2D platformer. Map creators can upload their map to the web application which will start a personal game server for them to playtest their map alone or with a partner. It's written in Python and PHP.

The source code is maintained on [GitHub](https://github.com/timakro/ddnet-trashmap).

{% picture software/ddnet-trashmap.png mobile: software/ddnet-trashmap-mobile.png --alt DDNet Trashmap logo %}

<h3 class="date-section">2014</h3>

## Towers of Hanoi Robot

Robot built with the [Lego Mindstorms EV3](https://en.wikipedia.org/wiki/Lego_Mindstorms_EV3) robitics kit and programmed with the [python-ev3](https://github.com/topikachu/python-ev3) library moving wooden discs around to solve Towers of Hanoi.

You can watch a video of the robot in action on [YouTube](https://youtu.be/mgyLTcA3iis).

{% picture software/towers-of-hanoi.png --alt Towers of Hanoi robot %}
