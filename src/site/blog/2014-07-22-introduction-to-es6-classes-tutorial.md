---
permalink: /blog/introduction-to-es6-classes-tutorial

title: An introduction to ES6 classes.
intro: Today we'll look at another new feature of ES6, classes.
date: 2014-07-22
---

## Support

ES6 support varies across environments and platforms, implementations get updated all the time and it's important to also note that the spec is in draft, so everything below has the potential to change. I recommend using [The ES6 Compatability Table](http://kangax.github.io/es5-compat-table/es6/) to see the current state of affairs.

## Traceur

All the code examples seen in this post were run through [Traceur](https://github.com/google/traceur-compiler), a tool for compiling ES6 code into ES5 code which has much better browser support. The beauty of Traceur is that it allows you to write ES6, compile it and use the result in environments where ES6 features are not implemented. Traceur is installed through npm:

```shell
npm install --global traceur
```

And then used on a source file like so:

```shell
traceur --out build.js --script my_source_file.js
```

You'll also need to include the Traceur runtime in your HTML. The runtime comes as part of the Node module, and is found in the `bin` directory, called `traceur-runtime.js` directory. If you'd like to see an example of this, you can [check out the sample repo on GitHub](https://github.com/javascript-playground/es6-classes).

## Classes

ES6 classes are syntactical sugar over the Objects and prototypes that we're used to working with. They simply offer a much nicer, cleaner and clearer syntax for creating these objects and dealing with inheritance.

To show this in action we're going to build our own small (and very simplified) framework for building web applications to demonstrate using classes. We're going to have two classes, one to represent a view, and another to represent a model. Here's the `View` class:

```js
class View {
  constructor(options) {
    this.model = options.model;
    this.template = options.template;
  }

  render() {
    return _.template(this.template, this.model.toObject());
  }
}
```

Notice how we still set properties through `this.property`, but defining methods on the class is done very differently to how you might be used to. Not a `function` keyword in sight! Functions are defined by putting their name, followed by any arguments within brackets, and then a set of braces. That's it. Our view class is very simple, and provides just a simple `render()` method, which takes the template (I'm using Underscore here for templating) and the model object and then returns the compiled template.

```js
class Model {
  constructor(properties) {
    this.properties = properties;
  }

  toObject() {
    return this.properties;
  }
}
```

Our `Model` class is equally as simple. It stores all the properties and provides the `toObject` method that gives access to the properties.

We can now use these to output some HTML:

```js
var jack = new Model({
  name: 'jack',
});

var view = new View({
  model: jack,
  template: 'Hello, <%= name %>',
});

console.log(view.render());
```

The classes are instantiated just as they are in the ES5 and below world, with the `new` keyword used. The `constructor` function is called automatically when an instance of the class is created.

If you run the above code (remembering to run it through Traceur), you'll see `"Hello, jack"` logged to the console.

## Extending

Say we have some views where we actually just want the `render` method not to return the compiled template, but to simply just `console.log` the resulting rendered HTML. (This is a contrived example, but stick with me!). We might call this view `LogView`, and we can implement it by extending our regular `View` class. I'll explain the call to `super.render()` shortly.

```js
class LogView extends View {
  render() {
    var compiled = super.render();
    console.log(compiled);
  }
}
```

Using the `extends` keyword to extend a class is a great example of where the simplicity of the class syntax shines. Extending `View` means that `LogView` inherits everything that `View` has. If we were to just have:

```js
class LogView extends View {}
```

Then `LogView` functionality would be effectively identical to `View`.

Instead though, we override the `render` method:

```js
render() {
  var compiled = super.render();
  console.log(compiled);
}
```

We first call `super.render()`. This calls the parent class' `render()` method, and returns the result. Using `super`, you can access methods and properties available on the parent class. This means that the `render` method on the `View` class is first called, and the result is stored in the `compiled` variable. We then simply log out the result.

```js
var jack = new Model({
  name: 'jack',
});

var view = new LogView({
  model: jack,
  template: 'Hello, <%= name %>',
});

view.render();
```

If you rerun Traceur and refresh the browser, you'll still see `Hello, jack` logged to the console, but this time the only `console.log` call was from within the `LogView` class.

## Conclusion

I hope that serves as a nice introduction to ES6 classes. Just because they exist, it doesn't mean that you should immediately seek to change every object in your system to classes, but they certainly have some great use cases.

The code I used in this post is [on GitHub](https://github.com/javascript-playground/es6-classes), so feel free to check it out and have a play around.

_Thanks to [@toddmotto](http://twitter.com/toddmotto) for his help reviewing a draft of this piece._
