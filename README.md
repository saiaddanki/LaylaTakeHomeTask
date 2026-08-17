# QA Assessment Report: Bellboy Hotel Search Application
## Part 1: Exploratory Testing

**Test Environment:** https://hoteller-theta.vercel.app/  
**Assessment Date:** August 17, 2026  
**Time Spent:** ~60 minutes  
**Tester:** QA Architect

---

## Executive Summary

**Overall Quality Assessment: HIGH RISK - Critical functional issues prevent core user journey completion**

The application's primary user flow (search → filter → select → book → confirm) contains multiple breaking points that would significantly impact user experience, conversion rates, and revenue. While the UI is polished, the underlying functionality has serious gaps in filter logic, state management, and payment error handling.

**Key Risk Areas:**
- Security vulnerability in authentication (weak password policy)
- Filter functionality unreliable across all tested scenarios
- Inconsistent feature availability between views (list vs. map)
- Payment failure UX leaves users stranded
- Missing transactional emails (cancellation)

---

## Test Scope & Approach

### Core Flow Tested
```
Search → View Results → Apply Filters → Select Hotel → View Details → Book → Payment → Confirmation
```

### Preconditions
- User account created and authenticated
- Test payment card: 4242 4242 4242 4242

### Test Data
- Primary search: "Hotels in Paris for September 2026"
- Budget filters: €100, €200
- Natural language queries with price constraints

---

## Findings by Severity

### 🔴 P0 - CRITICAL (Security/Compliance/Blocking)

#### BUG-001: Weak Password Policy - Security Vulnerability
**Severity:** P0 - Critical  
**Category:** Security  
**Affected:** Signup flow (Test & Production environments)

**Description:**  
Password validation only enforces minimum 6 characters with no complexity requirements. Users can create accounts with:
- 6 spaces (empty password)
- Simple strings like "123456" or "aaaaaa"
- No uppercase/lowercase/special character requirements

**Business Impact:**  
- Accounts vulnerable to brute force attacks
- Payment methods stored in accounts at risk
- Potential PCI-DSS compliance violation
- Bot/automated account creation trivial

**Recommendation:**  
Implement password policy requiring:
- Minimum 8 characters
- At least 1 uppercase, 1 lowercase, 1 number
- At least 1 special character
- Block common passwords (dictionary check)

---

#### BUG-002: Missing Cancellation Confirmation Email
**Severity:** P0 - Critical  
**Category:** Compliance/UX  
**Affected:** Booking cancellation flow

**Description:**  
When a user cancels a booking, no cancellation confirmation email is sent. Confirmation email exists for bookings, but not for cancellations.

**Business Impact:**  
- Legal/compliance risk (proof of cancellation required in many jurisdictions)
- Customer disputes without paper trail
- Support ticket volume increase
- Trust erosion

**Recommendation:**  
Implement automated cancellation email with:
- Cancellation reference number
- Refund details and timeline
- Original booking details
- Customer support contact

---

### 🟠 P1 - HIGH (Major Functionality Broken)

#### BUG-003: Date Range Interpretation Incorrect
**Severity:** P1 - High  
**Category:** Search/AI Interpretation  

**Description:**  
Search query "Hotels in Paris for September 2026" interprets as specific dates (September 1-2, 2026) instead of the entire month or allowing flexible date selection.

**Expected Behavior:**  
- Interpret "September 2026" as the full month
- OR prompt user to specify exact dates
- OR show availability calendar for the month

**Actual Behavior:**  
Arbitrarily selects September 1-2, limiting results and missing user intent.

**User Impact:**  
Users with flexible travel dates miss available options and may abandon to competitors.

---

#### BUG-004: Filter Logic Fundamentally Broken
**Severity:** P1 - High  
**Category:** Search/Filters  

**Description:**  
Multiple filter types fail to correctly constrain results:

| Filter Type | Issue |
|-------------|-------|
| **Budget (€100)** | Returns hotels >€100 even with "Must have" constraint |
| **Natural Language Price** | "under 200 euro" returns hotels >€200 |
| **City Filter** | Removing city drops to 9 results even when city already selected in search |
| **Free Breakfast** | Returns hotels without breakfast included |

**Evidence:**  

*Budget filter showing €100 limit with >€100 results:*
<img width="1728" height="1037" alt="Screenshot 2026-08-17 at 08 23 20-1" src="https://github.com/user-attachments/assets/43cf7b0d-9fdc-4d45-925b-ebff5af6e299" />


*Data with <100:*
<img width="1714" height="984" alt="Screenshot 2026-08-17 at 09 17 15" src="https://github.com/user-attachments/assets/701e754e-1c39-40f8-adb7-f345878755d6" />



