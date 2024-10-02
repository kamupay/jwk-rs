# kamupay-jsonwebkey

This is a fork of [Nick Hynes's jsonwebkey](https://github.com/nhynes/jwk-rs)

*[JSON Web Key (JWK)](https://tools.ietf.org/html/rfc7517#section-4.3) (de)serialization, generation, and conversion.*

**Goals**

tl;dr: get keys into a format that can be used by other crates; be as safe as possible while doing so.

- Serialization and deserialization of *Required* and *Recommended* key types (HS256, RS256, ES256)
- Conversion to PEM for interop with existing JWT libraries (e.g., [jsonwebtoken](https://crates.io/crates/jsonwebtoken))
- Key generation (particularly useful for testing)

**Non-goals**

- be a fully-featured JOSE framework

## Examples

### Deserializing from JSON

```rust
extern crate jsonwebkey as jwk;
// Generated using https://mkjwk.org/.
let jwt_str = r#"{
   "kty": "oct",
   "use": "sig",
   "kid": "my signing key",
   "k": "Wpj30SfkzM_m0Sa_B2NqNw",
   "alg": "HS256"
}"#;
let the_jwk: jwk::JsonWebKey = jwt_str.parse().unwrap();
println!("{:#?}", the_jwk); // looks like `jwt_str` but with reordered fields.
```

### Using with other crates

```rust
#[cfg(all(feature = "generate", feature = "jwt-convert"))] {
extern crate jsonwebtoken as jwt;
extern crate jsonwebkey as jwk;

#[derive(serde::Serialize, serde::Deserialize)]
struct TokenClaims {
   exp: usize
}

let mut my_jwk = jwk::JsonWebKey::new(jwk::Key::generate_p256());
my_jwk.set_algorithm(jwk::Algorithm::ES256);

let alg: jwt::Algorithm = my_jwk.algorithm.unwrap().into();
let token = jwt::encode(
    &jwt::Header::new(alg),
    &TokenClaims { exp: 1492 },
    &my_jwk.key.to_encoding_key(),
).unwrap();

let mut validation = jwt::Validation::new(alg);
validation.validate_exp = false;
jwt::decode::<TokenClaims>(&token, &my_jwk.key.to_decoding_key(), &validation).unwrap();
}
```

## Features

- `pkcs-convert` - enables `Key::{to_der, to_pem}`.
                   This pulls in the [yasna](https://crates.io/crates/yasna) crate.
- `generate` - enables `Key::{generate_p256, generate_symmetric}`.
               This pulls in the [p256](https://crates.io/crates/p256) and [rand](https://crates.io/crates/rand) crates.
- `jwt-convert` - enables conversions to types in the
                  [jsonwebtoken](https://crates.io/crates/jsonwebtoken) crate.
