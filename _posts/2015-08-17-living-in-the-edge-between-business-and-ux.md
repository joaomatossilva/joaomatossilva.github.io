---
layout: post
published: false
title: Living in the edge between business and ux
---

The line that dictates the separation of user experience (ux) and the business logic is not always clear.
Remember me talking about multi layers applications? This one I will talk about some code that is near between the business and the presentation layer.

## The scenario

Let's start from the presentation. Here it's where the ux is implemented. In this case, we're doing a on/off button, specificly, a toggle button.

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


