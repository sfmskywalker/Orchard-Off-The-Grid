## Container Elements ##
In the previous chapter, we learned how to write custom elements by deriving fro, the `Element` base class, or a derivative thereof. Writing custom elements that can contain other elements are similar, but require you to inherit from the `Container` class instead, which in turn inherits from `Element`.

However, there is much more involved when writing custom container elements due to the way the layout editor works with container elements.

In this chapter, we will unravel all the steps that are involved when creating custom container elements.

### Writing Custom Container Elements ###
Writing custom container elements boils down to the following steps:

1. Implement a class that drives from `Container`.
2. Implement a driver for your container element class. As is the case with any element driver, your driver needs to be derived from `ElementDriver<T>`, where the generic `T` type argument is to be substituted with your custom element type.

### The Container Class ###
Container elements are instances of classes that inherit from the `Container` abstract base class which lives in the `Orchard.Layouts.Elements` namespace.

The `Container` class adds single member called `Elements`, which is a list of elements:

```
public abstract class Container : Element {
    public IList<Element> Elements { get; set; }
}
```

This `Elements` list is initialized to an empty list from the constructor.

### Element Drivers ###
When writing custom element classes, you need to implement a driver for that element, even if it contains no actual implementation. This is the same when writing container elements.

### Layout Editor Integration ###
Most of the work when creating a custom container goes into making it work with the layout editor. The primary reason for this is the fact that the layout editor has a client side model of elements, and it needs to know what the type of object is to be used on the client.

The layout editor roughly divides elements into two categories:

- Content elements (Html, Image, Heading, etc.)
- All the rest (Canvas, Grid, Row, Column)

The distinction is made as follows: if a given element type declares its own **Model Map**, it means it has a specific client side (JavaScript) representation of that element. If not, the client side representation is always the **Content** class.

### Layout Model Mapper ###
Layout model mapping is a conversion process that basically converts a list of `Element` objects into a JSON format that the layout editor can work with, and the other way around: to convert a layout editor data JSON string back into a list of `Element` objects.

The reason we need to do that is because the layout editor requires its own JSON schema to work with elements, so we need a mechanism to serialize elements into that JSON format. That mechanism is called *layout model mapping*.

The layout model mapper is defined as the `ILayoutModelMapper` service, and is declared as follows:

```
/// <summary>
/// Maps element data to an editor compatible object model.
/// </summary>
public interface ILayoutModelMapper : IDependency {
    /// <summary>
    /// Maps the specified layout data to a JSON representation of a layout editor compatible object model.
    /// </summary>
    /// <param name="layoutData">The layout serialized as a string to map to the editor JSON format.</param>
    /// <param name="describeContext">A context for the element activator when describing elements.</param>
    /// <returns>Returns an object that represents the layout model compatible with the layout editor.</returns>
    object ToEditorModel(string layoutData, DescribeElementsContext describeContext);

    /// <summary>
    /// Maps the specified element to a JSON representation of a layout editor compatible object model.
    /// </summary>
    /// <param name="element">The element to map to the editor JSON format.</param>
    /// <param name="describeContext">A context for the element activator when describing elements.</param>
    /// <returns>Returns an object that represents the element compatible with the layout editor.</returns>
    object ToEditorModel(Element element, DescribeElementsContext describeContext);

    /// <summary>
    /// Maps the specified editor data to an hierarchical list of elements.
    /// </summary>
    /// <param name="editorData">The editor JSON sent from the client browser.</param>
    /// <param name="describeContext">A context for the element activator when describing elements.</param>
    /// <returns>Returns an hierarchical list of elements.</returns>
    IEnumerable<Element> ToLayoutModel(string editorData, DescribeElementsContext describeContext);

    ILayoutModelMap GetMapFor(Element element);
}
```

The way the model mapper works is by relying on individual `ILayoutModelMap` implementations to either provide a DTO (anonymous object) to be serialized into JSON, or parse a `JNode` object back into an `Element` instance.

The `ILayoutModelMap` is declared as follows:

