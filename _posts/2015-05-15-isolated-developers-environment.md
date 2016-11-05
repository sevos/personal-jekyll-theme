---
layout: post
section-type: post
title: Isolated developers.environment
---

Few months ago I attended Ruby User Group meetup in Berlin at Wimdu's office. Jan Schulte was speaking about his development environment enclosed in a virtual machine managed by Vagrant. The idea was always appealing to me and I tried it multiple times. Unfortunately I stumbled upon blockers which brought me always back to native environment. One of the Issues I raised in Q&A part was: how to keep the code  editor responsive while accessing files on virtual machine. 

Almost month later I decided to try to connect Elixir/Phoenix framework with the newest Ember.js and create small, deployable prototype. It was an amazing weekend hackathon I will describe in later post. The problem occurred on Monday, when it turned out that not every stack has so great tools like RVM and bundler and I need to somehow bring back the old version of Node.js to my computer (NVM was sort of not working for me). Bummer! After fixing the issue I started to think how to avoid it in the future. What if I would want to test another version of Erlang or, even more tricky, see how the newest version of PostgreSQL is doing?

Naturally I turned to VMs and Vagrant. I had few major concerns though:

- How much of performance I would have to sacrifice?
- How to ensure that my shell and Emacs will behave as closely to those on my Mac as possible?
- VMs take a lot of storage space
- And tmux is not really so convenient for everyone and can collide with Emacs key bindings

First, I decided to benchmark the three major VMs solutions compatible with Vagrant: VirtualBox, Parallels and VMWare. I decided to not apply any optimization hacks nor to tune the database. I ran simply the RSpec test suite from the project I work on. Thus following results are not representative and should not be taken as such (YMMV). When making your decision you might want to consider the cost factor (in brackets). Here are my results:

- Native: approx. 4 min 20s (no costs)
- VirtualBox: approx. 5min 30s (free)
- Parallels: approx. 5min 10s (around 70€ for the software)
- VMWare: approx. 4min 50s (around 70€ for the software and 70€ for the Vagrant plugin)

I went with the trial version of VMware to see how much I can get from the VM. 

As the next step I reviewed my shell and Emacs configuration. I was able to trim the first down to few files, which can be easily installed inside the machine. I just had to make few adjustments like to make the ssh key forwarding work in tmux. The point of using tmux is to not require new ssh connection, when you need just another she'll inside the VM. Normally I use iTerm2, so I could benefit from deep tmux integration, which works really great these days. When I connect to the VM, I just launch new tmux session and then the regular iTerm shortcuts open tabs and windows inside the machine! Actually, thanks to iTerm profiles I could change the appearance of the tmux session and by the background color I can tell where particular session belongs to.

Okay, you might ask, how to preserve such environment? Indeed, the VM images are not the only option. There are few provision era supported by Vagrant out of the box. My favorite is Ansible. I coded the whole development environment in it and I am able to destroy and recreate it in 30 minutes (while I am doing something else). Quit hint: you really do want to lock on particular versions of the software you are installing, otherwise you'll be exposed to nasty surprises.

I am successfully working in an isolated environment since few weeks. I discovered that it has several, either technical and lifestyle, benefits. Many start with the statement that the strongest advantage of using provisioned virtual machines as a development environment  is the possibility of sharing it among your team. I don't think so. The biggest advantage is the ability to separate your work, hobby and fun, while still possessing only one machine.

Pros:

- Isolated, easy to recreate environment
Linux environment is certainly closer to your production servers than Yosemite and homebrew
- Positive impact of shutting the machine down after work - it's a clear message to my mind that it's time for other activities even if they're involving using computer
- Possibility to temporarily remove your work environment to make some space when you go on holidays or decide to play Diablo over the weekend
- Possibility to easily manage development domains (*.dev in my case) or even ability to set up SSL in dev. How cool is that? I know, you can achieve it by installing nginx on OS X, but it gets hard to manage over time. There is also [pow](http://pow.cx/), but it was too simple for me since I don't always make ruby projects.

Cons:

- Noticeable performance cost
- Money cost if you go with the paid software
- Your colleagues want to reuse it, but they might need to setup own stack eventually: the understanding and feeling as owner of your dev env is a very important thing.
- It works nice really only if you editor works in the terminal

Links:

- https://github.com/sevos/vagrant-devenv - the skeleton of my development environment