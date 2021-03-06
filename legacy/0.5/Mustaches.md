# Mustaches


## What is Mustache?

[Mustache](http://mustache.github.io/) is one of the most popular templating languages. It's a very lightweight, readable syntax with a comprehensive specification - which means that implementations (such as Ractive) can test that they're doing things correctly.

If you've used [Handlebars](http://handlebarsjs.com) or [Angular](http://angularjs.org) you'll also find mustaches familiar.

## What are mustaches?

Within this documentation, and within Ractive's code, 'mustache' means two things - a snippet of a template which uses mustache delimiters, such as `{{name}}`, and the object within our [parallel DOM](parallel DOM.md) that is responsible for listening to data changes and updating the (real) DOM.

We say that the `{{name}}` mustache has a *[reference](references)* of `name`. When it gets rendered, and we create the object whose job it is to represent `name` in the DOM, we attempt to *resolve the reference according to the current context stack*. For example if we're in the `user` context, and `user` has a property of `name`, `name` will resolve to a [keypath](keypaths.md) of `user.name`.

As soon as the mustache knows what its keypath is (which may not be at render time, if data has not yet been set), it registers itself as a *[dependant](dependants.md)* of the keypath. Then, whenever data changes, Ractive scans the dependency graph to see which mustaches need to update, and notifies them accordingly.

## Mustache basics

If you already know Mustache, Ractive supports all the Mustache features - basic Mustache variables like `{{name}}`, as well as sections, partials, and even delimiter changes. If you're already familiar with Mustache, skip to the Extensions section below.

You can also check out the [tutorials](http://learn.ractivejs.org).

### Variables

The most basic mustache type is the variable. A `{{name}}` tag in a template will try to find the `name` key in the current context. If there is no `name` key in the current context, the parent contexts will be checked recursively. If the top context is reached and the name key is still not found, nothing will be rendered.

All variables are HTML escaped by default. If you want to return unescaped HTML, use the triple mustache: `{{{name}}}`.

You can also use `&` to unescape a variable: `{{& name}}`. This may be useful when changing delimiters (see "Set Delimiter" below).


Template:

```html
 * {{name}}
 * {{age}}
 * {{company}}
 * {{{company}}}
```

With the following data:

```javascript
{
  "name": "Chris",
  "company": "<b>GitHub</b>"
}
```

Will generate the following output:

```
 * Chris
 *
 * &lt;b&gt;GitHub&lt;/b&gt;
 * <b>GitHub</b>
```

### Sections
Sections render blocks of text one or more times, depending on the value of the key in the current context.

A section begins with a pound and ends with a slash. That is, `{{#person}}` begins a "person" section while `{{/person}}` ends it.

The behavior of the section is determined by the value of the key.

#### False Values or Empty Lists

If the person key exists and has a value of false or an empty list, the HTML between the pound and slash will not be displayed.

Template:

```html
Shown.
{{#person}}
  Never shown!
{{/person}}
```

Data:

```javascript
{
  "person": false
}
```
Output:

```html
Shown.
```
#### Non-Empty Lists

If the person key exists and has a non-false value, the HTML between the pound and slash will be rendered and displayed one or more times.

When the value is a non-empty list, the text in the block will be displayed once for each item in the list. The context of the block will be set to the current item for each iteration. In this way we can loop over collections.

Template:

```html
{{#repo}}
  <b>{{name}}</b>
{{/repo}}
```

Data:

```javascript
{
  "repo": [
    { "name": "resque" },
    { "name": "hub" },
    { "name": "rip" }
  ]
}
```

Output:

```html
<b>resque</b>
<b>hub</b>
<b>rip</b>
```

#### Non-False Values

When the value is non-false but not a list, it will be used as the context for a single rendering of the block.

Template:

```html
{{#person?}}
  Hi {{name}}!
{{/person?}}
```

Data:

```javascript
{
  "person?": { "name": "Jon" }
}
```

Output:

```html
Hi Jon!
```

#### Inverted Sections

An inverted section begins with a caret (hat) and ends with a slash. That is  `{{^person}}` begins a "person" inverted section while `{{/person}}` ends it.

While sections can be used to render text one or more times based on the value of the key, inverted sections may render text once based on the inverse value of the key. That is, they will be rendered if the key doesn't exist, is false, or is an empty list.

Template:

```html
{{#repo}}
  <b>{{name}}</b>
{{/repo}}
{{^repo}}
  No repos :(
{{/repo}}
```

Ractive borrows a trick from Handlebars here and lets you perform:

```html
{{#repo}}
  <b>{{name}}</b>
{{else}}
  No repos :(
{{/repo}}
```

Data:

```javascript
{
  "repo": []
}
```

Output:

```html
No repos :(
```

### Comments
Comments begin with a bang and are ignored. The following template:

```html
<h1>Today{{! ignore me }}.</h1>
```

Will render as follows:

```html
<h1>Today.</h1>
```

If you'd like the comments to show up, just use html comments and set [stripComments](options.md#stripComments) to `false`.
Comments may contain newlines.

### Partials

Partials begin with a greater than sign:

```html
{{> box}}
```

Recursive partials are possible. Just avoid infinite loops.

They also inherit the calling context. For example:

```html
{{> next_more}}
```


In this case, `next_more.mustache` file will inherit the size and start methods from the calling context.

In this way you may want to think of partials as includes, or template expansion:

For example, this template and partial:

base.mustache:

```html
<h2>Names</h2>
{{#names}}
  {{> user}}
{{/names}}
```

With `user.mustache` containing:

```html
<strong>{{name}}</strong>
```

Can be thought of as a single, expanded template:

```html
<h2>Names</h2>
{{#names}}
  <strong>{{name}}</strong>
{{/names}}
```

### Custom delimiters

Custom delimiters are set with a 'Set delimiter' tag. Set delimiter tags start with an equal sign and change the tag delimiters from `{{` and `}}` to custom strings.

```html
{{foo}}
  {{=[[ ]]=}}
[[bar]]
```

Custom delimiters may not contain whitespace or the equals sign.


You can also set custom delimiters using the `delimiters` and `tripleDelimiters` options in your Ractive instance.

```javascript
var ractive = new Ractive({
  el: whatever,
  template: myTemplate,
  data: {
    greeting: 'Hello',
    greeted: 'world',
    triple: '<strong>This is a triple-stache</strong>'
  },
  delimiters: [ '[[', ']]' ],
  tripleDelimiters: [ '[[[', ']]]' ]
});
```

## Extensions

Ractive is 99% backwards-compatible with Mustache, but adds eight additional features (array index references, object iteration, restricted references, ancestor references, expressions, special sections, aliasing, and static mustaches) which are detailed below.

### Array Index references

Index references are a way of determining where we are within a list section. It's best explained with an example:

```html
{{#items:i}}
  <!-- within here, {{i}} refers to the current index -->
  <p>Item {{i}}: {{content}}</p>
{{/items}}
```

If you then set `items` to `[{content: 'zero'}, {content: 'one'}, {content: 'two'}]`, the result would be

```html
<p>Item 0: zero</p>
<p>Item 1: one</p>
<p>Item 2: two</p>
```

This is particularly useful when you need to respond to user interaction. For example you could add a `data-index='{{i}}'` attribute, then easily find which item a user clicked on.

### Object Iteration

Mustache can also iterate over objects, rather than array. The syntax is the same as for Array indices. Given the following ractive:

```javascript
ractive = new Ractive({
  el: container,
  template: template,
  data: {
    users: {
      'Joe': { email: 'joe@example.com' },
      'Jane': { email: 'jane@example.com' },
      'Mary': { email: 'mary@example.com' }
    }
  }
});
```

We can iterate over the users object with the following:

```html
<ul>
  {{#users:name}}
    <li>{{name}}: {{email}}</li>
  {{/users}}
</ul>
```

to create:

```html
<ul>
  <li>Joe: joe@example.com</li>
  <li>Jane: jane@example.com</li>
  <li>Mary: mary@example.com</li>
</ul>
```

In previous versions of Ractive it was required to close a section with the opening keypath. In the example above `{{#users}}` is closed by `{{/users}}`. This is no longer the case, you can now simply close an iterator with `{{/}}`. Ractive will attempt to warn you in the event of a mismatch, `{{#users}}` cannot be closed by `{{/comments}}`. This will not effect [Expressions](Expressions.md) as they have always been able to be closed by `{{/}}`.

```html
<!--- valid markup -->
{{#users}}

{{/users}}

{{#users:i}}

{{/users}}

{{#users}}

{{/}}

{{#users.topUsers}}
<!-- still matches the first part of the keypath, thus a valid closing tag -->
{{/users}}

<!-- invalid markup -->
{{#users}}

{{/comments}}
```

### Restricted references

Normally, references are resolved according to a specific algorithm, which involves *moving up the context stack* until a property matching the reference is found. In the vast majority of cases this is exactly what you want, but occasionally (for example when dealing with [recursive partials](parials.md#recursive-partials)) it is useful to be able to specify that a property must exist *in the current context*.

To restrict a reference to the current context, prefix it with a `.`, e.g. `{{#.bar}}`:

```html
{{#foo}}
  {{#bar}}This section will render, because it will resolve to 'bar'{{/bar}}
  {{#.bar}}This section will NOT render, because it resolves to 'foo.bar'{{/.bar}}
{{/foo}}
```

```js
ractive = new Ractive({
  el: myContainer,
  template: myTemplate,
  data: { bar: true, foo: {} }
});
```


### Ancestor references

Very occasionally, you might need to refer explicitly to a property higher up in the tree to avoid naming conflicts. You can do that by prefixing references with `../` (or `../../`, or `../../../`...):

```html
<h1>Blog posts by {{name}}</p>

<ul class='blog-posts'>
  {{#posts}}
    <li><a href='{{ slugify(../../name) }}/{{ slugify(name) }}'>{{name}}</a></li>
  {{/posts}}
</ul>
```

```js
var ractive = new Ractive({
  el: document.body,
  template: myTemplate,
  data: {
    name: 'Rich',
    posts: [
      { name: 'This is a blog post' },
      { name: 'And so is this' }
    ],
    slugify: function ( str ) {
      var slug;
      /* SOME CODE HAPPENS */
      return slug;
    }
  }
});
```

In this example, some damn fool decided it would be a good idea to give posts a 'name' property as well as authors. Which means that in the `href` attribute, it wouldn't be possible to refer to the root-level `name` property, because it would always resolve to the `name` property of the current post - and that's no good, because the root-level `name` property is used to construct the URL.

By using `../../name` instead of `name`, we're saying 'go up one level (to `posts`), then up one more (to the root), and *then* look for the `name` property'.


### Expressions

Expressions are a big topic, so they have a [page of their own](Expressions.md). But this section is about explaining the difference between vanilla Mustache and Ractive Mustache, so they deserve a mention here.

Expressions look like any normal mustache. For example this expression converts `num` to a percentage:

```html
<p>{{ num * 100 }}%</p>
```

The neat part is that this expression will recognise it has a dependency on whatever keypath `num` resolves to, and will re-evaluate whenever the value of `num` changes.

Mustache fans may bristle at expressions - after all, the whole point is that mustache templates are *logic-less*, right? But what that really means is that the logic is *embedded in the syntax* (what are conditionals and iterators if not forms of logic?) rather than being language dependent. Expressions just allow you to add a little more, and in so doing make complex tasks simple.


### Special Sections

In addition to implicit conditional and iterative sections, Ractive adds ```if```, ```unless```, ```each```, and ```with``` to handle branching, iteration, and context control. For ```if``` and ```each```, ```{{else}}``` may be used to provide an alternate branch for a false condition or an empty iterable.

```html
<button on-click="flip">Flip Coin</button>
<p>Coin flip result: {{#if heads}}heads{{else}}tails{{/if}}</p>
<ul>
  {{#each result}}
    <li>{{.}}</li>
  {{else}}
    <li>No results yet...</li>
  {{/each}}
</ul>
<p>Here is a {{#with some.nested.value}}{{.}}{{/with}} value.</p>
```

```js
var ractive = new Ractive({
  el: document.body,
  template: myTemplate,
  data: {
    results: [],
    heads: true,
    some: { nested: { value: 'nested' } }
  }
});

ractive.on('flip', function() {
  var sadRandom = Math.floor(Math.random() * 2) === 1;
  this.set('heads', sadRandom);
  this.unshift('results', sadRandom ? 'heads' : 'tails');
});
```

In this example, clicking the button gets a "random" coin flip result, sets it in an ```if``` conditional section, and prepends it in an ```each``` iterative section. There is also a ```with``` context section throw in for good measure.


### Aliasing

Since a section with a non-falsy value uses the value as its context, an object expression can be used to create an arbitrary context within a section. Any currently visible context keypaths can be referenced within the new context object, which allows you to keep keypaths available that would otherwise be unreachable.

```html
{{# { id: '10t' } }}
  Error: {{id}}
{{/}}
```

Output:
```html
Error: 10t
```

Using a section with an object expression to keep references accessible or convenient:

```html
<ul>
{{#things:outerIndex}}
  {{# { thing: ., i: outerIndex } }}
    {{#colors:j}}
      <li>{{i}}-{{j}} {{.}} {{thing}}</li>
    {{/}}
  {{/}}
{{/}}
</ul>
```

Data:
```js
{
  colors: ['Purple', 'Orange', 'Green'],
  things: ['People-Eater', 'Orange', 'Hornet']
}
```

Output:
```html
<ul>
  <li>0-0 Purple People-Eater</li>
  <li>0-1 Orange People-Eater</li>
  <li>0-2 Green People-Eater</li>
  <li>1-0 Purple Orange</li>
  <li>1-1 Orange Orange</li>
  <li>1-2 Green Orange</li>
  <li>2-0 Purple Hornet</li>
  <li>2-1 Orange Hornet</li>
  <li>2-2 Green Hornet</li>
</ul>
```


### Static Mustaches

Sometimes it is useful to have portions of a template render once and stay the same even if their references change. A static mustache will be updated only when its template is rendered and not when its keypath is updated. So, if a static mustache is a child of a section or partial that get re-rendered, the mustache will also be re-rendered using the current value of its keypath.

The default static mustache delimiters are `[[ ]]` for escaped values and `[[[ ]]]` for unescaped values.

```html
[[ foo ]] {{ foo }}
{{^flag}}
  [[ foo ]]
{{/}}
```

```js
var ractive = new Ractive({
  data: { foo: 'bar' },
  ...
});
ractive.set('foo', 'bippy');
ractive.set('flag', true);
ractive.set('flag', false);
```

Output:
```html
bar bippy bippy
```

Static mustaches may also be used for sections that should only be updated on render.

```html
[[# if admin ]]
Hello, admin
[[else]]
Hello, normal user
[[/if]]
```

```js
var ractive = new Ractive({
  data: { admin: false },
  ...
});
ractive.set('admin', true);
```

Output:
```
Hello, normal user
```


## Footnote

*Ractive implements the Mustache specification as closely as possible. 100% compliance is impossible, because it's unlike other templating libraries - rather than turning a string into a string, Ractive turns a string into DOM, which has to be restringified so we can test compliance. Some things, like lambdas, get lost in translation - it's unavoidable, and unimportant.
