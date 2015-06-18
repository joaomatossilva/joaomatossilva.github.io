---
published: true
title: Unit tests beyond Test Driven Development
layout: post
---


I'm not really a fan of Test Driven Development (TDD). All that write a failing test before the feature, then implement it until the test passes, gives me a feeling of the amount of hours spent on writing code without need.

That said, it doesn't mean I don't like Unit tests. Quite the opposite. I like them very much, and I think they're a very useful tool on your belt.

This is an example of how I sometimes use Unit tests on my day to day. 

In one of those pesky summer days that almost resemble winter, for being so cold and rainy, I received a bug report saying that the user wasn't unable to subscribe his child named "João" that only has 2 months old, in order to start receiving the newsletter with baby stuff promotions. He tried with different browsers, but ended up always on the error page.

Well, say no more... Let's put it to the test (pun intended).

    [TestClass]
    public class BugReport_102
    {
        /* 
         * User complains that he cannot subscribe his child with name "João" and age 0 
         */

        [TestMethod]
        public void CanInvokeSubscribeWithBugReportData()
        {
            var userController = new UserController();
            var result = userController.Subscribe("joao@email.org", "João", 0) as ViewResult;
            Assert.AreEqual("Subscribe", result.ViewName);
        }
    }
    
Pretty simple test. We just invoke our controller with the request data. For demo only, we don't need to mock any dependency. That would fall out the scope of this article.

![unit test failing](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/bugreport_test_fail_zps6qvorojf.png)

There you go. Bug report replicated successfully. Now that we replicated the problem, it's easier to pin point the problem. Apparently, someone (the virtual me) forgot to double check the age when performing some rate per age.

![unit test pass](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/bugreport_test_pass_zpsaudqhnsw.png)

Done. Bug fixed. the best thing is that if you keep the test, you're ensuring that you don't have regressions on this particular bug.

Some may argue that this is very similar to TDD. I honestly disagree, mainly in the point that I don't like to write code without knowing if ever will going to be used.

## Is TDD a bad thing?

Of course not. It's a very good way to minimize bug within your application, and so, keeping the quality very high. 

It's only my personal opinion that sometimes feels like we're writing lots of code, for simple stuff. 

One might argue that writing the test before is cheaper than writing it afterwards, and maybe they're right...
