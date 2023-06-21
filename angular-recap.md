# Angular component recap

Every component in Angular is split into three parts. If we were to imagine an "example" component using the CLI, we would see the following:

- example.component.ts
- example.component.html
- example.component.css

## example.component.ts file

The TS file is where the logic of the component lives. It's a typescript class that will contain any properties, methods, or dependencies that the component requires. In addition to this, it will define the component name, and point to the HTML, and the CSS file.

In our imagined `example.component.ts` we might see code such as this:

```ts
import { Component } from "@angular/core";

@Component({
  selector: "app-example", // defining the name of our component
  templateUrl: "./example.component.html", // pointing the the component's html file
  styleUrls: ["./example.component.css"], // pointing to the component's CSS file
})
export class ExampleComponent {
  greeting = "Hello, world"; // a property containing a greeting we want to display in our component
}
```

## example.component.html file

The HTML file is where our component's HTML lives. It is important to know that there are some _slight_ differences between the HTML we use in Angular when compared to traditional HTML. The Angular HTML is a superset of traditional HTML. This means that any traditional HTML will work in Angular, but there is a chance that some of the HTML you write in Angular wouldn't work in a traditional website. For an overview of the different features we can use in Angular's HTML - [take a look at the Angular Template documentation](https://angular.io/guide/template-syntax)

In our imagined `example.component.html` we might see the following code:

```html
<!-- output the value of the greeting property inside out html -->
<p>{{ greeting }}</p>
<!-- A hardcoded second greeting -->
<p>Nice to meet you!</p>
```

## example.component.css file

The CSS file is where our component's CSS lives. For the most part, the CSS we use in Angular is exactly the same as CSS we would use in any traditional website. There is, however, a small exception to this. With Angular components - we are essentially creating new HTML elements that we can use. If we want to add styles to that element, we have to use the `:host` selector. For example, if we wanted all of the text within our `example` component to be red, we could add the following code to our css file:

```css
:host {
  color: red;
}
```

## Using components

If we were to use our `example` component in an Angular app, we would write something like the following in a parent component:

```html
<app-example></app-example>
```

We would expect to see the following text in our browser:

```
Hello, world
Nice to meet you!
```

## Component inputs

It is often the case that we want to be able to pass data from a parent component to a child component. Angular provides us a very powerful way of doing this, by allowing us to pass data into a component when we use it inside a HTML template.

Using the `example` component we explored above - let's take a look at how we might do this.

We want to be able to configure the `greeting` property of our component so that any component that uses the `example` component can change the greeting.

To do this, the first thing we need to do is add the `@Input` directive to the greeting property inside our `example.component.ts` file like below:

```ts
import { Component, Input } from "@angular/core";

@Component({
  selector: "app-example", // defining the name of our component
  templateUrl: "./example.component.html", // pointing the the component's html file
  styleUrls: ["./example.component.css"], // pointing to the component's CSS file
})
export class ExampleComponent {
  @Input("greeting") greeting = "Hello, world"; // a property containing a greeting we want to display in our component
}
```

This will allow any component that uses the `example` component to pass a value for the `greeting` property. For example, we could write this to set our greeting to be `Goodbye, Mars`

```html
<app-example greeting="Goodbye, Mars"></app-example>
```

Let's take a look at that in a little bit more detail.

```ts
 @Input('greeting')
```

`Input` is the Angular directive that allows components to take inputs as properties within html.

`'greeting'` is the name of the property will will use inside the html. If instead, we wrote:

```ts
 @Input('bob') greeting = "Hello, world";
```

We would change the greeting property by writting HTML like so:

```html
<app-example bob="Goodbye, Mars"></app-example>
```

**Note** It is best practice to use the same name as the property. In fact, in newer versions of Angular, you may get an error when trying to name your input differently to the property that it controls.

## Component outputs

When we want a component to inform a parent component that something has happened (a button click, a form has been submitted, etc) we can use the `@Output` directive.

Let's update our `example` component's HTML so that we have a button our user can click to say hello back to the parent component.

```html
<p>{{ greeting }}</p>
<p>Nice to meet you!</p>
<button>Say hello</button>
```

Now that we have a button, we want to configure our component to emit an output whenever the user clicks the button. For now, we'll just console.log something to show that we're listening to the click event.

In our `example.component.ts` file, let's add a method we can call when the button is clicked

```ts
import { Component, Input } from "@angular/core";

@Component({
  selector: "app-example", // defining the name of our component
  templateUrl: "./example.component.html", // pointing the the component's html file
  styleUrls: ["./example.component.css"], // pointing to the component's CSS file
})
export class ExampleComponent {
  @Input("greeting") greeting = "Hello, world"; // a property containing a greeting we want to display in our component

  sayHello() {
    console.log("user says: hello!");
  }
}
```

The last thing we need to do to get the button calling this function, is to hook into the `click` event on the button. We want do this in our HTML like so:

```html
<button (click)="sayHello()">Say hello</button>
```

Now, when a user clicks the button, we should see `user says: hello!` in our browser console.

The final part of getting our `example` component to inform the parent component that the user has clicked the button is to add an output to our component. We can do this by adding a new property to our component:

```ts
import { Component, Input, Output, EventEmitter } from "@angular/core";

@Component({
  selector: "app-example", // defining the name of our component
  templateUrl: "./example.component.html", // pointing the the component's html file
  styleUrls: ["./example.component.css"], // pointing to the component's CSS file
})
export class ExampleComponent {
  @Input("greeting") greeting = "Hello, world"; // a property containing a greeting we want to display in our component
  @Output("greetBack") greetBack = new EventEmitter<string>();

  sayHello() {
    console.log("user says: hello!");
  }
}
```

Let's break this down to look at the different parts of what we just added.

```ts
  @Output("greetBack") greetBack = new EventEmitter<string>()
```

First, we specify that the property we're adding to our component is going to be an output. The string we're passing into the `@Output` directive is the name of the property a parent component will use to reference the output. In this case, we're calling the property `greetBack`. Any parent component that uses our `example` component will listen to this output by writing the following HTML

```html
<app-example (greetBack)="doSomething($event)"></app-example>
```

Notice that the `greetBack` property is wrapped in `()`. This is Angular's way of saying "listen to events from this component". Any event that a component (or an in-built element such as a button) can emit has to be wrapped in `()`.

In the above example, the parent component wants to call the `doSomething` function whenever the event is emitted. You'll also notice that the parent component is passing the `$event` variable into the function. The `$event` variable is a special variable that refers to whatever the event is emitting. In in-built components, such as a button, the `$event` variable is likely to refer to the event that has occured; for example, a [click event](https://www.w3schools.com/jsref/event_onclick.asp).

When a parent component listens to our `greetBack` event, the `$event` variable will refer to whatever value we choose to emit. In our case, we created an `EventEmitter<string>` - this means the value of our event will be a `string`.

Finally, we need to hook both of these parts together, by emitting an event when the user clicks the button. We can do this by changing our `sayHello` function to look emit our event.

```ts
import { Component, Input, Output, EventEmitter } from "@angular/core";

@Component({
  selector: "app-example", // defining the name of our component
  templateUrl: "./example.component.html", // pointing the the component's html file
  styleUrls: ["./example.component.css"], // pointing to the component's CSS file
})
export class ExampleComponent {
  @Input("greeting") greeting = "Hello, world"; // a property containing a greeting we want to display in our component
  @Output("greetBack") greetBack = new EventEmitter<string>();

  sayHello() {
    this.greetBack.emit("user says: hello!");
  }
}
```
