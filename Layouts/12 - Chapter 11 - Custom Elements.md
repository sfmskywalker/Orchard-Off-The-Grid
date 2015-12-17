## Custom Elements ##
Elements are at the core of the Layouts module, and in this chapter, we will see how we can write our own.

Writing custom elements essentially boils down to two things:

1. Implement a class that drives from `Element`.
2. Implement a driver for your element class. Your driver needs to be derived from `ElementDriver<T>`, where the generic `T` type argument is to be substituted with your custom element type.

### The Element Class ###
Elements are instances of .NET classes that inherit from the `Element` abstract base class which lives in the `Orchard.Layouts.Framework.Elements` namespace. Sub classes typically add properties to their type definition and override inherited properties such as `Category`.

The following table is a complete list of all the properties of the `Element` class:

<table>
<thead>
    <tr>
        <th>Property</th>
        <th>Type</th>
        <th>Modifier</th>
        <th>Default</th>
        <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td><strong>Category</strong></td>
        <td>String</td>
        <td>abstract</td>
        <td></td>
        <td>The category of the element. Elements are grouped by their category.</td>
    </tr>
    <tr>
        <td><strong>Type</strong></td>
        <td>String</td>
        <td>virtual</td>
        <td>The .NET type name (GetType().Name)</td>
        <td>The type name of the element. This value is used to deserialize elements.</td>
    </tr>
    <tr>
        <td><strong>DisplayText</strong></td>
        <td>LocalizedString</td>
        <td>virtual</td>
        <td>The .NET type name, made camel friendly (T(GetType().Name.CamelFriendly()))</td>
        <td>The "friendly" name of the element that is used in the UI.</td>
    </tr>
    <tr>
        <td><strong>Description</strong></td>
        <td>LocalizedString</td>
        <td>virtual</td>
        <td>A string based on the DisplayText (T("{0} element.", DisplayText))</td>
        <td>The description of the element that is used in the UI.</td>
    </tr>
    <tr>
        <td><strong>ToolboxIcon</strong></td>
        <td>String</td>
        <td>virtual</td>
        <td>\uf1c9</td>
        <td>The Font Awesome unicode character to display as the element's icon in the toolbox.</td>
    </tr>
    <tr>
        <td><strong>IsSystemElement</strong></td>
        <td>Boolean</td>
        <td>virtual</td>
        <td>false</td>
        <td>This is used by the <em>TypedElementHarvester</em>, which yields element descriptors based on types derived from <em>Element</em>. If the element is a &quot;system&quot; element, it is not harvested. This is typically used in conjunction with other element harvesters that provide element descriptors based on other sources.</td>
    </tr>
    <tr>
        <td><strong>HasEditor</strong></td>
        <td>Boolean</td>
        <td>virtual</td>
        <td>true</td>
        <td>This used by the layout editor to determine whether or not to show the element editor dialog when an element is being dragged & dropped from the toolbox onto the canvas and whether or not to display the <em>edit</em> glyph. If your custom element does not provide an editor, you should override this property and return false.</td>
    </tr>
    <tr>
        <td><strong>HtmlId</strong></td>
        <td>String</td>
        <td></td>
        <td></td>
        <td>Stores the configured HTML ID value that the user can set via the layout editor. Default templates typically render this value on the <strong>id</strong> attribute of the root tag. For example: &lt;div id=&quot;@Model.Element.HtmlId&quot;&gt;</td>
    </tr>
    <tr>
        <td><strong>HtmlClass</strong></td>
        <td>String</td>
        <td></td>
        <td></td>
        <td>Stores the configured CSS class value that the user can set via the layout editor. Default templates typically render this value on the <strong>class</strong> attribute of the root tag. For example: &lt;div class=&quot;@Model.Element.HtmlClass&quot;&gt;</td>
    </tr>
    <tr>
        <td><strong>HtmlStyle</strong></td>
        <td>String</td>
        <td></td>
        <td></td>
        <td>Stores the configured CSS inline-style value that the user can set via the layout editor. Default templates typically render this value on the <strong>style</strong> attribute of the root tag. For example: &lt;div style=&quot;@Model.Element.HtmlStyle&quot;&gt;</td>
    </tr>
    <tr>
        <td><strong>Rule</strong></td>
        <td>String</td>
        <td></td>
        <td></td>
        <td>Stores the display rule for the element. The rule engine used is the same as the one being used by the Widgets module. The user can provide a rule expression via the layout editor.</td>
    </tr>
    <tr>
        <td><strong>Container</strong></td>
        <td>Orchard.Layouts.Elements.Container</td>
        <td></td>
        <td></td>
        <td>A reference to the containing element, if any.</td>
    </tr>
    <tr>
        <td><strong>Data</strong></td>
        <td>Orchard.Layouts.Framework.Elements.ElementDataDictionary</td>
        <td></td>
        <td>An instance of the <em>ElementDataDictionary</em> type so that derived types can access this dictionary right off the bat.</td>
        <td>The <em>ElementDataDictionary</em>, which inherits from <em>Dictionary&lt;string, string&gt;&lt;/em&gt;, provides storage to child classes of the Element class. Child classes typically provide additional public properties whose getters and setters read from and write to the Data dictionary.</td>
    </tr>
    <tr>
        <td><strong>ExportableData</strong></td>
        <td>Orchard.Layouts.Framework.Elements.ElementDataDictionary</td>
        <td></td>
        <td>An instance of the <em>ElementDataDictionary</em> type.</td>
        <td>Provides an additional storage bag for when the element is being exported. Most elements won't typically have to use this bag, as the Data property is automatically exported. The ExportableData property is typically used for elements that reference content items by ID, so they need to export the content items' Identity instead of ID, which is a primary key database value.</td>
    </tr>
    <tr>
        <td><strong>IsTemplated</strong></td>
        <td>Boolean</td>
        <td></td>
        <td>false</td>
        <td>This is more of a framework-property and is used by the master layout system to mark elements as being <em>sealed</em> when they're part of a master layout. The layout editor prevents users from changing sealed elements.</td>
    </tr>
    <tr>
        <td><strong>Descriptor</strong></td>
        <td>Orchard.Layouts.Framework.Elements.ElementDescriptor</td>
        <td></td>
        <td></td>
        <td>The Descriptor contains metadata information about the element as well as delegates to element event handlers that need to be invoked when certain events occur for a given element. Element Descriptors are created by Element Harvesters, and it is in fact these descriptors that are presented as elements in the layout editor's toolbox.</td>
    </tr>
    <tr>
        <td><strong>T</strong></td>
        <td>Orchard.Localization.Localizer</td>
        <td></td>
        <td>An instance of the Localizer delegate.</td>
        <td>The T property is the standard Orchard delegate that translates strings. A value is automatically provided for you as soon as an element is instantiated, so you can use it when overriding the localized properties such as DisplayText and Description.</td>
    </tr>
