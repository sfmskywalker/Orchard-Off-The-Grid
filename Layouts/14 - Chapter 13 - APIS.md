## APIs ##
So far we've seen how to write custom elements by creating custom element classes and harvesters. In this chapter, we will learn about APIs at our disposal to programmatically work with elements. More specifically, we will learn how to:

- Work with the element manager to query all available element categories and descriptors;
- Use the element factory to instantiate elements;
- Render elements and layouts of elements with the layout manager and element manager;
- Serialize and deserialize elements;
- Handle element events by implementing the `IElementEventHandler` interface;
- Reuse the layout editor from our own module.

Knowing about these APIs will enable you to take advantage of all that the Layouts module has to offer and implement all sorts of applications yourself. For example, in the next part, we will write a *SlideShowPart* that uses the APIs from this chapter to enable the user to create slides that are based on layouts. To achieve such functionality, it is important to have decent understanding of how the various APIs work, and how they work together.

We'll get started with an overview of the most interesting APIs first, and then onward with some examples to see how they work. 

### Managing Layouts and Elements ###
The Layouts module comes with two services that you can use to programmatically manage layouts and elements:

- **ILayoutManager**
- **IElementManager**

#### ILayoutManager ####
The Layout Manager itself uses the Element Manager to manage elements on a somewhat higher level. For example, it has a method that accepts a serialized string of elements which it can render into a complete tree of element shapes. It also provides methods to manage Layout content items and apply layout templates.

The following code snippet lists all members of the `ILayoutManager` class:

```
public interface ILayoutManager : IDependency {
    IEnumerable<LayoutPart> GetTemplates();
    IEnumerable<LayoutPart> GetLayouts();
    LayoutPart GetLayout(int id);
    IEnumerable<Element> LoadElements(ILayoutAspect layout);
    dynamic RenderLayout(string data, string displayType = null, IContent content = null);
    IEnumerable<Element> ApplyTemplate(LayoutPart layout, LayoutPart templateLayout);
    IEnumerable<Element> ApplyTemplate(LayoutPart layout);
    IEnumerable<Element> ApplyTemplate(IEnumerable<Element> layout, IEnumerable<Element> templateLayout);
    IEnumerable<Element> DetachTemplate(IEnumerable<Element> elements);
    IEnumerable<LayoutPart> GetTemplateClients(int templateId, VersionOptions versionOptions);
    IEnumerable<Element> CreateDefaultLayout();
    void Exporting(ExportLayoutContext context);
    void Exported(ExportLayoutContext context);
    void Importing(ImportLayoutContext context);
    void Imported(ImportLayoutContext context);
    void ImportCompleted(ImportLayoutContext context);
}
```

The following table provides a description for each member of `ILayoutManager`:

<table>
<thead>
    <tr>
        <th>Member</th>
        <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>GetTemplates</td>
        <td>Returns a list of all content items with a <strong>LayoutPart</strong> attached whose <strong>IsTemplate</strong> content type/part setting is set to true. This method is used to populate the Templates dropdownlist on the layout editor from which the user can choose a layout template to apply to the current layout.</td>
    </tr>
    <tr>
        <td>GetLayouts</td>
        <td>Returns a list of all content items with a <strong>LayoutPart</strong> attached. This method is currently not used anywhere.</td>
    </tr>
    <tr>
        <td>GetLayout</td>
        <td>Returns a content item with a <strong>LayoutPart</strong> attached with the specified content item ID. It's basically a wrapper around the IContentManager.Get<LayoutPart>() method.</td>
    </tr>
    <tr>
        <td>LoadElements</td>
        <td>Deserializes the LayoutData string value of the specified ILayoutAspect (which is typically just a content item with a LayoutPart which itself impolements this interface) and returns a list of elements, which represents the layout's elements.</td>
    </tr>
    <tr>
        <td>RenderLayout</td>
        <td>Deserializes the specified data string into a list of elements (internally invoking LoadElements) and then invokes <strong>IElementDisplay.DisplayElements</strong> internally, which returns an hierarchy of element shapes. In addition, this method accepts two optional arguments: displayType and content. The display type is used to add shape alternates. The content argument should be specified if there is any. For example, if the LayoutPartDriver would use this method, it would pass in a reference to the LayoutPart instance itself. Note however that nothing is currently using this method. Instead, the LayoutPartDiver invokes the IElementDisplay service directly.</td>
    </tr>
    <tr>
        <td>ApplyTemplate</td>
        <td>Used by the layout editor to apply the elements of the selected layout template to the elements of the current layout elements, essentially combining the two sets of elements and returning the resulting list of elements.</td>
    </tr>
    <tr>
        <td>DetachTemplate</td>
        <td>Used by the layout editor to essentially strip out all sealed (templated) elements of the specified list and returning the result of that.</td>
    </tr>
    <tr>
        <td>GetTemplateClients</td>
        <td>Returns a list of all content items that use the specified layout template. This is used by the layout content handler to re-apply an updated layout template to all content items using the updated layout template. This is in fact what enables content items using a given template to reflect updates made to its template. For eample, if you changed the contents of an element in a layout template, you would see that change reflected in all content items using that template.</td>
    </tr>
    <tr>
        <td>CreateDefaultLayout</td>
        <td>Returns a list of elements that represents a default layout which consists of a root Canvas, a Grid, Row and Column. Currently not used anymore.</td>
    </tr>
    <tr>
        <td>Exporting</td>
        <td>Invokes the Exporting method on the <strong>IElementManager</strong> service for each element in the specified list of elements (as provided via the context argument).</td>
    </tr>
    <tr>
        <td>Exported</td>
        <td>Invokes the Exported method on the <strong>IElementManager</strong> service for each element in the specified list of elements (as provided via the context argument).</td>
    </tr>
    <tr>
        <td>Importing</td>
        <td>Invokes the Importing method on the <strong>IElementManager</strong> service for each element in the specified list of elements (as provided via the context argument).</td>
    </tr>
    <tr>
        <td>Imported</td>
        <td>Invokes the Imported method on the <strong>IElementManager</strong> service for each element in the specified list of elements (as provided via the context argument).</td>
    </tr>
    <tr>
        <td>ImportCompleted</td>
        <td>Invokes the ImportCompleted method on the <strong>IElementManager</strong> service for each element in the specified list of elements (as provided via the context argument).</td>
    </tr>
