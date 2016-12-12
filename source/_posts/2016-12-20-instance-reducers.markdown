---
layout: post
title: "Instance Reducers"
date: 2016-12-20 14:58:23 +0000
comments: true
categories: javascript react redux
---
I recently came across a situation with <a href="https://github.com/reactjs/redux" target="_blank">redux</a> that I had not considered before.  I had created an autocomplete component that was using redux to store its internal state. This was all well and good when there was only one autocomplete component per page but when I have multiple autocomplete components on a given page then a change to one component was reflected in every component as you can see in the screen shot below where selecting a value in one component instance is reflected in all component instances:

{% img /images/autocomplete.png %}

The problem is that every component is reading and writing to the same redux store state element.

If you look at the code below that uses `combineReducers`, there is only one key for the autocomplete state on *line 7*:

{% codeblock combineReducers.js %}
export default combineReducers({
  managers,
  auth,
  results,
  teams,
  form,
  autocomplete,
  routing: routerReducer,
  error
})
{% endcodeblock %}

{% blockquote %}
For the record, I am now firmly of the opinion that using redux to store a component's internal state is actually a bad idea.  If I was to code the autocomplete component again then I would store the state internally in the component.  I still think there is a place though for an instance reducer.
{% endblockquote %}

My autocomplete reducer was working as I would expect with one component so I just needed to ensure that any code change would work with multiple components without changing the working reducer code.  I also wanted to make this reusable for other situations.  I just needed some layers of indirection in my <a href="https://github.com/reactjs/react-redux/blob/master/docs/api.md" target="_blank">mapStateToProps</a> and any <a href="http://redux.js.org/docs/basics/Reducers.html" target="_blank">redux reducer</a> that I wanted to ensure was segrating its global state by instance.

The steps I needed to complete this task would be:

1. Ensure that any *connected component* subscribing to redux store state change events had its own unique id.
2. `mapStateToProps` subscribes a component to redux store updates.  I would create a `mapStateToProps` that would retrieve the relevant state that would be tagged with the unique identifier mentioned in step 1.
3. `mapDispatchToProps` wraps <a href="http://redux.js.org/docs/basics/Actions.html">action creators</a> in a `dispatch` function which sends an action to a reducer and updates the global store.  I could hijack this functionality to send a component identifier with each **dispatched** action.
4. I would need some capability to wrap any reducer and only mark the state in the store by the unique identifier mentioned in step 1.

For the above requirements, I would create a custom <a href="https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options" target="_blank">connect</a> function that would call the real <a href="https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options" target="_blank">connect</a> and add some code to ensure that state was retrived by unique identifier.  I would also need a function that added some functionality around calling a reducer.

`connect` **connects** a component to the redux store.  It does this by returning a <a href="https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e#.jbkmtfd6r" target="_blank">higher order component</a> which abstracts away the internals of subscribing to store updates.  Higher order components are one of the many things I love about react.

Below is the function that is returned from what I ended up calling `instanceConnect` that in turn calls <a href="https://github.com/reactjs/react-redux" target="_blank">react-redux</a>'s <a href="https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options" target="_blank">connect</a>.

{% codeblock hoc.js %}
  return (WrappedComponent) => {
    const finalWrappedComponent = connect(finalMapStateToProps, finalMapDispatchToProps)(WrappedComponent);

    class InstanceComponent extends Component {
      constructor(...args) {
        super(...args);

        const prefix = WrappedComponent.displayName || WrappedComponent.name || 'Component';
        this.instanceId = uniqueId(prefix);
      }

      render() {
        const props = {
          instanceId: this.instanceId,
          ...this.props
        };

        return createElement(finalWrappedComponent, props);
      }
    }

    return hoistStatics(InstanceComponent, finalWrappedComponent);
  }
{% endcodeblock %}

