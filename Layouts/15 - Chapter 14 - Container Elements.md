## Container Elements ##
In the previous chapter, we learned how to write custom elements by deriving fro, the `Element` base class, or a derivative thereof. Writing custom elements that can contain other elements are similar, but require you to inherit from the `Container` class instead, which in turn inherits from `Element`.

However, there is much more involved when writing custom container elements due to the way the layout editor works with container elements.

In this chapter, we will unravel all the steps that are involved when creating custom container elements.

### Writing Custom Container Elements ###
Writing custom container elements boils down to the following steps:

1. Implement a class that derives from `Container`.
2. Implement a driver for your container element class. As is the case with any element driver, your driver needs to be derived from `ElementDriver<T>`, where the generic `T` type argument is to be substituted with your custom element type.
3. Implement a class that derives from `LayoutModelMapBase<T>` that takes care of mapping values between the layout editor and elements back and forth, as well as define the client-side name of the element class.
4. Implement a client-side model for the element as well as an Angular directive, which in turn requires a template.
5. Implement a resource manifest provider that provides the client script (and potentially CSS style) to the layout editor.
6. Implement a shape table provider that invokes the resource manager to include the element resources.

We will go through each of these steps in a moment, but first I would like to quickly describe the various components involved. 

### The Container Class ###
Container elements are classes derived from the `Container` abstract base class, which lives in the `Orchard.Layouts.Elements` namespace.

The `Container` class adds single member called `Elements`, which is a list of elements:

```
public abstract class Container : Element {
    public IList<Element> Elements { get; set; }
}
```

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

The line of interest here is where we use `context.Session.GetItemFromSession` to get a reference of the content item we're interested in. The `context.Session` object is an `IContentImportSession` which maintains a dictionary of imported content items, keyed by content identity. Once we get our hands on the referenced content item, we use its primary key value (ID) to update the element's `BackgroundImageId`.

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

The easiest way to execute Gulp is to use the Task Runner Explorer:

![](./figures/fig-105-task-runner-explorer.png)

Simply right-click on the *build* task and hit *Run*. When you do this for the first time, the resulting files will be created, but not be made part of the project automatically. You'll have to do so manually (*Show All Files*, then right-click on the four generated files, and click *Include In Project*).

![](./figures/fig-106-including-generated-files.png)

> When the Task Runner Explorer shows the message "Failed to load. See output window", this is usually an indication that you haven't installed the NodeJS packages. This is easily fixed by opening the **Package.json** file in the **Solution Items/Gulp** solution folder, and simply saving that file (CTRL + S). When you do, Visual Studio will start downloading NodeJS packages, including Gulp. Gulp is used to compile, minify and bundle files such as CSS, LESS, JavaScript and TypeScript. Installing the various NodeJS packages may take a few minutes, so keep a close eye on the status bar (which should read "Installing packages" while installing packages, and "Installing paklages complete" when done.
> 
> Once the packages have been installed, click the refresh icon in the Task Runner Explorer and wait a second for it to refresh. Now you should see the **Tasks** and its child nodes **build**, **rebuild** and **watch**. Right-click on *build* and then left-click *Run* to have your Assets.json file executed.

With that in place, we are ready to provide actual contents to each asset file. We'll start with *Assets/Elements/Tile/Model.js*.

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

    // Registers the factory function with the element factory.
    LayoutEditor.registerFactory("Tile", function (value) {
        var tile = new LayoutEditor.Tile(
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
        tile.toolboxIcon = value.toolboxIcon;
        tile.toolboxLabel = value.toolboxLabel;
        tile.toolboxDescription = value.toolboxDescription;

        return tile;
    });

})(LayoutEditor || (LayoutEditor = {}));
```

It is not complex code, but there are quite a few things that aren't obvious without proper documentation. The inline comments should be self-explanatory, but I will go over some of the more interesting pieces section by section.

```
// Inherit from the Element base class.
LayoutEditor.Element.call(self, "Tile", data, htmlId, htmlClass, htmlStyle, isTemplated, rule);
```

As the comment indicates, this calls into the `LayoutEditor.Element` function, which will add all the required members to our Tile "class".

The second parameter being passed in, `"Tile`", is the client side type name of the element. This is not the same value as the server side element type name. This value must be the same as the value returned from `TileModelMap.LayoutElementType`.

