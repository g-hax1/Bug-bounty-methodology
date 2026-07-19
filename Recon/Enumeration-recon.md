# Wide subdomain enumeration

### Key concepts

The goal of wide subdomain enumeration is to discover as much authorised attack surface as possible, especially hidden development, staging, API, and administrative subdomains. Keep raw results, but use one canonical file for each type of result so that later stages do not miss earlier discoveries.

---

### Before starting

1. Read the program rules and create the target directory structure described in the main `README.md`.
    - Record in-scope domains, excluded domains, allowed testing types, rate limits, and the required contact or user-agent header in `scope.md`, `out-of-scope.md`, and `recon/automatic-recon/notes.md`.
    - Run the commands below from `target.com/recon/automatic-recon/` and replace `target.com` with the authorised root domain.
    - Do not scan third-party hosts, cloud storage, or non-production systems unless they are explicitly in scope.

### Steps for enumerating subdomains and their attack surface

1. Run `subfinder` to find subdomains from passive sources.
    Commands:
    - `subfinder -d target.com -all -silent | sort -u > domains-subfinder.txt`

2. Run `dnsx` to brute-force additional subdomains, then merge all domain sources into the canonical `domains.txt` file.
    Commands:
    - `dnsx -silent -d target.com -w "/home/g/Hacking/hacking-map/Bug bounty methodology/Recon/wordlists/namelist.txt" > domains-dnsx.txt`
    - `cat domains-subfinder.txt domains-dnsx.txt domains-manual.txt 2>/dev/null | sed 's/\.$//' | sort -u > domains.txt.tmp && mv domains.txt.tmp domains.txt`
    - Add reviewed results from certificate transparency, search engines, or other manual sources to `domains-manual.txt` before merging them.

3. Enumerate virtual hosts.
    Determine the default response size or word count first, then filter that baseline to reduce wildcard and default-host false positives.
    Commands:
    - `ffuf -w "/home/g/Hacking/hacking-map/Bug bounty methodology/Recon/wordlists/subdomains-top1million-110000.txt" -u "https://target.com/" -H "Host: FUZZ.target.com" -ac -rate 20 -of json -o ffuf-vhosts.json`
    - Review `ffuf-vhosts.json`, validate candidate hostnames with DNS or `httpx`, then add confirmed names to `domains-manual.txt` and merge them into `domains.txt`.

4. Use search engines and public sources to find relevant company information. Save the query, date, source URL, and relevant result in `dorks.txt`; do not treat search snippets as confirmed findings.
    Tools:
    - https://pentest-tools.com/information-gathering/google-hacking for example search operators.
    - https://ahrefs.com/blog/google-advanced-search-operators/ for additional search operators.

5. Resolve and probe the complete domain list to identify live web services.
    Commands:
    - `httpx -l domains.txt -silent -sc -ip -tech-detect -follow-redirects > live-with-info.txt`
    - `httpx -l domains.txt -silent -follow-redirects | sort -u > live.txt`
    - `sed -E 's#https?://([^/:]+).*#\1#' live.txt | sort -u > live-hosts.txt`
    - Keep `domains.txt` as all discovered in-scope names and `live.txt` as the canonical list of reachable web URLs.

6. Build an automatic technology inventory for every live web service.
    This creates a broad view of delivery infrastructure, server software, frameworks, client-side libraries, API protocols, identity providers, storage providers, and security controls. Treat automated fingerprints as leads and verify important ones during manual review.
    Commands:
    - `mkdir -p technology`
    - `httpx -l live.txt -silent -json -sc -title -tech-detect -web-server -ip -cdn -asn -follow-redirects > technology/httpx.jsonl`
    - `jq -r '[.url, ((.status_code // "") | tostring), (.title // ""), (.webserver // ""), ((.tech // []) | join(",")), (.cdn_name // ""), ((.asn // "") | tostring)] | @tsv' technology/httpx.jsonl > technology/technology-inventory.tsv`
    - `whatweb --input-file live.txt --no-errors -a 1 > technology/whatweb.txt` use as a second, low-impact fingerprinting source if WhatWeb is installed.
    - `grep -RniE 'graphql|graphiql|swagger|openapi|/api/|socket\.io|wss://|oauth|openid|saml|stripe|paypal|s3|firebase|cloudfront|vercel|netlify' technology/ > technology/technology-leads.txt`
    - Review `technology/technology-inventory.tsv` and `technology/technology-leads.txt`; record promising hosts and the evidence in `notes.md` before moving them to manual recon.

