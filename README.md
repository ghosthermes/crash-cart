# crash-cart

When production breaks and nobody knows why. 

Crash Cart is a first-response deployment diagnostic toolkit. It runs a rapid environmental verification to identify common causes of outages before engineers start digging into application code.

Most debugging tools assume you already know where to look. They expect you to query a specific log stream or attach a profiler to a running process. But what do you do when the container instantly exits? Or when the reverse proxy returns a cryptic 502 and the application logs are completely empty?

You stop guessing. 

I built this utility to automate the first five minutes of any incident response. Instead of manually testing ports and certificates while Slack is blowing up, you execute one command. 

The toolkit runs five explicit checks:
* Validates DNS records against public resolvers to catch propagation failures
* Inspects the TLS certificate chain for mismatches and silent expirations
* Tests raw port availability independent of the application layer
* Compares the active environment variables against a strict baseline manifest
* Audits critical file permissions for mounted storage volumes

It does not fix the problem. It isolates the variable.

### Example Output

```text
$ ./crash-cart --target api.production.internal

[OK] DNS routing clear
[FAIL] TLS handshake rejected
[OK] Port 443 open
[FAIL] Environment variables incomplete

DIAGNOSIS
Likely failure sources detected:
1. TLS certificate mismatch at the reverse proxy layer.
2. Missing ENV variables (STRIPE_SECRET, REDIS_HOST).
```

### Usage

Drop the script onto the affected server or container environment. You don't need root permissions for the basic network and environment diffing checks.

```bash
git clone https://github.com/ghosthermes/crash-cart.git
cd crash-cart
python3 main.py --env-baseline ./known_good.env
```

Is the outage caused by a bad application build? Maybe. Run this first to rule out the infrastructure. If Crash Cart returns all green, you can confidently tell the backend team to start combing through their application stack traces because the server environment itself is completely functional. 

If it flags a missing database credential instead, you just saved your team two hours of useless log hunting.
