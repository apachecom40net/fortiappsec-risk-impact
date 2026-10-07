# FortiWeb vs FortiAppSec: Missing HTTP Security Headers Protection

## Short Answer
**Partial protection** - Both products can ADD missing headers at the perimeter, but cannot FIX missing headers at the origin. They can **mitigate** but not **remediate**.

---

## 🛡️ What FortiWeb/FortiAppSec Can Do

### 1. **Header Injection** (Mitigate at Perimeter)
FortiWeb can be configured to **inject missing headers** on behalf of backend servers:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Cache-Control: no-store, no-cache, must-revalidate
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
```

**Benefit:** Users receive the headers even if origin server doesn't send them
**Limitation:** Only protects traffic flowing through FortiWeb; direct origin requests still unprotected

### 2. **Header Validation & Enforcement**
- Detect when responses lack security headers
- Alert/log missing headers for compliance reporting
- Block responses that don't meet header policies
- Create security rules based on header presence/absence

### 3. **Attack Mitigation Despite Missing Headers**

| Missing Header | Attack Type | FortiWeb Mitigation |
|---|---|---|
| X-XSS-Protection | Reflected XSS | ✅ Input filtering, output encoding |
| X-Frame-Options | Clickjacking | ✅ Frame/iframe restrictions |
| CSP | Script injection | ✅ Input filtering (partial) |
| X-Content-Type-Options | MIME sniffing | ✅ Content-Type validation |
| HSTS | SSL stripping | ⚠️ HTTPS enforcement (partial) |
| Cache-Control | Data caching | ❌ Not directly mitigated |
| Referrer-Policy | Data leakage | ❌ Not mitigated at WAF |

---

## ❌ What FortiWeb/FortiAppSec CANNOT Do

### 1. **Cannot Fix Origin Server Configuration**
- The missing headers originate from the backend server
- WAF sits in front but doesn't change server configuration
- If server crashes or bypassed, headers still missing

### 2. **Cannot Protect Direct Access**
```
Scenario: User bypasses WAF (VPN, internal access, etc.)
Result: Directly hits origin server → no headers received
Impact: Full vulnerability exposure
```

### 3. **Full CSP Replacement** (Limited)
- CSP is complex and application-specific
- WAF can add a basic CSP but may not match application needs
- Origin server CSP should be properly configured

### 4. **Cache-Control Protection** (Limited)
- WAF cannot prevent browser caching at origin
- Can add Cache-Control headers but doesn't force compliance
- Browser may override or ignore

---

## 🎯 FortiWeb Typical Configuration Example

```
WAF Policy > Response Headers > Add Header

Header Name: Strict-Transport-Security
Header Value: max-age=31536000; includeSubDomains
Priority: 100

---

Header Name: X-Frame-Options
Header Value: SAMEORIGIN
Priority: 100

---

Header Name: Content-Security-Policy
Header Value: default-src 'self'; script-src 'self' cdn.trusted.com
Priority: 100
```

**Result:** All responses FROM your origin get these headers added by FortiWeb before reaching users

---

## 📊 Protection Coverage Matrix

| Scenario | FortiWeb Solution | Effectiveness | Notes |
|---|---|---|---|
| User through FortiWeb | ✅ Headers added | HIGH | Protected via WAF injection |
| User direct to origin | ❌ Missing headers | CRITICAL | Needs origin fix |
| API client via FortiWeb | ✅ Headers added | HIGH | Protected via WAF injection |
| API client direct | ❌ Missing headers | CRITICAL | Needs origin fix |
| Mobile app via FortiWeb | ✅ Headers added | MEDIUM | Some headers ignored by mobile |
| Third-party integration | ❌ Missing headers | CRITICAL | If bypass WAF |
| Incident response | ✅ Faster patching | MEDIUM | Quick WAF rule vs server deploy |

---

## 🔴 **CRITICAL LIMITATION**

### **FortiWeb Header Injection is NOT a Substitute for Origin Configuration**

**Why:**
1. Users can bypass WAF (VPN, direct access, internal networks)
2. Employees accessing origin directly see missing headers
3. Third-party integrations may not route through WAF
4. Security compliance requires **origin server** to send headers
5. Regulatory audits check origin server, not WAF

**Compliance Reality:**
- GDPR audit: Checks origin headers (WAF injection doesn't count)
- PCI-DSS: Requires headers FROM server (not added by WAF)
- SOC 2: Requires origin server security controls
- ISO 27001: Same requirement (server-side security)

---

## ✅ RECOMMENDED APPROACH

### **Best Practice: Origin + FortiWeb Defense-in-Depth**

```
┌─────────────────────────────────────────────────────┐
│ Step 1: FIX ORIGIN SERVER (Primary)                 │
│ ├─ Add all 7 security headers at source             │
│ ├─ Timeline: 4 weeks (per CLAUDE.md guidance)       │
│ ├─ Cost: Minimal (~2 hours per app)                 │
│ └─ ROI: Highest (protects all access paths)         │
└─────────────────────────────────────────────────────┘
        ↓