</tbody>
</table>

#### IElementManager ####
The `IElementManager` service deals with a variety of things when it comes to elements. The following code listing lists all of its members:

```
public interface IElementManager : IDependency {
    IEnumerable<ElementDescriptor> DescribeElements(DescribeElementsContext context);
    IEnumerable<CategoryDescriptor> GetCategories(DescribeElementsContext context);
    ElementDescriptor GetElementDescriptorByTypeName(DescribeElementsContext context, string typeName);
    ElementDescriptor GetElementDescriptorByType<T>(DescribeElementsContext context) where T : Element;
    ElementDescriptor GetElementDescriptorByType<T>() where T : Element;
    Element ActivateElement(ElementDescriptor descriptor, Action<Element> initialize = null);
    T ActivateElement<T>(ElementDescriptor descriptor, Action<T> initialize = null) where T : Element;
    T ActivateElement<T>(Action<T> initialize = null) where T : Element;
    IEnumerable<IElementDriver> GetDrivers<TElement>() where TElement : Element;
    IEnumerable<IElementDriver> GetDrivers(ElementDescriptor descriptor);
    IEnumerable<IElementDriver> GetDrivers(Element element);
    IEnumerable<IElementDriver> GetDrivers(Type elementType);
    IEnumerable<IElementDriver> GetDrivers();
    EditorResult BuildEditor(ElementEditorContext context);
    EditorResult UpdateEditor(ElementEditorContext context);
    void Saving(LayoutSavingContext context);
    void Removing(LayoutSavingContext context);
    void Exporting(IEnumerable<Element> elements, ExportLayoutContext context);
    void Exported(IEnumerable<Element> elements, ExportLayoutContext context);
    void Importing(IEnumerable<Element> elements, ImportLayoutContext context);
    void Imported(IEnumerable<Element> elements, ImportLayoutContext context);
    void ImportCompleted(IEnumerable<Element> elements, ImportLayoutContext context);
}
```

The following table describes the various methods:

