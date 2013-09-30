---
published: false
title: "Code Style - I went to war... and I lost..."
layout: post
---

## Code Style - I went to war... and I lost...

My post today will be about indenting style in C#. There are essentially two styles that were broadly adopted:
- K&R
- Allman

The most notorious difference between the two is where to put the curly braces.
K&R opens the curly brace in the same line of the block.

    if (started) {
        stop();
    } else {
        start();
    }

Allman instead, always opens a curly brace in a new line.

    if (started)
    {
        stop();
    }
    else
    {
        start();
    }

As you can see, it's evident the difference in the number of total lines and most important vertical space.

# I went to war...

I'm a clear fan of K&R style. I think it has a clear distinct block structure and doesn't waste vertical space.
Why vertical space you say... A good monitor has a lot of vertical lines. Almost too many to be tracking each and every one of it. Well... That's true. The problem is, from your company, you almost never get what you want. Especially on computer hardware.

On my actual company, I got the lowest (on lacking of better word) computer. It's an i5 with 4Gb ram (shared with gpu), and a stunning display with 1366x768 resolution.
Visual Studio takes about 1 full minute to load (without any solution).
It's clearly not a developer machine.

That said, I still find important not to waste space. Even well written production software it's not written on 5 or 6 lines of code. If you still waste lines (an `else` block uses 3 lines...) screen space falls off quite fast.

# And I lost...

A long time ago, I had an idea to use github to analyze a few open source projects to check the trend of indent styles used. As pointed out by Phill Haack, a recent [project](http://sideeffect.kr/popularconvention/#c#) did just that.  

The numbers speak for themselves.
- Allman style: 88%
- K&R style: 11%
- Other 1%

As it turns out, vertical space is a thing of the past. At least for someone... not me tho.