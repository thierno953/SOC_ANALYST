#### Stage 2: Suspicious Activity Investigation

- Identify Compromised Host

`(Statistics → Conversations → IPv4 → Apply as Filter → Selected → Filter on stream ID)`
`ip.stream eq 1`

- Malicious HTTP Requests

`http.request.method=="GET"`

```sh
(HyperText Transfer Protocol → Right-click on GET /dvwa/vulnerabilities/sqli → Copy → All Visible Items)

(HyperText Transfer Protocol → Right-click on Request URI: /dvwa/vulnerabilities/exec → Copy → Value)
```

- Executed Commands

`(Statistics → Conversations → TCP → Apply as Filter → Selected → Filter on stream ID)`

`ip.stream eq 7`

`(Right-click → Follow → TCP Stream)`
