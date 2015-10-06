# Identity Credentials Browser API

A browser polyfill that provides the Identity Credentials Browser API, which
supports:

 * Registration of decentralized identifiers (DIDs)
 * Storing credentials
 * Getting credentials

# Documentation

This API enables a developer to Web applications that can create new DIDs for
an entity, get credentials, and store credentials through the browser. The
API is outlined below, separated by different actors in the system:

Credential issuer API:
* *navigator.credentials.store(* **credential** *)*

Credential consumer API:
* *navigator.credentials.get(* **options** *)*

Identity provider APIs:
* *navigator.credentials.registerDid(* **options** *)*
* *navigator.credentials.getPendingOperation(* **options** *)*
* *CredentialOperation.complete(* **result** *)*

## Registering a decentralized identifier

The *navigator.credentials.registerDid(* **options** *)* call can be
used to register a new decentralized identifier and link it to the entity's
identity provider.

The call takes the following arguments:

* **options** (**required** *object*)
 * **idp** (*string*) - A decentralized identifier for the identity provider
   that should be associated with the newly created decentralized identifier.

The call returns a *Promise* that resolves to the document associated with
the registered DID.

Example:

```javascript
navigator.credentials.registerDid({
  idp: 'did:d1d1d1d1-d1d1-d1d1-d1d1-d1d1d1d1d1d1'
}).then(function(didDocument) {
  // ...
});
```

The example above will result in an a JSON-LD document that looks like
the following:

```jsonld
{
  "@context": "https://w3id.org/identity/v1",
  "id": "did:59cf8ba9-70f6-456e-aa7f-6e898c3a3e5f",
  "idp": "did:d1d1d1d1-d1d1-d1d1-d1d1-d1d1d1d1d1d1",
  "accessControl": {
    "writePermission": [{
      "id": "did:59cf8ba9-70f6-456e-aa7f-6e898c3a3e5f/keys/1",
      "type": "CryptographicKey"
    }, {
      "id": "did:d1d1d1d1-d1d1-d1d1-d1d1-d1d1d1d1d1d1",
      "type": "Identity"
    }]
  },
  "publicKey": [{
    "id": "did:59cf8ba9-70f6-456e-aa7f-6e898c3a3e5f/keys/1",
    "type": "CryptographicKey",
    "owner": "did:59cf8ba9-70f6-456e-aa7f-6e898c3a3e5f",
    "publicKeyPem": "-----BEGIN PUBLIC KEY-----\r\nMIIBI...AQAB\r\n-----END PUBLIC KEY-----\r\n"
  }],
  "signature": {
    "type": "GraphSignature2012",
    "created": "2015-07-02T16:54:27Z",
    "creator": "did:59cf8ba9-70f6-456e-aa7f-6e898c3a3e5f/keys/1",
    "signatureValue": "JukNUu...I0g=="
  }
}
```

## Storing a Credential

The *navigator.credentials.store(* **credential** *)* call can be
used to store a set of attributes about an entity, backed by credentials,
at an entity's identity provider.

The call takes the following arguments:

* **credential** (**required** *object*) - A JSON-LD document that
  contains at least one valid *credential* entry.

The call returns a *Promise* that resolves to a JSON-LD document that contains
the credentials that were stored.

```javascript
navigator.credentials.store({
  "@context": "https://w3id.org/identity/v1",
  "id": "did:04054703-8c94-46a3-bae7-7ffd07c0c962",
  "credential": [{
    "@graph": {
      "@context": "https://w3id.org/identity/v1",
      "id": "https://issuer.example.com/creds/1",
      "type": ["Credential", "EmailCredential"],
      "claim": {
        "id": "did:04054703-8c94-46a3-bae7-7ffd07c0c962",
        "email": "test@example.com"
      },
      "signature": {
        "type": "GraphSignature2012",
        "created": "2015-07-02T17:41:39Z",
        "creator": "https://issuer.example.com/keys/1",
        "signatureValue": "Tyd5S0A...nx33Yg=="
      }
    }
  }]
}).then(function(identity) {
  // ...
});
```

The example above will result in a JSON-LD document that looks like
the one that was passed via the *credential* parameter. Optionally, the
recipient of the credentials may choose to not store some of the credentials
and can notify the issuer that those credentials were not stored by
omitting them from the response.

## Getting a Credential

