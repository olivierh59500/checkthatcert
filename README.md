Check That Cert!
=============

Unless you verify who you are communicating with, using an encrypted channel won't provide you confidentiality or integrity. Not checking the SSL certificate of a remote server you are communicating with allows anyone with control of the network to stand up their own server and certificate, impersonating the server and subjecting the application to a MiTM attack.

Check That Cert is an attempt to document correct usage of TLS libraries that perform certificate checking.

# Self signed certificates vs certificates signed by a certificate authority

Self signed certificates are certificates that are not signed by a trusted authority, in other words - no "trusted" third party has vouched for these certificates. These certificates should never be trusted without explicitly verifying that the certificate of the remote server is the server you intend to communicate with. Certificates signed by a certificate authority can be verified by ensuring that a trusted certificate authority signed that certificate. When you need to communicate with a specific server, you'll want to ensure that your client is set to only trust certificate the server is presenting, rather than any certificate signed by a certificate authority. When you have many users who will use standard software you do not have control of (for example, a web browser), you'll want to submit a certificate signing request to a certificate authority to obtain a proper certificate.

# Generating a self signed certificate

While manging PKI is out of scope for Check That Cert, self-signed certificates often fulfill the needs of application developers. If you only need to communicate with one server that you already control (and nobody else needs to communicate with) you should create a self signed certificate. This can be done with the openssl utilities, included with Linux and available on windows with cygwin.

````
openssl genrsa -out server.pem 4096
openssl req -new -x509 -key server.pem -out cert.pem -days 30
````

Sample output:

````
[user@localhost testcase]$ openssl genrsa -out server.pem 4096
Generating RSA private key, 4096 bit long modulus
....................................................................................++
...........................++
e is 65537 (0x10001)
[user@localhost testcase]$ openssl req -new -x509 -key server.pem -out cert.pem -days 30
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []:Washington
Locality Name (eg, city) [Default City]:Seattle
Organization Name (eg, company) [Default Company Ltd]:Check That Cert
Organizational Unit Name (eg, section) []:Testcases
Common Name (eg, your name or your server's hostname) []:localhost
Email Address []:someone@example.com
[user@localhost testcase]$ 
````

Pay special attention when setting the "hostname" or "Common Name" field. This must match the hostname of the server you are communicating with. You should only use hostnames that you control to ensure that you are the only one who can issue a certificate that will be trusted by your software.

## Hosting a simple server to test encrypted communication

````
openssl s_server -tls1 -tls1_1 -tls1_2 -key server.pem -cert cert.pem -www
````


# What needs verification?

## Hostname

The hostname (or CN) is a commonly accepted way of identifying the name of the system you are communicating with. In the event that an attacker redirects your traffic to another system trusted by client, validation of the hostname will prevent the transport from operating to the incorrect service.

## Issuer

The Issuer is the name of the authority vouching for a certificate. It is important that this is bound to certificate that is present on the system and not simply a string comparison of the Subject name.

## Valid from-to timestamps

In order to ensure that a certificate isn't used beyond it's expiration, the valid-to timestamp should be verified against the machine's time. In order to prevent the system from trusting an expired certificate when the clock is wrong, the valid-from timestamp should be verified. Short lived certificates can reduce the chance of an attacker being able to use a certificate should they be able to compromise it. 

## Extended Key Use

# Pitfalls to certificate validation

## Not checking the certificate

Failure to check the certificate means your client will communicate with any server using that transport. If you send secrets over the channel, it is trivial for an attacker with the ability to redirect network traffic to read them.

## Trusting more than the server you need to communicate with

The more certificates and certificate authorities you trust, the higher the chance of your transport being intercepted. Limit the parties you trust and review them frequently to ensure you still trust them. 

## Not checking to see if the certificate is expired

Certificates are intended to be live for a finite period of time. Having certificates expire ensures that systems will not trust the certificate forever if it is compromised, lost, or revoked. If your system trusts a compromised certificate, an attacker could stand up a fake server and your client would connect to it.

## Not checking a certificate revocation list

In some cases, checking the CRL is a critical step to ensuring that the certificate should be trusted. This is particularly important for remote access software that uses client-certificates to perform authentication, and is less interesting for environments where a client uses a CRL to verify whether or not the server's certificate has expired, as an attacker can who can MiTM the SSL communication can also prevent the CRL from being downloaded and communication will continue as normal.

# Beyond Certificate Checking

The scope of this project is currently to identify how to properly check the authenticity of the remote server. Future work includes identifying places where weaknesses exist within the TLS client implementation or configuration.

# Additional Resources

The paper "The most dangerous code in the world: validating SSL certificates in non-browser software" from the Stanford Crypto Group provides some valuable background information on certificate validation and was a motivation for this project.
