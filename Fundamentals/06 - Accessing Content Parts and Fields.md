##Accessing Content Parts and Fields##

Whether you're writing a new module or a new theme, there will come a day where you need to programmatically access various parts and fields of a content item. If you know the structure of a content item, this is easy.

The object model of a content item looks like this:

    public class ContentItem : DynamicObject, IContent {
        public IEnumerable<ContentPart> Parts { get; }
    }

	public class ContentPart : DynamicObject, IContent {
		public ContentPartDefinition PartDefinition { get; }
		public IEnumerable<ContentField> Fields { get; }
	}

	public class ContentField {
		public string Name { get; }
	}

In order to access any part on a content item, a simply lambda expression will do the trick. For example, let's say we have a content item variable called `contentItem` of type `ContentItem` of the following *Page* content type:

- Page
	- TitlePart
	- BodyPart
	- Page (implictly created part)
		- SubTitle (TextField)


In order to get our hands to the `TitlePart`, we could use the following expression:

	var titlePart = (TitlePart)contentItem.Parts.Single(part => part.PartDefinition.Name == "TitlePart");

That would work just fine. However, since we know the type of the content part, we can take advantage of the `As<T>` extension method, which is an extension on `IContent`:

	public static T As<T>(this IContent content) where T : IContent

This extension lives in the `Orchard.ContentManagement` namespace and enables us to simplify our code like this:

	var titlePart = contentItem.As<TitlePart>();

Very convenient. But we can do even better.
Notice that `ContentItem` and `ContentPart` both derive from `DynamicObject`. This enables them to implement specific behavior when instances of these classes are typed as `dynamic`. Which is exactly what the Orchard developers did.

When casting a content item to `dynamic`, we can access its parts *directly*. Similarly, when we have a dynamically typed part, we can access its fields directly as well. Let's say that we have a content item variable called `dynContentItem` of type `dynamic` of the same *Page* content type.

Accessing its `TitlePart` looks like this:  

	var titlePart = dynContentItem.TitlePart;

This blew my mind when I first grasped what was going on here. Basically, the dynamic behavior lets us access content parts *as if they were properties* on the content item. 

The same behavior works when accessing content fields on parts. Let's say we want to access the `SubTitle` content field attached to the implicit `Page` part. We could of course do this with chained method syntax as follows:

	var subTitleField =
		contentItem
			.Parts.Single(part => part.PartDefinition.Name == "Page")
			.Fields.Single(field => field.Name == "SubTitle");

That would work perfectly fine. But compare that with the following syntax (using the dynamically typed content item);

	var subTitleField = dynContentItem.Page.SubTitle;

Boom. Clean and simple. This syntax works great regardless of whether you have a strongly typed representation of your part or not. 

To summarize, to access parts and fields on a dynamically typed content item, memorize the following structure:

	part = contentItem.{Part}
	field = contentItem.{Part}.{Field}