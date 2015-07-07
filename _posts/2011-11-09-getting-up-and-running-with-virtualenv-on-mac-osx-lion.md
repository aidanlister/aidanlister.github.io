---
layout: post
title: Getting up and running with virtualenv on Mac OSX Lion.
section: developer
summary: |
    A quick walkthrough for using virtualenv on Mac OSX Lion.
---
I recently purchased a new Macbook Air and had forgotten all of the various steps to get <em>virtualenv</em> up and running. Using the native Python packaged with OSX resulted in <em>Could not call install_name_tool -- you must have Apple's development tools installed</em> which I found confusing given that, you know, I have Xcode installed.

Resorting to my old friend <a href="http://macports.com">MacPorts</a>, it took me a few tries and plenty of googling to get up and running. To save you some time should you be in a similar position, here are the commands you will need;

The steps required install Python, easy_install, pip and virtualenv in Mac OSX Lion:
<code>$ sudo port install python27
$ sudo port select --set python python27
$ sudo port install py27-distribute
$ PYDIR=`which python`;
$ echo "export PATH=`dirname $PYDIR`:\$PATH" >> ~/.profile
$ source ~/.profile
$ sudo easy_install -U pip
$ sudo pip install -U virtualenv</code>

There's a bit of magic in there to add pip and easy_install into your path, I found this solution to be nicer than symlinking them to your /usr/bin folder. Once this is done, you are ready to rock:

<code>$ virtualenv --no-site-packages --distribute hooray</code>

Another reader has pointed out this alternative:
<code>git clone https://github.com/gregglind/virtualenv.git
cd virtualenv
git checkout feature/install_name_tool
sudo python setup.py install</code>
