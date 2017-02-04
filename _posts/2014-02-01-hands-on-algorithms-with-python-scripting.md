---
layout: post
title: "Hands on algorithms with Python scripting"
description: "Writing a toolset of useful scripts for the Linux shell"
category: Programming
tags: [Python, Scripting]
---
{% include JB/setup %}

Two days ago I looked for a good project to partecipate with the purpose of enhancing my skills about writing good alghoritms. So I moved my very first steps searching a good starting point in this matter; as a gift of the holy fate I received a mail from coursera reminding me the forthcoming of a new session of the Algorithms I course by the Princeton University College, but this is not what I was looking for, at least not completely. For the gods sake I then recalled the presence of an unfinished project of my friend @origama, here on GitHub, under the [Zencoders group](http://www.zencoders.org/) (read the manifesto and feel free to take part). This project moved from the original idea of @etr to make a [set of libraries in C++](https://github.com/zencoders/algoritmica).
<!--more-->

I do not know C++ programming language, so I opted for **algoritmica.sh**, wich is no less than a parallel project conceived in Bash. I then decided to drag in this endavour, encouraged by the possibility of using these *experiments* shortly afterwards in an upcoming project. With the best expectations I started studying the code made by origama, but a soft voice from my soul was suggesting to stop and choose another way («follow the force» and «use a Jedi technology» were the recurring phrases). Then I bumped into [this](http://magazine.redhat.com/2008/02/07/python-for-bash-scripters-a-well-kept-secret/):

> [...] let’s get away from the nonsensical scripts and actually write something useful. Python has a massive standard library that can be used by simple importing modules.

I suddendly realized that the right way was Python. So here the real story starts. I resumed this [doc page](http://docs.python.org/2/library/optparse.html) but the highlighted message in red on the top of that page redirects me on `argparse`, rather than `optparse`. My problem on that precise moment? I use Ubuntu 10.10 as my operating system and I have Python 2.6: I should have to [install the new version of the interpreter](https://gist.github.com/sentenza/8769261)... There was also an alternative by using a `virtualenv`

     virtualenv --python=/usr/bin/python2.7 algopy



[Why virtualenv?](http://pypi.python.org/pypi/virtualenv)

>The basic problem being addressed is one of dependencies and versions, and indirectly permissions. Imagine you have an application that needs version 1 of LibFoo, but another application requires version 2. How can you use both these applications? If you install everything into /usr/lib/python2.4/site-packages (or whatever your platform's standard location is), it's easy to end up in a situation where you unintentionally upgrade an application that shouldn't be upgraded.


[Eriol](http://mornie.org), from the Catania GLUG, pointed me in the right direction suggesting to include a setup.py with the necessary dependencies and also to give a message to the user with a try-except on the import with an error message linking to <https://pypi.python.org/pypi/argparse>.

``` python
try:
     import argparse
except ImportError, e:
     sys.stderr.write(e)
     sys.stderr.write("You must install argparse: https://pypi.python.org/pypi/argparse")
```

### Resources

- [Python for Bash scripters: A well-kept secret](http://magazine.redhat.com/2008/02/07/python-for-bash-scripters-a-well-kept-secret/)
- [Python Scripts as a Replacement for Bash Utility Scripts](http://www.linuxjournal.com/content/python-scripts-replacement-bash-utility-scripts)
- [Chromium Python style guidelines](http://www.chromium.org/chromium-os/python-style-guidelines)