```
public interface ILayoutModelMap : IDependency {
    int Priority { get; }
    string LayoutElementType { get; }
    bool CanMap(Element element);
    Element ToElement(IElementManager elementManager, DescribeElementsContext describeContext, JToken node);
    void FromElement(Element element, DescribeElementsContext describeContext, JToken node);
}
```

The following table provides a description for each member:

<table>
<thead>
	<tr>
		<th>Member</th>
		<th>Description</th>
	</tr>
</thead>
<tbody>
	<tr>
		<td>Priority</td>
		<td>If multiple model maps can convert from and to a given element type, the one with the highest priority is selected to do that job. This is useful in cases such as the <strong>ContentModelMap</strong>, which can basically map any element that is not a container. So in order to allow other non-container elements to provide their own model map implementation, they need to be able to provide a higher priority.</td>
	</tr>
	<tr>
		<td>LayoutElementType</td>
		<td>Returns the <strong>client-side class name</strong> of the element. When the layout editor needs to instantiate an element based on its JSON data, it uses this name to instantiate an object of that class.</td>
	</tr>
	<tr>
		<td>CanMap</td>
		<td>Returns <strong>true</strong> if this model map implementation can map the specified element, false otherwise.</td>
	</tr>
	<tr>
		<td>ToElement</td>
		<td>Returns an <Strong>Element</strong> object initialized with the provided <strong>JNode</strong> data. This is a way for the client-side layout editor to pass in data set by the user and into the element as stored on the server side. Examples of this are the HtmlId, HtmlClass and HtmlStyle properties on an Element.</td>
	</tr>
	<tr>
		<td>FromElement</td>
		<td>The counterpart of <strong>ToElement</strong>. This method expects implementations to populate the specified <strong>JNode</strong> with data from the specified Element.</td>
	</tr>
</tbody>
</table>

> The layout editor allows the user to manipulate element properties in two ways.
> 
> The first way is by invoking the element editor dialog, where element drivers are involved to render the editor UI and handle form submissions. The resulting element data is then sent back as a JSON string to the layout editor via JavaScript.
> 
> The second way is to edit element properties *directly* from the layout editor. Since the layout editor works with its own JSON schema, we need a way to convert this data back to the standard JSON schema. This is where the layout model mappers come into play. 

### Client Side Support ###
If you thought model mappers were kind of icky, wait until you see what needs to be done for the client side representation of custom container elements. That process might feel somewhat arcane to say the least, but since you're reading this book it will turn out just fine.

Implementing the client side story of container elements essentially requires the following four steps:

1. Implement the client-side model of the element;
2. Implement the directive (since the layout editor is implemented with AngularJS);
3. Implement the template for the directive;
4. Register the element with the client-side element factory.

Once you got all that in place, you'll be able to drag & drop your custom container element onto the canvas, and add any other elements to it (depending on the allowable element types that you specified when implementing the client-side model).

Now that we've seen the overview of the process, let's dive in and see what it looks like when actually implementing a container element.

### Custom Container Walkthrough ###
In this walkthrough, we'll implement a custom container element that enables the user to specify a background image. Once you understand how to implement container elements, you'll be able to create any other type of elements with any number of properties, since the principles will be the same.

We'll call our element the *Tile* element, which has the following characteristics:

- A Tile element can contain any other types of elements.
- A Tile element can optionally have one background image.

Since the Tile element stores a reference to a content item (the background image), we'll need to make sure that we export the content item identity, since the content item id (a primary key value) will be useless when importing.

We'll also need to add a project reference to the *Orchard.MediaLibrary* project, since we'll be using the `ImagePart` type from that project.

Let's see how it all works.

#### The Tile Element ####
Create the following class in the Elements folder of our module:

```
using System;
using System.Collections.Generic;
using System.Linq;
using Orchard.Layouts.Elements;
using Orchard.Layouts.Helpers;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class Tile : Container {
        public override string Category {
            get { return "Demo"; }
        }

        public int BackgroundImageId {
            get {
                var data = this.Retrieve("BackgroundImageId", () => "");
                return data.Split(',').Select(Int32.Parse).ToArray();
            }
            set {
                this.Store("BackgroundImageIds", String.Join(",", value));
            }
        }
    }
}
```

