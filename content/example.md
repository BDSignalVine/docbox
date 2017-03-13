## Using the API

Using our API is as easy as following the next 4 steps.

### Create a Signature

Create a string from components of the HTTP request, and encrypt it using your application secret. The token acts as the shared key, which is then used by the api to decrypt and verify the contents of the signature.

Lowercase & concatenate the following fields together, using \n as a separator.
*  token
*  HTTP verb
*  request path (no server or querystring data)
*  request body
*  timestamp (ISO80601)

Use your secret and token to encrypt the string using SHA2 - 256bit encryption.


### Custom headers

The API looks for 2 particular headers on all inbound requests: 
*  The timestamp you encoded into the signature
*  The token and signature themselves. The signature should be sent as a BASE64 string.



```endpoint
POST https://api.signalvine.com/Foo/Bar?waz=xax
```

#### POST request

```curl
Host: https://app.example.com
Content-Type: application/json
Content-Length: 10
SignalVine-Date: 2014-03-11T05:03:08.619Z
Authorization: SignalVine 123456:7LWA3dHvkOC87h5uRVBofIehmeGRxZJ0DFgWra2E6rs
{woo: war}
          
```
#### String to Sign

```curl
123456
post
/foo/bar
{woo: war}
2014-03-11t05:03:08.619z
```

#### GET Request 

```endpoint
GET https://api.signalvine.com/Foo/Bar?waz=xax
```
#### GET Request 

```curl
Host: https://app.example.com
Content-Type: application/json
Content-Length: 0
SignalVine-Date: 2014-03-11T05:03:08.619Z
Authorization: SignalVine 123456:7LWA3dHvkOC87h5uRVBofIehmeGRxZJ0DFgWra2E6rs
```

#### String to Sign

```curl
123456
get
/foo/bar

2014-03-11t05:03:08.619z
```

### API Functions

There are 3 primary entities available via the API: program, participant and message. 

Further, there are different types of messages: sent, received, scheduled and template.

#### GET Request for program

```endpoint
GET /v1/accounts/:accountId/programs
```

#### GET Request for participants

```endpoint
GET /v1/accounts/:accountId/participants
```


#### GET Request for messages

```endpoint
GET /v1/accounts/:accountId/messages
```

### Get specific entity

To query for entities for a given program use the program endpoints

#### GET Request for Participants 

```endpoint
GET /v1/programs/:programId/participants
```

#### GET Request for Messages 

```endpoint
GET /v1/programs/:programId/messages
```

### Get specific message

Find messages for a given participant

#### GET Request for Participants 

```endpoint
GET /v1/participants/:participantId/messages
```

You can retrieve the different types of messages by specifying the `message_type` in the query string: /v1/participants/:participantId/messages?message_type=received

Note : Different representations are available for each resource, you can append the type parameter to any route to select either the simple or full representation - i.e. /v1/programs/:programId/participants?type=full
