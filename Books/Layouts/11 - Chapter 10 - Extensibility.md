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

### Element Data Storage ###
Notice how the `FormatString` property is implemented in the `Clock` element class. It uses the extension methods `Retrieve` and `Store` on the `Element` type. What these methods do is read and write data from and to the `Data` property inherited from `Element`. The `Data` property is a dictionary of key/value pairs, and is where elements child element classes store their data. This works similar to the way *InfoSet storage* works with content parts, where part information is stored into an `InfoSet` object that is serialized as XML. Element data is stored as a JSON string.

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

My friend, we are going places.

### Element Harvesters ###
As we have seen, we can add custom elements to the system by writing classes that derive from `Element`. The Layouts module, however, doesn't discover these types directly. Instead, it queries *element harvesters* to get all available *element descriptors*. The Layouts module comes with a few element harvesters out of the box, one of them being the `TypedElementHarvester`. That class is the one responsible for yielding element descriptors based on the available `Element` implementations.

This decoupling of Element types and Element Descriptors offers great flexibility and is why Orchard is able to provide all sorts of elements dynamically. For example, the Part and Field elements are provided by the `ContentPartElementHarvester` and `ContentFieldHarvester`, respectively. There are no actual Element classes for each content part in the system. Instead, these harvesters yield element descriptors dynamically based on the existence of content part and field definitions.

Element harvesters provide a low-level API to provide elements to the system, and we'll see how to implement our own shortly.

### Element Descriptors ###
An element descriptor is a class that describes an element. It contains information such as the concrete .NET type to use when creating element instances, the technical name, description and display text. It also contains more advanced things, such as delegates to methods that are responsible for displaying element instances and element event handlers.

As mentioned, element descriptors are provided by element harvesters. When you look at the Layout Editor Toolbox, the elements you see there represent the available element descriptors.

### Element Harvesters Out of the Box ###
The following table lists all of the element harvesters that ship with Orchard, along with a description of what source is used to yield element descriptors and which module provides the particular harvester:

<table>
    <thead>
        <tr>
            <th>Type</th>
            <th>Description</th>
            <th>Module</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>BlueprintElementHarvester</td>
            <td>Provides elements based on blueprint (pre-configured) elements.</td>
            <td>Layouts</td>
        </tr>
        <tr>
            <td>ContentFieldElementHarvester</td>
            <td>Provides elements based on the available content field definitions.</td>
            <td>Layouts</td>
        </tr>
        <tr>
            <td>ContentPartElementHarvester</td>
            <td>Provides elements based on the available content part definitions.</td>
            <td>Layouts</td>
        </tr>
        <tr>
            <td>PlaceableContentElementHarvester</td>
            <td>Provides elements based on the content types whose `Placeable` setting is set to `true`.</td>
            <td>Layouts</td>
        </tr>
        <tr>
            <td>SnippetElementHarvester</td>
            <td>Provides elements based on the existence of Razor views whose name end in "Snippet.cshtml".</td>
            <td>Layouts</td>
        </tr>
        <tr>
            <td>TypedElementHarvester</td>
            <td>Provides elements based on the existence of .NET types that inherit from the `Element` type.</td>
            <td>Layouts</td>
        </tr>
        <tr>
            <td>WidgetElementHarvester</td>
            <td>Provides elements based on the existence of Widget content types (content types whose Stereotype is "Widget").</td>
            <td>widgets</td>
        </tr>
    </tbody>
</table>

### Custom Element Harvesters ###
With all of the available element harvesters, you probably won't need to implement your own in the majority of cases. However, there may be advanced scenarios where it makes perfect sense to dynamically provide your own elements. For example, you could be writing a module that integrates with a third-party CRM that provides customer related fields as defined by that system, and you want to be able to add these fields as elements to content pages. Another example could be a custom harvester that yields elements based on the existence of content items of a certain type.

Whatever the case may be, knowing how to write custom element harvesters may come in handy. In the next two sections, we'll see how to implement our own.

