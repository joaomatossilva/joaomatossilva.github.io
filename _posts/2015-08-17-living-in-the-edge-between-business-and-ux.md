---
layout: post
published: false
title: Living in the edge between business and ux
---

The line that dictates the separation of user experience (ux) and the business logic is not always clear.
Remember me talking about multi layers applications? This one I will talk about some code that is near between the business and the presentation layer.

## The scenario

Let's start from the presentation. Here it's where the ux is implemented. In this case, we're doing a on/off button, specifically, a toggle button.

![](https://upload.wikimedia.org/wikipedia/commons/2/2a/Ringing_the_elevator_alarm.jpg)

So.. How should we implement this... Let's follow the dummy approach, no logic on the presentation. We have a toggle button, so let's make a form to submit a toggle action.

Controller (UX):

    public ActionResult Toggle()
    {
    	var statusModel = _myToggleService.GetStatus();
        return View(statusModel);
    }
    
    [HttpPost]
    public ActionResult Toggle(FormCollection form)
    {
        var statusModel = _myToggleService.Toggle();
        return View(statusModel);
    }

View (UX):

    @using(Html.BeginForm("Toggle")){
    	<span>Status: @Model.State</span>
        <input type="submit" value="Toggle" />
    }

Service (Business):

    public StatusModel GetStatus()
    {
    	/* Get entity code (or anything else...)*/
        return new StatusModel { Status = entity.Status };
    }
    
    public StatusModel Toggle()
    {
        /* Get entity code (or anything else...) */
        entity.Status = !entity.Status; //yes, it's a boolean
        /* Save entity (or anything else...) */
        return new StatusModel { Status = entity.Status };
    }

Ok.. So the Code works.. We can make Unit tests for this and they'll all pass. So what is the issue here? Why are we even talking about this?

## The web is not a lonely place

You're not alone on the web. When we design a product to be consumed on the web, intranet, or anyone's browser, we take the chances of having more than one user/browser/tab open on the same form.

Imagine 2 users wanting to turn off the toggle switch. First one does indeed shut it down, but the next one turns it on again. Worst, the first user got the feedback of the button has been shut down, but the second gets the feedback that he just turn it on. What?

So what's the solution? Instead of making the presentation dumb, why not make the actual business dumb? I mean, make the business layer explicitly set the desired state. For that, now we need to pass on the desired value through the form. This will result in the following:

Controller (UX):

    public ActionResult Toggle()
    {
    	var statusModel = _myToggleService.GetStatus();
        return View(statusModel);
    }
    
    [HttpPost]
    public ActionResult Toggle(bool state)
    {
        var statusModel = _myToggleService.SetStatus(state);
        return View(statusModel);
    }

View (UX):

    @using(Html.BeginForm("Toggle")){
    	<span>Status: @Model.State</span>
        <input type="hidden" name="state" value="@(!Model.State)" />
        <input type="submit" value="Toggle" />
    }

Service (Business):

    public StatusModel GetStatus()
    {
    	/* Get entity code (or anything else...)*/
        return new StatusModel { Status = entity.Status };
    }
    
    public StatusModel SetStatus(bool state)
    {
        /* Get entity code (or anything else...) */
        entity.Status = state;
        /* Save entity (or anything else...) */
        return new StatusModel { Status = entity.Status };
    }

So, everything still works, tests are green, business layer is dumb, stateless, and if a second user (for most unlikely to happen) tries to turn off, even if the first user already turn it off, it will stay off.

## Conclusion..

This was a simple scenario, but similar issues happen every day. It's not easy to separate what is presentation logic and what is in fact business logic. Not every **if** is business logic. The world is not back and white, there are some grey areas as well.
