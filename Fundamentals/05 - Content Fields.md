###Content Fields###
Content fields are similar to content parts in that they also provide data as well as behavior, but with the key difference being that you can add multiple content fields of a given type to any content part.

Notice that I said "content part"? This is because you can't attach content fields to content items directly. You always attach content fields to content parts.

>Don't let the admin panel fool you into thinking that you can attach content fields to content types. Despite the fact that you can seemingly attach fields to a content type via the admin panel, what really happens is that Orchard creates an *implicit part* under the hood. An implicit part is a regular content part, but with its name being the same as the content type.