# JWT
A Ruby implementation of [JSON Web Token draft 06](http://self-issued.info/docs/draft-jones-json-web-token-06.html).

## Installing

```bash
sudo gem install jwt
```

## Usage

```ruby
payload = {"some" => "payload"}
JWT.encode(payload, "secret")
```

Note the resulting JWT will not be encrypted, but verifiable with a secret key.

```ruby
JWT.decode('someJWTstring', 'secret')
```

If the secret is wrong, it will raise a `JWT::DecodeError` telling you as such. You can still get at the payload by setting the verify argument to false.

```ruby
JWT.decode('someJWTstring', nil, false)
```

`encode` also allows for different signing algorithms as well as customer headers.

```ruby
require 'openssl'
# it's recommended to use RSA keys with at least 2048 bit
some_private_key = OpenSSL::PKey::RSA.new File.read('path/to/my/private/and/secure.pem'), 'password_for_my_private_key'
rsa_signed_token = JWT.encode({ "exp" => Time.now.to_i+3600, "name" => "some_name" }, some_private_key, "RS512")
```

## Algorithms

The JWT spec supports several algorithms for cryptographic signing. This library currently supports:

**HMAC**

* HS256	- HMAC using SHA-256 hash algorithm (default)
* HS384	- HMAC using SHA-384 hash algorithm
* HS512 - HMAC using SHA-512 hash algorithm

**RSA**

* RS256 - RSA using SHA-256 hash algorithm
* RS384 - RSA using SHA-384 hash algorithm
* RS512 - RSA using SHA-512 hash algorithm

Change the algorithm with by setting it in encode:

```ruby
JWT.encode({'some' => 'payload'}, 'secret', 'HS512')
```

**Plaintext**

We also support unsigned plaintext JWTs as introduced by draft 03 by explicitly specifying `nil` as the key and algorithm:

```ruby
jwt = JWT.encode({'some' => 'payload'}, nil, nil)
JWT.decode(jwt, nil, nil)
```

## Support for reserved claim names
JSON Web Token defines some reserved claim names and defines how they should be
used. JWT supports these reserved claim names:

 - 'exp' (Expiration Time) Claim
 - 'nbf' (Not Before Time) Claim
 - 'iss' (Issuer) Claim
 - 'aud' (Audience) Claim
 - 'jti' (JWT ID) Claim
 - 'iat' (Issued At) Claim
 - 'sub' (Subject) Claim

### Expiration Time Claim

From [draft 01 of the JWT spec](http://self-issued.info/docs/draft-jones-json-web-token-01.html#ReservedClaimName):

> The exp (expiration time) claim identifies the expiration time on or after
> which the JWT MUST NOT be accepted for processing. The processing of the exp
> claim requires that the current date/time MUST be before the expiration
> date/time listed in the exp claim. Implementers MAY provide for some small
> leeway, usually no more than a few minutes, to account for clock skew. Its
> value MUST be a number containing an IntDate value. Use of this claim is
> OPTIONAL.

You pass the expiration time as a UTC UNIX timestamp (an int). For example:

```ruby
JWT.encode({'exp' => 1371720939}, 'secret')
JWT.encode({'exp' => Time.now.to_i()}, 'secret')
```

Expiration time is automatically verified in `JWT.decode()` and raises
`JWT::ExpiredSignature` if the expiration time is in the past:

```ruby
begin
  JWT.decode('JWT_STRING', 'secret')
rescue JWT::ExpiredSignature
  # Signature has expired
end
```

Expiration time will be compared to the current UTC time (as given by
`Time.now.to_i`), so be sure to use a UTC timestamp or datetime in encoding.

You can turn off expiration time verification with the `verify_expiration` option.

JWT also supports the leeway part of the expiration time definition, which
means you can validate a expiration time which is in the past but not very far.
For example, if you have a JWT payload with a expiration time set to 30 seconds
after creation but you know that sometimes you will process it after 30 seconds,
you can set a leeway of 10 seconds in order to have some margin:

```ruby
jwt_payload = JWT.encode({'exp' => Time.now.to_i + 30}, 'secret')
sleep(32)
# jwt_payload is now expired
# But with some leeway, it will still validate
JWT.decode(jwt_payload, 'secret', true, {:leeway => 10})
```

### Not Before Time Claim

From [draft-ietf-oauth-json-web-token-32](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html#nbfDef):

> The nbf (not before) claim identifies the time before which the JWT MUST NOT
> be accepted for processing. The processing of the nbf claim requires that the
> current date/time MUST be after or equal to the not-before date/time listed
> in the nbf claim. Implementers MAY provide for some small leeway, usually no
> more than a few minutes, to account for clock skew. Its value MUST be a number
> containing a NumericDate value. Use of this claim is OPTIONAL.

You pass the not before time as a UTC UNIX timestamp (an int). For example:

```ruby
JWT.encode({'nbf' => 1371720939}, 'secret')
JWT.encode({'nbf' => Time.now.to_i()}, 'secret')
```

Not before time is automatically verified in `JWT.decode()` and raises
`JWT::ImmatureSignature` if the not before time is in the future:

```ruby
begin
  JWT.decode('JWT_STRING', 'secret')
rescue JWT::ImmatureSignature
  # Signature is immature
end
```

Not before time will be compared to the current UTC time (as given by
`Time.now.to_i`), so be sure to use a UTC timestamp or datetime in encoding.

You can turn off not before time verification with the `verify_not_before` option.

In a similar way to the expiration time claim, the not before time claim supports
the leeway option.

```ruby
jwt_payload = JWT.encode({'nbf' => Time.now.to_i + 30}, 'secret')
sleep(25)
# jwt_payload is now immature
# But with some leeway, it will still validate
JWT.decode(jwt_payload, 'secret', true, {:leeway => 10})
```

# Development and Tests

We depend on [Echoe](http://rubygems.org/gems/echoe) for defining gemspec and performing releases to rubygems.org, which can be done with

```bash
rake release
```

The tests are written with rspec. Given you have rake and rspec, you can run tests with

```bash
rake test
```

**If you want a release cut with your PR, please include a version bump according to [Semantic Versioning](http://semver.org/)**

## Contributors

 * Jordan Brough <github.jordanb@xoxy.net>
 * Ilya Zhitomirskiy <ilya@joindiaspora.com>
 * Daniel Grippi <daniel@joindiaspora.com>
 * Jeff Lindsay <progrium@gmail.com>
 * Bob Aman <bob@sporkmonger.com>
 * Micah Gates <github@mgates.com>
 * Rob Wygand <rob@wygand.com>
 * Ariel Salomon (Oscil8)
 * Paul Battley <pbattley@gmail.com>
 * Zane Shannon [@zshannon](https://github.com/zshannon)
 * Brian Fletcher [@punkle](https://github.com/punkle)
 * Alex [@ZhangHanDong](https://github.com/ZhangHanDong)
 * Tim Rudat [@excpt](https://github.com/excpt) <timrudat@gmail.com> - Maintainer

## License

MIT

Copyright (c) 2011 Jeff Lindsay

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
