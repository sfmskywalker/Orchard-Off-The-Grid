## Introduction to Layouts ##

Layouts are everywhere. You find them in newspapers and magazines, in the office and your living room. Cities, planets and the universe all have layout. It is the layout of things. You could say that elements, objects and shapes that are placed in a particular position relative to each other, are said to be part of a layout.

In other words, **a layout is an arrangement of elements**.

In the world of websites, **a layout is the arrangement of visual elements on a page**. Visual elements include things such the site navigation, side bar, main content and footer and blocks of contents.

### Orchard and Layouts ###
In the world of the **Orchard.Layouts** module, a layout is **an hierarchical collection of elements placed on a canvas**.

This chapter we'll get to see what the Layouts module brings to the table and why (and when) we should use it.

Before we dig in, let's talk about the term "layout" for a bit, as this term is kind of overloaded in the context of Orchard. This will help get us a better understanding of what the Layouts module is, and what it is not.

#### Layout and Layout.cshtml ####
If you ever developed an Orchard theme, you'll be very familiar with the Razor view called **Layout.cshtml**. It basically contains all of the HTML of the site's overall design and layout. It is the shape template for the **root Layout shape** and renders the main layout of the site, also rendering **zones**.

This Layout shape is unrelated to the Layouts module.

#### LayoutPart and Layout Content Item ####
When you enable the `Layouts` feature of the `Orchard.Layouts` module, a new content part called `LayoutPart` is added to the system and attached to the `Page` content type by default. The `LayoutPart` enables content editors to visually arrange elements on a canvas. Elements are a new type of entity in Orchard and represent the things you can place on a canvas.

In addition to attaching the `LayoutPart` to the `Page`, the `Layouts` feature adds another content type called `Layout` which also has the `LayoutPart` attached. The key difference between a `Page` and a `Layout` content item is that `Layout` content items are intended to serve as **layout templates** that you can apply to other content items with the `LayoutPart` (such as `Page`). We'll get into layout templates in more detail later on.

### When to use Orchard.Layouts ###
So when would you want to use the Layouts module, and why was it created in the first place?

The easiest way to understand why the Layouts module is useful is by looking at a simple example. Consider the following web page:

![Figure 1.1 - A simple web page with two paragraphs.](./figures/fig-1-1-sample-layout-a.png)
*Figure 1.1 - A simple web page with two paragraphs.*

This web page consists of two paragraphs of text. Now, let's say that as a content editor, we wanted to display the two paragraphs as two adjacent columns instead as follows:

![Figure 1.2 - A simple web page with two columns.](./figures/fig-1-2-sample-layout-b.png)
*Figure 1.2 - A simple web page with two columns.*

Before we see how easy it is to achieve that with the Layouts module, let's explore our options we before the advent of Layouts.

**Option 1 - Direct HTML manipulation**

One way is to edit the HTML source of the `BodyPart` content and apply your HTML skills by adding an HTML table element, a single row and two cells, or maybe even using Bootstrap's Grid CSS classes and apply them on `div` elements. Although that would certainly work, it is far from ideal because it would require the content editor to know about HTML tables and how to work with them. For a simple two-column layout for a body of text this may not be that big of a deal, but it becomes icky real fast when working with more complex layouts.

Disadvantages of this approach:

- Requires the user to understand and apply relatively advanced HTML.
- Content is mixed with markup, so it's harder to change the content without accidentally breaking the HTML. 

![Figure 1.3 - Using HTML tables to create a two-column layout](./figures/fig-1-3-sample-layout-b-table-content.png)
*Figure 1.3 - Using HTML tables to create a two-column layout*

**Option 2 - Widgets and Zones**

Another option is to provide two zones, let's say *AsideFirst* and *AsideSecond*. Then we would simply add an Html Widget to both zones. Although this approach works, one disadvantage is that now the textual content becomes unrelated to the content item itself (since we're now using widgets). To manage the content on this page, the user has to go to the Widgets screen, create a layer just for the page they're interested in, and add two widgets. Now imagine you have 20 pages. That means 20 widget layers, 2 HTML widgets per layer, and 20 Page content items with no contents. Talk about maintenance headaches.

![Figure 1.4 - Adding two widgets per layer](./figures/fig-1-4-sample-layout-b-widgets.png)
*Figure 1.4 - Adding two widgets per layer*

You can imagine this too becomes real icky real fast. And here we're talking about supporting just two columns. Imagine you have other types of layouts, for example one row with two columns, another row with 4 columns, and perhaps rows with one column taking up 2/3 of the row and a second column 1/3 of the row. Crazy. Allowing this level of freedom to the content editor user would easily end up in a maintenance nightmare for both the developer as well as the user.