The remaining parameters are provided by the constructor function.

The next few lines cause the Tile element to inherit from the `LayoutEditor.Container` class:

```
// Inherit from the Container base class.
LayoutEditor.Container.call(self, ["Canvas", "Grid", "Content"], children);
```

That call turns the Tile into an actual container so it can receive elements. The second argument being passed in is an array of element types that the Tile element can contain. However, this list is not exhaustive, since any elment whose `isContainable` field is set to `true` can be added to any container. This means that the Tile element itself for example can be added to other Tile elements, since we're setting exactly that property on the next line:

```
// This Tile element is containable, which means it can be added to any container, including Tiles.
self.isContainable = true;
```

The following code sections assignthe specified `hasEditor` and `contentType` arguments to fields on the Tile element.

```
// Used by the layout editor to determine if it should launch
// the element editor dialog when creating new Tile elements.
// Also used by our "LayoutEditor.Template.Tile.cshtml" view that is used as the layout-tile directive's template.
self.hasEditor = hasEditor;

// The element type name, which is sent back to the element editor controller when being edited.
self.contentType = contentType;
```

Since we're coding the element ourselves and since we know that it has an editor, we could have used a hardcoded value of `true` for the `hasEditor` field, but it's considered best practice to use the values provided from the server as much as possible.

The following line is key to turn our element into a drop target:

```
// The "layout-common-holder" CSS class is used by the layout editor to identify drop targets.
self.dropTargetClass = "layout-common-holder";
``` 

I think this should have been set by default, but for now we'll just have to do it ourselves.
The same goes for the next and final block of code of the constructor function:

```
// Implements the toObject serialization function.
// This is called when the layout is being serialized into JSON.
var toObject = self.toObject; // Get a reference to the default implementation before we override it.
self.toObject = function () {
    var result = toObject(); // Invoke the original (base) implementation.
    result.children = self.childrenToObject();
    return result;
};
``` 

The `Element` base class defines a `toObject` function that is called by the layout editor when the elements are being serialized into JSON. However, the `Element` class does not have a notion of child elements, and the `Container` class doesn't override the `toObject` to include its children, so once more it's our responsibility to handle that.

The final piece of the *Model.js* file registers our Tile class with the layout editor's element factory:

```
// Registers the factory function with the element factory.
LayoutEditor.registerFactory("Tile", function (value) {
    var tile = new LayoutEditor.Tile(
        value.data,
        value.contentType,
        value.htmlId,
        value.htmlClass,
        value.htmlStyle,
        value.isTemplated,
        value.rule,
        value.hasEditor,
        LayoutEditor.childrenFrom(value.children));

    return tile;
});
```

The first argument is again the client-side element type (`"Tile"`) which serves as a key into some internal dictionary. The second argument takes a function that is invoked by the factory when a new Tile element needs to be instantiated. This is where we "new up" our Tile class, passing in various bits and pieces of the specified `value` argument. This value is an object that contains all the values provided from the `TileModelMap` class.

#### The Client-Side Tile Directive ####
In addition to implementing a client-side model for the tile element, we also need to implement an Angular directive, so that Angular templates can render the elements using their specific templates.

The Tile directive (Directive.js) requires the following code in order for it to function asa container:

```
angular
    .module("LayoutEditor")
    .directive("orcLayoutTile", ["scopeConfigurator", "environment",
        function (scopeConfigurator, environment) {
            return {
                restrict: "E",
                scope: { element: "=" },
                controller: ["$scope", "$element", "$attrs",
                    function ($scope, $element, $attrs) {
                        scopeConfigurator.configureForElement($scope, $element);
                        scopeConfigurator.configureForContainer($scope, $element);
                        $scope.sortableOptions["axis"] = "y";
                    }
                ],
                templateUrl: environment.templateUrl("Tile"),
                replace: true
            };
        }
    ]);
```

A few important aspects are worth mentioning:

