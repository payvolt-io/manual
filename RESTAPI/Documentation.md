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

* [/miners](#miners)

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

* [/settings/{payvolt-user-id}](#settingspayvolt-user-id)

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
    "success":Boolean,
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

> *In order to use the token in endpoints that require authorization, insert it in the* **header** *in the* **Authorization** *key (create it if it doesn't exist). the key value needs to be the token with the string* **bearer** *before it.*  
authenticated request header example:  

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

Endpoints for getting information about users, and their earnings for you.

### /settings/{payvolt-user-id}

This Endpoint is used to get information about a user.
which includes: name, description, profile pic, ethereum wallet, btc wallet, preferred payment.  

> url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/settings/{payvolt-user-id}`  
method: get  
auth-required: false  
returns: info about a user (name, description, avatar, wallet address, payment preference).  

*the endpoint doesn't require authentication*

**response body:**

```json
{
    "success": true,
    "data": {
        "name": "JohnDoe",
        "payment": "eth",
        "eth": "0xE75e39FEecf6e9E6AA293cA037A7Ae6a17940A40",
        "description": "ssssafasf",
        "avatar": "https://firebasestorage.googleapis.com/v0/b/payvolt-4ae09.appspot.com/o/profiles%2F3d8a1410-d492-4388-8280-defcbc9c857a?alt=media&token=f607da9b-d778-41aa-99cb-d0914f47b392",
        "btc": "1HyyZxZAQjZenHf9TGsdUY9cxvMpbGjGBz"
    },
    "errors": []
}
```

### /miners

Get a list of all the users that are mining or mined for you, requires authentication.

the response is a list of all the users that are mining for you inside the `data` property, where each key is the [user id](#user-id).
> url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/miners`  
method: get  
auth-required: true  
returns: list of all users that mine or mined for the requester. 

**request header:**

```yaml
 Authorization: Bearer fAKeToKeNiJSUzI1N...
```

**response body:**

```json
{
    "success": true,
    "data": {
        "J3Lc4Sxe26UkEBIzVWpJ4mdgYkg2": {
            "total_validated_shares": 106,
            "total_expected_shares": 45,
            "current_hashrate": 124124,
            "last_seen": 1659583506,
            "workers": [
                "d1c2ee95cf7b45d3c9a4e628b4a70427",
                "BhJ76495cf7b45d3c9a4e62tYyc46Nrg",
                "e1c25f7cbd3c9e9d4e625a48b4yT3n56"
            ],
            "user_info": {
                "avatar": "https://firebasestorage.googleapis.com/v0/b/payvolt-4ae09.appspot.com/o/profiles%2F3d8a1410-d492-4388-8280-defcbc9c857a?alt=media&token=f607da9b-d778-41aa-99cb-d0914f47b392",
                "description": "ssssafasf",
                "payment": "eth",
                "name": "volter1",
                "eth": "0xE75e39FEecf6e9E6AA293cA037A7Ae6a17940A40",
                "btc": "test3"
            }
        },
        "B5uK4TUebnUkzVEBWpJI4qerJcq3": {
            ...
            
        },
        "nT56gQwV88BUkEIJ4qezVWpr4fhUi": {
            ...
            
        },
        . 
        .
        .
    },
    "errors": []
}
```

`J3Lc4Sxe26UkEBIzVWpJ4mdgYkg2` is the user id.

`total_validated_shares` is how much shares were mined for you, while `total_expected_shares` are projected to be added to the validated based on the mining progress.

`workers` is a list of id's of devices preforming mining for you by that user, as he can have serval machines.

`last_seen` is a [unix time stamp](#https://www.unixtimestamp.com/) of when mining last occurred.

`current_hashrate` is how many hashes per second the users mines.

`user_info` is the user information same one as in the [settings/{uid}](#settingspayvolt-user-id) endpoint.

### /miners/{payvolt-user-id}

endpoint to get data about the mining of a specific users.

> url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/miners/{payvolt-user-id}`  
method: get  
auth-required: true  
returns: mining info of the requested miner id.  

for example getting `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/miners/J3Lc4Sxe26UkEBIzVWpJ4mdgYkg2`  
will return data of the miner with the [user id](#user-id) of `J3Lc4Sxe26UkEBIzVWpJ4mdgYkg2`.

**request header:**

```yaml
 Authorization: Bearer fAKeToKeNiJSUzI1N...
```

**response body:**

```json
{
    "success": true,
    "data": {
        "J3Lc4Sxe26UkEBIzVWpJ4mdgYkg2": {
            "total_validated_shares": 106,
            "total_expected_shares": 45,
            "current_hashrate": 124124,
            "last_seen": 1659583506,
            "workers": [
                "d1c2ee95cf7b45d3c9a4e628b4a70427",
                "BhJ76495cf7b45d3c9a4e62tYyc46Nrg",
                "e1c25f7cbd3c9e9d4e625a48b4yT3n56"
            ],
            "user_info": {
                "avatar": "https://firebasestorage.googleapis.com/v0/b/payvolt-4ae09.appspot.com/o/profiles%2F3d8a1410-d492-4388-8280-defcbc9c857a?alt=media&token=f607da9b-d778-41aa-99cb-d0914f47b392",
                "description": "ssssafasf",
                "payment": "eth",
                "name": "volter1",
                "eth": "0xE75e39FEecf6e9E6AA293cA037A7Ae6a17940A40",
                "btc": "test3"
            }
        }
    },
    "errors": []
}
```

same data as in [/miners](#miners) but you get only the data for requested miner id  

## User Verification

### high level preface

user verfication is a way for you to validate people their claim on a payvolt user, and establish a relation \ connection between their payvolt user and your platform.  

using the API you will create a 'verification object', this verification object can be verified by a payvolt user through the payvolt website. you can configure the verification time window, and if the verification can be verified only by a specific user and more.

the verification object is temporary and will be deleted between 1-23 hours after its creation.

how you use the verification object is up to you and your platform constraints. it can be a one time verification or before any transaction that occurs on your platform etc.

### /verification

with this endpoint you will create the verification object.

use the post method to create it.

> url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/verification`  
method: post  
auth-required: true  
returns: if verification link created successfully, id of verification.

this enpoint requires authentication.

**request header:**

```yaml
 Authorization: Bearer fAKeToKeNiJSUzI1N...
```

you can leave the request body empty, but there are optional inputs for this endpoint.

**request body:**

```json
// each one of the inputs is optional, you can have 2 or 1 or no inputs.
{
    "to": "userIdString",
    "expiration": "5m"/"30m"/"60m",
    "message": "custom string message"
}
                
```

`to` *(optional)* - a user id of someone we want to validate, if 'to' is specified only that user can verifiy the object.  

`expiration` *(optional)* - until when from the creation of the object the user can verify, the options are `5m` , `30m` or `60m`, if not specified the default value is `30m`. a user can't verify after expiration.

`message` *(optional)* - a custom message you display to the user, if not specified a default message will be displayed.

**response body:**

```json
{
    "success": true,
    "data": {
        "db_stats": {
            "_writeTime": {
                "_seconds": 1659634281,
                "_nanoseconds": 288066000
            }
        },
        "id": "cef3c9f06be754fa98fb5b86aa8c7d0b"
    },
    "errors": []
}
```

`db_stats` is just the time took for creation

`id` is the id of the verification object, it will be used by the user to verify and by you to view the verification status.

### user verifing a verification object with payvolt website

a user can verify your verification request by going to:

`https://payvolt.io/verification?id={verification-object-id}`

from the previous example it would be:

`https://payvolt.io/verification?id=cef3c9f06be754fa98fb5b86aa8c7d0b`

there he will be displayed a prompt to verify this link.

in the verification process, you should concat the `id` to the url `https://payvolt.io/verification?id=` and send it to the user in your app.

### /verification/{verification-id}

this is used to get data about the verification object, and check if it was verified.

> url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/verification/{verification-id}`  
method: get  
auth-required: true  
returns: get data about the verification.

this enpoint requires authentication.

**request header:**

```yaml
 Authorization: Bearer fAKeToKeNiJSUzI1N...
```

**response body:**

```json
{
    "success": true,
    "data": {
        "from": "J3Lc4Sxe26UkEBIzVWpJ4mdgYkg2",
        "verified": true,
        "expiration": {
            "_seconds": 1659636080,
            "_nanoseconds": 847000000
        },
        "to": "",
        "verifiedBy": "B5uK4TUebnUkzVEBWpJI4qerJcq3",
        "message": "Hello , volter1 is asking you to verifiy you payvolt identity (expires in 30 minutes)"
    },
    "errors": []
}
```

`from` - id of who issues the verification (you).

`verified` - was it verified by a user?

`expiration` - `_seconds` \ `_nanoseconds` unix time stamp of when does the option to verify expire.

`to` - only this id can verifiy, if empty anyone can verify.

`verifiedBy` - the id of the person who verified

`message` - the message that is displayed for the user at the verification page

## For Miners And Other stuff

TBD