7. Crawl live web applications and merge all discovered URLs into the canonical endpoint file.
    Commands:
    - `katana -list live.txt -d 3 -kf all -js-crawl -silent > endpoints/endpoints-katana.txt`
    - `katana -list live.txt -d 3 -kf all -js-crawl -headless -silent > endpoints/endpoints-headless.txt` use this where normal crawling misses rendered content.
    - `cat endpoints/endpoints-katana.txt endpoints/endpoints-headless.txt | sort -u > endpoints/endpoints.txt.tmp && mv endpoints/endpoints.txt.tmp endpoints/endpoints.txt`
    - Use only the programme-approved contact header and rate limit; do not copy a header value from another programme.

8. Run the JavaScript reconnaissance methodology.
    - Change into `javascript/` and use `../endpoints/endpoints.txt` as the endpoint input for `Javascript-recon.md`.
    - Merge confirmed JavaScript and source-map endpoints back into `../endpoints/endpoints.txt` with a temporary file, as shown in that guide.
    - Merge confirmed parameter-bearing URLs into `../endpoints/params-urls.txt` and record API, authentication, and client-side context for deep manual review.

9. Build a canonical parameter inventory without overwriting earlier results.
    Commands:
    - `{ grep -E '\?' endpoints/endpoints.txt; gau --subs target.com | grep '='; } | sort -u > endpoints/params-urls.txt.tmp && mv endpoints/params-urls.txt.tmp endpoints/params-urls.txt`
    - `grep -oE '[?&][^=&]+' endpoints/params-urls.txt | cut -c2- | sort -u > endpoints/params.txt`
    - Review URLs before testing: archived URLs and JavaScript findings can require authentication or refer to retired functionality.

10. Scan non-web services only when service scanning is permitted by the programme.
    Commands:
    - `nmap -iL live-hosts.txt -sV --open -T3 -oN vulnerability_scans/nmap-scan.txt`
    - Use default scripts (`-sC`) only where the programme explicitly permits them, because some scripts are intrusive.

11. Run only the automated checks allowed by scope, and keep raw results separate from verified findings.
    Commands:
    - `subzy run --targets live.txt > vulnerability_scans/subdomain-takeover-scan.txt`
    - `gau --subs target.com | gf <installed-pattern-name> > vulnerability_scans/<pattern-name>-urls.txt` use only installed, relevant `gf` patterns.
    - Save each scanner's raw output in `vulnerability_scans/`. Add a result to `vulnerability_scans/vulnerabilitys/` only after manual validation.

12. Identify high-interest subdomains and prepare them for deep review.
    - Prioritise authentication and administration hosts, APIs, GraphQL and WebSocket endpoints, upload and download flows, exposed cloud storage, unusual services, and explicitly in-scope non-production hosts.
    - Create `manual-recon/<interesting_subdomain_name>/` using the structure in the main `README.md` and record why the host was selected in its `notes.md` or `interesting-endpoints.md`.

---

### Deep subdomain attack surface enumeration

This phase examines one authorised subdomain in detail and preserves its context in `manual-recon/<interesting_subdomain_name>/`.

1. Gather the wide-scan results for the chosen subdomain into its manual-recon notes.
    Commands:
    - `grep -F 'sub.target.com' endpoints/endpoints.txt`
    - `grep -F 'sub.target.com' endpoints/params-urls.txt`
    - `grep -F 'sub.target.com' live-with-info.txt`
    - `grep -F 'sub.target.com' domains-manual.txt`
    - `grep -F 'sub.target.com' dorks.txt`
    - `grep -F 'sub.target.com' vulnerability_scans/nmap-scan.txt`
    - `grep -F 'sub.target.com' vulnerability_scans/subdomain-takeover-scan.txt`
    - `grep -F 'sub.target.com' technology/technology-inventory.tsv`
    - `grep -F 'sub.target.com' technology/whatweb.txt`
    - Copy relevant results into `technology-notes.md`, `api-notes.md`, `javascript-review.md`, and `interesting-endpoints.md` rather than leaving them only in terminal output.