For starters, it is important that the directive is defined as part of the `"LayoutEditor"` Angular module, and that the directive name starts with `"orcLayout"`, followed by the client-side element type name (`"Tile"`). This is a convention used by the `orcLayoutChild` directive that is provided by the Layouts module.

Next, we need to setup the controller function such that it configures the Angular *$scope* (which is more or less the model of the directive) using the injected `scopeConfigurator`, which adds all the necessary functions to the scope and the element. Also, since we're configuring a container, we need to configure the `sortableOptions`, which is specific to the `Sortable` jQueryUI widget used by the layout editor.

Finally, we need to configure the directive's template, which we do using the injected `environment` service and its `templateUrl` function. This function essentially constructs a relative path to the `Template` controller of the Layouts module, which returns a view based on the specified parameter. When passing in `"Tile"`, the resulting template URL will become: `"Admin/Layouts/Template/Get/Tile"`. The `TemplateController.Get` action looks like this:

```
public ContentResult Get(string id) {
    var shapeType = String.Format("LayoutEditor_Template_{0}", id);

    var content = _cacheManager.Get(shapeType, context => {
        var shape = _shapeFactory.Value.Create(shapeType);
        return _shapeDisplay.Display(shape);
    });

    return Content(content, "text/html");
}
```

This means that we'll have to provide a proper Shape template named "LayoutEditor.Template.Tile.cshtml" for our directive.

#### The Client-Side Tile Directive Template ####
After implenting the element's directive, we need to provide the directive's template, which is implemented as a shape template. Given the implementation of the `TemplateController` as we've seen just now, we need to create the following Razor view in the Views folder of our module: *"LayoutEditor.Template.Tile.cshtml"*, which requires the following markup:

```
<div class="layout-element-wrapper layout-element-background" ng-class="{'layout-container-empty': getShowChildrenPlaceholder()}">
    <ul class="layout-panel layout-panel-main">
        <li class="layout-panel-item layout-panel-label">Tile</li>
        <li class="layout-panel-item layout-panel-action layout-panel-action-edit" ng-show="{{element.hasEditor}}" title="Edit tile (Enter)" ng-click="edit()"><i class="fa fa-code"></i></li>
        @Display(New.LayoutEditor_Template_Properties(ElementTypeName: "tile"))
        <li class="layout-panel-item layout-panel-action" title="@T("Delete tile (Del)")" ng-click="delete(element)" ng-show="element.canDelete()"><i class="fa fa-remove"></i></li>
        <li class="layout-panel-item layout-panel-action" title="@T("Move tile up (Ctrl+Up)")" ng-click="element.moveUp()" ng-show="element.canMoveUp()"><i class="fa fa-chevron-up"></i></li>
        <li class="layout-panel-item layout-panel-action" title="@T("Move tile down (Ctrl+Down)")" ng-click="element.moveDown()" ng-show="element.canMoveDown()"><i class="fa fa-chevron-down"></i></li>
    </ul>
    <div class="layout-container-children-placeholder" style="{{element.getTemplateStyles()}}">
        @T("Drag an element from the toolbox and drop it here to add content.")
    </div>
    @Display(New.LayoutEditor_Template_Children())
</div>
```

All of the Angular bindings you see in this file are bindings against methods and fields of the `$scope`, which is configured by our directive code. This template takes care of the entire rendering of our element's representation in the layout editor. This also means that if we add additional information to our Tile model, we could bind that information with this template. This is how we can make our template more specific to the Tile element, which we'll see later.


#### Registering The Tile Assets ####
Although we created various Tile assets and configured Gulp to generate their output files, we haven't actually included those files anywhere yet, and the Layouts module isn't doing that for us automatically. So how do we take care of that?

Your initial thoughts may be to simply include those files from the Tile shape templates. But those templates are rendered *after the layout editor has been rendered* and using AJAX calls. So even though the `Script.Include` and `Style.Include` calls would register our assets, it won't do us any good since the main page will already have been rendered, and the HTML being sent back is just the HTML output of a single element, not the entire HTML document. So if that doesn't work, what do we do?

