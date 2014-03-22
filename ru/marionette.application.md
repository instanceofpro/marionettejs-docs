# Marionette.Application (В процессе перевода)

The `Backbone.Marionette.Application` object is the hub of your composite 
application. It organizes, initializes and coordinates the various pieces of your
app. It also provides a starting point for you to call into, from your HTML 
script block or from your JavaScript files directly if you prefer to go that 
route.

The `Application` is meant to be instantiated directly, although you can extend
it to add your own functionality.

```js
MyApp = new Backbone.Marionette.Application();
```

## Содержание

* [Adding Initializers](#adding-initializers)
* [Application Event](#application-event)
* [Starting An Application](#starting-an-application)
* [Messaging Systems](#messaging-systems)
  * [Event Aggregator](#event-aggregator)
  * [Request Response](#request-response)
  * [Commands](#commands)
* [Regions And The Application Object](#regions-and-the-application-object)
  * [jQuery Selector](#jquery-selector)
  * [Custom Region Type](#custom-region-type)
  * [Custom Region Type And Selector](#custom-region-type-and-selector)
  * [Get Region By Name](#get-region-by-name)
  * [Removing Regions](#removing-regions)

## Adding Initializers

Your application needs to do useful things, like displaying content in your
regions, starting up your routers, and more. To accomplish these tasks and
ensure that your `Application` is fully configured, you can add initializer
callbacks to the application.

```js
MyApp.addInitializer(function(options){
  // do useful stuff here
  var myView = new MyView({
    model: options.someModel
  });
  MyApp.mainRegion.show(myView);
});

MyApp.addInitializer(function(options){
  new MyAppRouter();
  Backbone.history.start();
});
```

These callbacks will be executed when you start your application,
and are bound to the application object as the context for
the callback. In other words, `this` is the `MyApp` object, inside
of the initializer function.

The `options` parameters is passed from the `start` method (see below).

Initializer callbacks are guaranteed to run, no matter when you
add them to the app object. If you add them before the app is
started, they will run when the `start` method is called. If you
add them after the app is started, they will run immediately.

## Application Event

The `Application` object raises a few events during its lifecycle, using the
[Marionette.triggerMethod](./marionette.functions.md) function. These events
can be used to do additional processing of your application. For example, you
may want to pre-process some data just before initialization happens. Or you may
want to wait until your entire application is initialized to start the
`Backbone.history`.

The events that are currently triggered, are:

* **"initialize:before" / `onInitializeBefore`**: fired just before the initializers kick off
* **"initialize:after" / `onInitializeAfter`**: fires just after the initializers have finished
* **"start" / `onStart`**: fires after all initializers and after the initializer events

```js
MyApp.on("initialize:before", function(options){
  options.moreData = "Yo dawg, I heard you like options so I put some options in your options!"
});

MyApp.on("initialize:after", function(options){
  if (Backbone.history){
    Backbone.history.start();
  }
});
```

The `options` parameter is passed through the `start` method of the application
object (see below).

## Запуск приложения

Once you have your application configured, you can kick everything off by 
calling: `MyApp.start(options)`.

This function takes a single optional parameter. This parameter will be passed
to each of your initializer functions, as well as the initialize events. This
allows you to provide extra configuration for various parts of your app, at
initialization/start of the app, instead of just at definition.

```js
var options = {
  something: "some value",
  another: "#some-selector"
};

MyApp.start(options);
```

## Messaging Systems

Application instances have an instance of all three [messaging systems](http://en.wikipedia.org/wiki/Message_passing) of `Backbone.Wreqr` attached to them. This
section will give a brief overview of the systems; for a more in-depth look you are encouraged to read
the [`Backbone.Wreqr` documentation](https://github.com/marionettejs/backbone.wreqr).

### Event Aggregator

The Event Aggregator is available through the `vent` property. `vent` is convenient for passively sharing information between
pieces of your application as events occur.

```js
MyApp = new Backbone.Marionette.Application();

// Alert the user on the 'minutePassed' event
MyApp.vent.on("minutePassed", function(someData){
  alert("Received", someData);
});

// This will emit an event with the value of window.someData every minute
window.setInterval(function() {
  MyApp.vent.trigger("minutePassed", window.someData);
}, 1000 * 60);
```

### Request Response

Request Response is a means for any component to request information from another component without being tightly coupled. An instance of Request Response is available on the Application as the `reqres` property. 

```js
MyApp = new Backbone.Marionette.Application();

// Set up a handler to return a todoList based on type
MyApp.reqres.setHandler("todoList", function(type){
  return this.todoLists[type];
});

// Make the request to get the grocery list
var groceryList = MyApp.reqres.request("todoList", "groceries");

// The request method can also be accessed directly from the application object
var groceryList = MyApp.request("todoList", "groceries");
```

### Commands

Commands is used to make any component tell another component to perform an action without a direct reference to it. A Commands instance is available under the `commands` property of the Application.

Note that the callback of a command is not meant to return a value.

```js
MyApp = new Backbone.Marionette.Application();

MyApp.model = new Backbone.Model();

// Set up the handler to call fetch on the model
MyApp.commands.setHandler("fetchData", function(reset){
  MyApp.model.fetch({reset: reset});
});

// Order that the data be fetched
MyApp.commands.execute("fetchData", true);

// The execute function is also available directly from the application
MyApp.execute("fetchData", true);
```

## Регионы и объект приложения

Объект `Region` может быть добавлен в приложение вызовом метода `addRegions`. 

Существуют три способа добавления региона в объект приложения.

### jQuery-селектор

Первый способ - это определение jQuery-селектора as the value of the region
definition. This will create an instance of a Marionette.Region directly,
and assign it to the selector:

```js
MyApp.addRegions({
  someRegion: "#some-div",
  anotherRegion: "#another-div"
});
```

### Собственный тип региона

Второй способ - это определение собственного типа региона, которому задан селектор:

```js
MyCustomRegion = Marionette.Region.extend({
  el: "#foo"
});

MyApp.addRegions({
  someRegion: MyCustomRegion
});
```

### Собственный тип региона и селектор

Третий способ - это определение собственного типа региона и jQuery-селектора для него с помощью литерала объекта: 

```js
MyCustomRegion = Marionette.Region.extend({});

MyApp.addRegions({

  someRegion: {
    selector: "#foo",
    regionType: MyCustomRegion
  },

  anotherRegion: {
    selector: "#bar",
    regionType: MyCustomRegion
  }

});
```

### Получение региона по его имени

Ссылку на регион можно получить по его имени с помощью метода `getRegion`:

```js
var app = new Marionette.Application();
app.addRegions({ r1: "#region1" });

// r1 === r1Again; true
var r1 = app.getRegion("r1");
var r1Again = app.r1;
```

Доступ к региону через точенную нотацию как к свойству объекта приложения  эквивалентен доступу через метод `getRegion`.

### Удаление регионов

Регионы могут быть удалены с помощью метода `removeRegion`, который принимает в виде строки имя удаляемого региона:

```js
MyApp.removeRegion('someRegion');
```

Removing a region will properly close it before removing it from the
application object.

Для более подробной информации ознакомьтесь с [документацией по регионам](./marionette.region.md).