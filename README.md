# GOV.UK Prototype + express-openid-connect + GOV.UK One Login

This proof-of-concept integrates the [GOV.UK Prototype Kit](https://prototype-kit.service.gov.uk/)
with [GOV.UK One Login](https://www.sign-in.service.gov.uk/) using Auth0's
[express-openid-connect](https://github.com/auth0/express-openid-connect) `npm` module.

The user experience is hardly Alton Towers: you log in, you get your profile. But see [/app/routes.js](app/routes.js)
to see how little configuration is required to get your Express prototype integrated as a Relying Party.

## Environment variables

The Prototype requires the following environment variables:

    ISSUER_BASE_URL

Likely to be https://oidc.integration.account.gov.uk or similar

    BASE_URL

The base URL of wherever you are hosting this prototype.  It should have been registered as a Relying Party with GOV.UK One Login.

    CLIENT_ID

The Client ID of your Relying Party, which you should have received when you registered with GOV.UK One Login.

    SECRET

A long, random string used to encrypt the session cooke.  You can generate one with `openssl rand -base64 32`

    RSA_PRIVATE_KEY

The private key of the keypair you generated when registering your Relying Party with GOV.UK One Login.  You can generate an RSA keypair as follows:

    openssl genrsa -out private.pem 2048
    openssl rsa -in private.pem -pubout

(Obviously) only supply the public key to the OP.  The private key should remain just that.

### Optional environment variables

    REQUEST_IDENTITY_PROOFING

Set this to the string `"true"` if you want Identity Proofing and Verification.  If you do, the following are also required:

    CREDENTIAL_ISSUER_URL

Likely to be https://identity.integration.account.gov.uk

    SPOT_PUBLIC_KEY

The RSA Public Key of the Credential Issuer (i.e. the above host)

## Running locally

Unless you really know what you're doing, you're going to find it hard to run a working Relying Party on localhost.  But to start the Prototype Kit:

    npm install
    npm run dev

and browse to https://localhost:3000

## Bonus: a quick round of code golf

This is (almost) a complete Relying Party that will spit out your profile in JSON:

```js
const express = require('express')
const app = express()
const { auth } = require('express-openid-connect')
app.use(auth({
  authorizationParams: {
    response_type: 'code',
    scope: 'openid email phone',
  },
  clientAssertionSigningKey: process.env.RSA_PRIVATE_KEY,
  idTokenSigningAlg: 'ES256'
}))
app.get('/', async (req, res) => {
  userinfo = await req.oidc.fetchUserInfo()
  res.json(userinfo)
})
app.listen(8080)
```
