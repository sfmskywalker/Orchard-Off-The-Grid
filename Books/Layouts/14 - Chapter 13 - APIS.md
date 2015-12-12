## APIs ##
So far we've seen how to write custom elements by creating custom element classes and harvesters. In this chapter, we will learn about APIs at our disposal to programmatically work with elements. More specifically, we will learn how to:

- Work with the element manager to query all available element categories and descriptors;
- Use the element factory to instantiate elements;
- Render elements and layouts of elements with the layout manager and element manager;
- Serialize and deserialize elements;
- Handle element events by implementing the `IElementEventHandler` interface;
- Reuse the layout editor from our own module.

Knowing about these APIs will enable you to take advantage of all that the Layouts module has to offer and implement all sorts of applications yourself. For example, in the next chapter we will write a *SlideShowPart* that uses the APIs from this chapter to enable the user to create slides that are based on layouts.

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

And the following table describes each member (except for the method overloads):

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
</tbody>
</table>