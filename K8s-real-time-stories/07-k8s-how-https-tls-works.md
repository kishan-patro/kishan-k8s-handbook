# How HTTPS Actually Works — A DevOps Perspective

HTTP sends your password in plain text.
Anyone intercepting the traffic reads everything.

HTTPS encrypts it using TLS.
Intercepted traffic looks like noise.

## The TLS Handshake in 5 Steps

1. **Client Hello:** Browser says "I want to connect. Here are the cipher suites I support."
2. **Server Hello:** Server responds with its SSL/TLS certificate. It proves "I am really example.com."
3. **Certificate Verification:** Browser checks — is this cert signed by a CA I trust? Is it expired? Does the domain match?
4. **Key Exchange:** Using asymmetric cryptography, both sides agree on a shared secret. The key itself is never transmitted directly.
5. **Encrypted Session:** All further communication is encrypted with the shared symmetric key. Fast and secure.

## What a DevOps Engineer Actually Does

- Obtain and install SSL/TLS certificates
- Configure the web server (Nginx, Apache, etc.) to use them
- Set up auto-renewal with Certbot (free via Let's Encrypt)
- Monitor cert expiry before it causes outages
- Enforce HTTPS redirects and HSTS headers

## The 3 AM Incident Nobody Wants

SSL cert expires because nobody set up auto-renewal.

**Fix:** Certbot + cron job. Runs twice daily, renews 30 days before expiry. Never think about it again.

```bash
# Certbot auto-renewal (typically added by Certbot itself)
0 0,12 * * * certbot renew --quiet --post-hook "systemctl reload nginx"
```

## The Thing Nobody Tells You

HTTPS means the connection is encrypted. It does not mean the site is trustworthy.
A phishing site can have a perfectly valid SSL certificate.

## Common SSL Issues in Production

- **Mixed content:** Page served over HTTPS but loads resources over HTTP — browser blocks them.
- **Certificate chain incomplete:** Server sends the leaf cert but not the intermediate CA — some clients reject it.
- **SNI mismatch:** Multiple domains on one IP, wrong cert served because the server doesn't support SNI properly.
- **TLS version too old:** Still allowing TLS 1.0/1.1 — security scanners flag it, modern browsers may refuse.
