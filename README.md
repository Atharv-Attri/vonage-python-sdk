# Nexmo Client Library for Python

[![PyPI version](https://badge.fury.io/py/nexmo.svg)](https://badge.fury.io/py/nexmo)
[![Build Status](https://api.travis-ci.org/Nexmo/nexmo-python.svg?branch=master)](https://travis-ci.org/Nexmo/nexmo-python)
[![Coverage Status](https://coveralls.io/repos/github/Nexmo/nexmo-python/badge.svg?branch=master)](https://coveralls.io/github/Nexmo/nexmo-python?branch=master)
[![Python versions supported](https://img.shields.io/pypi/pyversions/nexmo.svg)](https://pypi.python.org/pypi/nexmo)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)

<img src="https://developer.nexmo.com/assets/images/Vonage_Nexmo.svg" height="48px" alt="Nexmo is now known as Vonage" />

This is the Python client library for Nexmo's API. To use it you'll
need a Nexmo account. Sign up [for free at nexmo.com][signup].

- [Installation](#installation)
- [Usage](#usage)
- [SMS API](#sms-api)
- [Voice API](#voice-api)
- [Verify API](#verify-api)
- [Number Insight API](#number-insight-api)
- [Number Management API](#number-management-api)
- [Managing Secrets](#managing-secrets)
- [Application API](#application-api)
- [Overriding API url's](#overriding-api-urls)
- [License](#license)

## Installation

To install the Python client library using pip:

    pip install nexmo

To upgrade your installed client library using pip:

    pip install nexmo --upgrade

Alternatively, you can clone the repository via the command line:

    git clone git@github.com:Nexmo/nexmo-python.git

or by opening it on GitHub desktop.

## Usage

Begin by importing the `nexmo` module:

```python
import nexmo
```

Then construct a client object with your key and secret:

```python
client = nexmo.Client(key=api_key, secret=api_secret)
```

For production, you can specify the `NEXMO_API_KEY` and `NEXMO_API_SECRET`
environment variables instead of specifying the key and secret explicitly.

For newer endpoints that support JWT authentication such as the Voice API,
you can also specify the `application_id` and `private_key` arguments:

```python
client = nexmo.Client(application_id=application_id, private_key=private_key)
```

To check signatures for incoming webhook requests, you'll also need
to specify the `signature_secret` argument (or the `NEXMO_SIGNATURE_SECRET`
environment variable).

## SMS API

### Send a text message

```python
response = client.send_message({'from': 'Python', 'to': 'YOUR-NUMBER', 'text': 'Hello world'})

response = response['messages'][0]

if response['status'] == '0':
  print('Sent message', response['message-id'])

  print('Remaining balance is', response['remaining-balance'])
else:
  print('Error:', response['error-text'])
```

Docs: [https://developer.nexmo.com/api/sms#send-an-sms](https://developer.nexmo.com/api/sms?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#send-an-sms)

### Tell Nexmo the SMS was received

The following submits a successful conversion to Nexmo with the current timestamp. This feature must
be enabled on your account first.

```python
response = client.submit_sms_conversion(message_id)
```

### Signing a Message

_You may also like to read the [documentation about message signing](https://developer.nexmo.com/concepts/guides/signing-messages)._

The SMS API supports the ability to sign messages by generating and adding a signature using a "Signature Secret" rather than your API secret. The algorithms supported are:

md5hash1
md5
sha1
sha256
sha512

Both your application and Nexmo need to agree on which algorithm is used. In the dashboard, visit your account settings page and under "API Settings" you can select the algorithm to use. This is also the location where you will find your "Signature Secret" (it's different from the API secret).

### Create a client using these credentials and the algorithm to use, for example:

```python
client = nexmo.Client(
            key = os.getenv('NEXMO_API_KEY'),
            signature_secret = os.getenv('NEXMO_SIGNATURE_SECRET'),
            signature_method = 'sha256'
)
```

Using this client, your SMS API messages will be sent as signed messages.

### Verifying an Incoming Message Signature

_You may also like to read the [documentation about message signing](https://developer.nexmo.com/concepts/guides/signing-messages)_.

If you have message signing enabled for incoming messages, the SMS webhook will include the fields sig, nonce and timestamp.

To verify the signature is from Nexmo, you create a Signature object using the incoming data, your signature secret and the signature method.

Then use the `check_signature()` method with the actual signature that was received (usually present in request.form or request.args. you can merge those in a single variable called params) to make sure that it is correct.

### Get the params

```python
if request.is_json:
    params = request.get_json()
else:
    params = request.args or request.form
is_valid = client.check_signature(params)// is it valid? Will be true or false
```

Using your signature secret and the other supplied parameters, the signature can be calculated and checked against the incoming signature value.

## Voice API

### Make a call

```python
response = client.create_call({
  'to': [{'type': 'phone', 'number': '14843331234'}],
  'from': {'type': 'phone', 'number': '14843335555'},
  'answer_url': ['https://example.com/answer']
})
```

Docs: [https://developer.nexmo.com/api/voice#createCall](https://developer.nexmo.com/api/voice?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#createCall)

### Retrieve a list of calls

```python
response = client.get_calls()
```

Docs: [https://developer.nexmo.com/api/voice#getCalls](https://developer.nexmo.com/api/voice?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#getCalls)

### Retrieve a single call

```python
response = client.get_call(uuid)
```

Docs: [https://developer.nexmo.com/api/voice#getCall](https://developer.nexmo.com/api/voice?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#getCall)

### Update a call

```python
response = client.update_call(uuid, action='hangup')
```

Docs: [https://developer.nexmo.com/api/voice#updateCall](https://developer.nexmo.com/api/voice?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#updateCall)

### Stream audio to a call

```python
stream_url = 'https://nexmo-community.github.io/ncco-examples/assets/voice_api_audio_streaming.mp3'

response = client.send_audio(uuid, stream_url=[stream_url])
```

Docs: [https://developer.nexmo.com/api/voice#startStream](https://developer.nexmo.com/api/voice?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#startStream)

### Stop streaming audio to a call

```python
response = client.stop_audio(uuid)
```

Docs: [https://developer.nexmo.com/api/voice#stopStream](https://developer.nexmo.com/api/voice?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#stopStream)

### Send a synthesized speech message to a call

```python
response = client.send_speech(uuid, text='Hello')
```

Docs: [https://developer.nexmo.com/api/voice#startTalk](https://developer.nexmo.com/api/voice?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#startTalk)

### Stop sending a synthesized speech message to a call

```python
response = client.stop_speech(uuid)
```

Docs: [https://developer.nexmo.com/api/voice#stopTalk](https://developer.nexmo.com/api/voice?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#stopTalk)

### Send DTMF tones to a call

```python
response = client.send_dtmf(uuid, digits='1234')
```

Docs: [https://developer.nexmo.com/api/voice#startDTMF](https://developer.nexmo.com/api/voice?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#startDTMF)

### Get recording

```python
response = client.get_recording(RECORDING_URL)
```

## Verify API

### Create an instance of the class

To create an instance of the Verify class, Just follow the following steps:

- **Import the class from module** (3 different ways)

```python
#First way
from nexmo import Verify

#Second way
from nexmo.verify import Verify

#Third valid way
import nexmo #then tou can use nexmo.Verify() to create an instance
```

- **Create the instance**

```python
#First way - pass key and secret to the constructor
verify = Verify(key=NEXMO_API_KEY, secret=NEXMO_API_SECRET)

#Second way - Create a client instance and then pass the client to the Verify constructor
client = Client(key=NEXMO_API_KEY, secret=NEXMO_API_SECRET)
verify = Verify(client)
```

### Search for a Verification request

```python
client = Client(key='API_KEY', secret='API_SECRET')

verify = Verify(client)
response = verify.search('69e2626cbc23451fbbc02f627a959677')

if response is not None:
    print(response['status'])
```

### Send verification code

```python
client = Client(key='API_KEY', secret='API_SECRET')

verify = Verify(client)
response = verify.request(number=RECIPIENT_NUMBER, brand='AcmeInc')

if response["status"] == "0":
    print("Started verification request_id is %s" % (response["request_id"]))
else:
    print("Error: %s" % response["error_text"])
```

### Send verification code with workflow

```python
client = Client(key='API_KEY', secret='API_SECRET')

verify = Verify(client)
response = verify.request(number=RECIPIENT_NUMBER, brand='AcmeInc', workflow_id=1)

if response["status"] == "0":
    print("Started verification request_id is %s" % (response["request_id"]))
else:
    print("Error: %s" % response["error_text"])
```

### Check verification code

```python
client = Client(key='API_KEY', secret='API_SECRET')

verify = Verify(client)
response = verify.check(REQUEST_ID, code=CODE)

if response["status"] == "0":
    print("Verification successful, event_id is %s" % (response["event_id"]))
else:
    print("Error: %s" % response["error_text"])
```

### Cancel Verification Request

```python
client = Client(key='API_KEY', secret='API_SECRET')

verify = Verify(client)
response = verify.cancel(REQUEST_ID)

if response["status"] == "0":
    print("Cancellation successful")
else:
    print("Error: %s" % response["error_text"])
```

### Trigger next verification proccess

```python
client = Client(key='API_KEY', secret='API_SECRET')

verify = Verify(client)
response = verify.trigger_next_event(REQUEST_ID)

if response["status"] == "0":
    print("Next verification stage triggered")
else:
    print("Error: %s" % response["error_text"])
```

### Send payment authentication code

```python
client = Client(key='API_KEY', secret='API_SECRET')

verify = Verify(client)
response = verify.psd2(number=RECIPIENT_NUMBER, payee=PAYEE, amount=AMOUNT)

if response["status"] == "0":
    print("Started PSD2 verification request_id is %s" % (response["request_id"]))
else:
    print("Error: %s" % response["error_text"])
```

### Send payment authentication code with workflow

```python
client = Client(key='API_KEY', secret='API_SECRET')

verify = Verify(client)
verify.psd2(number=RECIPIENT_NUMBER, payee=PAYEE, amount=AMOUNT, workflow_id: WORKFLOW_ID)

if response["status"] == "0":
    print("Started PSD2 verification request_id is %s" % (response["request_id"]))
else:
    print("Error: %s" % response["error_text"])
```

Docs: [https://developer.nexmo.com/api/verify](https://developer.nexmo.com/api/verify?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library)

## Number Insight API

### Basic Number Insight

```python
client.get_basic_number_insight(number='447700900000')
```

Docs: [https://developer.nexmo.com/api/number-insight#getNumberInsightBasic](https://developer.nexmo.com/api/number-insight?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#getNumberInsightBasic)

### Standard Number Insight

```python
client.get_standard_number_insight(number='447700900000')
```

Docs: [https://developer.nexmo.com/api/number-insight#getNumberInsightStandard](https://developer.nexmo.com/api/number-insight?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#getNumberInsightStandard)

### Advanced Number Insight

```python
client.get_advanced_number_insight(number='447700900000')
```

Docs: [https://developer.nexmo.com/api/number-insight#getNumberInsightAdvanced](https://developer.nexmo.com/api/number-insight?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#getNumberInsightAdvanced)

## Number Management API

### List Your Numbers

```python
client.get_account_numbers()
```

Docs: [https://developer.nexmo.com/api/numbers#getOwnedNumbers](https://developer.nexmo.com/api/numbers?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#getOwnedNumbers)

### Search for a Number

```python
client.get_available_numbers('GB', {"type":"SMS"})
```

Docs: [https://developer.nexmo.com/api/numbers#getAvailableNumbers](https://developer.nexmo.com/api/numbers?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#getAvailableNumbers)

### Buy a Number

```python
client.buy_number({"country": 'GB', "msisdn": '447700900000'})
```

Docs: [https://developer.nexmo.com/api/numbers#buyANumber](https://developer.nexmo.com/api/numbers?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#buyANumber)

### Cancel a Number

```python
client.cancel_number({"country": 'GB', "msisdn": '447700900000'})
```

Docs: [https://developer.nexmo.com/api/numbers#cancelANumber](https://developer.nexmo.com/api/numbers?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#cancelANumber)

## Managing Secrets

An API is provided to allow you to rotate your API secrets. You can create a new secret (up to a maximum of two secrets) and delete the existing one once all applications have been updated.

### List Secrets

```python
secrets = client.list_secrets(API_KEY)
```

### Create A New Secret

Create a new secret (the created dates will help you know which is which):

```python
client.create_secret(API_KEY, 'awes0meNewSekret!!;');
```

### Delete A Secret

Delete the old secret (any application still using these credentials will stop working):

```python
client.delete_secret(API_KEY, 'my-secret-id')
```

## Application API

### Create an application

```python
response = client.application_v2.create_application({name='Example App', type='voice'})
```

Docs: [https://developer.nexmo.com/api/application.v2#createApplication](https://developer.nexmo.com/api/application.v2#createApplication?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#create-an-application)

### Retrieve a list of applications

```python
response = client.application_v2.list_applications()
```

Docs: [https://developer.nexmo.com/api/application.v2#listApplication](https://developer.nexmo.com/api/application.v2#listApplication?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#retrieve-your-applications)

### Retrieve a single application

```python
response = client.application_v2.get_application(uuid)
```

Docs: [https://developer.nexmo.com/api/application.v2#getApplication](https://developer.nexmo.com/api/application.v2#getApplication?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#retrieve-an-application)

### Update an application

```python
response = client.application_v2.update_application(uuid, answer_method='POST')
```

Docs: [https://developer.nexmo.com/api/application.v2#updateApplication](https://developer.nexmo.com/api/application.v2#updateApplication?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#update-an-application)

### Delete an application

```python
response = client.application_v2.delete_application(uuid)
```

Docs: [https://developer.nexmo.com/api/application.v2#deleteApplication](https://developer.nexmo.com/api/application.v2#deleteApplication?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library#destroy-an-application)

## Validate webhook signatures

```python
client = nexmo.Client(signature_secret='secret')

if client.check_signature(request.query):
  # valid signature
else:
  # invalid signature
```

Docs: [https://developer.nexmo.com/concepts/guides/signing-messages](https://developer.nexmo.com/concepts/guides/signing-messages?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library)

Note: you'll need to contact support@nexmo.com to enable message signing on
your account before you can validate webhook signatures.

## JWT parameters

By default, the library generates short-lived tokens for JWT authentication.

Use the auth method to specify parameters for a longer life token or to
specify a different token identifier:

```python
client.auth(nbf=nbf, exp=exp, jti=jti)
```

## Overriding API url's

By default, our API url's are hardcoded. For use cases where these url's are not accessible, best practices to override these url's are the following:

- Setting new API url's when creating an instance of the client:

```python
import nexmo
client = nexmo.Client()
client.host = 'new.host.url'
client.api_host = 'new.api.host'
```

- Creating a new class that extends from client class and overrides these values in the constructor:

```python
class MyClient(nexmo.Client):
    def __init__(self, NEXMO_API_KEY, NEXMO_API_SECRET, APPLICATION_ID, APPLICATION_PRIVATE_KEY_PATH):
        super().__init__(application_id=APPLICATION_ID, private_key=APPLICATION_PRIVATE_KEY_PATH, key=NEXMO_API_KEY, secret=NEXMO_API_SECRET)
        self.host = 'new.hosts.url'
        self.api_host = 'new.api.hosts'

#usage
client = MyClient(NEXMO_API_KEY, NEXMO_API_SECRET, APPLICATION_ID, APPLICATION_PRIVATE_KEY_PATH)
```

For a more specific case, another way to customise is:

```python
import nexmo

class NexmoClient(nexmo.Client):
    def __init__(....):
        super().__init__(....)
        api_server = BasicAuthenticatedServer(
            "mycustomurl",
            user_agent=user_agent,
            api_key=self.api_key,
            api_secret=self.api_secret,
        )
        self.application_v2 = ApplicationV2(api_server)
```

Then proceed to create your personalised instance of the class.

## Contributing

We :heart: contributions! But if you plan to work on something big or controversial, please [contact us](mailto:devrel@nexmo.com) first!

We recommend working on `nexmo-python` with a [virtualenv][virtualenv]. The following command will install all the Python dependencies you need to run the tests:

```bash
make install
```

The tests are all written with pytest. You run them with:

```bash
make test
```

## License

This library is released under the [MIT License][license].

[virtualenv]: https://virtualenv.pypa.io/en/stable/
[report-a-bug]: https://github.com/Nexmo/nexmo-python/issues/new
[pull-request]: https://github.com/Nexmo/nexmo-python/pulls
[signup]: https://dashboard.nexmo.com/sign-up?utm_source=DEV_REL&utm_medium=github&utm_campaign=python-client-library
[license]: LICENSE.txt