*Additional filter evidence:*
<img width="1728" height="1037" alt="Screenshot 2026-08-17 at 08 23 20" src="https://github.com/user-attachments/assets/fa719b75-c5f4-43bf-b681-e0c9745baacd" />


*City filter issue - results drop to 9 when city filter removed:*
<img width="1160" height="1041" alt="Screenshot 2026-08-17 at 09 53 25" src="https://github.com/user-attachments/assets/461901a0-ac35-4601-b912-65d63b0d7fa5" />


*Contradictory breakfast information on same card:*
<img width="1719" height="1037" alt="Screenshot 2026-08-17 at 09 45 35" src="https://github.com/user-attachments/assets/64d57bb3-92fa-4ad7-8888-f2f6ffa30a68" />


**Root Cause Hypothesis:**  
- Filters may be "soft" preferences rather than hard constraints
- AI interpretation overriding explicit filter selections
- Backend query not properly applying WHERE clauses
- Data inconsistency between amenity fields

---

#### BUG-005: Feature Parity Gap Between List/Map Views
**Severity:** P1 - High  
**Category:** Feature Inconsistency  

**Description:**  
| Feature | List View | Map View |
|---------|-----------|----------|
| Book Button | ❌ Missing | ✅ Present |
| Filters | ✅ Functional | ❌ Non-functional |

**Evidence:**

*List view - no book option:*
<img width="1728" height="1036" alt="Screenshot 2026-08-17 at 10 31 11" src="https://github.com/user-attachments/assets/3b8cc5bc-49c2-41f3-9b90-38dff6bafbea" />


*Map view - book option present:*
<img width="1728" height="1040" alt="Screenshot 2026-08-17 at 10 31 20" src="https://github.com/user-attachments/assets/d58b6fc1-5328-48ac-bfeb-7782b2d3c099" />


*Filters non-functional in map view:*


https://github.com/user-attachments/assets/759c43a0-f7fc-42ff-ade3-9836eb8ec29f



**User Impact:**  
- List view users cannot book (conversion killer)
- Map view users cannot filter (usability killer)
- No clear indication of feature availability differences

---

#### BUG-006: Date/Guest Changes Don't Update Results
**Severity:** P1 - High  
**Category:** State Management  

**Description:**  
After initial search, changing:
- Check-in date
- Checkout date  
- Number of rooms
- Number of guests

...does NOT update the search results. Results remain from original search.

**Evidence:**



https://github.com/user-attachments/assets/3dc84c52-e711-4cfe-bfc8-269557bdad15




**User Impact:**  
Users adjusting travel plans see stale/incorrect availability and pricing.

---

#### BUG-007: Payment Retry Flow Broken
**Severity:** P1 - High  
**Category:** Payment/Checkout  

**Description:**  
After payment failure:
1. User cannot switch to different payment method (only card retry shown)
2. "Pay without link" option exists but is not discoverable
3. PayPal and other methods hidden after initial card attempt

**Evidence:**

*Limited retry options after payment failure:*
<img width="1723" height="1035" alt="Screenshot 2026-08-17 at 10 55 32" src="https://github.com/user-attachments/assets/60437659-fcb2-4a38-87e6-a547fb3e21fe" />


**User Impact:**  
- Abandoned bookings when card fails
- Users with insufficient funds on one card cannot use another
- Revenue loss at final conversion step

---

### 🟡 P2 - MEDIUM (Functionality Issues)

#### BUG-008: User Details Lost on Card Update
**Severity:** P2 - Medium  
**Category:** Form State Management  

**Description:**  
When user updates card information in payment form, all previously entered user details (name, email, phone, address) are cleared and must be re-entered.

**User Impact:**  
Frustration, increased form abandonment, time wasted.

---

#### BUG-009: Payment Failure Message Non-Actionable
**Severity:** P2 - Medium  
**Category:** Error Handling/UX  

**Description:**  
Payment failure shows generic message with only "Retry" or "Contact Support" options. No indication of:
- Why payment failed (declined, insufficient funds, network error, invalid card)
- What user should do differently
- Which field might have an error

**Evidence:**
<img width="1723" height="990" alt="Screenshot 2026-08-17 at 10 52 39" src="https://github.com/user-attachments/assets/242c8325-41a6-4ea7-9ff3-58facbb5483e" />


**Recommendation:**  
Display specific, actionable error messages based on payment processor response codes.

---

### 🟢 P3 - LOW (UX/Cosmetic)

