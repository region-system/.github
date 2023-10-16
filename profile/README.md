# User/Service->Service Authentication

## Why
Other methods of server to service authentication are painful. You're probably using IP allow-listing or preset tokens, if that, let's be honest a good amount of you are hoping someone never finds your internal endpoints (I feel sick at the idea of that). The region system creates a registry where each service can sign data, identify itself, and the endpoint service can verify that data came from a specific service, you can also do routing between services! (at a load-balancer level)

## Quick FYI
This is just a general overview of how the Region System works, it's pretty hands on dirty work here. If you're looking for code, head the [libraries](https://github.com/region-suite/about/libraries.md) and [examples](https://github.com/region-suite/about/examples.md).

## Example payloads 
(this data will be wrapped in a JWT, it's raw JSON so you get an idea of what it is)

### Requester-Service Authentication Payload
<img width="507" alt="Screenshot 2023-10-09 at 9 26 21 PM" src="https://github.com/region-suite/about/assets/91714073/57de7380-9f6d-4a5b-9e38-bc04671a4f6d">

### Region DNS Record
<img width="511" alt="Screenshot 2023-10-09 at 9 27 07 PM" src="https://github.com/region-suite/about/assets/91714073/520b67a8-38f0-4209-ac59-2f75c960b210">

### Capability DNS Record
<img width="536" alt="Screenshot 2023-10-09 at 9 26 57 PM" src="https://github.com/region-suite/about/assets/91714073/328c55c9-6eb3-4f8f-848e-224160c0e59c">

## Registries
- DNS
- HTTPS
- Local store

In the region system, whenever a client wants to verify a request it will reach out to the registry sent in the authentication packet so long as that registry is in it's allow-list. Each registry type returns the same data, but in (very) slightly different formats and methods of getting that data. [Libraries](https://github.com/region-suite/about/libraries.md) take care of reaching out to the required registry and verifying the data for you.

DNS: A DNS registry is nice because you don't need to worry about keeping registry servers up, however, DNS is heavily cached but using a DNS server like [1.1.1.1](https://1.1.1.1/) with a low TTL in both the DNS record and DNS client should be perfectly fine for most people.

HTTPS: When calling an HTTPS registry, the endpoint web-server sends a GET request to the specified URL so long as that _exact_ url (including the pathname and params) is in it's allow-list. That webserver should return a response like:
```
{
  "regions": [
    {..}
  ],
  "capabilities": [
    {..}
  ]
}
```

Local-Store: A Local-Store registry returns the exact same data as an HTTPS registry, but you can get it from a location such as a text file or environment variable and pass it to a library/your code.

``If you're wanting to generate your own HTTPS/Local-Store responses and don't hate yourself, use a library.``

## DNS Registry Explaination
<img src="https://github.com/region-system/.github/assets/91714073/9283f467-6ce8-4401-994a-5ed4d220f7c5" height="350"/>

*Graphic design is my passion...as you can see /j*

## User Authentication
For user authentication, you can replace the "capability" field value with the name of your user group. In the "id" field, you should put a user ID, **do not reveal any personally identifying information about the user, registries are public**.

For example, you could have a web portal at "sales.staff.example.com". When a user is successfully authenticated (using a login page at "auth.example.com"), the client uploads a public key, your authentication service can add a new entry to the registry with that public-key, something like:
```

{
  "id": "[userid]",
  "capability": "staff_members",
  "publickey": "[public-key from client]"
}
```
Then "auth.example.com" should sign a JWT (pstt.. the [Javascript](https://example.com) and other libraries have a function to do this for you [Regions.GateKeeper.prove_myself()](https://github.com/region-suite/npm/prove_myself.md)) and store it in the browser's Cookies (so it's sent in every request). Then the user should be redirected to "sales.staff.example.com".

Then, upon redirect to "sales.staff.example.com":
- If you're running NGINX (or something similar), you can forward the request to something like [Gate-Keeper](https://github.com/region-suite/gate-keeper). Gate-Keeper will verify the request is valid and return the Registry Object (as we entered before) which includes the userid, from there you can query your DB to get the relevant user information. You should still ensure the user is on the relevant allowlists to access your resource, for example, sales teams don't need access to codebases.
- If you're using your own codebase, you can either also forward to [Gate-Keeper](https://github.com/region-suite/gate-keeper), or use the relevant [library](https://github.com/region-suite/about/libraries.md) to verify the request yourself.