The key challenge here is that we need a way to be able to register our assets *at the same time the layout editor is rendered*. This will cause all elements' assets to be rendered during the initial page load, which is good.
However, there's no formal extensibility point or event that we can handle to register those resources.

So we do the next best thing: we implement an `IShapeTableProvider` and handle the `OnDisplaying` event for the `EditorTemplate` shape, and check if its `TemplateName` value equals `"Parts.Layout"`, since that's the template name used by the `LayoutPartDriver` when creating the layout editor object.

> This is in fact a major limitation currently with the layout editor in case its used outside the context of the `LayoutPart`. For example, if you were to manually construct the `LayoutEditor` object yourself using the `ILayoutEditorFactory` and then render that object using the `Html.Editor` HTML helper, the layout editor would be rendered, but the handler created in the shape table provider would not be triggered, since we're not rendering any `EditorTemplate` shape. At the time of this writing, there is an open bug for this on GitHub (https://github.com/OrchardCMS/Orchard/issues/6221).

So let's go ahead and create the following class in the *Handlers* folder:

```
using Orchard.DisplayManagement.Descriptors;
using Orchard.Environment;
using Orchard.UI.Resources;

namespace OffTheGrid.Demos.Layouts.Handlers {
    public class TileResourceRegistrations : IShapeTableProvider {
        private readonly Work<IResourceManager> _resourceManager;
        public TileResourceRegistrations(Work<IResourceManager> resourceManager) {
            _resourceManager = resourceManager;
        }

        public void Discover(ShapeTableBuilder builder) {
            builder.Describe("EditorTemplate").OnDisplaying(context => {
                if (context.Shape.TemplateName != "Parts.Layout")
                    return;

                _resourceManager.Value.Require("stylesheet", "TileElement");
                _resourceManager.Value.Require("script", "TileElement");
            });
        }
    }
}
```

What we're doing here is adding a handler for the `OnDisplaying` event of the `EditorTemplate` shape. If the template being displayed is the `"Parts.Layout"` template, we register two resources with the injected resource manager:

- "TileElement" stylesheet
- "TileElement" script

We haven't defined those resources yet, but we'll get to that in a moment. You may be wondering why I didn;t use the `Include` method of the resource manager instead of the `Require` method, because then we wouldn't have to create a resource manifest provider that defines these resources. The sole reason is that we need to ensure the layout editor scripts are included before our own scripts, and the only way to ensure that is to have our resources depend on that resource. Since we can't declare that dependency directly (as far as I know, that is), we will have to rely on a resource manifest provider to do so.

#### The Resource Manifest Provider ####
So here we are, at the final piece of the puzzle of implementing custom container elements. Let's go ahead and create the following class in a new folder called *ResourceManifests*:

```
using Orchard.UI.Resources;

namespace OffTheGrid.Demos.Layouts.Handlers {
    public class TileResourceManifest : IResourceManifestProvider {
        public void BuildManifests(ResourceManifestBuilder builder) {
            var manifest = builder.Add();
            manifest.DefineStyle("TileElement").SetUrl("TileElement.min.css", "TileElement.css");
            manifest.DefineScript("TileElement").SetUrl("TileElement.min.js", "TileElement.js").SetDependencies("Layouts.LayoutEditor");
        }
    }
}
```

Nothing fancy here, but do notice the `"TileElement"`'s dependency on the `"Layouts.LayoutEditor"` resource, which is provided by the Layouts module.

#### Second Look ####
Man, that was a lot of code we had to add. I don't know about you, but I'm ready to check it out and see if it actually works!

When you launch the site now, you should be able to drag & drop the Tile element like before, provide a background image and background size, and see the element appear on the canvas. Only this time , we no longer see the selected background being rendered:

![](./figures/fig-107-tile-element-refreshed.png)

But at least we're able to add elements to it:

![](./figures/fig-108-element-inside-tile-element.png)

And when we save this page and view it from the front-end, we see our Tile element with its configured background image and contained Html element rendered as expected:

![](./figures/fig-109-element-inside-tile-element-front-end.png)

