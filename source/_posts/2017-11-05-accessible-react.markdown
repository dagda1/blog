---
layout: post
title: "Creating an accessible react website"
date: 2017-11-05 10:33:17 +0000
comments: true
categories: react accessibility
---
I've recently been working on an online application form in the form of a multistep wizard that had strict accessibility requirements.  I've never worked on a project with such strict requirements before. I've also heard rumblings that it was not possible to make a SPA accessible. It turns out that there is not that much work involved in making your site accessible and I am going to ensure that any work I do from now on has an accessible first approach.  I'm now going to outline in no particular order what I have learned over the past few months.

## Use a router

The initial appeal appeal of the SPA was that it negated the need to go to the server to render new content.  The problem is that a newly server rendered page works great with a screen reader but when you change routes in an SPA, the screen reader does not know that there is new content.

One solution is to have a container component that checks for changes in the react-router `location` property and focus on an element at the top of the viewport each time the `location` changes.

Below is an example of such a container component:

{% codeblock ScrollToTop.js %}
class ScrollToTop extends Component {
  props: Props;

  el: HTMLElement;

  focusOnElement = () => {
    setTimeout(() => {
      this.el.focus();
    }, 100);
  };

  componentDidMount() {
    this.focusOnElement();
  }

  componentDidUpdate(prevProps: Props) {
    if (
      this.props.location.hash.length ||
      this.props.location === prevProps.location
    ) {
      return;
    }

    window.scrollTo(0, 0);

    this.focusOnElement();
  }

  render() {
    const { children } = this.props;

    return (
      <div ref={el => (this.el = el)} tabIndex="-1" className="main-content">
        {children}
      </div>
    );
  }
}

export default withRouter(ScrollToTop);
{% endcodeblock %}

The above container checks `location` prop changes in `componentDidUpdate` and will focus on the `ref` element tagged in `line 33` if a location change is detected.  A `tabIndex` of `-1` is set on the `ref` which does not add the element to the natural tab order but does allow you to progrmmatically set focus on an element that would not normally receive focus.

## Keyboard Navigation

As we now have a router and a container component that detects route changes, we should ensure that we can tab up and down the page on all elements that require focus.

There really is not a lot to this if you use sensible html element choices for buttons and links.  You should not make a `span` tag or a `div` a button or a link for example.  I'm sure I've been guilty of making a `span` clickable in the past but that is now firmly on the banned list.

One gotcha is links, if you have a link with no `href` then you need to give it a `tbIndex` of `0`.

Below is a cut down example of a `Link` component.

{% codeblock Link.js %}
export const Link: React.StatelessComponent<LinkProps> = ({ href, onClick, children, ...rest }) => {
  const linkProps = {};

  if(!href) {
    linkPops.tabIndex = 0;
  } else {
    linkProps.href = href;
  }

  return (
    <a {...linkProps} {...rest}>
      {children}
    </a>
  )
}
{% endcodeblock %}

## Use a component library

Using a component library that is outside of the context of the business rules is a great choice. I have controlled components for even simple html elements like `input`, `label` etc.  This allows me ensure that I am consistent about things like <a href="https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA" target="_blank">aria attributes</a>.  Somebody remarked in a code review that React was broken if I needed to create components for things like and `input` but they missed the point.  Below is a simple `input` component that I use instead of the default react element:

{%codeblock Input.js %}
export const Input: React.StatelessComponent<InputProps> = ({
  invalid,
  className,
  required,
  ...rest
}) =>
  <input
    autoComplete="off"
    required={required}
    aria-required={required}
    className={cs(className, {
      [styles.invalid]: invalid
    })}
    {...rest}
  />;

Input.displayName = 'Input';

Input.defaultProps = {
  type: 'text'
};

{% endcodeblock %}

It is also great to think about components in isolation or out of the context that they will be used.  Working in the context of a component library without `redux`, `mobx` etc. leads to better generic components.

## Label all Form Controls

All form or input controls should have labels that describe the purpose of the form control.

A label for a form control helps everyone better understand its purpose. In some cases, the purpose may be clear enough from the context when the content is rendered visually. The label can be hidden visually, though it still needs to be provided within the code to support other forms of presentation and interaction, such as for screen reader and speech input users.

With the above in mind, I have the following higher order componet that wraps anything needing a label and tags it appropriately.

{% codeblock FormControl.js %}
export function FormControl<T>(
  Comp: Component<T>
): React.Component<T> {
  return class FormControlWrapper extends React.Component<T> {
    id: string;
    constructor(props) {
      super(props);

      this.id = this.props.id || this.props.name || prefixId();
    }

    render() {
      const {
        invalid,
        name,
        label,
        errorMessage,
        className,
        required,
        ...rest
      } = this.props as any;

      const errorId = `${this.id}-error`;

      return (
        <div>
          <Label
            id={`${this.id}-label`}
            htmlFor={this.id}
            required={required}
          >
            {label}
          </Label>
          <div>
            <Comp
              id={this.id}
              name={name}
              invalid={invalid}
              aria-invalid={invalid}
              required={required}
              aria-describedby={errorId}
              {...rest}
            />
          </div>
          <div
            id={errorId}
            aria-hidden={!invalid}
            role="alert"
          >
            {invalid &&
              errorMessage &&
              <Error
                errorMessage={errorMessage}
              />}
          </div>
        </div>
      );
    }
  };
}
{% endcodeblock %}

