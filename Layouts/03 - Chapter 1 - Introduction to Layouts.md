## Introduction to Layouts
The release of Orchard 1.9 introduced a new module called **Orchard.Layouts**. 
In this chapter, I will introduce you to what this module is and why it was created.

### Defining Layouts
Layouts are everywhere. You find them in newspapers and magazines, in the office and your living room. Cities, planets and the universe all have a layout. Elements, objects and shapes that are placed in a particular position relative to each other, are said to be part of a layout.

In other words, *a layout is an arrangement of elements*.

Great! So how does this relate to Orchard?

### Orchard and Layouts
In the world of websites, **a layout is the arrangement of visual elements on a page**. Visual elements include things such the site navigation, side bar, main content and footer and blocks of contents.

A website's layout is provided by the website's active theme. Content editors may select an existing layout, but they don't get to change that layout.

The **Orchard.Layouts** module does not change this fact; it is not a web design tool. Instead, it provides control over the layout of **content**.

#### Layout and Elements
When you enable the `Layouts` feature of the `Orchard.Layouts` module, a new content part called `LayoutPart` is added to the system and attached to the `Page` content type by default. The `LayoutPart` enables content editors to visually arrange elements on a canvas.

> Elements are a new type of entity in Orchard and represent the things you can place on a canvas.

### When to use Orchard.Layouts
As mentioned, the Layouts module enables content editors to manage the layout of contents.

Let's look at a simple example. Let's say we have a web page with content that consists of two paragraphs:

![Figure 1.1 - A web page with two paragraphs.](./figures/fig-1-1-sample-layout-a.png)
*Figure 1.1 - A web page with two paragraphs.*

Now, let's say that as a content editor, we wanted to display the two paragraphs as two adjacent columns instead as follows:

![Figure 1.2 - A web page with two columns.](./figures/fig-1-2-sample-layout-b.png)
*Figure 1.2 - A web page with two columns.*

Before we see how to achieve that with the layouts module, let's explore our options *before* **Orchard.Layouts**.

**Option 1 - Direct HTML manipulation**

One option is to edit the HTML source of the `BodyPart` content and apply your HTML skills by adding an HTML table element, a single row and two cells, or maybe even using Bootstrap's Grid CSS classes and apply them on `div` elements. Although that would certainly work, it is far from ideal, because it would require the content editor to know about HTML tables and how to work with them. For a simple two-column layout for a body of text this may not be that big of a deal, but it becomes icky real fast when working with more complex layouts.

Disadvantages of this approach:

- Requires the user to understand and apply relatively advanced HTML.
- Content is mixed with markup, so it's harder to change the content without accidentally breaking the HTML. 

![Figure 1.3 - Using HTML tables to create a two-column layout](./figures/fig-1-3-sample-layout-b-table-content.png)
*Figure 1.3 - Using HTML tables to create a two-column layout*

**Option 2 - Widgets and Zones**

Another option is to provide two zones, let's say **AsideFirst** and **AsideSecond**. Then we would simply add an **Html Widget** to both zones. Although this approach works, one disadvantage is that now the textual content becomes unrelated to the content item itself, since we're now using widgets. To manage the content on this page, the user has to go to the **Widgets** screen, create a layer just for the page they're interested in, and add two widgets. Now imagine you have 20 pages. That means 20 widget layers, 2 HTML widgets per layer, and 20 **Page** content items with no contents. Talk about maintenance nightmares.

![Figure 1.4 - Adding two widgets per layer](./figures/fig-1-4-sample-layout-b-widgets.png)
*Figure 1.4 - Adding two widgets per layer*

And this is just two columns. Imagine you have other types of layouts, for example one row with two columns, another row with 4 columns, and perhaps rows with one column taking up 2/3 of the row and a second column 1/3 of the row. Crazy. Allowing this level of freedom to the content editor user would easily end up in a maintenance nightmare.

Disadvantages if this approach include:

