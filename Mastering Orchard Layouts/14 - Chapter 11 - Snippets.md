## 11. Snippets
In chapter 3 we learned about *Snippet Elements*. They are elements dynamically discovered based on Razor views whose name follow a certain convention: Razor views that end with **Snippet.cshtml** are harvested as Snippet elements. For example, **ButtonSnippet.cshtml** would yield an element called **Button Snippet**.

### Parameterized Snippets
As of Orchard 1.10, this feature was enhanced to provide **parameterized snippets**. A parameterized snippet enables the user to add snippet elements to the layout editor and *configure these snippets with custom settings*.

To add settings to a snippet, all a developer needs to do is invoke an HTML helper called `SnippetField`. This nifty helper serves two purposes:

1. To provide meta information to the element editor about the configurable field.
2. To read the configured value for the field when the element is being rendered.

The signature of the helper looks like this:  

```
SnippetFieldDescriptorBuilder SnippetField(this HtmlHelper htmlHelper, string name, string type = null) {}
```

The following code demonstrates various usages usage of this HTML helper:

```
@* Render a field called "MyfieldName". *@
@Html.SnippetField("MyFieldName")

@* Render a field called "MyfieldName" with a friendly name and description. *@
@Html
	.SnippetField("MyFieldName")
	.DisplayedAs(T("My Field Name"))
	.WithDescription(T("The description of my field."))
```

The **name** argument represents the *technical name of the field*, while the **type** argument represents the *data type of the field*. At the time of this writing, only one type is supported: *text*, but it is easy to extend as we'll see in this chapter.

The method returns an object of type `SnippetFieldDescriptorBuilder` which enables the theme developer to provide more information to the snippet field using a fluent API. Currently there is just **DisplayName** and **Description**, both of which are used when presenting the user with the element editor dialog in the back-end.

Parameterized snippets are a great tool for theme developers to create custom elements without the need to write element classes. All they need to do is create a Razor view and provide the necessary markup.

To get a good understanding of what you can do with parameterized snippets, let's see how to create one.

#### Trying it out: Parameterized Snippets ####
In this example, we'll build a reusable, parameterized snippet called **JumbotronSnippet** with two fields: **Caption** and **Text**. The idea is that users can add any number of jumbotrons to their pages and provide a caption and a body text.

The first thing to do is create a view called **JumbotronSnippet.cshtml** in the current theme or any custom module, and provide the following markup:

*Views/JumbotronSnippet.cshtml*
```
@using Orchard.Layouts.Helpers
@*Using inline CSS for demo purposes.*@
@using (Script.Head()) {
    <style type="text/css">
        .jumbotron {
            padding: 2em;
            background: #ededed;
        }
    </style>
}
<div class="jumbotron">
    <h2>@Html.SnippetField("Caption").DisplayedAs(T("Caption")).WithDescription(T("The caption of this jumbotron."))</h2>
    @Html.Raw(Html.SnippetField("Text").DisplayedAs(T("Text")).WithDescription(T("The body text of this jumbotron")))
</div>
```

Notice that I'm chaining the `DisplayedAs` and `WithDescription` methods. The fluent API enables you to do everything using a single statement, rather than having to first instantiate a snippet field, initializing it with desired values, and then render it. 

With this snippet in place, all we need to do next is make sure that the **Layout Snippets** feature is enabled and add the **Jumbotron** element to our canvas.

![](./figures/fig-11-1-adding-jumbotron.png)

Notice that as soon as you add a **Jumbotron** element to the page, the element editor dialog appears, showing the two fields as defined in the **JumbotronSnippet.cshtml** file:

![](./figures/fig-11-2-jumbotron-editor.png)

As you can see, the editor took the information from the snippet field descriptors as created in the view and used it to render appropriate input controls.

Let's see how this looks on the front end:

![](./figures/fig-11-3-jumbotron-front-end.png)

### Custom Field Editors
Besides the default set of snippet field editors, you can provide your own snippet field editor types. All you need to do is create a Razor view in the **Views/EditorTemplates** folder of your theme or module, and give it a name as follows: **Elements.Snippet.Field.[YourEditorTypeName].cshtml**. For example, if you wanted to provide an editor called **Multiline**, you would create a Razor file called **Elements.Snippet.Field.Multiline.cshtml**. Then, when rendering snippet fields, you would specify **Multiline** as the snippet field type, as is demonstrated in the following sample snippet:

```C#
@Html.SnippetField("Paragraph", "Multiline")
```

The following code listing shows an example implementation of the field editor snippet:

*Views/EditorTemplates/Elements.Snippet.Field.Multiline.cshtml*
```C#
@model Orchard.Layouts.ViewModels.SnippetFieldViewModel
@{
    var field = Model;
}
<div class="form-group">
    @Html.Label(field.Descriptor.Name, field.Descriptor.DisplayName.ToString())
    @Html.TextArea(field.Descriptor.Name, field.Value, 10, 50, new { @class = "text large" })
    @if (field.Descriptor.Description != null) {
        @Html.Hint(field.Descriptor.Description)
    }
</div>
```

The only requirement of a custom snippet field type Razor view is to render some input field that has the *same name as the value provided by* `SnippetFieldViewModel.Descriptor.Name`, which wires up the user's input with the backing store of the snippet field. 

### Summary
In this chapter, we learned how snippets work and how we can parameterize them so that users can configure them from the layout editor. Snippets enable theme developers to create custom elements without having to write C# classes. All that's required is a Razor view.

In addition to being able to create snippets, we also saw how to provide custom snippet field type editors, simply by creating another Razor view that follows a particular naming convention.