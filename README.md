# Auth Astro

Auth Astro is the easiest way to add Authentication to your Astro Project. It wraps the core of [Auth.js](https://authjs.dev/) into an Astro integration, which automatically adds the endpoints and handles everything else.

#### Now supporting up to Astro 4
(**disclaimer**: Please don't confuse this package with [astro-auth](https://github.com/astro-community/astro-auth))

# Installation

The easiest way to get started is adding this package using the astro cli. 

```bash
npm run astro add auth-astro
```
This will install the package and required peer-dependencies and add the integration to your config.
You can now jump to [configuration](#configuration)

Alternatively, you can install the required packages on your own.

```bash
npm install auth-astro@latest @auth/core@^0.18.6
```

> **Note**: If you´re using `pnpm` you must also install cookie: `pnpm i cookie`

Next, you need to [add the integration to your astro config](https://docs.astro.build/en/guides/integrations-guide/#using-integrations) by importing it and listing it in the integrations array.

## Configuration

Create your [auth configuration](https://authjs.dev/getting-started/providers/oauth-tutorial) file in the root of your project.

```ts title="auth.config.ts"
// auth.config.ts
import GitHub from '@auth/core/providers/github'
import { defineConfig } from 'auth-astro'

export default defineConfig({
	providers: [
		GitHub({
			clientId: import.meta.env.GITHUB_CLIENT_ID,
			clientSecret: import.meta.env.GITHUB_CLIENT_SECRET,
		}),
	],
})
```

Some OAuth Providers request a callback URL be submitted alongside requesting a Client ID, and Client Secret. The callback URL used by the providers must be set to the following, unless you override the prefix field in the configuration:

```
[origin]/api/auth/callback/[provider]

// example
// http://localhost:4321/api/auth/callback/github
```

### Setup Environment Variables

Generate an auth secret by running `openssl rand -hex 32` in a local terminal or by visiting [generate-secret.vercel.app](https://generate-secret.vercel.app/32), copy the string, then set it as the `AUTH_SECRET` environment variable describe below.

Next, set the `AUTH_TRUST_HOST` environment variable to `true` for hosting providers like Cloudflare Pages or Netlify.
```sh
AUTH_SECRET=<auth-secret>
AUTH_TRUST_HOST=true
```

#### Deploying to Vercel?
Setting `AUTH_TRUST_HOST` is not needed, as we also check for an active Vercel environment.

### Using API Context and Runtime Environment in your Configuration

Some database providers like Cloudflare D1 provide bindings to your databases in the runtime
environment, which isn't accessible statically, but is provided on each request. You can define
your configuration as a function which accepts the APIContext (for API Routes) or the Astro 
global value (for Astro pages/components).

```ts title="auth.config.ts"
// auth.config.ts
import Patreon from "@auth/core/providers/patreon";
import { DrizzleAdapter } from "@auth/drizzle-adapter";
import { defineConfig } from "auth-astro";
import type { UserAuthConfig } from "auth-astro/src/config";
import { drizzle } from "drizzle-orm/d1";

export default {
	config: (ctx) => {
		const { env } = ctx.locals.runtime;
		const db = env.DB;
		return defineConfig({
			secret: env.AUTH_SECRET,
			trustHost: env.AUTH_TRUST_HOST === "true",
			adapter: DrizzleAdapter(drizzle(db)),
			providers: [
				Patreon({
					clientId: env.AUTH_PATREON_ID,
					clientSecret: env.AUTH_PATREON_SECRET,
				}),
			],
		});
	},
} satisfies UserAuthConfig;
```

### requirements
- node version `>= 17.4`
- astro config set to output mode `server`
- [ssr](https://docs.astro.build/en/guides/server-side-rendering/) enabled in your astro project

resources:
- [enabling ssr in your project](https://docs.astro.build/en/guides/server-side-rendering/#enabling-ssr-in-your-project)
- [adding an adapter](https://docs.astro.build/en/guides/server-side-rendering/#adding-an-adapter)

# Usage

Your authentication endpoints now live under `[origin]/api/auth/[operation]`. You can change the prefix in the configuration.

## Accessing your configuration

In case you need to access your auth configuration, you can always import it by
```ts
import authConfig from 'auth:config'
```

## Sign in & Sign out

Astro Auth exposes two ways to sign in and out. Inline scripts and Astro Components.

### With Inline script tags

The `signIn` and `signOut` methods can be imported dynamically in an inline script.

```html
---
---
<html>
<body>
  <button id="login">Login</button>
  <button id="logout">Logout</button>

  <script>
    const { signIn, signOut } = await import("auth-astro/client")
    document.querySelector("#login").onclick = () => signIn("github")
    document.querySelector("#logout").onclick = () => signOut()
  </script>
</body>
</html>
```
### With auth-astro's Components

Alternatively, you can use the `SignIn` and `SignOut` button components provided by `auth-astro/components` importing them into your Astro [component's script](https://docs.astro.build/en/core-concepts/astro-components/#the-component-script) 

```jsx
---
import { SignIn, SignOut } from 'auth-astro/components'
---
<html>
  <body>
    ...
    <SignIn provider="github" />
    <SignOut />
    ...
  </body>
</html>
```

## Fetching the session

You can fetch the session in one of two ways. The `getSession` method can be used in the component script section to fetch the session.

### Within the component script section

```tsx title="src/pages/index.astro"
---
import { getSession } from 'auth-astro/server';

const session = await getSession(Astro.request)
---
{session ? (
  <p>Welcome {session.user?.name}</p>
) : (
  <p>Not logged in</p>
)}
```
### Within the Auth component

Alternatively, you can use the `Auth` component to fetch the session using a render prop.

```tsx title="src/pages/index.astro"
---
import type { Session } from '@auth/core/types';
import { Auth, SignIn, SignOut } from 'auth-astro/components';
---
<Auth>
  {(session: Session) => 
    {session ? 
      <SignOut>Logout</SignOut>
    :
      <SignIn provider="github">Login</SignIn>
    }

    <p>
      {session ? `Logged in as ${session.user?.name}` : 'Not logged in'}
    </p>
  }
</Auth>
```

# State of Project

We currently are waiting for the [PR](https://github.com/nextauthjs/next-auth/pull/6463) in the official [next-auth](https://github.com/nextauthjs/next-auth/) repository to be merged. Once this has happened, this package will be deprecated. 

# Contribution
Waiting on the PR to be merged means, we can still add new features to the PR, so, if you miss anything feel free to open a PR or issue in this repo, and we will try to get it added to the official package.
