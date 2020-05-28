---
id: providers
title: Providers
---

export const Image = ({ children, src, alt = '' }) => ( 
  <div
    style={{
      padding: '0.2rem',
			width: '100%',
			display: 'flex',
			justifyContent: 'center'
    }}>
		<img alt={alt} src={src} />
  </div>
 )

## Supported Providers

| Name | API docs | App configuration| 
| --- | --- | --- |
| `Auth0` | https://auth0.com/docs/api/authentication#authorize-application | https://manage.auth0.com/dashboard |
| `Discord` | https://discord.com/developers/docs/topics/oauth2 | https://discord.com/developers/applications |
| `Email` | https://nodemailer.com/smtp/well-known | |
| `Facebook` | https://developers.facebook.com/docs/facebook-login/manually-build-a-login-flow/ | https://developers.facebook.com/apps/ |
| `Github` | https://developer.github.com/apps/building-oauth-apps/authorizing-oauth-apps/ | https://github.com/settings/apps/ |
| `Google OAuth2` | https://developers.google.com/identity/protocols/oauth2 | https://console.developers.google.com/apis/credentials |
| `Mixer` | https://dev.mixer.com/reference/oauth | https://mixer.com/lab/oauth |
| `Slack` | https://api.slack.com | https://api.slack.com/apps |
| `Twitch` | https://dev.twitch.tv/docs/authentication | https://dev.twitch.tv/console/apps |
| `Twitter` | https://developer.twitter.com | https://developer.twitter.com/en/apps |

## OAuth Configuration

:::note
NextAuth supports [OAuth](https://oauth.net/) 1.0, 1.0A and 2.0 providers. 
:::

### Basics

1. Register your application at the developer portal of your provider. There are links above to the developer docs for most supported providers with details on how to register your application.

2. The redirect URI should follow this format:
	```
	[origin]/api/auth/callback/[provider]
	```
	For example, Twitter on `localhost` this would be:
	```
	http://localhost:3000/api/auth/callback/twitter
	```
3. Create a `.env` file at the root of your project and add the client ID and client secret. For Twitter this would be:

	```
	TWITTER_ID=YOUR_TWITTER_CLIENT_ID
	TWITTER_SECRET=YOUR_TWITTER_CLIENT_SECRET
	```

4. Now you can add the provider settings to the NextAuth options object. You can add as many OAuth providers as you like, as you can see `providers` is an array. 

	```js title="/pages/api/auth/[...slug].js"
	...
	providers: [
		Providers.Twitter({
			clientId: process.env.TWITTER_ID,
			clientSecret: process.env.TWITTER_SECRET,
		}),
		Providers.Google({
			clientId: process.env.GOOGLE_ID,
			clientSecret: process.env.GOOGLE_SECRET
		}),
	],
	...
	```
5. Once a provider has been setup, you can sign in at the following URL: `[origin]/api/auth/signin`. This is an unbranded auto-generated page with all the configured providers.   

<Image src="https://user-images.githubusercontent.com/595695/82076867-5915f380-96d6-11ea-8975-2059ce1c81a7.png" alt="Login Screenshot" />


> If you want to create a custom sign in page you can use `[origin]/api/auth/signin/[provider]` which connects directly to the provider.


### Custom OAuth provider

Most OAuth providers only need a **Client ID** and a **Client Secret** to work but some need some additional options. There are gotchas with some providers.

| Provider | Options required | Notes |
| :--- | :--- | :--- |
| `Auth0` | accessTokenUrl, authorizationUrl, profileUrl | Doesn't need clientSecret. |
| `Discord`|  | Doesn't need clientSecret. |
| `GitHub` | | Only allows one callback URL. May not return email address if privacy enabled. |
| `Facebook` | | Doesn't allow testing with localhost callback URLs. May not return email address if the account was created with a mobile number. |
| `Twitter` | | Need to enable the *"Request email address from users"* option in your app's permissions. |

It's also possible to add an OAuth provider that isn't yet supported by NextAuth by creating a custom JSON object. Below you can see the configuration for the Google provider.

```json
{
  id: 'google',
  name: 'Google',
  type: 'oauth',
  version: '2.0',
  scope: 'https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email',
  params: { grant_type: 'authorization_code' },
  accessTokenUrl: 'https://accounts.google.com/o/oauth2/token',
  requestTokenUrl: 'https://accounts.google.com/o/oauth2/auth',
  authorizationUrl: 'https://accounts.google.com/o/oauth2/auth?response_type=code',
  profileUrl: 'https://www.googleapis.com/oauth2/v1/userinfo?alt=json',
  profile: (profile) => {
    return {
      id: profile.id,
      name: profile.name,
      email: profile.email,
      image: profile.picture
    }
  },
  clientId: '',
  clientSecret: ''
}
```
You can replace all the options in this JSON object with the ones from your custom provider and add it to the providers list.

```js title="/pages/api/auth/[...slug].js"
...
providers: [
  Providers.Twitter({
    clientId: process.env.TWITTER_ID,
    clientSecret: process.env.TWITTER_SECRET,
  }),
  {
    id: 'customProvider',
    name: 'CustomProvider',
    type: 'oauth',
    version: '2.0',
    scope: // When configuring OAuth providers you will need to make sure you get permission to request
    ...
  }
]
...
```

:::note
Feel free to open a PR for your custom configuration if you've created one for a provider that others may be interested in so we can support them out of the box 😉
:::

## Email configuration

### Basics

1. Make sure you have SMTP access with one of the well known support providers [here](http://nodemailer.com/smtp/well-known/)
2. There are two ways to configure the SMTP-server connection. You can either use a connection string or a configuration object.

	2.1 **Connection string**

	Create an .env file to the root of your project and the connection string and email address.
	```js title=".env" {2}
	EMAIL_FROM=noreply@example.com
	EMAIL_SERVER=smtp://username:password@smtp.example.com:587
	```
	Now you can add the provider settings, including this `EMAIL_SERVER` connection string, to the NextAuth options object in the Email Provider.

	```js {3} title="/pages/api/auth/[...slug].js"
	providers: [
		Providers.Email({
			server: process.env.EMAIL_SERVER, 
			from: process.env.EMAIL_FROM,
		}),
	],
	```

	2.2 **Configuration object**

	In your `.env` file in the root of your project simply add the configuration object options individually:

	```js title=".env"
	EMAIL_FROM=noreply@example.com
	EMAIL_SERVER_USER=username
	EMAIL_SERVER_PASSWORD=password
	EMAIL_SERVER_HOST=smtp.example.com
	EMAIL_SERVER_PORT=587
	```
	Now you can add the provider settings to the NextAuth options object in the Email Provider.

	```js title="/pages/api/auth/[...slug].js"
  providers: [
    Providers.Email({
      server: {
        host: process.env.EMAIL_SERVER_HOST,
        port: process.env.EMAIL_SERVER_PORT,
      auth: {
        user: process.env.EMAIL_SERVER_USER,
        pass: process.env.EMAIL_SERVER_PASSWORD
      }
    },
    from: process.env.EMAIL_FROM,
    }),
  ],
	```
3. You can sign in at `[origin]/api/auth/signin`.   
   This is an unbranded auto-generated page with all the configured providers. If you want to create a custom sign in page you can use `[origin]/api/auth/signin/email` which connects directly to this email provider.


