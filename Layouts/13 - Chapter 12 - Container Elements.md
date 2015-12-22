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

### Walkthrough: Writing A Custom Container ###
In this walkthrough, we'll implement a custom container element that enables the user to specify a background image. Once you understand how to implement container elements, you'll be able to create any other type of elements with any number of properties, since the principles will be the same.

We'll call our element the *Tile* element, which has the following characteristics:

- A Tile element can contain any other types of elements.
- A Tile element can optionally have a *background image*.
- A Tile element can optionally have a *background size*. The background size will be rendered as the `background-size` CSS attribute.
- The selected background image will be rendered as part of the Tile element in design mode.
- The user will be able to change the *background size* property using the *quick properties accessor*.

> The **quick properties accessor** is the little popout window that appears when you click the "Edit [element type] properties (Space)" icon in an element's toolbar, and offer a way for users to quickly change certain properties without having to launch the element editor's dialog. Common to all elements are properties such as *HTML ID*, *CSS Classes*, *CSS Styles* and *Visibility Rule*. However, as we'll see in this walkthrough, custom elements have complete control over this set of properties.

Since the Tile element stores a reference to a content item (the background image), we'll need to make sure that we export the content item identity, since the content item id (a primary key value) will be useless when importing.

We'll also need to add a project reference to the *Orchard.MediaLibrary* project, since we'll be using the `ImagePart` type from that project. 

Let's see how it all works.

#### The Tile Element ####
First of all, we need to create an element class called `Tile`, which will override the `Category` and `ToolboxIcon` properties, and add two additional properties: the `BackgroundImageId`, which will store the image content item ID that the user selected, and the `BackgroundSize` property, which will control how the background image gets applied in terms of the `background-size` CSS attribute. The following code listing shows our complete `Tile` class:

```
using Orchard.Layouts.Elements;
using Orchard.Layouts.Helpers;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class Tile : Container {
        public override string Category => "Demo";
        public override string ToolboxIcon => "\uf03e";

        public int? BackgroundImageId {
            get { return this.Retrieve(x => x.BackgroundImageId); }
            set { this.Store(x => x.BackgroundImageId, value); }
        }

        public string BackgroundSize {
            get { return this.Retrieve(x => x.BackgroundSize); }
            set { this.Store(x => x.BackgroundSize, value); }
        }
    }
}
```

Nothing too fancy going on here, so let's move on to the Tile driver.

#### The Tile Driver ####
The Tile driver will be responsible for the following things:

- Rendering the Tile element editor (which is displayed when adding / editing a Tile element instance);
- Handling post-back of the element editor (model binding the posted values and storing them as part of the Tile element instance);
- Preparing some data when the Tile element is being rendered (`OnDisplaying` event). In this demo, all we need is to load the selected background image by the ID that is stored in the `BackgroundImageId` property.
- Implement Import/Export for our Tile, since we're referencing a content item (the background image content item).

The following code listing shows the complete implementation of the Tile driver:

