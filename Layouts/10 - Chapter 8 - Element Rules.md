## Element Rules
**Element Rules** are very similar to **Widget Layer Rules**, or **Layer Rules** for short. In fact, element rules use the same rule engine provided by **Orchard.Conditions**.

> **Orchard.Conditions** is a new module as of 1.10, and replaces the now deprecated rules engine provided by the **Orchard.Widgets** module. 

The **Widgets** module uses layer rules to display widgets linked to a layer using a rule. If the rule evaluates to true, all widgets associated with the layer are rendered. If the rule evaluates to false, none of those widgets will be rendered.

The **Layouts** module leverages the same rules engine, but instead of controlling layer visibility, it controls *element visibility*.

This enables you for example to only display a certain element if a user is authenticated using the **authenticated** function.
If a rule applied to a container element evaluates to false, that container element, including its children, will not be rendered.

### Available Functions
To use rules, you use functions that evaluate to a **boolean** value. The following is a list of functions that are available out of the box:

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

Functions can be combined using the logical **and** and **or** operators and negated with the **not** operator. For example, the following rule will evaluate to **true** if the requested URL is either **~/contact** or **~/about** and the user is **not authenticated**:

    (url('~/contact') or url('~/about')) and not authenticated 

### Applying Element Rules
To apply an element rule, simply click on an element's **Edit** toolbar icon and enter the desired rule into the **Visibility Rule** text area:

![Figure 8.1 - The Element Rule editor.](./figures/fig-8-1-element-rule-editor.png)

*Figure 65 - The Element Rule editor.*

#### Trying it out: Element Rules
In this example, we'll add an **Html** element to the homepage and apply a simply rule called **authenticated**. This rule evaluates to **true** if the current user is authenticated, and **false** otherwise. This will effectively prevent the **Html** element from being rendered only if the current user is authenticated. If the user is an anonymous visitor, the element will not be rendered.

##### Step 1
Go the the **Welcome to Orchard** content item and add an **Html** element to the canvas with the following HTML contents:

    <h2>Welcome, #{User.Name}!</h2>

The **User.Name** token will display the currently logged in user name. 

##### Step 2
Click on the **Edit** toolbar icon and enter the following rule into **Visibility Rule**:

    authenticated

And hit **Publish Now**.

##### Step 3
While remaining authenticated, navigate to the homepage and observe that our **Html** element is visible as expected.

![Figure 66 - The Html element is visible on the front-end when the current user is authenticated.](./figures/fig-8-2-element-rule-frontend-authenticated.png)

*Figure 66 - The Html element is visible on the front-end when the current user is authenticated.*

##### Step 4
Click the **Sign Out** link at the bottom of the page. You'll be signed out and redirected to the homepage. Notice that the **Html** element we added in Step 1 is no longer visible.

![Figure 67 - The Html element is no longer visible on the front-end since the current user is now unauthenticated.](./figures/fig-8-3-element-rule-frontend-unauthenticated.png)

*Figure 67 - The Html element is no longer visible on the front-end since the current user is now unauthenticated.*

### Summary
That's really all there is to element rules. The rules engine used is the same one used by the Widgets module, and provides control over the visibility of a given elements.