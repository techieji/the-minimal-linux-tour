# Expanding the System

Now that we have the kernel set up, we'll be starting to move towards
using modern tools and accepted practices. However, we'll not be
implementing the complete Linux booting process until some time
after. For now, we'll focus on some important parts of your interaction
with Linux without worrying about exactly how it fits into how modern
systems work.

These following sections are where you're going to see some familiar
things, or maybe files that you've heard of but never really understood:
`fstab`, `inittab`. We'll also skim networking for a bit, a topic that
people -- me included! -- don't really understand unless they have
a particular interest in the topic.

Let's start by modernizing our init process to reflect something that
actual distributions use.