```
using System.Linq;
using OffTheGrid.Demos.Layouts.ViewModels;
using Orchard.ContentManagement;
using Orchard.Layouts.Framework.Display;
using Orchard.Layouts.Framework.Drivers;
using Orchard.Layouts.Helpers;
using Orchard.MediaLibrary.Models;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class TileDriver : ElementDriver<Tile> {
        private IContentManager _contentManager;

        public TileDriver(IContentManager contentManager) {
            _contentManager = contentManager;
        }

        protected override EditorResult OnBuildEditor(Tile element, ElementEditorContext context) {
            var viewModel = new TileViewModel {
                BackgroundImageId = element.BackgroundImageId?.ToString(),
                BackgroundSize = element.BackgroundSize
            };

            // If an Updater is specified, it means the element editor form is being submitted
            // and we need to read and store the submitted data.
            if(context.Updater != null) {
                if (context.Updater.TryUpdateModel(viewModel, context.Prefix, null, null)) {
                    element.BackgroundImageId = Orchard.Layouts.Elements.ContentItem.Deserialize(viewModel.BackgroundImageId).FirstOrDefault();
                    element.BackgroundSize = viewModel.BackgroundSize?.Trim();
                }
            }
            
            viewModel.BackgroundImage = GetBackgroundImage(element, VersionOptions.Latest);

            var editorTemplate = context.ShapeFactory.EditorTemplate(
                TemplateName: "Elements/Tile",
                Model: viewModel,
                Prefix: context.Prefix);

            return Editor(context, editorTemplate);
        }

        protected override void OnDisplaying(Tile element, ElementDisplayingContext context) {
            var versionOptions = context.DisplayType == "Design" ? VersionOptions.Latest : VersionOptions.Published;
            context.ElementShape.BackgroundImage = GetBackgroundImage(element, versionOptions);
        }

        protected override void OnExporting(Tile element, ExportElementContext context) {
            // Get the list of selected background content item id.
            var backgroundImageId = element.BackgroundImageId;

            // Load the actual background content item.
            var backgroundImage = GetBackgroundImage(element, VersionOptions.Latest);

            if (backgroundImage == null)
                return;

            // Use the content manager to get the content identities.
            var backgroundImageIdentity = _contentManager.GetItemMetadata(backgroundImage).Identity;

            // Add the content item identities to the ExportableData dictionary.
            context.ExportableData["BackgroundImage"] = backgroundImageIdentity.ToString();
        }

        protected override void OnImporting(Tile element, ImportElementContext context) {
            // Read the imported content identity from the ExportableData dictionary.
            var backgroundImageIdentity = context.ExportableData.Get("BackgroundImage");

            // Get the imported background content item from the ImportContentSesison store.
            var backgroundImage = context.Session.GetItemFromSession(backgroundImageIdentity);

            if (backgroundImage == null)
                return;

            // Get the background content item id (primary key value) for each background image.
            var backgroundImageId = backgroundImage.Id;

            // Assign the content id to the BackgroundImageId property so
            // that they contain the correct values, instead of the automatically imported value.
            element.BackgroundImageId = backgroundImageId;
        }

        private ImagePart GetBackgroundImage(Tile element, VersionOptions options) {
            return element.BackgroundImageId != null
                ? _contentManager.Get<ImagePart>(element.BackgroundImageId.Value, options, QueryHints.Empty.ExpandRecords<MediaPartRecord>())
                : null;
        }
    }
}
```
Let's go over that driver method-by-method.

##### OnBuildEditor #####
The `OnBuildEditor` method takes care of both displaying the element editor as well as handling its post-back. The first 4 lines initialize a new view model used by the editor template:

```
var viewModel = new TileViewModel {
    BackgroundImageId = element.BackgroundImageId?.ToString(),
    BackgroundSize = element.BackgroundSize
};
```

This simply constructs a new `TileViewModel` object and initializes it with existing Tile element data.
The `TileViewModel` is defined as follows:

```
using Orchard.MediaLibrary.Models;

namespace OffTheGrid.Demos.Layouts.ViewModels {
    public class TileViewModel {
        public string BackgroundImageId { get; set; }
        public string BackgroundSize { get; set; }
        public ImagePart BackgroundImage { get; set; }
    }
}
```

The reason I declared the `BackgroundImageId` of type `string` instead of `int` is simply because of the way the `MediaLibraryPicker` shape works, which we'll see in a moment. suffice it to say that it stores potentially multiple selected image IDs using a comma-separated list of ids.

After initializing the view model, we check to see if we're in post-back mode by checking whether the `context.Updater` is null or not. The `Updater` is specified by the framework when the user submitted the form on the element editor dialog, and really is just a reference to some controller, which implements the `TryUpdateModel` method. If the `Updater` is not null, we go ahead and use it to model bind the posted values against our view model like this:

```
// If an Updater is specified, it means the element editor form is being submitted
// and we need to read and store the submitted data.
if(context.Updater != null) {
    if (context.Updater.TryUpdateModel(viewModel, context.Prefix, null, null)) {
        element.BackgroundImageId = Orchard.Layouts.Elements.ContentItem.Deserialize(viewModel.BackgroundImageId).FirstOrDefault();
        element.BackgroundSize = viewModel.BackgroundSize?.Trim();
    }
}
```

If `TryUpdateModel` returns true, it means there were no model validation errors (which in our case will never happen since we didn't implement any validation rules in the first place, but still it's good practice to check for the return value in case we add model validation in the future). If there were no model validation errors, we go ahead and update the Tile `element` with the posted values.

Notice that we're converting the `viewModel.BackgroundImageId` from `string` to `IEnumerable<int>` using the `Deserialize` static method as defined on `Orchard.Layouts.Elements.ContentItem`. The `ContentItem` element just happened to provide such a convenient method, so I chose to reuse it here. Remember, the `MediaLibraryPicker` shape works with a `string` value, so we need to parse that string. Since we only support a single background image, we call `FirstOrDefault` to get the first selected image, if any. In addition to storing the `BackgroundImageId`, we also store the posted `BackgroundSize` value.

