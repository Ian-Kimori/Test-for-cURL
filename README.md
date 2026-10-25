# Test-for-cURL

On Linux connected to a corporate network, you need to verify three things: curl is installed, HTTPS requests work, and corporate proxy/TLS controls aren't blocking it.

Run these commands in your Linux terminal in order.

## 1. Verify curl is installed

```
curl --version
```

Expected output includes the curl version, supported protocols such as `http` and `https`, and TLS features.

Check its executable path:

```
command -v curl
```

Example: `/usr/bin/curl`

## 2. Test HTTPS connectivity

```
curl -I --connect-timeout 10 https://example.com
```

Interpret the result:

| Result                              | Meaning                                                  |
| ----------------------------------- | -------------------------------------------------------- |
| `HTTP/2 200` or `HTTP/1.1 200 OK`   | HTTPS request succeeded                                  |
| `HTTP/1.1 301` or `302`             | Server redirected the request; connectivity works        |
| `407 Proxy Authentication Required` | Corporate proxy requires authentication                  |
| `Could not resolve host`            | Possible DNS problem                                     |
| `Connection timed out`              | Possible firewall, routing, proxy, or connectivity issue |
| `SSL certificate problem`           | Certificate trust or TLS inspection issue                |

A `403` response can also mean the destination or corporate security policy denied access; it does not necessarily mean curl is broken.

## 3. Test DNS separately

```
getent hosts example.com
```

If this returns an IP address, DNS resolution is working for that hostname.

## 4. Check corporate proxy settings

```
env | grep -iE '^(http|https|all|no)_proxy='
```

If your organization uses a proxy, curl may need the approved proxy configuration. If the command returns nothing, it only means those proxy environment variables aren't set; the network might still use transparent proxying.

## 5. Check TLS certificate validation

```
curl -Iv --connect-timeout 10 https://example.com
```

Look for lines such as:

```
* SSL certificate verify ok.
```

If you get a certificate error, check whether your company requires an approved corporate CA certificate. Do not use `-k` or `--insecure` as a permanent workaround, since that disables certificate verification.

## 6. Run a quick diagnostic bundle

This prints the curl version, proxy configuration, DNS result, and HTTPS response:

```
echo "=== CURL ==="
curl --version | head -n 2

echo "=== CURL PATH ==="
command -v curl

echo "=== PROXY SETTINGS ==="
env | grep -iE '^(http|https|all|no)_proxy=' || echo "No proxy environment variables set"

echo "=== DNS ==="
getent hosts example.com

echo "=== HTTPS ==="
curl -sS -I -o /dev/null \
  --connect-timeout 10 \
  -w 'HTTP status: %{http_code}\nRemote IP: %{remote_ip}\nTLS verification result: %{ssl_verify_result}\n' \
  https://example.com
```

The final command returns an HTTP status code, the remote IP address, and a TLS verification result. A TLS result of `0` means certificate verification succeeded.

Corporate security note: Use a company-approved test destination and proxy configuration if your organization's policies restrict external connections.

If you paste the diagnostic output, I can help pinpoint whether the issue is curl itself, DNS, proxy configuration, firewall restrictions, or TLS inspection.
