# check-certificate

This is a Nagios/Icinga plugin to check a remote server's SSL/TLS certificate.
The script performs an SSL handshake using Python's `ssl` module (which uses
OpenSSL internally) and tests whether the server certificate is close to its
expiry date.

Any SSL-protected endpoint works, the check is not limited to HTTPS.

## Prerequisites

The script requires a Python interpreter. It has been tested on Python 2.7,
3.4, and 3.5 but earlier versions of Python 3 may also work. The CA's root
certificate has to be installed in your system's certificate store (rule of
thumb: if curl works, Python should be happy, too).

## Usage

Using the script is pretty simple:

```
$ ./check_cert -h
usage: check_cert [-h] -H HOST [-p PORT] [-w N]

optional arguments:
  -h, --help            show this help message and exit
  -H HOST, --host HOST  name of the host to connect to
  -p PORT, --port PORT  port to connect to (default: 443)
  -w N, --warn-days N   warn if cert expires in less than N days (default: 15)
$
```
Example:

```
$ ./check_cert -H google.com
OK - certificate will expire in 69 days
$
```

If the certificate expires sooner than you'd like, a warning is generated:

```
$ ./check_cert -H google.com -w 100
WARNING - certificate will expire in 69 days
$
```

If the certificate already has expired or validation failed for some other
reason like a missing root certificate or a host name mismatch, the check
returns `CRITICAL`. Connection errors and everything else yield an `UNKNOWN`
result. All exit codes following Nagios plugin conventions.

For test scenarios with broken SSL setups see Google's examples on
[badssl.com](https://badssl.com).

## Limitations

This plugin does *not* check whether
   * the root or any intermediate certificate in the chain expires soon
   * any certificate in the chain has been revoked via CRLs or OCSP
   
Client certificates are not supported at this point.
