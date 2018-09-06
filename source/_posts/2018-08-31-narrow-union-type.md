---
layout: post
title: "Narrowing a union type in typescript and a gotcha with callbacks"
date: 2018-08-31 18:01:52 +0100
comments: true
categories: typescript javascript
---
## Narrowing a union type
In typescript, a union type describes a value that can be one of several types separated by the vertical `|` bar, for example:
{% codeblock text.js %}
let text: string | string[];

text = 'text';

text = ['text1', 'text2'];
{% endcodeblock %}
In the above code `text` can either be a string or an array of strings.

Problems arise in union types because you can only access members that are common to all types in the union.

I have recently been commiting code to [afterjs](https://github.com/jaredpalmer/after.js) to add better type safety to the existing code.

On one of [my PRs](https://github.com/jaredpalmer/after.js/pull/166) I created the following union to model all the component types that afterjs might have to deal with when deciding whether or not the component is loaded via a [dynamic import](https://webpack.js.org/guides/code-splitting/).  The compponent might be an `AyncComponent` that knows how to load itself via a dynamic import or it might be a react-router aware component or just a plain old react component.  I created the following union:

{% codeblock union.js %}
export type AsyncRouteableComponent<Props = any> =
  | AsyncRouteComponentType<RouteComponentProps<Props>>
  | React.ComponentType<RouteComponentProps<Props>>
  | React.ComponentType<Props>;
{% endcodeblock %}

An `AsyncRouteableComonent` will have the following interface that gets added in a previous type.

{% codeblock AsyncRouteableComponent.js %}
interface AsyncComponent {
  getInitialProps: (props: DocumentProps) => any;
  load?: () => Promise<React.ReactNode>;
}
{% endcodeblock %}

## Type Guards
There is also in typescript the concept of <a href="https://medium.com/@OlegVaraksin/narrow-interfaces-in-typescript-5dadbce7b463" target="_blank">`Type Guards`</a>.  A type guard is an expression that performs a runtime check that guarantees that you have the requested type.  I came up with this type guard to ensure I am dealing with an `AsyncRouteableComponent`:
{% codeblock guard.js %}
export function isAsyncComponent(Component: AsyncRouteableComponent): Component is AsyncRouteComponentType<any> {
  return (<AsyncRouteComponentType<any>>Component).load !== undefined;
{% endcodeblock %}
Any time `isAsyncComponent` is called, typescript will *narrow* the specific variable to that specific type:
{% codeblock narrow.js %}
if (isAsyncComponent(component)) {
    component.load().then(() => doStuff)
}
{% endcodeblock %}
## Gotcha
There is a gotcha though and I encountered it with this code:
{% codeblock problem.js %}
  const match = routes.find((route: AsyncRouteProps) => {
    const matched = matchPath(pathname, route);

    if (matched && route.component && isAsyncComponent(route.component)) {
      promises.push(
        route.component.load
          ? route.component.load().then(() => route.component.getInitialProps({ matched, ...ctx }))
          : route.component.getInitialProps({ matched, ...ctx })
      );
    }

    return !!matched;
  });

  return {
    match,
    data: (await Promise.all(promises))[0]
  };
{% endcodeblock %}
Typescript narrows the scope in all cases due to the `isAsyncComponent` type guard on line 4 apart from in the anonymous function resolve handler of the `load` promise on line 7:
{% codeblock resolve.js %}
load().then(() => route.component.getInitialProps({ matched, ...ctx }))
{% endcodeblock %}

The `route.component` in the anonomus function that resolves after `load` gives  the following compiler error:

>> Property 'getInitialProps' does not exist on type 'AsyncRouteableComponent<any>'.
>>  Property 'getInitialProps' does not exist on type 'ComponentClass<RouteComponentProps<any, StaticContext>>'.

Typescript has not narrowed the scope because it cannot find a `getInitialProps` member on all types of the union.

It turns out narrowing for a mutable variable such as `route.component` does not apply inside callbacks such as the anonymous function resolve handler because typescript does not trust that the local variable will not be reassigned before the callback executes.  There is a whole big thread about it [here](https://github.com/Microsoft/TypeScript/issues/7662). 

The workaround is to copy `route.component` to a local variable before the route guard is called.

Here is the working solution:

{% codeblock working.js %}
import { matchPath } from 'react-router-dom';
import { AsyncRouteProps, InitialProps } from './types';
import { isAsyncComponent } from './utils';

export async function loadInitialProps(routes: AsyncRouteProps[], pathname: string, ctx: any): Promise<InitialProps> {
  const promises: Promise<any>[] = [];

  const match = routes.find((route: AsyncRouteProps) => {
    const matched = matchPath(pathname, route);

    const component = route.component;

    if (matched && component && isAsyncComponent(component)) {
      promises.push(
        component.load
          ? component.load().then(() => component.getInitialProps({ matched, ...ctx }))
          : component.getInitialProps({ matched, ...ctx })
      );
    }

    return !!matched;
  });

  return {
    match,
    data: (await Promise.all(promises))[0]
  };
}

{% endcodeblock %}

The copy happens on line 11 with `const` used to ensure the local variable cannot be reassigned, typescript is satisfied that it cannot be reassigned and the scope is narrowed for the union and the correct scope is applied.
