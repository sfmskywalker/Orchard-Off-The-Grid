## 10. Extensibility ##
So far we have seen how to use the Layouts module from a user's and theme developer's perspective. The functionality available out of the box is pretty impressive as it is. However, **Orchard.Layouts** wouldn't be a real Orchard module if it wasn't also extremely extensible.

One of the most interesting things to Orchard module developers is the extensibility points in a given module. When it comes to the Layouts module, one of the first things a developer asks himself is: "How can I create my own elements?"

As we have learned so far, users can create custom elements by creating *element blueprints*. These are pre-configured elements and doesn't require any bit of coding. We also learned how theme developers can provide custom elements called *snippets* by creating Razor views whose name ends in "Snippet.cshtml". These snippets can even be made configurable by defining *snippet fields* using the `Html.SnippetField` HTML helper.

But eventually there will be a point where you want complete control over how your custom elements behave and provide more advanced UI and behavior.

In this third part, we will learn everything there is to know about writing custom elements.

### Custom Elements ###
In order to write custom elements, it will help to understand what an element is made of. As it turns out, elements are simply instances of .NET sub classes that inherit from the `Element` abstract base class which lives in the `Orchard.Layouts.Framework.Elements` namespace. Sub classes then typically add properties to their type definition and override inherited properties such as the `Category` one.

While element classes handle the data side of things, it's the *element drivers* that handle the behavioral side of things. This works very similar to the way content parts, content part drivers, content fields and content field drivers work. Drivers handle things like displaying and editing the element.

If you're familiar with writing custom parts and fields, you'll recognize many of the concepts when writing custom elements.              

> Although I'm using the term "custom element", elements that you write are no different from the elements provided by the Layouts module. I'm just using the term "custom" to differentiate between out-of-box- elements and elements that you write as part of your custom modules.

We'll go into much deeper detail, but the following lists an overview of the typical steps involved when writing a custom element.

1. Create a class that derives from `Element` or any of its child classes. The only required member that needs to be implemented is the abstract `Category` property. 
2. Create a class that derives from `ElementDriver<T>`, where `T` is your element's class. This class does not require any members, but needs to be there in order for your element type to be discoverable due to the way the `TypedElementHarvester` works. We'll look into element harvesters later on.
3. Although not strictly required, create a Razor view for your element for the `"Detail"` display type. If you don't provide a view, a default view will be used which simply displays the name of the element.
4. Also not strictly required, create a design-time view for your element for the `"Design"` display type. If you don't provide a design-time view, the view for the `"Detail"` display type will be used. This view is used when your element is rendered by the layout editor.



#### Trying it out: Creating the Clock element ####
In this demo we'll create a very simple custom element called `Clock` that will display the current time. The goal of this demo is to introduce the basics of custom element development. We'll build more advanced elements later on.

> For the demos in this third part, I created a custom module called `OffTheGrid.Demos.Layouts` that you can download from the book's website.

First, create a directory in your module called *Elements* and add a class called `Clock` as follows:

```
using Orchard.Layouts.Framework.Elements;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class Clock : Element {
        public override string Category => "Demo";
        public override bool HasEditor => false;
    }
}
```

Next, create a directory called *Drivers* and add the following driver class:

```
using OffTheGrid.Demos.Layouts.Elements;
using Orchard.Layouts.Framework.Drivers;

namespace OffTheGrid.Demos.Layouts.Drivers {
    public class ClockDriver : ElementDriver<Clock> {
    }
}
```

The driver class doesn't have any implementation at this point, but as mentioned earlier this is necessary for the `TypedElementHarvester` to be able to discover our element.

Build the solution and launch the site. Create a new page called "Custom Elements" and add the *Clock* element to the canvas (make sure you enabled your module). This is what it should look like:

![](./figures/fig-83-clock-element-design.png)

The next step is to have the clock actually display the current time. To do so, we need to provide a view for our element. When an element is being rendered, a shape called `Element` is created, to which a number of alternates is added. One of these alternates is based on the type name of the element. The alternate we're interested in is as follows: `Elements.Clock`, so add a view called `Elements/Clock.cshtml` that looks like this:

```
<p>@DateTime.Now.ToString("t")</p>
```

And that's all it takes to write a basic element. Now, let's say that we wanted to enable the user to configure the date/time format string for the Clock element. To achieve that, we need to do three things:

1. Add a property to the `Clock` element that stores the format string and change the return value for `HasEditor` from `false` to `true`, since we will want to present the user with an editor dialog when adding and editing Clock elements.
2. Implement an edit view for our clock so that the user can provide a format string.
3. Update the display view to take advantage of the configured format string.

Let's start by adding the format string property and changing the return value from `false` to `true` for the `HasEditor` property:

```
using Orchard.Layouts.Framework.Elements;
using Orchard.Layouts.Helpers;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class Clock : Element {
        public override string Category => "Demo";
        public override bool HasEditor => true;

        public string FormatString {
            get { return this.Retrieve(x => x.FormatString); }
            set { this.Store(x => x.FormatString, value); }
        }
    }
}
```