After handling the post-back scenario, the method continues to further prepare the view model for the editor template by loading the selected background image using the content manager:

```
viewModel.BackgroundImage = GetBackgroundImage(element, VersionOptions.Latest);
```

This too is necessary for the `MediaLibraryPicker` shape, which uses this information to display the currently selected background images.

The `GetBackgroundImage` method is a private convenience method implemented as follows:

```
private ImagePart GetBackgroundImage(Tile element, VersionOptions options) {
    return element.BackgroundImageId != null
        ? _contentManager.Get<ImagePart>(element.BackgroundImageId.Value, options, QueryHints.Empty.ExpandRecords<MediaPartRecord>())
        : null;
}
```

This basically uses the content manager to load the image content item using the specified version options. It also expands the `MediaPartRecord`, since we'll be accessing its `MediaUrl` down the line, so it makes sense to have Orchard join the `ImagePartRecord` and `MediaPartRecord` tables when loading the image.

Finally, we create a new shape of type `EditorTemplate`, where we specify all the common stuff it needs: `TemplateName`, `Model` and `Prefix`, and return an `EditorResult` containing this shape:

```
var editorTemplate = context.ShapeFactory.EditorTemplate(
    TemplateName: "Elements/Tile",
    Model: viewModel,
    Prefix: context.Prefix);

return Editor(context, editorTemplate);
```

The `EditorTemplate` shape is configured to use the *Elements/Tile* template, which maps to the Razor view *Views/EditorTemplates/Elements/Tile.cshtml*, which has the following code:

```
@using Orchard.ContentManagement
@model OffTheGrid.Demos.Layouts.ViewModels.TileViewModel

@Display.MediaLibraryPicker(
    FieldName: Html.FieldNameFor(m => m.BackgroundImageId),
    DisplayName: T("Background Images").ToString(),
    Multiple: true,
    Required: false,
    Hint: T("Optionally select one or more background images. Additional background images may be used for specific breakpoints, depending on the current theme's implementation.").ToString(),
    ContentItems: Model.BackgroundImage != null ? new ContentItem[] { Model.BackgroundImage.ContentItem } : null,
    PromptOnNavigate: false,
    ShowSaveWarning: false)

<fieldset>
    <div class="form-group">
        @Html.LabelFor(m => m.BackgroundSize, T("Background Size"))
        @Html.TextBoxFor(m => m.BackgroundSize, new { @class = "text medium" })
        @Html.Hint(T("Optionally specify a background-size value. Examples: auto, [length], cover, contain, initial and inherit. Substitute [length] with a widht and optionally a height, either in pixels or percentage."))
    </div>
</fieldset>
```

The above code listing shows the usage of the `MediaLibraryPicker` shape as provided by the `Orchard.MediaLibray` module. Notice especially the `ContentItems` property initialization: that property expects an enumerable of `ContentItem`, so we're constructing an array of `ContentItem` if we have a non-null `BackgroundImage` available. Otherwise we simply specify `null`, which is handled by the `MediaLibraryPicker` shape.

The second part of the editor shape template renders a simple text box for the `BackgroundSize` property.

##### OnDisplaying #####
Back to the Tile driver.

The `OnDisplaying` method is called whenever the element is about to be rendered, and gives us a chance to prepare some data or objects for easy consumption from the view. In our case, we need to load the selected background image, which we pass into the element shape. Since the `OnDisplaying` event is triggered for both the front-end and layout editor back-end, we get a chance to optimize the data for both display types.

> Whenever the layout editor renders its elements, it does so using the `"Design"` display type. This enables us to provide tailor-made views optimized for being displayed as part of the layout editor.

Since the background image is a content item, it can potentially be saved as a draft. This means that we'll want to get the latest version when being rendered as part of the layout editor, but only the published version when being rendered on the front-end. the following code implements that logic:

```
protected override void OnDisplaying(Tile element, ElementDisplayingContext context) {
    var versionOptions = context.DisplayType == "Design" ? VersionOptions.Latest : VersionOptions.Published;
    context.ElementShape.BackgroundImage = GetBackgroundImage(element, versionOptions);
}
```

