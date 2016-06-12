## Writing a SlideShow Element
With everything you learned so far, you'll already be able to implement many types of elements.
However, there is one scenario we haven't covered yet: element editors that span multiple pages. Let's say that your element can store a collection of things, and for each thing, you want the user to be able to edit its properties. One could certainly implement this using a single element editor screen using JavaScript techniques. But depending on the nature of this thing, the UI could become too complex.

So we have a choice. In this chapter, I will walk you through the implementation of a sample element called **SlideShow**.

### Defining the Slide Show Element
A slide show basically consists of acollection of slides. What these slides are, that is up to us to decide. In this example, I decided that each slide is a simple class that stores a **layout of elements**.
This means that each individual slide of the slide show element will be **a layout itself**. This enables the user to create any sort of layout, ranging from simple elements to more complex layouts using grid elements.

When implementing the `SlideShow` element, we will need one screen to display the slides (in thumbnail form) and the slide show player specific configuration (interval, wrap, etc). However, the user needs a way to edit each individual slide using the layout editor. For that, we'll need a custom controller that handles create, edit and delete operations of a given slide.

The slide show element will have the following functionality:

- Being an element, the user can drag & drop the element onto a canvas.
- Each slide of a slideshow itself will be a *layout*. Not a content item, but a blob of data stored as part of the element's *Data* dictionary itself.
- The user can add/remove individual slides and rearrange them using drag & drop.
- The user can edit individual slides by reusing the layout editor.
- The design display type of the element will render the first slide.
- The slide show slides will be animated using Bootstrap's Carousel component.
- The user will be able to configure various properties of the slide show's playback, such as the *Interval* and whether to display *previous/next* controls. 

Before we dive in, have a look at the following pictures that demonstrate the functionality we'll create.

The slide show editor will be divided into two tabs:

1. Slides 
2. Player

The *Slides* tab enables the user to add, edit, remove and rearrange slides:
 
![](./figures/fig-113-slideshow-slides-editor.png)

The *Player* tab enables the user to configure Bootstrap Carousel-specific properties:

![](./figures/fig-114-slideshow-player-editor.png)

And on the front-end, each slide will be displayed one at a time by the Bootstrap Carousel script:

![](./figures/fig-115-slideshow-front-end.png)

In the next sections we will walk through the process of creating this slide show element in detail.
So without further ado, let's dive in.

In order to implement something like that, we need to understand how element data is transferred from the layout editor to the element editor dialog in the first place, so that will be our first topic.

### Transferring Element Data
When the user clicks the *Edit* icon on an element on the layout editor, here is what happens:

1. The element data (which is stored as part of the client-side object model) is **POST**-ed to a new window (the Element Editor Dialog). More specifically, this data is posted to the `Edit` action on the `ElementController` in the Layouts module.
2. The `Edit` action stores the posted data into a thing called the **object store**. The object store is basically a durable key/value store. The default implementation of `IObjectStore` relies on session state to store and retrieve entries. An important aspect of storing this data is the **key** being used. Whenever you store something in the object store, you need to provide a key, and then hold on to that key since it's the only way to get back your data.
4. The `Edit` action returns a *redirect* result to an overloaded version of the `Edit` action, passing in the *key* being used to store the element data. This key (called **session**) is then used to get the element data, query its drivers and display its editors.

The decision to store the element state in an intermediary object store is what enables us to create any number of screens we need to service our elements, since there is no direct way to retrieve the element status from the database (since elements may not have been persisted yet in the first place). Just as long as our custom controllers receive the key, we can get our data from the object store, make changes, and put it back into the store. Once we're done from our custom controller, we need to make sure to ultimately redirect back to the `ElementController`'s `Edit` action with the correct key.

### The SlideShow Element Class
Start by creating a new class called `SlideShow` as follows:

```
using Orchard.Layouts.Framework.Elements;
using Orchard.Layouts.Helpers;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class SlideShow : Element {
        public override string Category => "Media";
        public override string ToolboxIcon => "\uf03e";

        public int Interval {
            get { return this.Retrieve(x => x.Interval, () => 3000); }
            set { this.Store(x => x.Interval, value); }
        }

        public bool Controls {
            get { return this.Retrieve(x => x.Controls, () => true); }
            set { this.Store(x => x.Controls, value); }
        }

        public bool Indicators {
            get { return this.Retrieve(x => x.Indicators, () => true); }
            set { this.Store(x => x.Indicators, value); }
        }

        public string Pause {
            get { return this.Retrieve(x => x.Pause); }
            set { this.Store(x => x.Pause, value); }
        }

        public bool Wrap {
            get { return this.Retrieve(x => x.Wrap); }
            set { this.Store(x => x.Wrap, value); }
        }

        public bool Keyboard {
            get { return this.Retrieve(x => x.Keyboard); }
            set { this.Store(x => x.Keyboard, value); }
        }

        public string SlidesData {
            get { return this.Retrieve(x => x.SlidesData); }
            set { this.Store(x => x.SlidesData, value); }
        }
    }
}
```

