# medusa-auth-cognito

Use AWS Cognito to store customer credentials instead of the Postgres database

[Documentation](https://pevey.com/medusa-auth-cognito)

If you are not familiar with Medusa, you can learn more on [the project web site](https://www.medusajs.com/).

> Medusa is a set of commerce modules and tools that allow you to build rich, reliable, and performant commerce applications without reinventing core commerce logic. The modules can be customized and used to build advanced ecommerce stores, marketplaces, or any product that needs foundational commerce primitives. All modules are open-source and freely available on npm.

## Features

The goal of this plugin is to make the fewest changes needed to the Medusa core to have customer credentials (passwords) stored in Cognito instead of the Medusa database.  It uses Cognito for authentication only.  This plugin does not use Cognito for identity (user data) or authorization (session management).  Those functions continue to be handled by Medusa.

- Allows you to store customer credentials (passwords) in an AWS Cognito user pool instead of in your Medusa database.
- Medusa still handles all session management after a user is authenticated.  
- Can be used by a store with existing customers.  Passwords for existing customers will be moved from the Medusa database to Cognito after a successful login.
- Supports only username/password auth flow.

## Conversion of Customer Passwords

- After the plugin is installed, when customers log in successfully for the first time, their password will be saved in your AWS Cognito user pool, and the password hash in your Postgres table will be removed.
- WARNING: This is a one-way conversion.  Once the existing password hash is deleted, it cannot be recovered by removing this plugin.  This is by design. The point of this plugin is to remove the password hashes from the Medusa database to reduce the potential exposure of a breach of the application database.
- If you ever want to stop using your Cognito user pool and convert back to storing credentials in the Postgres database, you will either need to restore the hashes from a database backup, have customers reset their passwords, or create a custom Medusa plugin that creates and stores the password hashes over time as customers log in.

## Installation

```bash
yarn add medusa-auth-cognito
```

## Configuration

Add the plugin to your plugins array in medusa.config.js:

```bash
const plugins = [
   ...
   {
      resolve: `medusa-auth-cognito`,
         options: {
            region: process.env.COGNITO_REGION,
            accessKeyId: process.env.COGNITO_ACCESS_KEY_ID,
            secretAccessKey: process.env.COGNITO_SECRET_ACCESS_KEY,
            userPoolId: process.env.COGNITO_USER_POOL_ID,
            clientId: process.env.COGNITO_CLIENT_ID
         }
   },
   ...
]
```

- The region will be for example "us-east-1"
- Obtain the access key id and secret access key by creating an IAM user with permissions for the following Cognito operations:
	- AdminCreateUser
	- AdminUpdateUserAttributes
	- AdminInitiateAuth
	- AdminSetUserPassword
	- AdminDeleteUser  
- Obtain the userPoolId from the CLI or AWS console.  IMPORTANT NOTE: The user pool id is not the same as the user pool name.
- Obtain the clientID from creating an "App integration" for your user pool.  IMPORTANT NOTE: When creating your app, be sure to NOT select the option to generate a client secret. The AWS javascript SDK does not support the use of client secrets.  Also, we will only be communicating with Cognito directly from our Medusa server.  We will be using our IAM credentials obtained above.

## AWS User Pool Setup

When you create your user pool, select 'email' as an alias attribute for signing in.  Also, make sure you create an app integration with the ALLOW_ADMIN_USER_PASSWORD_AUTH authentication flow enabled.

## Revoking Customer Authorization

- As mentioned above, this plugin uses Cognito as a credentials provider only.  
- Once a user is logged in to an active Medusa session, that session is managed by Medusa in exactly the same way a session would be managed without this plugin.
- Therefore, to revoke customer authorization, you must expire the session in the Medusa database, the same as you would now.  
- As an example, if you wanted to implement in your storefront the ability for a customer to "Sign out on all other devices," or similar functionality, you would do that by expiring the other active Medusa sessions.
- Changing token expiration times, refresh times, etc., via the AWS user pool settings will have no effect.  
- Revoking tokens via the AWS console or API will have no effect.