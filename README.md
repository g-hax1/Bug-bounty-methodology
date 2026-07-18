### Bug bounty methodology

This is my own comprehensive bug bounty methodology for reconisance and explotation. Currently all of it except `Subdomain-discovery.md` is in first ai drafts

---

### File structure
The file structure should be as follows:
```
target.com/
│
├── recon/
│   │
│   ├── automatic-recon/
│   │   │
│   │   ├── notes.md
│   │   ├── domains.txt
│   │   ├── live.txt
│   │   ├── live_with_info.txt
│   │   ├── ffuf_vhosts.txt
│   │   ├── dorks.txt
│   │   ├── endpoints/
│   │   │  ├── endpoints.txt
│   │   │  └── params.txt
│   │   ├── javascript/
│   │   │  ├── js-files.txt
│   │   │  ├── endpoints.txt
│   │   │  └── secrets.txt
│   │   └── vulnerability_scans/
│   │       ├── subdomains_scan.md
│   │       ├── nmap_scan.txt
│   │       ├── subdomain_takeover_scan.txt
│   │       └── vulnerabilitys/
│   │           ├── xss.txt
│   │           ├── aws-keys.txt
│   │           ├── cors.txt
│   │           ├── debug-pages.txt
│   │           ├── debug-logic.txt
│   │           ├── firebase.txt
│   │           ├── idor.txt
│   │           ├── http-auth.txt
│   │           ├── img-traversal.txt
│   │           ├── interestingEXT.txt
│   │           ├── interestingparams.txt
│   │           ├── interestingsubs.txt
│   │           ├── lfi.txt
│   │           ├── rce.txt
│   │           ├── redirect.txt
│   │           ├── s3-buckets.txt
│   │           ├── sqli.txt
│   │           ├── ssrf.txt
│   │           ├── ssti.txt
│   │           ├── takeovers.txt
│   │           └── php-sinks.txt
│   │
│   └── manual-recon/
│       │
│       ├── notes.md
│       ├── interesting-endpoints.md
│       └── interesting_subdomain_name/
│           ├── all-endpoints.txt
│           ├── interesting-endpoints.md
│           ├── cloud_storage.md
│           ├── authentication.md
│           ├── authorization.md
│           ├── business-logic.md
│           ├── api-notes.md
│           ├── javascript-review.md
│           └── technology-notes.md
│
├── exploitation/
│   │
│   ├── xss.md
│   ├── sqli.md
│   ├── ssrf.md
│   ├── csrf.md
│   ├── idor.md
│   ├── xxe.md
│   ├── lfi.md
│   ├── rfi.md
│   ├── rce.md
│   ├── ssti.md
│   ├── open-redirect.md
│   ├── clickjacking.md
│   ├── cors.md
│   ├── authentication.md
│   ├── authorization.md
│   ├── file-upload.md
│   ├── path-traversal.md
│   ├── information-disclosure.md
│   ├── business-logic.md
│   ├── race-conditions.md
│   ├── graphql.md
│   ├── deserialization.md
│   ├── prototype-pollution.md
│   ├── request-smuggling.md
│   ├── web-cache-poisoning.md
│   ├── web-cache-deception.md
│   ├── host-header.md
│   ├── jwt.md
│   ├── oauth.md
│   ├── webhooks.md
│   ├── cloud-storage.md
│   └── reports/
│       ├── drafts/
│       └── submitted/
│
├── scope.md
├── out-of-scope.md
├── commands.md
└── README.md
```

---

### Useful resources
- https://github.com/danielmiessler/seclists for bruteforcing lists