> ##### Element Data Storage #####
> Notice how the `FormatString` property is implemented. It uses the extension methods `Retrieve` and `Store` on the `Element` type. What these methods do is read and write data from and to the `Data` property inherited by all element classes. The `Data` property is a dictionary of key/value pairs, and is the mechanism used for storing your custom element's data. This works somewhat similar to the way *InfoSet storage* works with content parts, where part information is stored into an `InfoSet` object that is serialized as XML. Element data is stored as a JSON string.

Next, we need a way for the user to configure the `FormatString` property. This is done by implementing the `OnBuildEditor` and `OnUpdateEditor` methods in the `ClockDriver` class as follows:

```
using OffTheGrid.Demos.Layouts.Elements;
using Orchard.Layouts.Framework.Drivers;

namespace OffTheGrid.Demos.Layouts.Drivers {
    public class ClockDriver : ElementDriver<Clock> {
        protected override EditorResult OnBuildEditor(Clock element, ElementEditorContext context) {
            return Editor(context, context.ShapeFactory.EditorTemplate(TemplateName: "Elements/Clock", Model: element));
        }

        protected override EditorResult OnUpdateEditor(Clock element, ElementEditorContext context) {
            context.Updater.TryUpdateModel(element, null, null, null);
            return OnBuildEditor(element, context);
        }
    }
}
```

The `OnBuildEditor` returns an EditorResult (returned by the `Editor` methods inherited from the base class) that takes the `context` argument as well as a shape object. we're passing in an `EditorTemplate` shape, which itself is configured with a `TemplateName` and `Model` property.

The `OnUpdateEditor` method is invoked when the user hits `Save` on the editor dialog screen. Here we use the `Updater` provided by the `context` argument to bind the submitted values against our model. In this simple example we're using the element directly as the model, but in more advanced scenarios you may choose to work with view models instead. We'll see how that works later on as well. Like the `OnBuildEditor` method, the `OnUpdateEditor` method too needs to return an `EditorResult`. We simply return the one as created by our `OnBuildEditor` method to save some duplication.

> Another pattern you typically see with element development is that only the `OnBuildEditor` is implemented. Since the `Updater` is passed in there as well, you could implement both building and updating code in a single method. This is especially useful if you work with view models, as it prevents you from having to initialize your view models in both methods. We'll see how that looks like when we implement more advanced editor UIs.

Next, create the "Elements/Clock.cshtml" view in the "Views/EditorTemplates" folder of your module, and provide the following code: 

```
@model OffTheGrid.Demos.Layouts.Elements.Clock

<div class="form-group">
    @Html.LabelFor(m => m.FormatString, T("Format String"))
    @Html.TextBoxFor(m => m.FormatString, new { @class = "text medium" })
    @Html.Hint(T("Provide a date/time format string for the current time when being displayed. If not value is provided, the \"t\" standard format is used."))
</div>
```

With that in place, we can now open the element's editor and provide a custom format string, like "g" for example:

![](./figures/fig-84-clock-element-editor.png)

Finally, we need to actually take advantage of the `FormatString` property in our element's view. When the `Element` shape is created, a property called `Element` is automatically set to the element's instance. This means that we can access the `Clock` element instance from our display view. Edit the `Elements/Clock.cshtml` view as follows:

```
@{
	var formatString = (string)Model.Element.FormatString; // Model.Element is of type "OffTheGrid.Demos.Elements.Clock".

	if (String.IsNullOrWhiteSpace(formatString)) {
        formatString = "t";
	}
}
<p>@DateTime.Now.ToString(formatString)</p>
```

### An Alternative Element Editor Implementation ###
In the previous demo, we saw one way to implement an element editor by taking advantage of the `EditorTemplate` shape. Another kind of shape we could leverage is shapes created by the *Forms API*. The Forms API is provided by the `Orchard.Forms` module, and offers a way to create forms programmatically without having to create Razor views.

To simplify the creation and binding of forms using the Forms API, the Layouts module comes with a base driver class called `FormsElementDriver`. All that your element driver has to do now is provide one or more *form names* that need to be rendered. This class also implements the `IFormProvider` interface so that your driver can provide a form programmatically right there and then.

Since our Clock's element editor is so simple, let's see what it takes to use the forms API to build our editor instead of using Razor views.

#### Trying it out: Element Editors using the Forms API  ####
The first things to do are to replace the base class of the `ClockDriver` class with `FormsElementDriver`, remove the `OnBuildEditor` and `OnUpdateEditor` methods and the editor view template (`EditorTemplates/Elements/Clock.cshtml`).

Next, we need to:

1. Override the `DescribeForm` method to programmatically create our editor.
2. Override the `FormNames` to tell our driver which form(s) to render when editing the `Clock` element.

The following code listing shows the updated version of the `ClockDriver` class:

