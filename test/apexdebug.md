# JWT Authentication

APEX Supports JWT Authentication as a secure means to secure the API transaction between a Consumer (API Requester) and Publisher (owner of API endpoint).

## Terminology

Publisher - This is the owner of the API endpoint hosted on APEX, which may be an Agency.

Developer/Consumer - This is the developer of the application which will use (or consume) the API belonging to the Publisher.

Key set - a pair of public and private key generated by the Developer.

## Prerequisites-API Endpoint URL

The Developer will have to get the API Endpoint URL from the Publisher. This will be the API which the API request is to be made to. (eg. <https://public-stg.api.gov.sg/agency/api>)

## Prerequisites-API Key(s)

The Developer will have to create the respective Application(s) to subscribe to the API(s) provided by the Publisher. In the Authentication section of the Application, the API Key can be generated (eg. xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx).

Please note that 2 API Keys will be required for a bridging API. The Publisher should inform you if your API is a bridging API.

> ![Image](../dev/image/api-key-portal-1.png) ![Image](../dev/image/api-key-portal-2.png) _For API Developer Portal Users_

> ![Image](../dev/image/api-key-manager-1.png) _For API Manager Users_

## Prerequisites-JWKS Endpoint

The Developer will have to generate a JSON Web Key (JWK) set for signing their authorization header, of an Elliptical Curve P-256 key set (ES256).

With the **public key** generated, the developer will have to either,

- Publish a JWKS (JSON Web Key Set) endpoint ([RFC7517](https://www.rfc-editor.org/rfc/rfc7517#appendix-A.1)) in the Endian [format](#example-of-jwks) typically in the URL of https://your-domain/.well-known/jwks.json, or
- In the APEX API Portal or API Manager, provide a string text of JWKS in the same [format](#example-of-jwks) when creating the APEX Application.

Do note that your key ID (kid) is used to identify your signing key in case more than 1 key exists in the JWKS (such as for purposes of key rotation).

Usually the key/value pairs of "**_use_**", "**_kid_**" will have to be appended to your key generated by a library.

A commerical service such as [auth0](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-key-sets#:~:text=The%20JSON%20Web%20Key%20Set,signing%20JWTs%3A%20RS256%20and%20HS256.) may can help with the hosting of the JWKS keys.

## Example of JWKS

This is an example of a public JWKS with a JWK consisting of Endian coordinates of P-256 EC curve. Do note that the "d" coordinate is omitted as it will define the private key.

```
{
    "keys": [
        {
            "kty": "EC",
            "crv": "P-256",
            "x": "usZhq9AL4aC-hkzGCBK3RuJjmxKE6zqEdFyp-tQ8kh4",
            "y": "wHI1r6rQCHQQSAdNxaJDA0Tw5Fq3B-icq-mbMVlLZA4",
            "use": "sig",
            "kid": "apex-example",
            "alg": "ES256"
        }
    ]
}
```

<!-- TODO: Add image -->

## API Payload Hash

If the API method is POST, PUT or PATCH, the API payload binary will have to be hashed using SHA-256. The Payload should be standardized [as below](#apex-standardized-json-payload) (for JSON payload).

<!-- TODO: Optionally Include Sample Code -->

## <small>**APEX Standardized JSON Payload**

For purposes of consistency of SHA256 data hash across different OS to ease troubleshooting, API payload containing JSON object and data shall be serialized into a single string, and hashed, before sending the serialized string as the payload for the API request.

The serialized string shall not have any white spaces, tabs, carriage returns ('\r' or 0x0D) or linefeed ('\n' or 0x0A) before or after any of the structural characters of JSON (ie. left square bracket '[', left curly bracket '{', right square bracket ']', right curly bracket '}', colon ':', comma ',').

One might describe this JSON string as "without formatting" or compressed.
This is as different operating systems and applications may parse different formatting characters differently (eg. Windows Servers might treat a new line as CRLF ('\r\n' or 0x0D 0x0A) while UNIX-based systems might treat a new line as LF ('\n' or 0x0A) ).</small>

```
An example of an APEX standardized JSON string is:

{"Image":{"Width":800,"Height":600,"Title":"View from 15th Floor","Thumbnail":{"Url":"http://www.example.com/image/481989943","Height": 125,"Width":100},"Animated":false,"IDs":[116,943,234,38793]}}
```

## <small>**APEX Standardized SOAP Payload**

For purposes of consistency of SHA256 data hash across different OS to ease troubleshooting, SOAP payload shall be serialized into a single string without any Carriage Return and Line Feed. These should be removed: CRLF ('\r\n' or 0x0D 0x0A) and LF ('\n' or 0x0A) and it is recommended to use UTF-8.

The resultant text is hashed, before sending the serialized string as the payload for the API request. </small>

```
An example of an APEX standardized SOAP string is:

<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns="https://www.w3schools.com/xml/"><soapenv:Header/><soapenv:Body><ns:CelsiusToFahrenheit><!--Optional:--><ns:Celsius>23</ns:Celsius></ns:CelsiusToFahrenheit></soapenv:Body></soapenv:Envelope>
```

## Generating JWT

The JWT can be generated using common libraries available based on RFC7519 with the following claims below and signed using either RS256 or ES256 with the Developer's private key.

| S/No | JWT Claim | Comments                                                                                                                                                                                                                                                 |
| ---- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | alg       | This should be '**_ES256_**' only.                                                                                                                                                                                                                       |
| 2    | typ       | Only '**_JWT_**' is accepted.                                                                                                                                                                                                                            |
| 3    | kid       | This is the key ID of the public key used to validate signature. APEX recommends monthly key rotation of the signing key. (eg. **_apex-example_**)                                                                                                       |
| 4    | iat       | This is the Unix epoch time in seconds issued at the of API call. The time zone is in UTC. This may be generated by the signing library.                                                                                                                 |
| 5    | exp       | This is the Unix epoch time in seconds of the expiry time of this JWT. The time zone is in UTC. The time cannot be more than 180 seconds from the **_iat_**.                                                                                             |
| 6    | jti       | This is a random text string (nonce) of at least 40 characters. A random UUIDv4 generator is recommended to be used to generate this.                                                                                                                    |
| 7    | iss       | This is your [API Key(s)](#prerequisites-api-keys) obtained above (eg. xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx and should be delimited by comma '**_,_**' if there are 2 keys (eg. xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx,yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyy). |
| 8    | aud       | This should match the [API endpoint URL](#prerequisites-api-endpoint-url), eg. ***https://public-stg.api.gov.sg/agency/api***. Do note that query parameters are NOT to be included here.                                                                |
| 9    | sub       | This is the method of the API (eg. **_POST_**)                                                                                                                                                                                                           |
| 10   | data      | This is the SHA-256 [API Payload Hash](#api-payload-hash) of the payload of API.(eg. SHA-256 hash of **_{"payload":"data"}_** is **_cc575c4ed557481e31d9a2a0580bc464e84b3a79c5fc94e4fd94ba33b3e54dbc_**                                                  |

Please see [here](docs/sample-codes/jwt-auth.md) for sample codes to generate the JWT.

## Authorization Header

The developer will have to attach the JWT to the **x-apex-jwt** header. The JWT can be tested with the [Hello World! API](docs/hello-world/jwt-auth.md) beforehand.

```
POST /agency/api
Host: public-stg.api.gov.sg
x-apex-jwt: eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImFwZXgtZXhhbXBsZSJ9.eyJkYXRhIjoiY2M1NzVjNGVkNTU3NDgxZTMxZDlhMmEwNTgwYmM0NjRlODRiM2E3OWM1ZmM5NGU0ZmQ5NGJhMzNiM2U1NGRiYyIsImlhdCI6MTY2NzAyMDM2MSwiZXhwIjoxNjY3MDIwNTQxLCJhdWQiOiJodHRwczovL3B1YmxpYy1zdGcuYXBpLmdvdi5zZy9hZ2VuY3kvYXBpIiwiaXNzIjoieHh4eHh4eHgteHh4eC14eHh4LXh4eHgteHh4eHh4eHh4eHgseXl5eXl5eXkteXl5eS15eXl5LXl5eXkteXl5eXl5eXl5eXkiLCJzdWIiOiJQT1NUIiwianRpIjoiZWZhNjZlMWQtNjNjMS00MGViLWFkMWMtZmVkMTQ5OGYxMWU3In0.UzQzgMlFWJ-fSbRkq7SZu0QVtzgAVpEfVtjJYls4oRgmrtNAz4Dla1J5kqBmWt2V9Q-QoBm_6r1KZZ97qZtO4Q
```

## JWK Rotation

It is recommended that the JWK generated in the [Prerequisite](#prerequisites-jwks-endpoint) step used in the signing of JWT is rotated at least monthly. As the JWKS consists of a collection (array) of JWKs, rotation can be done by replacing the JWKS entirely or by changing the Key ID (kid) if there are a number of JWKs in the JWKS published to APEX.

## Hello World! APIs

These [APIs](docs/hello-world/jwt-auth.md) can be subscribed to and can help the Developer to:

- Evaluate if the JWT authentication header has been generated correctly.
- Evaluate the SHA-256 hash of the API payload binary which is sent to the API.