</tbody>
</table>

We won't go into *element descriptors* just yet, since we don't have to deal with them directly when creating custom elements. We will, however, look into them in detail when we look into *element harvesters*.

### Element Drivers ###
While element classes handle the data side of things, it's the *element drivers* that handle the behavioral side of things. This works very similar to the way content parts, content part drivers, content fields and content field drivers work. Drivers handle things like displaying and editing the element.

If you're familiar with writing custom parts and fields, you'll recognize many of the concepts when writing custom elements.              

> Although I'm using the term "custom element" a lot, elements that you write are no different from the elements provided by the Layouts module. I'm using the term "custom" to differentiate between out-of-box- elements and elements that you write as part of your custom modules.

As is the case with content part and content field drivers, you can write *more than one element driver* for a given element type. As we'll see later in this book, this is key to extending existing elements with additional settings and behavior.

The following code listing shows the protected and virtual members potentially of interest to you when implementing your own drivers:

```
abstract class ElementDriver<TElement> : Component, IElementDriver where TElement: Element {
    public virtual int Priority { get; }
    protected virtual EditorResult OnBuildEditor(TElement element, ElementEditorContext context);
    protected virtual EditorResult OnUpdateEditor(TElement element, ElementEditorContext context);
    protected virtual void OnCreatingDisplay(TElement element, ElementCreatingDisplayShapeContext context);
    protected virtual void OnDisplaying(TElement element, ElementDisplayingContext context);
    protected virtual void OnDisplayed(TElement element, ElementDisplayedContext context);
    protected virtual void OnLayoutSaving(TElement element, ElementSavingContext context);
    protected virtual void OnRemoving(TElement element, ElementRemovingContext context);
    protected virtual void OnExporting(TElement element, ExportElementContext context);
    protected virtual void OnExported(TElement element, ExportElementContext context);
    protected virtual void OnImporting(TElement element, ImportElementContext context);
    protected virtual void OnImported(TElement element, ImportElementContext context);
    protected virtual void OnImportCompleted(TElement element, ImportElementContext context);
    protected EditorResult Editor(ElementEditorContext context, params dynamic[] editorShapes)
}
```

