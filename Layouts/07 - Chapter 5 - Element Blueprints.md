## Element Blueprints
In this chapter we'll take a look at **Element Blueprints**, which is a feature that enables the user to create pre-configured elements that become available from the toolbox.

### When to use Element Blueprints
Whenever you find that you are duplicating elements in various layouts, those elements are a candidate to be turned into an element blueprint. For example, if you find yourself duplicating elements all over the place without changing its contents, those elements could be turned into a blueprint.

### Creating Element Blueprints
Creating element blueprints is easy. To create one, go to the **Elements** admin menu item right under the **Layouts** admin menu item. This will take you to the index screen of all of your element blueprints, which is empty by default. On this screen, click the **Create** button to the top right of the window. The screen that appears next presents you with all of the available elements to use for your pre-configured element. When one is selected, the next screen will prompt the user for the following information:

- Element Display Name
- Element Type Name
- Element Description
- Element Category

The **Element Display Name** is the *user friendly name* used when displaying the element in the toolbox and on the layout design surface.

The **Element Type Name** is the *technical name* of the element, and is used when serializing and de-serializing the element. No two elements must have the same name.

The **Element Description** is an optional field that gives you the opportunity to describe what the element represents. The description helps users get a better understanding of what this element represents.

The **Element Category** is an optional field that lets you control in what category of the toolbox this element appears. If no category is specified, *Blueprints* is used as the category.

#### Trying it out: Creating an Element Blueprint
In the example that follows, I will show you step-by-step how to create a sample blueprint element called **Contact Details**. The purpose of this element is for the user to be able to manage their contact details from a single place, while being able to place this element on various pages. Whenever the user changes their contact details, the changes are reflected everywhere.

##### Step 1
Click on the *Elements* admin menu item.

![Figure 5.1 - Elements admin menu](./figures/fig-5-1-adminmenu-elements.png)

*Figure 5.1 - Elements admin menu*

##### Step 2
Click the **Create** button on the top right side of the window and select the **Html** element as the base element for our element blueprint.

![Figure 5.2 - Selecting a base element](./figures/fig-5-2-html-base-element.png)

*Figure 5.2 - Selecting a base element*

##### Step 3
On the screen that appears next, provide the following values:

- Element Display Name -> *Contact Details*
- Element Type Name -> *ContactDetails*
- Element Description -> *My contact details*
- Element Category: *Demo*

Hit **Create** to continue to the next screen, where the **Html** element editor will appear.

##### Step 4
Enter the following HTML code into the Html editor (using the HTML view of TinyMCE):

    <p>John van Dyke</p>
    <p>Cell: +18723456</p>
    <p>Email: <a href="mailto:j.vandyke@acme.com">j.vandyke@acme.com</a></p>
    <p>Skype: johnvandyke</p>

And hit **Save**.

![Figure 5.3 - A new blueprint element has been created](./figures/fig-5-3-demo-blueprint-element.png)

*Figure 5.3 - A new blueprint element has been created*

##### Step 5
Now that the blueprint has been created, we can start using it. Go to the **Content** screen and add or edit a Page content item. Notice the new category called *Demo* and the new **Contact Details** element.

![Figure 5.4 - The blueprint element is available from the toolbox](./figures/fig-5-4-demo-blueprint-element-toolbox-item.png)

*Figure 5.4 - The blueprint element is available from the toolbox*

Add the *Contact Details* element to the canvas.

![Figure 5.5 - The demo blueprint element as it appears on the canvas.](./figures/fig-5-5-demo-blueprint-element-design.png)

*Figure 5.5 - The demo blueprint element as it appears on the canvas*

We can add this element to as many pages as we like. When the time comes that your contact details change, all you have to do is update the blueprint element once, and the changes get reflected everywhere.

### Summary ###
Blueprint elements enable you to create pre-configured elements and use them as normal elements. Blueprint elements are useful when you find that you use the same elements with the same configuration in multiple places. Whenever you change the blueprint element itself, the changes are reflected everywhere the element appears.