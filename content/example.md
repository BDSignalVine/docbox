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

The token and secret for use with your account is available in the portal.  After logging into https://v3.signalvine.com, click on `Tools` at the top of the screen, and then click `API` from the left hand navigation.  

You can view the secret, by clicking the flashlight icon.  

```scala
package theseus.services

import javax.crypto.spec.SecretKeySpec
import javax.crypto.Mac

object RequestSignatureService {
  private def normalize(clientId: String, method: String, path:String, timeStamp: String, body: String): String = {
    if (clientId.isEmpty() || method.isEmpty() || path.isEmpty() || timeStamp.isEmpty()) throw new SignatureException("Invalid signature fields")

    try {
      val dt = DateTime.unsafeParse(timeStamp)
    } catch {
      case e: Exception => throw new SignatureException("Could not parse timestamp")
    }

    clientId.toLowerCase() ++ "\n" ++ method.toLowerCase() ++ "\n" ++ path.toLowerCase() ++ "\n" ++ body.toLowerCase() ++ "\n" ++ timeStamp.toLowerCase() 
  }

  def sign(clientId: String, secret: Array[Byte], method: String, path: String, timeStamp: String, body: String = ""): String = {
    val normalized = RequestSignatureService.normalize(clientId, method, path, timeStamp, body)
    val sec = new SecretKeySpec(secret, "HmacSHA256")

    var mac = Mac.getInstance("HmacSHA256")
    mac.init(sec)
    
    var data = mac.doFinal(normalized.getBytes("utf-8"))
  
    return new sun.misc.BASE64Encoder().encode(data)
  }
}
```
```c-sharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Security.Cryptography;
using System.Text;
using System.Web.Script.Serialization;

namespace signalVineApi
{
    class Program
    {
        static void Main(string[] args)
        {
            // this is a program that posts to SV
        }     
        public static Dictionary<string,object> APICall(string action,string endPoint, string body ="")
        {
            var token = // your token
            var secret = //your secret ;
            var timestamp = DateTime.UtcNow.ToString("o");
            var wc = new WebClient();
            wc.Headers.Add("Authorization", "SignalVine "+ token + ":" + Convert.ToBase64String(new HMACSHA256(Encoding.ASCII.GetBytes(secret)).ComputeHash(Encoding.ASCII.GetBytes(token + "\n" + action + "\n" + endPoint + "\n" + body + "\n" + timestamp.ToLower()))));
            wc.Headers.Add("SignalVine-Date", timestamp);
            string jsonResult = "";
            if (action.ToLower()=="get")
                jsonResult = wc.DownloadString("https://theseus-api-integrations.signalvine.com" + endPoint);
            if (action.ToLower() == "post")
                jsonResult = wc.UploadString("https://theseus-api-integrations.signalvine.com" + endPoint, body);
            if (action.ToLower() == "patch")
                jsonResult = wc.UploadString("https://theseus-api-integrations.signalvine.com" + endPoint, "PATCH", body);
            return new JavaScriptSerializer().Deserialize<Dictionary<string, object>>(jsonResult);
        }
        public static Dictionary<string,object> profileField(string name, string type,object value)
        {
            return new Dictionary<string, object>() { { "name", name }, { "type", type }, { "value", value } };
        }
    }
}
```

```ruby
# Creates the request signature for the specified data
# @param type      [String] The type of request i.e. GET OR POST
# @param path      [String] The path that this request is for
# @param timestamp [String] The iso8601 timestamp
# @param body      [String] The request's body
# @return          [String] The base64 encoded signature
def request_signature(type, path, timestamp, body)
  body = body.downcase unless body.nil?
  # Strip out any query parameters since they aren't used for
  path = path.gsub(/\?.*$/, '')
  s = "#{@token.downcase}\n#{type.downcase}\n#{path.downcase}\n#{body}\n#{timestamp.downcase}".force_encoding("utf-8")
  Base64.encode64(OpenSSL::HMAC.digest(OpenSSL::Digest.new('sha256'), @key, s)).gsub(/\n/,'')
end
```