- `Line 15` assigns an `id` for the passed in wrapped component by first of all checking the props or defaulting to a generated id from the `prefixId` function.
- `Line 29` assigns the `htmlFor` attribute of the `label` to the `id` member variable.
- `line 36` sets the `id` of the passed in wrapped component to ensure they are properly assigned.
- The correct `aria` tags are also set for things like `aria-invalid` on `line 39` in the wrapped component.  The beauty of higher order components is that the wrapped component is loosely coupled from the hoc allowing them to be developed orthogonally with each having little knowledge about each other.
- `Line 41` uses the <a href="https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Techniques/Using_the_aria-describedby_attribute" target="_blank">`aria-describedby`</a> attribute to connect any validation messages that might occur.  A correctly tagged error message element is displayed on lines 45-55 if the `invalid` prop is true.

With this higher order component in place, I can now add the correct labelling to any component such as the `Input` component previously described:

{% codeblock FormControls.js %}
export const FormInput = FormControl(Input);
{% endcodeblock %}

## Validation

The higher order component above takes care of displaying an error below each invalid field but a screen reader will not automatically pick this up unless the user tabs onto the form control.

To counter act this we supply a validation summary.  Below is an exmple of such a validaion summary from <a href="https://govuk-elements.herokuapp.com/errors/example-form-validation-multiple-questions" target="_blank">gov.uk</a> that I based our validation summary on:

{% img /images/accessibility.png %}

At first glance this is complete overkill for 2 fields but in the context of a screen reader, this is great practice.  In the event of an error, focus is placed on the `h2` element in the `ValidationSummary` component and a link is created for each validation error.  The link's `href` is a bookmark link to the invalid element.  When the user tabs off the `h2`, they get an explanation of the error and a chance to jump to the form element and fix the problem.

We used <a href="https://redux-form.com/7.1.2/" target="_blank">redux-form</a> and the following component was used to create a validation summary for all redux-form errors:

{% codeblock ValidationSummary.js %}
export default class ValidationSummary extends Component {
  props: ValidationSummaryProps;

  id: string;

  constructor(props: ValidationSummaryProps) {
    super(props);

    this.id = props.id || prefixId('alert');
  }

  static displayName = 'ValidationSummary';

  render() {
    const {
      syncErrors,
      submitFailed,
      heading,
      description,
      dataSelector
    } = this.props;

    const show = submitFailed && !isEmptyObject(syncErrors);

    const errors = show ? flattenValidationErrors(syncErrors) : [];

    const alertId = `alert-${this.id}`;

    return (
      <div
        className={styles.dialog}
        aria-live="polite"
        aria-labelledby={alertId}
        aria-hidden={!show}
        data-selector={dataSelector}
      >
        <Alert
          id={alertId}
          heading={heading}
          description={description}
          type="error"
          hidden={!show}
        >
          <div className="form-group">
            {errors.map((error: { error: string, id: string }, i: number) =>
              <ul className="current-errors" key={error.id}>
                <li role="tooltip" className={cs('error', 'required')}>
                  <ValidationLink
                    error={error}
                    dataSelector={`${dataSelector}-${i}`}
                  />
                </li>
              </ul>
            )}
          </div>
        </Alert>
      </div>
    );
  }
}
{% endcodeblock %}

## Aria tagging

When going for an accessibility review of the website, a knowledgable person said that the first rule of aria is to not use aria.  I might have been guilty of going overboard with my aria tagging and would welcome any feedback explaining if I have done this.

One definite place to use aria is to inform the screen reader that invisible content is now visible.

Below is a help container that expands and contracts when a link is clicked:

{% codeblock HelpLink.js %}
export const HelpLink = ({ 
  collapsibleId,
  linkText,
  helpText,
  open,
  onClick,
  children
}) =>
  <div className={styles.container}>
    <Link
      button
      onClick={onClick}
      aria-expanded={open}
      aria-controls={collapsibleId}
      tabIndex={0}
    >
      <span
        className={cs(
          styles['link__title'],
          open && styles['link__title__open']
        )}
      >
        <span>
          {linkText}
        </span>
      </span>
    </Link>
    <div
      id={collapsibleId}
      aria-hidden={!open}
      aria-live="polite"
      className={cs(styles['closed'], open && styles['open'])}
      role="region"
      tabIndex={-1}
    >
      {helpText}
      {open && children}
    </div>
  </div>;
{% endcodeblock %}

A combination of `aria-expanded`, `aria-controls` and `aria-live` are used to correctly instruct the screen reader that new content is toggled between visible and invisible states.

## Epilogue

I think we can make all our work much more accessible if we put just a little bit of effort in.  At the very least we should ensure we are keyboard accessible and let the screen reader know that new content is avilable.

If you disagree or agree with any of the above please leave a comment below.
