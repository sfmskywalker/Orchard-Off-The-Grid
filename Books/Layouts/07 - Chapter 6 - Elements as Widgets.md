## Elements as Widgets ##
So far we have seen how to work with elements by placing them on a design surface called the Layout Editor. But, as I eluded to before, elements can be used for more things than just being placed on a design surface. In this chapter, we'll see how we can use elements as widgets without the need to write any code.

### Why Elements as Widgets? ###
With the advent of the Layouts module, we now have the worlds of Widgets and the world of Elements, which are kind of like widgets, but not quite. Widgets are content items, while elements are something different.

The long-term goal is to unify the world of Widgets and Elements. Widgets, as we know them today, are ultimately just content items tied to a layer and a zone and are a way to place them in particular zones. The widgets screen could be replaced with a design surface that represents the various zones available to the theme. The user could then drag & drop elements into each zone (bound to the selected layer). This would simplify working with "widgets" significantly. 

Until we're there, there may be times that you need a custom widget that you also want to place on a layout design surface. Although you could write the widget code as well as the element code, it would involve a great deal of code duplication. Since it's currently not possible to add widgets to layouts, you would have to write a custom element with the same behavior as its widgets equivalent.

Fortunately there's no need for that, because the Layouts module enables you to use Elements as Widgets by using a content part called **ElementWrapperPart**.

### Element Wrapper Part ###
The `ElementWrapperPart` is a content part that you can attach to any content type, and has a single part setting called **Element Type Name**, which takes the technical name of the element to display.

> The way the `ElementWrapperPart` works is that the `ElementWrapperPartDriver` instantiates the configured element type by name and takes care of invoking the editor and display methods of that element and simply returns the created shapes. When we look at the code in Part 3, we'll see how similar elements are to content parts, which made it easy to implement `ElementWrapperPart`. 

With this part in place, we now can create *reusable building blocks in the form of elements* and use them as widgets whenever necessary. This approach is called the *Element First* approach.

### Elements First ###
Whenever you're about to write a new widget, you should consider implementing it as an element. This provides a number of advantages:

- You can place elements on the layout design surface.
- You can render them inline using tokens (which we'll cover in the next chapter).
- You can use elements as widgets without additional coding by using `ElementWrapperPart`, which means:
- No code duplication.

Having elements also means it's easy to place them into zones directly when the Widgets module gets revamped to taking advantage of the Layouts engine.

### Using Elements as Widgets ###
Before we can use an element as a widget, we need to define a Widget content type that is configured to use that element. At the very minimum, your Widget content type should have the following settings and parts:

- WidgetPart
- CommonPart (automatically added for you when creating a content type from the back-end)
- IdentityPart (required if you want your widget items to be exportable/importable)
- **ElementWrapperPart**
    - **Element Type Name** -> The type name of the element that this widget represents
- Stereotype -> Widget
- Listable -> unchecked (to prevent your widget content items from being listed on the Contents screen)
- Creatable -> unchecked (to prevent your widget content items from being creatable via the New section and Contents screen. Widgets should always be created via the Widgets screen so that they get assigned a Zone and a Layer)

The key content part is the `ElementWrapperPart`. Whatever element type name you specify here will be rendered by the widget. This works for any element type, including element blueprints.

Let's give this a try by creating a *Contact Details* widget that wraps our *Contact Details* element created in the previous chapter.

#### Demo: Creating a Widget based on an Element ####
##### Step 1 #####
Go to Content Definition and click the **Create new type** button on the top-right side of the window, and enter the following values on the **New Content Type** screen:

- Display Name -> *Contact Details*
- Content Type Id -> *ContactDetails*

And hit **Create**.

![Figure 60 - Creating the Contact Details wiget - step 1.](http://i.imgur.com/UE1tMMW.png)

*Figure 60 - Creating the Contact Details wiget - step 1.*

##### Step 2 #####
On the **Add Parts To "Contact Details"** screen, check the following parts:

- Element Wrapper
- Identity
- Widget

And hit **Save**.

##### Step 3 #####
On the **Edit Content Type - Contact Details** screen, perform the following operations:

- Uncheck the following fields:
    - Creatable
    - Listable
- Specify *Widget* as the Stereotype
- Expand the *Element Wrapper* part settings and provide the following element type name: *ContactDetails* (see the previous chapter for details on creating this element).

And hit **Save** to save your changes.

![Figure 61 - The Contact Details widget settings.](http://i.imgur.com/waENQ2A.png)

*Figure 61 - The Contact Details widget settings.*

##### Step 4 #####
Now that we created a new widget type, we can create and add Contact Details widgets to any of our zones and layers.

- Go to Widgets and click the **Add** button on the **AsideSecond** zone.
- On the **Choose A Widget** screen, select the **Contact Details** widget.
- On the **Add Widget** screen, use the following value for the **Title** field: *My Contact Details*.

And hit **Publish Now**.

##### Step 5 #####
Navigate to the front-end and, low and behold, there's our custom widget that is in fact powered by our *Contact Details* element.

![Figure 62 - The Contact Details widget as it appears on the front-end in the AsideSecond zone.](http://i.imgur.com/zoF7jGb.png)

*Figure 62 - The Contact Details widget as it appears on the front-end in the AsideSecond zone.* 

### Out of box Widgets based on Elements ###
As it turns out, being able to use elements as widget is quite useful. For example, the Layouts module comes with a *Content Item* element as we've seen in chapter 3: Meet the Elements. Orchard doesn't come with a content picker widget, but with the `ElementWrapperPart` it's very simple to create one.

In fact, the Layouts module does exactly that for us! When Layouts is enabled, it creates the following additional widgets based on elements:

- Text Widget
- Media Widget
- Content Widget

The Text Widget, like the Text element, aren't that useful, so we'll skip that one.
The Media Widget let's the user select one or more Media Items to be displayed.
The Content Widget let's the user select one or more Content Items to be displayed.

This enables scenarios where you want to display a certain piece of content multiple times on various locations on your website, without the need to duplicate that content, which of course greatly improves the site's maintenance.
Granted, you could achieve this by simply placing an Html Widget somewhere and associate it with the Default layer of course, but as soon as you want to display the same content twice in two different zones, you'll need to duplicate that content.

### Summary ###
Elements are versatile things. In this chapter, we learned about the `ElementWrapperPart` and how to use it to turn elements into widgets.

In the next chapter, we'll learn about yet another way to display elements using the *Element token*. 
 

     
 