```bash
#!/bin/bash

text_reset="$(tput sgr0)"
text_yellow="$(tput setaf 3)"

default_curl_opts="-s -S"
CURL_OPTS="${CURL_OPTS:-$default_curl_opts}"

usage() {
  echo "Usage: $0 [--verbose] [--text] [token] [secret] <method> <path> [body] [timestamp]

Arguments:
  --verbose             Optional; Log debugging output
  --text                Optional; Post data as text instead of json
  token                 Token used for auth
  secret                Token used for auth
  method                HTTP method; e.g. GET, POST
  path                  URL endpoint relative to API root
  body                  Body of the request to post
  timestamp             UTC timestamp used for auth, defaults to the current time

Environment variables:
  THESEUS_API_ROOT      Required; signifies the API host; e.g. https://theseus-api-integrations.signalvine.com
  SV_TOKEN              Can be supplied instead of the 'token' argument
  SV_SECRET             Can be supplied isntead of the 'secret' argument
  CURL_OPTS             Additional arguments passed to curl command; default: '$default_curl_opts'

Example:
  $0 GET /v1/accounts/1b99980f-e5e2-4032-8306-1f468f8623be/programs
"
}

if [ "$1" = "--verbose" ]; then
  verbose=1
  shift
fi

log() {
  if [ -n "$verbose" ]; then
    >&2 echo "${text_yellow}LOG:" "$@" "${text_reset}"
  fi
}

log_no_prefix() {
  if [ -n "$verbose" ]; then
    >&2 echo "${text_yellow}$@${text_reset}"
  fi
}

log_exec() {
  log "$@"
  "$@"
}

if [ "$1" = "--text" ]; then
  content_type="text/plain"
  shift
else
  content_type="application/json"
fi

token=$SV_TOKEN
if [ -z "$token" ]; then
  if [ -z "$1" ]; then
    >&2 usage
    exit 1
  fi
  token=$1
  shift
fi

if [ "$(wc -m <<< "$token")" -ne 33 ]; then
  >&2 echo "ERROR: Missing token"
  >&2 echo "-------------------------------"
  >&2 usage
  exit 1
fi

secret=$SV_SECRET
if [ -z "$SV_SECRET" ]; then
  if [ -z "$1" ]; then
    >&2 usage
    exit 1
  fi
  secret=$1
  shift
fi

if [ "$(wc -m <<< "$secret")" -ne 37 ]; then
  >&2 echo "ERROR: Missing secret"
  >&2 echo "-------------------------------"
  >&2 usage
  exit 1
fi

if [ "$#" -lt 2 ]; then
  >&2 usage
  exit 1
fi

method=$1
log "METHOD: $method"
url=$2
log "URL: $url"
body=$3
log "BODY: $body"
timestamp="${4:-$(date -u +'%Y-%m-%dT%H:%M:%S.000Z')}"
log "TIMESTAMP: $timestamp"

if [ "$content_type" = "application/json" ]; then
  # If our body is JSON, remove any unnecessary whitespace so the generated signature
  # will match that of theseus'
  body="$(jq '.' -c <<< "$body")"
  log "JSON BODY: $body"
fi

path="$(cut -d '?' -f 1 <<< "$url")"
log "PATH: $path"

api_root=$THESEUS_API_ROOT
log "API ROOT: $api_root"

if [ -z "$api_root" ]; then
  >&2 echo "Must define THESEUS_API_ROOT env var"
  exit 1
fi

string_to_sign="$token
$method
$path
$body
$timestamp"
log "STRING TO SIGN BELOW:"
log_no_prefix "$string_to_sign"
log "^^^^^"

sign() {
  printf "$1" |
    tr '[:upper:]' '[:lower:]' |
    openssl dgst -binary -sha256 -hmac "$secret" |
    openssl base64
}

signature="$(sign "$string_to_sign")"
log "SIGNATURE: $signature"

log_exec curl \
  -H "SignalVine-Date: $timestamp" \
  -H "Authorization: SignalVine ${token}:${signature}" \
  -H "Content-Type: $content_type" \
  -X "$method" \
  "${api_root}${url}" \
  -d "$body" \
  $CURL_OPTS
```


