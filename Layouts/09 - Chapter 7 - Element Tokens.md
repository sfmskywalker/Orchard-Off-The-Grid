## Element Tokens
Orchard has support for Tokens since [version 1.3](https://orchard.codeplex.com/releases/view/69668), and essentially provides a way to insert variables into content. Out of the box, Orchard provides many tokens. A few examples are:

- *Content.Author* - Renders the author name of the content item.
- *Content.DisplayUrl* - Renders the display URL of the content item.
- *Content.DisplayUrl.Absolute* - Renders the fully qualified URL of the content item.
- *Request.QueryString:** - Renders the specified querystring value, e.g. {Request.QueryString:MyQueryKey}.
- *Site.SiteName* - Renders the site name as configured in the *Settings* section.

There are various places where you can specify tokens. For example, the `Orchard.Autoroute` module relies on `Orchard.Tokens` for its configurable route patterns, and many `Orchard.Workflows` activities support tokens as configuration values. Other places where one may use tokens are `BodyPart` and `TextField` when you input content.

### The Element.Display Token
The **Orchard.Layouts** module adds another token to the system: **Element.Display:[SomeElementTypeName]*.
This token is provided by the `Element Tokens` feature provided by the Layouts module, which means you'll have to enable that feature before you can use the **Element.Display** token.

The following snippet demonstrates how to use the *Element.Display* token to render the **ContactDetails* element created in chapter 5:

    #{Element.Display:ContactDetails} 

*Snippet 7.1 - Rendering the ContactDetails element using a token.*

The **Element.Display** token does not support additional arguments to provide values for the element's properties, so simply rendering a **Projection** element for example would be useless, since there's no way for us to specify the query to render.

However, we can pre-configure elements and render them instead as we saw in chapter 5.

Let's see the token in action.

#### Trying it out: Using Element.Display
##### Step 1
Make sure the `Element Tokens` feature is enabled.

##### Step 2
Create a pre-configured element called *ContactDetails* as shown in chapter 5.

##### Step 3
Go to the *Welcome to Orchard* page edit screen and edit the first Html element containing the introductory text, and enter the *Element.Display* token anywhere between two sentences or paragraphs.

![Figure 63 - Enter the Element.Display token somewhere in the Html element's contents.](http://i.imgur.com/sjGQ7zq.png)

*Figure 63 - Enter the Element.Display token somewhere in the Html element's contents.*

And hit *Publish Now*.

> Notice that I'm using the new token syntax where tokens start with the hashtag symbol (#). Omitting this will cause your tokens not to be executed.

##### Step 4
Go to the homepage of your site and notice how the token has been replaced with the actual *ContactDetails* element.

![Figure 64 - The Contact Details element as rendered on the front-end by the Element.Display token.](http://i.imgur.com/cO1nip2.png)

*Figure 64 - The Contact Details element as rendered on the front-end by the Element.Display token.*

This works pretty anywhere you can use tokens, including the *BodyPart* and *Email* field of the *Email Workflows Activity*.

And since we used an element blueprint in this demo, we could change any of its properties, and see this change being reflected anywhere we render this element, be it used on a layout, as a widget, or as a token.

Powerful stuff if you ask me!

### Summary
In this chapter, we've seen yet another way to display elements, this time using the *Element.Display* token.
Although this token does not allow the element to be configured, we can create pre-configured elements and render them instead. This provides for a much nicer experience anyway, sine the user doesn't have to be bothered with cumbersome token syntax, especially when the same element and values are to be rendered multiple times.

In the next chapter, we'll look at a feature that you'll probably find very similar to Widgets' Layer Rules: *Element Rules*.