Nothing we haven't seen before; just an element class with a bunch of properties. We'll allow the user to change these properties from the element editor dialog, and use the configured values when rendering the HTML such that it works with Bootstrap Carousel.

### The SlideShowDriver Class
Next, we need a driver that will handle displaying the element editor, storing the posted values back into the element, and rendering each slide object into an actual slide shape using the layout APIs.

As mentioned, each slide is a "layout". What I mean by that is that each slide simply stores a collection of elements, representing an hierarchy of elements, "layout" for short.
We already know how to serialize individual layouts to JSON strings. What we need now is a way to store a list of serialized layouts. One easy way is to simply store the slides as JSON, where each slide consists of a serialized Layout string and any selected TemplateId (since the layout editor enables users to select a layout template).

To formalize the structure, it's a good idea to create a class that represents a slide as follows:

```
namespace OffTheGrid.Demos.Layouts.Models {
    public class Slide {
        public string LayoutData { get; set; }
        public int? TemplateId { get; set; }
    }
}
```

To help with the serialization, we'll create the following service:

```
using System.Collections.Generic;
using Orchard;

namespace OffTheGrid.Demos.Layouts.Models {
    public interface ISlidesSerializer : IDependency {
        string Serialize(IEnumerable<Slide> value);
        IEnumerable<Slide> Deserialize(string value);
    }
}
```

This serialization service can serialize a list of slides to and from JSON strings. Its implementation is as follows:

```
using System;
using System.Collections.Generic;
using System.Linq;
using Newtonsoft.Json;
using Orchard;

namespace OffTheGrid.Demos.Layouts.Models {

    public class SlidesSerializer : Component, ISlidesSerializer {
        public string Serialize(IEnumerable<Slide> value) {
            return JsonConvert.SerializeObject(value.ToList());
        }

        public IEnumerable<Slide> Deserialize(string value) {
            if (String.IsNullOrWhiteSpace(value))
                return Enumerable.Empty<Slide>();

            return JsonConvert.DeserializeObject<List<Slide>>(value);
        }
    }
}
```

The `SlidesSerializer` is mostly just a thin wrapper around the `JsonConvert` utility class.

With that in place, it's time to define the view model that we'll use to handle the position of each slide, as well as the Bootstrap Carousel slide show player settings. The following is a list of the Bootstrap Carousel options I'd like the user to be able to configure:

<table>
<thead>
   <tr>
      <th>Option</th>
	  <th>Description</th>
   </tr>
</thead>
<tbody>
   <tr>
      <td><strong>Interval</strong></td>
	  <td>The time to wait before moving to the next slide.</td>
   </tr>
   <tr>
      <td><strong>Pause</strong></td>
	  <td>The event name to use to cause the carousel to pause. Currently only one such event is supported: "hover".</td>
   </tr>
   <tr>
      <td><strong>Wrap</strong></td>
	  <td>Whether to "wrap around", or start from the beginning, when the last slide has been displayed. If set to false, the carousel simply stops. If set to true, the carousel displays the first slide after the last one has been displayed.</td>
   </tr>
   <tr>
      <td><strong>Keywboard</strong></td>
	  <td>Whether the user can use the arrow keys to move to the next or previous slide.</td>
   </tr>
   <tr>
      <td><strong>Controls</strong></td>
	  <td>Whether to display the left and right arrow buttons that can be clicked to move to the next or previous slide.</td>
   <tr>
      <td><strong>Indicators</strong></td>
	  <td>Whether to display indicators. Indicators visualize the number of slides using small circles, and the current slide index is visualized by highlighting the circle at that index. Also, it provides a way for the user to move to a specific slide by clicking an indicator.</td>
   </tr>
   </tr>
</tbody>
</table>

In order to be able to pass the configuration as well as the slide positions (remember, the user will be able to drag & drop the slides into position) to and from the view, we'll define the following view model:

```
using System.Collections.Generic;
using OffTheGrid.Demos.Layouts.Elements;

namespace OffTheGrid.Demos.Layouts.ViewModels {
    public class SlideShowViewModel {
        public SlideShow Element { get; set; }
        public string Session { get; set; }
        public IList<dynamic> Slides { get; set; }
        public IList<int> Indices { get; set; }
        public string SlidesData { get; set; }
        public int Interval { get; set; }
        public bool Controls { get; set; }
        public bool Indicators { get; set; }
        public string Pause { get; set; }
        public bool Wrap { get; set; }
        public bool Keyboard { get; set; }
    }
}
```

The `Element` and `Slides` properties are "view only", which means we'll set them from the driver, but we won't expect any data back to model bind these properties. They're just information that our views can use. The `Slides` property is a list of "rendered" slides, which means that each Layout stored in a slide is rendered using the `ILayoutManager.RenderLayout` method.

The following code shows the complete implementation for the driver:

