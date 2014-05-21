---
published: false
title: IEnumerable Bullying
layout: post
---

I know I didn't post anything for quite a while. I'm sorry for that, but I'm having a few events that absorb out my time entirely.


I found this little pearl lying around on a past project that I was called out to save from disaster. Some classes and properties were renamed to protect the business value of the code, but the algorith remains the same.
![IEnumerableBullying](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/GetViewModel_zpsa791edf2.png)

It's so wrong at so many levels I can't even know were to begin. At least some points are worthy to point out like:
- Why flatten the list in the first place? Having 10 properties named the same, doesn't make sence. What if tomorrow the business scenario changed from 10 questions to 100?
- Ok, It really needed to flatten. Shouldn't Reflection be an option? With refection just add an extra property with the right index and it should be filled with the right value.
- Why on earth enumerating the results over and over again? How many times `GetEnumerator()` and `MoveNext()` are called? The right way only 1+10 times per item were needed, but this way we have like 1+(10*10*n) times per item. If this isn't abusing the IEnumerable, I don't know what is.
- What really kicked me off my seat was the last statement `list.AsQueriable()`. slick....