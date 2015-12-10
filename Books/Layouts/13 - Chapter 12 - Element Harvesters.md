## Element Harvesters ##
In this chapter, we will talk about element harvesters, what they are and how we can write our own.
Element Harvesters are providers of elements. They are the ones that make available the elements we can choose from the layout editor's toolbox for example. The reason they are called "harvesters" is because of how they provide elements: elements are harvested from a variety of sources.

As we have seen in the previous chapter, we can add custom elements to the system by writing classes that derive from `Element`. The Layouts module, however, doesn't discover these types directly. Instead, it relies on element harvesters to get the available *element descriptors*. The Layouts module comes with a few element harvesters out of the box, one of them being the `TypedElementHarvester`. That class is the one responsible for yielding element descriptors based on the available `Element` implementations.

This decoupling of Element types and Element Descriptors offers great flexibility and is why Orchard is able to provide all sorts of elements dynamically. For example, the Part and Field elements are provided by the `ContentPartElementHarvester` and `ContentFieldHarvester`, respectively. There are no actual Element classes for each content part in the system. Instead, these harvesters yield element descriptors dynamically based on the existence of content part and field definitions.

Element harvesters provide a low-level API to provide elements to the system, and we'll see how to implement our own shortly.

### Element Descriptors ###
An element descriptor, as it name implies, is an object that describes an element. It contains information such as the concrete .NET type to use when creating element instances, the technical name, description and display text. It also contains more advanced things, such as delegates to methods that are responsible for displaying element instances and element event handlers.

As mentioned, element descriptors are provided by element harvesters. When you look at the Layout Editor Toolbox, the elements you see there represent the available element descriptors.

TODO: Anatomy of the Element Descriptor class.

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

##### Step 2: Writing a Custom Element Harvester #####

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

##### Step 3: Trying it out #####

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

#### Improvements ####
When we use the profile field elements as-is, they render their content field using the default template for that field. Although we could use one of the existing alternates based on content field name or content type, let's add our own alternate based on our element's type name and the content field type and name in question. This way, we can customize the rendering of content fields just for the user profile field elements without affecting the default rendering of these fields when they are rendered by default.

The alternates we will add will be based on the element type name (`"UserProfileField"`), the content field type name (e.g. `"TextField"`) and the content field name (e.g. `"FirstName"`). With these alternates in place we can then create common templates for `UserProfileField` elements in general, or provide more specific templates for elements rendering a specific content field type, and even customize individual content fields.

To add these alternates, we first need to get our hands onto the shape (or shapes) created by the content field driver (or drivers). Because you see, `IContentFieldDisplay` does not actually return the field shapes directly. Instead, it returns a shape of type `ContentField` which has a `Content` property of type `ZoneHolding`, which holds any shape as returned by the content field drivers and allowed by the placement rules.

Since content field drivers can return any number of shapes of any given type, we will iterate over each content field shape and add alternates to them.

The following code listing adds two alternates:

- An alternate based on the shape type name and the term `"UserProfile"` to indicate that it is create by our element harvester.
- Another alternate based on the shape type name and the term `"UserProfile"`, plus the name of the content field.

```
...
// Render the field.
var contentFieldShapeHolder = _contentFieldDisplay.Value.BuildDisplay(currentUser, field, context.DisplayType);

// The returned shape is a ContentField shape that has a Content property of type ZoneHolding,
// which in turn contains a single shape which is the field shape we're interested in.
var fieldShapes = ((IEnumerable<dynamic>)contentFieldShapeHolder.Content.Items);

// Add alternates to each content field shape.
foreach (var fieldShape in fieldShapes) {
    fieldShape.Metadata.Alternates.Add($"{fieldShape.Metadata.Type}__UserProfile");
    fieldShape.Metadata.Alternates.Add($"{fieldShape.Metadata.Type}__UserProfile__{field.Name}");
}

//  Assign the field shape to a new property of the element shape.
context.ElementShape.ContentField = contentFieldShapeHolder;

...
```

With that code in place, the *FirstName* text field for example now provides all of the following alternates:

![](./figures/fig-92-available-alternates.png)

I highlighted the last two alternates that were added by our harvester.
Now this means freedom buddy. Let's create two Razor templates that customize the Text Field shapes:

- One text field template to get rid of the field name.
- Another text field template that is specific to the *TwitterHandle* field so that we can render it as a hyperlink.

Create the first Razor view with the following contents:

*Views/Fields/Common.Text-UserProfile.cshtml*:
```
@if (HasText(Model.Value)) {
    <p>@Model.Value</p>
}
```

And the second Razor view specific to the twitter handle field:

*Views/Fields/Common.Text-UserProfile-TwitterHandle.cshtml*:
```
@if (HasText(Model.Value)) {
    <a href="@String.Format("https://twitter.com/{0}", Model.Value)">@Model.Value</a>
}
```

Now our profile page looks like this:

![](./figures/fig-93-user-profile-polished.png)

Much better. Shapes, and shape alternates, they rock.

Now we can visually layout the user profile page using the Layouts editor and our custom elements in any way we like.

