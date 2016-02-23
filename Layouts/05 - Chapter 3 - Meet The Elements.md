## Meet the Elements
In this chapter, we'll go over all of the available elements in the default Orchard distribution. Most elements should be self-explanatory to use, but I think that some of them could use a little bit of a background to get a decent understanding on how to use them.

### Element Categories
By default, elements are categorized using the following categories:

* Layout
* Content
* Media
* Parts
* Fields
* Snippets
* UI

Third party modules can add additional categories, or associate additional elements with existing ones.
The following is the complete list of available elements when all features (except the features from **Orchard.DynamicForms**) is enabled:

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
    * **Markdown** *(requires the `Markdown Element` feature to be enabled)*
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
    * *Any content field attached to the content type will be made available as an element*
* Snippets
    * **Shape**
    * *Any Razor file ending in `Snippet.cshtml` in the current theme or any module will be made available as an element*
* UI (New as of Orchard 1.10)
    * **Breadcrumbs** *(requires the `UI Elements` feature to be enabled)*
    * **Menu** *(requires the `UI Elements` feature to be enabled)*
    * **Notifications** *(requires the `UI Elements` feature to be enabled)*
* Widgets (New as of Orchard 1.10)
	* All widgets are available as elements

> The **Orchard.DynamicForms** module is another module introduced with Orchard 1.9. This module provides its own set of elements. Please checkout **Orchard Off The Grid - Dynamic Forms** for everything you ever wanted to know about the Dynamic Forms module.

Let's go over each category and elements one by one.

#### Layout
Elements in this category are typically container elements that layout their child elements in a certain way. Certain container elements only support certain child element types. For example, the **Grid** element can only contain **Row** elements, which in turn can only contain **Column** elements.

The following elements fall under the Layout category:

##### Grid
The **Grid** element is a container element that can hold only one type of child elements: **Row**.
We use the Grid element whenever we want to create a layout of elements. Grids can contain rows, and rows in turn can contain columns. Although the Layouts module is named that way because it enables users to create layouts, it is because of the Grid, Row and Column elements that we are able to create layouts of elements.

The Grid element has no specialized properties, but does have common element properties (**HTML ID**, **CSS Classes**, **CSS Styles** and **Visibility Rule**).

![Figure 3.1 - The Grid element](./figures/fig-3-1-grid.png)

*Figure 3.1 - The Grid element*

When you add a new Grid element to the design surface, it will not contain any Row elements initially; you need to drag and drop Row elements to the Grid yourself.

##### Row
The **Row** element, like the Grid element, is a container element. However, this element can only contain **Column** elements as its children, and the Row itself can only be a child of a Grid element.

The **Row** element is represented in the toolbox as 7 pre-configured Row elements. The first Row toolbox item will add a single Row element with 1 Column element, the second toolbox item will add a single row with 2 columns, and so forth.

![Figure 3.2 - Pre-configured Row elements](./figures/fig-3-2-toolbox-layout-rows.png)

*Figure 3.2 - Pre-configured Row elements*

The Row element has one specific toolbar command called **Distribute columns evenly**.
As its name implies, this command will evenly distribute the width of the columns of the row based on a *maximum size of 12* columns. For example, if a Row has 4 columns with varying sizes, and the maximum size is 12, each column will be re-sized to be exactly 3 units in size.

The Row element has no specialized properties, but does have common element properties (**HTML ID**, **CSS Classes**, **CSS Styles** and **Visibility Rule**).

![Figure 3.3 - The Row element](./figures/fig-3-3-row.png)

*Figure 3.3 - The Row element*

##### Column
Unlike the Grid and Row elements, the **Column** element can hold any type of child elements, except for Row and Column elements. Columns themselves can only be contained by Row elements.
A column has two specialized properties: **Width** and **Offset**. Together, these values make up the **total size** of the column. **Width**, **Offset** and **Total Size** are expressed in a unit called **cell size**. What this means is that the total cell size occupied by column elements in a row cannot exceed 12. For example, if a row has a single column, that column's width cannot be greater than 12. If a row has two columns, the sum of the two column's widths cannot exceed 12. One column could be 4 while another could be 8, making a total size of 12. Similarly, if a column has an offset of 1 and a width of 4, its size is 5, which means that the second column cannot be more than 7 columns wide.

