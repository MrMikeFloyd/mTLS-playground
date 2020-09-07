# mTLS playground

This is a minimal working sample involving all required certificates, keys and a self-generated
CA to allow mutual TLS communication between two communication parties.

## Prerequisites

You'll need `node` to run the sample server application.

## How to run

Start the server with `node index.js`, then connect to it using

```
curl -vvv --cacert ca/ca.crt --key client/client.key --cert client/client.crt https://localhost:3000
```

If all goes well, the server should respond with `200 OK` and a well-known greeting.

## How to break it

### No client cert

With mTLS, identity verification is _mutual_:

* The client verifies the server's identity (as it would with TLS)
* The server verifies the client's identity (this makes TLS _mTLS_)

Thus, if we let the client connect without providing certificate and key information, the
connection's going to fail:

```
curl -vvv --cacert ca/ca.crt https://localhost:3000
```

### Self signed certs

The example uses a self signed CA cert and works only because we explicitly tell `curl` to use it. When cURLing
with

```
curl -vvv --key client/client.key --cert client/client.crt https://localhost:3000
```

we can see that the connection is being brought down after TLS handshake due to the client rejecting the
server's self signed cert.

## [Re]Generating the required certificates

All required certs are contained in this repo, however should you want to create them yourself,
do as follows:

1. Create a fake (a.k.a self signed) CA

```
# Create it
openssl req -new -x509 -nodes -days 365 -subj '/CN=my-ca' -keyout ca.key -out ca.crt
# Check it
openssl x509 --in ca.crt -text --noout
```

2. Create the server cert

```
# Create it
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj '/CN=localhost' -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 365 -out server.crt
# Check it
openssl x509 -in server.crt -text -noout
```

3. Create the client cert

```
# Create it
openssl genrsa -out client.key 2048
openssl req -new -key client.key -subj '/CN=my-client' -out client.csr
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 365 -out client.crt
# Check it
openssl x509 -in client.crt -text -noout
```

## Sources

[1](https://codeburst.io/mutual-tls-authentication-mtls-de-mystified-11fa2a52e9cf) Helped greatly in creating this example.

[2](https://medium.com/@FreedomBen/what-is-mtls-and-how-does-it-work-9dcdbf6c1e41) Helped greatly in understanding mTLS.