The following table describes each of the above members:

<table>
<thead>
    <tr>
        <th>Member</th>
        <th>Description</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td><strong>Priority</strong></td>
        <td>A way for drivers to influence the order in which they are executed. This is useful in rare cases where you implement additional drivers for the same element type.</td>
    </tr>
    <tr>
        <td><strong>OnBuildEditor</strong></td>
        <td>Override this method when your element has an editor UI</td>
    </tr>
    <tr>
        <td><strong>OnUpdateEditor</strong></td>
        <td>Override this method to handle post-backs of your editor UI created by <em>OnBuildEditor</em>.</td>
    </tr>
    <tr>
        <td><strong>OnCreatingDisplay</strong></td>
        <td>Override this method to cancel the display of your element based on certain conditions. An example using this method is the NotificationsElementDriver, which prevents its element from being rendered when there are no notifications.</td>
    </tr>
    <tr>
        <td><strong>OnDisplaying</strong></td>
        <td>Override this method to provide additional information to the ElementShape that has been created for your element. Typical use cases are drivers that query or calculate additional information and add that to the ElementShape, which is provided by the context argument.</td>
    </tr>
    <tr>
        <td><strong>OnDisplayed</strong></td>
        <td>Override this method to provide additional information to the ElementShape after the Displaying event has been invoked. A good example of a use case for this is the RowElementDriver, which needs to determine whether or not it should be <em>collapsed</em> based on its child Column elements.</td>
    </tr>
    <tr>
        <td><strong>OnLayoutSaving</strong></td>
        <td>Override this method to perform some additional actions just before the element is being serialized. Nothing out of box is currently using this, so I have no good example of when you might want to use this. But, it's there if you need it.</td>
    </tr>
    <tr>
        <td><strong>OnRemoving</strong></td>
        <td>Override this method if you need to perform cleanup when your element is being removed. A good example are elements that create content items and are in charge of managing their lifetime. If the element is removed, you probably also want to remove the content item. The Removing event is invoked whenever an element was removed from the layout and the LayoutPart is being saved, but also when a content item with the LayoutPart is being removed.</td>
    </tr>
    <tr>
        <td><strong>OnExporting</strong></td>
        <td>Override this method if you need to add additional data when your element is being exported. A common example is the case where your element references a content item. In such cases, you'll need to export the identity of the content item, since you cannot rely on the content item ID itself remaining the same when the content item is being imported again.</td>
    </tr>
    <tr>
        <td><strong>OnExported</strong></td>
        <td>At the moment of this writing, the Exported event is triggered right after the Exporting event is triggered, so in practice there is no difference between the two. However, this may change in the future. For now, I recommend just sticking with the Exporting event when providing additional information.</td>
    </tr>
    <tr>
        <td><strong>OnImporting</strong></td>
        <td>Override this method when you have additinal data to process when your element is being imported. For example, if you exported a content item's identity value, here is where you read bck that information and get your hands on the content item in question, get its ID value and update your reference to that content item. A perfect example is the ContentItemElementDriver, which exports and imports a list of referenced content items.</td>
    </tr>
    <tr>
        <td><strong>OnImported</strong></td>
        <td>At the moment of this writing, the Imported event is triggered right after the Importing event is triggered, so in practice there is no difference between the two. However, this may change in the future. For now, I recommend just sticking with the Importing event when providing additional information.</td>
    </tr>
    <tr>
        <td><strong>OnImportCompleted</strong></td>
        <td>This event is triggered after the Importing/Imported events have been invoked on <em>all content items and elements</em>. Override this method when you need to update your element with additional referenced data, which may be avalable only after all other content items have been imported. The prime example here is the ProjectionElementDriver, which relies on the Query content items to be fully imported, since Query content items need to have their Layout records imported first before they are available to projections referencing those layout records.</td>
    </tr>
    <tr>
        <td><strong>Editor</strong></td>
        <td>This is a method you don't override, but use from your OnBuildEditor/OnUpdateEditor methods to create an EditorResult. An EditorResult provides a list of Editor Shapes to be rendered. Although you could construct an EditorResult yourself, the advantage of using the Editor method is that it takes care of setting the Metadata.Position property on each editor shape, which is required for your editor shapes to become visible in the element editor dialog.</td>
    </tr>