Notice that we're reusing the private `GetBackgroundImage` convenience method.

Although not strictly necessary, providing a specific element view is always a good idea, since the default `Element` shape template does nothing more than rendering the element's type name. In the case of the `Tile` element, we want to at least render its child elements, and of course any selected background image. So go ahead and create a Razor view called *Tile.cshtml* in the *Views/Elements* folder, and enter the following code:

```
@using Orchard.ContentManagement;
@using Orchard.DisplayManagement.Shapes
@using Orchard.Layouts.Helpers
@using Orchard.MediaLibrary.Models
@using OffTheGrid.Demos.Layouts.Elements
@{
    var tagBuilder = (OrchardTagBuilder)TagBuilderExtensions.CreateElementTagBuilder(Model);
    var element = (Tile)Model.Element;
    var backgroundImage = (ImagePart)Model.BackgroundImage;

    if(backgroundImage != null) {
        var mediaPart = backgroundImage.As<MediaPart>();
        var backgroundSize = !String.IsNullOrWhiteSpace(element.BackgroundSize) ? element.BackgroundSize : "cover";
        tagBuilder.Attributes["style"] = String.Format("background-image: url('{0}'); background-size: {1};", mediaPart.MediaUrl, backgroundSize);
    }
}
@tagBuilder.StartElement
@DisplayChildren(Model)
@tagBuilder.EndElement
```

The first thing this view does is creating an `OrchardTagBuilder` object using the `TagBuilderExtensions` helper class to programmatically manipulate an HTML tag (`<div>` by default).

Then we check if we have a non-null `BackgroundImage` available (set from the `OnDisplaying` method in the driver).
If so, we add a `style` attribute to our tag with the background image URL and background image size, falling back to `"cover"` as a default size.

We then render the tag, and any children the Tile element may have.

##### OnExporting and OnImporting #####
The driver also contains implementations for the `OnExporting` and `OnImporting` methods, which are invoked when the content item with the `LayoutPart` is being exported or imported, respectively.

The only reason we need to implement these methods here is because we are referencing a content item by its primary key value (ID), which may be of a different value when content is exported and then imported into another database. To handle that scenario, we need to export and import a predictable, non-volatile identity of the referenced image content item. Fortunately, Orchard has excellent support for that. The content manager exposes a method called `GetItemMetadata` which takes a content item as an argument and returns an object describing all sorts of useful information about that content item, including its **identity**. It is this identity that we want to export, so that we can reference the correct content item during import.

The exporting code looks like this:

```
protected override void OnExporting(Tile element, ExportElementContext context) {
    // Get the list of selected background content item id.
    var backgroundImageId = element.BackgroundImageId;

    // Load the actual background content item.
    var backgroundImage = GetBackgroundImage(element, VersionOptions.Latest);

    if (backgroundImage == null)
        return;

    // Use the content manager to get the content identities.
    var backgroundImageIdentity = _contentManager.GetItemMetadata(backgroundImage).Identity;

    // Add the content item identities to the ExportableData dictionary.
    context.ExportableData["BackgroundImage"] = backgroundImageIdentity.ToString();
}
```

Notice the usage of the `ExportableData` dictionary, which is where we store information to be exported and imported. We could have just as well used the `Data` property on the element, but the `ExportableData` is a more formal container for this sort of data.

The importing code essentially mirrors the exporting code:

```
protected override void OnImporting(Tile element, ImportElementContext context) {
    // Read the imported content identity from the ExportableData dictionary.
    var backgroundImageIdentity = context.ExportableData.Get("BackgroundImage");

    // Get the imported background content item from the ImportContentSesison store.
    var backgroundImage = context.Session.GetItemFromSession(backgroundImageIdentity);

    if (backgroundImage == null)
        return;

    // Get the background content item id (primary key value) for each background image.
    var backgroundImageId = backgroundImage.Id;

    // Assign the content id to the BackgroundImageId property so
    // that they contain the correct values, instead of the automatically imported value.
    element.BackgroundImageId = backgroundImageId;
}
```

The magic line here is where we use `context.Session.GetItemFromSession` to get a reference of the content item we're interested in. The `context.Session` object is an `IContentImportSession` which maintains a dictionary of imported content items, keyed by content identity. Once we get our hands on the referenced content item, we use its primary key value (ID) to update the element's `BackgroundImageId`.