<table>
<thead>
    <tr>
        <th>Member</th>
        <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>DescribeElements</td>
        <td>Returns a list of all available element descriptors by querying all available element harvesters. This method is used by the Layout Editor to provide the elements you see in the toolbox, as well as by the blueprint element controller</td>    
    </tr>
    <tr>
        <td>GetCategories</td>
        <td>Returns a list of all the element categories. Categories are provided by <strong>ICategoryProvider</strong> implementations. The purpose of these category descriptors are not to declare categories per se, but are a way to configure categories, such as the description and relative position of a category. As en element developer, all you need to do is implement the Category property by returning a string value. If that category is not described by any of the category providers, it assumes a default description and position. If one is described, its description and position information is used when rendering the layout editor's toolbox.</td>
    <tr>
        <td>GetElementDescriptorByTypeName</td>
        <td>Returns a single <strong>ElementDescriptor</strong> with the specified type name.</td>    
    </tr>
    <tr>
        <td>GetElementDescriptorByType</td>
        <td>Returns a single <strong>ElementDescriptor</strong> with the a type name based on the specified generic argument type. One overload accepts an <strong>DescribeElementsContext</strong> argument which you can use if there is a content item available that you want to use as the context when describing (discovering) elements.</td>    
    </tr>
    <tr>
        <td>ActivateElement</td>
        <td>Instantiates an actual element instance based on the given element descriptor. Internally relies on the <strong>IElementFactory</strong> to instantiate elements. The optional <strong>initialize</strong> delegate enables you to configure the created element in one statement (as opposed to first storing the element in a variable, then using that variable to further setup the element). This callback is invoked before the <strong>Created</strong> event is triggered for the element. Good to know.</td>    
    </tr>
    <tr>
        <td>GetDrivers</td>
        <td>Returns a list of all element drivers for the specified element descriptor or type, if specified. If no descriptor or type is specified, all element drivers are returned. It's unlikely that you'll want to invoke drivers manually, but it's there if you need it.</td>    
    </tr>
    <tr>
        <td>BuildEditor</td>
        <td>Invokes the BuildEditor method on all <strong>IElementEventHandler</strong> implementations, one of which being responsible for invoking the same-named method on all element drivers. Element drivers implementing this method will typically create and return a shape that represents the editor for the element. This is used from the element editor dialog.</td>    
    </tr>
    <tr>
        <td>UpdateEditor</td>
        <td>Invokes the UpdateEditor method on all <strong>IElementEventHandler</strong> implementations, one of which being responsible for invoking the same-name method on all element drivers. Element drivers implementing this method will typically perform some modelbinding, update the element's state, and return a shape that represents the editor for that element. This too is used from the element editor dialog.</td>    
    </tr>
    <tr>
        <td>Saving</td>
        <td>Invokes the LayoutSaving method on all <strong>IElementEventHandler</strong> implementations, one of which being responsible for invoking the same-name method on all element drivers. It's unlikely that you'll ever need to invoke this method yourself, unless you're writing your own element / layout editor.</td>    
    </tr>
    <tr>
        <td>Removing</td>
        <td>Invokes the Removing method on all <strong>IElementEventHandler</strong> implementations, one of which being responsible for invoking the same-name method on all element drivers. It's unlikely that you'll ever need to invoke this method yourself, unless you're writing your own element / layout editor.</td>    
    </tr>
    <tr>
        <td>Exporting</td>
        <td>Invokes the Exporting method on all <strong>IElementEventHandler</strong> implementations, one of which being responsible for invoking the same-name method on all element drivers. It's unlikely that you'll ever need to invoke this method yourself, unless you're writing your own element / layout editor.</td>    
    </tr>
    <tr>
        <td>Exported</td>
        <td>Invokes the Exported method on all <strong>IElementEventHandler</strong> implementations, one of which being responsible for invoking the same-name method on all element drivers. It's unlikely that you'll ever need to invoke this method yourself, unless you're writing your own element / layout editor.</td>    
    </tr>
    <tr>
        <td>Importing</td>
        <td>Invokes the Importing method on all <strong>IElementEventHandler</strong> implementations, one of which being responsible for invoking the same-name method on all element drivers. It's unlikely that you'll ever need to invoke this method yourself, unless you're writing your own element / layout editor.</td>    
    </tr>
    <tr>
        <td>Imported</td>
        <td>Invokes the Imported method on all <strong>IElementEventHandler</strong> implementations, one of which being responsible for invoking the same-name method on all element drivers. It's unlikely that you'll ever need to invoke this method yourself, unless you're writing your own element / layout editor.</td>    
    </tr>
    <tr>
        <td>ImportCompleted</td>
        <td>Invokes the ImportCompleted method on all <strong>IElementEventHandler</strong> implementations, one of which being responsible for invoking the same-name method on all element drivers. It's unlikely that you'll ever need to invoke this method yourself, unless you're writing your own element / layout editor.</td>    
    </tr>
</tbody>
</table>

### Element Events ###
Part of writing custom elements consists of implementing element drivers, which allow you to handle events specific to the element type of that driver. Alternatively, we can implement event handlers to handle events for any type of element. To do so, all you need to do is implement the `IElementEventHandler` interface, which looks like this:

#### IElementEventHandler ####
```
public interface IElementEventHandler : IEventHandler {
    void Creating(ElementCreatingContext context);
    void Created(ElementCreatedContext context);
    void CreatingDisplay(ElementCreatingDisplayShapeContext context);
    void Displaying(ElementDisplayingContext context);
    void Displayed(ElementDisplayedContext context);
    void BuildEditor(ElementEditorContext context);
    void UpdateEditor(ElementEditorContext context);
    void LayoutSaving(ElementSavingContext context);
    void Removing(ElementRemovingContext context);
    void Exporting(ExportElementContext context);
    void Exported(ExportElementContext context);
    void Importing(ImportElementContext context);
    void Imported(ImportElementContext context);
    void ImportCompleted(ImportElementContext context);
}
```

