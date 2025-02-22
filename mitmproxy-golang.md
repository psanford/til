# Running mitmproxy with local Go binaries

To point a local go binary at a local mitmproxy set the following environment variables:
```
export HTTPS_PROXY=http://localhost:8080
export HTTP_PROXY=http://localhost:8080
export SSL_CERT_FILE=~/.mitmproxy/mitmproxy-ca.pem
```