```python
# Creates the request signature for the specified data
# @param type      [String] The type of request i.e. GET OR POST
# @param path      [String] The path that this request is for
# @param timestamp [String] The iso8601 timestamp
# @param body      [String] The request's body
# @return          [String] The base64 encoded signature
def request_signature(type, path, timestamp, body)
  body = body.downcase unless body.nil?
  # Strip out any query parameters since they aren't used for
  path = path.gsub(/\?.*$/, '')
  s = "#{@token.downcase}\n#{type.downcase}\n#{path.downcase}\n#{body}\n#{timestamp.downcase}".force_encoding("utf-8")
  Base64.encode64(OpenSSL::HMAC.digest(OpenSSL::Digest.new('sha256'), @key, s)).gsub(/\n/,'')
end
```

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

## API Functions

There are 3 primary entities available via the API: program, participant and message. 
Further, there are different types of messages: sent, received, scheduled and template.

## Programs 

### List Programs

With an account ID, you can request your programs like this

```endpoint
GET /v1/accounts/:accountId/programs
```
#### Example Response 
```json
{
  "items": [
    {
      "id": "f3be52fa-e78a-4080-85e7-1487fb99c583",
      "accountId": "aa32b0bd-2c1b-480a-94d7-cff05fcc53b0",
      "name": "TXT-4-SUCCESS",
      "active": true
    },
    {
      "id": "9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d",
      "accountId": "aa32b0bd-2c1b-480a-94d7-cff05fcc53b0",
      "name": "HS_Life",
      "active": true
    }
  ],
  "links": [
    {
      "rel": "self",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/programs"
    },
    {
      "rel": "first",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/programs"
    },
    {
      "rel": "prev",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/programs"
    },
    {
      "rel": "next",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/programs"
    },
    {
      "rel": "last",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/programs"
    }
  ],
  "total": 2,
  "offset": 0,
  "count": 25,
  "nextOffset": 0,
  "prevOffset": 0,
  "lastOffset": 0,
  "pages": 1
}
```

### Program Participants

To get the list of participants in a program 

```endpoint
GET /v1/programs/:programId/participants
```

#### Participants Response

```json
{
  "items": [
    {
      "id": "b81bce68-af95-4f7a-9dfd-5e85599cd88f",
      "customerId": "",
      "programId": "9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d",
      "phone": "+15552813987"
    },
    {
      "id": "405b609a-9e48-4f4e-a1f7-c3292465a8e0",
      "customerId": "",
      "programId": "9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d",
      "phone": "+15557059460"
    },
    {
      "id": "e76a8e4d-e04e-4059-854f-8a5ad0c18f34",
      "customerId": "",
      "programId": "9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d",
      "phone": "+15550738497"
    }
  ],
  "links": [
    {
      "rel": "self",
      "uri": "/v1/programs/9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d/participants"
    },
    {
      "rel": "first",
      "uri": "/v1/programs/9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d/participants"
    },
    {
      "rel": "prev",
      "uri": "/v1/programs/9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d/participants"
    },
    {
      "rel": "next",
      "uri": "/v1/programs/9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d/participants"
    },
    {
      "rel": "last",
      "uri": "/v1/programs/9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d/participants"
    }
  ],
  "total": 10,
  "offset": 0,
  "count": 25,
  "nextOffset": 0,
  "prevOffset": 0,
  "lastOffset": 0,
  "pages": 1
}
```

### Upsert Participants

By generating a single CSV, you can update or insert participants to a particular program.

```endpoint
POST /v2/programs/:programId/participants
```

T​he CSV should contain a row of headers, named the same as the program variables.

​The `options` object controls how the CSV is processed.

*  `new = add`: any participants in the CSV that don't exist in Signal Vine will be added (i.e. customer_id & phone don't exist in the given Signal Vine program).

