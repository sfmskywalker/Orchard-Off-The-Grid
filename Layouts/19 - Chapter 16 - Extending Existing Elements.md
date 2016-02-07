## Extending Existing Elements
In the previous chapters, we learned how to create new types of elements by creating new element classes and harvesters.

In this chapter, instead of creating new element types, we will see how we can extend existing ones by creating additional *element drivers* and *element handlers*.

By creating drivers and handlers, we can store additional information for an element, provide additional UI to manage that information, and provide additional behavior.

Extending elements with additional information does not work the same as with content types. The content type system is one of *composition*. Elements on the other hand are *atomic*, which means one does not simply stick additional properties on them. So how can we add additional information then?

The answer to that question is quite simple: elements have a *Data* dictionary, which is a public property defined on the `Element` base class.

### Using Multiple Drivers
Being able to store additional information on an element is one thing. Another thing is providing additional UI on the element editor dialog screen, which we can do by implementing our own driver for a given element type.

As is the case with content parts and content fields, elements too can have more than one driver. This is what enables us to provide additional UI. We could write drivers for a specific element type, or for a base class that is shared by multiple element types.

### Using Multiple Handlers
In addition to writing additional drivers, we can also implement additional *element event handlers*. In fact, the element driver system is implemented by such an element handler. This means that we can extend very specific element types as well as any other element types based on whatever criteria we like.

### Trying It Out: Extending Elements
If you know how to write an element handler or element driver, you already know how to extend existing elements, as the process is the same. The primary difference is that we don't "own" the element, and that we can't implement strongly-typed properties on them that access their `Data` dictionary.

To see how it all works, we'll implement a common driver for any element type and provide an editor UI for the a custom property called `"FadeIn"`. This will demonstrate how to add additional properties to existing elements. This property will be a simple boolean value, and when set to `true`, will append a certain CSS class that will cause the element to fade in on page load. 

#### Creating The CommonElementDriver
The driver's goal is to provide an additional editor for all elements, so we'll create a driver for the base `Element` class:

```
public class CommonElementDriver<Element> {}
``` 

That will cause the driver to execute for any type of element.

Next, we'll implement the `OnBuildEditor` method to display and handle post backs. For that we'll create a view model to pass in and receive back the `"FadeIn"` value. We'll also assign the editor UI to a tab named **Visibility**.

To simplify working with the `"FadeIn"` property, we'll create two extension methods on the `Element` class so we don't have to deal with converting to and from strings all the time. The extension methods look like this:

```
using System;
using Orchard.ContentManagement;
using Orchard.Layouts.Framework.Elements;
using Orchard.Layouts.Helpers;

namespace OffTheGrid.Demos.Layouts.Helpers {
    public static class ElementExtensions {
        private const string FadeInKey = "FadeIn";
        private const string DefaultFadeInValue = "false";

        public static bool GetFadeIn(this Element element) {
            return XmlHelper.Parse<bool>(element.Data.Get(FadeInKey, DefaultFadeInValue));
        }

        public static void SetFadeIn(this Element element, bool value) {
            element.Data[FadeInKey] = XmlHelper.ToString(value);
        }
    }
}
```

> If the C# team ever adds support for *extension properties*, we could replace these methods with actual property methods. Now wouldn't that be sweet? 

As you can see, the extension methods work with the element's `Data` dictionary directly and set and get the value stored by the `"FadeIn"` key.

The view model is defined as follows:

```
using System;

namespace OffTheGrid.Demos.Layouts.ViewModels {
    public class ElementViewModel {
        public bool FadeIn { get; set; }
    }
}
```

With that in place, we can implement the driver as follows:

```
using OffTheGrid.Demos.Layouts.Helpers;
using OffTheGrid.Demos.Layouts.ViewModels;
using Orchard.Layouts.Framework.Drivers;
using Orchard.Layouts.Framework.Elements;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class CommonElementDriver : ElementDriver<Element> {
        
        protected override EditorResult OnBuildEditor(Element element, ElementEditorContext context) {
            // Initialize the view model with existing data.
            var viewModel = new ElementViewModel {
                FadeIn = element.GetFadeIn()
            };
            
            // Model bind the view model if an Updater is provided.
            if(context.Updater != null) {
                if (context.Updater.TryUpdateModel(viewModel, context.Prefix, null, null)) {
                    element.SetFadeIn(viewModel.FadeIn);
                }
            }

            // Create the editor template shape.
            var visibilityEditorTemplate = context.ShapeFactory.EditorTemplate(
                TemplateName: "Elements/Common.Visibility",
                Model: viewModel,
                Prefix: context.Prefix);

            // Specify the position of the editor shapes.
            // This is key to assigning editor templates to a tab.
            visibilityEditorTemplate.Metadata.Position = "Visibility:1";

            // Return the editor shape.
            return Editor(context, visibilityEditorTemplate);
        }
    }
}
```