- The widgets are unrelated to the content item, making the site much more difficult to maintain.
- Requires one layer per content item, making the site much more difficult to maintain.
- Difficult to introduce other, more complex layouts of contents.

> There is a way to associate widgets with content items directly by taking advantage of a free gallery module called [IDeliverable.Widgets](http://www.ideliverable.com/products/ideliverable.widgets "IDeliverable.Widgets"), which provides a **WidgetsContainerPart**. When you attach this part to your content type, you can associate widgets with your content items directly, instead of having to create layers. Although this is better than having to create a layer per page, it is still not ideal.

**Option 3 - Content Fields and Placement.info**
 
Yet another option is to create various content types, where a content type would have multiple content fields.
For example, we could create a new content type called `TwoColumnPage` with two `TextField` fields. The theme would use `Placement.info` to place one field into the `AsideFirst` zone and the other field into the `AsideSecond` zone.

![Figure 1.5 - Creating a special content type for two columns.](./figures/fig-1-5-sample-layout-b-fields.png)

*Figure 1.5 - Creating a special content type for two columns.*

Although this option is (arguably) better than the previous option using widgets, there is still the limitation of freedom when you want to introduce additional layouts. Not to mention the fact that we're now basing the content type name on what it looks like, rather than its semantic meaning. Not good.

Disadvantages of this approach:

- Content type pollution - the number of content types grows fast, making it more complex for the user.
- Hard to change the layout - The only way to change a different layout is by creating a new content item of another type and then copying over the contents.
- Mixing the semantic meaning of the content type with how it is rendered visually.

#### Enter Orchard.Layouts
with the release of 1.9, a fourth option appeared. And a much better one too. I sometimes even wonder how I have been able to create websites without.

With **Orchard.Layouts**, creating a two-column layout could not be simpler. Simply add a **Grid** element with a single **Row** and two **Column** elements to the **Canvas**, add any number of content elements, and we're done. No need for HTML editing, no additional zones, no widgets and layers, and no additional content types. The next figure gives you an idea of how the layout editor looks like.

![Figure 1.6 - Say hello to the new kid on the block - the LayoutPart and its layout editor](./figures/fig-1-6-layout-editor.png)
*Figure 1.6 - Say hello to the new kid on the block - the LayoutPart and its layout editor*

As you can see, the layout editor consists of a design surface and a toolbox. 

The layout editor makes creating content layouts a breeze, even complex ones.

![](./figures/fig-1-7-layout-editor-layout-c.png)
*Figure 1.7 - Complex layouts made simple*

To sum it up, this is why we need **Orchard.Layouts**:

- No longer necessary to create page-specific layers and widgets to achieve complex layouts of contents.
- No longer necessary to create specific content types just for supporting multiple layouts.
- An easy way to create various layouts of content.

### Frequently Asked Questions
Some of the frequently asked questions about the Layouts module are:

1. Where did **BodyPart** go?
2. Do I have to use Layouts?
3. What happens to my existing site and its contents when upgrading to Orchard 1.9?
4. Are **Elements** Content Items?
5. How does **Orchard.Layouts** relate to widgets?
7. Does **Orchard.Layouts** work with grid systems such as **Bootstrap**?

Let's go over each question one by one.

#### Where did the BodyPart go?
The **BodyPart** is still a happy citizen within the Orchard, and will remain as such. The **LayoutPart** simply serves a different purpose (laying out content). The **BodyPart** is great when all you need is an editable body of text.
New Orchard installations use **LayoutPart** by default for the **Page** content type, but you're free to remove the **LayoutPart** and attach the **BodyPart**. Or you can even chose to delete the **Page** content type entirely and create custom ones. It's all about choice.

#### Do I have to use Layouts? ####
Not at all. If you're starting with a fresh Orchard installation and prefer to work with the **BodyPart**, all you need to do is remove the **LayoutPart** from the **Page** content type and attach the **BodyPart**. Similarly, you;re free to delete the **Layout** content type and **LayoutPart** and disable the feature entirely.  

#### What happens to my existing site and its contents when upgrading to Orchard 1.9? ####
If you're worried about your existing content, fear not, dear reader. When you upgrade your site to 1.9 or beyond, the Layouts feature will not be automatically enabled. And even when you enable the feature yourself, it will not change your content type definitions. If you *do* want to use Layouts on existing installations that have been upgraded, you will simply have to enable the **Layouts** feature and attach the **LayoutPart** manually if you want to give it a try.

#### Are Elements Content Items? ####
Elements are not content items, but are instances of the `Element` class. The elements on a layout are serialized as part of the content item's **InfoSet**.

#### How does Orchard.Layouts relate to Orchard.Widgets?
The Layouts feature is unrelated to widgets. But both Widgets and Layouts modules share a similar purpose, namely to provide users a level of control of getting content on to the screen. The Widgets module takes the approach of letting the user associate content items (stereotyped as "Widget") with **Layers** and **Zones**, whereas the `Layouts` module takes the approach of letting the user drag & drop a bunch of elements onto a canvas.

> Although elements and layouts are currently scoped to content items, there are plans to have the Widgets module be refactored to take advantage of the elements provided by layouts. so instead of adding **Widget content items** to zones and layers, you would be adding **elements** to zones and layers. 

#### Does Orchard.Layouts work with grid systems such as Bootstrap?
Yes, and this scenario is well-supported. The `Grid`, `Row` and `Column` elements map nicely to CSS grid framework elements such as Bootstrap's `container`, `row` and `col-md-*` CSS classes. You will have to override the shape templates for these elements in your theme and modify the CSS classes being rendered. We'll look into this in detail in chapter 10.

### About Grid Frameworks ###
Many websites these days use CSS grid frameworks to help arrange visual elements. Two examples of CSS grid frameworks are [Bootstrap](http://getbootstrap.com/) and [Foundation](http://foundation.zurb.com/).

The basic idea behind a grid is to help layout elements on a canvas in a uniform, consistent manner.
It is an invisible structure (although visible while working on your layout using an editor) that helps ensure you place elements in the right location to achieve balance as well as to help with alignment, continuity and consistency of the design.

Grid layouts have a fixed maximum number of columns per row, and each cell in a row has a span of at least one column and at most the maximum number of columns. 

Figure 8 shows an example of a 12-column grid onto which visual elements could be laid out.

![Figure 1.8 - A 12 column grid](./figures/fig-1-8-sample-grid.png)

*Figure 1.8 - A 12 column grid*

As you can see, the total span size of cells in each row always adds up to the maximum number of columns.

Grids are great for designing layouts. The layout-editor enables users to create such grids easily, as we'll see in the next chapter.

#### Responsive Layouts ####
One big advantage of working with CSS grid frameworks for the web is that these frameworks help with implementing *responsive layouts*. A responsive layout is one that adapts itself automatically based on the view-port's width of the browser. As the view-port width decreases in size, the cells in a grid decrease in size as well up to a specified minimum, after which the cell will break onto the next line. The next figure shows an example of a layout on a desktop device as well as on a mobile device, and how the various columns collapse underneath one another.

![Figure 1.9 - A responsive grid works well on various widths](./figures/fig-1-9-responsive-grid.png)

*Figure 1.9 - A responsive grid works well on various widths*

### Summary ###
In this chapter, we introduced the new Layouts module, what it is for and why we need it.   

We learned that **Orchard.Layouts** enables users to arrange elements of various types onto a canvas. 
We also explored what problem the Layouts module solves and when to use it. Where we had to resort to cumbersome solutions before, Layouts makes it a breeze to create all sorts of content layouts. 

We concluded with a brief section on CSS Grid Frameworks. The layout editor of the Layouts module can be used to create grids that render CSS framework specific CSS classes, which we'll explore in chapter 10.
 
In the next chapter, we'll have a closer look at **Orchard.Layouts** from a user's perspective, and see how to actually use it.