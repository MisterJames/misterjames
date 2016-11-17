title: Bending ASP.NET Core MVC To Your Will
layout: post
tags:
  - ASP.NET Core MVC
  - Web API
categories:
  - Development
authorId: james_chambers
originalUrl: 'http://jameschambers.com/'
date: 2016-09-09 12:46:34
---

The default conventions of ASP.NET Core MVC allow us to easily construct applications without having to worry about the minutiae of wiring up the "how-to" parts that are required in nearly every application that will be built.  There are times when these conventions do not meet your application needs, but you can instruct the framework to work the way you need it to by building your own.

![Bring Your Own Conventions](https://jcblogimages.blob.core.windows.net:443/img/2016/09/bunnies-are-good-at-multiplying.png)


<!-- more -->

## Understanding The Current Conventions
If you've worked inside of the MVC Framework you've either explicitly noticed or been implicitly subjected to some of the conventions at work. These include things like:

 - Names of methods in the startup class
 - Project layout 
 - View search locations
 - Routing expressions as they relate to controller and action names
 - Web API exposing actions that are named as HTTP verbs

These conventions work to remove some of the effort we need to get our application running. Some of them are locked in - we can't change the names of methods that are invoked in the startup class, for instance, as there is an explicit search for `Configure` and `ConfigureServices` - but others can be ammended, removed, and replaced on our whim.

There are four categories of conventions that we're going to briefly discuss here:

| Convention | Interface | Description |
|-------------|-------------------------------|---------------------------------------------------------------------------------------------------------|
| Application (Widest net) | `IApplicationModelConvention` | Provides access to application-wide conventions, allowing you to iterate over each of the levels below. |
| Controller | `IControllerModelConvention` | Conventions that are specific to a controller, but also allows you to evaluate lower levels. |
| Action | `IActionModelConvention` | Changes to action-level conventions can be made here, as well as on any parameters of the actions. |
| Parameter (Smallest scope) | `IParameterModelConvention` | Specific to parameters only. |

As your application loads it will use any conventions that you have added starting at the outter-most field of view, application conventions, then working it's way in through controller and action conventions to parameter conventions. In this way, the most specific conventions are applied last, meaning there is a caveat that if you add a parameter convention by using `IControllerModelConvention` then it could be overwritten by any `IParameterModelConvention`, regardless of the order in which you add the conventions to the project. This is different from middleware, in a sense, because the order of conventions only applies within the same level, and there is a priority on level that you can't adjust.

## Building Your Own Conventions
I wanted to build a convention that celebrated how great rabbits were at math, specifically, multiplication. You know those rabbits! What I did first was to create an interface:

```
public interface IAmRabbit { }
```

...so that I could use it in an attribute:

```
public class RabbitControllerAttribute : Attribute, IAmRabbit { }
```

...which allowed me to apply the attribute to my controller:
```
[RabbitController]
public class HomeController : Controller
{  
  // ...
}
```

...so that I could leverage the attribute in my convention:

```
public class RabbitConvention : IControllerModelConvention
{
    public void Apply(ControllerModel controller)
    {
        if (IsConventionApplicable(controller))
        {
            var multipliedActions = new List<ActionModel>();

            foreach (var action in controller.Actions)
            {
                var existingAction = action;

                var bunnyAction = new ActionModel(existingAction);
                bunnyAction.ActionName = $"Bunny{bunnyAction.ActionName}";
                
                multipliedActions.Add(bunnyAction);
            }
            foreach (var action in multipliedActions)
            {
                controller.Actions.Add(action);
            }
        }
    }

    private bool IsConventionApplicable(ControllerModel controller)
    {
        return controller.Attributes.OfType<IAmRabbit>().Any();
    }

}
```

Now, the great math capabilities of bunnies are available! I loop through all the actions on my controller and create a cloned version of the action with the prefix `Bunny`. So there will be an `Index` action and a `BunnyIndex` and so forth at runtime. Now, you may think that this isn't too relevant at first glance, so I'll leave it as an excercise to the reader to think about how Web API actions might be handled _by convention_ when you have action names that are verbs.   

## Wiring Up Your Application With Custom Conventions
Wiring up the convention is easy...just add it to the conventions collection when you're adding MVC in the `ConfigureServices` method in `Startup`:

```
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddMvc(options =>
        {
            options.Conventions.Add(new RabbitConvention());
        }
    );            
}
```


## Next Steps and Further Reading

Here are some great resources that will help you explore other uses of these interfaces.

**Filip Wojcieszyn's Posts and Community Contributions**
 - [Typed Routing](https://github.com/filipw/Strathweb.TypedRouting.AspNetCore)
 - [Localized Routes](http://www.strathweb.com/2015/11/localized-routes-with-asp-net-5-and-mvc-6/)
 - [Application Model Conventions](http://www.strathweb.com/2015/03/strongly-typed-routing-asp-net-mvc-6-iapplicationmodelconvention/)
 - [Route Prefixes](http://www.strathweb.com/2016/06/global-route-prefix-with-asp-net-core-mvc-revisited/)
 
**Steve Smith's article on feature folders**
 - [Feature Folders on MSDN](https://msdn.microsoft.com/magazine/mt763233)

As you can see, you are not locked into the default behaviours of ASP.NET Core MVC, and you have many surface areas acting as cusomization points for you to exploit.