*  `new = ignore`: any participants in the CSV that don't exist in Signal Vine will be ignored (default if `new` not present in the `options` object).

*  `existing = ignore`: any participants in the CSV that exist in Signal Vine will be ignored (default if `existing` not present in the `options` object).

*  To update a Signal Vine participant's profile from the CSV, set the `existing` field to an array containing the profile field names you wish to update (i.e. one or more values in the header row of the CSV file).

*  `absent = stop`: any participants that exist in Signal Vine that are not present in the CSV will be stopped.

*  `absent = ignore`: any participants that exist in ​Signal Vine that are not present in the CSV will be ignored (default if `absent` is not present in the `options` object)

#### JSON Body
```json
{
  "options": {
    "new": "add" | "ignore"
    "existing": "ignore" | ["field1", "field2"],
    "absent": "ignore" | "stop"
  },
  "participants": "customer_id,field1,field2\n123,val1,val2\n,val3,val4" // CSV file up to 10MB
}
```


## Participants

### List Participants

With an account ID, you can request the participants in your account with this endpoint

```endpoint
GET /v1/accounts/:accountId/participants
```

#### Example JSON
```json
{
  "items": [
    {
      "id": "0a4439aa-46b8-4586-a943-f00c4066de30",
      "customerId": "",
      "programId": "f3be52fa-e78a-4080-85e7-1487fb99c583",
      "phone": "+15550610306"
    },
    {
      "id": "3ba273af-6986-4831-b017-ab726e12eca6",
      "customerId": "",
      "programId": "f3be52fa-e78a-4080-85e7-1487fb99c583",
      "phone": "+15555596868"
    },
    {
      "id": "b81bce68-af95-4f7a-9dfd-5e85599cd88f",
      "customerId": "",
      "programId": "9c93c9ab-b58c-49d8-a8f7-4b4a26707c3d",
      "phone": "+15552813987"
    }
  ],
  "links": [
    {
      "rel": "self",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/participants"
    },
    {
      "rel": "first",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/participants"
    },
    {
      "rel": "prev",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/participants"
    },
    {
      "rel": "next",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/participants"
    },
    {
      "rel": "last",
      "uri": "/v1/accounts/aa32b0bd-2c1b-480a-94d7-cff05fcc53b0/participants"
    }
  ],
  "total": 20,
  "offset": 0,
  "count": 25,
  "nextOffset": 0,
  "prevOffset": 0,
  "lastOffset": 0,
  "pages": 1
}
```



### Participants Messages

To get a message sent particularly to a participant, use this endpoint

```endpoint
GET /v1/participants/:participantId/messages
```

#### Response JSON 