#### BUG-010: Currency Change Doesn't Update Booking Price
**Severity:** P3 - Low  
**Category:** Localization  

**Description:**  
Changing currency selector (e.g., EUR → USD) on payment page doesn't convert/update the displayed booking price.

---

#### BUG-011: Contact Support Not Obvious
**Severity:** P3 - Low  
**Category:** UX  

**Description:**  
- Contact support is email-only but this isn't clearly communicated
- Chat option on contact page doesn't function

---

## Quality Assessment Summary

| Area | Status | Notes |
|------|--------|-------|
| **Authentication** | ⚠️ Critical | Password policy vulnerable |
| **Search** | ⚠️ Issues | Date interpretation problematic |
| **Filters** | ❌ Broken | Multiple filter types unreliable |
| **Hotel Listing** | ⚠️ Partial | Works but inconsistent between views |
| **Booking Flow** | ⚠️ Partial | Missing book button in list view |
| **Payment** | ⚠️ Issues | Poor error handling, retry flow broken |
| **Transactional Emails** | ❌ Incomplete | Missing cancellation email |
| **UI/UX Polish** | ✅ Good | Visually appealing interface |

---

## Recommendations

### Immediate (Before Production)
1. **Fix password validation** - Security critical
2. **Implement cancellation emails** - Compliance critical  
3. **Add book button to list view** - Conversion critical
4. **Fix filter logic** - Core functionality

### Short-term (Next Sprint)
1. Review and fix state management for date/guest changes
2. Improve payment error messages with actionable guidance
3. Unify feature parity between list and map views
4. Fix form state preservation on card update

### Investigation Needed
- Filter logic requires deep-dive with engineering (AI vs. hard filters)
- Data consistency audit for amenity information
- End-to-end payment flow testing with various failure scenarios

---

## What I Would Investigate Next (If More Time)

1. **Filter Logic Deep Dive**
   - Test with controlled data set where expected results are known
   - Trace API calls to understand if filters are applied server-side
   - Test "Must have" vs. "Nice to have" filter semantics

2. **Multi-Currency Flow**
   - Verify currency conversion accuracy
   - Test booking in different currencies end-to-end
   - Check if payment processor receives correct currency

3. **Concurrent Session Testing**
   - Same user logged in on multiple devices
   - Booking same hotel simultaneously
   - Session timeout handling

4. **Edge Cases**
   - Very long search queries
   - Special characters in guest names
   - Dates spanning year boundaries
   - Maximum room/guest limits

5. **Performance Testing**
   - Search response times under load
   - Filter application latency
   - Map view with 100+ results

---
NOTE:  I'm not testing for things like site accessibility, performance on different devices, or cross-browser compatibility in the interest of time.


Part 2: Investigation
Customer Support sends the following report:
“We’ve had a few users saying that sometimes they repeat the same hotel search
and get very different results. One user said it happened several times yesterday,
but we haven’t been able to reproduce it consistently.”


### Confirmed Defect

**Title:** Repeated Berlin searches can return results for unrelated cities
**Severity:** P1 - High
**Area:** Search result generation / search-state isolation

**Reproduction:**
1. Open two or more browser tabs, or repeat the test in separate browsers.
2. Search for `find hotels in berlin`.
3. Compare the generated result pages and their network responses.

**Observed:**
The issue is reproducible. The same Berlin query produces different results in separate tabs and browsers.
<img width="1702" height="1014" alt="Screenshot 2026-08-17 at 11 58 14" src="https://github.com/user-attachments/assets/7777fa33-17dc-471d-a782-e323380614f5" />
 <img width="1722" height="1037" alt="Screenshot 2026-08-17 at 11 58 23" src="https://github.com/user-attachments/assets/0dc428ae-e674-465d-9db5-5ed0cfb1884f" />
<img width="1728" height="1046" alt="Screenshot 2026-08-17 at 11 59 15" src="https://github.com/user-attachments/assets/d85c1d49-4027-4c18-98a8-fc97cde3275e" />


**Captured evidence:**
- The captured response for the Berlin search renders the title `City Hotels in Paris for September 2026` and describes a Paris stay from September 17-22, 2026.
- Its embedded hotel data includes Paris, London, and New York properties. Therefore, the wrong destination is already present in the response before the browser renders the hotel cards.
- The generated URLs use different opaque IDs, such as `01M07HJ8FEV9ZTYH3HD283R5TM` and `01M07HMMW4PNEZ244JHZGJ8DCP`, for different runs.

**Expected:**
Every successful search for Berlin should use the parsed Berlin search criteria and return only Berlin inventory, subject to any clearly communicated nearby-location expansion.