The *navigator.credentials.get(* **options** *)* call can be used to
request a set of properties about an entity that are backed by
credentials from an entity's identity provider.

The call takes the following arguments:

* **options** (**required** *object*)
 * **query** (**required** *object*) - A JSON-LD document that is a
   "query by example". The query consists of the attributes associated with
   an entity that the credential consumer would like to see.

The call returns a *Promise* that resolves to a JSON-LD document that contains
the credentials that were retrieved.

```javascript
navigator.credentials.request({
  query: {
    '@context': 'https://w3id.org/identity/v1',
    id: '',
    email: ''
  }
}).then(function(identity) {
  // ...
});
```

The example above will eventually result in the following JSON-LD document:

```jsonld
{
  "@context": "https://w3id.org/identity/v1",
  "id": "did:04054703-8c94-46a3-bae7-7ffd07c0c962",
  "type": "Identity",
  "credential": [{
    "@graph": {
      "@context": "https://w3id.org/identity/v1",
      "id": "urn:credential-1",
      "type": ["Credential", "CryptographicKeyCredential"],
      "claim": {
        "id": "did:04054703-8c94-46a3-bae7-7ffd07c0c962",
        "email": "test@example.com",
        "publicKey": {
          "id": "did:04054703-8c94-46a3-bae7-7ffd07c0c962/keys/1",
          "publicKeyPem": "-----BEGIN PUBLIC KEY-----\r\nMIIBIj...IDAQAB\r\n-----END PUBLIC KEY-----\r\n"
        }
      },
      "signature": {
        "type": "GraphSignature2012",
        "created": "2015-07-02T17:45:21Z",
        "creator": "https://authorization.dev:33443/idp/keys/1",
        "signatureValue": "S33Qcs...zWDqQQ=="
      }
    }
  }],
  "signature": {
    "type": "GraphSignature2012",
    "created": "2015-07-02T17:46:04Z",
    "creator": "did:04054703-8c94-46a3-bae7-7ffd07c0c962/keys/1",
    "signatureValue": "LpoVj...LP2A=="
  }
}
```

## Getting a Pending Credential Operation

The `getPendingOperation` method is only used by identity providers to complete
a pending `get` or `store` credentials operation once authorization has been
provided by the entity.

The call takes no arguments. It returns a *Promise* that resolves to a
*CredentialOperation*. A *CredentialOperation* has the following properties:

* **name** - The name of the pending operation (ie: 'get' or 'store').
* **options** - Present if the operation name is 'get'.
 * **query** - The query passed to `navigator.credentials.get`.
 * **publicKey** - The entity's public key for the device they are using.
* **credential** - Present if operation name is `store`. Contains the
  credentials document passed to `navigator.credentials.store`.

The identity provider can now help the entity to fulfill the credentials
query or ask it to accept the storage request. Once the identity provider
has completed the operation, it must call `complete` on the
*CredentialOperation* instance, passing the result of the operation. This
call will cause the browser to navigate away from the identity provider
with the result.

```javascript
navigator.credentials.getPendingOperation({
  "@context": "https://w3id.org/identity/v1",
  "id": "did:04054703-8c94-46a3-bae7-7ffd07c0c962",
  "type": "Identity",
  "credential": [{
    "@graph": {
      "@context": "https://w3id.org/identity/v1",
      "id": "urn:credential-1",
      "type": ["Credential", "CryptographicKeyCredential"],
      "claim": {
        "id": "did:04054703-8c94-46a3-bae7-7ffd07c0c962",
        "publicKey": {
          "id": "did:04054703-8c94-46a3-bae7-7ffd07c0c962/keys/1",
          "publicKeyPem": "-----BEGIN PUBLIC KEY-----\r\nMIIBIj...IDAQAB\r\n-----END PUBLIC KEY-----\r\n"
        }
      },
      "signature": {
        "type": "GraphSignature2012",
        "created": "2015-07-02T17:45:21Z",
        "creator": "https://authorization.dev:33443/idp/keys/1",
        "signatureValue": "S33Qcs...zWDqQQ=="
      }
    }
  }]
}).then(function(operation) {
  // ...
  
  // operation now complete
  operation.complete(result);
});
```

Source
------

The source code for the JavaScript implementation is available at:

https://github.com/digitalbazaar/credentials-polyfill
