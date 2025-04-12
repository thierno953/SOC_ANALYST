# Emotet Malware Analysis

- Post-Infection, Emotet malware initiated HTTP GET request to four URL domains. One of them responded with a DLL file. What is that filename ?

#### Stage 1: Initial Infection

- Filter Suspicious HTTP Requests

`(http.request or tls.handshake.type eq 1) and !(ssdp)`

- Enable Network Resolution

`Go to View > Name Resolution > Check both`

```sh
✓ Resolve Physical Addresses
✓ Resolve Network Addresses
```

- Extended Traffic Filter

`(http.request or http.response or tls.handshake.type eq 1) and !(ssdp)`

- Analyze Suspicious Domains

```sh
# hangarlastik.com   [Stream #121] GET
# padreescapes.com  [Stream #128] GET
# sarture.com       [Stream #131] GET
# seo.udaipurkart.com [Stream #142] GET (DLL response)
```

`Right-click > Follow > TCP Stream`

- Extract Malicious DLL

```sh
# seo.udaipurkart.com [Stream #1436] Port 80 → 49775
```

`Go to File > Export Objects > HTTP > seo.udaipurkart.com > Save All`
