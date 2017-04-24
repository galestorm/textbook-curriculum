
# Intro to Backbone Views

## Learning Goals
By the end of this lesson you should be able to:

- Explain what Backbone views are and why they're useful
- Create a view around some data
- Render HTML using a view & an Underscore Template

## What is a View?
Backbone views are kind of middle-people in the Backbone world, filling a similar role to controllers in Rails. A view's job is to coordinate between the data and the DOM. When a DOM event happens, it's the controller's job to handle it and update the data as needed, and when the data changes it's the view's job to modify the DOM to match.

We will first create two views, one for a single task item and a second for the collection of Tasks.  Then we will add event handling for the views.

## Creating A TaskView

To start we create a file in `app/views/task_view.js`

```JavaScript
// src/app/views/task_view.js

import Backbone from 'backbone';
import _ from 'underscore';
import $ from 'jquery';

var TaskView = Backbone.View.extend({
  render: function() {
    // Select the template using jQuery
    var template_text = $('#taskItemTemplate').html();
    // Get an underscore template object
    var template = _.template(template_text);

    this.$el.html(template(this.model.toJSON()));
    return this;
  }
});

export default TaskView;
```

Just like Models and Collections a view extends `Backbone.View`.  This model has 3 important properties, `el`, `model`, and `render`.
-  `el` is an HTML DOM element that by default is an empty `div`.  We use `el` to insert our view into the page when it is rendered.  
	- There is also a corresponding property `$el` which is a jQuery selection of `el`, and you can use jQuery functions on it.
- `model` is the Backbone model which provides the data for the view.  The view's `model` can be a Backbone Model or Collection.  
- `render` is a function called to draw (or redraw) the view.  By convention the render function always returns `this` so that it can be chained with other methods.

## Adding our view to `app.js`

We can then modify our application code to use our view by creating a new `TaskView` in our event handler.

```JavaScript
taskList.on("update", function() {
    $('main').empty();
    taskList.each(function(task) {
      var taskView = new TaskView({
        model: task
      });
      $('main').append(taskView.render().el);
    });
  });
```

In the code above we first cleared out the `main` element in the `html` and then for each Task model, created a new TaskView.  Then we rendered the View and appended the resulting element (`el`) to `main`.

The view now renders, but the buttons no longer work.  Next we will introduce event handling in Views and get the Toggle button working.  

### Check-In

You & your SeatSquad member should now have the basic TaskList displaying and the buttons should no longer function.  Check & verify that you both have it working.  

## Event Handling 

We can add another element to our view to list the event handlers, `events`.  The `events` object matches events to functions.  In the example below the `events` object links the `click` event on any sub-element with the class of `toggle` to an event handler function called `toggle`.  Then in the `toggle` function we change the model's `complete` attribute and then re-render the view.


```Javascript
// src/app/views/task_view.js

import Backbone from 'backbone';
import _ from 'underscore';
import $ from 'jquery';

var TaskView = Backbone.View.extend({
  render: function() {
    // Select the template using jQuery
    var template_text = $('#taskItemTemplate').html();
    // Get an underscore template object
    var template = _.template(template_text);

    this.$el.html(template(this.model.toJSON()));
    return this;
  },
  events: {
    'click .toggle': 'toggle'
  },
  toggle: function() {
    this.model.toggleComplete();
    this.render();
  }
});

export default TaskView;
```

### Check-In

Check & verify that your toggle button is working.  

## Viewing a Collection of Tasks

We can create another view to manage a collection of Tasks, and this view will store the Collection in the model property and render a view for each task and appending the resulting html to `$el`.  

```JavaScript
// src/app/views/task_list_view.js

import Backbone from 'backbone';
import _ from 'underscore';
import $ from 'jquery';
import TaskView from 'app/views/task_view';


var TaskListView = Backbone.View.extend({
  el: $('body'),
  render: function() {
    var collection = this.model;
    collection.each(function(task) {
      var taskView = new TaskView({model: task});

      this.$el.find('main').append(taskView.render().$el);
    }, this);

    return this;
  }
});

export default TaskListView;
```

Then we can use the view in `app.js` to render the collection.

```JavaScript
// src/app.js
import $ from 'jquery';
import _ from 'underscore';

import Task from 'app/models/task';
import TaskList from 'app/collections/task_list';
import TaskView from 'app/views/task_view';
import TaskListView from 'app/views/task_list_view';

var taskData = [ {
  title: "Study JavaScript",
  completed: true
},
{
  title: "Learn Backbone Collections",
  completed: false
},
{
  title: "Take out the trash",
  completed: false
}];

var taskList = new TaskList(taskData);
var taskListView = new TaskListView({model: taskList});


$(document).ready(function() {
  taskListView.render();
});
```

## Adding Event Handlers



## Optimizations

What could we do to improve on this?  There are a few things we could change.  Noice we recreate all the `TaskView`s every time the `TaskListView` is rendered.  We could store them in an array to avoid recreating them.  

### Adding a View
Views are created calling `Backbone.View.extend()`. `extend()` is a Backbone thing, and it returns a constructor function we'll use to instantiate our views. If we were in Ruby we'd say we're making a class that inherits from `Backbone.View`. We'll see `extend()` many times in the coming weeks. The only argument to `extend()` is a JavaScript object containing all the extra things we want our view to have.

Backbone can recognize many things in the argument to `extend`, but the two most important for a view are two functions: `initialize()` and `render()`. Let's see what that looks like.

```javascript
var TaskView = Backbone.View.extend({
  initialize: function(options) {
  },

  render: function() {
    // Enable chained calls
    // This is important enough that we'll leave it in, but
    // we wont talk about it until later.
    return this;
  }
});
```

#### Initialize
Let's start with `initialize()`. This function will be run once when the view is first created. Its job is to get everything ready to go. It takes one argument, `options`, which contains all the stuff the view was created with.

Views are created with the familiar `new` keyword. It will look something like this:

```javascript
var card = new TaskView();
```

Right now our view isn't doing anything, so let's give it some data to keep track of. We'll create each view around one of the tasks in our list.

```javascript
var TaskView = Backbone.View.extend({
  initialize: function(options) {
    this.task = options.task;
  },

  render: function() {
    // Enable chained calls
    return this;
  }
});

$(document).ready(function() {
  var card = new TaskView({task: taskData[0]});
});
```

## What Did We Accomplish?
- Create a basic Backbone view to display a task. It had one function:
  - `initialize()` is run once to set everything up
  - `render()` generates HTML, and may be run many times
- Use the underscore templating engine to separate concerns and clean up our rendering code
- Create a more complex Backbone view to manage our whole application

## Additional Resources
- [Backbone View Documentation](http://backbonejs.org/#View)
- [Backbone Applications Intro to Views](https://addyosmani.com/backbone-fundamentals/#views-1)
- [Underscore documentation](http://underscorejs.org/)
- [SitePoint Underscore tutorial](https://www.sitepoint.com/getting-started-with-underscore-js/)