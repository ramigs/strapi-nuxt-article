# Strapi Authentication in Nuxt.js

This tutorial is a step-by-step guide on how to implement Strapi Authentication
and in Nuxt.js app.

We are going to build two projects:

- Backend: Strapi app to store and manage users
- Frontend: Nuxt app that will use Strapi's API for authentication purposes

We'll be relying on Nuxt's [Auth Module](https://auth.nuxtjs.org/), which is the
official zero-boilerplate authentication module for Nuxt.js.

JWT token username password

We will also be implementing some basic authorization features, such as
restricting certain pages to only logged in users.

This guide builds on top of Chimezie Enyinnaya's excellent
[work](https://www.digitalocean.com/community/tutorials/implementing-authentication-in-nuxtjs-app),
adapting it to Strapi's specific use case, and also adding the following
features:

- Email confirmation for registration
- Password reset
- Token expiration strategy

I won't go into detailed explanation at every step. Instead, I'll focus on
Strapi and the added features/functionality.

For everything else, I recommend you refer to Chimezie's tutorial already does
it so well.

## Pre-requisites

To follow this tutorial, make sure you have Node.js installed:

- Node.js 12.x
- npm 6.x

Although not required, a basic knowledge of Strapi and Nuxt.js is recommended.

> Disclaimer: this tutorial was written based on Strapi 3.0.1 (stable release)
> and Nuxt.js 2.12.2. It should work on different versions too, but you may need
> to make some adaptations it here and there.

Let's get started!

## Install Strapi

First things first, we begin by creating a Strapi project:

```shell
npx create-strapi-app strapi-users --quickstart
```

We've used the `--quickstart` flag that installs Strapi with default settings.
It also tells Strapi to use SQLite as the database engine and automatically run
our application:

> In our installation, we used the default database, SQLite, because it doesn’t
> require a dedicated database server. Instead, the database is just a single
> file. For practicability, we'll be using it in this tutorial, but the steps in
> this guide also apply to different database engines.

After the installation, Strapi will automatically launch in your browser, asking
you to create an administrator account.

To finish the setup and secure your app, go ahead and create the root admin
user, by entering the necessary information below:

PRINTSCREEN

From now on, you can access Strapi's local Admin interface by visiting
[http://localhost:1337/admin](http://localhost:1337/admin).

## Enable email confirmation

By default, the user registration process in Strapi does not include email
confirmation.

To enable it, go to **Roles & Permissions** from the left sidebar of the Admin
dashboard. Click the Advanced Settings tab, enable the email confirmation option
and save:

![Enable email confirmation Strapi](./strapi-nuxt-enable-email-confirmation.png)

## Email console provider

Thanks to Strapi's Email plugin, we can have a Strapi instance send emails from
our server's local email system or externals providers (e.g., SendGrid).

Email setup and configuration is not within the scope of this article. We just
want to make sure that the emails are being fired when expected and inspect
their content.

For that, we're going to use a ["fake" email
provider](https://github.com/aperron/strapi-provider-email-console) for
development, that simply outputs the emails to the console.

So, let's install the email provider module. Stop the Strapi app that's running
and install the new email provider:

```shell
cd strapi-users
npm install strapi-provider-email-console
```

Once the installation is done, you'll need to add the provider in
`./config/plugins.js`:

```javascript
module.exports = ({ env }) => ({
  email: {
    provider: "console",
  },
});
```

Since we've used `--quickstart` the first time we ran our Strapi app, before
running it in development mode we need to install its dependencies:

```shell
npm install
```

Once that's done, start the Strapi app in development mode with the command:

```shell
npm run develop
```

## Create a Nuxt app

Now, let's change gears and focus on the frontend app. It will be a Universal
SSR Nuxt.js app.

To get started quickly, we'll create the project using Nuxt's scaffolding tool:

```shell
npx create-nuxt-app nuxt-auth
```

Go through the guide and make sure the following options are selected:

- axios and dotenv in the **Nuxt.js modules** step
- Bulma CSS in the **UI framework** step, which we'll use to style our app

![Create a Nuxt app](./create-nuxt-app-options.png)

Strapi's API URL will vary depending on the environment. So, we want to have it
loaded dynamically from an environment variable.

Once the tool finishes creating the app, we're going to edit the `.env` file in
the project's root directory, and add a new environment variable that points to
the URL of the Strapi app that's running:

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

At the top of `nuxt.config.js`, add also the following code to load the
environment variables:

```javascript
require("dotenv").config();
```

One last step we need to do in this file. Configure the base URL that axios will
use when making API requests (which is the environment variable we've added
previously):

```javascript
axios: {
  baseURL: process.env.API_AUTH_URL
},
```

## Configure Nuxt Auth

The Auth module uses Vuex's state management to store the user authentication
status and user info.

Enable the Vuex store by creating a `index.js` file inside the `store` directory
with the store getters:

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

Now, we are ready to configure the Auth module. This is done by specifying the
authentication endpoints for the local strategy with Strapi's authentication
endpoints. The local strategy is based on username and password authentication.

Paste the code below into `nuxt.config.js`:

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

We've configured two endpoints:

- `login`: authenticates the user. On successful authentication, the JWT token
  will be available in the `jwt` property of the response object.
- `user`: retrieves the authenticated user's info. If the user is authenticated,
  the JWT token will be sent in the request, allowing Strapi to identify the
  user. Since the response object is already the user info, we set
  `propertyName` to `false`.

We've also disabled the `logout` endpoint, since logging out a user is only done
locally, and doesn't require any request to Strapi's API. The token is simply
removed the from the local storage when the user logs out.

## Navbar Component

Create a file `./components/Navbar.vue` with the following code:

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
      <div v-if="isAuthenticated" class="navbar-start">
        <div class="navbar-item has-dropdown is-hoverable">
          <a class="navbar-link">
            {{ loggedInUser.username }}
          </a>

          <div class="navbar-dropdown">
            <a class="navbar-item" href="/profile">My Profile</a>
            <hr class="navbar-divider" />
            <a class="navbar-item" @click="logout">Logout</a>
          </div>
        </div>
      </div>

      <div v-if="!isAuthenticated" class="navbar-end">
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
```

Let us make component functional by adding the following `<script></script>`
code:

```vue
<script>
import { mapGetters } from "vuex";

export default {
  computed: {
    ...mapGetters(["isAuthenticated", "loggedInUser"]),
  },
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
  methods: {
    async logout() {
      await this.$auth.logout();
    },
  },
};
</script>
```

The code in the `mounted` hook comes from [Bulma's Navbar
documentation](https://bulma.io/documentation/components/navbar/). Its purpose
is to toggle the display of the `navbar-menu` when the `navbar-burger` icon is
clicked.

We've also added the `logout` method that's triggered "of the Auth module, which
deletes the JWT token from the local storage and redirects the user to the
homepage.

## Default Layout

Edit the file `./layouts/default.vue` and replace its content with the
following:

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

Before we test the user registration, let’s create a Notification component so
we can get some feedback in the browser.

If everything goes as expected it should display a success message. Otherwise,
an error message should be displayed.

Create a new `./components/Notification.vue` file and paste in it the code
below:

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
post request to the /register endpoint, passing along the user data.

If the registration was successful, we display a success message, requesting
that the user completes the registration process by clicking the confirmation
link in sent email:

The user simply has to check a confirmation email they received when they signed
up. Within that email, there is a link which will redirect you back to your app.

PRINTSCREEN

If an error occurs, the error message is displayed by the Notification component
we've created previously:

PRINTSCREEN

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
local storage and redirect the user to the homepage."

```javascript
methods: {
    async logout() {
      await this.$auth.logout()
    }
  }
```

## User Profile

Time for the user profile page.

Create a new `/pages/profile.vue` file with the following code:

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

The `auth` middleware guarantees that only logged in users will be able to
access the page.

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

This action will send an email to the user with a link containing the required
code for the reset password. must link to your reset password form in your
frontend application.

To configure it you will have to go in the Roles & Permissions settings and
navigate to the Advanced Settings tab
`http://localhost:3000/reset-password`:

PRINTSCREEN

Create a `./pages/reset-password.vue` file and paste the following code into it:

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

"Great! We have now done a lot, but something is still missing because the"

One last thing before we wrap up. once we get a 401 unauthorized error we need
to redirect the user to the login page.

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