The majority of these methods are invoked either by `ElementManager` or `ElementDisplay`, implementations of `IElementManager` and `IElementDisplay`, respectively.

Oftentimes, when implementing an event handler, you're typically only interested in one or some event methods. So instead of implementing this interface directly, it is recommended to implement the abstract class `ElementEventHandlerBase` instead. That way you just need to override the method you're interested in handling.

### Displaying Elements ###
The service that is responsible for creating shapes from elements is `IElementDisplay`, which is defined as:

#### IElementDisplay ####
```
public interface IElementDisplay : IDependency {
    dynamic DisplayElement(Element element, IContent content, string displayType = null, IUpdateModel updater = null);
    dynamic DisplayElements(IEnumerable<Element> elements, IContent content, string displayType = null, IUpdateModel updater = null);
}
```

The `DisplayElement` method returns a shape of type `"Element"`, while the `DisplayElements` method returns a shape of type `"LayoutRoot"`, whose child shapes are all of type `"Element"`.

### The Layout Editor ###
The `ILayoutEditorFactory` service enables us to instantiate a fully initialized `LayoutEditor` object, which is used as a view model. We can use the Html helpers `Editor` and `EditorFor` on `LayoutEditor` objects to the entire layout editor.

You typically use this layout editor factory class from your own controller, and supply the `LayoutEditor` object to your view, from where you render it using `Html.EditorFor`.

An interesting aspect you may realize here is that a layout is *independent from the LayoutPart*. The LayoutPart is a user of the layout editor factory, but the layout editor itself s not dependent on the LayoutPart. The LayoutPart does however take care of reading the elements back from the editor and supplying element data. When you use the layout editor, it is up to you to handle persisting and restoring the element data. We'll see how this works in practice shortly.

### Serialization API ###
The Layouts module comes with two services that handle serialization and deserialization of elements:

- *IElementSerializer*
- *ILayoutSerializer*

The `ILayoutSerializer` can serialize an array of elements into a JSON string, and deserialize a JSON string back into an array of elements.

Internally, it relies on `IElementSerializer` which contains the logic to serialize and deserialize individual elements and JSON nodes. 

The following two code listings reveal the definition of the two services:

```
public interface ILayoutSerializer : IDependency {
    IEnumerable<Element> Deserialize(string data, DescribeElementsContext describeContext);
    string Serialize(IEnumerable<Element> elements);
}
```

```
public interface IElementSerializer : IDependency {
    Element Deserialize(string data, DescribeElementsContext describeContext);
    string Serialize(Element element);
    object ToDto(Element element;
    Element ParseNode(JToken node, Container parent, int index, DescribeElementsContext describeContext);
}
```

The following table describes each member of `ILayoutSerializer`:

<table>
<thead>
    <tr>
        <th>Member</th>
        <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>Deserialize</td>
        <td>Deserializes a JSON string into an hierarchy of elements.</td>
    </tr>
    <tr>
        <td>Serialize</td>
        <td>Serializes an hierarchy of elements into a JSON string.</td>
    </tr>
</tbody>
</table>

And the following table describes the members of `IElementSerializer`:

<table>
<thead>
    <tr>
        <th>Member</th>
        <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>Deserialize</td>
        <td>Deserializes a JSON string into a single element. If the element has child elements, they too will be deserialized recursively.</td>
    </tr>
    <tr>
        <td>Serialize</td>
        <td>Serializes an element into a JSON string. If the element has child elements, they too will be serialized recursively.</td>
    </tr>
    <tr>
        <td>ToDto</td>
        <td>This is used by the Serialize method to turn a given element into an anonymous object, which is then serialized into a JSON string. The reason this method is exposed on the interface is because ILayoutSerializer needs to use the same code. It is unlikely that you'll ever need to use this member yourself.</td>
    </tr>
    <tr>
        <td>ParseNode</td>
        <td>This is the counterpart of ToDto and used by the Deserialize method to parse a given JSON node into an actual element instance. The reason this method is exposed on the interface is because ILayoutSerializer needs to use the same code. It is unlikely that you'll ever need to use this member yourself.</td>
    </tr>
</tbody>
</table>

### Demo: Working with the APIs ###
In this demo, I'll demonstrate how to work with the various APIs covered in this chapter. We'll see how to manually construct a tree of elements, initialize them, serialize and deserialize them and finally render them. We'll then see how to work with the layout editor.


