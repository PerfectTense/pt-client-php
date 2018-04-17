# pt-client

A PHP library used to interact with the Perfect Tense API.

## Authentication

Most of the Perfect Tense API endpoints require the following two authentication tokens to be sent in the header of requests.

### App Key

Every request from your integration must be sent with an `App key`, which is obtained upon registering your integration [here](https://app.perfecttense.com/api).

This token allows us to verify the source of all API requests.

### Api Key

Every user, upon creating a Perfect Tense account, immediately receives an `Api key` (found [here](https://app.perfecttense.com/api)).

While the `App key` is used to verify the source of a request, an `Api key` is used to verify that the request is sent on behalf of a user whose account is in good standing and is below their daily request limit.


[TODO]