> I have seen a fair amount of Orchard implementations taking this approach, causing a real maintenance nightmare for the content editors. Creating content, widgets and layers is one thing. Finding the appropriate widgets to edit is quite another. Enabling the Widget Wrapper feature is a must in this sort of scenarios, since it renders an *Edit* link on each widget on the front-end, making it easier to edit the site.
> Another solution to this problem is to take advantage of a free gallery module called [IDeliverable.Widgets](http://www.ideliverable.com/products/ideliverable.widgets "IDeliverable.Widgets"), which provides a *WidgetsContainerPart*. When you attach this part to your content type, you can associate widgets with your content items directly, instead of having to create layers.

Even if you are fine with working with widgets and zones like this, you're still quite limited in terms of creating other content layouts. For example, adding another row with 3 columns and then another with two columns again would be far from straightforward.

Disadvantages if this approach:

- The widgets are unrelated to the content item, making the site much more difficult to maintain.
- Requires one layer per content item, making the site much more difficult to maintain.
- Difficult to introduce other, more complex layouts of contents.

**Option 3 - Content Fields and Placement.info**
 
Yet another solution is to create various content types, where a content type would have multiple content fields.
For example, we could create a new content type called `TwoColumnPage` with two `TextField` fields. The theme would use `Placement.info` to place one field into the `AsideFirst` zone and the other field into the `AsideSecond` zone.

![Figure 1.5 - Creating a special content type for two columns.](./figures/fig-1-5-sample-layout-b-fields.png)
*Figure 1.5 - Creating a special content type for two columns.*

Although this option is arguably better than the previous option using widgets, there is still the limitation of freedom when you want to introduce additional layouts. Not to mention the fact that we're now basing the content type name on what it looks like, rather than its semantic meaning. For example, whether a page has two columns or ten columns, it is still a page.

Disadvantages of this approach:

- Content type pollution - the number of content types grows fast, making it more complex for the user.
- Hard to change the layout - The only way to change a different layout is by creating a new content item of another type and then copying over the contents.
- Complex - the user now needs to understand what content will end up where without any visual cues.
- Mixing the semantic meaning of the content type with how it is rendered visually.

But that was before we had Layouts.

#### Orchard.Layouts ####
All of the above workarounds and their disadvantages went away with the advent of `Orchard.Layouts`. Creating a two-column layout now could not be simpler. Simply add a `Grid` element with a single row and two columns to the canvas, add any number of content elements, and done. No need for HTML editing, no additional zones necessary, no widgets and layers, and no additional content types necessary.

![Figure 6 - Say hello to the new kid on the block - the LayoutsPart and its layout editor](./figures/fig-1-6-layout-editor.png)
*Figure 6 - Say hello to the new kid on the block - the LayoutsPart and its layout editor*

The `LayoutPart` editor makes creating content layouts a breeze. So far we only discussed how to deal with a two-column layouts, and as we've seen, it was a pain. But with Layouts, it could not be easier. Even layouts that seemed complex to create before are now dead simple to implement as seen in figure 7 and figure 8.

![](http://i.imgur.com/QiO0RcC.png)
*Figure 7 - Complex layouts made simple*

To summarize, this is why were in dire needs of the Layouts module:

- No longer necessary to create page-specific layers and widgets.
- No longer necessary to create specific content types just for supporting multiple layouts.
- A simple way to create an infinite amount of layout types per page.

### Frequently Asked Questions ###
With the release of the Layouts module, interesting questions were raised. Some of the frequently asked questions about the Layouts module are:

1. Where did `BodyPart` go? Is it still relevant?
2. What happens to my existing site when upgrading to Orchard 1.9?
3. Do I *have* to use Layouts?
4. Are Elements Content Items?
5. How does Layouts relate to widgets? Can I use Layouts instead of Widgets?
7. Does it work with grid systems such as Bootstrap? How?

Let's go over them one by one.

#### Where did the BodyPart go? Is it still relevant? ####
The `BodyPart` is absolutely still relevant and did not go anywhere. The `LayoutPart` is not a replacement for `BodyPart`. If all you need is an editable body of text, then `BodyPart` is your guy.
New Orchard installs will use `LayoutPart` by default for the `Page` content type primarily to show the new feature. If you don't need the layout capabilities, then it's arguably more user friendly to just use the `BodyPart`, as it allows the user to start entering and editing content directly.

#### What happens to my existing site when upgrading to Orchard 1.9? ####
Nothing. When you upgrade your site from 1.8 to 1.9, the Layouts feature will not be automatically enabled. This means that your content items with the `BodyPart` will retain those parts.

Even enabling the `Layouts` feature will not cause your `BodyPart` to be removed from your content types. What will happen though when enabling `Layouts` is that the `LayoutPart` will be attached to the `Page` content type. If you don't appreciate that, simply detach it.

#### Do I have to use Layouts? ####
Not at all. Simply disable the feature and detach the `LayoutPart` on new installs of you prefer to work with the `BodyPart`. Layouts is here to serve, not to dictate.

#### Are Elements Content Items? ####
Most certainly not. One of the advantages of using the layouts feature is that no matter how complex your layout becomes, all of the information representing the layout and its elements are stored as a serialized JSON string as part of the `LayoutPart` XML element in the `InfoSetPart`. This means that all of the element information is available as soon as the content item with the `LayoutPart` is loaded.

#### How does Layouts relate to widgets? Can I use Layouts instead of Widgets? ####
The Layouts feature is unrelated to widgets. Although semantically speaking you could regard an *element* as a *widget*, technically speaking they live on completely different planes of reality. Both the `Widgets` and `Layouts` feature of course share a similar purpose: to provide users some level of control of getting content on to the screen. The `Widgets` feature takes the approach of letting the user associate content items (stereotyped as "Widget") with Layers and Zones, whereas the `Layouts` feature takes the approach of letting the user drag & drop a bunch of elements onto a canvas.

As for the second part of the question: "Can I use Layouts instead of Widgets?", the answer is: it depends.

For example, before we had Layouts, the default Orchard installation created three HTML widgets for the homepage and placed them in the *TripelFirst*, *TripelSecond* and *TripelThird* zones, respectively. Now that we have Layouts, this is no longer necessary. The three widgets have been removed and instead are now part of the layout of the "Welcome to Orchard" Page content item.

Could we do the same for the *Menu Widget*? Not really. We could in theory get rid of all of the Zones except for the Content zone, and rely on Layouts to layout the various pages, probably taking advantage of Layout Templates. The problem is, however, that these layouts are bound to content items. This would mean that if you navigated to the Login screen for example, which is served by an MVC controller other than the `ItemController` of the `Contents` module, you would suddenly no longer see the site's main menu.

So the rule of thumb here is: use Layouts to layout content, but don't use Layouts to control UI.

Now, there are plans for the next version of Orchard to unify the Widgets and Layouts worlds into one, where the Widgets module would take advantage of the Layout editor so that users would drag & drop Elements into Zones for a given Layer, instead of adding Widgets. This would still of course not enable you to arrange Zones yourself, but it might bring us one step closer to unlocking that possibility.   

#### Does it work with grid systems such as Bootstrap? How? ####
It most certainly does, and it does so very well. The `Grid`, `Row` and `Column` elements map nicely to Grid CSS framework concepts such as Bootstrap's `container`, `row` and `col-md-*` CSS classes. This means that you could override the shape templates for these elements in your theme and modify the CSS classes being rendered, essentially enabling your users to create Bootstrap grids using the Layout editor. And what's more, the list of Elements is extensible, so you can invent your own elements that render using Bootstrap specific CSS classes, or anything else that you need. Users would simply drag & drop those elements onto the canvas, provide some element-specific properties, if any, and the element takes care of rendering itself.

We'll look into this in much more detail later in this book.

### About Grid Frameworks ###
Modern websites these days use something called CSS grid frameworks to help arrange visual elements. Two examples of CSS grid frameworks are [Bootstrap](http://getbootstrap.com/) and [Foundation](http://foundation.zurb.com/).

The basic idea behind a grid is to help laying out elements on a canvas in a uniform, consistent manner.
It is an invisible structure (although visible while working on your layout using an editor) that helps ensure you place elements in the right location to achieve balance as well as to help with alignment, continuity and consistency of the design.  

Grid layouts have a fixed maximum number of columns per row, and each cell in a row has a span of at least one column and at most the maximum number of columns. 

Figure 8 shows an example of a 12-column grid onto which visual elements could be laid out.

![Figure 1 - A 12 column grid](http://i.imgur.com/JocbOZk.png)

*Figure 8 - A 12 column grid*

> The grid in figure 1 consists of four rows. The first row has a single cell spanning the entire width of the grid and has a span size of twelve columns. The second row has two evenly sized cells; each cell has a span size of six columns. The third row has three evenly sized cells, where each cell has a span size of four columns.
> The last row consists of twelve cells, each cell spanning a single column.

As you can see, the total span size of cells in each row always adds up to the maximum number of columns.

Grids are great for designing layouts. They lead to consistency and bring order and clarity to a design.

The `Orchard.Layouts` layout editor enables users to create such grids easily, as we'll see in the next chapter.

#### Responsive Layouts ####
One big advantage of working with CSS grid frameworks for the web is that these frameworks help with implementing *responsive layouts*. A responsive layout is one that adapts itself automatically based on the viewport's width of the browser. As the viewport width decreases in size, the cells in a grid decrease in size as well up to a specified minimum, after which the cell will break onto the next line. Figure 2 shows an example of a layout on a desktop device as well as on a mobile device.

![](http://i.imgur.com/zyQRuFQ.png)
*Figure 9 - A responsive grid works well on various widths*

### Summary ###
In this chapter, we introduced the new Layouts module and what it is.   

The `Orchard.Layouts` module enables users to arrange elements of various types onto a canvas. 
In fact, we could think of the Layouts module as an Elements engine, since all it really does is enable users to place elements and render them. Layouts emerge by virtue of using the `Grid`, `Row` and `Column` elements.

We also explored what problem the Layouts module solves and when to use it. Where we had to resort to cumbersome solutions before, Layouts makes it a breeze to create all sorts of content layouts. 

We concluded with a brief section on CSS Grid Frameworks. The layout editor of the Layouts module can be used to create grids that render CSS framework specific CSS classes, which we'll explore in later chapters.
 
In the next chapter, we'll have a closer look at the `Orchard.Layouts` module from a user's perspective, and see how we can use it to manage and layout content using grids.