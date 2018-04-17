# Perfect Tense PHP

A PHP library used to interact with the Perfect Tense API.

## Authentication

Most of the Perfect Tense API endpoints require the following two authentication tokens to be sent in the header of requests.

### App Key

Every request from your integration must be sent with an `App key`, which is obtained upon registering your integration [here](https://app.perfecttense.com/api).

This token allows us to verify the source of all API requests.

### Api Key

Every user, upon creating a Perfect Tense account, immediately receives an `Api key` (found [here](https://app.perfecttense.com/api)).

While the `App key` is used to verify the source of a request, an `Api key` is used to verify that the request is sent on behalf of a user whose account is in good standing and is below their daily request limit.


## Installing

Currently, the best way to use this SDK is to check out the project from this repository or download the project as a zip file.

## Generate an App Key

The best way to generate an App key is to use our UI [here](https://app.perfecttense.com/api).

However, you can alternatively use our `/generateAppKey` endpoint as well, available through this SDK.

```
$response = pt_generate_app_key("[API Key]", "Test App", "This is a brief description of the use of this application",
		"[Email address]", "[Application URL]");

$appKey = $response['key'];
```

## Initializing

Once you have obtained an `App key` [here](https://app.perfecttense.com/api), you can initialize a PTClient object and begin interacting with our API.

```
// Create a client for interaction with API
$ptClient = new PTClient(
	array(
		"appKey"=>$appKey
	)
);

```

## Submitting a Job

Most interaction with the Perfect Tense API will use the `/correct` endpoint. Every request sent to this endpoint specifies an array call `responseType`, which is an array of response types to receive. Please see our [API documentation](https://www.perfecttense.com/docs/#introduction) for a description of the available response types. By default, this library will request all available response types.

Use the provided `submitJob` function to handle this interaction with the `/correct` endpoint.

Note that while the `ptClient` object was configured with the registered `App key`, individual API requests must still provide the additional `API key` authentication token.


```
// Submit job to PT
$result = $client->submitJob("This iz a test sentence.", $apiKey, null, null);

```

## Get Usage Statistics

Users have a limited number of daily requests. Using our `/usage` endpoint, you may fetch usage statistics on behalf of users at any time.

```
// Fetch usage statistics
$usage = $ptClient->getUsage($apiKey);

// Number of API requests this user has remaining today.
$numReqRemaining = $usage['apiRemainToday'];
```

## Interaction With Result

### Interactive Editor

The interactive editor is the recommended way to interact with a Perfect Tense result. 

You can instantiate an interactive editor as follows:

```
$result = $client->submitJob("This iz a test sentence.", $apiKey, null, null);

$intEditor = new PTInteractiveEditor(
	array(
		'ptClient' => $ptClient,
		'data' => $result,
		'apiKey' => $apiKey,
		'ignoreNoReplacement' => True // ignore pure suggestions (This sentence is a fragment, etc.)
	)
);

```

If the interactive editor's API is not sufficient for your use-case, there are additional utilities provided directly through the PTClient.

When operating independently on the result object, please note that not all corrections are available at all times. Some corrections are dependent on others.

For example:

```
Input: "He hzve be there before"

Transformation 1: "hzve" -> "have" (spelling error)
Transformation 2: "have be" -> "has been" (verb agreement error)
```

Transformation 2 is not available until Transformation 1 has been accepted. If Transformation 1 is rejected, Transformation 2 will be lost.

The interactive editor will manage this state information for you.

When in doubt, there is a utility function to determine if a transformation is currently available or not:

```
// Returns true if the transform is currently available, else false
$canMakeTransform = $intEditor->canMakeTransform(transform);
```

### Iterate Through Corrections

Corrections may or may not be available at all times. Some are dependent on others, and may become invalid if certain corrections are accepted or rejected. All of this state information is managed for you by the interactive editor. Simply use the `hasNextTransform()` and `getNextTransform()` function calls to iterate through all available corrections.

```
while ($intEditor->hasNextTransform()) {

	// Get the next available transform
	$nextTransform = $intEditor->getNextTransform();

	if (...) {
		// Accept the correction
		$intEditor->acceptCorrection($nextTransform);
	} else {
		// Reject the correction
		$intEditor->rejectCorrection($nextTransform);
	}
}

```

### Print the Current State

While iterating through and accepting or rejecting corrections, the state of the text will change. At any time, call `getCurrentText()` to see the current state.

```
// Re-construct the current state of the job (considering accepted/rejected corrections)
$currentText = $intEditor->getCurrentText();

console.log(currentText)
```

### Retrieve Character Offsets

Many applications will need to reference the character offset of corrections. These can be recovered using the `getSentenceOffset` and `getTransformOffset` utilities as follows:

```
// Find character offset of the transformation
$offset = $intEditor->getTransformDocumentOffset($transform);

// Length of the text affected by this transformation
$affectedLength = strlen($intEditor->getAffectedText($transform));

// Your application can highlight the range (offset, offset + affectedLength)

```
### Grammar Score

By default, the Perfect Tense client will fetch a numerical grammar score for the text (0 - 100). This can be recovered as follows:

```
$grammarScore = $intEditor->getGrammarScore();
```

## API Documentation

See our [API documentation](https://www.perfecttense.com/docs/#introduction) for more information.