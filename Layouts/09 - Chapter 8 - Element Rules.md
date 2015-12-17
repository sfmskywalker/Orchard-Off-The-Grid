## Element Rules ##
Yes, elements do rule in my opinion. They are powerful little graphical units with none to advanced behavior. In this chapter, we'll learn about yet another feature that can be applied to elements when using in the context of the layout editor: *Element Rules*.

### About Element Rules ###
Element Rules are very similar to Widget Layer Rules, or Layer Rules for short, and use in fact the exact same rule engine.

As you may know, Layer Rules are a way to display widgets linked to a layer using a rule to be rendered or not, depending on whether or not the rule evaluates to true.

The Layouts module leverages this rule system so that users can apply rules on a per-element level, controlling their visibility based on the specified rule.

This enables you to for example only display a certain element if a user is authenticated, or any other condition that the rules system supports.

This also means that if a rule applied to a container element evaluates to false, that container element, including its children, will not be rendered.

Element Rules can also be applied on Layout Templates for more advanced use cases.

### Available Functions ###
To use rules, one must use functions that evaluate to a boolean value. The following is a list of functionas that are available out of the box:

<table>
  <thead>
  <tr>
    <th>Function</th>
    <th>Description</th>
    <th>Example</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>authenticated</td>
    <td>Evaluates to true if the current user is authenticated, false otherwise</td>
    <td>authenticated</td>
  <tr>
  <tr>
    <td>contenttype(contentTypeName)</td>
    <td>Evaluates to true if the current content item being displayed is of the specified content type, false otherwise</td>
    <td>contenttype('Page')</td>
  <tr>
  <tr>
    <td>url(relativePath)</td>
    <td>Evaluates to true if the current URL matches the specified relative path, false otherwise</td>
    <td>url('~/contact')</td>
  <tr>
  </tbody> 
</table>

Functions can be combined using the logical `and` and `or` operators and negated with the `not` operator. For example, the following rule will evaluate to `true` if the current URL is either *~/contact* or *~/about* and the user is not authenticated:

    (url('~/contact') or url('~/about')) and not authenticated 

### Applying Element Rules ###
To apply an element rule, simply click on an element's *Edit* toolbar icon and enter the desired rule into the *Visibility Rule* text area.

![Figure 65 - The Element Rule editor.](http://i.imgur.com/fgsDdtN.png)

*Figure 65 - The Element Rule editor.*

#### Demo: Element Rules in Action ####
In this demo, we'll add an Html element to the homepage content item and apply a simply rule called `authenticated`. This rule evaluates to `true` if the current user is authenticated, and `false` otherwise. This will effectively prevent the Html element from being rendered only if the current user is authenticated. If the user is an anonymous visitor, the element will not be rendered.

##### Step 1#####
Go the the *Welcome to Orchard* page edit screen and add an *Html* element to the canvas with the following HTML contents:

    <h2>Welcome, #{User.Name}!</h2>

Notice that we're using another token. The *User.Name* token will display the currently logged in user name. 

##### Step 2 #####
Click on the *Edit* toolbar icon and enter the following rule into the *Visibility Rule*:

    authenticated

And hit **Publish Now**.

##### Step 3 #####
While remaining authenticated, navigate to the homepage and observe that our Html element is visible as expected.

![Figure 66 - The Html element is visible on the front-end when the current user is authenticated.](http://i.imgur.com/cPV9zLb.png)

*Figure 66 - The Html element is visible on the front-end when the current user is authenticated.*

##### Step 4 #####
Click on the *Sign Out* link at the bottom of the page to sign out of the site. You'll be redirected to the homepage. Notice that the Html element we added in Step 1 is no longer visible.

![Figure 67 - The Html element is no longer visible on the front-end since the current user is now unauthenticated.](http://i.imgur.com/VUcEx0v.png)

*Figure 67 - The Html element is no longer visible on the front-end since the current user is now unauthenticated.*

### Summary ###
That's really all there is to the Element Rules. We learned that the rules engine is the same one used for the Widgets module and where to put those rules.

This was the final chapter of Part 1. In Part 2, we'll learn about all there is to know about theming layouts and elements and really take control of how things are rendered on the front-end, including integrating grid frameworks such as Bootstrap.