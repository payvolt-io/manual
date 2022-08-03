# Payvolt RESTful API Documentation

## Issues and support

if you encountered any issues or need guidance, you can open a ticket in our [discord](https://discord.gg/MvFSsFm96y).

## Preface

Payvolt provides a simple RESTful API that you can embed in your app\platform.  
This document explains how to use the payvolt RESTful API and assumes you are familiar with using and preforming [REST API](https://aws.amazon.com/what-is/restful-api/) calls.  
The features of this API include getting data about user\s (how much they mined for you), connecting and verifying payvolt users to your platform.

**to use this API you will need a payvolt user, sign up at [payvolt](https://payvolt.io).**

## Table Of Contents

0. [What Is User Id.](#user-id)

1. [Url and Endpoints list.](#url-and-endpoints)

2. [Base Response Model Structure.](#base-response-model-structure)

3. [Authentication.](#authentication)

4. [User Data.](#user-data)

5. [User Verification.](#user-verification)

6. [For Miners And more](#for-miners-and-other-stuff)

## User Id

User id is used to uniquely identify a user by other users and payvolt itself.  
user id is permanent and its not sensitive information.  
user id for example: `HYeO8UlAYVRolfoEBJCWG0X29oY2`  
there are two ways of getting **your** user id:  

1. after logging in at the [payvolt](https://payvolt.io) website, click your email in the top right corner.

2. through the [/auth](#auth) API endpoint.

there are 3 ways of getting user id of **another user**.

1. searching their name in the [search page](https://payvolt.io/search).

2. using the [/verification](#user-verification) API endpoint(s), in conjunction of communication with the user.

3. the user sending you their id or profile page (by them click their email and sending you the link), for example <https://payvolt.io/profile?id=HYeO8UlAYVRolfoEBJCWG0X29oY2>.

## URL And Endpoints

* Base URL

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api`  
    method: none  
    auth-required: false  
    returns: none  

* [/ping](#ping)

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/ping`  
    method: get  
    auth-required: false  
    returns: if request was successful, used to check connectivity with payvolt.  

* [/signin](#signin)

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/signin`  
    method: post  
    auth-required: false  
    returns: jwt bearer token to be used in endpoints that require authentication.  

* [/auth](#auth)

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/auth`  
    method: get  
    auth-required: true  
    returns: data about authentication token.  

* /miners

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/miners`  
    method: get  
    auth-required: true  
    returns: list of all users that mine or mined for the requester.  

* /miners/{payvolt-user-id}

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/miners/{payvolt-user-id}`  
    method: get  
    auth-required: true  
    returns: data of specific user that mine or mined for the requester.  

* /recivers

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/recivers`  
    method: get  
    auth-required: true  
    returns: list of all users that the requester is mining or mined for.  

* /recivers/{payvolt-user-id}

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/recivers/{payvolt-user-id}`  
    method: get  
    auth-required: true  
    returns: data of specific user that the requester mine or mined for them.  

* /settings/{payvolt-user-id}

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/settings/{payvolt-user-id}`  
    method: get  
    auth-required: false  
    returns: info about a user (name, description, avatar, wallet address, payment preference).  

* /verification

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/verification`  
    method: post  
    auth-required: true  
    returns: if verification link created successfully, id of verification.

* /verification/{verification-id}

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/verification/{verification-id}`  
    method: delete  
    auth-required: true  
    returns: delete the verification link.

* /verification/{verification-id}

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/verification/{verification-id}`  
    method: get  
    auth-required: true  
    returns: data about the verification, was it verified and by who.

## Base Response Model Structure

The basic structure of a response body will always be the following model  

```json
{  
    "success":Bool,
    "data":Object,
    "errors":Array
}
```

for example  

```json
{  
    "success":true,
    "data": {
        "jwt":"s0m3SeCrEtSTUFFH3re12421421421"
        },
    "errors":[]
}
```

The only case you will not get a response in this model, is if `data` is null. meaning you will get only `success` and `errors`. or if there are connectivity issues / server is down. which in that case will prevent you from reaching the api at all.

### /ping

Your first endpoint to try, this endpoint can be used to check if there is connectivity to payvolt api. it will alway return `success` as `true` and the `data` property will be absent.  

> url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/ping`  
method: get  
auth-required: false  
returns: if request was successful, used to check connectivity with payvolt.  

**response:**

```json
{  
    "success":true,
    "errors":[]
}
```

### Error

the base model structure for an error will alway be the same

```json
{
    "msg": String,
    "param": String,
    "location": String
}
```

* `msg` a description of what went wrong.
* `param` the suspected input in the request that caused the issue.
* `location` the location in the request bad param is present, possible location: header, body, url.  

in a response with an error the `success` property will always be `false`.  
example error of an expired jwt token:

**response body:**

```json
{
    "success": false,
    "errors": [
        {
            "msg": "Firebase ID token has \"kid\" claim which does not correspond to a known public key. Most likely the ID token is expired, so get a fresh token from your client app and try again.",
            "param": "authorization",
            "location": "header"
        }
    ]
}
```

the response will usually be in an appropriate status code for the error (403,404,500 etc).

a response might have multiple errors in the errors list if a few request inputs were bad.

## Authentication

### *high level preface*

Payvolt uses the [bearer token](https://swagger.io/docs/specification/authentication/bearer-authentication/) ([jwt](https://jwt.io)) type authentication. in short, to use an endpoint that requires authentication a bearer token will need to be present in the request header.

```yaml
Authorization: Bearer <token>
```

in order to **get the token** you will need to use the sign-in endpoint by sending the email you registered with, and your password in the request body.  

a token expires after **one hour** of issuing and there is no way to prolong it for security reasons. preform a sign-in before every api usage flow or get the token every hour in an interval/cron.

**Don't** use your sign in credentials in a front end app where users has access to it, you will have a high risk of being compromised.

it's **highly recommended** to use your sign in credentials as [**secrets**](https://www.cyberark.com/what-is/secrets-management/).

### /signin

the sigh-in endpoint is used to get the jwt bearer token.
the http request method for this endpoint is `post`, the request body needs include a json object with an `email` and a `password` property.  

a successful response will return your token in the `jwt` property inside the `data` property.

> url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/signin`  
method: post  
auth-required: false  
returns: jwt bearer token to be used in endpoints that require authentication.  

**request body:**

```json
{
    "email":"youremail@mail.com",
    "password":"yourpassword"
}
```

**response body:**

```json
{
    "success": true,
    "data": {
        "jwt": "fAKeToKeNiJSUzI1NiIsImtpZCI6IjA2M2E3Y2E0M2MzYzc2MDM2NzRlZGE0YmU5NzcyNWI3M2QwZGMwMWYiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL3NlY3VyZXRva2VuLmdvb2dsZS5jb20vcGF5dm9sdC00YWUwOSIsImF1ZCI6InBheXZvbHQtNGFlMDkiLCJhdXRoX3RpbWUiOjE2NTk0ODk1ODIsInVzZXJfaWQiOiJIWWVPOFVsQVlWUm9sZm9FQkpDV0cwWDI5b1kyIiwic3ViIjoiSFllTzhVbEFZVlJvbGZvRUJKQ1dHMFgyOW9ZMiIsImlhdCI6MTY1OTQ4OTU4MiwiZXhwIjoxNjU5NDkzMTgyLCJlbWFpbCI6InRlc3QyQG1haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJmaXJlYmFzZSI6eyJpZGVudGl0aWVzIjp7ImVtYWlsIjpbInRlc3QyQG1haWwuY29tIl19LCJzaWduX2luX3Byb3ZpZGVyIjoicGFzc3dvcmQifX0.0unZ9TrATq3Pk-SMuOfkRaZNnGcvR4isKitMspNSDlWjq2zHrBOClwmEqzZvx-JvdARR3wDLmvQhZEIST3veWZIWQ_NS1gLVRwha_YFQpkzqr-UhTV_Z6YQvB81QY6clPXjpGcPguMCsacIWqctF_2wDl-eKMJDYP6w0CulHbRicvgBkiurDgu9BKehyYUecXDx-tlpw7lGtChwmWRQnlAc0C9_igFajHYg9KsmbhsllybUKXRW0tW23tr9bt1RXcEW0PSQwLoPeX7r7bEQk8r36zQYVoK38v4dk8VJ7nmSHEOaz_z38nADy3Aw1fH-lZGJnA3-gBspwUh5YJ6x3BN"
    },
    "errors": []
}
```

*In order to use the token in endpoints that require authorization, insert it in the* **header** *in the* **Authorization** *key (create it if it doesn't exist). the key value needs to be the token with the string* **bearer** *before it.*  

header example:  

```yaml
Authorization: Bearer fAKeToKeNiJSUzI1NiIsImtp...
```

### /auth

the auth endpoint decrypts the token for its information, the endpoint is useful if you want to check if the token is expired, when it is about to get expired or to get your [user id](#user-id).
> url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/auth`  
method: get  
auth-required: true  
returns: data about authentication token. 

**request header:**

```yaml
 Authorization: Bearer fAKeToKeNiJSUzI1NiIsImtpZCI6IjA2M2E3Y2E0M2MzYzc2MDM2NzRlZGE0YmU5NzcyNWI3M2QwZGMwMWYiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL3NlY3VyZXRva2VuLmdvb2dsZS5jb20vcGF5dm9sdC00YWUwOSIsImF1ZCI6InBheXZvbHQtNGFlMDkiLCJhdXRoX3RpbWUiOjE2NTk0ODk1ODIsInVzZXJfaWQiOiJIWWVPOFVsQVlWUm9sZm9FQkpDV0cwWDI5b1kyIiwic3ViIjoiSFllTzhVbEFZVlJvbGZvRUJKQ1dHMFgyOW9ZMiIsImlhdCI6MTY1OTQ4OTU4MiwiZXhwIjoxNjU5NDkzMTgyLCJlbWFpbCI6InRlc3QyQG1haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJmaXJlYmFzZSI6eyJpZGVudGl0aWVzIjp7ImVtYWlsIjpbInRlc3QyQG1haWwuY29tIl19LCJzaWduX2luX3Byb3ZpZGVyIjoicGFzc3dvcmQifX0.0unZ9TrATq3Pk-SMuOfkRaZNnGcvR4isKitMspNSDlWjq2zHrBOClwmEqzZvx-JvdARR3wDLmvQhZEIST3veWZIWQ_NS1gLVRwha_YFQpkzqr-UhTV_Z6YQvB81QY6clPXjpGcPguMCsacIWqctF_2wDl-eKMJDYP6w0CulHbRicvgBkiurDgu9BKehyYUecXDx-tlpw7lGtChwmWRQnlAc0C9_igFajHYg9KsmbhsllybUKXRW0tW23tr9bt1RXcEW0PSQwLoPeX7r7bEQk8r36zQYVoK38v4dk8VJ7nmSHEOaz_z38nADy3Aw1fH-lZGJnA3-gBspwUh5YJ6x3BN
```

**response body:**

```json
{
    "success": true,
    "data": {
        "iss": "https://securetoken.google.com/payvolt-4ae09",
        "aud": "payvolt-4ae09",
        "auth_time": 1659489582,
        "user_id": "HYeO8UlAYVRolfoEBJCWG0X29oY2",
        "sub": "HYeO8UlAYVRolfoEBJCWG0X29oY2",
        "iat": 1659489582,
        "exp": 1659493182,
        "email": "user@mail.com",
        "email_verified": false,
        "firebase": {
            "identities": {
                "email": [
                    "user@mail.com"
                ]
            },
            "sign_in_provider": "password"
        },
        "uid": "HYeO8UlAYVRolfoEBJCWG0X29oY2"
    },
    "errors": []
}
```

## User Data

Fetching data about users

## User Verification

Verifiying a user and connecting him to your platform

## For Miners And Other stuff
