##Conventions##
ORchard comes with a number of conventions. Some of them are inherited from technologies used such as ASP.NET MVC and NHibernate, others are Orchard specific.

In this section, we'll see the conventions in place related to content items, types, parts and fields.

###Content Types###
Whenever you create a new content type, you get to provide two names:

1. Display Name
2. Technical Name

The display name can be named anything you like and has no special character restrictions.
It is essentially the "friendly" name of the content type and can be changed at any time.

The technical name has a few more restrictions:

- The technical name must be a valid C# name.
- The technical name cannot be changed once it's chosen.

In case you're wondering about the requirement for a technical name being a valid C# name, we'll see why in the chapter **Accessing Content Parts and Fields**.    
 
###Content Parts###
Whenever you create a new content part, its name should end in **Part**.
For example, the following content parts are all suffixed with "Part": `TitlePart`, `AutoroutePart`, `BodyPart`, `CommonPart`, etc.

When displaying a list of content parts, Orchard displays a list without the *Part* suffix, but don't let this get to you - their real names use the *Part* suffix.

>Knowing the actual name of a content part is important when you want to programmatically access a content part on a content item.

The exception to the rule is **implicit parts**. What's that you say?

An implicit part is a part that will be created implicitly by Orchard when you attach a content field to a content type through the admin panel. As we've learned, content fields can only be attached to content parts. So in order for Orchard to trick you into believing it can defy the law of physics, it will create a content part *with the same name as the content type*. Because of this, the part will never appear in the list of content parts.