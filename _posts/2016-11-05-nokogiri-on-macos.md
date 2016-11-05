---
layout: post
section-type: post
title: Nokogiri on macOS - ultimate fix
category: Quicktips
---

Today just a short reminder to my future self. If you ever would have problems with installing nokogiri in your freshly installed or updated macOS, follow these steps:

1. Remove any lines containing `--use-system-libraries` from `~/.bundle/config`. There is no need to have this anymore
2. Install the newest Xcode from App Store
3. Run `xcode-select --install` to install additional tools required for compilation.

Bonus: these steps should fix your Homebrew, too!