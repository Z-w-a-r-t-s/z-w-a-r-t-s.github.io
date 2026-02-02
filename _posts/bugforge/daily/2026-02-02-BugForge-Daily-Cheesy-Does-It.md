---
layout: post
title:  "BugForge - Daily - Cheesy Does It"
date:   2026-02-02 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw,insufficient-validation]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It
><br/><b>Vulnerabilities Covered:</b>
<br/>
Business Logic Flaw<br/>
Insufficient Input Validation
<br/>
<br/>
<b>Summary:</b>
<br/>
A business logic flaw in the tip functionality of the Cheesy Does It pizza ordering application allows users to submit negative tip percentages during the checkout process. The payment validation endpoint (`/api/payment/validate`) and order creation endpoint (`/api/orders`) both accept negative values for the tip parameter without proper server-side validation. By manipulating the tip percentage to a negative value, an attacker can effectively receive credits instead of paying, resulting in unauthorized discounts or even profit from placing orders. This vulnerability demonstrates insufficient input validation on financial parameters and a failure to enforce business rules that tips should only be zero or positive values.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
Begin by exploring the Cheesy Does It pizza ordering application to identify new or updated functionality. This version of the application includes a tip feature integrated into the checkout process, allowing customers to add a gratuity percentage to their order total. Analyze the complete ordering flow from menu selection through payment to understand how tips are calculated and processed.

---

### Step 2 - Analyzing the Checkout Flow
Examine the checkout process by capturing network requests during a normal order. The payment flow consists of three sequential API calls:

![Tip - UI](/images/bug-forge/daily/cheesy-does-it/broken-logic-tip-funcitonality/tips-ui.png)

1. `/api/payment/validate` - Validates the payment details and returns a payment token
2. `/api/payment/process` - Processes the transaction using the payment token
3. `/api/orders` - Creates the final order record

The tip percentage is submitted as part of the payment validation request, making it a potential target for manipulation.

---

### Step 3 - Testing Negative Tip Values
Since the tip functionality accepts a percentage value, test whether the application properly validates that tips must be positive. Intercept the `/api/payment/validate` request and modify the tip parameter to a negative value (e.g., -50).

![Negative Order - Request](/images/bug-forge/daily/cheesy-does-it/broken-logic-tip-funcitonality/validate-negative-request.png)

![Negative Order - Response](/images/bug-forge/daily/cheesy-does-it/broken-logic-tip-funcitonality/validate-negative-response.png)

The response returns with `valid: true`, confirming that the server accepts negative tip values without proper validation. This indicates a business logic flaw where the application fails to enforce that tips cannot be negative.

---

### Step 4 - Exploiting the Vulnerability
Complete the exploit by also setting a negative tip value in the `/api/orders` request. The server processes the order with the negative tip, effectively reducing the total amount or generating credits for the attacker.

![Negative Order - Request](/images/bug-forge/daily/cheesy-does-it/broken-logic-tip-funcitonality/negative-order-request.png)

![Flag](/images/bug-forge/daily/cheesy-does-it/broken-logic-tip-funcitonality/flag.png)

The flag is returned, confirming successful exploitation of the negative tip vulnerability.

---

### Impact
- Financial loss through unauthorized discounts by submitting negative tip values
- Potential for receiving credits or refunds on orders that should require payment
- Abuse of the payment system to place orders at reduced or negative costs
- Undermines the integrity of the tip feature intended to benefit service staff
- Scalable attack that could be automated to drain funds or generate fraudulent credits

---

### Vulnerability Classification
- **OWASP Top 10:** A04:2021 - Insecure Design (lack of business logic validation)
- **Vulnerability Type:** Business Logic Flaw / Insufficient Input Validation
- **Attack Surface:** Payment validation (`/api/payment/validate`) and order creation (`/api/orders`) endpoints
- **CWE:** CWE-840 - Business Logic Errors, CWE-20 - Improper Input Validation

---

### Root Cause
The backend fails to implement proper validation on the tip parameter, accepting negative values that violate fundamental business logic. The payment validation endpoint trusts client-supplied tip percentages without enforcing that they must be zero or positive. Additionally, the order creation endpoint does not perform secondary validation on financial parameters, allowing the negative tip to propagate through the entire transaction flow. The application lacks server-side business rules that would reject logically invalid tip values before processing.

---

### Remediation
- Implement strict server-side validation to ensure tip values are zero or positive
- Reject any payment requests containing negative values for tip, subtotal, or total amounts
- Add business logic validation that recalculates totals server-side rather than trusting client-supplied values
- Implement input type checking to ensure numeric financial fields contain valid positive numbers
- Add transaction monitoring to detect and alert on anomalous patterns such as negative tip submissions
- Perform comprehensive input validation at both the payment validation and order creation endpoints
- Log all financial transactions with full request details for audit and fraud detection purposes
- Consider implementing minimum and maximum bounds for tip percentages based on business requirements

---