#### The IElementHarvester Interface ####
All element harvesters implement the `IElementHarvester` interface, which lives in the `Orchard.Layouts.Framework.Harvesters` namespace. The following code listing shows how this interface looks like:

```
public interface IElementHarvester : ISingletonDependency {
    IEnumerable<ElementDescriptor> HarvestElements(HarvestElementsContext context);
}
```

The interface contains a single member that accepts a `HarvestElementsContext` argument and is expected to return an enumerable of `ElementDescriptor` objects.

Also notice that the interface is derived from the `ISingletonDependency`, which means that you need to be aware of the lifetime scopes of other dependencies you may inject in your implementations. For example, if you wanted to use the `IContentManager` service in your element harvester class, which has a *per-request* lifetime scope, you better inject that service using the `Work<T>` wrapper so that instances of `IContentManager` implementations are resolved in the current request context as opposed to the *application* lifetime scope.

The `HarvestElementsContext` object has only one property:

```
public class HarvestElementsContext {
    public IContent Content { get; set; }
}
```

The `Content` property is provided whenever the harvesters are invoked from a class or controller class that is somehow related to a content item. For example, in the case of the `LayoutEditorPartDriver`, which is responsible for creating the Layout Editor shape (which includes the elements toolbox), it passes along the content item associated with the `LayoutPart`. It is up to the individual harvesters if they need to do something with this information.

There are also cases where element harvesters are invoked outside the context of a specific content item. For example, when you create a blueprint element and are presented with a list of available base elements, there is no content item available to be used as context, so the `Content` property will be null. Therefore it's important to always check for null if your harvester can potentially do something with the Content property. Two examples that does take the `Content` property into account are the `ContentPartElementHarvester` and `ContentFieldElementHarvester`. These harvesters yield part and field element descriptors based on the parts and fields attached to the current content item for which the layout editor is being displayed.

With this information in our pockets, let's give it a try and write our own little demo harvester.

#### Trying it out: Writing a Custom Element Harvester ####
In this demo we will write a simple element harvester that teaches you how to dynamically provide elements to the system. We will create a new part called `UserProfilePart`. the harvester will select all fields from this part and present them as elements available from the toolbox. A content editor user can then create pages using these elements, which will show their values based on the currently logged in user.

Let's see how that works.

##### Step 1: UserProfilePart #####
The first thing we need to do is define the `UserProfilePart` and add content fields to it. We don't need an actual C# class to represent this part, so all we need for that is the following migration class:

```
using Orchard.ContentManagement.MetaData;
using Orchard.Core.Contents.Extensions;
using Orchard.Data.Migration;

namespace OffTheGrid.Demos.Layouts.Migrations {
    public class UserProfileMigrations : DataMigrationImpl {
        public int Create() {

            // Define the UserProfilePart.
            ContentDefinitionManager.AlterPartDefinition("UserProfilePart", part => part
                .WithField("FirstName", f => f
                    .OfType("TextField")
                    .WithSetting("TextFieldSettings.Flavor", "Wide")
                    .WithDisplayName("First Name"))
                .WithField("LastName", f => f
                    .OfType("TextField")
                    .WithSetting("TextFieldSettings.Flavor", "Wide")
                    .WithDisplayName("Last Name"))
                .WithField("TwitterHandle", f => f
                    .OfType("TextField")
                    .WithSetting("TextFieldSettings.Flavor", "Wide")
                    .WithDisplayName("Twitter Handle"))
                .WithDescription("Provides additional information about the user."));

            // Attach the UserProfilePart to the User content type.
            ContentDefinitionManager.AlterTypeDefinition("User", type => type
                .WithPart("UserProfilePart"));

            return 1;
        }
    }
}
```

Next, create a new class that implements `IElementHarvester`:

