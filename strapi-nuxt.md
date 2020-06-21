# Strapi Authentication in Nuxt.js

In this tutorial, I'll walk you through on to implement Strapi Authentication
and Authorization in a Nuxt.js app.

A step-by-step guide

We are going to build two projects one Strapi to store and manage users and on
Nuxt app that will serve as the fronted that will access Strapi for
authentication purposes.

Strapi exposes the endpoints required

We'll be using Nuxt's [Auth Module](https://auth.nuxtjs.org/), which is
zero-boilerplate authentication module support for Nuxt. the official
authentication module for Nuxt.

This guide builds on top of Chimezie Enyinnaya's excellent
[tutorial](https://www.digitalocean.com/community/tutorials/implementing-authentication-in-nuxtjs-app),
adapting it to Strapi's specific use case with the following functionality being
added: I won't go in detailed explanation at each step, Chimezie's tutorial
already does it so well. I'll focus on

- Email confirmation for registration
- Password reset
- Token expiration strategy

## Pre-requisites

To follow this tutorial, make sure you have Node.js installed:

- Node.js 12.x
- npm 6.x

Although not required, a basic knowledge of Strapi and Nuxt.js is recommended.

Also, it's important to mention that this guide was written based on the
following versions:

- Strapi 3.0.1 (stable release)
- Nuxt.js 2.12.2

Let's get started!

## Install Strapi

"First things first, we begin by creating a local Strapi app. you to have a
generated Strapi project. If not already generated, you can quickly get started
here to generate one."

```shell
npx create-strapi-app strapi-users --quickstart
```

"If you created your application using --quickstart flag, it will automatically
run your application."

### Enable Email Confirmation

on the left sidebar:
Go to Roles & Permissions under Plugins
in the Advanced Settings tab:
Enable email confirmation

![Enable email confirmation Strapi](./strapi-nuxt-enable-email-confirmation.png)

It comes by default but aMake sure Enable registration route for
is enabled for Public role

"To start your Strapi application you will have to run the following command in
your application folder:"

```shell
npm install
npm run develop
```

### Install and Configure Email Console provider

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

## Create Nuxt project

Now, let's change gears and focus on the frontend app. It will be a Nuxt.js
isomorphic application (server-side rendering + client-side navigation).

To get started quickly, we'll initialize the project using Nuxt's scaffolding
tool:

```shell
npx create-nuxt-app nuxt-auth
```

Go through the guide and make sure to select the following options:

- axios and dotenv in the **Nuxt.js modules** step
- Bulma CSS in the **UI framework** step, which we'll use to style our app

![Create Nuxt app](./create-nuxt-app-options.png)

Once the tool finishes creating the app, we're going to edit the `.env` file in
the project's root directory, adding a new environment variable pointing to the
URL of the Strapi app that's running:

```
API_AUTH_URL=http://localhost:1337
```

Next, navigate to the project's root directory and install the required Nuxt
Auth module dependency:

```shell
cd nuxt-auth
npm install @nuxtjs/auth
```

Once the installation is done, add the module in `nuxt.config.js`:

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

At the top of `nuxt.config.js` add also the following code that loads the
environment variables:

```javascript
require("dotenv").config();
```

One last step we need to do in this file. Configure the base URL that axios will
use when making API requests (which is the env variable we've added previously):

```javascript
axios: {
  baseURL: process.env.API_AUTH_URL
},
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

## Configure Nuxt Auth

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

local: username and password authentication

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

Switch to console where Strapi is running and copy the link to the browser and
visit.

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

## Logout

"We call the logout() of the Auth module. This will delete the user’s token from
localstorage and redirect the user to the homepage."

"Great! We have now done a lot, but something is still missing because the"

## User Profile

"Let’s allow logged in users to view their profile. Create a new profile.vue file
inside the pages directory and paste the code below in it:"

```vue
<template>
  <section class="section">
    <div class="container">
      <h2 class="title">My Profile</h2>
      <div class="content">
        <p>
          <strong>Username:</strong>
          {{ loggedInUser.username }}
        </p>
        <p>
          <strong>Email:</strong>
          {{ loggedInUser.email }}
        </p>
      </div>
    </div>
  </section>
</template>

<script>
import { mapGetters } from "vuex";

export default {
  middleware: "auth",
  computed: {
    ...mapGetters(["loggedInUser"]),
  },
};
</script>
```

## Restricting Register and Login for logged in user

create file `middleware/guest.js`:

```javascript
export default function ({ store, redirect }) {
  if (store.state.auth.loggedIn) {
    return redirect("/");
  }
}
```

`login.vue` and `register.vue`:

```javascript
export default {
  middleware: "guest",
};
```

## Password Reset

user forgets password
add a link to the forgot password page in `login.vue`:

```html
<p>
  <nuxt-link to="/forgot-password">Forgot Password?</nuxt-link>
</p>
```

PRINTSCREEN

Now, lets create the new page `forgot-password.vue`:

"This action sends an email to a user with the link to your reset password page.
This link contains a URL param code which is required to reset user password."

"If account exists, an email will be sent with further instructions"
By not promising the user that an email will be sent and not disclosing whether
the email/username even exists in the database, we 1) encourage them to double
check their credentials (in case they don’t get the link) and 2) prevent
phishing attacks.

"This action will send the user an email that contains a URL with the needed code
for the reset password. The URL must link to your reset password form in your
frontend application.

To configure it you will have to go in the Roles & Permissions settings and
navigate to the Advanced Settings tab":

http://localhost:3000/reset-password

PRINTSCREEN

Create `/pages/reset-password.vue`:

This action will reset the user password.

"Now that the code is ready, you have to make sure that the API consumer or user
has the permissions to access all the necessary actions. For this, you would
have to open up your UI Admin Panel and goto the ‘Roles & Permissions’ menu. If
your using the API publicly, click on the role ‘Public’ and make sure all the
actions under every plugin dropdowns of ‘Permissions’ section are checked (or
at least all the ones relating to email’s send action)."

PRINTSCREEN

![Strapi Reset Password](./strapi-reset-password.png)

## Token Expiration

One last thing we need to do before wrapping up.
once we get a 401 unauthorized error we need to redirect the user to the login page.

create a new file `plugins/axios.js`:

```javascript
export default function ({ $axios, redirect }) {
  $axios.onError((error) => {
    const code = parseInt(error.response && error.response.status);
    if (code === 401) redirect("/login");
  });
}
```

in `nuxt.config.js` import the file we've just created:

```javascript
plugins: ['~plugins/axios'],
```

## Conclusion

motivated me to consolidate all my findings and write this article. Hope it
helps!
"We hope you find this project useful for learning testing and Cypress."

https://youtu.be/0hAmccuaK5Q
https://www.npmjs.com/package/strapi-provider-email-console