</tbody>
</table>

### Element Data Storage ###
Element classes typically implement additional properties that are specific to them. In almost all cases, you'll want the information stored in those properties to be persisted when the element instance itself is being persisted. To persist information, all you need to do is store that information into the `Data` dictionary that each element inherits. The *element serializer* will serialize all contents of the Data dictionary.

The following example demonstrates how you can use the `Data` as the *backing mechanism* for your properties:

```
public string MyProperty {
    get { return Data.ContainsKey("MyProperty") ? Data["MyProperty"] : null; }
    set { Data["MyProperty"] = value; }
}
```

The `Data` dictionary only stores string values, so if your property uses any other types, you;ll have to handle the parsing. Fortunately there is a nice helper class called `XmlHelper` in the `Orchard.ContentManagement` namespace that can help with that. For example, let's implement an `Int32` property:

```
public int MyProperty {
    get { return Data.ContainsKey("MyProperty") ? XmlConvert.Parse<int>(Data["MyProperty"]) : 0; }
    set { Data["MyProperty"] = value.ToString(); }
}
```

Now that we understand the basics of element data storage, let's see if we can simplify the property implementation a bit. For example, I would like to not have to check for the existence of a dictionary key in each and every property getter. As it turns out, we have a nice little extension method for that called `Get` in the `Orchard.Layouts.Helpers` namespace, which we can use as follows:

```
public int MyProperty {
    get { return XmlConvert.Parse<int>(Data.Get("MyProperty")); }
    set { Data["MyProperty"] = value.ToString(); }
}
```
Since `XmlHelper.Parse<T>` returns a `default(T)` in case we pass in a null string, we don't have to worry about null checking ourselves.

But we can do even better. Instead of working with magic string values as dictionary keys, we can implement our properties using strongly-typed expressions using two following extensions methods: `Retrieve` and `Store`, which also live in the `Orchard.Layouts.Helpers` namespace. This is how to use them:

```
public int MyProperty {
    get { return this.Retrieve(x => x.MyProperty); }
    set { this.Store(x => x.MyProperty, value); }
}
```
Much better! And the extension methods take care of the parsing too.

