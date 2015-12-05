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

#### Trying it out: Writing a Custom Element ####
In this demo we'll create a very simple custom element just to get a feel for it. The element won't do anything useful yet, but will serve as the basis for a more advanced element.

First of all, make sure you understand how to create custom modules. For the demos in this part, I created a custom module called `OffTheGrid.Demos.Layouts`.  

### Anatomy of an Element ###
An element, at its core, is an object instance of a class that ultimately derives from the `Element` base class, which lives in the `Orchard.Layouts.Framework.Elements` namespace.

The public signature of that class looks like this:

```
public abstract class Element : IElement {
    protected Element() {}

    public Localizer T { get; set; }
    public Container Container { get; set; }
    public virtual bool IsSystemElement => false;
    public virtual bool HasEditor => true;
    public virtual string Type => GetType().FullName;
    public virtual LocalizedString DisplayText => T(GetType().Name.CamelFriendly());
    public virtual LocalizedString Description => T("{0} element.", DisplayText);
    public virtual string ToolboxIcon => "\uf1c9";
    public abstract string Category { get; }
    public string HtmlId { get; set; }
    public string HtmlClass { get; set; }
    public string HtmlStyle { get; set; }
    public string Rule { get; set; }
    public ElementDataDictionary ExportableData { get; set; }
    public ElementDescriptor Descriptor { get; set; }
    public ElementDataDictionary Data { get; set; }
    public bool IsTemplated { get; set; }
    public int Index { get; set; }
}
```

That's a lot of properties. Some of these properties should be overridden by child classes while others are used to store information for an element instance. Let's go over them one by one.

<table>
    <thead>
        <tr>
            <th>Property</th>
            <th>Description</th>    
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>T</td>
            <td>The localizer delegate to localize strings. This is injected automatically by the IoC container. The Element and its child classes can use this property to return localized strings.</td>
        </tr>
        <tr>
            <td>Container</td>
            <td>A reference to the element that contains this one, if any. The `Container` element derives directly from `Element` and adds a single property called `Elements`. We'll see Container elements in action later on.</td>
        </tr>
        <tr>
            <td>IsSystemElement</td>
            <td>Used by the Layout Editor to determine wether or not this element should be displayed in the Toolbox. If set to `true`, this element will not be available from the toolbox. We'll see why this is useful later on.</td>
        </tr>
        <tr>
            <td>HasEditor</td>
            <td>Used by the Layout Editor to determine wether or not to display the Element Editor Dialog when creating an instance of this element. Most elements have editable properties, so this property returns `true` by default, but can be overridden, as some element types don't have configurable properties.</td>
        </tr>
        <tr>
            <td>Type</td>
            <td>The technical type name of this element. This name is used as a type identifier when element instances are serialized. This information is necessary when deserializing so that the element factory knows what element type to instantiate. Note: although the .NET type name is used by default, that's not a requirement. Any value is valid to use as a type, just as long as its globally unique.</td>
        </tr>
        <tr>
            <td>DisplayText</td>
            <td>The friendly name of the element. This is used in various places, such as in the Layout Editor's Toolbox as well as the Element Editor dialog's title bar.</td>
        </tr>
        <tr>
            <td>Description</td>
            <td>The description of the element. This is used in the Layout Editor's Toolbox.</td>
        </tr>
    </tbody>
</table>
 

#### Element Descriptor ####
An Element Descriptor contains everything there is to know about an element definition and acts as the blueprint for element instances. The following code snippet shows the public signature of the `ElementDescriptor` class.

```
public class ElementDescriptor {
    // Constructor.
    public ElementDescriptor(Type elementType, string typeName, LocalizedString displayText, LocalizedString description, string category);

    // Properties.
    public LocalizedString DisplayText { get; set; }
    public LocalizedString Description { get; set; }
    public string ToolboxIcon { get; set; }
    public string Category { get; set; }
    public Type ElementType { get; set; }
    public string TypeName { get; set; }
    public Func<IEnumerable<IElementDriver>> GetDrivers { get; set; }
    public Action<ElementCreatingDisplayShapeContext> CreatingDisplay { get; set; }
    public Action<ElementDisplayingContext> Displaying { get; set; }
    public Action<ElementDisplayedContext> Displayed { get; set; }
    public Action<ElementEditorContext> Editor { get; set; }
    public Action<ElementEditorContext> UpdateEditor { get; set; }
    public Action<ElementSavingContext> LayoutSaving { get; set; }
    public Action<ElementRemovingContext> Removing { get; set; }
    public Action<ExportElementContext> Exporting { get; set; }
    public Action<ExportElementContext> Exported { get; set; }
    public Action<ImportElementContext> Importing { get; set; }
    public Action<ImportElementContext> Imported { get; set; }
    public Action<ImportElementContext> ImportCompleted { get; set; }
    public bool IsSystemElement { get; set; }
    public bool EnableEditorDialog { get; set; }
    public IDictionary<string, object> StateBag { get; set; }
}
```                 