Notice that we're storing a list of integers that represent the selected image content IDs.

#### The Tile Element Driver ####
Create the following class in the Drivers folder:

```
using System;
using System.Linq;
using Orchard.ContentManagement;
using Orchard.Layouts.Framework.Drivers;
using Orchard.Layouts.Helpers;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class TileDriver : ElementDriver<Tile> {
        private IContentManager _contentManager;

        public TileDriver(IContentManager contentManager) {
            _contentManager = contentManager;
        }

        protected override void OnExporting(Tile element, ExportElementContext context) {
            // Get the list of selected background content item ids.
            var backgroundImageIds = element.BackgroundImageIds;

            // Load the actual background content items.
            var backgroundImages = _contentManager.GetMany<IContent>(backgroundImageIds, VersionOptions.Latest, QueryHints.Empty);

            // Use the content manager to get the content identities.
            var backgroundImageIdentities = backgroundImages.Select(x => _contentManager.GetItemMetadata(x).Identity).ToList();

            // Add the content item identities to the ExportableData dictionary.
            context.ExportableData["BackgroundImages"] = String.Join(",", backgroundImageIdentities);
        }

        protected override void OnImporting(Tile element, ImportElementContext context) {
            // Read the imported content identities from the ExportableData dictionary.
            var backgroundImageIdentities = context.ExportableData.Get("BackgroundImages", "").Split(',');

            // Get the imported background content item from the ImportContentSesison store.
            var backgroundImages = backgroundImageIdentities.Select(x => context.Session.GetItemFromSession(x)).Where(x => x != null);

            // Get the background content item id (primary key value) for each background image.
            var backgroundImageIds = backgroundImages.Select(x => x.Id);

            // Assign the content ids to the BackgroundImageIds property so
            // that they contain the correct values, instead of the automatically imported values.
            element.BackgroundImageIds = backgroundImageIds;
        }
    }
}
```

The driver handles the following things:

- Import/Export of the selected background image content items.

#### The Tile Model Map ####

```
using Orchard.Layouts.Services;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class TileModelMap : LayoutModelMapBase<Tile> {
    }
}
```

#### The Client-Side Tile Model ####
#### The Client-Side Tile Directive ####
#### The Client-Side Tile Directive Template ####
#### Assets.json ####

> When the Task Runner Explorer shows the message "Failed to load. See output window", this is usually an indication that you haven't installed the NodeJS packages. This is easily fixed by opening the **Package.json** file in the **Solution Items/Gulp** solution folder, and simply saving that file (CTRL + S). When you do, Visual Studio will start downloading NodeJS packages, including Gulp. Gulp is used to compile, minify and bundle files such as CSS, LESS, JavaScript and TypeScript. Installing the various NodeJS packages may take a few minutes, so keep a close eye on the status bar (which should read "Installing packages" while installing packages, and "Installing paklages complete" when done.
> 
> Once the packages have been installed, click the refresh icon in the Task Runner Explorer and wait a second for it to refresh. Now you should see the **Tasks** and its child nodes **build**, **rebuild** and **watch**. Right-click on *build* and then left-click *Run* to have your Assets.json file executed.

#### Registering the Tile.js Script ####


### Summary ###
In this chapter, we have seen how we can extend the list of available elements by implementing our own element classes. We learned about the `Element` class itself as well as the `ElementDriver<T>` class. We then continued and created a custom element called the `Clock`, demonstrating what it takes to provide custom rendering as well as an element editor. We also learned about implementing element editors using the *Forms API* provided by the `Orchard.Forms` module, which is great for basic element editors where there is no need for more advanced editor UIs.

With this knowledge in our pocket, we have all we need to be able to write any kind of elements, whether they be simple or more complex. It always boils down to implementing an element class and driver, and implementing the various methods on the driver.

However, what if we wanted to provide elements based not on static element classes, but on something else entirely? For example, what if you have a database table of records that you would like to present as elements? Well, that is in fact the next chapter's topic, where we'll learn everything there is to know about *Element Harvesters*.   