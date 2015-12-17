##Content Items##
One of Orchard's powerful features is its flexible type composition system.
This system enables you to compose your own custom **content types**, where each content type has its own set of properties and behavior.

What this means in practice is that this system enables you to create various *types of content* while at the same time treating content in a homogeneous manner. Examples of content items are *pages*, *blogs*, *blog posts*, *comments* and even the *user*, *site* and *menu items* are content items.

You can easily modify existing content types or create new ones, simply by adding and removing content parts. Working with content types and content parts is not unlike playing with lego blocks.

###Anatomy of a Content Item###
A content item consists of smaller parts, called ****content parts****. Each content part is an atom of a content item.
From a structural point of view, a content item looks like this:

- Content Item
	- Content Type
	- Parts
		- Part 1
			- Fields
				- Field 1
				- Field 2
				- Etc.
		- Part 2
			- Fields
				- Field 1
				- Field 2
				- Etc.
		- Etc.

As you can see, a content item essentially consists of a reference to its content type and a set of content parts. The set of content parts attached to a content item is determined by its **content type**, which acts as a "blueprint" or "class" of content items.

> In other words, a content item is an instance of a content type.

This means that whenever you create a new content item of a particular content type, a new content item is created and its list of content parts is populated based on the list of content parts of the content type.

The following list depicts the structure of a `Page` content item: 
    
- Content Item
	- Content Type ("Page")
	- Parts
		- TitlePart
		- AutoroutePart
		- LayoutPart
		- TagsPart
		- MenuPart
		- CommonPart

Each part is responsible for optionally providing the following things:

- Creating its UI when editing the content item and handling postback.
- Creating its UI when viewing the content item.
- Exporting its information when the content item is being exported.
- Importing information when the content item is being imported.

Figure 2.1 shows the edit screen of a Page content item.
![](http://i.imgur.com/XAaeuQh.png)
> Figure 2.1 - Each content part is responsible for rendering its editor shape.

Figure 2.2 shows the front-end screen of the same Page content item.
![](http://i.imgur.com/lrReyRg.png)
> Figure 2.2 - Each content part is responsible for rendering its display shape.

Notice that content parts are in charge of what they render or not. For example, the `AutoroutePart` renders an editor shape, but it does not render a display shape.