#### Creating the Controller ####
For starters, create the following controller:

```
using System.Web.Mvc;

namespace OffTheGrid.Demos.Layouts.Controllers {
    public class ElementsApiController : Controller {
        public ActionResult Index() {
            return View();
        } 
    }
}
```

Make sure to create an empty Razor View as well.

#### Creating Elements ####
With that in place, let's create an element of type `Html` and initialize it with some HTML content:

```
using System.Web.Mvc;
using Orchard.Layouts.Elements;
using Orchard.Layouts.Services;

namespace OffTheGrid.Demos.Layouts.Controllers {
    [Themed]
    public class ElementsApiController : Controller {
        private readonly IElementManager _elementManager;

        public ElementsApiController(IElementManager elementManager) {
            _elementManager = elementManager;
        }

        public ActionResult Index() {

            // Create a new instance of the Html element using the element manager.
            var html = _elementManager.ActivateElement<Html>();

            // Configure the element.
            html.Content = "<p>This is an <strong>HTML</strong> element.</p>";
            html.HtmlClass = "demo-content";
            html.HtmlId = "Paragraph1";
            html.HtmlStyle = "color: hotpink;";

            return View();
        } 
    }
}
```

The above code listing demonstrates how to work with the `IElementManager` to instantiate an element of a given type.

Instead of setting up the instantiated HTML element as I did above, I could have provided an anonymous function instead, which looks like this:

```
// Create a new instance of the Html element using the element manager.
var html = _elementManager.ActivateElement<Html>(e => {
    // Configure the element.
    e.Content = "<p>This is an <strong>HTML</strong> element.</p>";
    e.HtmlClass = "demo-content";
    e.HtmlId = "Paragraph1";
    e.HtmlStyle = "color: hotpink;";
});
```

