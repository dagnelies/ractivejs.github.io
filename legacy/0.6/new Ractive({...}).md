# new Ractive({...})

## Initialisation

Ractive instances are created using standard javascript instantiation with the `new` keyword:

```js
var ractive = new Ractive();
```

There are no _required_ options, however you'll usually want to specify these base options:

```js
var ractive = new Ractive({
	el: '#container',
	template: '#template',
	data: data
})
```

The full list of initialisation options is [covered here](Options.md).

## Initialising `Ractive.extend`

Ractive offers an [extend](Ractive.extend().md) method for standard javascript prototypical inheritance:

```js
var MyRactive = Ractive.extend({
	template: '#mytemplate'
})
```
The same [initialisation options](Options.md) can be supplied to the extend method, plus some additional options.

These are instantiated in exactly the same way as above, supplying any additional options:

```js
var ractive = new MyRactive({
	el: '#container',
	data: data
})
```

See [Ractive.extend()](Ractive.extend().md) for more details.