###### Adding Columns
Once you have a Row element, there are various ways to add Columns to it. If the row doesn't contain any Column, you need to add a Column element to it. If the Row element contains at least one Column, you can split that column into two.

###### Offsetting
Another feature of the Column element is called *offsetting*. Offsetting simply means that you provide an offset value for a column. By default, the offset for any column is 0. Increasing the offset increases the overall size of the column. You typically use an offset if you want the contents of a column to appear more to the right. You can do this either by dragging the left border to the right, by using the little toolbar icon called *Increase column offset*, or by using the keyboard shortcut Alt+Right:

![Figure 3.4 - A Row with two columns, the first column having an offset of 1, a size of 7, and the second column having a size of 4](./figures/fig-3-4-row-two-cols.png)

*Figure 3.4 - A Row with two columns, the first column having an offset of 1, a size of 7, and the second column having a size of 4*

Combined with Row and Grid, the Column element is the key ingredient for creating layouts.

![Figure 3.5 - The Column element](./figures/fig-3-5-column.png)

*Figure 3.5 - The Column element*

##### Canvas
The **Canvas** element is the last element in this category. Like the Column element, the Canvas can contain any type of element, except for Row and Column elements. Unlike the Column element, the Canvas element can be added to any other container element, except for Grid and Row, since they have an exclusive white-list of allowed children.

> Whenever you create a new layout, that layout always start with a root element of type Canvas.

The Canvas element is great if you ever need a generic type of container element. Although in most cases you probably won't need it since the Column element already is a container, there are occasions where a Canvas is very useful. The prime example being when working with **Layout Templates**. We'll look at Layout Templates into much more detail in the next chapter, but basically a layout template enables the reuse of pre-created layouts. When a layout template is applied to a layout, the elements from the template are *sealed*, which means the user cannot make any modifications to these elements. If a sealed *container element is empty* however, it will accept elements as its children. But if a sealed container element contains at least one element, no elements are allowed to be added to that container. To work around that, you need to add an empty container element that acts as a placeholder. The Canvas is quite suitable for that purpose.

![Figure 3.6 - The Canvas element](./figures/fig-3-6-canvas.png)

*Figure 3.6 - The Canvas element*

#### Content
The **Content** category contains all elements that have to do with, you guessed it, contents. Content range from small visual components such as the Break element (which directly maps to the `<hr>` HTML tag) to HTML text and images.

##### Break
The **Break** element is one of the simplest elements available. It has no specialized properties, but does have common element properties (**HTML ID**, **CSS Classes**, **CSS Styles** and **Visibility Rule**). The element maps directly to the `<hr>` HTML element.

##### Content Item
The **Content Item** element is very similar to the **Content Picker Field** and enables the user to select one or more content items to render inline on the canvas.
When adding or editing a Content Item element, the user is presented with a dialog window displaying the content properties of the element (see figure 3.7). The element has two specialized properties:

- A list of content items to render;
- The **DisplayType** to use when rendering the selected content items. 

![Figure 3.7 - The content properties of the Content Item element](./figures/fig-3-7-content-item-element-properties.png)

*Figure 3.7 - The properties of the Content Item element*

Selecting existing content items and rendering them using custom display types enables you to place content anywhere you like in any display type you want. This enables scenarios where you for example want to render the same content item at various locations.  

##### Heading
The **Heading** element maps directly to the `<h1>` to `<h6>` HTML elements and have the following content properties:

- Level
- Text

![Figure 3.8 - The content properties of the Heading element](./figures/fig-3-8-heading-properties)

*Figure 3.8 - The properties of the Heading element*

The **Level** indicates the size of the heading and ranges from **1** to **6**. For example, if you specify level 3, the `<h3>` tag will be rendered. The following is an example of HTML output when specifying level 6 and the text *"Hello Layouts!"*:

    <h6>Hello Layouts!</h6>

##### Html
The **Html** element is probably the most commonly used one when it comes to placing content onto the canvas. It has a single property called **HTML**, which stores the HTML markup. Use this element whenever you want to display textual content anywhere on the canvas.

![Figure 3.9 - the content properties of the Html element](./figures/fig-3-9-html-properties.png)

*Figure 3.9 - the content properties of the Html element*

