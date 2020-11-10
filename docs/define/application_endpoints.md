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




