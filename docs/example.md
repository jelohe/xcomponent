# xcomponent example

Let's create a login component. We want the user to be able to log in on our site, and to notify the parent window
that the user has logged in, without exposing any of the users credentials to the parent window.

Take a look at the [demos](./demo/index.md) to see this example in action.

### As the component creator

First I'd create a spec for the component's interface:

```javascript
var MyLoginComponent = xcomponent.create({

    // The html tag used to render my component

    tag: 'my-login-component',

    // The url that will be loaded in the iframe or popup, when someone includes my component on their page

    url: 'http://www.my-site.com/my-login-component',

    // The size of the component on their page

    dimensions: {
        width: 400,
        height: 200
    },

    // The properties they can (or must) pass down to my component

    props: {

        prefilledEmail: {
            type: 'string',
            required: false
        },

        onLogin: {
            type: 'function',
            required: true
        }
    }
});
```

This spec describes everything needed to render the component on the parent page, including the props the component expects.

Now we need to actually implement the business logic of the component -- the code that will run inside the iframe.

I just need to use `window.xprops` to get the props that are passed down.

```html
<!-- Pull in the login component we defined above -->

<script src="./my-login-component.js"></script>

<!-- Set up a login form -->

<form>
    <input id="email" type="text" />
    <input id="password" type="password" />
    <button id="login">Log In</button>
</form>

<script>
    // Pre-polulate the email field, if we're passed an email

    if (window.xprops.prefilledEmail) {
        document.querySelector('#email').value = window.xprops.prefilledEmail;
    }

    // Handle the button click to log the user in

    document.querySelector('#login').addEventListener('click', function(event) {
        event.preventDefault();

        var email    = document.querySelector('#email').value;
        var password = document.querySelector('#password').value;

        jQuery.post('/api/login', { email: email, password: password }, function() {

            // Since we had a successful login, call-back the parent page to let them know which user logged in:

            window.xprops.onLogin(email);
        });
    });
</script>
```

Now anyone can render the component we defined onto their page!


### As the component user

My life is even easier. I just need to drop in your component onto my page:

```html
<!-- Pull in the login component we defined above -->

<script src="./my-login-component.js"></script>

<!-- Set up a container for the iframe to be rendered into -->

<div id="container"></div>

<script>
    // Render the component

    MyLoginComponent.render({

        prefilledEmail: 'foo@bar.com',

        onLogin: function(email) {
            console.log('User logged in with email:', email);
        }

    }, '#container');
</script>
```

This is even easier if you're using a supported framework like React, Ember or Angular -- xcomponent will automatically
set up bindings for these frameworks:

### React / JSX

Drop the component in any `render()` method:

```javascript
render: function() {
    return (
        <MyLoginComponent.react prefilledEmail='foo@bar.com' onLogin={onLogin} />
    );
}
```

### Angular

Specify the component name as a dependency to your angular app:

```javascript
angular.module('myapp', ['my-login-component'])
```

Include the tag in one of your templates (don't forget to use dasherized prop names):

```html
<my-login-component prefilled-email="foo@bar.com" on-login="onLogin" />
```

---

And we're done! Notice how I never had to write any code to create an iframe, or send post messages? That's all taken care of for you.
When you call `this.props.onLogin(email);` it looks like you're just calling a function, but in reality `xcomponent` is transparently
turning that callback into a post-message and relaying it to the parent for you.

- [Now try building a cross-domain React component!](https://medium.com/@bluepnume/creating-a-cross-domain-react-component-with-xcomponent-fbcccc4778fd#.73jnwv44c)