```
using Orchard.Layouts.Framework.Drivers;
using OffTheGrid.Demos.Layouts.ViewModels;
using Orchard.Layouts.Framework.Display;
using System.Collections.Generic;
using OffTheGrid.Demos.Layouts.Models;
using System.Linq;
using Orchard.Layouts.Services;
using Orchard.ContentManagement;

namespace OffTheGrid.Demos.Layouts.Elements {
    public class SlideShowDriver : ElementDriver<SlideShow> {
        private ILayoutManager _layoutManager;
        private ISlidesSerializer _slidesSerializer;

        public SlideShowDriver(ILayoutManager layoutManager, ISlidesSerializer slidesSerializer) {
            _layoutManager = layoutManager;
            _slidesSerializer = slidesSerializer;
        }

        protected override EditorResult OnBuildEditor(SlideShow element, ElementEditorContext context) {
            var slides = RetrieveSlides(element).ToList();
            var slideShapes = DisplaySlides(slides, context.Content).ToList();
            var viewModel = new SlideShowViewModel {
                Element = element,
                Session = context.Session,
                Slides = slideShapes,
                Controls = element.Controls,
                Indicators = element.Indicators,
                Interval = element.Interval,
                Keyboard = element.Keyboard,
                Pause = element.Pause,
                Wrap = element.Wrap
            };

            if (context.Updater != null) {
                if (context.Updater.TryUpdateModel(viewModel, context.Prefix, null, new[] { "Element", "Session", "Slides" })) {
                    var currentSlides = slides;
                    var newSlides = new List<Slide>(currentSlides.Count);

                    newSlides.AddRange(viewModel.Indices.Select(index => currentSlides[index]));
                    StoreSlides(element, newSlides);

                    element.Controls = viewModel.Controls;
                    element.Indicators = viewModel.Indicators;
                    element.Interval = viewModel.Interval;
                    element.Keyboard = viewModel.Keyboard;
                    element.Pause = viewModel.Pause;
                    element.Wrap = viewModel.Wrap;
                }
            }

            var slidesEditor = context.ShapeFactory.EditorTemplate(TemplateName: "Elements/SlideShow.Slides", Prefix: context.Prefix, Model: viewModel);
            var playerEditor = context.ShapeFactory.EditorTemplate(TemplateName: "Elements/SlideShow.Player", Prefix: context.Prefix, Model: viewModel);

            slidesEditor.Metadata.Position = "Slides:0";
            playerEditor.Metadata.Position = "Player:1";

            return Editor(context, slidesEditor, playerEditor);
        }

        protected override void OnDisplaying(SlideShow element, ElementDisplayingContext context) {
            var slides = RetrieveSlides(element);
            context.ElementShape.Slides = DisplaySlides(slides, context.Content).ToList();
        }

        private IEnumerable<dynamic> DisplaySlides(IEnumerable<Slide> slides, IContent content) {
            return slides.Select(x => _layoutManager.RenderLayout(x.LayoutData, content: content));
        }

        private IEnumerable<Slide> RetrieveSlides(SlideShow element) {
            var slidesData = element.SlidesData;
            return _slidesSerializer.Deserialize(slidesData);
        }

        private void StoreSlides(SlideShow element, IEnumerable<Slide> slides) {
            element.SlidesData = _slidesSerializer.Serialize(slides);
        }
    }
}
```

Nothing too complex, really. It contains code to save and load `Slide` objects from a given `SlideShow` element as well as to render them using the `ILayoutManager`. One interesting part is how we're modelbinding and then using a list called `Indices`. That list represens a list of slide indexes. When the user rearranges slides on the client, we'll get back an array of the newly positioned indexes. The driver simply uses this new order to create a new list of slides by that order:

```
newSlides.AddRange(viewModel.Indices.Select(index => currentSlides[index]));
```

The `OnBuildEditor` method builds to `EditorTemplate` shapes: one to handle the slides, and another to handle the Bootstrap Carousel player configuration. Both editor templates use the same view model.

The `OnDisplaying` method is very simple. All it does is get the list of `Slide` objects and then render them. We'll look at the `Elements/SlideShow.cshtml` template shortly to see how we're rendering these slide shapes in combination with Bootstrap related attributes.

### The SlideShow Editor Views
Since our driver returns two editor template shapes, we'll have to provide two views:

- EditorTemplates/Elements/SlideShow.Slides.cshtml
- EditorTemplates/Elements/SlideShow.Player.cshtml

The first one handles the management of slides, and looks like this:

