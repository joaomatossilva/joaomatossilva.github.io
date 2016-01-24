---
layout: post
published: true
title: "Single Entry, Single Exit... No wonder it's still single..."
---


During an internal code review, I asked a colleague of mine why was he using a variable inside a method just to hold the result of it and return it in the end. Why not just return immediately as soon as it knows the output?
He told me that he was trying to follow the "Single Entry, Single Exit Principle (SESE)".

My eyes rolled a bit... What was he talking about? The answer was a google away.
In short, a routine should start only on one single location and exit on another single location.

Well... Let's see the entry first... On modern iterative programming languages (like C#), all methods always start on the first line of the method declaration. This is almost true, since weird stuff can be accomplished with the `goto` statement but I can't recall the last time I have ever used it (on c#) or even see it being used outside decompiled code. So.. I think we can assume the we're using single entry by default.

Single Exit... this is where the issues arise. This mean that we can only use `return` once? This should have been called "Single Return" (k). Does this means we can't use a `throw new Exception("bla bla bla")`? 
The issue I have with this, is that I like to return early. I like to keep the level of nesting and indentation inside a method to the minimum.

Take a look at this example. A simple routine (for demo proposes) to calculate the length of any string except for spaces until it finds the work `stop`.

    public int CalculateStringLength(string text)
	{
		int length = 0;
		if(!string.IsNullOrEmpty(text))
		{
			var indexOfStop = text.IndexOf("stop");
			if(indexOfStop > 0)
			{
				text = text.Substring(0, indexOfStop);
			}
			if(indexOfStop != 0)
			{
				text = text.Replace(" ", "");
				length = text.Length;
			}
		}
		return length;
	}

I see there 2 level of indentation. This was a dummy method just built only for this example. A real one could have pretty worst levels of nesting...

Let's see how the same exercise would look without the single "return" rule.

    public int CalculateStringLength(string text)
	{
		if(string.IsNullOrEmpty(text))
		{
			return 0;
		}
		var indexOfStop = text.IndexOf("stop");
		if(indexOfStop == 0)
		{
			return 0;
		}
		if(indexOfStop > 0)
		{
			text = text.Substring(0, indexOfStop);
		}
		var textWihoutSpaces = text.Replace(" ", "");
		return textWihoutSpaces.Length;
	}

Less indentation improves readability and maintainability. In this example we kept the nesting level only to 1.

So I guess this is why this Principle is still Single after all these years. Maybe it should create a profile on Match dot com.