The reason we're not seeing the selected background image in the layout editor is because we're no longer rendering the element using the *Elements/Tile.cshtml* template. Instead, the Angular directive's template use now used, which is necessary to render its children.

> The reason regular *Content* client-side elements are able to render their templates using the *Design* display type is because those elements are actually *pre-rendered* on the server side. The resulting HTML is then passed into the `Content` client-side model, and used by its Angular directive's template to display it. We could in principle do the same for the Tile element, if only it didn't need the ability to receive child elements (from drag & drop operations for example).

All this means is that instead of relying on the *Elements/Tile.cshtml* shape template to render the background, we'll just have to render it from our directive's template. This requires us to provide the image URL to our client-side Tile model and update the directive's template to model-bind against that value.

And while we're at it, we may just as well add support for editing the background size property of the element from the property editor, so that the user doesn't have to launch the element's editor dialog to change the background size. Instead, they can do it directly from the layout editor and instantly see the result.

Let's see how that works.

#### Updating TileModelMap ####
The first thing we need to do is get the background image URL and background image size down to our client-side Tile model. To do so, update the `TileModelMap` class as follows:

```
using Newtonsoft.Json.Linq;
using OffTheGrid.Demos.Layouts.Elements;
using Orchard.ContentManagement;
using Orchard.Layouts.Services;
using Orchard.MediaLibrary.Models;

namespace OffTheGrid.Demos.Layouts.Handlers {
    public class TileModelMap : LayoutModelMapBase<Tile> {
        private IContentManager _contentManager;

        public TileModelMap(IContentManager contentManager) {
            _contentManager = contentManager;
        }

        public override void FromElement(Tile element, DescribeElementsContext describeContext, JToken node) {
            base.FromElement(element, describeContext, node);

            var backgroundImage = element.BackgroundImageId != null
                ? _contentManager.Get<ImagePart>(element.BackgroundImageId.Value)
                : default(ImagePart);
            var backgroundImageUrl = backgroundImage?.As<MediaPart>().MediaUrl;

            node["backgroundUrl"] = backgroundImageUrl;
            node["backgroundSize"] = element.BackgroundSize;
        }

        protected override void ToElement(Tile element, JToken node) {
            base.ToElement(element, node);

            element.BackgroundSize = (string)node["backgroundSize"];
        }
    }
}
```

OK, that's a lot more code than before. What we're doing here is implementing the `FromElement` and `ToElement` methods to map the background image URL and background size properties. Remember, these values are sent to and received from the client-side representation of the Tile element.

#### Updating Tile/Model.js ####
Since we want to model-bind the background image URL and background size properties with the Tile element directive, we'll need to stick those values into our Tile model. The updated file looks like this:

```
var LayoutEditor;
(function (LayoutEditor) {

    LayoutEditor.Tile = function (data, contentType, htmlId, htmlClass, htmlStyle, isTemplated, rule, hasEditor, backgroundUrl, backgroundSize, children) {
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

        // The configured background image URL and background size.
        self.backgroundUrl = backgroundUrl;
        self.backgroundSize = backgroundSize;

        // Implements the toObject serialization function.
        // This is called when the layout is being serialized into JSON.
        var toObject = self.toObject; // Get a reference to the default implementation before we override it.
        self.toObject = function () {
            var result = toObject(); // Invoke the original (base) implementation.
            result.children = self.childrenToObject();
            result.backgroundUrl = self.backgroundUrl;
            result.backgroundSize = self.backgroundSize;
            return result;
        };

        // Override the getEditorObject so we can include our backgroundSize property.
        // This is called when the element editor dialog is being invoked and we need to
        // pass in the client side values.
        var getEditorObjectBase = this.getEditorObject;
        this.getEditorObject = function () {
            var props = getEditorObjectBase();
            props.BackgroundSize = self.backgroundSize;
            return props;
        }

        // Executed after the element editor dialog closes.
        this.applyElementEditorModel = function (data) {
            self.backgroundUrl = data.backgroundUrl;
            self.backgroundSize = data.backgroundSize;
            self.applyBackground();
        }

        this.hasBackground = function () {
            return self.backgroundUrl && self.backgroundUrl.length > 0;
        };

        this.applyBackground = function () {
            if (self.hasBackground()) {
                var styles = {
                    "background-image": "url('" + self.backgroundUrl + "')",
                    "background-size": self.backgroundSize && self.backgroundSize.length > 0 ? self.backgroundSize : "cover"
                };

                if (self.children.length == 0)
                    self.templateStyles = styles;
                else
                    self.containerTemplateStyles = styles;
            }
            else {
                self.templateStyles = {};
                self.containerTemplateStyles = {};
            }
        }

        self.applyBackground();
    };

    // Registers the factory function with the element factory.
    LayoutEditor.registerFactory("Tile", function (value) {
        return new LayoutEditor.Tile(
            value.data,
            value.contentType,
            value.htmlId,
            value.htmlClass,
            value.htmlStyle,
            value.isTemplated,
            value.rule,
            value.hasEditor,
            value.backgroundUrl,
            value.backgroundSize,
            LayoutEditor.childrenFrom(value.children));
    });

})(LayoutEditor || (LayoutEditor = {}));
```

