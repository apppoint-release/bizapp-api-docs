Infrastructure/Solution level REST Endpoints
==========================
Infrastructure or solution specific REST end points are typically used by external integrations. Modeler allows users to define API's that external parties would use to perform data transfer, displaying generic information that need to be user agnostic. This page provides information on defining such integration API's for external applications either from server to server calls or in some cases from client to server.

## Supported mechanisms
The following authentication mechanisms are supported.
* API key and secret
* HMAC Authentication

## API Key and Secret
API key based authentication allows uncontrolled access to underlying resources. This is usually required to perform server level integration on exchange of information. 

## HMAC Authentication
Another variant authentication supported is HMAC. As the name suggests, it is hash-based message authentication code (HMAC) is a mechanism for calculating a message authentication 
code involving a hash function in combination with a secret key. This can be used to verify the integrity and authenticity of a a message.

It is very important to note that one of the BIG difference with this type of authentication is it signs the entire request, if the content-md5 is included, this basically guarantees the authenticity of the action. If a party in the middle fiddles with the API call either for malicious reasons, or bug in a intermediary proxy that drops some important headers,the signature will not match.

The use HMAC authentication a digest is computed using a composite of the URI, request timestamp and some other headers (dependeing on the implementation) using the supplied secret key. The key identifier along with the digest, which is encoded using Base64 is combined and added to the authorisation header.

## Create API Key and Secret
To create a new API end point key and secret, navigate to System Administration -> API Endpoints.
Click on Create API Endpoint as shown in the image.

![list api endpoint](..\images\List-api-endpoints.png)

New pop up window appears with some fields pre-filled. 
* Provider key defaults to *apikey*.
* Provider secret shows a key. Click on Generate Secret if you want to regenerate the key.
* HMAC authentication requires the HMAC function to use one of the crypotgraphic hash function.
* Select *Use Digest* option.
* By default, API key is active. It can also be turned off after it is created.

![create api endpoint](..\images\create-api-endpoints.png)
* Click on *Save* button.
* The newly generated API key appears in the list view.

## Defining API's in Modeler
API's are defined in the modeler either using *user code* or *code library*. Any API's defined in the modeler is automatically enabled to be accessed from the API end points. 
In addition, any framework/platform defined end points are also accessible using additional headers.

## Defining API's in external assemblies

API's can be defined in an external assembly and exposed as end points. This will be particularly useful in scenarios where you want to expose an API from 3rd party library
as public end points. This can be achieved by using any of the method signatures suggested in the following sections. 

### API Signatures

One of the following method signature(s) is required in the modeler. All methods must be static and return value can be one of the following.

* IActionResult that returns any of the valid MVC Action Results.
* Any primitive value or string - Value is returned as is. Additionally content type can be set before returning the value.
* Non-primitive value such as an instance of an object - It is automatically serialized as json.

**Signature 1**

```csharp
public static IActionResult Echo( ISessionService sessionService, string payload )
{
	// your code goes here.
}
```

**Signature 2**

Additional parameters that includes query string values as NameValueCollection.

```csharp
public static IActionResult Echo( ISessionService sessionService, NameValueCollection headers, string payload )
{
	// your code goes here.
}
```

**Signature 3**

Additional parameters that includes query string values as NameValueCollection, form variables as NameValueCollection and files collection as HttpFileCollection.

```csharp
public static IActionResult Echo( ISessionService sessionService, NameValueCollection headers, NameValueCollection formVariables, HttpFileCollection files )
{
	// your code goes here.
}
```

## Using API's from Client

All API's defined either in the modeler or framework supported methods can be invoked using any of the clients that allow invoking REST api's. Url to access generic REST endpoints is

**http(s)://[host]/apihandler.ashx[?query string]**

### Authentication
Authentication is required to invoke any api's available. It could be either a simple apikey/secret or HMAC authentication with the headers required. 
The following headers are the same for both modes.

| Header Name        | Header Value          |
| ----------------------- | -----------------------------------------------------|
| X-BIZAPP-KEY 		 | apikey            										 |
| X-BIZAPP-SECRET    | KpOAHMNl0qDbNxQhRL1hiiQAZK00lZMp/9oNVrH+w==   			 |
| X-BIZAPP-TYPEID	 | ESystema478 - Type id of the metadata					 |
| X-BIZAPP-METHOD	 | Echo - Name of the method either from code library or user code  |
| X-BIZAPP-AUTHDIGEST| Use digest for authentication							 |
| X-BIZAPP-DIGESTEPOCH | Hash computed 											 |
| X-BIZAPP-CUSTOMTYPE | 1 or true - For platform specific API's.				 |

HMAC digest and epoch generation can be generated as shown below.

