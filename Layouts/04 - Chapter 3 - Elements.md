## Meet the Elements ##
In this chapter, we'll go over all of the available elements in the default Orchard distribution. Most elements should be self-explanatory to use, but I think that some of them could use a little bit of a background to get a good understanding on how to use them.

Elements are categorized using the following categories:

* Layout
* Content
* Media
* Parts
* Fields
* Snippets
* UI

This list is not necessarily exhaustive: third party modules can add their own categories as they see fit. It is however the list of categories available to you from a default Orchard installation.
Not all categories appear immediately: for some elements to become available, you'll have to enable certain features. For other elements to become available, you'll have to attach certain parts and fields to your content types. This all depends on the *element harvesters* being enabled.

> *Element Harvesters* are another key component in the Layouts module. Users don't see them directly, but it's the element harvesters' responsibility to provide the list of available elements to the user. Elements could be provided from any source you can imagine. One source of elements is all implementations of the `IElement` interface. Another source is Razor file names ending in `-Snippet.cshtml`, and yet another source of elements is all of the available content parts attached to the current content type.
> Users don't need to concern themselves with Element harvesters however; that is more the domain of module developers, which we'll cover in detail in the third part of this book.

The following is the complete list of available elements when all features (except the features from `Orchard.DynamicForms`) is enabled:

* Layout
    * **Grid**
    * **Row**
    * **Column**
    * **Canvas**
* Content
    * **Break**
    * **Content Item**
    * **Heading**
    * **HTML**
    * **Markdown**
    * **Paragraph**
    * **Projection** *(requires the `Projection Element` feature to be enabled)*
    * **Text**
* Media
    * **Image**
    * **Media Item**
    * **Vector Image**
* Parts
    * **BodyPart**
    * **CommonPart**
    * **TagsPart**
    * **TitlePart**
* Fields
    * *(Any content field attached to the content type will be made available as an element)*
* Snippets
    * **Shape**
    * *(Any Razor file ending in `Snippet.cshtml` in the current theme will be made available as an element)*
* UI (New as of Orchard 1.10)
    * **Breadcrumbs** *(requires the `UI Elements` feature to be enabled)*
    * **Menu** *(requires the `UI Elements` feature to be enabled)*
    * **Notifications** *(requires the `UI Elements` feature to be enabled)*

> The `Orchard.DynamicForms` module is another new module since Orchard 1.9. This module provides its own set of elements, but it's outside the scope of this book to handle them all. Please checkout **Orchard Off The Grid - Dynamic Forms** for everything you ever wanted to know about the Dynamic Forms module.

Let's go over each category, and in turn their elements, one by one.

### Layout ###
Elements in this category are typically container elements that layout their child elements in a certain way. Certain container elements only support certain child element types. For example, the **Grid** element can only contain **Row** elements, which in turn can only contain **Column** elements.