```
using System.Collections.Generic;
using Orchard.Layouts.Framework.Elements;
using Orchard.Layouts.Framework.Harvesters;

namespace OffTheGrid.Demos.Layouts.Layouts.Harvesters {
    public class UserProfileElementHarvester : IElementHarvester {
        public IEnumerable<ElementDescriptor> HarvestElements(HarvestElementsContext context) {
            // TODO: Return element descriptors based on the content fields attached to the UserProfilePart.
        }
    }
}
```

To reiterate, this is what we need to do to complete the implementation:

1. Get the `UserProfilePart` content part definition and query the attached content fields.
2. For each field, instantiate a new `ElementDescriptor` and configure it with appropriate data and handlers that implement the element's functionality.
3. Since the `ElementDescriptor` constructor requires a `Type` based on `Element`, which itself is abstract, we need to implement a so-called *system element*. A system element is an element class that is not harvested by the `TypedElementHarvester`, but serves only to instantiate elements from it. To turn an element class into a system element, all we need to do is override the `IsSystemElement` property on our custom element class and return `true`. We'll create a class called `UserProfileFieldElement`.
4. Return the created element descriptors.

That doesn't sound all too complex. The key thing here is getting the `UserProfilePart`, which we can get using the `IContentDefinitionManager` service, which we need to inject into our harvester's constructor. But remember, the harvester is an `ISingletonDependency`, so we need to resolve an instance of `IContentDefinitionManager` from the per-request lifetime scope.

The following code listing demonstrates how to get a content part and its content fields, as well as how to create element descriptors and return them:

```
using System;
using System.Collections.Generic;
using System.Linq;
using OffTheGrid.Demos.Layouts.Elements;
using Orchard;
using Orchard.ContentManagement.MetaData;
using Orchard.Environment;
using Orchard.Layouts.Framework.Elements;
using Orchard.Layouts.Framework.Harvesters;

namespace OffTheGrid.Demos.Layouts.Harvesters {
    public class UserProfileElementHarvester : Component, IElementHarvester {

        // Wrapping the content definition manager with a Work<T> to resolve the instance from the per-request lifetime scope.
        private readonly Work<IContentDefinitionManager> _contentDefinitionManager;

        public UserProfileElementHarvester(Work<IContentDefinitionManager> contentDefinitionManager) {
            _contentDefinitionManager = contentDefinitionManager;
        }

        public IEnumerable<ElementDescriptor> HarvestElements(HarvestElementsContext context) {
            // Get the UserProfilePart definition.
            var partDefinition = _contentDefinitionManager.Value.GetPartDefinition("UserProfilePart");

            // Get the content fields from the UserProfilePart definition.
            var fieldDefinitions = partDefinition.Fields;

            // For each field, yield an element descriptor.
            return from field in fieldDefinitions
                let settingKeys = field.Settings.Keys
                let descriptionKey = settingKeys.FirstOrDefault(x => x.IndexOf("description", StringComparison.OrdinalIgnoreCase) >= 0)
                let description = descriptionKey != null ? field.Settings[descriptionKey] : $"The {field.DisplayName} field."
                select new ElementDescriptor(
                    elementType: typeof(UserProfileField),
                    typeName: $"UserProfile.{field.Name}",
                    displayText: T(field.DisplayName),
                    description: T(description),
                    category: "User"
                ) {
                    ToolboxIcon = "\uf040"
                };
        }
    }
}
```

The key portion of the above code listing is the LINQ query that projects the field definitions into element descriptors. Notice also the way that I am constructing the element description. We could of course provide a static string, but instead I am inspecting the `Settings` dictionary of the field to see if it has a setting whose name contains the word `"description"`. If it does, I'm using that as the description. If not, I construct one myself based on the name of the field.

The following code listing shows the `UserProfileElement` class implementation I used:

```
using Orchard.Layouts.Framework.Elements;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class UserProfileField : Element {
        public override bool IsSystemElement => true;
    }
}
```

Whenever Orchard instantiates elements from our harvested element descriptors, it will use this class. The `IsSystemElement` returns `false` to prevent the `TypedElementHarvester` from harvesting this element class as an element. 

