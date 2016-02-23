## First Look
In this chapter, we'll take a tour through the Layouts module and see how it works from a user's perspective.

### The Layouts Feature
When you setup a new Orchard 1.9 or later installation with the **Default** recipe, the Layouts feature will be enabled by default.
One noticeable difference between 1.9 and previous versions is that the **Page** content type will have the **LayoutPart** attached, instead of the **BodyPart**.

### Layout Part
The Layout Part provides an editor that enables the user to drag and drop **elements** onto a canvas. These elements then get rendered on the front-end when the content item is being displayed.

### Elements
Elements are a new concept in Orchard. They are things that contain data, provide behavior and provide their own rendering. Elements can even contain other elements, allowing for nesting. This, in fact, is how layouts are achieved, as we'll see shortly. 

Out of the box, there are seven categories of elements:

* Layout
* Content
* Media
* Parts
* Fields
* Snippets
* UI

It's all lovely stuff, and we'll get to know all of the available elements in the next chapter.

### Layout Editor
The Layout Editor, which is what powers the Layout Part, enables users to visually compose collections of elements onto a design surface, ultimately making up the layout of a piece of content.

Let's see how this looks like when editing the default Orchard homepage (figure 12):

![Figure 2.1 - The layout editor](./figures/fig-2-1-layoutpart-editor.png)

*Figure 2.1 - The layout editor*

The editor consists of two main sections: the *canvas* and the *toolbox*.
The canvas is the area onto which you place elements that are made available via the toolbox.

> The canvas itself is an element of type Canvas, and is the root of the tree of elements.

The toolbox is a repository of all available elements in the system, grouped per category. Elements are bound to Orchard features, which means that other modules can provide additional element types.

The user places elements from the toolbox onto the surface by using drag & drop. If the selected element has an editor associated with it, a dialog window presenting the element's properties will appear immediately when it's dropped onto the canvas.

![](./figures/fig-2-3-drag-html-element.png)
*Figure 2.3 - The Layout editor uses drag & drop to add elements to the design surface*

#### Element Editor Controls ####
Depending on the element being selected, the user can perform certain operations. These operations are represented as little icons as part of a mini toolbar that becomes visible when an element is selected. Common operations are *Edit*, *Edit Properties*, *Delete*, *Move up* and *Move down*. More specific operations are *Distribute columns evenly* and *Split column*, which apply to **Row** and **Column**, respectively.

![](./figures/fig-2-4-html-element-toolbar.png)

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

*Table 2.1 - A list of the various toolbar icons*

#### Keyboard Support
In addition to the keyboard shortcuts listed above, there is also keyboard support for doing things like **copy**, **cut**, **paste**, and **navigating** around the hierarchy of elements on the canvas.

The layout editor provides a link to a small pop-up window listing all of the available keyboard shortcuts.

![Figure 2.5 - Dialog window listing all of the available keyboard shortcuts](./figures/fig-2-5-layout-editor-keyboard-shortcuts.png)

*Figure 2.5 - Dialog window listing all of the available keyboard shortcuts*

The following table lists the complete set of keyboard shortcuts.

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

*Table 2.2 - The complete set of keyboard shortcuts available*

#### Moving Elements Within its Container
Once an element is placed on the canvas, its position can be changed within its container using drag & drop.

#### Moving Elements Across Containers
At the time of this writing, it is not possible to move an element to another container using drag & drop. Instead, use the Cut/Paste keyboard shortcuts (**Ctrl+X** and **Ctrl+V**) to cut and paste an element from its current container to another one.

#### Re-sizing Columns
**Column** elements can be re-sized by dragging their left and right edges. When you re-size a column, its adjacent column will be re-sized as well. If you want to re-size a column and introduce an offset ("detaching" the column from its neighbor), press the **Alt** key while dragging the edges.

### Front-end ###
Enabling the user to create and manage layouts from the back-end is only half of the story. The second half is getting that layout out on the screen on the front-end.

Figure 2.6 demonstrates what the layout will look like on the front end.

![Figure 2.6 - The front-end view of the Layout Part](./images/fig-2-6-layoutpart-frontend.png)

*Figure 2.6 - The result of the layout as constructed on the default Orchard homepage*

The red bordered box is the section as rendered by the **LayoutPart**.

> In previous Orchard versions, the three columns shown in figure 2.6 were originally implemented as 3 Html widgets that were placed in the **TripelFirst**, **TripelSecond** and **TripelThird** zones. In case you're wondering if I made a typo, rest assured; The naming of these zones are based on the name of a strong Belgium beer, and is an internal joke. Since these widgets were specific to the homepage, they have been replaced by simply adding a row and three columns to the content's layout.

We will get into this much deeper when we look at how elements and the rendering engine works in detail later in this book, but for now I wanted to show you how the rendered markup looks like. Let's have a look at figure 19:

![Figure 2.7 - A closer look at the rendered output of the LayoutPart](./figures/fig-2-7-layoutpart-frontend-analyzed.png)

*Figure 2.7 - A closer look at the rendered output of the **LayoutPart***

What we see here is a portion of the rendered HTML output of the Layout Part. The way it works is that each element is responsible for its own rendering. Container elements render their HTML and then iterate over each child element and tell them to render itself. This rendering system leverages Orchard's *shapes* mechanism.

Notice that the **Column** elements have a **class** attribute with value **span-4 cell**. If you're familiar with Bootstrap 2, you'll probably recognize this. However, the Layouts module does not use Bootstrap's grid system on the front-end. Instead, it uses a CSS file called **default-grid.css** which is based on [Sebastien Ros'](https://github.com/sebastienros "Sebastien Ros") [custom grid CSS](https://github.com/sebastienros/custom-grid "Custom Grid"), a lightweight CSS grid framework.

We'll see how to replace this markup with Bootstrap-specific markup instead and learn how to completely take over control of rendering any elements from our themes in chapter 10.

### Summary ###
In this chapter, we learned at a high level overview of what the Layouts module is all about. In essence, it's about being able to easily create layouts of contents.
We were introduced to the **Orchard.Layouts** module, the **LayoutPart**, its editor and how that appears on the front-end by default.

In the next chapter, we'll have a look at all of the elements that come with the box.