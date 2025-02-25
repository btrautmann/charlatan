# charlatan

[![pub package](https://img.shields.io/pub/v/charlatan.svg)](https://pub.dev/packages/charlatan)
[![Build status](https://github.com/Betterment/dart_fake_http/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/Betterment/charlatan/actions/workflows/ci.yml?query=branch%3Amain)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://GitHub.com/Betterment/charlatan/pulse)

This package provides the ability to configure and return fake HTTP
responses from your [Dio HTTP Client](https://pub.dev/packages/dio).
This makes it easy to test the behavior of code that interacts with
HTTP services without having to use mocks.

It consists of two components:

- `Charlatan` - a class for configuring and providing fake HTTP responses
  based on HTTP method and URI template.
- `CharlatanHttpClientAdapter` - an implementation of Dio's
  `HttpClientAdapter` that returns responses from a configured
  `Charlatan` instance.

## Usage

Add `charlatan` to your pubspec.yaml's `dev_dependencies`:

```yaml
# pubspec.yaml
dev_dependencies:
  charlatan:
```

### Configuring fake responses

Create an instance of `Charlatan` and call the corresponding
configuration method for the HTTP method you want to map a request to.

You can configure fakes responses using a specific path or a URI
template. You can also use the request object to customize your
response.

```dart
final charlatan = Charlatan();
charlatan.whenPost('/users', (_) => { 'id': 1, 'bilbo' });
charlatan.whenGet('/users/{id}', (req) => { 'id': req.pathParameters['id'], 'name': 'bilbo' });
charlatan.whenPut('/users/{id}/profile', (_) => null, statusCode: 204);
charlatan.whenDelete('/users/{id}', (_) => null, statusCode: 204);
```

If you need to further customize the response, you can return a
`CharlatanHttpResponse`.

```dart
charlatan.whenPost('/users', (req) {
  final data = req.body as Map<String, Object?>? ?? {};
  final name = data['name'] as String?;
  if (name == null) {
    return CharlatanHttpResponse(
      statusCode: 422,
      body: {
        'errors': {
          'name': ['cannot be blank'],
        },
      },
    );
  }

  return CharlatanHttpResponse(
    statusCode: 201,
    body: { 'id': 1, 'name': name },
  );
});
```

### Building a fake HTTP client

Build the `CharlatanHttpClientAdapter` from the `Charlatan` instance and then
assign it to your `Dio` instance's `httpClientAdapter`.

```dart
final charlatan = Charlatan();
// ... configure fake responses ...
final dio = Dio()..httpClientAdapter = charlatan.toFakeHttpClientAdapter();
```

Now make HTTP requests like your normally would and they will be routed
through your configured fakes.

```dart
final result = await dio.get<Object?>('/users/1');
expect(result.data, {'id', 1, 'name': 'bilbo'});
```

### FAQ

> What happens if I make a request that doesn't match a configured fake
> response?

You get a helpful error message like this:

```
Unable to find matching fake http response definition for:

GET /blahhhh

Did you configure it?

The fake http response definitions configured were:
GET /users
POST /users
PUT /users
DELETE /users
```

> How can I configure a fake response that relies upon the result of
> another fake request? e.g. a POST followed by a GET that can "read its
> own writes"

Check out the example directory.