The above code is what is returned from my `instanceConnect` function, I will supply the full listing later in the post or you can view it all <a href="https://github.com/dagda1/liverpool-managers/blob/master/client/app/hoc/InstanceConnect.js" target="_blank">here</a>.

- *Line 3* calls the react-redux connect function with a modified `mapStateToProps` and `mapDispatchToProps`, I will discuss this later.  The returned function accepts a component as an argument which will be the component that receives store update notifications.  This connected component which is assigned to the `finalWrappedComponent` variable will be used in the `render` function of the higher order component.
- *Lines 4 - 20* define a component class named `InstanceComponent` that will be the result of calling `instanceConnect`.
- The main reason for being of the `InstanceComponent` class is to add a unique id to each instance and also to return the connected component that is constructed on *line 3*.
- An instance id is generated using <a href="https://lodash.com/docs/4.17.2" target="_blank">lodash</a>'s <a href="https://lodash.com/docs/4.17.3#uniqueId" target="_blank">uniqueId</a> utility.
- An `instanceId` property is assigned to a member variable in the constructor on *line 9* and this is added to the `props` on *line 18* of the `render` funtion of the `InstanceComponent`.
- `InstanceComponent` renders the connected component with the `instanceId` appended to the props on *line 18*.

Now that I can identify each component instance with an `instanceId` property that now exists in the props, I somehow need to communicate this unique id to the reducer so I can only pull back the state that is relevant to each component instance.

Below is my modified `mapDispatchToProps` that will call the passed in user supplied `mapDispatchToProps` that wraps each action creator with `dispatch`:

{% codeblock mapDispatchToProps.js %}
  const finalMapDispatchToProps = (dispatch, initialProps) => {
    const { instanceId } = initialProps;
    const keys = Object.keys(mapDispatchToProps);
    const actions = {};

    for(let i = 0; i < keys.length; i += 1) {
      const key = keys[i];
      const actionCreator = mapDispatchToProps[key];

      if(typeof actionCreator !== 'function') {
        throw new Error('InstanceConnect mapDispatchToProps actions');
      }

      actions[key] = (...args) => {
        const action = actionCreator(...args);

        action.meta = action.meta || {};

        action.meta = merge({}, action.meta, { instanceId });

        dispatch(action);
      };
    }

    return actions;
  }
{% endcodeblock %}

- *Line 2* pulls the `instanceId` from the props.
- *Lines 6 - 23* iterates over each key in the supplied object and *lines 14 - 22* create a modified version of the action creator.
- When the action creator is called by the client code, the `instanceId` is appended to the `meta` property of the action on *lines 17 and 19*.  The `meta` property of an action is just a plain object where additional properties can be added without polluting the original `payload`.
- The newly appended action is dispatched on *line 21*.

Now that I can identify a component instance from the action, I need to use this identifier to pull the relevant redux store state information when  state changes are published via `mapStateToProps`.  Below is the modified `mapStateToProps` that pulls out the instance state information:

{% codeblock mapStateToProps.js %}
  const finalMapStateToProps = (state, props) => {
    const { instanceId } = props;

    const instanceState = state.getIn([reducerName, instanceId]) ||
                          state.getIn([reducerName, 'initialState']);

    return mapStateToProps(instanceState, props);
  }
{% endcodeblock %}

The `reducerName` argument should marry up to the value you give to the key that is supplied to `combineReducers`.

- The `instanceId` is pulled from the props on *line 2*.
- The observant of you will will notice that I am using <a href="https://facebook.github.io/immutable-js/" target="_blank">immutablejs</a> to store the state.
- *Lines 4 - 5* use the <a href="https://facebook.github.io/immutable-js/docs/#/List/getIn" target="_blank">getIn</a> function of the immutablejs <a href="https://facebook.github.io/immutable-js/docs/#/Map" target="_blank">Map</a> which is excellent for dealing with nested values in a state hash.  The state either returns the state for that instance if one exists or tries to retrieve data for a property named `initialState` for this particular slice of the store state pie.