##### First Look #####
So far the process of writing a container element hasn't been that different from creating a regular element. The key differences so far are the fact that we derived our `Tile` class from `Container` instead of `Element` and some additional code to deal with the `MediaLibraryPicker` shape and additional import/export code because of that.

Let's see where that got us so far by launching the solution and adding a Tile element to the canvas:

Drag the Tile element onto the canvas:
![](./figures/fig-101-adding-tile-element-1.png)

Observe the Tile editor that we created and select your favorite background image:
![](./figures/fig-102-adding-tile-element-2.png)

Optionally specify a value for the *Background Size* setting. When left empty, the `"cover"` value will be used.
![](./figures/fig-103-adding-tile-element-3.png)

When you hit save, you should see the Tile element and its configured background image.
![](./figures/fig-104-adding-tile-element-4.png)

#### Implementing Containment ####
Surely enough, the Tile element works. However, try as you may, the element does not accept child elements to be dropped on it. Apparently there is some additional work to be done.  

#### The Tile Model Map ####
Due to some specific requirements of the layout editor implementation, we need to register a custom JavaScript element class that is configured such that it accepts child elements. To tell the layout editor what element class that is and what data to provide it with, we need the Tile element to hook into the editor model mapping process. 

Fortunately, this is easy. All we need to do is define a class that inherits from `LayoutModelMapBase<T>`, where `T` is our `Tile` type: 

```
using Orchard.Layouts.Services;

namespace OffTheGrid.Demos.Layouts.Handlers {
    public class TileModelMap : LayoutModelMapBase<Tile> {
    }
}
```

I added the above code to a file called *TileModelMap.cs* inside a new folder called *Handlers*. The folder name doesn't really matter, but I figured it's some kind of handler, so *Handler* seemed appropriate.

The `LayoutModelMapBase<T>` class does some fundamental things that is important for the client side Tile element (which we have yet to define) to work. First of all, it implements the `ILayoutModelMap` interface which was discussed earlier. The following code snippet shows the most interesting members of `LayoutModelMapBase<T>`:

```
public abstract class LayoutModelMapBase<T> : ILayoutModelMap where T : Element {
        public int Priority {
            get { return 10; }
        }

        public virtual string LayoutElementType { get { return typeof(T).Name; } }

        public virtual bool CanMap(Element element) {
            return element is T;
        }

        public Element ToElement(IElementManager elementManager, DescribeElementsContext describeContext, JToken node) {
            var descriptor = elementManager.GetElementDescriptorByType<T>(describeContext);
            var element = elementManager.ActivateElement<T>(descriptor);
            ToElement(element, node);
            return element;
        }

        void ILayoutModelMap.FromElement(Element element, DescribeElementsContext describeContext, JToken node) {
            FromElement((T)element, describeContext, node);
        }

        public virtual void FromElement(T element, DescribeElementsContext describeContext, JToken node) {
            node["data"] = element.Data.Serialize();
            node["htmlId"] = element.HtmlId;
            node["htmlClass"] = element.HtmlClass;
            node["htmlStyle"] = element.HtmlStyle;
            node["rule"] = element.Rule;
            node["isTemplated"] = element.IsTemplated;
            node["hasEditor"] = element.Descriptor.EnableEditorDialog;
            node["contentType"] = element.Descriptor.TypeName;
            node["contentTypeLabel"] = element.Descriptor.DisplayText.Text;
            node["contentTypeClass"] = element.DisplayText.Text.HtmlClassify();
            node["contentTypeDescription"] = element.Descriptor.Description.Text;
        }

        protected virtual void ToElement(T element, JToken node) {
            element.Data = ElementDataHelper.Deserialize((string)node["data"]);
            element.HtmlId = (string)node["htmlId"];
            element.HtmlClass = (string)node["htmlClass"];
            element.HtmlStyle = (string)node["htmlStyle"];
            element.IsTemplated = (bool)(node["isTemplated"] ?? false);
            element.Rule = (string)node["rule"];
        }
    }
```

As you can see, it handles the mapping of properties shared by all elements.

Another important aspect is the implementation of the `LayoutElementType` property, which returns the short name of the `T` type. In the case for our `TileModelMap<Tile>` class, this property will return the string `"Tile"`. This is important because we will use that name in our client side code that represents the Tile in the layout editor.

#### Client-Side Assets ####
When implementing custom elements with a customized client-side representation, it is important to realize that the layout editor is implemented using Angular and that it relies heavily on model binding. At the moment of this writing, there is no generic container element class that we could reuse. 

