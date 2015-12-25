## 11. Snippets ##
In chapter 3 we learned about *Snippet Elements*. They are elements dynamically discovered based on Razor views whose name follow a certain convention: Razor views that end with **Snippet.cshtml** are harvested as a Snippet element. For example, **ButtonSnippet.cshtml** would yield an element called **Button Snippet**.

As of Orchard 1.10 this feature was enhanced by yours truly to *provide parameterizable snippets*. A parameterized snippet enables users to add snippet elements to the layout editor and *configure these snippets with custom settings*.

To add settings to a snippet, all the theme developer needs to do is invoke an Html helper called `SnippetField`. This nifty helper serves two purposes:

1. To provide meta information to the element editor about the configurable field.
2. To read the configured value for the field when the element is being rendered.

The signature of the helper looks like this:  

```
SnippetFieldDescriptorBuilder SnippetField(this HtmlHelper htmlHelper, string name, string type = null) {}
```

As you can see, it accepts a *name* and a *type*. The name represents the technical name of the field, while the type represents the data type of the field. At the time of this writing, only one type is supported: *text*.
The method returns a thing called `SnippetFieldDescriptorBuilder` which enables the theme developer to provide more information to the snippet field using a fluent API, currently just a *DisplayName* and *Description*, both of which are used when presenting the user with the element editor dialog in the back-end.

Parameterized snippets are a great tool for theme developers to create custom elements without the need to write element classes. All they need to do is create a Razor view and provide the necessary markup.

To get a good understanding of what you can do with parameterized snippets, let's create one.

#### Demo: Parameterized Snippets ####
In this demo, we'll build a reusable, parameterized snippet called *JumbotronSnippet* with two fields: Caption and Text. The idea is that users can add any number of jumbotrons to their pages and provide a caption and a body text.

The first thing to do is create a view called "JumbotronSnippet.cshtml" in our theme and provide the following markup:

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

Notice that I'm chaining the `DisplayedAs` and `WithDescription` methods. The fluent API prevents us from having to first instantiate a snippet field, initializing it with desired values, and then render it. 

With this snippet in place, all we need to do next is make sure that the *Layout Snippets* feature is enabled and add the `Jumbotron` element to our canvas.

![](./figures/fig-80-adding-jumbotron.png)

Notice that as soon as you drag & drop a Jumbotron element to the page, the element editor dialog appears, showing the two fields as defined in the `JumbotronSnippet.cshtml` file:

![](./figures/fig-81-jumbotron-editor.png)

As you can see, the editor took the information from the snippet field descriptors as created in the view and used it to render appropriate input controls.
Yeah, that's pretty awesome, even if I do say so myself.

Let's see how this looks on the front end:

![](./figures/fig-82-jumbotron-front-end.png)

### Creating Custom Editors ###


### Summary ###
In this chapter, we learned about shapes, which are dynamic objects that serve as view models for shape templates. Shape templates take care of generating HTML, and can be implemented either using Shape methods or Razor views. We also learned about shape alternates, which are key to understanding how to customize shape templates in your custom theme.

We then looked at a specific category of elements: Part and Field elements. They provide the user with great control over where to place them on a canvas, however, it's the theme's responsibility to prevent the default shapes from being rendered using **Placement.info**.

Although the Layouts module comes with a default CSS grid stylesheet, many themes these days will come with their own CSS grid frameworks, such as the popular Bootstrap framework. We learned how to take advantage of such frameworks by overriding the Grid, Column and Parts.Layout shape templates.

We learned a bit about the responsive aspects of Bootstrap and how to take advantage of that using the Layout editor to gain control over column sizes depending on various breakpoints.

To learn more about Bootstrap and responsive grids, visit http://getbootstrap.com/css.

Finally, we demonstrated how snippets work and how we can parameterize them so that users can configure them from the layout editor. Snippets enable theme developers to create custom elements without having to write C# classes. All that's required is a Razor view and some markup. 

And that's actually it for part 2! In the first part we looked at the Layouts module from a user's perspective. In part two, we looked at the module from a theme developer's perspetive. In the third and final part, we'll do a deep dive and check this module out from a module developer's perspective and learn about all the extensibility points that the Layouts module has to offer. Fasten your seatbelt!   