```csharp
var ticks = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds( );
var hmacMessage = $"{ticks}|{requestPath}";
var hmacDigest = default( string );
using ( var hmacSHAx = new HMACSHA256( System.Text.Encoding.ASCII.GetBytes( apiSecret ) ) )
{
	hmacDigest = Convert.ToBase64String( hmacSHAx.ComputeHash( System.Text.Encoding.ASCII.GetBytes( hmacMessage ) ) );
}
```
The above code assumes the following
1. API Key created uses HMAC256 digest mechanism. 
2. Http method for the request is GET.

If you are using POST, then entire payload needs to be used in place of requestPath. Using the above code, both AUTHDIGEST AND DIGESTEPOCH can be set.

### POST body or Payload
The payload for the request is normally a json string. This will be passed to the static API defined in the solution identified by the type id specified in the header.

### Errors
All errors use appropriate HTTP error codes. 
* 401 - Unauthorized.
* 500 - Internal server error while processing the request. The error message sent back to the client.
* Others - Method can return appropriate error codes where required. 

### Platform defined REST end points
Platform comes with pre-defined REST API end points that provide some common functionality.

#### Get Validation Code
This API allows custom solutions to implement MFA(Multi Factor Authentication) in any of the flows. For e.g. if a solution requires an approval flow and needs to validate if the
current user can be allowed.

**Url** : http(s)://[host]/apihandler.ashx

**HTTP Method** : POST

**Headers**

| Header Name        | Header Value          |
| ----------------------- | -----------------------------------------------------|
| X-BIZAPP-TYPEID	 | BizAPP.Runtime.Core.Framework.Utils.RestAPIClientMethods, BizAPP.Runtime.Core.Framework, Version=1.1.0.37, Culture=neutral, PublicKeyToken=5cd91901593ba07f					 |
| X-BIZAPP-METHOD	 | ChallengeUserId											 |
| X-BIZAPP-CUSTOMTYPE | true												     |

> *Other header values are removed for brevity.*

**Payload**

The following payload is required to generate a validation code and send to the target user.

```json
{
    "email":"email@yourcompany.com",
    "validateUser":true,
    "authFeature":"ForgotPassword",
    "templates":["Email"]
}
```
Valid values for **templates** are
* Email
* SMS
* PushNotification

The above values can be specified as an array in any possible combinations.

Valid values for **authFeature** are
* LoginWeb
* LoginMobile
* Signup
* ForgotPassword
* Impersonation
* ApplyStep

**Response**
* 200 - Successful
* 500 - Internal error occurred, response includes the error message.

#### Challange User Response
Helps to validate the user response validation code sent (to user by the previous API). 

**Url** : http(s)://[host]/apihandler.ashx

**HTTP Method** : POST

**Headers**

| Header Name        | Header Value          |
| ----------------------- | -----------------------------------------------------|
| X-BIZAPP-TYPEID	 | BizAPP.Runtime.Core.Framework.Utils.RestAPIClientMethods, BizAPP.Runtime.Core.Framework, Version=1.1.0.37, Culture=neutral, PublicKeyToken=5cd91901593ba07f					 |
| X-BIZAPP-METHOD	 | ChallengeResponse											 |
| X-BIZAPP-CUSTOMTYPE | true														 |

> *Other header values are removed for brevity.*

**Payload**

The payload accepts email id and validation response code entered by the user.

```json
{
    "email":"email@yourcompany.com",
    "userPromptCookie":"109789"
}
```

**Response**
* 200 - Successful
* 500 - Internal error occurred, response includes the error message.

#### Change Password
Helps to change password after user validation is completed.

**Url** : http(s)://[host]/apihandler.ashx

**HTTP Method** : POST

**Headers**

| Header Name        | Header Value          |
| ----------------------- | -----------------------------------------------------|
| X-BIZAPP-TYPEID	 | BizAPP.Runtime.Core.Framework.Utils.RestAPIClientMethods, BizAPP.Runtime.Core.Framework, Version=1.1.0.37, Culture=neutral, PublicKeyToken=5cd91901593ba07f					 |
| X-BIZAPP-METHOD	 | ChangePassword											 |
| X-BIZAPP-CUSTOMTYPE | true													 |

> *Other header values are removed for brevity.*

**Payload**

```
{
    "email":"email@yourcompany.com",
    "cookie":"134288",
    "password":"test@123"
}
```

**Response**
* 200 - Successful
* 500 - Internal error occurred, response includes the error message.

### User defined external REST end points
Invoking the end points from external assemblies requires the same headers as Platform defined REST end points. The below is an example of using end points from custom assemblies.

| Header Name        | Header Value          |
| ----------------------- | -----------------------------------------------------|
| X-BIZAPP-TYPEID	 | Assembly qualified type name 							 |
| X-BIZAPP-METHOD	 | Name of the static method matching the signature required |
| X-BIZAPP-CUSTOMTYPE | true												     |

> *Other header values are removed for brevity.*
