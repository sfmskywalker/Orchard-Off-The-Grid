## Layout Templates ##
In this chapter, we will look at Layout Templates, what they are and what they bring to the table.
In a nutshell, Layout Templates, or Templates for short, are content types that have the `LayoutPart` attached, but which is configured to be used as a template. Any content item of such content types are considered templates that can be applied to other content items that have the `LayoutPart` attached.

If you're familiar with ASP.NET WebForms and Master Pages, you can regard Layout Templates as the functional equivalent of Master Pages.

This is great for scenarios where you have a layout that is commonly used across your content pages. For example, if you have a bunch of Page content items where you want to use a two-column layout with a one-column row before and three-column row after, instead of recreating this layout for each Page, you can create this layout once as a template, and apply this template to each of your Page content items, and just add content elements to the various columns.

### The Layout Content Type ###
When the `Layouts` feature is enabled, it defines a new content type called `Layout`, and consists of the following parts:

- Common Part
- Identity Part
- Layout Part
- Title Part

It's the `LayoutPart` that we're interested in. When you go to the content definition screen of the Layout type and expand the LayoutPart's settings, you'll see that its **Use as Template** setting is checked. This setting used when querying for layout template content items when populating the dropdown list of the Layout Editor. If you never created a Layout content item, you never saw this dropdown list, since its only rendered if there's at least one template in the system.

![Figure 50 - The Layout Part of the Layout type is configured to be used as a template](http://i.imgur.com/rKGY1vZ.png)

*Figure 50 - The Layout Part of the Layout type is configured to be used as a template*

### Trying it out ###
In this demo, we will create a new Layout content item with a grid, some rows and columns, and apply that to a few Page content items.

First, we need to create the Layout content item to be used as a template. To do that, click on the *Layouts* menu item from the admin menu and then click the *Create New Layout* button on the top right side of the window. Doing so will take you to a new screen where you get to enter a *Title* and design a layout. I used *"Master Layout"* as the title and added a grid with three rows and some columns (figure 51).

![Figure 51 - A new Layout with an empty grid](http://i.imgur.com/P2fwJO5.png)

*Figure 51 - A new Layout with an empty grid*

Publish this layout and create a new Page content item. When you do, you'll notice a dropdown list labeled *Use existing layout:* has appeared with a single entry: *Master Layout*. This dropdown list enables us to apply the *Master Layout* to our new page. Before you do, you'll probably want to make sure you delete the default Grid element first, because otherwise the Layout Editor will try and keep that element and add it to the first available container it can find on the template layout being applied. In some cases this may be desirable, but in this demo we don't want that.

After you applied the *Master Layout*, you'll notice that the elements inherited from that template are not selectable or editable. With the exception of the Column elements - they are still selectable so that you can still add elements to them by using drag & drop or by pasting using `CTRL + V`.

Let's add an `Html` element to the first row and column using the following markup:

    <h2>Page 1</h2><p>This is page 1.</p>

![Figure 52 - A new Page with the Master Layout template applied and an Html element added to the first row and column.](http://i.imgur.com/rc9NQ6K.png)

*Figure 52 - A new Page with the Master Layout template applied and an Html element added to the first row and column.*

A powerful feature of layout templates is that whenever you make a change to a template, that change gets applied to all content items using that template. For example, imagine we wanted to display some footer text on all pages using the Master Layout. All we would have to do is update the Master Layout. Let's try that out as well.

First, we'll create another page so we can see that the change to the template does in fact gets reflected on all "child" pages. Create a new page with title "Page 2", use the *Master Layout* as its template, and add an HTML element with the following markup:

    <h2>Page 2</h2><p>This is page 2.</p>

Before going back to the *Master Layout* content item to add an element to the footer column, let's spice things up a bit. What we'll do instead of adding some Html element, we'll add a Shape element, with a shape created as a *Template* content item. In order to do so, we must first enable the *Razor Templates* feature. Once enabled, create a new *Template* content item by going to the *Templates* admin menu item and clicking the *Create New Template* button. Use "PageFooterContent" as its title, and the following Razor code as its content:

    All Rights Reserved &copy; @DateTime.Now.Year

Publish this new template, go back to the *Master Layout* content item, and add a new `Shape` element to the last row and column of the layout. Use "PageFooterContent" (the name for the Template we just created) as the shape type, hit "Save" and publish the changes made to the layout. When you now navigate to Page 1 and Page 2 on the front end, you'll see that both of them show the added shape element, since both of them are using the *Master Layout* template.

![Figure 53 - Changes made to the linked layout template get applied to every content item using that layout template.](http://i.imgur.com/pqhegCI.png)

*Figure 53 - Changes made to the linked layout template get applied to every content item using that layout template.*

> Don't confuse the *Template* content type with the *Layout* content type. Although both represent a template, they are different kinds of templates: The *Template* type represents a shape template, whereas the *Layout* content type represents a layout template.

### Sealed elements and Placeholder Containers ###
When creating Layout templates, you are not limited to just creating grids as we have just seen. However, as soon as you add an element to a container on a layout template, its child layouts (the layouts that use that template) can no longer add more elements to that container, because it will have been *sealed*.

> Sealed elements are elements part of a layout template and can therefore not be edited in child layouts.

If you did want child layouts be able to add additional elements to a sealed container, al you have to do is add a nested child container element to the container in the Layout content item.

For example, in our footer demo, we added a Shape element to the footer column in the layout template (*Master Layout*). If we go to Page 1 or Page 2's edit screen, we'll find that we can no longer add elements to that footer column, since, as we just learned, that column has been sealed. To change this, go back to the *Master Layout* and add the Canvas element above the Shape element. This will enable us to still add content in the footer column above the sealed Shape element.

![Figure 54 - An Html element has been added to the placeholder Canvas element in the footer column, allowing us to add more elements to the footer from child layouts.](http://i.imgur.com/fls050D.png)

*Figure 54 - An Html element has been added to the placeholder Canvas element in the footer column, allowing us to add more elements to the footer from child layouts.*

> One notable downside of this approach is of course that by depending on placeholder containers, an additional `<div>` element will be rendered, which may not be desirable. To work around that, you could override the shape template for the Canvas element and not render the outer `<div>` HTML element. For that you probably want to override just an alternate of the shape. The set of alternates are at the time of this writing limited, but there's a [GitHub issue](https://github.com/OrchardCMS/Orchard/issues/5842 "GitHub Issue #5842 - More alternates for Layout Elements") that requests more alternates be available. One such alternate could be based on the fact whether or not the element is sealed, which would be perfect to override.

### Summary ###
That's pretty much all there is regarding Layout templates. Layout templates are there to be used as templates for content items using the `LayoutPart`. Elements inherited from a template are sealed, which means they cannot be modified from child layouts. In order to be able to add elements to a container inherited from a template, that container must be empty, otherwise it becomes sealed as well.

In conclusion, Layout templates are a powerful way to reuse commonly used layouts across many pages. Changing a template will affect all of its child layouts.

In the next chapter, we'll look at a feature that is similar in concept, but acts at the Element level: Element Blueprints.