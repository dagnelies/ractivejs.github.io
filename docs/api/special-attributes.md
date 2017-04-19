# Special attributes

## `style-*` attributes

`style-*` attributes update individual `style` properties. There are two forms of the syntax: `style-property-name="value"` and `style-propertyName="value"`. The former will be normalized into the latter and is applied to the element's `style` property.

```html
<div style-vertical-align="middle" style-textAlign="center">...</div>
```

Mustaches can also be used to supply the values. When the values are updated, the appropriate style property on the element will update to the new value. `style-*` attributes are only processed as text.

```html
<div style-vertical-align="{{ vAlign }}" style-textAlign="{{ tAlign }}">...</div>
```

## `class-*` attributes

`class-*` attributes toggle individual class names based on the truthiness of its value. Unlike `style-*`, no normalization is done to the class name. `class-class-name` and `class-className` will represent `class-name` and `className`, respectively.

`class-*` attributes are only processed as text, and any text provided is considered a truthy value.

```html
<div class-highlighted="true">I'm highlighted</div>
<div class-highlighted="false">I'm also highlighted</div>
```

To supply the expression that determines the presence of the class name, an interpolator must be used. When the values are updated, the appropriate class name will be added to or removed from the element.

```html
<div class-highlighted="{{ isHighlighted }}">Highlighted if true</div>
<div class-highlighted="{{ age === 42 }}">Highlighted if forty-two</div>
```

## `on-*` attributes

Both element and component events are handled by the `on-*` attribute. They are designed to look similar to regular DOM `on*` attributes, the only difference being the hyphen.

`on-*` accepts an event name which will be fired in its place. They are handled by registering a function that listens to the assigned name using [`ractive.on`](./instance-methods.md#ractiveon).

```js
Ractive({
  template: `
    <button type="button" on-click="clicked">Push me!</button>
  `,
  oninit(){
    this.on('clicked', event => {
      console.log('clicked!');
    });
  }
});
```

`on-*` also accepts expressions allowing it to call functions like inline scripts.

```js
Ractive({
  template: `
    <button type="button" on-click="@this.someMethod()">Push me!</button>
  `,
  someMethod(){
    console.log('clicked!');
  }
});
```

## `*-in`, `*-out`, and `*-in-out` attributes

Another item in the set of things-that-look-like-attributes-but-aren't, [transitions](../extend/transitions.md) allow you to specify how elements should behave when they enter and/or leave the DOM. `*-in` specifies intro-only, `*-out` specifies outro-only, and `*-in-out` for both intro and outro. All three accept optional values, an expression that gets passed into the transition function.

```html
<div fade-in>Fades on render</div>
<div fade-out>Fades before removal</div>
<div fade-in-out>Fades on render and before removal</div>
<div fade-in-out="{ duration: 500 }">Fades with 500ms duration</div>
```
