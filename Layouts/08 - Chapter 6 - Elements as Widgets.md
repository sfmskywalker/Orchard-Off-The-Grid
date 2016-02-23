## Elements as Widgets
So far we have seen how to work with elements and the layout editor on content items that have the Layout Part. But elements can be used for more things than that. In this chapter, we'll see how we can use elements as widgets without writing any code.

### Why Elements as Widgets
With the advent of the Orchard.Layouts module, we now have the worlds of Widgets and the world of Elements, which are kind of like widgets, but not quite. Widgets are content items, while elements are a different kind of entity.

The long-term goal is to unify the two worlds. Widgets, as we know them today, are ultimately just content items tied to a **layer** and a **zone**. Now, instead of adding widgets directly to layers and zones, we could conceive of a way where we add **elements** to layers and zones. This will have the advantage of not having to write elements and widgets, but just elements.

Now, elements are not content items and do not support composition like content items do, so you may be wondering if we wouldn't lose something important: Widgets are content items, which means we can extend widgets with content parts. The answer is that we will lose nothing. On the contrary, there is a lot to gain in terms of ease of use and flexibility. Widgets as we know them would be replaced by content item types harvested as elements. These elements manage the creation and destruction of content items. And that is just one type of element. Many more elements are available, such as the Content Item element which enables the user to select existing content items. The Widgets UI would also be more intuitive to use.

But anyway, we're not there yet. And there may be times that you need an element that you also want to be able to use as a widget, and the other way around.

As of Orchard 1.10, there are two ways to handle this situation without having to write duplicate code:

1. Write an element once and configure a new widget content type with the **Element Wrapper Part** to use the element.
2. Write a content part once and configure a new widget content type.

If you're using 1.9, then option 1 is the only option available, and is the topic of this chapter.

> To use option 2, all you need to do is enable the **Widget Elements** feature.

### Element Wrapper Part
The **Element Wrapper Part** is a content part that has a single setting called **Element Type Name**, which takes the technical name of the element to display.

The way the Element Wrapper Part works is that it instantiates the configured element type by name and takes care of invoking the editor and display methods of that element and simply returns the created shapes. When we look at the code in Part 3, we'll see that elements are quite similar to the way content parts work.

### Elements First
Whenever you're about to write a new widget, consider implementing it as an element. This provides a number of advantages:

- You can place elements on the layout design surface.
- You can render them inline using tokens (which we'll cover in the next chapter).
- You can use elements as widgets without additional coding by using `ElementWrapperPart`, which means:
- No code duplication.

### Using Elements as Widgets
Before you can use an element as a widget, you need to define a Widget content type that is configured to use that element. At the very minimum, your Widget content type should have the following settings and parts:

- WidgetPart
- CommonPart (automatically added for you when creating a content type from the back-end)
- IdentityPart (required if you want your widget items to be exportable/importable)
- **ElementWrapperPart**
    - **Element Type Name** -> The type name of the element that this widget represents
- Stereotype -> Widget
- Listable -> unchecked (to prevent your widget content items from being listed on the Contents screen)
- Creatable -> unchecked (to prevent your widget content items from being creatable via the New section and Contents screen. Widgets should always be created via the Widgets screen so that they get assigned a Zone and a Layer)

The key content part here of course is the **Element Wrapper Part**.

#### Trying it out: Creating a Widget based on an Element
So, let's see how it works.

In this example, we'll see how the **Contact Details** element created in chapter 5 can be reused as a widget.

##### Step 1
Go to **Content Definition** and click the **Create new type** button on the top-right side of the window, and enter the following values on the **New Content Type** screen:

- Display Name -> *Contact Details*
- Content Type Id -> *ContactDetails*

And hit **Create**.

![Figure 6.1 - Creating the Contact Details widget - step 1.](./figures/fig-6-1-creating-contact-details-widget-step1.png)

*Figure 6.1 - Creating the Contact Details widget - step 1.*

##### Step 2
On the **Add Parts To "Contact Details"** screen, include the following parts:

- Element Wrapper
- Identity
- Widget

And hit **Save**.

##### Step 3
On the **Edit Content Type - Contact Details** screen, do the following:

- Uncheck the following fields:
    - Creatable
    - Listable
- Specify **Widget** as the Stereotype.
- Expand the **Element Wrapper** part settings and provide the following *element type name*: **ContactDetails** (see the previous chapter for details on creating this element).

And hit **Save** to save your changes.

![Figure 6.2 - The Contact Details widget settings.](./figures/fig-6-2-contact-details-widget-settings.png)

*Figure 6.2 - The Contact Details widget settings.*

##### Step 4
With that in place, we can add **Contact Detail** widgets to any zone and layer.

- Go to Widgets and click the **Add** button on the **AsideSecond** zone.
- On the **Choose A Widget** screen, select the **Contact Details** widget.
- On the **Add Widget** screen, use the following value for the **Title** field: *My Contact Details*.

And hit **Publish Now**.

##### Step 5
Navigate to the front-end and, low and behold, there's the **Contact Details widget** that is in fact powered by the **Contact Details element**.

![Figure 6.3 - The Contact Details widget as it appears on the front-end in the AsideSecond zone.](./figures/fig-6-3-contact-details-widget-frontend.png)

*Figure 6.3 - The Contact Details widget as it appears on the front-end in the AsideSecond zone.* 

### Existing Widgets based on Elements
Being able to use elements as widget can be quite useful. For example, the Layouts module comes with a **Content Item** element as we've seen in chapter 3. Although Orchard doesn't come with a content picker widget, it's very simple to create one with the **Element Wrapper Part**.

In fact, when the Layouts feature is enabled, the following additional widget types based on elements are added to the system:

- Text Widget
- Media Widget
- Content Widget

In all fairness, the Text Widget (as is the case with the Text element), isn't all that useful. We already have the Html Widget.
The Media Widget lets the user select one or more Media Items to be displayed.
The Content Widget lets the user select one or more Content Items to be displayed.

### Summary ###
Elements can be used in many different ways other than being added to a canvas. In this chapter, we learned about the **Element Wrapper Part** and how to use it to turn elements into widgets. 
 

     
 