---
layout: post
title:  "BugForge - Daily - Cafe Club"
date:   2026-01-04 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw]
categories: [BugForge,daily,cafe-club]
---

# Daily - Cafe Club
><br/><b>Vulnerabilities Covered:</b>
<br/>
Business Logic Flaw
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge exploits a **business logic vulnerability** in the checkout process where the server fails to validate the `points_to_use` parameter against the user's actual point balance. By intercepting the checkout request and modifying the points value to exceed the available balance, an attacker can redeem more points than they possess, effectively obtaining products for free. The server blindly trusts the client-supplied value, resulting in negative point balances and unauthorized discounts.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis and Account Setup

Begin by exploring the CafeClub application. The site functions as an e-commerce platform selling coffee-related products. Launch Caido to monitor all HTTP requests during the testing process.

Create a new user account through the sign-up option on the login page and log in to access the application's features. This establishes a baseline for testing the shopping and checkout functionality.

---

### Step 2 - Feature Enumeration

After logging in, analyze the available application features:
- Product listing and search functionality
- Product detail pages with ID parameters
- Shopping cart functionality
- Points/rewards system
- Checkout process

Initial testing on product ID parameters for IDOR and SQL injection yielded no results. XSS testing on input fields showed proper encoding. Focus shifted to the API endpoints captured in Caido, particularly those under the `/api/` directory.

---

### Step 3 - Understanding the Points System

The cart page indicates that users earn 1 point for every 1 euro spent. Purchasing a product (e.g., Brazilian Santos Coffee Beans for 21.99 euros) credits the corresponding points to the account.

![Points UI](/images/bug-forge/daily/cafe-club/points-to-use/points-to-use-ui.png)

Complete a legitimate purchase to understand the normal checkout flow and observe how points are earned and displayed. After purchasing, the account balance reflects the earned points.

![Points to use Checkout](/images/bug-forge/daily/cafe-club/points-to-use/points-to-use-ui-checkout.png)

---

### Step 4 - Checkout Request Analysis

Add a product to the cart and proceed to checkout. Intercept the checkout request using Caido. The checkout process consists of three steps:

1. Review Order
2. Payment
3. Confirmation

Analyze the POST request to the checkout endpoint. Notice that the request includes a `points_to_use` parameter that specifies how many points the user wants to redeem for a discount.

![Request - Points to use](/images/bug-forge/daily/cafe-club/points-to-use/request-with-points-to-use.png)


---

### Step 5 - Business Logic Exploitation

1. Add a new product to the cart
2. Click on `Proceed to Checkout`
3. Click on `Continue to Payment`
4. Turn on Intercept (Button should show `Queuing`)
![Intercept On](/images/bug-forge/daily/cafe-club/points-to-use/caido-interceptor.png)
1. Enter `Cardholder Name` and click on `Place Order`
![Payment - UI](/images/bug-forge/daily/cafe-club/points-to-use/payment-ui.png)
1. Intercept the request and modify the `points_to_use` parameter to a value far exceeding the actual point balance (e.g., 10,000 points)
![Intercept Request](/images/bug-forge/daily/cafe-club/points-to-use/caido-intercept-request.png)

1. Forward the modified request

The server processes the request successfully despite the user not having sufficient points. The application fails to validate whether the user actually possesses the claimed points before applying the discount and returned the flag in the response.

![Flag](/images/bug-forge/daily/cafe-club/points-to-use/flag.png)

---


### Impact
- Unauthorized acquisition of products at reduced or zero cost
- Ability to redeem non-existent reward points for discounts
- Financial loss for the e-commerce platform
- Complete bypass of the rewards/loyalty program constraints
- Potential for automated exploitation at scale
- Negative point balances indicating systemic accounting failures
- Undermines the integrity of the entire loyalty points system

---

### Vulnerability Classification
- **OWASP Top 10:** A04:2021 - Insecure Design (missing server-side validation of business rules)
- **Vulnerability Type:** Business Logic Flaw / Insufficient Input Validation
- **Attack Surface:** E-commerce checkout API endpoint
- **CWE:** CWE-20 - Improper Input Validation

---

### Root Cause
The backend fails to validate the `points_to_use` parameter against the user's actual point balance before processing the checkout transaction. The application trusts the value sent from the client without verifying it server-side. This is a flawed assumption that users will only submit legitimate values through the intended UI, ignoring the possibility of request manipulation through tools like Caido.

The developer assumed that because the UI disables the points option when the balance is zero, users would never be able to use points they don't have. However, client-side controls provide no security guarantee when the server fails to enforce the same rules.

---

### Remediation
- Implement server-side validation to verify the user's actual point balance before applying any discount
- Reject checkout requests where `points_to_use` exceeds the user's available balance
- Never trust client-supplied values for security-critical calculations
- Use database lookups to retrieve the actual point balance during checkout processing
- Implement transaction logging to detect anomalous point usage patterns
- Add integrity checks to prevent negative point balances
- Apply the principle of "never trust user input" for all business-critical parameters
- Conduct security testing specifically targeting business logic flows

---