```
@model OffTheGrid.Demos.Layouts.ViewModels.SlideShowViewModel
@{
    Style.Include("~/Modules/Orchard.Layouts/Styles/default-grid.css");
    Style.Include("SlideShow.Admin.css", "SlideShow.Admin.min.css");
    Script.Require("jQuery");
    Script.Require("jQueryUI_Sortable");
    Script.Require("ShapesBase");
    Script.Include("SlideShow.Admin.js", "SlideShow.Admin.min.js");
}
@{
    var slides = Model.Slides.ToArray();

    if (!slides.Any()) {
        <div class="message message-Information">@T("No slides have been added yet.")</div>
    }
    else {
        <div class="slides-wrapper interactive">
            <div class="dirty-message message message-Warning">@T("You have unsaved changes.")</div>
            <div class="group">
                <ul class="slides">
                    @{ var slideIndex = 0;}
                    @foreach (var slide in slides) {
                        <li>
                            <input type="hidden" name="@Html.FieldNameFor(m => m.Indices)" value="@slideIndex" />
                            <div class="slide-wrapper">
                                <div class="slide-preview">
                                    @Display(slide)
                                </div>
                            </div>
                            <div class="actions">
                                @Html.ActionLink(T("Edit").Text, "Edit", "SlideAdmin", new { session = Model.Session, index = slideIndex, area = "OffTheGrid.Demos.Layouts" }, null)
                                @T(" | ")
                                @Html.ActionLink(T("Delete").Text, "Delete", "SlideAdmin", new { Session = Model.Session, index = slideIndex, area = "OffTheGrid.Demos.Layouts" }, new { data_unsafe_url = T("Are you sure you want to delete this slide?") })
                            </div>
                        </li>
                        ++slideIndex;
                    }
                </ul>
            </div>
        </div>
    }
    @Html.ActionLink(T("Create Slide").Text, "Create", "SlideAdmin", new { Session = Model.Session, area = "OffTheGrid.Demos.Layouts" }, new { @class = "button" })
}
```

The reason we're including `"default-grid.css"` from the Layouts module is because we are rendering each individual slide, which are in fact layouts.

Notice that I'm passing in the `Session` value when generating the links to the *Edit* and *Delete* actions of the (yet to be created) `SlideAdminController`. The controller will use this value to read the element's state from the `IObjectStore`.

We're also including a custom stylesheet called `"SlideShow.Admin.css"` and a custom script called `"SlideShow.Admin.js"`. Both files are Gulp output files, based on the following asset files:

```
Assets
	Elements
		SlideShow
			Admin.js
			Admin.less
```

Assets.json contains the following entries to turn the asset files into the mentioned output files:

```
{
    "inputs": [ "Assets/Elements/SlideShow/Admin.less" ],
    "output": "Styles/SlideShow.Admin.css"
},
{
    "inputs": [ "Assets/Elements/SlideShow/Admin.js" ],
    "output": "Scripts/SlideShow.Admin.js"
},
```

The contents of Admin.less:

```
.slides-wrapper
{
    margin-bottom: 2em;
    position: relative;
}

    .slides-wrapper .sortable-placeholder
    {
        display: block;
        height: 125px;
    }

    .slides-wrapper .dirty-message
    {
        display: none;
    }

    .slides-wrapper ul
    {
        display: none;
        float: left;
        list-style: none;
        margin: 0;
        padding: 0;
    }

        .slides-wrapper ul.slides li
        {
            float: left;
            width: 150px;
            margin: 0 1em 0.2em 0;
            border: none;
            list-style: none;
        }

            .slides-wrapper ul.slides li .slide-wrapper
            {
                background: #ebebeb;
                border: 1px solid #ebebeb;
            }

            .slides-wrapper.interactive ul.slides li:hover .slide-wrapper
            {
                border-color: #aeaeae;
            }

            .slides-wrapper ul.slides li img
            {
                vertical-align: middle;
                display: block;
                max-width: 100%;
                height: auto;
            }

```

And the contents of Admin.js:

```
(function ($) {
    // Slides preview.
    var applyCssScale = function (element, scale, translate) {
        var browserPrefixes = ["", "-ms-", "-moz-", "-webkit-", "-o-"],
            offset = ((1 - scale) * 50) + "%",
            scaleString = (translate !== false ? "translate(-" + offset + ", -" + offset + ") " : "") + "scale(" + scale + "," + scale + ")";
        $(browserPrefixes).each(function () {
            element
                .css(this + "transform", scaleString);
        });
        element
            .data({ scale: scale })
            .addClass("scaled");
    };

    var scaleSlides = function () {
        $(".slides")
            .css("display", "block")
            .each(function () {
                var slideshow = $(this),
                    slide = slideshow.find(".slide-preview"),
                    parent = slide.parent(),
                    width = 150,
                    height = 150,
                    boundingDimension = 150,
                    slideStyle = slide.attr("style");

                if ((slideStyle != null && slideStyle.indexOf("width:") == -1)) width = 1024;
                if ((slideStyle != null && slideStyle.indexOf("height:") == -1)) height = 768;

                slide.css({
                    width: width + "px",
                    height: height + "px",
                    position: "absolute"
                });
                var scaledForWidth = width > height,
                    largestDimension = (scaledForWidth ? width : height),
                    scale = boundingDimension / largestDimension;

                parent.css({
                    width: Math.floor(width * scale) + "px",
                    height: Math.floor(height * scale) + "px",
                    position: "relative",
                    overflow: "hidden"
                });

                applyCssScale(slide, scale);
                slideshow.parent(".slides-wrapper").css("overflow", "visible");
            });
    };

    $(window).load(function () {
        scaleSlides();
    });

    // Sortable slides.
    $(function () {
        $(".slides-wrapper.interactive").each(function () {
            var wrapper = $(this);

            var showChangedMessage = function () {
                wrapper.find(".dirty-message").show();
            };

            var onSlideIndexChanged = function (e, ui) {
                showChangedMessage();
            };

            var slidesList = wrapper.find("ul.slides");
            slidesList.sortable({
                placeholder: "sortable-placeholder",
                stop: onSlideIndexChanged
            });
            slidesList.disableSelection();
        });
    });
})(jQuery);
```

