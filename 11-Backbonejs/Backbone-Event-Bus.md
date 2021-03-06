# An Event Bus

When we want different objects, Views, Models, & Collections to communicate with each-other without tightly integrating them we can construct something called an Event Bus.

Let's look at a [baseline for TaskList in this Codepen](https://codepen.io/adadev/pen/vWovEW?editors=1011). We want the "Current Task" section to display details about the most recently clicked on Task.  How would you do that? Because the two different sections belong to different views, to keep the two views loosely coupled, we can create an object to respond to events.

```javascript
let bus = {};
bus = _.extend(bus, Backbone.Events);
```
Now our `bus` object can trigger and subscribe to Backbone Events.  

Then we can pass this object as a shared resource between each of the views.

```javascript
const listView = new TaskListView({model: myList, bus: bus});
```
And
```javascript
const currentTaskView = new CurrentTaskView({bus: bus});
```

We will also need to create an `initialize` method in the views to keep a reference to our bus object.

```javascript
const TaskListView = Backbone.View.extend({
  tagName: 'ul',
  id: 'todo-list',
  initialize(options) {
    this.bus = options.bus
  },
  render() {
    this.model.each( (item) => {
      const view = new TaskView({model: item, bus: bus});
      this.$el.append(view.render().$el);
    });
    return this;
  }
});
```

Notice that the `TaskListView` sends a copy of the Bus to each TaskView created when rendering the collection.  So we also need to add an initialize method to TaskView.  

```javascript
const TaskView = Backbone.View.extend({
  tagName: "li",
  events: {
    "click": "onClick"
  },
  initialize(options) {
    this.bus = options.bus;
  },
  onClick: function() {
    console.log("Clicked!  Model:  " + this.model.get("title"));
  },
  render() {
    this.$el.html(this.model.get("title"));
    return this;
  }
});
```

What did this do for us?  Now each of the TaskViews and the CurrentTaskView have a reference to the same bus object, which will let them communicate.  

#### Adding an Event to the Bus

Now we can create an Event in the Bus and let the TaskView trigger that event to publish information about that particular task.  

In the CurrentTaskView we can specify an Event on the Bus that we want to listen to.  In the initialize method:

```javascript
// In CurrentTaskView
initialize: function(options) {
  this.bus = options.bus;
  this.listenTo(this.bus, 'taskSelected', (model) => {
    if (model) {
      this.model = model;
      this.render();
    }
  });
},
```

And we can set the TaskView to trigger this event in it's click Event Handler.

```javascript
  // In TaskView
onClick: function() {
  this.bus.trigger("taskSelected", this.model)
},
```

#### So what have we seen here?

The TaskView registered a `click` event and when that event occurs it triggers an event on the bus which causes any callbacks on that event to run.  

TaskView-Click Event --> Bus --> CurrentItemView

So the Bus is really simply a shared object which can register Events and callbacks and acts as a common line of communication.  You can see the working version [here](https://codepen.io/adadev/pen/YEmdKx)  

When we want to establish events on one view which trigger actions in another, we can use this 'bus' technique to let them communicate.  
