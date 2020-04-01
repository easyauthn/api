# api-doc

EasyAuthn provide easy way to integrate and use [WebAuthn](https://en.wikipedia.org/wiki/WebAuthn) into your application on any device. Here is described EasyAuthn API that you can use for that integration. We'll look at how to integrate a test client with a name `DemoClient` with EasyAuthn.

## Table of contents

1. [How it works](#how-it-works)
  * [Registration](#registration)
  * [Authentication](#authentication)

2. [Commands](#commands)
 * [requestUserRegistration](#requestuserregistration)
 * [requestInstanceIdUrl](#requestinstanceidurl)
 * [getUserCredentials](#getusercredentials)
 * [deleteUserCredential](#deleteusercredential)
 * [doesUserHaveCredentials](#doesuserhavecredentials)
 * [isInstanceIdAuthn](#isinstanceidauthn)

3. [Errors](#errors)

4. [Socket listeners](#socket-listeners)
 * [Registration listener front](#registration-listener-front)
 * [Sign in listener front](#sign-in-listener-front)

## How it works

First of all, we will describe the concept on which EasyAuthn works. There are two main processes that are named respectively [Registration](#registration) and [Authentication](#authentication).

### Registration

The `Registration` process is intended to describe how `DemoClient` can create user and WebAuthn credentials for that user in the EasyAuthn platform.

![EasyAuthn-Registration](https://raw.githubusercontent.com/easyauthn/api/master/EasyAuthn-Registration-1.0.0.png)

To implement `requestUserRegistration` from the diagram, use [requestUserRegistration](#requestuserregistration). To implement `registration successful` in web browser, use  [Registration listener front](#registration-listener-front).

### Authentication

The `Authentication` process is used to verify credentials created in [Registration](#registration).

## Commands

EasyAuthn API endpoint is `https://easyauthn.com/api`. It accepts POST requests with parametar `p` wich must hold JSON.

Unless otherwise noted, all the commands params are required.

Unless otherwise noted, all the commands have following in params:
```
 ssk: text - Service Secret Key (SSK) is used to identify the client in EasyAuthn. It must be kept securely by the client server side.
```

Unless otherwise noted, all the commands have following out params in JSON:
```
 status: text - Command status. Possible values are:
 ok - the command was executed successfully
 error - server error
 client_error - the client send a command that cannot be executed due to the data state. In other words, the client should not send such a command at this time. 
```

### requestUserRegistration

Registers new user credential in EasyAuthn.

In params:
```
 userId: text - Unique identifier used by the client server to identify the user in front of another
```

Out params:
```
 userId: text - the same as in param
 registrationUrl: text - url to be used for registration
 qrRegistrationUrl: text - base64 string with QR code of url for registration
 statusRoom: text - emit event with this value in Registration listener front
```

### requestInstanceIdUrl

In params:
```
 userId: text - user id from requestUserRegistration
```

Out params:
```
 url: text - url for authentication
 urlQr: text - base64 string iwith QR code of url for authentication
 instanceId: text - id to identify verification instance
 statusRoom: text - emit event with this value in Sign in listener front
```

The following error codes means the user dont have EasyAuthn credentials:
```
{"status":"client_error","code":"CE22","msg":"Not valid 'ssk' and 'user_id'!"}
{"status":"client_error","code":"CE24","msg":"User without registration!"}
```

### getUserCredentials

In params:
```
 userId: text - user id from requestUserRegistration
```

Out params:
```
 credentials: [{
  id: int - credential id
  created_at: text - date in format 'DD Mon YYYY HH24:MI:SS TZ'
 }]
```

### deleteUserCredential

In params:
```
 userId: text - user id from requestUserRegistration
```

Out params:
```
 userId: text - user id from requestUserRegistration
 credentialId: int - credential id to be deleted
 keepAtLeastOne: bool - optional with default false. true means that there will always be at least one credential for the user.
```

Error code when try to delete last user credential and set `keepAtLeastOne: true`:
```
{"status":"client_error","code":"CE31","msg":"Keep at least on credential!"}
```

### doesUserHaveCredentials

In params:
```
 userId: text - user id from requestUserRegistration
```

Out params:
```
 credentials: bool - the user has or not credentials 
```

### isInstanceIdAuthn

In params:
```
 userId: text - user id from requestUserRegistration
 instanceId: text - id from requestInstanceIdUrl
```

Out params:
```
 authn: bool - instanceId was authenticate or not
```

## Errors

The commands can return following errors:
`error` type:
```
{ status: "error", code: "E00", msg: "Error!" }
```

`client_error` type:
```
{"status":"client_error","code":"CE01","msg":"Missing param 'p'!"}
{"status":"client_error","code":"CE02","msg":"Not valid JSON!"}
{"status":"client_error","code":"CE03","msg":"Missing 'cmd'!"}
{"status":"client_error","code":"CE04","msg":"Missing command param '<param name>'!"}
{"status":"client_error","code":"CE05","msg":"Not valid command!"}
{"status":"client_error","code":"CE06","msg":"Not valid 'ssk'!"}
{"status":"client_error","code":"CE07","msg":"Not valid 'key'!"}
{"status":"client_error","code":"CE08","msg":"Not valid 'msg' base64 encode!"} - Log command
{"status":"client_error","code":"CE09","msg":"Wrong 'type'!"}
{"status":"client_error","code":"CE10","msg":"Missing 'clientDataJSON'!"}
{"status":"client_error","code":"CE11","msg":"Not valid 'response.clientDataJSON'!"}
{"status":"client_error","code":"CE12","msg":"Not valid registration!"}
{"status":"client_error","code":"CE13","msg":"Not valid 'response.clientDataJSON.challenge'!"}
{"status":"client_error","code":"CE14","msg":"Not valid 'response.clientDataJSON.origin'!"}
{"status":"client_error","code":"CE15","msg":"Not valid 'origin'!"}
{"status":"client_error","code":"CE16","msg":"Not valid 'attestationObject'!"}
{"status":"client_error","code":"CE17","msg":"Not supported device!"}
{"status":"client_error","code":"CE18","msg":"Not supported device!"}
{"status":"client_error","code":"CE19","msg":"Not supported device!"}
{"status":"client_error","code":"CE20","msg":"Not unique registration!"}
{"status":"client_error","code":"CE21","msg":"Not unique registration!"}
{"status":"client_error","code":"CE22","msg":"Not valid 'ssk' and 'user_id'!"}
{"status":"client_error","code":"CE23","msg":"Not valid 'key'!"}
{"status":"client_error","code":"CE24","msg":"User without registration!"}
{"status":"client_error","code":"CE25","msg":"Wrong 'type'!"}
{"status":"client_error","code":"CE26","msg":"Authentication failed!"}
{"status":"client_error","code":"CE27","msg":"Authentication failed!"}
{"status":"client_error","code":"CE28","msg":"Authentication failed!"}
{"status":"client_error","code":"CE29","msg":"Wrong credentials!"}
{"status":"client_error","code":"CE30","msg":"Not valid params!"}
{"status":"client_error","code":"CE31","msg":"Keep at least on credential!"}
```

## Socket listeners

EasyAuthn gives the opportunity to clients (e.g `DemoClient`) users to have socket listeners on web browser. Based on that events can implement behaviour.

Socket endpoing is `https://easyauthn.com` with path `/ws`.

### Registration listener front

To activate the listener must emit event `room` with value of `statusRoom` returned from [requestUserRegistration](#requestuserregistration) command.

To receive event must listen for `registration-status` event. It will receive value `ok` when registration was successful.

### Sign in listener front

To activate the listener must emit event `instanceRoom` with value of `statusRoom` returned from [requestInstanceIdUrl](#requestinstanceidurl) command.

To receive event must listen for `instance-status` event. It will receive value `ok` when registration was successful.