Although the HTML editor used is **TinyMCE** by default, this can be changed by enabling other features that provide another editor for the **html** flavor. For example, if you enable the **CKEditor** feature, that's the editor you'll see when editing Html elements.

##### Markdown
The **Markdown** element let's the user use Markdown syntax which gets transformed into HTML when being rendered on the front-end.

> You'll need to enable the **Markdown Element** feature before this element becomes available.

![Figure 3.10 - the properties of the Markdown element](./figures/fig-3-10-markdown-properties.png)

*Figure 3.10 - the properties of the Markdown element*

##### Paragraph
The **Paragraph** element maps directly to the `<p>` HTML element and enables the user to add individual paragraphs to the page. You may be wondering why you would want to use this element over the Html element. Well, here's the idea: all elements have common properties such as **HTML ID**, **HTML Class** and **Html Style**. These property values are rendered as HTML attributes on the HTML tag output of the element. When the Html element is rendered and has a value for at least one of the three common properties, a surrounding `<div>` element is rendered onto which the common properties are rendered as attributes. However, there may be occasions where you actually want these common property values to be rendered as attributes on a `<p>` tag directly instead of a surrounding `<div>` element. That's when you use the Paragraph element.

![Figure 3.11 - the properties of the Paragraph element](./figures/fig-3-11-paragraph-properties.png)

*Figure 3.11 - the properties of the Paragraph element*

##### Projection
The **Projection** element is very similar to the **Projection Part** (which is attached to the **Projection** content item and **Projection Widget**) in that it allows the user to select a **Query** to project to a list of content items to a selected **Layout**. "Layout" in this context means the "Query Layout".

> In order to make this element available from the toolbox, you need to enable the **Projection Element** feature.

Like the Projection Part, the Projection element has the following properties:

- **Query** - The Query to execute.
- **Items to display** - the number of items to display per page.
- **Offset** - The number of items to skip. For example, if 2 is used, the first 2 items won't be displayed.
- **Max number of items** - The maximum number of items that can be queried at once. This is used as a fail-safe when the number of items comes from a user-provided source such as the query string.
- **Suffix** - Optionally provide a suffix to use when multiple pagers are displayed on the same page.
- **Show pager** - Whether to show a pager in case the returned number of items is greater than the configured page size.

![Figure 3.12 - the properties of the Projection element](./figures/fig-3-12-projection-properties.png)

*Figure 3.12 - the content properties of the Projection element*

##### Text
The **Text** element provides a simple textarea input control for its content input and renders that input as raw HTML.

![Figure 3.13 - the properties of the Text element](./figures/fig-3-13-text-properties.png)

*Figure 3.13 - the content properties of the Text element*

#### Media
The **Media** category contains all elements that display some form of media.

##### Image
The **Image** element allows the user to pick a single image content item from the Media Library. When rendered on the front-end, the element renders the `<img>` HTML element. Use the Image element when:

- You only need to display a single image per element.
- You want the common properties to be rendered as part of the `<img>` HTML tag.

![Figure 3.14 - the properties of the Image element](./figures/fig-3-14-image-properties.png)

*Figure 3.14 - the properties of the Image element*

##### Media Item
The **Media Item** element allows the user to pick more than one media item. The user can control what display type to use when rendering the selected media items. Use the Media Item element when:

- You want to display a list of various types of media items.
- You want to control the display type being used to render each media item.

![Figure 3.15 - the properties of the Media Item element](./figures/fig-3-15-media-item-properties.png)

*Figure 3.15 - the properties of the Media Item element*

##### Vector Image
The **Vector Image** element is similar to the Image element, but only supports vector graphics such as `.svg`. In addition to a selected media item, the Vector Image element has two additional properties: **Width** and **Height**, both expressed in number of pixels. These values will be rendered as `width` and `height` attributes on the `<img>` HTML tag.

![Figure 3.16 - the properties of the Vector Image element](./figures/fig-3-16-vector-image-properties.png)

*Figure 3.16 - the properties of the Vector Image element*

#### Parts
The **Parts** category contains elements for parts that:

- Have their `Placeable` property set to `true`.
- Are attached to the current content item's type.

Part elements are useful when you want to allow the user to provide content for them in the standard way, using the editor shapes as provided by their drivers, but let the user control where they appear within the layout via elements.

