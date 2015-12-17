## Element Blueprints ##
Element Blueprints are pre-configured elements defined by the user that become available as elements in the toolbox.
The user defines the display name, the technical name, the base element type to preconfigure, and optionally a category for their new element.

### When to use Element Blueprints ###
Whenever you find that you are duplicating elements in various layouts, those elements are a candidate to be turned into an element blueprint. Examples are company logos, contact information and slogans that appear in multiple places on the site.

### How to create Element Blueprints ###
Creating element blueprints is easy. To create one, you click on the *Elements* admin menu item underneath the *Layouts* admin menu item. This will take you to the index screen of all of your element blueprints, which is empty by default. On this screen, click the *Create* button to the top right of the window. The screen that appears next presents you with all of the available elements to use for your pre-configured element. Once you picked one, the next screen will ask for the following information:

- Element Display Name
- Element Type Name
- Element Description
- Element Category

The **Element Display Name** is the *user friendly name* used when displaying the element in the toolbox and on the layout design surface.

The **Element Type Name** is the *technical name* of the element, and is used to when serializing and de-serializing the element. It's basically its unique identifier. No two elements must have the same name.

The **Element Description** is an optional field that gives you the opportunity to describe what the element represents, providing users a better understanding of what this element represents.

The **Element Category** also is an optional field, and lets you control in what category of the toolbox this element appears. If no category is specified, *Blueprints* is assumed as its category.

The following step-by-step guide demonstrates the creation of a sample element blueprint called *Contact Details*.

#### Demo: Creating the Contact Details Element Blueprint ####
##### Step 1 #####
Click on the *Elements* admin menu item.

![](http://i.imgur.com/kyrW6F0.png)

##### Step 2 #####
Click on the *Create* button on the top right side of the window and select the *Html* element as the base element for our element blueprint.

![](http://i.imgur.com/sJyyTxM.png)

##### Step 3 #####
Provide the following values for the following fields:

- Element Display Name -> *Contact Details*
- Element Type Name -> *ContactDetails*
- Element Description -> *My contact details*
- Element Category: *Demo*

And press *Create* to continue to the next screen, where the Html element editor will appear.

#### Step 4 #### 
Provide any content you like for this step. I entered the following HTML code:

    <p>John van Dyke</p>
    <p>Cell: +18723456</p>
    <p>Email: <a href="mailto:j.vandyke@acme.com">j.vandyke@acme.com</a></p>
    <p>Skype: johnvandyke</p>

And hit *Save*
Our demo blueprint element has now been created. The next step is to use this element on one of our pages.

![Figure 57 - Our demo blueprint element.](http://i.imgur.com/kaVUn66.png)

*Figure 57 - Our demo blueprint element.*

#### Step 5 ####
Go to the Content screen and edit an existing page or create a new one. Once the editor is loaded, you'll see that there's a new category called *Demo* available with our newly created *Contact Details* element.

![Figure 58 - Our demo blueprint element available from the toolbox.](http://i.imgur.com/TdpVI34.png)

*Figure 58 - Our demo blueprint element available from the toolbox.*

Simply drag & drop the *Contact Details* element to the design surface, and behold, there's our pre-configured element.

![Figure 59 - Our demo blueprint element as it appears on the design surface.](http://i.imgur.com/iPnxkWj.png)

*Figure 59 - Our demo blueprint element as it appears on the design surface.*

### Summary ###
In conclusion, blueprint elements enable you to create pre-configured elements and use them as regular elements. Blueprint elements are useful when you find that you use the same elements with the same configuration on multiple layouts. And whenever you change the blueprint element itself, the changes are reflected wherever the element appears.

In the next chapter, we'll look at another content part that is provided by the Layouts module: the *ElementWrapperPart*. which enables us to use elements as widgets, including blueprint elements.