# JWTDataProvider

The JWTDataProvider is a plugin for service that use JWT authentication, allowing you to get data from out side sources, such as another one of your services or a third party API, and add that data to your JWT's payload.

## Installation

Start by adding the dependency to your `Package.swift`:

    .package(url: "https://github.com/skelpo/JWTDataProvider.git", .upToNextMajor("0.8.0"))

Then add the `JWTDataProvider`  debendency to any of the targets you want to access it in and update your packages

## Configuration

Create a `Config/service.json` configuration file. Start by adding an empty `services` object:

```json
{
    "services": {}
}
```

The entries of this object are used to get data outside the service that needs to be stored in the access token payload. The structure of the objects that you can place in them will look like this:

```json
"<service_name>": {
    "url": "https://api.myotherservice.io/...",

    "//": "All the following keys are optional",
    "//": "The method key is case-insensative. It defaults to 'GET'",
    "method": "GET",

    "//": "Defaults to an empty object",
    "body": {...},

    "//": "Defaults to an empty object",
    "header": {"Content-Type": "application/json", ...},

    "//": "The below key defaults to false",
    "requiresAccessToken": false,

    "//": "Defaults to an empty array (will get whole JSON object)",
    "filters": [
        "path",
        "to",
        "json",
        "values"
    ],

    "//": "Defaults to nil",
    "default": "Some empty value of any type"
}
```

The filters key is an array of the key path to get from the JSON returned from the given URL. This is able to fetch from objects held in arrays.

The authentication allowed by this configuration is a bit constrained at this time. It uses an access token generated by the User Service to authenticate with other services, so you can only authenticate with services that use your User Service. The access token that is passed through will only ever contain the basic payload and then will have the additional data added before being returned froim the service's authentication route.

When the data is retreived from the outside service, the JSON is added to the access token payload with the JSON value fetch as the value and the service name as the key.

## Implementation

First, import that package:

    import JWTDataProvider

If you don't need to authenticate with any of the services you are attatching to, then you can just do the following:

    var payload: JSON = // Create payload here...
    payload = try payload(with: [:])
    
If you do need to authenticate with one of your services to access it, then you will need to create a JWT with the standard payload, get and merge the data the payload instace, then create another JWT with the updated payload:

    var accessTokenData: JSON = myPayload
    var accessJwt: JWT = try JWT(/* Init stuff with standard payload */)

    accessTokenData = try accessTokenData.mergeFetch(with: [:], accessJwt.createToken())
    accessJwt = try JWT(/* Init stuff with updated payload */)

The dictionary that we are passing in when creating and merging the payload data is used to set place holders in the service's `url` value. For example, in the `service.json`, we might have service that accesses a user in the service with it's ID, so the `url` key might look like this:

    https://api.userservice.io/users/{user_id}

We can set the `user_id` placeholder in that URL when we get the payload data by passing in the following dictionary:

    ["user_id": myUserId]

The JWTDataProvider will then replace the `user_id` placeholder in the URL with the value in the `myUserId` variable.