Very straightforward code, but it does demonstrate how to assign an editor template to a tab.

Now all that's left is actually implementing the editor template's view, and for this demo's purposes, implement the `"FadeIn"` property so that it will add a certain CSS class and a stylesheet that defines the rule for that CSS class.

The *Views/EditorTemlates/Elements/Common.Visibility.cshtml* view:

```
@model OffTheGrid.Demos.Layouts.ViewModels.ElementViewModel
<fieldset>
    <div class="form-group">
        @Html.CheckBoxFor(m => m.FadeIn)
        @Html.LabelFor(m => m.FadeIn, T("Fade In").ToString(), new { @class = "forcheckbox" } )
        @Html.Hint(T("Causes the element to fade in on document ready (using CSS animations)."))
    </div>
</fieldset>
```

The result should look something like this:

![](./figures/fig-112-visibility-editor-tab.png)

#### Implementing the FadeIn Behavior ####
To make the `"FadeIn"` setting actually do something, we need to do the following:

1. Append a CSS class to the element's tag being rendered;
2. Provide the CSS rule for that class.
	1. Which means we need a stylesheet;
	2. Get that stylesheet included on the page;

Appending the CSS class is easy. All we need to do is implement the `OnDisplaying` method in the driver, and add a CSS class to the `Classes` list of the element shape:

```
protected override void OnDisplaying(Element element, ElementDisplayingContext context) {
    if (context.DisplayType == "Design")
        return;

	if (!element.GetFadeIn())
        return;

    context.ElementShape.Classes.Add("auto-fade-in");
}
```

Notice I added a check on the display type. Although this check is not strictly necessary, I think it's cleaner if we don't add the CSS class when the element is rendered in design mode.

And of course we need to check for the value of the `"FadeIn"` property; we only add the CSS class `"auto-fade-in"` if it's set to `true`.