```
using System.Collections.Generic;
using OffTheGrid.Demos.Layouts.Elements;
using Orchard.Forms.Services;
using Orchard.Layouts.Framework.Drivers;
using Orchard.Layouts.Services;

namespace OffTheGrid.Demos.Layouts.Drivers {
    public class ClockDriver : FormsElementDriver<Clock> {
        public ClockDriver(IFormsBasedElementServices formsServices) : base(formsServices) { }

        protected override IEnumerable<string> FormNames {
            get { yield return "ClockEditor"; }
        }

        protected override void DescribeForm(DescribeContext context) {
            context.Form("ClockEditor", shapeFactory => {
                var shape = (dynamic)shapeFactory;
                var form = shape.Fieldset(
                    Id: "Clock",
                    _FormatString: shape.Textbox(
                        Id: "FormatString",
                        Name: "FormatString", // -> This name needs to match the name of the FormatString property on the Clock class.
                        Title: T("Format String"),
                        Classes: new[] { "text", "medium" },
                        Description: T("Provide a date/time format string for the current time when being displayed. If not value is provided, the \"t\" standard format is used.")));

                return form;
            });
        }
    }
}
```

There are a few important aspects to using the Forms API with element editors:

1. The form name being returned by the `FormNames` property needs to match the name of the form that is provided when describing the form (`context.Form("ClockEditor",...` in the `DescribeForm` method.
2. The `FormsElementDriver` base class implementation stores the form field values in the `Data` dictionary of the `Element` class. Since the `Clock` class reads and writes from and to this dictionary, we need to make sure the "Name" value of the "FormatString" text box element matches exactly with the key into the `Data` dictionary.
3. Notice that the root shape being returnd is not a `Form`, but a `Fieldset`. This is important, because the element editor dialog already renders a form element. And we obviously don't want nested forms, as that would break the entire editor.

With these changes in place, the editor experience looks exactly the same, but without the need for a Razor view for the editor template.

### Providing Data to the Display View ###
When writing custom elements, often times you will need to prepare stuff for the display view from the driver. The `Clock` example didn't have that requirement, but let's say that we wanted its display view to be cleaned up a bit. For exaple, the view currently contains some logic to set a default format string in case none was provided by the editor:

```
@{
	var formatString = (string)Model.Element.FormatString; // Model.Element is of type "OffTheGrid.Demos.Elements.Clock".

	if (String.IsNullOrWhiteSpace(formatString)) {
        formatString = "t";
	}
}
``` 

Now let's say we wanted to move this logic out of the view and into our driver. To achieve that, we need to override the `OnDisplaying` method on our `ClockDriver`. That method receives a context argument which provides a reference to the shape being displayed. This is the perfect place to initialize a `formatString` variable, using a default value if none was provided by the editor, and sticking that value into the element shape. That value can then be used directly by the view.

#### Trying it out: Providing Data from the OnDisplaying Method ####
Add the following code to the `ClockDriver` class:

```
protected override void OnDisplaying(Clock element, ElementDisplayingContext context) {
    var formatString = element.FormatString;

    // If the clock editor didn't provide a format string, provide a default value.
    if (String.IsNullOrWhiteSpace(formatString)) {
        formatString = "t";
    }

    // Add a new property called "formatString" to the element shape being displayed.
    context.ElementShape.FormatString = formatString;
}
```

Next, update the `Clock.cshtml` display view with the following, simplified, code:

```
@{
	var formatString = (string)Model.FormatString; // Model.FormatString is set by the ClockDriver.OnDisplaying method.
}
<p>@DateTime.Now.ToString(formatString)</p>
```

Or even simpler:

```
<p>@DateTime.Now.ToString(Model.FormatString)</p>
```

### Custom Descriptions and Toolbox Icons ###
When writing custom elements, it is a good practice to provide a description for your element. The description is displayed as a tooltip when hovering over the element in the toolbox. Providing a description is easily done by overriding the `Description` property. You can also specify a custom toolbox icon by overriding the `ToolboxIcon` property on your element class. The Layout Editor expects a valid Font Awesome icon identifier to be returned.

#### Tying it out: Polishing the Clock Element #### 
In this demo, we'll polish our Clock element by providing a proper description and a more repsentative Font Awesome icon. All we need to do is override the `Description` and `ToolboxIcon` properties as follows:

```
public override LocalizedString Description => T("Displays the current date and time");
public override string ToolboxIcon => "\uf017";
```

Which yields the following result:

![](./figures/fig-85-clock-description-and-icon.png)

Yes my friend, we are going places.

### Element Harvesters ###
In addition to providing elements by implementing `Element` classes, another, more lower-level way of providing elements, is implementing a *element harvesters*.

An element harvester yields elements that are available to the user from the toolbox. More accurately, element harvesters yield *element descriptors*.

#### Element Descriptors ####
An element descriptor is a class that describes an element. It contains information such as the concrete .NET type to use when creating element instances, the technical name, description and display text. It also contains more advanced things, such as delegates to methods that are responsible for displaying element instances and element event handlers.

### Element Harvesters Out of the Box ###
The Layouts module comes with the following element harvesters:

- Blue
- TypedElementHarvester
-  



                