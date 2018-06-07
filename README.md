# Apollo inside of NuxtJS

* Nuxt.js module to use [vue-apollo](https://github.com/Akryum/vue-apollo) (integrates graphql-tag loader to parse `.gql` & `.graphql` files)
* uses internally same approach as `vue-cli-plugin-apollo`

[![npm version](https://img.shields.io/npm/v/@nuxtjs/apollo.svg)](https://www.npmjs.com/package/@nuxtjs/apollo)
[![license](https://img.shields.io/github/license/nuxt-community/apollo-module.svg)](https://github.com/nuxt-community/apollo-module/blob/master/LICENSE)


## Setup

Install apollo module:

```bash
npm install --save @nuxtjs/apollo
```

Add `@nuxtjs/apollo` to `modules` section of `nuxt.config.js`

- clientConfigs: `Object` Config passed to ApolloClient
  - default: `Object`
  - otherClient: `Object` (Optional)

```js
{
  // Add apollo module
  modules: ['@nuxtjs/apollo'],

  // Give apollo module options
  apollo: {
    clientConfigs: {
      default: {
        // required  
        httpEndpoint: 'http://localhost:4000',
        // You can use `wss` for secure connection (recommended in production)
        // Use `null` to disable subscriptions
        wsEndpoint: 'http://localhost:4000', // optional
        // LocalStorage token
        tokenName: 'apollo-token', // optional
        // Enable Automatic Query persisting with Apollo Engine
        persisting: false, // Optional
        // Use websockets for everything (no HTTP)
        // You need to pass a `wsEndpoint` for this to work
        websocketsOnly: false // Optional
      },
      test: {
        httpEndpoint: 'http://localhost:5000',
        wsEndpoint: 'http://localhost:5000',
        tokenName: 'apollo-token'
      }
    }
  }
}
```

All available options you find [here](https://github.com/Akryum/vue-cli-plugin-apollo/blob/master/graphql-client/src/index.js#L15)

## Configuration possibilities

Check out [official vue-apollo-cli](https://github.com/Akryum/vue-cli-plugin-apollo) where possible usecases are presented.

## Usage

See [Official example](https://github.com/nuxt/nuxt.js/tree/dev/examples/vue-apollo) and [vue-apollo](https://github.com/Akryum/vue-apollo).

## Build in methods

On top of auto-configuration of apollo-module you have following methods available
```js
 this.$apolloHelpers.onLogin(token, /* if not default you can pass in client as second argument */)
 this.$apolloHelpers.onLogout(/* if not default you can pass in client as second argument */)
 this.$apolloHelpers.getToken(/* you can provide named tokenName if not 'apollo-token' */)
```
Check out the [full example](https://github.com/nuxt-community/apollo-module/tree/master/test/fixture)

#### User login
```js
methods:{
  async onSubmit () {
    const credentials = this.credentials
    try {
        const res = await this.$apollo.mutate({
            mutation: authenticateUserGql,
            variables: credentials
        }).then(({data}) => data && data.authenticateUser)
        await this.$apolloHelpers.onLogin(res.token)
    } catch (e) {
        console.error(e)
    }
  },
}
```
#### User logout
```js
methods:{
  async onLogout () {
    await this.$apolloHelpers.onLogout()
  },
}
```

#### getToken
```js
// middleware/isAuth.js
  export default function ({app, error}) {
    const hasToken = !!app.$apolloHelpers.getToken()
    if (!hasToken) {
        error({errorCode:503, message:'You are not allowed to see this'})
    }
}
```

#### Examples to access the defaultClient of your apolloProvider
##### Vuex actions
```js
export default {
  actions: {
    foo (store, payload) {
      let client = this.app.apolloProvider.defaultClient
    }
  }
}
```

##### asyncData/fetch method of page component
```js
export default {
  asyncData (context) {
    let client = context.app.apolloProvider.defaultClient
  }
}
```

##### onServerInit
```js
export default {
  nuxtServerInit (store, context) {
    let client = context.app.apolloProvider.defaultClient
  }
}
```


##### access client or call mutations of any method inside of component
```js
export default {
  methods:{
    foo(){
      // receive the associated Apollo client 
      const client = this.$apollo.getClient()

      // most likely you would call mutations like following:
      this.$apollo.mutate({mutation, variables})
    }
  }
}
```

##### query on any component
```js
export default {
  apollo: {
    foo: {
      query: fooGql,
      variables () {
        return {
          myVar: this.myVar
        }
      }
    }
  }
}
```

#### Add GQL file recognition on node_modules
```js
  apollo: {
    clientConfigs: {
      default: '~/apollo/client-configs/default.js'
    },
    includeNodeModules: true
  }
```

## Upgrade

### Upgrade Guide apollo-module v3 => v4

Version 4 of this module leaves you with zero configuration. This means we use the best possible approach available from `vue-cli-plugin-apollo` and the same configuration behaviour. This means you don't need to wire up your own configuration, simply pass 

Edit your configuration as following:
```js
// nuxt.config.js
apollo:{
 clientConfigs:{
  default:{
    httpEndpoint: YOUR_ENDPOINT,
    wsEndpoint: YOUR_WS_ENDPOINT
  }
 }
}
```

### Upgrade Guide apollo-client v1 => v2

Version 3 of this module is using apollo-client 2.x. You need to make sure to update all your middle/afterware according to the upgrade guide of apollo-client. Check this source for a reference: https://github.com/apollographql/apollo-client/blob/master/Upgrade.md

## Troubleshooting 

### Proxies

CORS errors are most often resolved with proxies.  If you see a Cross-Origin-Request error in your client side console look into setting up a proxy.  Check out https://github.com/nuxt-community/proxy-module for quick and straight forward setup.

###  ctx.req.session - req is undefined

This is just a placeholder.  You'll want to replace it with whatever storage mechanism you choose to store your token.
Here is an example using local storage : https://github.com/Akryum/vue-apollo/issues/144