It's almost the same, but it's arguably cooler to be able to initialize the element without having to do so after the element has been assigned to its variable (which you wouldn't need anymore, which is useful in scenarios where you instantiate and add elements to container elements).
Another difference is that this code initializes the element before the `Created` event is triggered. Irrelevant in almost all cases, but good to keep in mind for certain advanced scenarios where you handle the Created event and require a fully initialized element for example. 

Although I'm using the strongly typed `Html` type to indicate the type of element I want, what actually happens under the covers is that the element manager simply uses the C# `typeof` keyword to get the .NET type metadata to get the `FullName` property of the type, by which the element descriptors provided by the `TypedElementHarvester` are registered. This means that if you ever wanted to instantiate a dynamically provided element descriptor, such as an element blueprint for example, you'll need to use the overloaded version that takes an `ElementDescriptor` as its argument. To get a descriptor by name, use the `GetElementDescriptorByTypeName` method.

The following code snippet demonstrates how we could instantiate the Html element by its element type name:

```
// Create a new instance of the Html element using the element manager.
var htmlDescriptor = 
    _elementManager
    .GetElementDescriptorByTypeName(DescribeElementsContext.Empty, "Orchard.Layouts.Elements.Html");

// Need to cast to Html, since this overload does not know the .NET type of the element being activated.
var html = (Html)_elementManager.ActivateElement(htmlDescriptor);
```   

### Rendering Elements ###
Now that we have seen how to programmatically instantiate elements, it would be nice if we knew how to render them. Obviously I could simply send the element to my view, and render it from there. For example, the following would totally work:

```
public ActionResult Index() {

    // Create a new instance of the Html element using the element manager.
    var html = _elementManager.ActivateElement<Html>(e => {
        // Configure the element.
        e.Content = "<p>This is an <strong>HTML</strong> element.</p>";
        e.HtmlClass = "demo-content";
        e.HtmlId = "Paragraph1";
        e.HtmlStyle = "color: hotpink;";
    });

    // Assign the Html element to a property on the dynamic ViewBag.
    ViewBag.HtmlElement = html;
    return View();
} 
```

And then in the `Index.cshtml` view:

```
@using Orchard.Layouts.Elements
@{
    // Access the Html element from the ViewBag.
    var htmlElement = (Html)ViewBag.HtmlElement;
}
<div id="@htmlElement.HtmlId" style="@htmlElement.HtmlStyle"  class="@htmlElement.HtmlClass">
    @Html.Raw(htmlElement.Content)
</div>
```

However, this would fail pretty fast when you create trees of elements dynamically, because this view requires hardcoded knowledge about the structure of the element tree, including the types of each element.
Sure, you could create Html helpers and partial views that render individual elements and their children recusrively, and that would work. But you would be reinventing the wheel.

First of all, Orchard solved rendering of hierarchies of things using the shapes system.
Secondly, Orchard.Layouts comes with a service that creates an hierarchy of shapes based in an hierarchy of elements. That service is the `IElementDisplay` service.

Let's see that in action.
Change the controller code as follows:

```
public ActionResult Index() {

    // Create a new instance of the Html element using the element manager.
    var html = _elementManager.ActivateElement<Html>(e => {
        // Configure the element.
        e.Content = "<p>This is an <strong>HTML</strong> element.</p>";
        e.HtmlClass = "demo-content";
        e.HtmlId = "Paragraph1";
        e.HtmlStyle = "color: hotpink;";
    });

    // Render the Html element.
    var htmlShape = _elementDisplay.DisplayElement(html, content: null);

    // Assign the HtmlShape to a property on the dynamic ViewBag.
    ViewBag.HtmlShape = htmlShape;

    return View();
}
```

The above code shows the usage of the `IElementDisplay` service, which is injected via the constructor (not shown above). Invoking the element display service is easy: simply put in the element instance to display and optionally a content item to serve as context to elements that find that useful (`null` in our case), and what you'll get back is an element shape that is ready for rendering, which we'll do from the view:

```
@Display(ViewBag.HtmlShape)
```

![](./figures/fig-94-html-element-front-end.png) 

This also means that, by virtue of leveraging shapes, any alternates get applied to the shape templates.

Before we move on to serialization, let's see how to create a more advanced element hierarchy. For example, let's create a Grid with one Row and 2 Columns. We'll add one Html element to each column.

```
public ActionResult Index() {

    // Create an hierarchy of elements.
    var rootElement = New<Grid>(grid => {
        // Row.
        grid.Elements.Add(New<Row>(row => {
            // Column 1.
            row.Elements.Add(New<Column>(column => {
                column.Width = 8;
                column.Elements.Add(New<Html>(html => html.Content = "This is the <strong>first</strong> column."));
            }));

            // Column 2.
            row.Elements.Add(New<Column>(column => {
                column.Width = 4;
                column.Elements.Add(New<Html>(html => html.Content = "This is the <strong>second</strong> column."));
            }));
        }));
    });

    // Render the Html element.
    var gridShape = _elementDisplay.DisplayElement(rootElement, content: null);

    // Assign the HtmlShape to a property on the dynamic ViewBag.
    ViewBag.RootElementShape = gridShape;

    return View();
}

// A private helper method that acts as an alias to: _elementManager.ActivateElement.
private T New<T>(Action<T> initialize) where T:Element {
    return _elementManager.ActivateElement<T>(initialize);
}
```

And the view:

```
@Display(ViewBag.RootElementShape)
```

When you now navigate to `/OrchardLocal/OffTheGrid.Demos.Layouts/ElementsApi`, you should see the following:

![](./figures/fig-95-grid-front-end.png)

Probably not entirely what you expected. Since we created a grid, a row and two columns, it would be fair to expect the two Html elements to be rendered next to eachother as opposed to be above eachother.

When we inspect the generated HTML, however, everything seems fine:

![](./figures/fig-96-grid-html-output.png)

As it turns out, the *TheThemeMachine* doesn't have any notion of our `"table"`, "`row`", `"span-8"`, `"span-4"` and `"cell"` CSS classes. That stylesheet is provided by the Layouts module, and more specficially, included when the default `"Parts.Layout.cshtml`" view is rendered. Since we are rendering elements ourselves, we will also need to include a stylesheet that implements these CSS rules. So let's do just that:

```
@{
    Style.Include("~/Modules/Orchard.Layouts/Styles/default-grid.css");
}
@Display(ViewBag.RootElementShape)
```

Now the front-end should look like this:

![](./figures/fig-97-grid-html-with-style.png)

Now let's see how we can serialize and deserialize elements.

### Element Serialization ###
The following code listing demonstrates the usage of `ILayoutSerializer` to serialize and deserialize an hierarchy of elements. To make it a little bit more interesting, we'll provide the resulting JSON string to a textarea and allow the user to make changes to it and submit it back. We'll then deserialize the submitted JSON and render the updated set of elements.

The following code snippets shows element serialization in action:

```
// Serialize the root element.
var json = _elementSerializer.Serialize(rootElement);
```

Next, set add the JSON string to the ViewBag of our controller:

```
// Assign the JSON string to a property in the dynamic ViewBag.
ViewBag.LayoutData = json;
```

Then update the view as follows:

```
@{
    Style.Include("~/Modules/Orchard.Layouts/Styles/default-grid.css");
}
<h2>@T("Layout Display")</h2>
@Display(ViewBag.RootElementShape)

<h2>@T("Layout Data")</h2>
@using (Html.BeginFormAntiForgeryPost()) {
    @Html.TextArea("LayoutData", (string) ViewBag.LayoutData, new {rows = 30, cols = 150})
    <button type="submit">@T("Update")</button>
}
```

Now let's update our action method so that it will handle the form submission by deserializing the posted JSON string and rendering the elements.

The completed controller looks like this:

```
using System;
using System.Web.Mvc;
using Orchard.Layouts.Elements;
using Orchard.Layouts.Framework.Display;
using Orchard.Layouts.Framework.Elements;
using Orchard.Layouts.Services;
using Orchard.Themes;

namespace OffTheGrid.Demos.Layouts.Controllers {
    [Themed]
    public class ElementsApiController : Controller {
        private readonly IElementManager _elementManager;
        private readonly IElementDisplay _elementDisplay;
        private readonly IElementSerializer _elementSerializer;

        public ElementsApiController(IElementManager elementManager, IElementDisplay elementDisplay, IElementSerializer elementSerializer) {
            _elementManager = elementManager;
            _elementDisplay = elementDisplay;
            _elementSerializer = elementSerializer;
        }

        public ActionResult Index(string layoutData = null) {

            Element rootElement;

            if (layoutData == null) {
                // Create a default hierarchy of elements.
                rootElement = New<Grid>(grid => {
                    // Row.
                    grid.Elements.Add(New<Row>(row => {
                        // Column 1.
                        row.Elements.Add(New<Column>(column => {
                            column.Width = 8;
                            column.Elements.Add(New<Html>(html => html.Content = "This is the <strong>first</strong> column."));
                        }));

                        // Column 2.
                        row.Elements.Add(New<Column>(column => {
                            column.Width = 4;
                            column.Elements.Add(New<Html>(html => html.Content = "This is the <strong>second</strong> column."));
                        }));
                    }));
                });

                // Serialize the root element.
                var json = _elementSerializer.Serialize(rootElement);

                // Assign the JSON string to a property in the dynamic ViewBag.
                ViewBag.LayoutData = json;
            }
            else {
                rootElement = _elementSerializer.Deserialize(layoutData, DescribeElementsContext.Empty);
            }

            // Render the Html element.
            var rootElementShape = _elementDisplay.DisplayElement(rootElement, content: null);

            // Assign the HtmlShape to a property on the dynamic ViewBag.
            ViewBag.RootElementShape = rootElementShape;
            
            return View();
        }

        private T New<T>(Action<T> initialize) where T:Element {
            return _elementManager.ActivateElement<T>(initialize);
        }
    }
}
```

The key updates are the addition of the optional `layoutData` parameter, which we, if not null, deserialize using the following code:

```
rootElement = _elementSerializer.Deserialize(layoutData, DescribeElementsContext.Empty);
```

So basically, we either construct a layout of elements manually, or deserialize a JSON string, and render the resulting element instance. We can now view the default JSON, manipulate it, and submit it back to the controller.

![](./figures/fig-98-default-json-result.png)

Pasting in the following JSON:

```
{
    "typeName": "Orchard.Layouts.Elements.Grid",
    "elements": [
    {
        "typeName": "Orchard.Layouts.Elements.Row",
        "elements": [
        {
            "typeName": "Orchard.Layouts.Elements.Column", 
            "data": "Width=4", 
            "elements": [
            {
                "typeName": "Orchard.Layouts.Elements.Html", 
                "data": "Content=This+is+the+%3cstrong%3efirst%3c%2fstrong%3e+column.", 
            }], 
        },
        {
            "typeName": "Orchard.Layouts.Elements.Column", 
            "data": "Width=4", 
            "elements": [
            {
                "typeName": "Orchard.Layouts.Elements.Html", 
                "data": "Content=This+is+the+%3cstrong%3esecond%3c%2fstrong%3e+column.", 
                "htmlStyle": "color:red;", 
            }], 
        },
        {
            "typeName": "Orchard.Layouts.Elements.Column", 
            "data": "Width=4", 
            "elements": [
            {
                "typeName": "Orchard.Layouts.Elements.Html", 
                "data": "Content=This+is+the+%3cstrong%3ethird%3c%2fstrong%3e+column."
            }]
        }]
    }]
}
```

Will render the following output:

![](./figures/fig-99-custom-json-result.png)

### Working with the Layout Editor ###
Although editing layouts of elements with raw JSON isn't ideal, it does demonstrate an interesting fact: you can potentially implement any sort of editor to enable users to work with elements.

The Layouts module itself provides a fully-function layout editor that we can reuse, instead of using a simple textarea.

The easiest way to get our hands on a layout editor object is to leverage the `ILayoutEditorFactory` that we mentioned earlier.

We'll continue with our current sample controller and replace the textarea with a full-blown element controller. First order of business is to inject the `ILayoutEditorFactory` and create the `LayoutEditor` viewmodel, which we'll render from our view. We'll also change out the `IElementSerializer` with the `ILayoutSerializer`. The primary difference between the two is that the latter one expects an array of elements instead of a single element, which in turn the default `ILayoutEditorFactory` implementation expects.

> The layout editor requires the root element to by a *Canvas*, so the default implementation of *ILayoutEditorFactory* relies on *ILayoutEditorFactory*, which always serializes from and to an array of elements, the first element expected to be a Canvas. If the first element is *not* a canvas, a root Canvas element is created on the fly to which the array of elements is added.

We'll also use the `Canvas` element as the root, which is a requirement for the layout editor.

Another thing we'll change is the usage of the `[Themed]` attribute decorating our controller class. Since the layout editor is designed to work from the backend only, we need to replace the `[Themed]` attribute with the `[Admin]` attribute.

> To have your controller apply `TheAdmin` theme, either apply the `AdminAttribute` or rename your controller to `AdminController`. If you change your controller's name, be sure to also update the corresponding view folder.

The following code snippet shows the updated controller.

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web.Mvc;
using Orchard.Layouts.Elements;
using Orchard.Layouts.Framework.Display;
using Orchard.Layouts.Framework.Elements;
using Orchard.Layouts.Services;
using Orchard.Layouts.ViewModels;
using Orchard.UI.Admin;

namespace OffTheGrid.Demos.Layouts.Controllers {
    [Admin]
    public class ElementsApiController : Controller {
        private readonly IElementManager _elementManager;
        private readonly IElementDisplay _elementDisplay;
        private readonly ILayoutSerializer _layoutSerializer;
        private readonly ILayoutEditorFactory _layoutEditorFactory;
        private readonly ILayoutModelMapper _modelMapper;

        public ElementsApiController(
            IElementManager elementManager,
            IElementDisplay elementDisplay,
            ILayoutSerializer layoutSerializer,
            ILayoutEditorFactory layoutEditorFactory,
            ILayoutModelMapper modelMapper) {

            _elementManager = elementManager;
            _elementDisplay = elementDisplay;
            _layoutSerializer = layoutSerializer;
            _layoutEditorFactory = layoutEditorFactory;
            _modelMapper = modelMapper;
        }

        [ValidateInput(false)]
        public ActionResult Index(LayoutEditor layoutEditor) {

            IEnumerable<Element> layout;
            string layoutData = null;
            
            if(layoutEditor.Data != null) {
                // The posted layout data is not the raw Layouts JSON format, but a more tailored one specific to the layout editor.
                // Before we can use it, we need to map it to the raw layout format.
                layout = _modelMapper.ToLayoutModel(layoutEditor.Data, DescribeElementsContext.Empty).ToList();

                // Serialize the layout.
                layoutData = _layoutSerializer.Serialize(layout);
            }
            else {
                // Create a default hierarchy of elements.
                layout = CreateDefaultLayout();

                // Serialize the layout.
                layoutData = _layoutSerializer.Serialize(layout);
            }

            // Create and initialize a new LayoutEditor object.
            var sessionKey = "DemoSessionKey";
            layoutEditor = _layoutEditorFactory.Create(layoutData, sessionKey);

            // Assign the LayoutEditor to a property on the dynamic ViewBag.
            ViewBag.LayoutEditor = layoutEditor;

            // Render the elements and assign the resulting shape to a property on the dynamic ViewBag.
            ViewBag.LayoutShape = _elementDisplay.DisplayElements(layout, content: null); ;

            return View();
        }

        private IEnumerable<Element> CreateDefaultLayout() {
            return new[] { New<Canvas>(canvas => {
                canvas.Elements.Add(
                    New<Grid>(grid => {
                    // Row.
                    grid.Elements.Add(New<Row>(row => {
                        // Column 1.
                        row.Elements.Add(New<Column>(column => {
                            column.Width = 8;
                            column.Elements.Add(New<Html>(html => html.Content = "This is the <strong>first</strong> column."));
                        }));

                        // Column 2.
                        row.Elements.Add(New<Column>(column => {
                            column.Width = 4;
                            column.Elements.Add(New<Html>(html => html.Content = "This is the <strong>second</strong> column."));
                        }));
                    }));
                }));
            })};
        }

        private T New<T>(Action<T> initialize) where T : Element {
            return _elementManager.ActivateElement<T>(initialize);
        }
    }
}
```

Notice that I moved the creation of the initial elements to a private method.
  