The first part of the script takes care of resizing each slide layout. I kind of understand what the code does, but I didn't write it myself; I took it from my good friend Bertrand Leroy's work on the **Onestop.SlideShow** gallery module.

The second part of the script takes care of turning the rendered list of slides into a sortable list, which enables the user to drag & drop the slides into position.

The *SlideShow.Player.cshtml* view is much simpler:

```
@model OffTheGrid.Demos.Layouts.ViewModels.SlideShowViewModel
@{
    var pauseList = new[] { "hover" };
    var pauseOptions = pauseList.Select(x => new SelectListItem { Text = x, Value = x, Selected = x == Model.Pause }).ToList();
}
<fieldset>
    <div class="form-group">
        @Html.LabelFor(m => m.Interval, T("Interval"))
        @Html.TextBoxFor(m => m.Interval, new { @class = "text medium" })
        @Html.Hint(T("The interval in milliseconds to wait before showing the next slide."))
    </div>
    <div class="form-group">
        @Html.CheckBoxFor(m => m.Controls)
        @Html.LabelFor(m => m.Controls, T("Show Controls").Text, new { @class = "forcheckbox" })
        @Html.Hint(T("Check this option to show the next and prev control buttons."))
    </div>
    <div class="form-group">
        @Html.CheckBoxFor(m => m.Indicators)
        @Html.LabelFor(m => m.Indicators, T("Show Indicators").Text, new { @class = "forcheckbox" })
        @Html.Hint(T("Check this option to the show indicators."))
    </div>
    <div class="form-group">
        @Html.CheckBoxFor(m => m.Wrap)
        @Html.LabelFor(m => m.Wrap, T("Wrap").Text, new { @class = "forcheckbox" })
        @Html.Hint(T("Whether the carousel should cycle continuously or have hard stops."))
    </div>
    <div class="form-group">
        @Html.CheckBoxFor(m => m.Keyboard)
        @Html.LabelFor(m => m.Keyboard, T("Keyboard").Text, new { @class = "forcheckbox" })
        @Html.Hint(T("Whether the carousel should react to keyboard events."))
    </div>
    <div class="form-group">
        @Html.LabelFor(m => m.Pause, T("Pause"))
        @Html.DropDownListFor(m => m.Pause, pauseOptions, T("never").ToString())
        @Html.Hint(T("Pauses the cycling of the carousel on mouseenter and resumes the cycling of the carousel on mouseleave."))
    </div>
</fieldset>
```

Next, let's have a look at the *Elements/SlideShow.cshtml* template itself as well as its alternate *Elements/SlideShow.design.cshtml*. The latter one will be used when the SlideShow element is rendered in the Layout editor:

```
@{
    var slides = (IList<dynamic>)Model.Slides;
}
@if (!slides.Any()) {
    <div class="message message-Information">@T("No slides have been added yet.")</div>
}
else {
    var firstSlide = slides.First();
    <div class="slides-wrapper img-responsive">
        @Display(firstSlide)
    </div>
}
```

As you can see, all we're doing here is getting the first rendered slide shpe, if any, and render that one.

The *Elements/SlideShow.cshtml* is more interesting, as it contains the actual markup that we need for the Bootstrap Carousel component to work:

```
@using OffTheGrid.Demos.Layouts.Elements
@{
    Style.Include("SlideShow.css", "SlideShow.min.css");
    Script.Require("jQuery");
    Script.Include("SlideShow.js", "SlideShow.min.js");

    var element = (SlideShow)Model.Element;
    var slides = (IList<dynamic>)Model.Slides;
    var slideShowId = String.Format("Slideshow-{0}", DateTime.Now.Ticks);
}
@if (slides.Any()) {
    <div id="@slideShowId" class="carousel slide"
         data-ride="carousel"
         data-interval="@element.Interval"
         data-wrap="@element.Wrap.ToString().ToLower()"
         data-keyboard="@element.Keyboard.ToString().ToLower()"
         data-pause="@element.Pause">
        @if (element.Indicators) {
                <!-- Indicators -->
            <ol class="carousel-indicators">
                @for (var i = 0; i < slides.Count; i++) {
                    <li data-target="#@slideShowId" data-slide-to="@i" @if (i == 0) { <text> class="active" </text>  }></li>
                }
            </ol>
        }

        <!-- Wrapper for slides -->
        <div class="carousel-inner" role="listbox">
            @for (var i = 0; i < slides.Count; i++) {
                <div class="item @if(i == 0){<text>active</text>}">
                    @Display(slides[i])
                </div>
            }
        </div>

        @if (element.Controls) {
                <!-- Controls -->
            <a class="left carousel-control" href="#@slideShowId" role="button" data-slide="prev">
                <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
                <span class="sr-only">@T("Previous")</span>
            </a>
                <a class="right carousel-control" href="#@slideShowId" role="button" data-slide="next">
                    <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
                    <span class="sr-only">@T("Next")</span>
                </a>
        }
    </div>
}
```

