# knockout-react
A wrapper / bridge for using React.js with Knockout and Knockout with React.js


## Usage

### Using Knockout from within the context of React

Using the `KnockoutMixin` mixin for a React component makes that components rendered DOM tree 
bound to a viewmodel with `props` and `state` properties, which stay in sync with the 
corresponding `props` and `state` of the React component.  You simply use the `data-bind` attribute
syntax like you normally would with Knockout.

```js
var KnockoutMixin = require('knockout-react').KnockoutMixin;

var ToDoList = React.createClass({
    mixins: [ KnockoutMixin ],

    propTypes: {
        todos: React.PropTypes.array.isRequired
    },

    render() {
        return (
            <ul data-bind="foreach: props.todos">
                <li data-bind="text: $data"></li>
            </ul>
        );
    }
});
```


### Using React from within the context of Knockout

Using the `react` binding handler within knockout requires that you pass it a `$` option with the 
react Component you would like to render in the sub-tree. You can optionally pass a `props` option
(which can be observable) to be passed in as props to the component. By default, the binding context 
`$data` is passed in as props.

```html
<ul data-bind="react: { $: ToDoList, props: { todos: todos } }><ul>
```

## How does it work?

The Knockout mixin is pretty straight-forward:

```js
var KnockoutMixin = {

    updateKnockout() {
        this.__koTrigger(!this.__koTrigger());
    },

    componentDidMount() {
        this.__koTrigger = ko.observable(true);
        this.__koModel = ko.computed(function () {
            this.__koTrigger(); // subscribe to changes of this...

            return {
                props: this.props,
                state: this.state
            };
        }, this);

        ko.applyBindings(this.__koModel, this.getDOMNode());
    },

    componentWillUnmount() {
        ko.cleanNode(this.getDOMNode());
    },

    componentDidUpdate() {
        this.updateKnockout();
    }
};
```

Similarly, the `react` bindingHandler is a straight-forward wrapper around `React.render`:

```js
var reactHandler = ko.bindingHandlers.react = {
    render: function ( el, Component, props ) {
        React.render(
            React.createElement(Component,props),
            el
        );
    },

    init: function ( el, valueAccessor, allBindingsAccessor, viewModel, bindingContext ) {
        var options = valueAccessor();
        var Component = ko.unwrap(options.component || options.$);
        var props = ko.toJS('props' in options ? options.props : viewModel);

        reactHandler.render(el, Component, props);

        return { controlsDescendantBindings: true };
    },

    update: function ( el, valueAccessor, allBindingsAccessor, viewModel, bindingContext ) {
        var options = valueAccessor();
        var Component = ko.unwrap(options.component || options.$);
        var props = ko.toJS(options.props || viewModel);

        reactHandler.render(el, Component, props);

        return { controlsDescendantBindings: true };
    }
};
```