The `reducerName` property on *lines 4 -5* is passed in as an argument from the `instanceConnect` function that returns the higher order component, you can see the full listing <a href="https://github.com/dagda1/liverpool-managers/blob/master/client/app/hoc/InstanceConnect.js" target="_blank">here</a>:

{% codeblock signature.js %}
export default function instanceConnect(
  reducerName,
  mapStateToProps,
  mapDispatchToProps
{% endcodeblock %}

The modified `mapStateToProps` and `mapDispatchToProps` are passed to react-redux's connect.

All that needs to be done is provide a function that can wrap a reducer with some indirection that ensures the state is modified with respect to the `instanceId`.

Below is the reducer that accepts another reducer as an argument and ensures that the new version of the state is marked with the `instanceId`.

{% codeblock reducer.js %}
export const reducerByInstance = (reducer) => {
  return (state, action = {}) => {
    const instanceId = action.meta && action.meta.instanceId;

    if(!state) {
      const initialState = reducer(undefined);

      return Map(fromJS({ initialState }));
    }

    if (!instanceId) {
      return state;
    }

    const instanceState = state.get(instanceId);

    const result = reducer(instanceState, action);

    return result === instanceState ? state : state.setIn([instanceId], result);
  };
}
{% endcodeblock %}

- The `instanceId` property is retrieved from the `meta` property that was appended with the `instanceId` in the modified `mapDispatchToProps`.
- *Lines 5 - 9* handles the case when redux invokes the `@@init` action.  There will be no state at this stage and so we call the reducer with `undefined` on *line 6*, this will give the reducer the opportunity to supply some initial state.  We then append this to a property we will name `initialState`.  This property can then be retrieved in our modified `mapStateToProps` when there is no instance state.
- *Lines 11 - 13* simply return the state when there is no `instanceId` in the props.
- *Line 15* retrieves any instance state.
- *Line 17* calls the reducer and passes in any instance state that might already exist.  The fact that a reducer is a <a href="https://en.wikipedia.org/wiki/Pure_function" target="_blank">pure function</a> is critical to this working correctly.
- *Line 19* updates the state hash with respect to the `instanceId` by calling the immutablejs <a href="https://facebook.github.io/immutable-js/docs/#/Map/setIn" target="_blank">setIn</a> function.  This update only happens if the state has actually changed.

## Usage

With the `instanceConnect` function ready to use, then it is simply a matter of calling it like so:

{% codeblock usage.js %}
const mapStateToProps = (state, ownProps) => {
  return { ...ownProps, ...state.toJS() };
}

const mapDispatchToProps = {
  init: actions.init,
  setText: actions.setText,
  selectItem: actions.selectItem,
  clearItems: actions.clearItems,
  setHighlight: actions.setHighlight,
  closeList: actions.closeList,
  openList: actions.openList
};

export default instanceConnect('autocomplete', mapStateToProps, mapDispatchToProps)(Autocomplete);
{% endcodeblock %}

You also need to wrap the appropriate reducers with the `reducersByInstance` function:

{% codeblock reducerByInstance.js %}
export default combineReducers({
  managers,
  auth,
  results,
  teams,
  form,
  autocomplete: reducerByInstance(autocomplete),
  routing: routerReducer,
  error
})
{% endcodeblock %}

I think what is missing would be to remove the component's state when the component is unmounted and you could also create the slice when the component is mounted.  I'll possibly look at adding that.

If you disagree or agree with any of the above then please leave a comment below.

<a href="https://github.com/dagda1/liverpool-managers/blob/master/client/app/hoc/InstanceConnect.js" target="_blank">Here</a> is a full listing of the code file that contains both the `reducerByInstance` and `instanceConnect` functions.

<a href="https://github.com/dagda1/liverpool-managers/blob/master/client/test/redux/connect-test.js" target="_blank">Here</a> is a test I wrote to drive out the functionality.