So far we stored primitive types. What if we wanted to store complex objects? Unfortunately, the `Store` and `Retrieve` methods don't support that (which in turn rely on `XmlConvert` to do the string parsing).
However, since the `Data` property is of type `ElementDataDictionary`, we can take advantage of the `GetModel` method, which takes advantage of the `DefaultModelBinder`.

For example, let's say we have the following complex type:

```
public class MyElementSettings {
    public int MyNumber { get; set; }
    public string MyAddress { get; set; }
}
```

Then implementing an element property of that type would look like this:

```
public MyElementSettings MyProperty {
    get { return Data.GetModel<MyElementSettings>(""); }
    set {
        Data["MyNumber"] = value?.MyNumber;
        Data["MyAddress"] = value?.MyAddress; 
    }
}
``` 

It's kind of sad that there's no convenient method for the property setter at this moment of writing, but who knows when that will change.

### Developing Custom Elements ###
When developing custom elements, the following are typically the steps involved.

1. Create a class that derives from `Element` or any of its child classes. The only required member that needs to be implemented is the abstract `Category` property. 
2. Create a class that derives from `ElementDriver<T>`, where `T` is your element's class. This class does not require any members, but needs to be there in order for your element type to be discoverable due to the way the `TypedElementHarvester` works. We'll look into element harvesters later on.
3. Although not strictly required, create a Razor view for your element for the `"Detail"` display type. If you don't provide a view, a default view will be used which simply displays the name of the element.
4. Also not strictly required, create a design-time view for your element for the `"Design"` display type. If you don't provide a design-time view, the view for the `"Detail"` display type will be used. This view is used when your element is rendered by the layout editor.

Learning how to write custom elements is best done by following an example, so let's try it out.

#### Trying it out: Creating the Clock element ####
In this demo we'll create a very simple custom element called `Clock` that will display the current time. The goal of this demo is to introduce the basics of custom element development.
We'll allow the user to provide a *custom date format string*, so we'll need to implement an element editor UI as well. 

> For the demos in this third part, I created a custom module called `OffTheGrid.Demos.Layouts` that you can download from the book's website.

##### Writing the Clock element and driver #####
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

##### Creating the element shape template #####
The next step is to have the clock actually display the current time. To do so, we need to provide a view for our element. When an element is being rendered, a shape called `Element` is created, to which a number of alternates is added. One of these alternates is based on the type name of the element. The alternate we're interested in is as follows: `Elements.Clock`, so add a view called `Elements/Clock.cshtml` that looks like this:

```
<p>@DateTime.Now.ToString("t")</p>
```

And that's all it takes to write a basic element.

##### Writing the Clock element editor #####
Now, let's say that we wanted to enable the user to configure the date/time format string for the Clock element. To achieve that, we need to do three things:

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

The `OnBuildEditor` returns an `EditorResult` (returned by the `Editor` methods inherited from the base class) that takes the `context` argument as well as a shape object. we're passing in an `EditorTemplate` shape, which itself is configured with a `TemplateName` and `Model` property.

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

### An Alternative Element Editor Implementation using the Forms API ###
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

### Summary ###
In this chapter, we have seen how we can extend the list of available elements by implementing our own element classes. We learned about the `Element` class itself as well as the `ElementDriver<T>` class. We then continued and created a custom element called the `Clock`, demonstrating what it takes to provide custom rendering as well as an element editor. We also learned about implementing element editors using the *Forms API* provided by the `Orchard.Forms` module, which is great for basic element editors where there is no need for more advanced editor UIs.

With this knowledge in our pocket, we have all we need to be able to write any kind of elements, whether they be simple or more complex. It always boils down to implementing an element class and driver, and implementing the various methods on the driver.

However, what if we wanted to provide elements based not on static element classes, but on something else entirely? For example, what if you have a database table of records that you would like to present as elements? Well, that is in fact the next chapter's topic, where we'll learn everything there is to know about *Element Harvesters*.   