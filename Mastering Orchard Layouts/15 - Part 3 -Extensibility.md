## Part 3 - Extensibility ##
So far we have seen how to use the Layouts module from a user's and theme developer's perspective. The functionality available out of the box is pretty impressive as it is, but the module wouldn't be an Orchard module if it weren't also extremely extensible.

There will come a point in your career as an Orchard developer where you want to write custom elements and generally need a deeper understanding of the Layouts module and the services it provides. Whatever the case may be, if you are looking to write custom elements and element providers or reuse some of the functionality provided by **Orchard.Layouts**, then this part is for you.

Part 3 is divided into the following 6 chapters:

**Chapter 13 - Writing Custom Elements**

The title says it all: in this chapter, you'll learn how to write your own elements.

**Chapter 14 - Container Elements**

In this chapter, we'll go through the process of writing a container element. Container elements are just elements, but have the ability to contain a list of child elements. the most challenging part of writing container elements is the integration with the layout editor, which unfortunately isn't as straightforward as one might have hoped. Good thing you're reading this book!

**Chapter 15 - Element Harvesters**

Chapter 13 showed how to implement custom elements by deriving from the Element class. In this chapter, you'll learn about the underlying mechanism that makes this possible, and even demonstrates how to write custom element harvesters for advanced scenarios.

**Chapter 16 - Extending Existing Elements**

Instead of creating new elements, there may be scenarios where you want to simply extend a particular, or multiple, elements. This chapter explains how this works and provides a demonstration.

**Chapter 17 - Layout and Element APIs**

All the services used by the Layouts module can be reused by you, awesome Orchard developer. This chapter explains how you can reuse the Layout Editor, how to instantiate and render elements programmatically, and more.

**Chapter 18 - Writing a SlideShow Element**

With all of the new-found knowledge under your belt, you will be ready to write the ultimate element: the SlideShow element. That is, it will be the most advanced element you'll have seen so far in terms of the element editor experience. This chapter takes you through some advanced techniques, taking advantage of everything you learned in previous chapters.