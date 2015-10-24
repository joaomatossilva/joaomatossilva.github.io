---
layout: post
published: false
title: Breaking Enum
---

Breaking changes comes in many ways.

Typically we're used to think of breaking changes when we change method signatures, classes, return types or public interfaces. We as programmers know that we should avoid breaking changes because that can affect every little piece of code that depends on the changed part.

While they must be avoided, they're normal part of code evolution. New features sometimes need big changes. Semantic versioning give us some helper lines in order to manage expectations on the library consumers whether a new version might include or not breaking changes. Usually only on major versions increments, a library can include them, but even so, they're should be avoided. There's always some "disappointment" when we have to re-write our code that was already working in order to adapt to a new library version.

## Enougth of talking...

Imagine that we have an application that consists on a core library (named _BreakingEnum.Common.Dll_ on this example), another helper library that can be a third party plugin (named _BreakingEnum.Helper.Dll_) and a frontend application, that puts it all in motion. All of this components are only referenced by the libraries and not by source, as they are provided by different providers.

On _BreakingEnum.Common.Dll_ for this example, only this enum if defined:

    public enum StatusEnum
    {
        Active,
        Disabled
    }

The fact that Active is defined first, is indeed arguable, but it was only to better explain this example. It has no impact, it would only require me to write this example in a different way.
On the _BreakingEnum.Helper.dll_, the library that simulates a third party plugin, It only consists on a method that accept a string and check if the value matches the active state.

    public class StatusHelper
    {
        public static bool IsActive(string statusText)
        {
            var status = (StatusEnum) Enum.Parse(typeof (StatusEnum), statusText);
            var isActive = status == StatusEnum.Active;
            return isActive;
        }
    }

Seems simple enough... For the last part, the application that sets everything in motion:

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

The first few lines, are only used to display the versions used by each assembly to demonstration only. The remaining logic is pretty simple, so I think there isn't a need to explain it.

Running the application give us the result we're expecting.

![Status is active](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/BreakingEnum1_zpsro6jxans.png)

"Status is active", and all assemblies are version 1.0. All is working, life is good, etc...

### But then...

A new version of the core, changed the enum to:

    public enum StatusEnum
    {
        Unknown,
        Active,
        Disabled
    }

At first sight, it seems a pretty inoffensive change, because we're using enums and not magic numbers, so everything should be ok.
Since the _Helper_ library is a 3rd party, it doesn't get update at the same rhythm as the core. Let's try our application again.

![Status is Disabled](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/BreakingEnum2_zps88k0jfgp.png)

"Status is Disabled". Hum... that's a bit unexpected.... Luckily, Helper is an open source project, so we can actually include the source on our solution and try to debug the issue. But even so, because it's a third party, it references the _Common_ by assembly, and the project wasn't been upgraded yet. This means the project targets the _Common_ v1.0.

Let's look at the debug

![Debugging](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/BreakingEnum3_zps9dtekv8s.png)

So.. `status` has been parsed into `Active`. That is right. Next...
The equality `status == StatusEnum.Active` is evaluated as true, ok.. Next...
`isActive` is assigned with the result of the previous equality, that we already evaluated as true.. `false`!!!!!! What?!?!!?!

At first we thought this should be a Visual Studio bug, but even after restart the issue remained.

### After hours of tries and research...

We finally discovered the issue. As it turns out, Visual Studio was right, as it uses the loaded version of the _Common_ assembly at version 2.0. The Helper library was compiled against the version 1.0 of the _Common_ assembly.
The `Enum.Parse()` uses the run-time version of the _Common_, the 2.0. but the static part of the comparison `StatusEnum.Active` is compiled against the version 1.0. Being a constant, the value of that enum is used by the compiler and replaced inline. It doesn't care about the literal part of the Enum (Enums are mostly `int`s by default, but you did knew that already, right?).

So basically, the run-time was comparing `0 == 1` and that is obviously `false`. You can se exacly that, if we cast the values to `int` like in the image below.

![](http://i1299.photobucket.com/albums/ag77/kappyzor/Blog/BreakingEnum4_zpsoqkay6ft.png)

## Conclusion

Changing the value of Enums, can be dangerous, especially when they're used as constants, and let's be real, everyone uses them as constants. I really don't remember to see a `static readonly` enum anywhere (yes, using `static readonly` would not fall into this issue, because the value would be resolved at run-time other than at compile time.

So next time you need to change an Enum, either, set the values explicitly in order not to change the previous values, or don't add any value before, just append new ones. Right @Umbraco? No more stuff like [this](https://github.com/umbraco/Umbraco-CMS/commit/b77521cbc54e45554c0a51a99ebf3baae7555613#diff-326c0207194aecf1d0a252fcc283270e). Let's keep uneeded changes to a minimum.

You can find the entire code used by this example, and try it yourself here: [https://github.com/kappy/BreakingEnum](https://github.com/kappy/BreakingEnum)


