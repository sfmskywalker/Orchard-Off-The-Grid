##Content Parts##
A content part represents a reusable building block for content items that provides data and behavior.  
Or, in other words, a content part is an atom of a content item.

A content part can have properties in the form of content fields as well as in the form of a strongly typed class representation of that content part.

The following list depicts the structure of a content part:

- Content Part
	- Name
	- Content Fields

Examples of content parts are `TitlePart`, `BodyPart` and `CommentsPart`. Each of these parts are reusable and can be attached to any other content type, providing specialized behavior and information to content items of that type.

An important aspect about content parts is that you can only have one content part type per content item. For example, you can attach the `TitlePart` to both the *Page* and *BlogPost* content types, but you cannot attach the `TitlePart` twice to either one of them.

Content parts more or less represent something that a content item *is*.
If you do want to add multiple pieces of information of a particular data type to a content item, you can use **content fields** for that.

###Content Part Schema###


###Strongly Typed Content Parts###
