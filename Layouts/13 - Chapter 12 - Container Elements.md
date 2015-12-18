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

The only member that the `Container` class adds is the `Elements` property, which is a list of elements:

```
public abstract class Container : Element {
    public IList<Element> Elements { get; set; }
}
```

This `Elements` list is initialized to an empty list from the constructor, so you don;t have to worry about instantiating that yourself.

### Element Drivers ###
When writing custom element classes, you need to implement a driver for that element, even if it contains no actual implementation. This is the same when writing container elements.  

### Layout Editor Integration ###
The most work when creating a custom container goes into making it work with the layout editor. The primary reason for this is the fact that the layout editor has a client side model of elements, and it needs to know what the type of object is to be used on the client.

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
If you thought model mappers were kind of icky, wait until you see what needs to be done for the client side representation of custom container elements. That process feels arcane to say the least. But fortunately, you were smart enough to read this book, and everything will turn out all right.

To implement the client side story of container elements, we essentially need four things:

- Implement a client side model of the element
- Implement a directive (since the layout editor is implemented with AngularJS)
- Implement a template for the directive
- Register the element with the element factory.




### Custom Container Walkthrough ###
The best way to understand how everything fits together is by 

### Summary ###
In this chapter, we have seen how we can extend the list of available elements by implementing our own element classes. We learned about the `Element` class itself as well as the `ElementDriver<T>` class. We then continued and created a custom element called the `Clock`, demonstrating what it takes to provide custom rendering as well as an element editor. We also learned about implementing element editors using the *Forms API* provided by the `Orchard.Forms` module, which is great for basic element editors where there is no need for more advanced editor UIs.

With this knowledge in our pocket, we have all we need to be able to write any kind of elements, whether they be simple or more complex. It always boils down to implementing an element class and driver, and implementing the various methods on the driver.

However, what if we wanted to provide elements based not on static element classes, but on something else entirely? For example, what if you have a database table of records that you would like to present as elements? Well, that is in fact the next chapter's topic, where we'll learn everything there is to know about *Element Harvesters*.   