Next, create the following LESS file: *Assets/Elements/Common/Style.less*.
And type in the following CSS (copied form the following StackOverflow answer: [http://stackoverflow.com/questions/11679567/using-css-for-fade-in-effect-on-page-load](http://stackoverflow.com/questions/11679567/using-css-for-fade-in-effect-on-page-load))

```
.auto-fade-in {
    -webkit-animation: fadein 2s; /* Safari, Chrome and Opera > 12.1 */
       -moz-animation: fadein 2s; /* Firefox < 16 */
        -ms-animation: fadein 2s; /* Internet Explorer */
         -o-animation: fadein 2s; /* Opera < 12.1 */
            animation: fadein 2s;
}

@keyframes fadein {
    from { opacity: 0; }
    to   { opacity: 1; }
}

/* Firefox < 16 */
@-moz-keyframes fadein {
    from { opacity: 0; }
    to   { opacity: 1; }
}

/* Safari, Chrome and Opera > 12.1 */
@-webkit-keyframes fadein {
    from { opacity: 0; }
    to   { opacity: 1; }
}

/* Internet Explorer */
@-ms-keyframes fadein {
    from { opacity: 0; }
    to   { opacity: 1; }
}

/* Opera < 12.1 */
@-o-keyframes fadein {
    from { opacity: 0; }
    to   { opacity: 1; }
}
```

Also add the following configuration to *Assets.json*:

```
{
    "inputs": [
        "Assets/Elements/Common/Style.less"
    ],
    "output": "Styles/CommonElement.css"
}
```

The tricky part is registering this stylesheet. Since this driver executes for all shapes being rendered, there's no single shape template that we can use to include our stylesheet, and we definitely don't want to override every element shape template out there. For one thing, that's a lot of work. Not to mention that whenever a new element is added to the system by some third party module, that element won't be able to take advantage of our nifty Fade In feature.

So the best thing we can do is define a resource manifest for the stylesheet, and *require* it using the `IResourceManager`.

The resource manifest provider looks like this:

```
using Orchard.UI.Resources;

namespace OffTheGrid.Demos.Layouts.Handlers {
    public class CommonElementResourceManifest : IResourceManifestProvider {
        public void BuildManifests(ResourceManifestBuilder builder) {
            var manifest = builder.Add();
            manifest.DefineStyle("CommonElement").SetUrl("CommonElement.min.css", "CommonElement.css");
        }
    }
}
```

With that in place, we can update our driver's `OnDisplaying` method by adding the following line:

```
_resourceManager.Require("stylesheet", "CommonElement");
```

The `_resourceManager` is a private field on the driver's class and injected as follows:

```
private readonly IResourceManager _resourceManager;

public CommonElementDriver(IResourceManager resourceManager) {
    _resourceManager = resourceManager;
}
```

With that in place, we can now have any element we like fading in. Except for the elements Canvas, Grod, Row and Column, since they are defined as not having an element editor. Which is a bummer, really. Hopefully in some future iteration we get more control over dynamically setting the `HasEditor` setting for those elements.

Now, you may be thinking that this is a lot of effort where we could simply just specify the CSS class directly using the HTML Class setting on each element, and you would be right. However, this excercise demonstrates how we can create more intuitive settings for the user. It's much easier to tick a checkbox than remembering to set a CSS class called `"auto-fade-in"`, for example.

### An Alternative Implementation
We have just seen how to extend an element using an element driver. But what if you wanted to extend only elements that meet a different criteria than being of a certain type? For example, what if we only wanted to expose the `"FadeIn"` property for the following elements: *Html* and *Image*?

Since an element driver targets a specific element type class, we can't use that.
So we'll just have to implement an element handler directly instead.

The following code listing demonstrates exactly that:

```
using System;
using System.Collections.Generic;
using OffTheGrid.Demos.Layouts.Helpers;
using OffTheGrid.Demos.Layouts.ViewModels;
using Orchard.Layouts.Elements;
using Orchard.Layouts.Framework.Display;
using Orchard.Layouts.Framework.Drivers;
using Orchard.Layouts.Services;
using Orchard.UI.Resources;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class CommonElementHandler : ElementEventHandlerBase {
        private readonly IResourceManager _resourceManager;

        public CommonElementHandler(IResourceManager resourceManager) {
            _resourceManager = resourceManager;
        }

        private IList<Type> SupportedTypes = new List<Type> {
            { typeof(Html) },
            { typeof(Image) },
        };

        public override void BuildEditor(ElementEditorContext context) {
            var element = context.Element;
            var elementType = element.GetType();

            if (!SupportedTypes.Contains(elementType))
                return;
            
            // Initialize the view model with existing data.
            var viewModel = new ElementViewModel {
                FadeIn = element.GetFadeIn()
            };

            // Model bind the view model if an Updater is provided.
            if (context.Updater != null) {
                if (context.Updater.TryUpdateModel(viewModel, context.Prefix, null, null)) {
                    element.SetFadeIn(viewModel.FadeIn);
                }
            }

            // Create the editor template shape.
            var visibilityEditorTemplate = context.ShapeFactory.EditorTemplate(
                TemplateName: "Elements/Common.Visibility",
                Model: viewModel,
                Prefix: context.Prefix);

            // Specify the position of the editor shapes.
            // This is key to assigning editor templates to a tab.
            visibilityEditorTemplate.Metadata.Position = "Visibility:1";

            // Add the editor shape.
            context.EditorResult.Add(visibilityEditorTemplate);
        }

        public override void Displaying(ElementDisplayingContext context) {
            if (context.DisplayType == "Design")
                return;

            var element = context.Element;
            var elementType = element.GetType();

            if (!SupportedTypes.Contains(elementType))
                return;

            if (!element.GetFadeIn())
                return;

            context.ElementShape.Classes.Add("auto-fade-in");
            _resourceManager.Require("stylesheet", "CommonElement");
        }
    }
}
```

Notice that the element handler is almost the same as the driver implementation.
The primary differences are:

- Inheriting from `ElementEventHandlerBase` instead of `ElementDriver<T>`;
- Which means a slightly different signature for the `BuildEditor` and `Displaying` methods;
- Getting a reference to the element from the `context` argument, as it's not provided as a separate argument as is the case with the driver;
- Added very specific logic to when the `"FadeIn"` editor and behavior is applied. I declared a `SupportedTypes` collection property as some sort of 'white list'. Only elements of those types can be configured to fade in.    

Everything else remains the same: no need to change anything in our views. 

### Summary ###
In this chapter, we learned that we can extend existing elements with additional properties and behavior. We can do this at a very granular level or on a global level, or anything in between, by implementing element drivers and element handlers.