**Impact:**
Users can be shown and potentially book hotels in a different country than requested. This undermines search trust, drives abandonment, and creates a material risk of incorrect bookings and support contacts.

### Investigation Approach

1. **Preserve a reproducible evidence set.** For each run, record the exact query, timestamp, browser/incognito state, generated URL and opaque ID, request URL/body/headers, response body, response cache headers, and result-city counts. This lets engineering correlate an incorrect response with application and infrastructure logs.

2. **Establish the failure boundary.** The initial check is whether the search-create request already returns Paris/London/New York criteria, or whether a later request that resolves the opaque ID returns them. The captured response suggests the fault is upstream of UI rendering; a client-only rendering bug is therefore unlikely.

3. **Run controlled comparisons.** Repeat the identical Berlin query in a clean incognito session, an authenticated session, separate browsers, sequentially, and concurrently. Compare it with a Paris query. Vary one factor at a time: destination, dates, guests, account, locale, and request timing. A query-to-result matrix will reveal whether the failure is deterministic, tenant/session-specific, or race-dependent.

4. **Inspect cache behavior.** Check the response `Cache-Control`, `Age`, CDN cache status/key, and any search-service cache logs. The cache key must include the normalized destination, dates, guests, filters, locale/currency, and any user-specific inputs. Purge or disable the affected cache path temporarily if it can return one user's search configuration for another query.

5. **Trace the search lifecycle.** Add or retrieve a correlation ID spanning natural-language parsing, search persistence, generated-ID creation, inventory lookup, and page rendering. For each failing correlation ID, compare the input text, parsed destination, stored search record, cache key/hit status, and downstream inventory request. This identifies the first stage at which `Berlin` changes to another destination.

6. **Check shared-state and concurrency paths.** Review serverless/module-level mutable variables, async request handling, database upserts, ID generation, and retry logic. Parallel identical and mixed-city searches should never overwrite or retrieve another request's search criteria. Run a small concurrency test that submits Berlin and Paris searches simultaneously, then verifies every returned ID resolves to its original criteria.

### Current Hypotheses

- **Incorrect cache key or cached response replay:** plausible because different browser sessions receive inconsistent results, but it requires cache-header and cache-key evidence before being identified as the root cause.
- **Search-record lookup/ID association defect:** plausible because each run receives an opaque ID. The ID may map to the wrong persisted request, be reused, or be read before a write completes.
- **Shared mutable backend state or asynchronous race:** plausible if concurrent searches can overwrite a server-side "current search" object before the generated page is rendered.
- **Natural-language parser failure:** possible, but less likely if logs show `Berlin` was parsed correctly and a later stage substituted Paris. It remains a candidate until the parser input/output is captured.

The absence of relevant browser local storage, cookies, and visible client application state makes a purely client-persisted-state explanation less likely. It does not rule out server-side session state, CDN caching, database records, or a race condition.

### Information Needed From Engineering

| Information | Why it is needed |
|-------------|------------------|
| Search endpoint traces and correlation IDs for the failing timestamps | Locate the first component that produced the incorrect destination. |
| Request and response payloads for search creation and opaque-ID resolution | Determine whether the wrong state is created, persisted, or retrieved. |
| Cache/CDN configuration, cache keys, headers, hit/miss logs, and invalidation policy | Confirm or eliminate cached-response leakage. |
| Search-record schema, ID-generation approach, and read/write consistency model | Verify that IDs cannot collide, be reused, or be read before the intended record is committed. |
| Deployment version, region, instance/pod ID, and feature-flag state | Detect a bad instance, version-specific regression, or inconsistent configuration. |
| Natural-language parser input/output and fallback behavior | Verify that `Berlin` is parsed as Berlin and is not silently replaced by a default or previous destination. |

### What I Would Do Next

1. File the defect with the screenshots, captured response, timestamps, IDs, and the above reproduction steps; classify it as P1 and stop relying on the search result for booking validation.
2. Ask engineering to query the failing IDs and correlation IDs, then identify the first stage at which the destination is incorrect.
3. Add a temporary invariant at the search boundary: reject or alert when a result hotel's destination does not match the normalized requested destination. This is containment, not a substitute for fixing the underlying state or cache defect.
4. After the fix, automate regression coverage for sequential and concurrent searches across Berlin, Paris, London, and New York; assert that the parsed criteria, generated ID, cache key, and all returned hotel destinations remain consistent.
5. Release only after the issue cannot be reproduced across fresh sessions, browsers, and a concurrent mixed-city test, and monitoring shows no destination-mismatch alerts for an agreed observation period.