When we build and run the code now, we see indeed that new elements appear in the toolbox:

![](./figures/fig-86-user-profile-elements-first-look.png)

This is looking good. The final missing piece is actually displaying the user profile information. We nee need to assign a callback to the `Displaying` delegate property of our element descriptors that is invoked when our element is being displayed. Our callback will then use the field name to access the `UserProfilePart` from the currently logged in user and display its value. the following code listing shows how this works:

```
...
{
    ToolboxIcon = "\uf040",
    Displaying = displayingContext => OnDisplaying(field.Name, displayingContext)
};
...

private void OnDisplaying(string fieldName, ElementDisplayingContext context) {
    var workContext = _workContextAccesor.GetContext();
    var currentUser = workContext.CurrentUser;

    // If there is no authenticated user, return.
    if (currentUser == null)
        return;

    var profilePart = currentUser.ContentItem.Parts.SingleOrDefault(x => x.PartDefinition.Name == "UserProfilePart");

    if (profilePart == null) {
        // The UserProfilePart has been removed from the User content type.
        return;
    }

    var field = profilePart.Fields.SingleOrDefault(x => x.Name == fieldName);

    if (field == null) {
        // Someone removed the field from the UserProfilePart.
        // This situation can occur when a user removed the field and the harvested element descriptors cache entry hasn't been evicted yet.
        // A future version of Orchard will provide custom control over cache entry eviction of dynamic elements.
        return;
    }

    // Render the field and add it to the element shape.
    var fieldShape = _contentFieldDisplay.Value.BuildDisplay(currentUser, field, context.DisplayType);
    context.ElementShape.ContentField = fieldShape;
}

```

To make the above code work, I injected the `IWorkContextAccessor` and `IContentFieldDisplay` services. Since the lifetime scope of `IWorkContextAccessor` is bound to the application lifetime, we don't have to wrap it around with `Work<T>`, but we do with `IContentFieldDisplay`.

> The `IContentFieldDisplay` is a service provided by the Layouts module and enables you to render an individual content field.

Since we are setting the content field shape to a property on the element shape being displyed, we should create a shape template for our elements. As discussed earlier, when an element shape is created, an alternate is added based on the .NET type name of the element class. Since we are providing the `UserProfileField` type as the element type, there will be an alternate available of `"Elements.UserProfileField"`. So let's create a Razor view in the Views folder of our custom module (*Views/Elements/UserProfileField.cshtml*, next to *Clock.cshtml*) and provide the following code:

```
@if(Model.ContentField != null) {
    @Display(Model.ContentField)
}
else {
	<!-- The content field is not available. -->
}
```

All this code does is render the ContentField shape, if there is one set.

And with that in place, let's see it in action!

First, let's update our admin user's profile:

![](./figures/fig-87-updating-user-profile.png)

Next, let's create an additional user so we can verify that our user profile field elements work as intended:

![](./figures/fig-88-adding-new-user-profile.png)

Now create a new page called "User Profile" and add the user profile elements anywhere you like:

![](./figures/fig-89-user-profile-design.png)

With all that in place, let's check out the front-end while being logged in as *admin*:

![](./figures/fig-90-user-profile-1.png)

Logout and login as the other user *john*:

![](./figures/fig-91-user-profile-2.png)

Nice. Notice that the user profile field elements now display the values from the user *john*.

What's also nice is that since the harvester is querying content fields, any additional fields that we add to the `UserProfilePart` will become automatically available as elements.

#### Additional Improvements ####
When we use the profile field elements as-is, they render the content fields using the "Default" display type, which for the majority of content fields will render the field name and field value (e.g. "First Name: John"). Most likely you will want to provide your own template so that only the field value is rendered for example, and perhaps surrounded with some custom HTML.

Since the content fields are rendered using shapes, we can customize their rendering based on their alternates. One possible alternate we could use is the one based on the content type to which the field is attached. Another candidate is the alternate based on the field name. However, using any of those 