> As mentioned earlier, this notion of a container only supporting a particular type of child elements, may very well change in a future version of the layout editor, which is one of the goals of the [Ayos](https://github.com/IDeliverable/Ayos) project.

Let's go over each element in this category.

#### Grid ####
The **Grid** element is a container element that can hold only one type of child elements: *Row*.
We use the Grid element whenever we want to create a layout of elements. Grids can contain rows, and rows in turn can contain columns. Although the Layouts module is named that way because it enables users to create layouts, it is thanks to this Grid element that the Layouts module is able to achieve that in the first place.
So it is fair to say that the Grid is a key element.

The Grid element has no content properties, but does have the common element properties (HTML ID, CSS Classes, CSS Styles and Visibility Rule).

![Figure 20 - The Grid element](http://i.imgur.com/JCi5YPz.png)

*Figure 20 - The Grid element*

When you add a new Grid element to the design surface, it will not contain any Row elements automatically. You will need to drag & drop Row elements yourself.

#### Row ####
The **Row** element, like the Grid element, is a container element. However, this element can only contain **Column** elements as its children, and the Row itself can only be a child of a Grid element.

We mentioned that the Layout category contains a **Row** element. However, when we look at the layout editor toolbox, there are definitely more **Row** elements than just one it seems (see figure 20):

![Figure 21 - Preconfigured Row elements](http://i.imgur.com/K0X4B9m.png)

*Figure 21 - Preconfigured Row elements*

What we see here is not 7 types of Row elements, but simply 7 preconfigurations of a single Row element. The first Row toolbox item will add a single Row element with 1 Column element, the second toolbox item will add a single row with 2 columns, etc.

The Row element has one specific toolbar command specific to itself: **Distribute columns evenly**.
As its name implies, this command will evenly distribute the width of the columns of the row based on a *maximum width of 12* units. For example, if a Row has 4 columns with varying widths, and the maximum width is 12 column units, each column will be resized to be exactly 3 units wide.

The Row element has no content properties, but does have the common element properties (HTML ID, CSS Classes, CSS Styles and Visibility Rule).

![Figure 22 - The Row element](http://i.imgur.com/6qUkrGm.png)

*Figure 22 - The Row element*

#### Column ####
The **Column** element, you guessed it, is also a container element. Unlike the Grid and Row elements, the Column element can hold any type of child elements (except for Row and Column elements). Itself, however, can only be a child of a Row element.
A column has two distinctive properties: **Width** and **Offset**. Together, these values make up the **total size** of the column. *Width*, *Offset* and *Total Size* are expressed in a unit called *columns*. A given Row can hold up to 12 column units. What this means is that the total number of column units occupied by column elements in a row cannot exceed 12. For example, if a row has a single column, that column's width cannot be greater than 12. If a row has two columns, the sum of the two column's widths cannot exceed 12. One column could be 4 while another could be 8, making a total of 12. Similarly, if a column has an offset of 1 and a width of 4, its size is 5, which means that the second column cannot be more than 7 columns wide.

Let's see this in action.

First of all, let's start with a Grid, a single Row and 2 Columns:

![Figure 23 - A Row with 2 evenly distributed Columns](http://i.imgur.com/ZgMPDXX.png)

*Figure 23 - A Row with 2 evenly distributed Columns*

Both columns each have a width of 6 column units, making up a total of 12.
Now let's say we wanted to make the first column wider, say 8 columns wide. That will leave only 4 columns left for the second column:

![Figure 24 - A Row with one column with size 8 and the second column of size 4](http://i.imgur.com/sz2BzQp.png)

*Figure 24 - A Row with one column with size 8 and the second column of size 4*

If we wanted to add a third column, we could either drag & drop another Column element from the toolbox into the Row (which would cause the existing columns to decrease in size), or split an existing column into two. Either action would ultimately yield the same result: a new Column element would be added to the Row.

Another feature of the Column element is called *offsetting*. Offsetting simply means to provide an offset value for a column. By default, the offset for any column is 0. Increasing the offset increases the overall size of the column. You typically use an offset if you want the contents of a column to appear more to the right. As an example, let's increase the offset of the first column by one. You can do this either by dragging the left border to the right, by using the little toolbar icon called *Increase column offset*, or by using the keyboard shortcut Alt+Right:

![Figure 25 - A Row with two columns, the first column having an offset of 1, a width of 7, and the second column having a width of 4](http://i.imgur.com/lKc2qvs.png)

*Figure 25 - A Row with two columns, the first column having an offset of 1, a width of 7, and the second column having a width of 4*

What we see in figure 25 is a row with two columns, where the first column has an offset of 1, a width of 7 (making its total size 8) and a second column with a width of 4.

And that's the Column element in a nutshell. Combined with Row and Grid, it is the key ingredient for creating layouts.

The Column element has no content properties, but does have the common element properties (HTML ID, CSS Classes, CSS Styles and Visibility Rule).

![Figure 26 - The Column element](http://i.imgur.com/qtoNWpY.png)

*Figure 26 - The Column element*

#### Canvas ####
The **Canvas** element is the last element in this category and, too, is a container element. Like the Column element, the Canvas can contain any type of element (except for Row and Column elements). Unlike the Column element, the Canvas element can be added to any other container element (except for Grid and Row, since they have an exclusive whitelist of allowed children).

Whenever you create a new layout, that layout always start with a root element of type Canvas, which you'll notice when creating a new Page content item.

The Canvas element is great if you ever need a generic type of container element. Although in most cases you probably won't need it since the Column element already is a container, there are occasions where a Canvas is necessary. The prime example being when creating **Layout Templates**. We'll look at Layout Templates into much more detail in the next chapter, but basically a layout template enables the reuse of precreated layouts. when a layout template is applied to a layout, the elements from the template are *sealed*, which means the user cannot make any modifications to these elements. If a sealed container element is empty however, it will accept elemtns as its children. But if a sealed container element has at least a single child element, no elements can be added to that container. Unless you add an empty, nested child container to that element, serving as a placeholder. The Canvas is perfect for that purpose.

The Canvas element has no content properties, but does have the common element properties (HTML ID, CSS Classes, CSS Styles and Visibility Rule).

![Figure 27 - The Canvas element](http://i.imgur.com/yBm3RBs.png)

*Figure 27 - The Canvas element*

And this concludes the *Layout* category of elements.

### Content ###
The Content category contains all elements that have to do with, unsurprisingly enough, contents.

#### Break ####
The Break element is one of the simplest ones. It has content properties, but does have the common element properties (HTML ID, CSS Classes, CSS Styles and Visibility Rule). The element maps directly to the `<hr>` HTML element.

#### Content Item ####
The **Content Item** element is very similar to the Content Picker Field and enables the user to select one or more content items to render inline on the design surface.
When dropping or editing a Content Item element, the user is presented with a dialog window displaying the content properties of the element (see figure 28). The element has two content properties:

- A list of content item IDs
- The *DisplayType* to use when rendering the selected content items. 

![Figure 28 - The content properties of the Content Item element](http://i.imgur.com/u1nrJid.png)

*Figure 28 - The content properties of the Content Item element*

Being able to select existing content items and render them using custom display types is a very powerful and flexible feature, as it enables you to place content anywhere you like in any display type you want (which can be used for theming). It's pretty close to the way widgets works, except for the fact that you can't reuse widgets.

#### Heading ####
The **Heading** element maps directly to the `<h1>` to `<h6>` HTML elements and have the following content properties:

- Level
- Text

![Figure 29 - The content properties of the Heading element](http://i.imgur.com/Vcxnszj.png)

*Figure 29 - The content properties of the Heading element*

The `Level` indicates the size of the heading and ranges from 1 to 6. For example, if you specify level 3, the `<h3>` tag will be rendered. The `Text` will be rendered as the text value of the HTML element. The following is an example of HTML output when specifying level 6 and the text "Hello Layouts!":

    <h6>Hello Layouts!</h6>

#### Html ####
The **Html** element is probably the most commonly used one when it comes placing content onto the design surface. It has a single content property called *HTML*, which stores the HTML markup. Use this element whenever you want to display textual content anywhere on the design surface.

![Figure 30 - the content properties of the Html element](http://i.imgur.com/DG0EGTq.png)

*Figure 30 - the content properties of the Html element*

The HTML editor you see when editing the HTML element is rendered using the *html* flavor, which means that the editor that appears depends on which feature you have enabled. By default, the `TinyMce` feature is enabled, but if you enable the `CKEditor` feature, that's the editor you'll see when editing Html elements.

#### Markdown ####
The **Markdown** element is functionally the same as the the Html element, but with the difference that it uses Markdown syntax, which gets transformed into HTML when being rendered on the front-end.

![Figure 31 - the content properties of the Markdown element](http://i.imgur.com/JIG3mol.png)

*Figure 31 - the content properties of the Markdown element*

#### Paragraph ####
The **Paragraph** element maps directly to the `<p>` HTML element and enables the user to add individual paragraphs to the page. In case you're wondering why you would want to use this element over the Html element, here's the idea: all elements have common properties such as HTML ID, HTML Class and Html Style. These property values are rendered as HTML attributes on the HTML tag output of the element. When the Html element is rendered and has a value for at least one of the three common properties, a surrounding `<div>` element is rendered onto which the common properties are rendered as attributes. Now, there may be occasions where you actually want these common property values to be rendered as attributes on a `<p>` tag directly instead of a surrounding `<div>` element. That's when you use the Paragraph element.

![Figure 32 - the content properties of the Paragraph element](http://i.imgur.com/0toAmwM.png)

*Figure 32 - the content properties of the Paragraph element*

As you can see in figure 32, the text editor is a plain textarea input control. The reason for this is that HTML editors allow users to add paragraphs to its content,while the Paragraph element itself represents the paragraph. However, the Paragraph element does support raw HTML that you can input manually, which is useful if you want to mark words as `<strong>` for example.

Figure 33 shows the rendered output of the Paragraph content as shown in figure 32.

![The rendered HTML output of the Paragraph element](http://i.imgur.com/3tQnTtv.png)

*The rendered HTML output of the Paragraph element*

As you can see, the `<p>` start and end tag have been added automatically.

#### Projection ####
The **Projection** element is similar to the **Projection Part** in that it allows the user to select a **Query** to project to a list of content items to a selected **Layout**. Layout in this context means the Query Layout and has nothing to do with the Layouts module itself.

> In order to make this element available from the toolbox, you need to enable the **Projection Element** feature.

Like the Projection Part, the Projection element has the following properties:

- **Query** - The Query to execute.
- **Items to display** - the number of items to display per page.
- **Offset** - The number of items to skip. For example, if 2 is used, the first 2 items won't be displayed.
- **Max number of items** - The maximum number of items that can be queried at once. This is used as a fail-safe when the number of items comes from a user-provided source such as the querystring.
- **Suffix** - Optionally provide a suffix to use when multiple pagers are displayed on the same page.
- **Show pager** - Whether to show a pager.

![Figure 34 - the content properties of the Projection element](http://i.imgur.com/Uphk1CW.png)

*Figure 34 - the content properties of the Projection element*

#### Text ####
The **Text** element is similar to the Html and Paragraph elements combined, in that it uses a simple textarea input control for its content input and renders that input as raw HTML. It is basically like the Html element, but without a fancy HTML editor such as TinyMce. Because of this similarity, you will probably never use the Text element since it has nothing to add in addition to the Html element, and this element may even become deprecated in a future version of Orchard. But since it's currently part of the distribution, I wanted to cover this element nonetheless for the sake of being complete.

![Figure 35 - the content properties of the Text element](http://i.imgur.com/5p4uYtw.png)

*Figure 35 - the content properties of the Text element*

And that's it for the Content category. Next, we'll check out the elements in the Media category.

### Media ###
The **Media** category contains all elements that display some form of media. Out of the box, Orchard comes with three types of Media elements.

#### Image ####
The **Image** element, quite simply, allows the user to pick a single image content item from the Media Library. When rendered on the front-end, the element renders the `<img>` HTML element. Choose the Image element when:

- You only need to display a single image per element.
- You want the common properties to be rendered as part of the `<img>` HTML tag.

![Figure 36 - the content properties of the Image element](http://i.imgur.com/d3GukFa.png)

*Figure 36 - the content properties of the Image element*

#### Media Item ####
The **Media Item** element is similar to the Image element, except for a few key differences. First of all, it allows the user to pick more than one media item. Secondly, any type of media is allowed here. Also, the user can control what display type to use when rendering the selected media items. The Media Item element is useful when:

- You want to display a list of various types of media items..
- You want to control the display type being used to render each media item.

![Figure 37 - the content properties of the Media Item element](http://i.imgur.com/tfTAqh1.png)

*Figure 37 - the content properties of the Media Item element*

#### Vector Image ####
The **Vector Image** element is similar to the Image element, but only supports vector graphics such as `.svg`. In addition to a selected media item, the Vector Image element has two additional properties: *Width* and *Height*, both expressed in number of pixels. These values will be rendered as `width` and `height` attributes on the `<img>` HTML tag.

![Figure 38 - the content properties of the Vector Image element](http://i.imgur.com/cPyX12S.png)

*Figure 38 - the content properties of the Vector Image element*

### Parts ###
The **Parts** category contains a special kind of elements, because the contents of this category depends on two conditions:

- Have their `Placeable` property set to `true`.
- Are attached to the current content item's type.

Part elements are useful when you want to allow the user to provide content for them in the "normal: way, using the editor shapes as provided by their drivers, but let the user control where they appear within the layout.

#### Placeable ####
The `Placeable` property is a new part property introduced with the Layouts module, and controls whether the content part is harvested by the `ContentPartHarvester` as an element. By default, the `Placeable` property is set to `true` for the following content parts:

- BodyPart
- CommonPart
- TagsPart
- TitlePart

![Figure 39 - The TitlePart configuration and the new Placeable property, which is checked by default](http://i.imgur.com/SYbOjDt.png)

*Figure 39 - The TitlePart configuration and the new Placeable property, which is checked by default*

This is why by default, you only see the `BodyPart`, `CommonPart` and `TitlePart` available as elements from the toolbox when creating or editing a *Page* content item. If we added the `BosyPart` to the *Page* content type, that part would become available as an element as well.

What makes content part elements interesting is that it enables the user to place various content parts onto the design surface and therefor controlling where they are displayed, or even *when*, using visibility rules. The contents of the content parts are still managed as always, using their part-specific editors. But by virtue of the part elements, the user can position their placement using the design surface.

Let's see this in action.

First, go to the default homepage content item editor. Once there, drag & drop the `TitlePart` element anywhere on the design surface (figure 40).

![Figure 40 - Drag the Title Part element to the design surface](http://i.imgur.com/rJOKIzU.png)

*Figure 40 - Drag the Title Part element to the design surface*

As soon as you drop the element, you'll see it rendered using the current value of the `TitlePart` (figure 41).

![Figure 41 - The Title Part element reflects the actual value of the title part](http://i.imgur.com/tXRe9Ik.png)

*Figure 41 - The Title Part element reflects the actual value of the title part*

Now when we go to the home page on the front-end, we'll of course see the page title twice:

![Figure 42 - The homepage now shows the TitlePart twice.](http://i.imgur.com/mwQx0x3.png)

*Figure 42 - The homepage now shows the TitlePart twice.*

That of course is probably not what you want, so how can we fix that?

The reason we see the TitlePart twice is because the **Title** module provides a `placement.info` configuration that places the `Parts_Title` shape into the `Header` zone (of the `Content` shape) for the `Detail` display type. So your initial thought may be that we should simply override that configuration in our theme and just don't render that shape. If we do that, however, the part element will not be able to render that shape either, since it relies on part shape creation as provided by the `IContentManager`, which leverages the placement system (`Placement.info` files).

So what we need to do instead is still override that shape placement, but assign it to a different zone that is not actually rendered. The convention to use here is to use the `Layout` zone name (which is not actually rendered by any Razor view). The rendered output will still be captured by the pat element, and is thus able to render the part.

The following is a sample `Placement.info` file that you would use in your theme to hide the default rendering of the `TitlePart` and take advantage of the Title Part element:

    <Placement>
        <Match DisplayType="Detail">
            <Place Parts_Title="Layout"/>
        </Match>
    </Placement>

With this change in effect, the default Title Part will no longer be rendered, while the Title Part element will be rendered wherever you placed it:

![Figure 43 - Only the Title Part element is rendered.](http://i.imgur.com/1OETtjN.png)

*Figure 43 - Only the Title Part element is rendered.*

This principle works the same for all other parts, not just the Title Part.

I think this type of element harvester really goes to show what is possible with the Layouts module, which we'll get to into much more depth when we get to Part 3 - Extensibility.

### Fields ###
The **Fields** category is similar in nature to the Parts category, the differences being that only content fields attached to the current content item's type are displayed, and that there is no *Placeable* property for content fields, which means that all contnrt fields are placeable when attached to a content type (or more accurately: to content parts of a content type).

To see how this works, we'll add a `TextField` content field to the *Page* content type and call it *Author*.

![Figure 44 - Added a TextField called Author to the Page content type.](http://i.imgur.com/KdOdm9d.png)

*Figure 44 - Added a TextField called Author to the Page content type.*

Now when we go to the home page content item editor, we'll see the *Author* text field as an additional input field as well as an additional *Fields* category in the layout editor toolbox, with a single element called *Author* (figure 45).

![Figure 45 - The Author TextField and the Author TextField Element.](http://i.imgur.com/yFyt9jk.png)

*Figure 45 - The Author TextField and the Author TextField Element.*

Users can now place the Author text field anywhere they want and hide the default rendering of the Author text field using `Placement.info`.

> You'll notice that when you initially place the Author field element onto the design surface, it will not have a value. That's because the Author text field does not initially have a value. After providing a value (e.g. "John") and then saving the page, you'll see that change reflected as part of the Author field element's value.

### Snippets ###
The **Snippets** category holds two types of elements by default: One is a generic *Shape* element, and the others are based on the existence of *specifically named Razor view files*. We'll see how that works in a moment. First, let's meet the Shape element and see how it works.

#### Shape ####
The **Shape** element is a very simple element that has just one content property: *Shape Type*. If you provide the shape type of an existing shape here, then that shape will be rendered wherever you place the Shape element. As simple as that.

Let's look at an example.

By default, the current theme is *The Theme Machine*. When we look at its *Views* folder's contents, we'll see that it has a bunch of shape templates. Most of them are created and initialized by code, but the `BadgeOfHonor.cshtml` and `Branding.cshtml` require no special initialization, which make them perfect candidates for this demo.

Let's go ahead and drag & drop the Shape element onto our design surface, and let's use the `BadgeOfHonor` shape as the Shape Type (figure 46).

![Figure 46 - Adding a new Shape element with the BadgeOfHonor shape type.](http://i.imgur.com/cgzMUdD.png)

*Figure 46 - Adding a new Shape element with the BadgeOfHonor shape type.*

I added it above the Grid of the "Welcome to Orchard" page:

![Figure 47 - The BadgeOfHonor shape element.](http://i.imgur.com/DuDtsom.png)

*Figure 47 - The BadgeOfHonor shape element.*

Now publish the page and switch to the front-end. You'll now see the `BadgeOfHonor` shape appear twice: once somewhere near the footer, which is added by the theme itself, and the other one added by us through the layout editor (figure 48).

![Figure 48 - The BadgeOfHonor shape element on the front-end.](http://i.imgur.com/IgamUq9.png)

*Figure 48 - The BadgeOfHonor shape element on the front-end.*

Besides using shapes provided by the current theme, you could just as well use shapes provided by other modules (unless those modules inject required information into that shape during creation). You could even enable the **Templates** feature, create a template, and use that as the shape type for the Shape element. This unlocks some truly interesting scenarios, since the Templates module uses Razor syntax. Being able to create snippets of Razor from the back-end and then insert them into your layouts is powerful stuff.

#### Snippet Elements ####
The second type of elements part of the Snippets category are called *snippets*. Snippets are very similar to the Shape element, but the key difference being that instead of you providing the shape type name, the Snippet element harvester provides elements based on the existence of Razor files in the current theme whose file names end in `Snippet.cshtml`.

> Before you can use Snippets, you need to enable the *Layout Snippets* feature. 

For example, a Razor file called `LogoSnippet.cshtml` in the current theme would yield an element called `Logo`, which you could then drag & drop anywhere on your design surface. Let's try it out.

First, create a new Razor file named `LogoSnippet.cshtml` in the Views folder of the current theme with the following content:

    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/21/Orchard_logo_1.svg/250px-Orchard_logo_1.svg.png" alt="Orchard Logo"/>

Go to the Page content item edit screen and notice that there's a new element in town called `Logo` (figure 49).

![Figure 49 - A new element called Logo is now available.](http://i.imgur.com/om1z2Cf.png)

*Figure 49 - A new element called Logo is now available.*

#### UI ####
> This feature is new as part of Orchard 1.10

The **UI** category currently consists of just three elements, and are kind of experimental.
The idea behind the UI elements is that they provide you with elements that make up the UI of your site. Think menus, breadcrumbs and notifications.

> The UI elements are provided by the *UI Elements* feature, so make sure to enable that one first before you can use the elements.

However, these elements will probably not be very useful before Orchard has support for *adding elements to zones and layers*. A clear example is the following: imagine you wanted to display the main menu using the Menu element. There are two major limitations with this:

1. There's no way to add elements to global zones. Although you could work around this by simply designing your theme in such a way that your entire website consists of a single Content zone and leveraging Layout Templates, this won't solve the next problem, which is:
2. The layout part is associated with content items. This means that their layouts only appear when you request the content item. This means that if you navigated to the Login screen for example, you would no longer see the main menu if you implemented that as a Menu element.

So where does that leave us?

Unless you want to display a UI element on specific pages, my advice is to not use them until Orchard unifies their Widgets and Elements story. Which I'm happy to convey is the next big thing on the list of yours truly. :)

### Summary ###
In this chapter, we got to meet all of the elements that ship with Orchard out of the box, except for one category: **Form**, which is part of the *Dynamic Forms* module and covered in another book of this series.

We went over each element in detail, and should give you a good understanding on how each one works and when to use them.
In the most scenarios, you'll most likely use the layout, content and media elements.

However, `Orchard.Layouts` would not be a true Orchard module if we couldn't extend it with out own element types. One way to extend the available elements is by creating Snippets, which are nothing but Razor files whose name end in `Snippet.cshtml`. This is useful in cases where there's no further configuration required. However, in many cases you probably do want to allow the user to provide information to configure your elements. For those scenarios, you would write your own elements and maybe even element harvesters as part of your own custom module. We will look into that in detail in Part 3 - Extensibility.
But we're not there yet.

In the next chapter, we will look at **Layout Templates**, why they are useful and how to use them.   


  