> Both the *SlideShow.css* and *SlideShow.js* are actually Bootstrap's Carousel component's script and CSS that I stored in the Assets folder and processed using Gulp, but those files are too long to show here in the book. Instead. If you're coding along, you can get the files from the accompanying demo source code (see the **OffTheGrid.Demos.Layouts** module.

I essentially copied the markup from the http://getbootstrap.com website and replaced some of it with some Razor code to conditionally render the indicators and controls for example. As for the slides, all I had to do is iterate over each of them and then render:

```
@for (var i = 0; i < slides.Count; i++) {
    <div class="item @if(i == 0){<text>active</text>}">
        @Display(slides[i])
    </div>
}
```   

At this point, only one major thing is missing to make it all work: actually being able to create and edit individual slides.

### The SlideAdminController Class
In order to enable the user to create, edit and delete individual slides, we will need a controller to handle those actions. When creating and editing a slide, we will have the controller reuse the layout editor.

Create a new controller called **SlideAdminController** with the following skeleton code:

```
using System.Web.Mvc;
using OffTheGrid.Demos.Layouts.ViewModels;
using Orchard.UI.Admin;

namespace OffTheGrid.Demos.Layouts.Controllers {
    [Admin]
    public class SlideAdminController : Controller {

        public ActionResult Create(string session) {
        }

        [HttpPost]
        [ValidateInput(false)]
        public ActionResult Create(SlideEditorViewModel viewModel) {
        }

        public ActionResult Edit(string session, int index) {
        }

        [HttpPost]
        [ValidateInput(false)]
        public ActionResult Edit(SlideEditorViewModel viewModel, int index) {
        }

        [HttpPost]
        public ActionResult Delete(string session, int index) {
        }
    }
}
```

I added action methods for *Create*, *Edit* and *Delete*, including the "post-back" variants.

#### The Create Action
The first action we'll implement is the `Create` action, which will render the layout editor and handle post-back. Notice that we're expecting the `session` key value to be provided by the slide show element editor.

```
public ActionResult Create(string session) {
    var viewModel = new SlideEditorViewModel {
        Session = session,
        LayoutEditor = _layoutEditorFactory.Create(null, _objectStore.GenerateKey())
    };
    return View(viewModel);
}
``` 

What we did here is instantiate a new `SlideEditorViewModel` and a `LayoutEditor` using the layout editor factory API, so be sure to inject an `ILayoutEditorFactory`.

The `SlideEditorViewModel` is defined as follows:

```
using Orchard.Layouts.ViewModels;

namespace OffTheGrid.Demos.Layouts.ViewModels {
    public class SlideEditorViewModel {
        public string Session { get; set; }
        public int? SlideIndex { get; set; }
        public LayoutEditor LayoutEditor { get; set; }
    }
}
```

The `SlideIndex` is needed when we edit an individual slide. Since the object store stores the entire slide show's element data, including its slides, we need a way to know which slide we need to edit. Receiving the slide index helps solve that puzzel.

The **Create.cshtml** view looks like this:

```
@model OffTheGrid.Demos.Layouts.ViewModels.SlideEditorViewModel
@{
    Layout.Title = T("Create Slide");
}
@using (Html.BeginFormAntiForgeryPost()) {
	@Html.HiddenFor(m => m.Session)
	@Html.HiddenFor(m => m.SlideIndex)
	@Html.EditorFor(m => m.LayoutEditor)
    <fieldset>
        <button type="submit" name="submit.Save" value="submit.Save">@T("Create")</button>
        @Html.ActionLink(T("Cancel").ToString(), "Edit", "Element", new { session = Model.Session, area = "Orchard.Layouts" }, new { @class = "button" })
    </fieldset>
}
```


To implement the post-back, we need to do the following things:

- Get a reference to the slide show element's data being edited and activate the element.
- Get the slides from the slide show element so that we can add a new slide to it.
- Serialize the list slides and store it back into the slide show element.
- Serialize and store the updated slide show element into the object store.
- Redirect back to the slide show element edit screen.

The implementation looks like this:

