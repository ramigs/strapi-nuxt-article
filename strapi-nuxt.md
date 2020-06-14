# Strapi Authentication in a Nuxt.js App

Implementing Strapi Authentication and Authorization in a Nuxt.js SSR project,
using @nuxtjs/auth.

In this tutorial I'll be showing you how I built a Vue.js app with
Authentication using Cosmic JS and AWS Lambda prior to deploying to Netlify.

is going to walk you through on how to implement Strapi
authentication in a Nuxt.js app.

We'll be using the Nuxt's [Auth Module](https://auth.nuxtjs.org/)
Zero-boilerplate authentication support for Nuxt

This tutorial builds on top of Chimezie Enyinnaya's awesome
[tutorial](https://www.digitalocean.com/community/tutorials/implementing-authentication-in-nuxtjs-app),
adapting it to Strapi and adding the following:

- Email confirmation for registration
- Refresh token strategy
- Password recovery

## Pre-requisites

You will be required to have Node JS and npm before starting. Make sure you already have them installed. If not you can find them here: https://nodejs.org
Before we move on, I’d like to mention that although not mandatory, a working
knowledge of the following technologies is beneficial:

Before you begin, you'll need:

- Node.js 12.x
- npm 6.x

Strapi 3.0.1
Nuxt.js v2.12.2

Let's get started!

## Installing Strapi

"First things first, we begin by creating a local Strapi app. you to have a generated
Strapi project (currently v3.0.0-alpha.14). If not already generated, you can
quickly get started here to generate one."

```shell
npx create-strapi-app strapi-users --quickstart
```

"If you created your application using --quickstart flag, it will automatically
run your application."

### Enabling Email Confirmation

on the left sidebar:
Go to Roles & Permissions under Plugins
in the Advanced Settings tab:
Enable email confirmation

![Enable email confirmation Strapi](./strapi-nuxt-enable-email-confirmation.png)

It comes by default but aMake sure Enable registration route for
is enabled for Public role

```shell
npm install
npm run develop
```

### Installing and Configuring Email Console provider

"Establishing some technical details before we start — we will be using Strapi’s
default email provider (local email system) which as I mentioned is built-in or
already installed in every generated project. If you want to use another
provider, you would need to first install the corresponding npm package. You can
check npmjs.com for all the email providers
[list](https://www.npmjs.com/search?q=strapi-email-)."

Thanks to the plugin Email, you can send email from your server or externals
providers such as Sendgrid.

By default Strapi provides a local email system (sendmail). If you want to use a
third party to send emails, you need to install the correct provider module.
Otherwise you can skip this part and continue to Configure your provider.

You can check all the available providers developed by the community on
npmjs.org - Providers list

To install the new provider run:

```shell
cd strapi-users
npm install strapi-provider-email-console
```

After installing your provider you will need to add some settings in
config/plugins.js. Check the README of each provider to know what configuration
settings the provider needs.

`./config/plugins.js`:

```javascript
module.exports = ({ env }) => ({
  email: {
    provider: "console",
  },
});
```

## Creating our Nuxt App

Now, let's change gears and look at this tutorial's core app.

To get started quickly, we'll initialize the project using Nuxt's scaffolding
tool:

```shell
npx create-nuxt-app nuxt-auth
```

Go through the guide and make sure to select the following options:

- axios and dotenv in the **Nuxt.js modules** step
- Bulma CSS in the **UI framework** step, which we'll use to style our app

![Create Nuxt app](./create-nuxt-app-options.png)

Once the tool finishes creating our Nuxt app,

Let's edit the .env file and add an environment variable for Strapi's API URL:
"Next, you'll need to make sure you have a .env file in your root project
directory to change the VUE_APP_API_HOST variable to point to the lambda server
you just started running."

```
API_AUTH_URL=http://localhost:1337
```

Enable vuex store by creating `store/index.js`:

```javascript
export const getters = {
  isAuthenticated(state) {
    return state.auth.loggedIn;
  },

  loggedInUser(state) {
    return state.auth.user;
  },
};
```

Now, install also the required
Nuxt Auth module dependency:

```shell
cd nuxt-auth
npm install @nuxtjs/auth
```

Edit `nuxt.config.js` adding the module we've just installed:

```javascript
modules: [
  // Doc: https://github.com/nuxt-community/modules/tree/master/packages/bulma
  '@nuxtjs/bulma',
  // Doc: https://axios.nuxtjs.org/usage
  '@nuxtjs/axios',
  // Doc: https://github.com/nuxt-community/dotenv-module
  '@nuxtjs/dotenv',
  '@nuxtjs/auth'
],
```

In the top of nuxt.config.js require and configure dotenv:

```javascript
require("dotenv").config();
```

"axios. Paste the code below into `nuxt.config.js`:"

```javascript
axios: {
  baseURL: process.env.API_AUTH_URL
},
```

## Configuring Nuxt Auth

"Next, we need to set up the modules. Paste the code below into `nuxt.config.js`:"

```javascript
/*
 ** Auth module configuration
 ** See https://auth.nuxtjs.org/schemes/local.html#options
 */
auth: {
  strategies: {
    local: {
      endpoints: {
        login: {
          url: 'auth/local',
          method: 'post',
          propertyName: 'jwt'
        },
        user: {
          url: 'users/me',
          method: 'get',
          propertyName: false
        },
        logout: false
      }
    }
  }
},
```

"Here, we set the base URL (which is that of our API from earlier on) that Axios
will use when making requests. Then we define the authentication endpoints for
the local strategy corresponding to those on our API. On successful
authentication, the token will be available in the response as a token object
inside a data object, hence why we set propertyName to data.token. Similarly,
the response from the /me endpoint will be inside a data object. Lastly, we set
logout to false since our API doesn’t have an endpoint for logout. We’ll just
remove the token from localstorage when a user logs out."

## Navbar Component

`Navbar.vue` and replace it content with the following:

```vue
<template>
  <nav class="navbar" role="navigation" aria-label="main navigation">
    <div class="navbar-brand">
      <nuxt-link class="navbar-item" to="/">Home</nuxt-link>

      <a
        role="button"
        class="navbar-burger burger"
        aria-label="menu"
        aria-expanded="false"
        data-target="navbarBasicExample"
      >
        <span aria-hidden="true"></span>
        <span aria-hidden="true"></span>
        <span aria-hidden="true"></span>
      </a>
    </div>

    <div id="navbarBasicExample" class="navbar-menu">
      <!--       <div class="navbar-start">
        <div class="navbar-item has-dropdown is-hoverable">
          <a class="navbar-link">
            My Account
          </a>

          <div class="navbar-dropdown">
            <nuxt-link class="navbar-item" to="/profile">My Profile</nuxt-link>
            <hr class="navbar-divider" />
            <a class="navbar-item">Logout</a>
          </div>
        </div>
      </div> -->

      <div class="navbar-end">
        <div class="navbar-item">
          <div class="buttons">
            <nuxt-link class="button is-primary" to="/register">
              <strong>Register</strong>
            </nuxt-link>
            <nuxt-link class="button is-light" to="/login">
              Log in
            </nuxt-link>
          </div>
        </div>
      </div>
    </div>
  </nav>
</template>

<script>
export default {
  mounted() {
    // Get all "navbar-burger" elements
    const $navbarBurgers = Array.prototype.slice.call(
      document.querySelectorAll(".navbar-burger"),
      0
    );
    // Check if there are any navbar burgers
    if ($navbarBurgers.length > 0) {
      // Add a click event on each of them
      $navbarBurgers.forEach((el) => {
        el.addEventListener("click", () => {
          // Get the target from the "data-target" attribute
          const target = el.dataset.target;
          const $target = document.getElementById(target);
          // Toggle the "is-active" class on both the "navbar-burger" and the "navbar-menu"
          el.classList.toggle("is-active");
          $target.classList.toggle("is-active");
        });
      });
    }
  },
};
</script>
```

"The Navbar component contains links to login or register, links to view profile
or logout."

## Default Layout

"Next, let’s update the default layout to make use of the Navbar component. Open
layouts/default.vue and replace its content with the following:"

```vue
<template>
  <div>
    <Navbar />
    <nuxt />
  </div>
</template>

<script>
import Navbar from "~/components/Navbar";

export default {
  components: {
    Navbar,
  },
};
</script>
```

## Homepage

"Also, let’s update the homepage. Open pages/index.vue and replace its content
with the following:"

```vue
<template>
  <section class="section">
    <div class="container">
      <h1 class="title">Nuxt Auth</h1>
    </div>
  </section>
</template>
```

This is what we have so far:

## Notification Component

If there is an error, the error message is displayed by a Notification
component, which we’ll create shortly.

Before we test the user registration out, let’s create the Notification
component. Create a new Notification.vue file inside components and paste the
code below in it:

```vue
<template>
  <div class="notification is-danger">
    {{ message }}
  </div>
</template>

<script>
export default {
  name: "Notification",
  props: ["message"],
};
</script>
```

## Register

Inside the pages directory, create a new register.vue file and paste the code below in it:

```vue
<template>
  <section class="section">
    <div class="container">
      <div class="columns">
        <div class="column is-4 is-offset-4">
          <h2 class="title has-text-centered">Register User</h2>

          <Notification v-if="success" type="success" :message="success" />
          <Notification v-if="error" type="danger" :message="error" />

          <form v-if="!success" method="post" @submit.prevent="register">
            <div class="field">
              <label class="label">Username</label>
              <div class="control">
                <input
                  v-model="username"
                  type="text"
                  class="input"
                  name="username"
                  required
                />
              </div>
            </div>
            <div class="field">
              <label class="label">Email</label>
              <div class="control">
                <input
                  v-model="email"
                  type="email"
                  class="input"
                  name="email"
                  required
                />
              </div>
            </div>
            <div class="field">
              <label class="label">Password</label>
              <div class="control">
                <input
                  v-model="password"
                  type="password"
                  class="input"
                  name="password"
                  required
                />
              </div>
            </div>
            <div class="control">
              <button type="submit" class="button is-dark is-fullwidth">
                Register
              </button>
            </div>
          </form>

          <div class="has-text-centered" style="margin-top: 20px">
            Already got an account? <nuxt-link to="/login">Login</nuxt-link>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import Notification from "~/components/Notification";

export default {
  components: {
    Notification,
  },

  data() {
    return {
      username: "",
      email: "",
      password: "",
      success: null,
      error: null,
    };
  },

  methods: {
    async register() {
      try {
        this.$axios.setToken(false);
        await this.$axios.post("auth/local/register", {
          username: this.username,
          email: this.email,
          password: this.password,
        });
        this.success = `A confirmation link has been sent to your email account. \
        Please click on the link to confirm your email to complete the registration process.`;
      } catch (e) {
        this.error = e.response.data.message[0].messages[0].message;
      }
    },
  },
};
</script>
```

"This contains a form with three fields: username, email, and password. Each
field is bound to a corresponding data on the component. When the form is
submitted, a register method will be called. Using the Axios module, we make a
post request to the /register endpoint, passing along the user data. If the
registration was successful, we make use of the Auth module’s loginWith(), using
the local strategy and passing the user data to log the user in. Then we
redirect the user to the homepage. If there is an error during the registration,
we set the error data as the error message gotten from the API response.

If there is an error, the error message is displayed by a Notification
component, which we’ll create shortly."

### Confirm Email

Switch to console where Strapi is running and copy the link to the browser and visit.

This will change confirm the user.

## Login

"Now let’s allow returning users ability to log in. Create a new login.vue file
inside the pages directory and paste the code below in it:"

```vue
<template>
  <section class="section">
    <div class="container">
      <div class="columns">
        <div class="column is-4 is-offset-4">
          <h2 class="title has-text-centered">Log In</h2>

          <Notification v-if="error" type="danger" :message="error" />

          <form method="post" @submit.prevent="login">
            <div class="field">
              <label class="label">Email</label>
              <div class="control">
                <input
                  v-model="email"
                  type="email"
                  class="input"
                  name="email"
                />
              </div>
            </div>
            <div class="field">
              <label class="label">Password</label>
              <div class="control">
                <input
                  v-model="password"
                  type="password"
                  class="input"
                  name="password"
                />
              </div>
            </div>
            <div class="control">
              <button type="submit" class="button is-dark is-fullwidth">
                Log In
              </button>
            </div>
          </form>
          <div class="has-text-centered" style="margin-top: 20px">
            <p>
              Don't have an account?
              <nuxt-link to="/register">Register</nuxt-link>
            </p>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import Notification from "~/components/Notification";

export default {
  components: {
    Notification,
  },

  data() {
    return {
      email: "",
      password: "",
      error: null,
    };
  },

  methods: {
    async login() {
      try {
        await this.$auth.loginWith("local", {
          data: {
            identifier: this.email,
            password: this.password,
          },
        });
        this.$router.push("/");
      } catch (e) {
        this.error = e.response.data.message[0].messages[0].message;
      }
    },
  },
};
</script>
```

"This is quite similar to the register page. The form contains two fields: email
and password. When the form is submitted, a login method will be called. Using
the Auth module loginWith() and passing along the user data, we log the user in.
If the authentication was successful, we redirect the user to the homepage.
Otherwise set error to the error message gotten from the API response. Again, we
are using the Notification component from earlier on to display the error
message."

## User Profile

## Logout

## Recover Password

## Conclusion

"So there you have it — a static API."

https://youtu.be/0hAmccuaK5Q
https://www.npmjs.com/package/strapi-provider-email-console
