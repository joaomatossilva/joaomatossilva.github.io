---
layout: post
published: false
title: Breaking Enum
---

Breaking changes comes in many ways.

Tipically we're used to think of breaking changes when we change method signatures, classes, return types or public interfaces. We as programmers know that we should avoid breaking changes because that can affect every little piece of code that depends on the changed part.

While they must be avoided, they're normal part of code evolution. New features sometimes need big changes. Semantic versioning give us some helper lines in order to manage expectations on the library consumers wheter a new version might include or not breaking changes. Usually only on major versions increments a library can include them, but even so, they're should be avoided. There's always some "disapointment" when we have to re-write our code that was already working in order to adapt to a new library version.

## Enougth if the talking...

Imagine that we have an application that consists on a core library (named BreakingEnum.Common.Dll on this example), another helper library that can be a third party plugin (named BreakingEnum.Helper.Dll) and a frontend aplication, that puts it all in motion. All of this components are only referenced by the libraries and not by source, as they are provided by different providers.

On BreakingEnum.Common.Dll for this example, only this enum if defined:

    public enum StatusEnum
    {
        Active,
        Disabled
    }

The fact that Active is defined first, is indeed arguable, but it was only to better explain this example. It has no impact, it would only require me to write this example in a different way.
On the BreakingEnum.Helper.dll, the library that simulates a third party plugin, It only consists on a method that accept a string and check if the value matches the active state.

    public class StatusHelper
    {
        public static bool IsActive(string statusText)
        {
            var status = (StatusEnum) Enum.Parse(typeof (StatusEnum), statusText);
            var isActive = status == StatusEnum.Active;
            return isActive;
        }
    }

Seems simple enougth... For the last part, the application that sets everyting in motion:

    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Application: {0}", typeof(Program).Assembly.GetName().Version);
            Console.WriteLine("Helper: {0}", typeof(StatusHelper).Assembly.GetName().Version);
            Console.WriteLine("Common: {0}", typeof(StatusEnum).Assembly.GetName().Version);

            var status = StatusHelper.IsActive("Active");
            if (status)
            {
                Console.WriteLine("Status is active");
            }
            else
            {
                Console.WriteLine("Status is Disabled");
            }
        }
    }

The first few lines, are only used to display the versions used by each assembly to demostration only. The remaining logic is pretty simple, so I think there isn't a need to explain it.

Running the application give us the result we're expecting.

"Status is active", and all assemblies are version 1.0. All is working, life is good, etc...

### But then...

A new version of the core, changed the enum to:

    public enum StatusEnum
    {
        Unknown,
        Active,
        Disabled
    }

At first sight, it seems a pretty enofensive change, because we're using enums and not magic numbers, so everything should be ok.
Since the Helper library is a 3rd party, it doesn't get update at the same rythm as the core. Let's try our application again.



"Status is Disabled". Hum... that's a bit unexpected.... Luckly, Helper is an open source project, so we can actually include the source on our solution and try to debug the issue. But even so, because it's a third party, it references the Common by assembly, and the project wasn't been upgraded yet. This means the project targets the Common v1.0.