```
[HttpPost]
[ValidateInput(false)]
public ActionResult Create(SlideEditorViewModel viewModel) {
    var slideShowSessionState = _objectStore.Get<ElementSessionState>(session);
    var elementData = ElementDataHelper.Deserialize(slideShowSessionState.ElementData);
    var slideShow = _elementManager.ActivateElement<SlideShow>(x => x.Data = elementData);
    var slides = _slidesSerializer.Deserialize(slideShow.SlidesData).ToList();
    var slide = new Slide();
    var slideLayout = _layoutModelMapper.ToLayoutModel(viewModel.LayoutEditor.Data, DescribeElementsContext.Empty).ToList();
    var recycleBin = (RecycleBin)_layoutModelMapper.ToLayoutModel(viewModel.LayoutEditor.RecycleBin, DescribeElementsContext.Empty).First();
    var context = new LayoutSavingContext {
        Updater = this,
        Elements = slideLayout,
        RemovedElements = recycleBin.Elements
    };

	// Trigger the Saving and Removing events on the elements.
    _elementManager.Saving(context);
    _elementManager.Removing(context);

    // Create a new slide.
    var slide = new Slide {
		TemplateId = viewModel.LayoutEditor.TemplateId;
    	LayoutData = _layoutSerializer.Serialize(slideLayout);
	};
    
	// Add the slide.
	slides.Add(slide);

    // Serialize the in-memory list and assign it back to the SlidesData property of the SlideShow element.
    slideShow.SlidesData = _slidesSerializer.Serialize(slides);

    // Serialize the slide show element itself.
    slideShowSessionState.ElementData = ElementDataHelper.Serialize(slideShow.Data);

    // Replace the slide show in the object store with the updated data.
    _objectStore.Set(session, slideShowSessionState);

	// Display a success notification.
	_notifier.Information(T("That slide has been added."));

	// Redirect back to the element editor screen.
	return RedirectToAction("Edit", "Element", new { session = viewModel.Session, area = "Orchard.Layouts" });
}
```

Notice that this method relies on a few new services that we haven't declared yet:
- `_elementManager` of type `IElementManager`
- `_slidesSerializer` of type `ISlidesSerializer`
- `_layoutModelMapper` of type `ILayoutModelMapper`
- `_layoutSerializer` of type `ILayoutSerializer`
- `_objectStore` of type `IObjectStore`
- `INotifier` of type `INotifier`
- `T` of type `Localizer`

Make sure to inject these services and store them in private fields. Except for the `T` property of course, that one is property-injected by Orchard rather than constructor-injected.

#### The Edit Action
The **Edit** action is very similiar to the **Create** action, the primary differences being:

1. The view model needs to be initialized with existing layout data.
2. When posting back, instead of adding a slide, we need to get a reference to an existing slide (by index).

The Edit action looks like this:

```
public ActionResult Edit(string session, int index) {
    // Get the element's state from the object store.
    var slideShowSessionState = _objectStore.Get<ElementSessionState>(session);

    // Deserialize the element's state into a dictionary.
    var elementData = ElementDataHelper.Deserialize(slideShowSessionState.ElementData);

    // Activate a new SlideShow element and initialize it with the state dictionary.
    var slideShow = _elementManager.ActivateElement<SlideShow>(x => x.Data = elementData);

    // Deserialize the slide show slides data into a list of actual Slide objects.
    var slides = _slidesSerializer.Deserialize(slideShow.SlidesData).ToList();

    // Get a reference to the specified slide by index.
    var slide = slides[index];

    // Create the view model and initialize the layout editor with the slide's layout data.
    var viewModel = new SlideEditorViewModel {
        SlideIndex = index,
        Session = session,
        LayoutEditor = _layoutEditorFactory.Create(slide.LayoutData, _objectStore.GenerateKey(), slide.TemplateId)
    };

    // Return the view.
    return View(viewModel);
}
```

The Edit view looks very similar to the Create view:

```
@model OffTheGrid.Demos.Layouts.ViewModels.SlideEditorViewModel
@{
    Layout.Title = T("Edit Slide");
}
@using (Html.BeginFormAntiForgeryPost()) {
    @Html.HiddenFor(m => m.Session)
	@Html.HiddenFor(m => m.SlideIndex)
	@Html.EditorFor(m => m.LayoutEditor)
    <fieldset>
        <button type="submit" name="submit.Save" value="submit.Save">@T("Save")</button>
        @Html.ActionLink(T("Cancel").ToString(), "Edit", "Element", new { session = Model.Session, area = "Orchard.Layouts" }, new { @class = "button" })
    </fieldset>
}
```

The only real difference is the title being set.

The post-back version of the Edit action, too, looks very similar to the Create action method:

