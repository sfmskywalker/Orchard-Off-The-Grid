## First Look
In this chapter, we'll take a tour through the Layouts module and see how it works from a user's perspective.
We will assume you have a clean Orchard 1.9.1 or later installation running.

### The Layouts Feature
When you setup a new Orchard 1.9.1 or later installation, the Layouts feature will be enabled by default.

![](http://i.imgur.com/HnyIuz5.png) 

*Figure 10 - The new Layouts feature is enabled by default on new Orchard installations*

Out of the box, the `Page` content type will have the `LayoutPart` attached by default instead of the `BodyPart`, as you can see in figure 11.

![](http://i.imgur.com/cxzFyWY.png)
*Figure 11 - The Page content type has a LayoutPart instead of a BodyPart*

### Layout Part ###
The `LayoutPart` is a key component that enables the user to create layouts of elements on a page. When attached to a content type, it provides a nifty layout editor that consists of a *design surface* onto which one can drag & drop elements from a *toolbox*.

So whenever you need a content type that needs a body of text, you should consider using the `LayoutPart` instead of the `BodyPart`, as this gives you greater flexibility over the layout of the content.

### Elements ###
Elements are a new concept in Orchard. They are the entities that we can drag & drop onto the design surface or even render inline in a piece of HTML code using the `Element.Display` token, and basically represent visual entities for users to work with. They are the fundamental building blocks to the way the Layouts module works.

Elements are powerful building blocks that are visual, contain data and provide behavior.
In fact, Elements are so fundamental to the Layouts module that the module might as well have been named `Orchard.Elements`. If you've ever worked with a game level editor, Orchard elements are to the design surface as sprites are to a game level.

Elements have the following characteristics:

* They have a technical name as well as a display name.
* They have a category.
* They can have data.
* They can have behavior.
* They can contain other elements.
* They can render their edit screen.
* They can render themselves on the design surface.
* They can render themselves on the front end.

Out of the box, there are seven categories of elements:

* Layout
* Content
* Media
* Parts
* Fields
* Snippets
* UI

It's all lovely stuff, and we'll get to know all of the available elements in the next chapter.

### Layout Editor ###
The Layout Editor is another key component of the Layouts module, as it enables users to visually compose collections of elements onto a design surface, ultimately making up the layout of a piece of content.

Let's see how this looks like when editing the default Orchard homepage (figure 12):

![Figure 12 - The layout editor](http://i.imgur.com/I5yhOXl.png)

*Figure 12 - The layout editor*

As you can see, the entire `BodyPart` and its default `TinyMce` editor have been replaced with the *Layout Editor*. 

The editor consists of two main parts: the *design surface* and the *toolbox*.
The design surface is the area onto which you place *elements*.

> Elements are a new type of entity in Orchard, and represent atomic objects that can be placed onto a canvas. Whenever you want to be able to place something onto a canvas,you need an element for it.

The toolbox is a repository of all available elements in the system, grouped per category. Elements are bound to Orchard features, which means that other modules can provide more element types. To make those elements available, all you need to do is enable those module's features. One such example is the `Projection Element` feature provided by the `Orchard.Layouts` module. We will go over each element that ships with orchard later in this chapter.

The way the editor works is that the user will place elements from the toolbox onto the surface by using drag & drop. If the selected element has an editor associated with it, a dialog window will appear immediately when it's dropped onto the surface. Figure 14 and 15 show the `Html` element being dragged & dropped onto the canvas as an example.

![](http://i.imgur.com/6SQVEbd.png)
*Figure 14 - Dragging the Html element onto the canvas*

![](http://i.imgur.com/BefBmXe.png)
*Figure 15 - When dropping the Html element onto the canvas, a dialog window appears enabling the Html properties to be edited*

Regardless of the changes you make to the design surface, only when you actually save the content item will they persist. The way this works is that as soon as the `Save` or `Publish Now` button is clicked, the layout editor serializes the root `Canvas` element and its children into a JSON string which is set to the value of a hidden input field, which gets submitted as part of the form post.

So even deleted elements will not be really gone until you save the content item.

#### Element Editor Controls ####
All elements on a canvas are selectable. Depending on the element being selected, the user can perform certain operations on that element. These operations are represented as little icons as part of a mini toolbar that becomes visible when an element is selected. Common operations are *Edit*, *Edit Properties*, *Delete*, *Move up* and *Move down*. Each element provides its own toolbar. Elements such as `Row` and `Column` for example provide additional operations such as *Distribute columns evenly* and *Split column*.

![](http://i.imgur.com/ZICdJ2D.png) 

*Figure 16 - The Html element toolbar*

The following table lists all operations available to various elements:

<table>
  <thead>
    <tr>
      <th>Icon</th>
      <th>Name</th>
	  <th>Keyboard&nbsp;shortcut</th>
      <th>Description</th>
	  <th>Element type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><img src="http://i.imgur.com/BLdVf12.png" /></td>
      <td>Edit element</td>
      <td>Enter</td>
      <td>Launches the element specific editor dialog.</td>
      <td>Applies to all elements that have an editor.</td>
    </tr>
    <tr>
      <td><img src="http://i.imgur.com/6HWgfou.png" /></td>
      <td>Edit element properties</td>
      <td>Space</td>
      <td>Displays an inline popup window with properties common to all elements. These include <strong>HTML ID</strong>, <strong>CSS Classes</strong>, <strong>CSS Styles</strong> and <strong>Visibility Rule.</strong></td>
      <td>Applies to all elements.</td>
    </tr>
    <tr>
      <td><img src="http://i.imgur.com/vlDYULk.png" /></td>
      <td>Delete element</td>
      <td>Del</td>
      <td>Deletes the selected element.</td>
      <td>Applies to all elements, except the root Canvas element.</td>
    </tr>
    <tr>
      <td><img src="http://i.imgur.com/ExEXWPE.png" /></td>
      <td>Move element up</td>
      <td>Ctrl + Up</td>
      <td>Moves the element up, while moving the element currently above the element down.</td>
      <td>Applies to all elements.</td>
    </tr>
    <tr>
      <td><img src="http://i.imgur.com/wJ2jYR8.png" /></td>
      <td>Move element down</td>
      <td>Ctrl + Down</td>
      <td>Moves the element down, while moving the element currently below the element up.</td>
      <td>Applies to all elements.</td>
    </tr>
    <tr> 
      <td><img src="http://i.imgur.com/SN34L5I.png"/></td>
      <td>Distribute columns evenly</td>
      <td></td>
      <td>Distributes the columns in the selected row evenly.</td>
      <td>Row</td>
    </tr>
    <tr> 
      <td><img src="http://i.imgur.com/SN34L5I.png"/></td>
      <td>Split column</td>
      <td></td>
      <td>Splits the selected column in two.</td>
      <td>Column</td>
    </tr>
    <tr> 
      <td><img src="http://i.imgur.com/t4S5Wq0.png" /></td>
      <td>Decrease column offset</td>
      <td>Alt + Left</td>
      <td>Decreases the column offset by one.</td>
      <td>Column</td>
    </tr>
    <tr> 
      <td><img src="http://i.imgur.com/yeos4Wq.png"></td>
      <td>Increase column offset</td>
      <td>Alt + Right</td>
      <td>Increases the column offset by one.</td>
      <td>Column</td>
    </tr>
  </tbody>
</table> 

*Table 1 - A list of the various toolbar icons available to varous types of elements*

#### Keyboard Support ####
In addition to the keyboard shortcuts listed above per element action from the toolbar, there is also keyboard support for doing things like copy, cut, paste, and navigating around the hierarchy of elements on the design surface.

The layout editor provides a link to a small popup window listing all of the available keyboard shortcuts (figure 17), which I'll repeat here for your convenience.

![Figure 17 - Dialog window listing all of the available keyboard shortcuts](http://i.imgur.com/UsLshtd.png)

*Figure 17 - Dialog window listing all of the available keyboard shortcuts*

<table>
  <thead>
    <tr>
	  <th>Windows</th>
      <th>Mac OS</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th colspan="3" align="left">Clipboard</th>
    </tr>
    <tr>
      <td>Ctrl + X</td>
      <td>⌘ + X</td>
      <td>Cuts the selected element.</td>
    </tr>
    <tr>
      <td>Ctrl + C</td>
      <td>⌘ + C</td>
      <td>Copies the selected element.</td>
    </tr>
    <tr>
      <td>Ctrl + V</td>
      <td>⌘ + V</td>
      <td>Pastes the copied element from the clipboard into the selected container element.</td>
    </tr>
    <tr>
      <th colspan="3" align="left">Resizing columns</th>
    </tr>
    <tr>
      <td colspan="2">Alt + Left</td>
      <td>Moves the left edge of the focused column left.</td>
    </tr>
    <tr>
      <td colspan="2">Alt + Right</td>
      <td>Moves the left edge of the focused column right.</td>
    </tr>
    <tr>
      <td colspan="2">Shift + Left</td>
      <td>Moves the right edge of the focused column left.</td>
    </tr>
    <tr>
      <td colspan="2">Shift + Right</td>
      <td>Moves the right edge of the focused column right.</td>
    </tr>
    <tr>
      <td colspan="3">The <em>Alt</em> and <em>Shift</em> keys can also be combined to move both edges simultaneously.</td>
    </tr>
    <tr>
      <th colspan="3" align="left">Focus</th>
    </tr>
    <tr>
      <td colspan="2">Up</td>
      <td>Moves focus to the previous element (above).</td>
    </tr>
    <tr>
      <td colspan="2">Down</td>
      <td>Moves focus to the next element (below).</td>
    </tr>
    <tr>
      <td colspan="2">Left</td>
      <td>Moves focus to the previous column (left).</td>
    </tr>
    <tr>
      <td colspan="2">Right</td>
      <td>Moves focus to the next column (right).</td>
    </tr>
    <tr>
      <td colspan="2">Alt + Up</td>
      <td>Moves focus to the parent element.</td>
    </tr>
    <tr>
      <td colspan="2">Alt + Down</td>
      <td>Moves focus to the first child element.</td>
    </tr>
    <tr>
      <th colspan="3" align="left">Editing</th>
    </tr>
    <tr>
      <td colspan="2">Enter</td>
      <td>Opens the content editor of the focused element.</td>
    </tr>
    <tr>
      <td colspan="2">Space</td>
      <td>Opens the properties popup of the focused element.</td>
    </tr>
    <tr>
      <td colspan="2">Esc</td>
      <td>Closed the properties popup of the focused element.</td>
    </tr>
    <tr>
      <td colspan="2">Del</td>
      <td>Deletes the focused element.</td>
    </tr>
    <tr>
      <th colspan="3" align="left">Moving</th>
    </tr>
    <tr>
      <td colspan="2">Ctrl + Up</td>
      <td>Moves (reorders) the focused element up.</td>
    </tr>
    <tr>
      <td colspan="2">Ctrl + Down</td>
      <td>Moves (reorders) the focused element down.</td>
    </tr>
    <tr>
      <td colspan="2">Ctrl + Left</td>
      <td>Moves (reorders) the focused element left.</td>
    </tr>
    <tr>
      <td colspan="2">Ctrl + Right</td>
      <td>Moves (reorders) the focused element right.</td>
    </tr>
  </tbody>
</table>

#### Drag & Drop ####
As mentioned already, the layout editor supports drag & drop operations.
The user can drag and drop any element from the toolbox onto the design surface.

Once an element sits within a container element such as `Canvas` or `Column`, the user can change position within that container using drag & drop operations as well.

> At the time of this writing, it is not possible to move an element to another container using drag & drop Instead, use the Cut/Paste keyboard shortcuts (`Ctrl+X` and `Ctrl+V`) to cut and paste an element elsewhere.

Columns can be resized by dragging their left and right edges. When you resize a column, its adjacent column will be resized as well. If you want to resize a column and introduce an offset ("detaching" the column from its neighbor), press the `Alt` key while dragging the edges.

> [Daniel Stolt](https://github.com/DanielStolt), the contributor who wrote the layout editor, is working on an exciting new open source project called [Ayos](https://github.com/IDeliverable/ayos "Ayos"). In a nutshell, Ayos is a standalone layout editor and will be more powerful, rich and flexible than the layout editor that currently ships with Orchard at the time of this writing.

### Layout Frontend ###
Enabling the user to create and manage layouts from the back-end is only half of the story of course. The second half is getting that layout out on the screen on the front-end.

Figure 18 demonstrates what the layout will look like on the front end.

![Figure 18 - The front-end view of the Layout Part](http://i.imgur.com/Vz1lSHK.png)

*Figure 18 - The result of the layout as constructed on the default Orchard homepage*

The red bordered box is the section as rendered by the `LayoutPart`.
As you can see, the three columns of the second row are neatly rendered as expected: each column appears horizontally adjacent.

We will dig into this much deeper when we look at how elements and the rendering engine works in detail later in this book, but for now I wanted to show you how the rendered markup looks like. Let's have a look at figure 19:

![Figure 19 - A closer look at the rendered output of the Layouts Part](http://i.imgur.com/uuBU5uw.png)

*Figure 19 - A closer look at the rendered output of the Layouts Part*

What we see here is a portion of the rendered HTML output of the `LayoutsPart`. The way it works is that each element is responsible for its own rendering. Container elements render their HTML and then iterate over each child element and tell them to render itself. This rendering system leverages Orchard's *shapes* mechanism.

Notice that the Column elements have a `class` attribute with value `span-4 cell`. If you;re familiar with Bootstrap 2, you'll probably recognize this. However, the Layouts module does not use Bootstrap's grid system on the front-end. Instead, it uses a home-grown CSS file called `default-grid.css` which is based on [Sebastien Ros'](https://github.com/sebastienros "Sebastien Ros") [custom grid CSS](https://github.com/sebastienros/custom-grid "Custom Grid"), a lightweight CSS grid framework.

Later on we'll see how we can replace this markup with Bootstrap-specific markup instead and learn how to completely take over control of rendering any elements from our themes.  

### Summary ###
In this chapter, we were given a high level overview of what the Layouts module is all about. In essence, it's about being able to easily create layouts of contents.
We were introduced to the `Layouts` feature, the `LayoutPart`, its editor and how that appears on the front end with the default *ThemeMachine*.

However, there is much more to Layouts than we have seen so far, as we'll find out.
In the next chapter, we'll have a look at all of the elements that come with the box.