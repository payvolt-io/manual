# Payvolt RESTful API Documentation

## Issues and support

if you encountered any issues or need guidance, you can open a ticket in our [discord](https://discord.gg/MvFSsFm96y).

## Preface

Payvolt provides a simple RESTful API that you can embed in your app\platform.  
This document explains how to use the payvolt RESTful API and assumes you are familiar with using and preforming [REST API](https://aws.amazon.com/what-is/restful-api/) calls.  
The features of this API include getting data about user\s, connecting and verifying payvolt users in your platform.

## Table Of Contents

1. [url and endpoints list.](#url--endpoints)

2. [Base response structure.](#base-response-structure)

3. [Authentication.](#authentication)

4. [User Data.](#user-data)

5. [User Verification.](#user-verification)

## URL & Endpoints

* Base URL

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api`  
    method: none  
    auth-required: false  
    returns: none  

* /ping

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/ping`  
    method: get  
    auth-required: false  
    returns: if request was successful, used to check connectivity with payvolt.  

* /signin

    > url: `https://us-central1-payvolt-4ae09.cloudfunctions.net/api/signin`  
    method: post  
    auth-required: false  
    returns: jwt bearer token to be used in endpoints that require authentication.  

* /auth

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

## Base Response Structure

The basic structure of a response.  

## Authentication

Authentication

## User Data

Fetching data about users

## User Verification

Verifiying a user and connecting him to your platform
