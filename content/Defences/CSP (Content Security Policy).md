# What is CSP (content security policy)?
- Restricts which resources a page can load.
- Response needs header: `Content-Security-Policy`
# Testing CSP

## 1. Check Enforcement

**Active vs Report-Only**
- `Content-Security-Policy`: Enforced.
 - `Content-Security-Policy-Report-Only`: Only reports, doesn’t block.
- **Tip:** Report-Only headers can leak allowed sources.    
## 2. Weak Directives / Misconfigurations

- `default-src *` → All sources allowed, useless CSP.
- `script-src *` → Any script can run.
- `style-src *` → Inline CSS/XSS possible if `'unsafe-inline'` is allowed.
- `img-src *` → Potential data exfiltration via image loads.
- Missing `frame-ancestors` → Possible clickjacking.
- Missing `form-action` → Forms can post to attacker domains.
## 3. Inline Script/Style

- `unsafe-inline` → Inline scripts/styles execute.
    - If combined with `eval()` or `Function` → classic XSS vectors.
- Hash-based or nonce-based scripts:
    - Nonce reused improperly → attacker may inject scripts with valid nonce.
    - Hashes change on whitespace formatting → sometimes bypassable if outdated hash applied.
## 4. Script-src Weaknesses
- `strict-dynamic` absent → attacker cannot easily inject dynamically created scripts.
- Fallback to `'self'` or `*` → try hosting malicious JS on allowed domains.
- Mixed HTTP/HTTPS or `upgrade-insecure-requests` missing → man-in-the-middle injection possible.
## 5. External Domains

- Check `script-src`, `connect-src`, `img-src`, `frame-src` for:
    - Trusted CDNs that may host vulnerable JS.
    - Third-party scripts you can compromise or abuse.
- Subdomain takeovers → if `*.example.com` allowed.
## 6. Reporting & Info Leak
- `report-uri` / `report-to` → can reveal which CSP violations are happening.
- Can be abused for reconnaissance.
## 7. Sandbox & Object Restrictions
- Missing `sandbox` → iframe abuse possible.
- Missing `object-src 'none'` → Flash/Java/legacy injection possible.
## 8. Forms & Navigation
- Missing `form-action` → phishing form submission.
- Weak or missing `frame-ancestors` → clickjacking attacks feasible.
## 9. Strategy
1. **Check if CSP is enforced** (`Report-Only` vs real enforcement).
2. **Look for permissive sources**: `*`, `http:`, inline scripts/styles, unsafe-eval.
3. **Check nonces/hashes**: predictability, reuse, or weak implementation.
4. **Look for trusted CDNs / third-party hosts**: potential injection point.
5. **Frame & form weaknesses**: clickjacking or phishing opportunities.
6. **Bypass known CSP bypass vectors**:
    - JSONP endpoints, unsafe eval, data URLs, blob URLs, `strict-dynamic` missing.    
    - Inline event handlers if allowed (`onclick`, `onload`).
7. **Leverage reporting endpoints** for recon.
# Tools:
- https://csp-evaluator.withgoogle.com/
	- Check how permissive a CSP is
	- Scan for misconfiguration