## Element Tokens
Tokens provide a way to insert variables into content. These variables are processed at runtime, and it is up to the token providers to provide the result. Tokens are provided by modules. A few examples are:

- *Content.Author* - Renders the author name of the content item.
- *Content.DisplayUrl* - Renders the display URL of the content item.
- *Content.DisplayUrl.Absolute* - Renders the fully qualified URL of the content item.
- *Request.QueryString:** - Renders the specified querystring value, e.g. {Request.QueryString:MyQueryKey}.
- *Site.SiteName* - Renders the site name as configured in the *Settings* section.

There are various places where you can leverage tokens. For example, **Orchard.Autoroute** uses tokens for its configurable route patterns, and many **Orchard.Workflows** activities support tokens as configuration values.

### The Element.Display Token
The Layouts module also provides a token to the system, namely **Element.Display:<SomeElementTypeName>**.
This token is bound to the **Element Tokens** feature, so you'll have to enable that before you can use the this token.

What the **Element.Display** token does is simply rendering the element of the type that is provided as its argument. For example, the following will render the **ContactDetails** element created in chapter 5:

    #{Element.Display:ContactDetails} 

*Snippet 7.1 - Rendering the ContactDetails element using a token.*

This token is especially useful when you want to render elements in HTML contents. For example, let's say you have a content type that uses the **BodyPart**. Users can input some HTML, and insert visual elements using the **element.Display** token.

Although the **Element.Display** token does not support additional arguments to provide values for the element's properties, you can create element blueprints (pre-configured elements) and for example render a **Projection** element. If an element does not require configuration, then you can use it directly without creating a blueprint.

#### Trying it out: Using Element.Display
In this example, I'll show you how the **Element.Display** token works.

##### Step 1
Make sure the **Element Tokens** feature is enabled.

##### Step 2
Create an element blueprint called **ContactDetails** as shown in chapter 5.

##### Step 3
Go to the **Welcome to Orchard** content item and edit the first **Html** element containing the introductory text, and insert the **Element.Display:ContactDetails** token anywhere between two sentences or paragraphs as follows:

![Figure 7.1 - Enter the Element.Display token somewhere in the Html element's contents.](./figures/fig-7-1-element-display-token-applied.png)

*Figure 7.1 - Enter the Element.Display token somewhere in the Html element's contents.*

And hit **Publish Now**.

> Notice that I'm using the new token syntax where tokens start with the hash-tag symbol (#). Omitting this will cause your tokens not to be executed.

##### Step 4
Go to the homepage of your site and notice how the token has been replaced with the actual **ContactDetails** element output.

![Figure 7.2 - The Contact Details element as rendered on the front-end by the Element.Display token.](./figures/fig-7-2-element-token-contact-details-frontend.png)

*Figure 7.2 - The Contact Details element as rendered on the front-end by the Element.Display token.*

This works pretty much anywhere you can use tokens, including the **Body Part** and **Email** field of the **Email Workflows Activity**.

### Summary
In this chapter, we've seen yet another way to display elements, this time using the **Element.Display** token.
Although this token does not support the specified element to be configured, we can create element blueprints and render them instead. The intended use of the token is to allow elements to be rendered anywhere, such as inside bodies of text.