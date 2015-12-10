## 10. Extensibility ##
So far we have seen how to use the Layouts module from a user's and theme developer's perspective. The functionality available out of the box is pretty impressive as it is. However, **Orchard.Layouts** wouldn't be a real Orchard module if it wasn't also extremely extensible.

One of the most interesting things to Orchard module developers is the extensibility points in a given module. When it comes to the Layouts module, one of the first things a developer asks himself is: "How can I create my own elements?"

As we have learned so far, users can create custom elements by creating *element blueprints*. These are pre-configured elements and doesn't require any bit of coding. We also learned how theme developers can provide custom elements called *snippets* by creating Razor views whose name ends in "Snippet.cshtml". These snippets can even be made configurable by defining *snippet fields* using the `Html.SnippetField` HTML helper.

But eventually there will be a point where you want complete control over how your custom elements behave and provide more advanced UI and behavior.

In this third part, we will learn everything there is to know about writing custom elements and cover the following topics:

- Writing **elements**.
- Writing **element harvesters**.
- Writing **container elements**.
- Working with elements **programmatically**
    - We'll learn how to programmatically instantiate elements and render them.
    - We'll learn about various element events that we can hook into.
- Reusing the Layout Editor shape from our own module.