```json
{
  "items": [
    {
      "id": "84a49ebd-2761-4050-b111-ebee8f045366",
      "participantId": "0a4439aa-46b8-4586-a943-f00c4066de30",
      "programId": "f3be52fa-e78a-4080-85e7-1487fb99c583",
      "customerId": "",
      "sentBy": "habagat@aelogica.com",
      "time": "2016-10-20T04:26:37.391Z",
      "text": "Testing",
      "type": "sent"
    },
    {
      "id": "915fb7a4-963e-4253-8cf4-91a067a43705",
      "participantId": "0a4439aa-46b8-4586-a943-f00c4066de30",
      "programId": "f3be52fa-e78a-4080-85e7-1487fb99c583",
      "customerId": "",
      "sentBy": "habagat@aelogica.com",
      "time": "2016-09-22T01:16:13.790Z",
      "text": "This message will be sent too!",
      "type": "sent"
    },
    {
      "id": "1de6dce3-4ab1-4cad-ad90-fb2649cc568c",
      "participantId": "0a4439aa-46b8-4586-a943-f00c4066de30",
      "programId": "f3be52fa-e78a-4080-85e7-1487fb99c583",
      "customerId": "",
      "sentBy": "habagat@aelogica.com",
      "time": "2016-08-26T06:10:40.798Z",
      "text": "Hey Bill",
      "type": "sent"
    },
    {
      "id": "e0c2ef52-fc8f-4f15-b033-8a99a0f3cd4d",
      "participantId": "0a4439aa-46b8-4586-a943-f00c4066de30",
      "programId": "f3be52fa-e78a-4080-85e7-1487fb99c583",
      "customerId": "",
      "sentBy": "habagat@aelogica.com",
      "time": "2016-08-25T00:59:53.289Z",
      "text": "Hey Guys",
      "type": "sent"
    },
    {
      "id": "490ff7f9-579c-4a24-a2b9-6d1b504443ac",
      "participantId": "0a4439aa-46b8-4586-a943-f00c4066de30",
      "programId": "f3be52fa-e78a-4080-85e7-1487fb99c583",
      "customerId": "",
      "sentBy": "habagat@aelogica.com",
      "time": "2016-08-02T06:37:34.146Z",
      "text": "Hello there!",
      "type": "sent"
    },
    {
      "id": "d921e479-1874-4af8-a576-10eb932d48de",
      "participantId": "0a4439aa-46b8-4586-a943-f00c4066de30",
      "programId": "f3be52fa-e78a-4080-85e7-1487fb99c583",
      "customerId": "",
      "sentBy": "habagat@aelogica.com",
      "time": "2016-08-02T04:35:08.843Z",
      "text": "Study hard!",
      "type": "sent"
    }
  ],
  "links": [
    {
      "rel": "self",
      "uri": "/v1/participants/0a4439aa-46b8-4586-a943-f00c4066de30/messages"
    },
    {
      "rel": "first",
      "uri": "/v1/participants/0a4439aa-46b8-4586-a943-f00c4066de30/messages"
    },
    {
      "rel": "prev",
      "uri": "/v1/participants/0a4439aa-46b8-4586-a943-f00c4066de30/messages"
    },
    {
      "rel": "next",
      "uri": "/v1/participants/0a4439aa-46b8-4586-a943-f00c4066de30/messages"
    },
    {
      "rel": "last",
      "uri": "/v1/participants/0a4439aa-46b8-4586-a943-f00c4066de30/messages"
    }
  ],
  "total": 16,
  "offset": 0,
  "count": 25,
  "nextOffset": 0,
  "prevOffset": 0,
  "lastOffset": 0,
  "pages": 1
}
```


#### GET Request for Participants 

```endpoint
GET /v1/participants/:participantId/messages
```

You can retrieve the different types of messages by specifying the `message_type` in the query string: /v1/participants/:participantId/messages?message_type=received

Note : Different representations are available for each resource, you can append the type parameter to any route to select either the simple or full representation - i.e. /v1/programs/:programId/participants?type=full

## Messages

### List Messages

To list messages for your account, use the following endpoints

```endpoint
GET /v1/accounts/:accountId/messages
```

#### Response JSON

```json

```

## Notes

When implementing the API, here are a few points to keep in mind

*  The timestamp parameter must include milliseconds to 3 digits - i.e. append .000 to your timestamp, if your language doesn't produce an ISO8601 string to that precision

*  The encrypted string is case sensitive -​ ​the `T` and `Z` in the timestamp should be lowercase when building the string to be encrypted for the signature, but upper case in the `SignalVine-Date` header.

*  Use the bash script's verbose option to verify that the signature you are generating is the correct one, but, remember that different timestamps will create different signatures. You can specify the timestamp for use in the bash script.  So, output the elements of the signature string along with the signature from your code, and call the bash script with the verbose switch & the time stamp to compare the string before it's encrypted.

```
THESEUS_API_ROOT​=https://theseus-api-integrations.signalvine.com/ \
SV_TOKEN=​​xxxx \
SV_SECRET=​xxxx​ \
./sign_request --verbose \
GET /v1/​accounts/​​xxxx
 \
2016-10-04T12:00:00.000Z​
```

Where the last parameter is the same timestamp that your implementation is using.