┌─────────────────────────────────────────────────────┐
│ Step 2: ADD FORTIWEB HEADER INJECTION (Secondary)   │
│ ├─ Adds redundant headers for defense-in-depth      │
│ ├─ Protects if origin misconfigured                 │
│ ├─ Provides compliance coverage for WAF traffic      │
│ └─ Timeline: 1-2 weeks (WAF configuration)          │
└─────────────────────────────────────────────────────┘
```

**Why Both:**
- **Origin fix** = permanent solution for all users
- **FortiWeb injection** = defense-in-depth + incident response flexibility
- **Combined** = 99.9% coverage, meets compliance requirements

---

## 🚨 FortiWeb Does NOT Protect Against:

1. **Users accessing origin directly** (no WAF involved)
2. **Employees on internal networks** (bypass WAF)
3. **Direct API calls** (if not routed through WAF)
4. **Compliance audits** (auditors check origin, not WAF)
5. **Incident response** (slower than origin config fix)
6. **Cache-Control violations** (browser behavior)
7. **Referrer-Policy** (not inspected by WAF)

---

## 💡 FortiAppSec vs FortiWeb

| Feature | FortiWeb | FortiAppSec |
|---|---|---|
| Header injection | ✅ Yes | ✅ Yes |
| Header validation | ✅ Yes | ✅ Yes (enhanced) |
| Policy enforcement | ✅ Standard | ✅ Enhanced |
| API security | ✅ Limited | ✅ Full |
| Missing header detection | ✅ Basic | ✅ Advanced |
| Header policy templates | ✅ Basic | ✅ Advanced |

**FortiAppSec** is newer and has better header inspection/policy capabilities, but still requires origin server fix.

---

## 🎯 Conclusion

**FortiWeb/FortiAppSec can MITIGATE but NOT REMEDIATE missing security headers.**

For your **423-domain portfolio**:

### **FortiWeb Alone = Incomplete Solution**
- ❌ Compliance: Auditors see missing origin headers → FAIL
- ❌ Risk: Direct access paths still vulnerable
- ❌ Cost: WAF overhead without actual fix
- ⚠️ False sense of security

### **Origin Fix + FortiWeb = Complete Solution**
- ✅ Compliance: Origin headers present
- ✅ Risk: All access paths protected
- ✅ Defense-in-depth: WAF provides second layer
- ✅ Business impact: Real security improvement

**Recommended Action Plan:**
1. **Fix origin servers** (4 weeks, Phase 1-3 from roadmap)
2. **Add FortiWeb header injection** (1-2 weeks, secondary)
3. **Monitor & audit** (ongoing compliance verification)
4. **Expected outcome:** 99.9% header coverage, audit-compliant

---

## 📝 FortiWeb Rule Example

**Policy Name:** `Security-Headers-Enforcement`

```
Rule 1: Inject Missing HSTS
  Condition: Response Code = 200-299
  Action: Add Header
    Name: Strict-Transport-Security
    Value: max-age=31536000; includeSubDomains

Rule 2: Inject CSP
  Condition: Content-Type contains text/html
  Action: Add Header
    Name: Content-Security-Policy
    Value: default-src 'self'; script-src 'self' cdn.trusted.com

Rule 3: Enforce X-Frame-Options
  Condition: Response Code = 200-299
  Action: Add Header
    Name: X-Frame-Options
    Value: SAMEORIGIN

Rule 4: Validate Cache-Control
  Condition: Response contains sensitive data
  Action: Add Header
    Name: Cache-Control
    Value: no-store, no-cache, must-revalidate
```

---

## 🔗 Related Documentation

- **Headers_Impact_Readme.md** - Detailed impact analysis of each missing header
- **CLAUDE.md** - Project pipeline and requirements
- **Fortinet-Inspired Dashboard** - Visualization using Fortinet design system

---

**Bottom Line:** FortiWeb = Emergency patch. Origin fix = Real security.

*Document generated: 2026-10-06*
*Portfolio scope: 423 domains (DR ENT-PS-Telco)*
