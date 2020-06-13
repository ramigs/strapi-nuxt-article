# Strapi Authentication in a Nuxt.js App

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

Tailwind (because I've been wanting to
learn it for quite some time now :) also checkout the new nuxt/tailwind module.
https://www.npmjs.com/package/@nuxtjs/tailwindcss

## Pre-requisites

Before we move on, I’d like to mention that although not mandatory, a working
knowledge of the following technologies is beneficial:

Before you begin, you'll need:

- Node.js 12.x
- npm 6.x

Strapi 3.0.1
Nuxt.js v2.12.2

Let's begin!

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
- Tailwind CSS in the **UI framework** step, which we'll use to style our app

![Create Nuxt app](./create-nuxt-app-options.png)

Once the tool finishes creating our Nuxt app,

Let's edit the .env file and add an environment variable for Strapi's API URL:

```
API_AUTH_URL=http://localhost:1337
```

Enable vuex store by creating `store/index.js`.

Now, install also the required
Nuxt Auth module dependency:

```shell
cd nuxt-auth
npm install @nuxtjs/auth
```

Edit `nuxt.config.js` adding the module we've just installed:

```javascript
modules: [
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

"Next, we need to set up the modules. Paste the code below into `nuxt.config.js`:"

```javascript
/*
 ** Auth Module configuration
 ** See https://auth.nuxtjs.org/api/options.html
*/
auth: {
  strategies: {
    local: {
      endpoints: {
        login: {
          url: `${process.env.API_AUTH_URL}/auth/local`,
          method: 'post',
          propertyName: 'jwt'
        },
        user: {
          url: `${process.env.API_AUTH_URL}/users/me`,
          method: 'get',
          propertyName: false
        },
        logout: false
      },
      autoFetchUser: true
    }
  }
},
```

## Configuring Nuxt Auth

## Creating Navbar

## Register

- Confirm Email

## Login

## User Profile

## Logout

## Recover Password

## Conclusion

"So there you have it — a static API."

https://youtu.be/0hAmccuaK5Q
https://www.npmjs.com/package/strapi-provider-email-console