> Regular (non-container) elements do have a generic client-side representation, which i the `Content` element model. Future versions of Orchard may provide a generic `Container` representation to implement custom containers, but until then we'll have to implement our own. Which isn't necessarily a bad thing, since it provides us with a lot of customization opportunities as well.

So we'll just have to write our own.
When doing so, I chose to write JavaScript and Less files that take advantage of the Gulp pipeline support provided with Orchard, but this is not required.

##### Gulp and Assets.json #####
To see how the Gulp supports works, let's go ahead and create the following file & folder structure in the module:

	Assets
		Elements
  			Tile
				Directive.js
				Model.js
				Style.less
	Assets.json

These files won't be referenced by our views directly. Instead, we'll provide a configuration file called *Assets.json* that Gulp will use to compile, combine and minify the assets and output them to the *Scripts* and *Styles* folder.

The Assets.json file lives is to be created in the root of our module, and will have the following contents:

Assets.json:
```
[
    {
        "inputs": [
            "Assets/Elements/Tile/Style.less"
        ],
        "output": "Styles/TileElement.css"
    },
    {
        "inputs": [
            "Assets/Elements/Tile/Directive.js",
            "Assets/Elements/Tile/Model.js"
        ],
        "output": "Scripts/TileElement.js"
    }
]
```

If you have worked with Gulp before, the structure of this JSON file probably looks familiar. It's basically an array with inputs/output objects, where the inputs are compiled, combined and minified, which output is stored in the output file. The specified paths are relative to the module's root.

The above Assets.json configuration will generate four files:

- /Scripts/TileElement.js
- /Scripts/TileElement.min.js

And
 
- /Styles/TileElement.css
- /Styles/TileElement.min.css

With this in place, we are ready to provide actual contents to each asset file. We'll start with *Assets/Elements/Tile/Model.js*.

#### The Client-Side Tile Model ####
The following code shows the minimum amount of boilerplate code required to make the Tile element a container element:

```
var LayoutEditor;
(function (LayoutEditor) {

    LayoutEditor.Tile = function (data, contentType, htmlId, htmlClass, htmlStyle, isTemplated, rule, hasEditor, children) {
        var self = this;

        // Inherit from the Element base class.
        LayoutEditor.Element.call(self, "Tile", data, htmlId, htmlClass, htmlStyle, isTemplated, rule);

        // Inherit from the Container base class.
        LayoutEditor.Container.call(self, ["Canvas", "Grid", "Content"], children);

        // This Tile element is containable, which means it can be added to any container, including Tiles.
        self.isContainable = true;

        // Used by the layout editor to determine if it should launch
        // the element editor dialog when creating new Tile elements.
        // Also used by our "LayoutEditor.Template.Tile.cshtml" view that is used as the layout-tile directive's template.
        self.hasEditor = hasEditor;

        // The element type name, which is sent back to the element editor controller when being edited.
        self.contentType = contentType;

        // The "layout-common-holder" CSS class is used by the layout editor to identify drop targets.
        self.dropTargetClass = "layout-common-holder";

        // Implements the toObject serialization function.
        // This is called when the layout is being serialized into JSON.
        var toObject = self.toObject; // Get a reference to the default implementation before we override it.
        self.toObject = function () {
            var result = toObject(); // Invoke the original (base) implementation.
            result.children = self.childrenToObject();
            return result;
        };
    };

    // Implements the factory function invoked by the element factory.
    LayoutEditor.Tile.from = function (value) {
        var result = new LayoutEditor.Tile(
            value.data,
            value.contentType,
            value.htmlId,
            value.htmlClass,
            value.htmlStyle,
            value.isTemplated,
            value.rule,
            value.hasEditor,
            LayoutEditor.childrenFrom(value.children));

        // Initializes the toolbox specific properties.
        result.toolboxIcon = value.toolboxIcon;
        result.toolboxLabel = value.toolboxLabel;
        result.toolboxDescription = value.toolboxDescription;

        return result;
    };

    // Registers the factory function with the element factory.
    LayoutEditor.registerFactory("Tile", function (value) {
        return LayoutEditor.Tile.from(value);
    });

})(LayoutEditor || (LayoutEditor = {}));
```

It is not very complex, but there are a lot of things that aren't obvious without any proper documentation. The inline comments should be self-explanatory, but I will go over some of the more interesting pieces section by section.



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