Let's go over each section that's been added from top to bottom:

```
LayoutEditor.Tile = function (data, contentType, htmlId, htmlClass, htmlStyle, isTemplated, rule, hasEditor, backgroundUrl, backgroundSize, children) {
```

Notice that I added the `backgroundUrl` and `backgroundSize` parameters, which are stored here:

```
// The configured background image URL and background size.
self.backgroundUrl = backgroundUrl;
self.backgroundSize = backgroundSize;
```

Since we want to allow the user to change the background size directly from the layout editor, we'll need to ensure that this value gets serialized:

```
// Implements the toObject serialization function.
// This is called when the layout is being serialized into JSON.
var toObject = self.toObject; // Get a reference to the default implementation before we override it.
self.toObject = function () {
    var result = toObject(); // Invoke the original (base) implementation.
    result.children = self.childrenToObject();
    result.backgroundUrl = self.backgroundUrl;
    result.backgroundSize = self.backgroundSize;
    return result;
};
```

The same goes for the `backgroundUrl` property. Although the user can't change this value directly, the layout editor uses the serialization mechanism for copy & paste. If we don't include the `backgroundUrl` as part of this serialization process, it will appear as if we "lost" the background when the user copies and pastes a Tile element (although in reality the selected background ID will still be part of the element's Data property).

The following code handles the scenario where the user invokes the element's editor dialog. Since the user can change the background size directrly from the layout editor, we should pass along that value when the element editor is invoked. To do that, we need to make sure the value becomes part of the object created by the `getEditorObject` function:

```
// Override the getEditorObject so we can include our backgroundSize property.
// This is called when the element editor dialog is being invoked and we need to
// pass in the client side values.
var getEditorObjectBase = this.getEditorObject;
this.getEditorObject = function () {
    var props = getEditorObjectBase();
    props.BackgroundSize = self.backgroundSize;
    return props;
}
```

Notice the name `BackgroundSize`: this needs to exactly match the key as used by the `Tile` element C# class for its `BackgroundSize` property. This editor object is essentially converted into a dictionary and merged into the element's Data dictionary when rendering the element's editor dialog.

We also need to handle the reversed scenario, where the user changes the background size value from the element editor dialog screen and then saves that change. We'll want to update the client-side Tile model's `backgroundSize` as well as `backgroundUrl` field to reflect any changes made. That's what the following code is for:

```
// Executed after the element editor dialog closes.
this.applyElementEditorModel = function (data) {
    self.backgroundUrl = data.backgroundUrl;
    self.backgroundSize = data.backgroundSize;
    self.applyBackground();
}
```

The `applyElementEditorModel` is called by the layout editor when the element editor dialog is closed using the `Save` command. In our implementation, we're calling a function called `applyBackground`, which is a function defined next:

```
this.hasBackground = function () {
    return self.backgroundUrl && self.backgroundUrl.length > 0;
};

this.applyBackground = function () {
    if (self.hasBackground()) {
        var styles = {
            "background-image": "url('" + self.backgroundUrl + "')",
            "background-size": self.backgroundSize && self.backgroundSize.length > 0 ? self.backgroundSize : "cover"
        };

        if (self.children.length == 0)
            self.templateStyles = styles;
        else
            self.containerTemplateStyles = styles;
    }
    else {
        self.templateStyles = {};
        self.containerTemplateStyles = {};
    }
}
```

What this code does is preparing a JSON object called `styles`, and then depending on the length of the `children` array, assignes it to either the `templateStyles` or the `containerTemplateStyles` field. So what are those things?

Looking back at our `Views/LayoutEditor.Template.Tile.cshtml` view, notice that we're binding the `style` attribute on the children placeholder `<div>` element against a function called `getTemplateStyles()` here:

```
<div class="layout-container-children-placeholder" style="{{element.getTemplateStyles()}}">
    @T("Drag an element from the toolbox and drop it here to add content.")
</div>
```

That function is defined by the client-side `Element` model, which converts the `templateStyles` JSON object into a CSS string.

Similarly, the client-side `Container` model provides a function called `getContainerTemplateStyles()` that turns the `containerTemplateStyles` JSON object into a CSS string as well. That value is then bound against some `<div>` element in the *LayoutEditor.Template.Children.cshtml* view provided by the Layouts module, which looks like this:

```
<div class="layout-children clearfix" ng-model="element.children" ui-sortable="sortableOptions" style="{{element.getContainerTemplateStyles()}}">
    <div class="clearfix" ng-repeat="child in element.children" ng-class="getClasses(child)" ng-mouseenter="child.setIsActive(true)" ng-mouseleave="child.setIsActive(false)" ng-click="click(child, $event)" tabindex="{{$id}}">
        <orc-layout-child element="child" />
    </div>
</div>
```

So essentially, the `templateStyles` and `containerTemplateStyles` fields allow us to provide styles on two different places on the element directive template, which we use to set a background image and background size.

The reason we're setting both fields is because the first one is only rendered if our element does not contain any elements, and the second one is only rendered if there is at least one child element.

#### Updating Tile/Directive.js ####
Although not strictly necessary, we should update our Tile directive file as well. The reason being that when the user changes the background size property from the property window of the element, ideally we would want to see that change reflected instantly. Although that text field is bound against the `backgroundSize` field of the element, that is not enough to reflect the change, since setting the background image is done by the `applyBackground` function.

So let's go ahead and add a *linker* to our directive:

```
link: function ($scope, $element, $attrs) {
    $element.on("change", "[ng-model='element.backgroundSize']", function () {
        $scope.element.applyBackground();
    });
}
```

This linker just binds the `change` event of the text field bound against the `element.backgroundSize` property with an event handler. Whenever the contents change of that text field, we call the `applyBackground` function on the bound element.

The complete code for the directive looks like this:

```
angular
    .module("LayoutEditor")
    .directive("orcLayoutTile", ["scopeConfigurator", "environment",
        function (scopeConfigurator, environment) {
            return {
                restrict: "E",
                scope: { element: "=" },
                controller: ["$scope", "$element", "$attrs",
                    function ($scope, $element, $attrs) {
                        scopeConfigurator.configureForElement($scope, $element);
                        scopeConfigurator.configureForContainer($scope, $element);
                        $scope.sortableOptions["axis"] = "y";
                    }
                ],
                templateUrl: environment.templateUrl("Tile"),
                replace: true,
                link: function ($scope, $element, $attrs) {
                    $element.on("change", "[ng-model='element.backgroundSize']", function () {
                        $scope.element.applyBackground();
                    });
                }
            };
        }
    ]);
```

#### Updating LayoutEditor.Template.Tile.cshtml ####
Finally, we'll also need to update the directive's template so that the property editor actually displays a text field for the background size property. The easiest way to do that is by providing a list of `LayoutEditorPropertiesItem` to the `LayoutEditor_Template_Properties` shape that's being used by the directive template. This enables us to simply declare additional property editors without having to duplicate existing ones and without having to provide markup.

The updated template looks like this:

```
@using Orchard.Layouts.ViewModels
@{
    var additionalProperties = new[] {
        new LayoutEditorPropertiesItem() {
            Label = T("Background Size:").ToString(),
            Model = "element.backgroundSize"
        }
    };
}
<div class="layout-element-wrapper layout-element-background" ng-class="{'layout-container-empty': getShowChildrenPlaceholder()}">
    <ul class="layout-panel layout-panel-main">
        <li class="layout-panel-item layout-panel-label">Tile</li>
        <li class="layout-panel-item layout-panel-action layout-panel-action-edit" ng-show="{{element.hasEditor}}" title="Edit tile (Enter)" ng-click="edit()"><i class="fa fa-code"></i></li>
        @Display(New.LayoutEditor_Template_Properties(ElementTypeName: "tile", Items: additionalProperties))
        <li class="layout-panel-item layout-panel-action" title="@T("Delete tile (Del)")" ng-click="delete(element)" ng-show="element.canDelete()"><i class="fa fa-remove"></i></li>
        <li class="layout-panel-item layout-panel-action" title="@T("Move tile up (Ctrl+Up)")" ng-click="element.moveUp()" ng-show="element.canMoveUp()"><i class="fa fa-chevron-up"></i></li>
        <li class="layout-panel-item layout-panel-action" title="@T("Move tile down (Ctrl+Down)")" ng-click="element.moveDown()" ng-show="element.canMoveDown()"><i class="fa fa-chevron-down"></i></li>
    </ul>
    <div class="layout-container-children-placeholder" style="{{element.getTemplateStyles()}}">
        @T("Drag an element from the toolbox and drop it here to add content.")
    </div>
    @Display(New.LayoutEditor_Template_Children())
</div>
```

Notice that we're initializing an array of type `LayoutEditorPropertiesItem[]` with a single item. This item is initialized with a user-friendly label and a model value of `"element.backgroundSize"`. This is a client-side expression that will be assigned to an Angular directive in the Razor view used by the `LayoutEditor_Template_Properties` shape, which you can find in the Views folder of the Layouts module.

#### Third Look ####
Man, that's a lot of code going on. Also, this process is kind of hard to discover yourself and takes a lot of trial and error. It's a good thing you're reading about it here though, because once you're able to build this kind of elements, you'll be able to build any kind of elements. And that should be worth something.

So let's see where this got us so far by launching the site and navigating to a page editor with the Tile element on it:

![](./figures/fig-110-tile-element-with-background.png)

> Whenever you make changes to client-side assets, make sure to execute the Build task from the Task Runner Explorer to update the generated files.

That seems to work nicely, although we did lose the green selection border around the Tile element. Fixing that probably requires some refactoring of some templates, but for now we can do without that.

Notice that as you change the value for the Background Size property, the change is reflected instantly. Also notice that when you launch the element editor dialog, the same background size value is used. When you change that value from the editor and hit Save, that value is once again round-tripped back to the layout editor.

One final touch we could make is to provide some CSS that gives the child elements a semi-transparent background to increase the contrast when working with the layout editor and background images.

Obviously I knew we would get here, which is why I had you include a Style.less file earlier on. Open that file (Assets/Elements/Tile/Style.less) and enter the following CSS rules:

```
.layout-element-background {
    .layout-content-markup {
        background: rgba(255, 255, 255, 0.7);
    }
}
```

This basically gives all child elements a semi-transparent (70% opaque) white background. Make sure to execute the Build task from the Task Runner Explorer, and then refresh the page:

![](./figures/fig-111-tile-element-with-background-and-semi-transparent-child-elements.png)


### Summary ###
That was quite a journey. In this chapter, we learned all of the intricacies of writing custom container elements, and I'll admit it: it is far from straightforward.

Implementing custom container elements require no less than the following 6 steps:

1. Derive from `Container`;
2. Create a driver;
3. Derive from `LayoutModelMapBase<T>`;
4. Implement a client-side model and directive;
5. Implement a resource manifest provider;
6. Implement a shape table provider.

This developer's story is bound to be improved in newer versions of Orchard, but this is what it is for the time being. And despite the fact that implementing custom container isn't very straightforward, it is worth the effort.   