---
published: true
title: "CQS on MVC - Crosscutting Concerns"
layout: post
---



On my last post I vaguely introduced how CQS and a middle sized MVC project was match that fits like a glove.

In this post I will dig a little deeper and I'll explain how we can add behavior whit the minimal impact.

## The problem

I'm building a web application for my wife ([source here](https://github.com/kappy/LuckyMe)) that can be multi-user. That means that the information and the permissions might vary by the logged on user.

Since most operations are user contextual, I need to access the logged user, and so, I found myself repeating the following lines of code, depending if I was on a Controller context or not:

    Thread.CurrentPrincipal.Identity.GetUserIdAsGuid();
    User.Identity.GetUserIdAsGuid();

Usually when I need to repeat something too many times, I always try to find a better way, because eventually, you might need to change it...

## Who's the responsibility?

So, now that we identified the problem, who should set the current user id? The controller should set it on the request message before call the mediator or should we get it from the `Thread.CurrentPrincipal` inside the Handler?

While the last one is not wrong, doing it like so, would imply that the handler would run on the same executing thread as the web request. I like to keep my handlers the most stateless I can. Remember I said migrating this to a service bus would be a breeze? Keeping your handler stateless, is the first step to achieve that.

Does this mean I opted out for the first choice? Almost... I opted out to pass the current user id on the request message, but like I said before, I don't want to keep writing the same code, all over on every controller action.

## DRY

We connect the handlers that encapsulate our business logic, and the controller with a mediator. That mediator relies on the IoC container to resolve the handler that is responsible to execute the request message. What if we could [decorate](https://en.wikipedia.org/wiki/Decorator_pattern) our handlers with another that inspected the request message and somehow we informed that decorator to inject the current user id into a specific property? Hum... That might do the trick...

I thought also to make this also based on property name conventions, but I decided that it would restrain me in the future.

I started out to create an attribute that I could set up on the property I want.

    [AttributeUsage(AttributeTargets.Property)]
    public class CurrentUserIdAttribute : Attribute
    {
    }

And a request message example using it would be:

    public class GetUserStatistiscsOverview : IAsyncRequest<IEnumerable<EarningsPerGameViewModel>>
    {
        [CurrentUserId]
        public Guid UserId { get; set; }
    }

So far so good... But now we need to decorate our handlers with this

    public class CurrentUserAsyncHandlerWrapper<TRequest, TResponse> : IAsyncRequestHandler<TRequest, TResponse> where TRequest : IAsyncRequest<TResponse>
    {
        private readonly IAsyncRequestHandler<TRequest, TResponse> _innerHander;

        public CurrentUserAsyncHandlerWrapper(IAsyncRequestHandler<TRequest, TResponse> innerHandler)
        {
            _innerHander = innerHandler;
        }

        public async Task<TResponse> Handle(TRequest message)
        {
            var messageType = typeof (TRequest);
            var userProperties = messageType.GetProperties()
                .Where(prop => prop.IsDefined(typeof(CurrentUserIdAttribute), false));
            foreach (var userProperty in userProperties)
            {
                userProperty.SetValue(message, Thread.CurrentPrincipal.Identity.GetUserIdAsGuid());
            }
            return await _innerHander.Handle(message);
        }
    }

Yes, we're using reflection to get the properties of the request message type. It's not the fastest code ever written, it has room for improvement, but for my requirements, it does the job entirely.

## The glue

This is going to deviate from the topic a bit, but putting this all together really depends on your IoC container of choice. In this application I'm using Autofac, and I find it very hard to configure decorators (comparing to other IoCs). Because of that on this project I used a slightly modified version of an helper that I found from [James Thurley](http://stackoverflow.com/users/37725/james-thurley) on this SO [answer](http://stackoverflow.com/a/26018954/4634243).

    public static class AutofacBuilderExtensions
    {
        public static void RegisterHandlers(this ContainerBuilder builder, Type handlerType, params Type[] decorators)
        {
            RegisterHandlers(builder, handlerType);
            for (var i = 0; i < decorators.Length; i++)
            {
                RegisterGenericDecorator(
                    builder,
                    decorators[i],
                    handlerType,
                    fromKeyType: i == 0 ? handlerType : decorators[i - 1],
                    isTheLast: i == decorators.Length - 1);
            }
        }

        private static void RegisterHandlers(ContainerBuilder builder, Type handlerType)
        {
            builder.RegisterAssemblyTypes(Assembly.GetExecutingAssembly())
                .As(t => t.GetInterfaces()
                        .Where(v => v.IsClosedTypeOf(handlerType))
                        .Select(v => new KeyedService(handlerType.Name, v)))
                .InstancePerRequest();
        }

        private static void RegisterGenericDecorator(ContainerBuilder builder, Type decoratorType, Type decoratedServiceType, Type fromKeyType, bool isTheLast)
        {
            var result = builder.RegisterGenericDecorator(decoratorType, decoratedServiceType, fromKeyType.Name);
            if (!isTheLast)
            {
                result.Keyed(decoratorType.Name, decoratedServiceType);
            }
        }
    }

With this in place, It allows me to register my handlers by simply calling:

    builder.RegisterHandlers(typeof(IRequestHandler<,>), typeof(CurrentUserHandlerWrapper<,>));
    builder.RegisterHandlers(typeof(IAsyncRequestHandler<,>), typeof(CurrentUserAsyncHandlerWrapper<,>));

## Final considerations

This is a great way to insert crosscutting behavior without impacting the business logic or code already written. This can also be applyed to insert auditing, logging, cross boundaries transactions, validation, etc..