2. Build a thorough manual technology profile for the selected host.
    The automatic inventory is only a starting point. Confirm the technologies that affect attack surface and record the evidence, version where exposed, confidence, and date in `technology-notes.md`.
    Commands:
    - `httpx -u https://sub.target.com -title -tech-detect -status-code -follow-redirects -web-server -ip -cdn -asn > technology-httpx.txt`
    - `curl -sS -I -L https://sub.target.com > response-headers.txt` record server, framework, cache, CDN, security, and identity-provider headers.
    - `whatweb -a 3 --no-errors https://sub.target.com > whatweb.txt` use where WhatWeb is installed; compare this result with `technology-httpx.txt` instead of trusting either tool alone.
    - `curl -sS https://sub.target.com/robots.txt > robots.txt` and `curl -sS https://sub.target.com/sitemap.xml > sitemap.xml` to identify documented paths and application components.
    - Use the Wappalyzer extension and browser developer tools to inspect loaded scripts, cookies, local storage, service workers, network requests, API protocols, WebSockets, and third-party domains.
    - Review newly found JavaScript and source maps using `Javascript-recon.md`; package names, build paths, and API clients often identify the framework and backend services.
    - If TLS, DNS, port, or certificate details are relevant to the host, copy the evidence from `live-with-info.txt`, `technology/`, and permitted service scans into `technology-notes.md`.
    Record at least: hostname and final URL; CDN/WAF and hosting clues; server and framework; client-side framework and major libraries; API type and documentation; authentication/identity provider; data, storage, analytics, and payment providers; security headers; relevant cookies; and confidence/source for every important fingerprint.

3. Translate the technology profile into focused manual-recon tasks.
    - Put login, sessions, MFA, OAuth/OIDC, SAML, and token observations in `authentication.md`.
    - Put roles, organisation boundaries, tenant identifiers, and object ownership observations in `authorization.md`.
    - Put API methods, schemas, GraphQL, WebSockets, and request context in `api-notes.md`.
    - Put client-side libraries, browser storage, service workers, source-map findings, and DOM data flows in `javascript-review.md`.
    - Put user journeys, feature flags, integrations, payments, invitations, and state-changing actions in `business-logic.md`.

4. Crawl the host again with its known context and run the JavaScript methodology on newly discovered files.
    Commands:
    - `httpx -u https://sub.target.com -title -tech-detect -status-code -follow-redirects -web-server -ip -cdn -asn > technology-httpx.txt`
    - `katana -u https://sub.target.com -d 5 -headless -js-crawl -jsluice -kf all -silent > all-endpoints.txt`
    - Run the Burp Suite crawler using authorised test accounts where applicable and save exported URLs to `all-endpoints.txt`.
    - Merge unique entries from `all-endpoints.txt` into `interesting-endpoints.md` and run `Javascript-recon.md` for newly discovered JavaScript files. Copy high-value API, authentication, and client-side findings to `api-notes.md`, `authentication.md`, and `javascript-review.md`.

5. Review cloud storage.
    Commands:
    - `cd /home/g/Hacking/tools/cloud_enum && uv run python cloud_enum.py -k company_name`
    - `aws s3 ls s3://bucket-name --no-sign-request` list only a confirmed, in-scope bucket.
    - `aws s3 cp s3://bucket-name/object-name ./downloaded-object --no-sign-request` download only an authorised object to a local review directory.
    - Record confirmed buckets, objects, permissions, and scope evidence in `cloud_storage.md`.

6. Perform additional public-information research for the selected host.
    - Record the query, date, source URL, and relevant result in the host's `notes.md`.
    - Confirm that any discovered host, bucket, or third-party service is in scope before interacting with it.

7. Enumerate endpoint and API parameters only after baselining the response and confirming the method is safe.
    Commands:
    - `ffuf -w "/home/g/Hacking/hacking-map/Bug bounty methodology/Recon/wordlists/raft-large-words.txt" -u "https://sub.target.com/page?FUZZ=test" -ac -rate 20 -of json -o parameter-name-fuzz.json`
    - `ffuf -w "/home/g/Hacking/hacking-map/Bug bounty methodology/Recon/wordlists/large.txt" -u "https://sub.target.com/api" -X POST -H "Content-Type: application/json" -d '{"FUZZ":"test"}' -ac -rate 20 -of json -o api-parameter-fuzz.json` run only against an authorised, non-state-changing request.
    - Record the baseline, authentication context, rate limit, endpoint, method, and parameter in `api-notes.md`. Manually validate response differences before treating them as findings.
