## Layout Templates
In this chapter, we will look at layout templates.

Layout templates are content items that have the Layout Part attached, but which is marked to be usable as a template for other content items that have a layout, such as the **Page** content item of a default Orchard installation. In that sense, layout templates are to layouts like Master pages are to WebForms pages.

### The Layout Content Type
When the **Layouts** feature is enabled, it defines a new content type called **Layout**, and consists of the following parts:

- Common Part
- Identity Part
- **Layout Part**
- Title Part

When you go to the content definition screen of the **Layout** type and expand the **LayoutPart**'s settings, you'll see that its **Use as Template** setting is checked.

![Figure 4.1 - The Layout Part of the Layout type is configured to be used as a template](./figures/fig-4-1-layoutpart.png)

*Figure 4.1 - The Layout Part of the Layout type is configured to be used as a template*

This setting is used to populate the **Use existing layout** dropdown list of the Layout Editor (figure 4.1). The templates dropdown list is only present when there's at least one template in the system.

![Figure 4.2 - The templates dropdown list becaomes visible when there's at least one layout template available](./figures/fig-4-2-templates-dropdown.png)

*Figure 4.2 - The templates dropdown list becaomes visible when there's at least one layout template available*

#### Trying it out
Let's give it a try and create a new **Layout** content item. We'll add a grid, some rows and columns, and apply that to a few **Page** content items.

First, create the **Layout** content item to be used as a template. To do that, click on the **Layouts** menu item from the admin menu and then click the **Create New Layout** button on the top right side of the window. Doing so will take you to a new screen where you get to enter a **Title** and design a layout. I used "Master Layout" as the title and added a grid with three rows and some columns (figure 4.3).

![Figure 4.3 - A new Layout with an empty grid](./figures/fig-4-3-new-layout-template.png)

*Figure 4.3 - A new Layout with an empty grid*

Publish this layout and create a new **Page** content item. When you do, you'll notice a dropdown list labeled **Use existing layout:** has appeared with a single entry: **Master Layout**. This dropdown list enables us to apply any available layout to our new page. Before you do, you'll probably want to delete the default Grid element first, because otherwise the layout editor will try and keep that element and add it to the first available container it can find on the template layout being applied.

After you applied the layout, you'll notice that the elements inherited from that template are not selectable or editable, with the exception of the column elements - they are still selectable so that you can still add elements to them.

Let's add an **Html** element to the first row and column using the following markup:

    <h2>Page 1</h2><p>This is page 1.</p>

![Figure 4.4 - A new Page with the Master Layout template applied and an Html element added to the first row and column.](./figures/fig-4-4-page-1.png)

*Figure 4.4 - A new Page with the Master Layout template applied and an Html element added to the first row and column.*

Whenever you make a change to a template, that change gets applied to all content items using that template. For example, let's say that you wanted to display some footer text on all pages using the Master Layout. All you would have to do is update the **Master Layout**. Let's try it.

First, create another page so you can see that changes made to the template gets reflected on all "child" pages. Create a new page with title "Page 2", use the *Master Layout* as its template, and add an HTML element with the following markup:

    <h2>Page 2</h2><p>This is page 2.</p>

What we'll do next instead of adding another **Html** element, we'll add a **Shape** element just  to make this example a little bit more interesting. We'll create a *Template* content item and use that as the shape type.

Go ahead and enable the *Razor Templates* feature. Once enabled, create a new *Template* content item by going to the *Templates* admin menu item and clicking the *Create New Template* button. Use **"PageFooterContent"** as its title, and the following Razor code as its content:

    All Rights Reserved &copy; @DateTime.Now.Year

Publish this new template, go back to the **Master Layout** content item, and add a new **Shape** element to the last row and column of the layout. Use **"PageFooterContent"** (the name for the Template we just created) as the shape type, and hit **Save** to publish the changes. When you now navigate to **Page 1** and **Page 2** on the front-end, you'll see that both of them show the added shape element.

![Figure 4.5 - Changes made to the linked layout template get applied to every content item using that layout template.](./figures/fig-4-5-layout-template-changes-applied.png)

*Figure 4.5 - Changes made to the linked layout template get applied to every content item using that layout template.*

> Don't confuse the **Template** content type with the **Layout** content type. Although both represent a template, they are different kinds of templates: The **Template** type represents a shape template, whereas the **Layout** content type represents a layout template.

### Sealed elements and Placeholder Containers
When applying a layout template, the elements inherited from that template are marked as **sealed**. Sealed elements can not be modified by the user. If the sealed element is an empty container, the user can still add elements to it. However, if this container contains at least one element added via the template, then the user cannot add additional elements. The thinking here is that when you design a template, you want to prevent users from messing up this template. As a template "designer", you need to explicitly provide empty containers that users can use to add elements to.

For example, in our footer demo, we added a Shape element to the footer column in the layout template (*Master Layout*). If we edit Page 1 or Page 2, we'll find that we can no longer add elements to that footer column, since that column has been sealed. To change this, go back to the **Master Layout** and add the **Canvas** element. This will enable us to still add content to the footer column.

![Figure 4.6 - An Html element has been added to the placeholder Canvas element in the footer column, allowing more elements to be added to the footer](./fig-4-6-additional-content-in-footer-canvas.png)

*Figure 4.6 - An Html element has been added to the placeholder Canvas element in the footer column, allowing more elements to be added to the footer*

> One downside of this approach is that by depending on placeholder containers, an additional `<div>` element will be rendered, which may not be desirable. To work around that, you could override the shape template for the Canvas element and not render the outer `<div>` HTML element. For that you probably want to override just an alternate of the shape. The set of alternates are at the time of this writing limited, but there's a [GitHub issue](https://github.com/OrchardCMS/Orchard/issues/5842 "GitHub Issue #5842 - More alternates for Layout Elements") that requests more alternates be available. One such alternate could be based on the fact whether or not the element is sealed.

### Summary ###
Layout templates are templates for content items using the **LayoutPart**. Elements inherited from a template are sealed, which means they cannot be modified from child layouts. In order to be able to add elements to a container inherited from a template, that container must be empty, otherwise it becomes sealed as well.

In conclusion, Layout templates are a powerful way to reuse commonly used layouts across many pages. Changing a template will affect all of its child layouts.