##### Placeable Parts
The **Placeable** property is a new part property that controls whether the content part is harvested by the `ContentPartHarvester` as an element. By default, the Placeable property is set to `true` for the following content parts:

- BodyPart
- CommonPart
- TagsPart
- TitlePart

![Figure 3.17 - The TitlePart configuration and the new Placeable property, which is checked by default](./figures/fig-3-17-content-part-placeable-property.png)

*Figure 3.17 - The TitlePart configuration and the new Placeable property, which is checked by default*

Content part elements enable the user to place various content parts onto the canvas and therefor to control where they are displayed. The contents of the content parts are managed as usual using their part-specific editors.

One side effect of using parts as elements you may not expect, is that in addition to the element being displayed, the part itself is displayed as well, ultimately causing the part to be displayed twice. Although this is by design, it is easy to prevent the original part from being displayed by modifying the Placement.info file.

The following demonstrates how by taking the Title Part element as an example, where the scenario is that we want to enable the user to edit the Page title via the Title Part, but control where it's placed via the layout editor.   

First, add the Title Part element to the canvas. 
![Figure 3.18 - Add the Title Part element to the canvas](./figures/fig-3-18-drag-title-part.png)

*Figure 3.18 - Add the Title Part element to the canvas*

As soon as you drop the element, you'll see it rendered using the current value of the Title Part (figure 3.19).

![Figure 3.19 - The Title Part element reflects the actual value of the title part](./figures/fig-3-19-drop-title-part.png)

*Figure 3.19 - The Title Part element reflects the actual value of the title part*

Now when we go to the home page on the front-end, we'll see the page title twice:

![Figure 3.20 - The homepage shows the Title Part twice.](./figures/fig-3-20-homepage-double-title.png)

*Figure 3.20 - The homepage now shows the Title Part twice.*

The reason we see the Title Part twice is because the **Title** module provides a **placement.info** configuration that places the `Parts_Title` shape into the local *Header* zone of the *Content* shape for the *Detail* display type. You may be inclined to simply override that configuration in the theme and prevent the shape from being rendered. But if you do that, the part element will also not being rendered, since it relies on the part driver to create the shape in the first place.

So what we need to do instead is to assign the part shape to a different (non-existing) zone that is not actually rendered by any template, so that the part driver still creates the shape to be displayed by the element. The convention is to use *Layout* as the zone name (which is not actually rendered by any Razor view). The rendered output will still be captured by the part element, and so is able to render the part shape.

The following is a sample Placement.info file that you would use in your theme to hide the default rendering of the Title Part and take advantage of the Title Part element:

    <Placement>
        <Match DisplayType="Detail">
            <Place Parts_Title="Layout"/>
        </Match>
    </Placement>

With this change in effect, the default Title Part will no longer be rendered, while the Title Part element will be rendered wherever you placed it.

![Figure 3.21 - Only the Title Part element is rendered.](./figures/fig-3-21-homepage-single-title.png)

*Figure 3.21 - Only the Title Part element is rendered.*

#### Fields
The **Fields** category is similar to the way the Part elements work, but there are a few differences:

- Only content fields attached to the current content item's type are displayed as elements
- There is no **Placeable** property for content fields, which means that all content fields are placeable when attached to a content type.

To see how this works, let's add a **TextField** content field to the **Page** content type and call it **Author**.

![Figure 3.22 - Add a Text Field called Author to the Page content type.](./figures/fig-3-22-page-author-textfield.png)

*Figure 44 - Add a Text Field called Author to the Page content type.*

Now when we go to the home page content item editor, we'll see the **Author** text field as an additional input field as well as an additional **Fields** category in the layout editor toolbox, with a single element called **Author** (figure 45).

![Figure 3.23 - The Author Text Field and the Author Text Field Element.](./figures/fig-3-23-page-author-textfield-element.png)

*Figure 3.23 - The Author Text Field and the Author Text Field Element.*

Users can now place the **Author** text field anywhere they want and hide the default rendering of the Author text field using **Placement.info**.

> You'll notice that when you initially place the Author field element onto the design surface, it will not have a value. That's because the Author text field does not initially have a value. After providing a value (e.g. "John") and then saving the page, you'll see that change reflected as part of the Author field element's value.

#### Snippets
The **Snippets** category holds two types of elements by default: One is a generic **Shape** element, and the others are based on the existence of *specifically named Razor view files*.