```
[HttpPost]
[ValidateInput(false)]
public ActionResult Edit(SlideEditorViewModel viewModel) {
    var slideShowSessionState = _objectStore.Get<ElementSessionState>(session);
    var elementData = ElementDataHelper.Deserialize(slideShowSessionState.ElementData);
    var slideShow = _elementManager.ActivateElement<SlideShow>(x => x.Data = elementData);
    var slides = _slidesSerializer.Deserialize(slideShow.SlidesData).ToList();
    var slide = slides[viewModel.SlideIndex.Value];
    var slideLayout = _layoutModelMapper.ToLayoutModel(viewModel.LayoutEditor.Data, DescribeElementsContext.Empty).ToList();
    var recycleBin = (RecycleBin)_layoutModelMapper.ToLayoutModel(viewModel.LayoutEditor.RecycleBin, DescribeElementsContext.Empty).First();
    var context = new LayoutSavingContext {
        Updater = this,
        Elements = slideLayout,
        RemovedElements = recycleBin.Elements
    };

	// Trigger the Saving and Removing events on the elements.
    _elementManager.Saving(context);
    _elementManager.Removing(context);

	// Update the slide.
    slide.TemplateId = viewModel.LayoutEditor.TemplateId;
    slide.LayoutData = _layoutSerializer.Serialize(slideLayout);

    // Serialize the in-memory list and assign it back to the SlidesData property of the SlideShow element.
    slideShow.SlidesData = _slidesSerializer.Serialize(slides);

    // Serialize the slide show element itself.
    slideShowSessionState.ElementData = ElementDataHelper.Serialize(slideShow.Data);

    // Replace the slide show in the object store with the updated data.
    _objectStore.Set(session, slideShowSessionState);

	// Display a success notification.
	_notifier.Information(T("That slide has been updated."));

	// Redirect back to the element editor screen.
	return RedirectToAction("Edit", "Element", new { session = viewModel.Session, area = "Orchard.Layouts" });
}
```

#### The Delete Action
The **Delete** action is responsible for removing a slide at the specified index. In terms of implementation, it is similar to the Create and Edit methods, except for the fact that don;t need to deal with the view model. All we need to do is materialize the slide show element, remove a slide from its list, then serialize and store it back into the object store. The implementation looks like this:

```
[HttpPost]
public ActionResult Delete(string session, int index) {
    var slideShowSessionState = _objectStore.Get<ElementSessionState>(session);
    var elementData = ElementDataHelper.Deserialize(slideShowSessionState.ElementData);
    var slideShow = _elementManager.ActivateElement<SlideShow>(x => x.Data = elementData);
    var slides = _slidesSerializer.Deserialize(slideShow.SlidesData).ToList();

    // Delete the slide at the specified index.
    slides.RemoveAt(index);

    // Serialize the in-memory list and assign it back to the SlidesData property of the SlideShow element.
    slideShow.SlidesData = _slidesSerializer.Serialize(slides);

    // Serialize the slide show element itself.
    slideShowSessionState.ElementData = ElementDataHelper.Serialize(slideShow.Data);

    // Replace the slide show in the object store with the updated data.
    _objectStore.Set(session, slideShowSessionState);

	// Redirect back to the element editor. The ReturnUrl contains the session key.
    _notifier.Information(T("That slide has been deleted."));

	// Redirect back to the element editor screen.
	return RedirectToAction("Edit", "Element", new { session = viewModel.Session, area = "Orchard.Layouts" });
}
```    

And that's really all there is to it! With this code, you can now drag & drop slide show elements onto the canvas and manage individual slides.

### Suggestions for Improvements
Although being able to craft individual slides using the layout editor is powerful, it is also potentially cumbersome in cases where all you really need is a list of simple images. In that case, it would be much nicer if the user had an option to simply select one or more images from the media library, and automatically generate a list of slides, where each slide would still be based on a layout, but with one single `Image` element. This has the benefit of quickly creating slide shows while retaining the ability to edit individual ones.

Another improvement could be the decoupling of the slide show player. Although the theme could use completely different carousel scripts rather than Bootstrap Carousel, tight now the slide show element only supports Bootstrap Carousel-specific settings. You could consider makeing this set of settings more universal, which has the upside of potentially supporting any carousel script out there, but also has the clear disadvantage that some settings may not be applicable to certain players, while certain advanced players provide settings that the user cannot configure. So a good way might be to introduce some notion of *Players*, where the module declares a *ISlideShowPlayer* class for example, which provides its own editing and display UI. Then from the slide show element editor, the user would be able to first choose the slide show player they want to use and configure that one. Even better would be the notion of *Player Profiles*, where the user would not select a player, but a profile, which is basically a pre-configured player.

Similar to decoupling the actual player, another improvement could be to decouple the source from which the slide show element gets its slides from. For example, imagine a **Query** from the **Projections** module that yields a set of content items, which you want to display as a slide show. Or another provider based on a **Twitter feed**.          

### Summary ###
In this chapter, we put various layout APIs to the test and implemented a much more advanced element editor than we have seen before. We implemented a Slide Show element that enables the user to create and edit individual slides, each slide being very dynamic in terms of content thanks to the reusability of the layout editor.