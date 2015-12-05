##Orchard Concepts##

###Content Items###
A content item is a single piece of content in the system, and implements the 'C' in CMS.
Content items are often associated with a URL, but are also things like widgets, users and menu items.
Other examples of content items are pages, blog posts and products.

###Content Types###
A content type serves as a "blueprint" or "class" of content items in the system. Examples of content types are *Page*, *BlogPost* and *Product*.

###Content Parts###
A content part represents a reusable building block for content items that provides data and behavior.  
In other words, a content part is an atom of a content item.

Examples of content parts are `TitlePart`, `BodyPart` and `CommentsPart`. Each of these parts are reusable and can be attached to any other content type, providing specialized behavior and information to content items of that type. 

###Content Fields###
Content fields are pieces of information that can be attached to content parts. Like content parts, they also provide data as well as behavior.