##### Shape
The **Shape** element is a very simple element that has just one content property: **Shape Type**. If you provide the shape type of an existing shape here, then that shape will be rendered wherever you place the Shape element.

Let's look at an example.

By default, the current theme is *The Theme Machine*. When we look at its *Views* folder's contents, we'll see that it has a bunch of shape templates. Most of them are created and initialized by code, but the **BadgeOfHonor.cshtml** and **Branding.cshtml** require no special initialization, which make them perfect candidates for this example.

Add the **Shape** element onto the canvas, and use the **BadgeOfHonor** shape as the **Shape Type** (figure 3.24).

![Figure 3.24 - Adding a new Shape element with the BadgeOfHonor shape type.](./figures/fig-3-24-shape-properties.png)

*Figure 3.24 - Adding a new Shape element with the BadgeOfHonor shape type.*

Now publish the page and switch to the front-end. You'll now see the **BadgeOfHonor** shape appear twice: once somewhere near the footer, which is added by the theme itself, and the other one added by us through the layout editor (figure 3.25).

![Figure 3.25 - The BadgeOfHonor shape element on the front-end.](./figures/fig-3-25-shape-badgeofhonor-frontend.png)

*Figure 3.25 - The BadgeOfHonor shape element on the front-end.*

Besides using shapes provided by the current theme, you can use shapes provided by other modules as well (unless those modules inject required information into that shape during creation).

You could even enable the **Templates** feature, create a template, and use that as the shape type for the Shape element. This unlocks some really interesting scenarios, since the **Templates** module uses Razor syntax.

##### Snippet Elements
The second type of elements part of the **Snippets** category are called **snippets**. Snippets are very similar to the Shape element, but the key difference is that instead of you providing the shape type name, the **Snippet element harvester** provides elements based on the existence of Razor files in the current theme whose file names end in **Snippet.cshtml**.

> Snippets are provided by the *Layout Snippets* feature. 

For example, a Razor file called **LogoSnippet.cshtml** in the **Views** folder of the current theme (or any module) would yield an element called **Logo**.

![Figure 3.26 - A new element called Logo is now available.](./fig-3-26-logo-snippet-element-toolbox.png)

*Figure 3.26 - A new element called Logo is now available.*

Snippets are great for theme and module developers alike, as they provide a quick way of providing elements without having to write much code. What's more, Snippets can be **parameterized**, which means that you, as a developer, can add **named fields** to your snippet templates that will be replaced at runtime with **user provided values**. We will see more about this in **Chapter 11**.

#### UI
> This feature is new as part of Orchard 1.10

The **UI** category currently consists of just three elements.
The idea behind the UI elements is that they provide you with elements that make up the UI of your site. Think menus, breadcrumbs and notifications.

> The UI elements are provided by the *UI Elements* feature.

As a caveat however, these elements will not be very useful before Orchard has support for *adding elements to zones and layers*. Imagine you wanted to display the main menu using the **Menu element**. There are two major limitations with this:

1. There's no way to add elements to global zones. Although you could work around this by simply designing your theme in such a way that your entire website consists of a single Content zone and leveraging Layout Templates, this won't solve the next problem, which is:
2. The layout part is associated with content items. This means that their layouts only appear when you request the content item. This means that if you navigated to the Login screen for example, you would no longer see the main menu if you implemented that as a Menu element.

So where does that leave us?

Unless you want to display a UI element on specific pages, my advice is to not use them until Orchard unifies its Widgets and Elements story.

#### Widgets
This category contains all widgets as elements and enables the user to add widgets to layouts. When you place a widget element onto a layout, the widget element harvester will create an actual widget content item for you and render that one on the front end. The created widget is not linked with any zone or layer, since it is linked with the content item containing the layout. If you delete the content item or the widget element, the widget itself is deleted as well, since it's managed by the element.

### Summary ###
In this chapter, we got to meet all of the elements that ship with Orchard out of the box, except for one category: **Form**, which is part of the *Dynamic Forms* module and outside the scope of this book.

We went over each element in detail, and should give you a good understanding on how each one works and when to use them.

The set of elements can be extended by custom **element harvesters**, which are responsible for providing elements to the system. Module developers can create custom harvesters, into which we will look in detail in *Part 3 - Extensibility*.   


  