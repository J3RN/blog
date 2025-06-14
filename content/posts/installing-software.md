---
date: 2025-05-04
title: "Installing Software"
description: "Why is this so hard?"
draft: true
---

When I first started building software circa 2011, my understanding was that software was installed via `apt` (on Ubuntu).  If you wanted Firefox, you run `sudo apt install firefox`.  If you wanted the Apache web server, you run `sudo apt install apache`.  The first programming language I wrote for money was PHP, and these two (Firefox and Apache) alongside `php-common` were enough to get going.

*However*, I didn't write PHP forever.  A few years later, I began developing with Ruby.  Now, to be clear, you absolutely could then and can today run `sudo apt install ruby` and get a functioning Ruby interpreter.  That said, this would lead to a number of issues:

1. You receive whichever version of Ruby your distro (e.g. Ubuntu) happens to be shipping at that time.  In 2013, that version was several years out-of-date.
2. Installing **Ruby** packages (termed "gems") required root access, because the directories where Ruby (also installed as root via `sudo apt ...`) looked for them were all owned by root.

Thus, my received wisdom at the time was to **not** use `apt` to install Ruby, but to use either [RVM] (Ruby Version Manager) or the combination of [rbenv] and [ruby-build].  The added benefit of RVM and rbenv was that not only could they install the *latest* Ruby, but they could also install *arbitrary* Ruby versions and allow you to switch the "active" one (the one used when you type `ruby` on your command line) at will.  This was very beneficial to a Ruby programmer in 2013 (and likely still is today) because Ruby versions are not likely to be backwards-compatible, and so a project that was started on Ruby 2.1 will likely only run on Ruby 2.1 unless effort was made to port it forward to a newer version or (Challenge Level: Extreme) write it in such a way that it runs on a variety of Ruby versions.

This was the very beginning of the long and winding road that leads to today's post.

You see, we have a few layers of package management now:
1. Most software via `apt`.  I'm still running `sudo apt install firefox`.
2. RVM/rbenv/etc.  These install *Ruby*.
3. [Bundler].  This is a Ruby tool that installs Ruby packages (gems).

This is, of course, only *one* programming language, and many of them have exactly this structure with their own tooling.  For instance, the recommended way to install [Rust] and manage different Rust versions is via [Rustup]; Rust packages ("crates") are installed via [Cargo].

- homebrew
- asdf/mise
- Nix

[RVM]: https://rmv.io
[rbenv]: https://github.com/rbenv/rbenv
[ruby-build]: https://github.com/rbenv/ruby-build
[Rust]: https://rust-lang.org
[Cargo]: https://doc.rust-lang.org/cargo
