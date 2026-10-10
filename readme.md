# 🍜 API Testing Lab: Auntie Som’s Noodle Stall

> **Objective:** Prove that Nephew Lek’s "it works on my machine" attitude is a recipe for disaster.

---

## 📖 The Backstory

Auntie Som runs a **legendary noodle stall**. Her Tom Yum is world-class, her customers are loyal, and her business is booming.  

Recently, her nephew **Lek** (who just finished a 2-week coding bootcamp) decided to "digitize" the business. He built a backend API to handle online orders. Auntie Som is thrilled, but the code… well, Lek says:  

> *"It’s fine lah Auntie! I tested it once with my own phone. No need for professional testing!"*

**You are the Last Line of Defense.**  
Before Auntie Som launches this app to 1,000+ hungry customers, you must audit the system, find the logic flaws, and automate the validation using **Bruno**.

---

## 🎯 Learning Objectives

By the end of this lab, you should be able to:
- 🖋️ Write **Bruno test scripts** for automated validation.
- 📦 Extract values from API responses using **Post-response scripts**.
- 🔐 Manage and use **Environment Variables**.
- 🧠 Verify **Business Rules**, not just HTTP status codes.
- 🚀 Catch regressions automatically before they hit production.

---

## 🛠️ Setup Instructions

### 1️⃣ Prepare the Project
- **Fork this repository** to your own GitHub account.
- **Clone it** locally to your machine.

### 2️⃣ Start the API Server
The backend is a lightweight Flask application.
- **Requirements:** Python 3.10+ installed.
- **Navigate** to the `auntie-som-noodle-api` folder.
- **Install dependencies:**  
  ```bash
  pip install flask
  ```
- **Run the server:**  
  ```bash
  python app.py
  ```
- The server will run at: `http://localhost:5000`

### 3️⃣ Setup Bruno
- Download and install [Bruno](https://www.usebruno.com).
- Create a **New Collection** named `Auntie Som Lab`.
- Create an **Environment** (e.g., `local`) and set a variable `baseUrl` to `http://localhost:5000`.

---

## 📑 API Reference

### 🔐 Authentication
`POST /auth/login` - Get an access token.
- **Body (JSON):**
  ```json
  {
    "username": "student",
    "password": "hungry"
  }
  ```
- **Response (200 OK):**
  ```json
  {
    "accessToken": "uuid-token-string",
    "expiresIn": 3600
  }
  ```

---

### 🍜 Menu
`GET /menu` - View available items and stock levels.
- **Response (200 OK):**
  ```json
  [
    { "id": 1, "name": "Tom Yum Noodles", "price": 50, "stock": 2 },
    { "id": 2, "name": "Fried Rice", "price": 45, "stock": 1 }
  ]
  ```

---

### 🛒 Orders
`POST /orders` - Place a new order.
- **Headers:** `Authorization: Bearer <token>`
- **Body (JSON):**
  ```json
  {
    "itemId": 1,
    "quantity": 1
  }
  ```
- **Response (200 OK):**
  ```json
  {
    "orderId": "ORD-abc123",
    "status": "created",
    "totalPrice": 45
  }
  ```

`GET /orders/<orderId>` - Retrieve order details.
- **Response (200 OK):**
  ```json
  {
    "orderId": "ORD-abc123",
    "status": "created",
    "totalPrice": 45
  }
  ```

---

## 🕵️ Your Mission: The Silent Auditor

Your task is to create a comprehensive Bruno collection that validates the entire workflow.  

⚠️ **The Catch:** Nephew Lek left several **logic bugs** in the system. Some endpoints might return `200 OK` even when the business logic is completely broken.

**Your collection must include:**
1.  **Auth Flow**: Automatically store the `accessToken` from login and use it for subsequent requests.
2.  **Order Validation**: Ensure orders can only be placed with valid auth, valid items, and available stock.
3.  **Data Integrity**: Check that the `totalPrice` calculated by the API matches your expectations.
4.  **Edge Cases**: What happens if you order more than what's in stock? What happens if you request a non-existent order?

> [!TIP]
> **Clicking "Send" is not testing.** Your tests must contain scripts (Assertions or Javascript) that **FAIL** when the system behaves incorrectly.

---

## 🧠 Rules of the Lab
- ❌ **Do NOT** modify the `app.py` backend code.
- ❌ **Do NOT** rely on manual checking.
- ✅ **Use environment variables** for the Base URL and Tokens.
- ✅ **Use Scripts/Assertions** for all validations.

---

## 📦 What to Submit

1.  **Push your Bruno collection folder** to your GitHub repository.
2.  **Update your README.md** (the one in your fork) with:
    - A summary of the bugs you discovered.
    - How your tests detect these bugs.
4.  **Send the GitHub link** to your instructor via Discord.

---
*Good luck. Auntie Som is counting on you!* 🍜🔥

---

## 🕵️ Audit Report: Bugs Found

The Bruno collection lives in [`auntie-som-noodle-api-test/`](auntie-som-noodle-api-test). Run it against a **freshly started** server (the data lives in memory, so stock changes carry over between runs):

```bash
cd auntie-som-noodle-api && python app.py                            # terminal 1
cd auntie-som-noodle-api-test && npx @usebruno/cli run --env local   # terminal 2
```

Result against Lek's code: **20/33 tests pass, 13 fail**. Every failure below is a real bug.

| # | Bug | Where in `app.py` | Test that catches it | Expected vs. actual |
|---|-----|-------------------|----------------------|---------------------|
| 1 | **Wrong total price**: every order is 5 baht too cheap | `total_price = (price * quantity) - 5` | `07 Create Valid Order`, `08 Get Created Order` | 50 expected, got 45 |
| 2 | **Can order more than the stock**: the code only checks `stock <= 0`, not `stock >= quantity` | `if item["stock"] <= 0` | `13 Order More Than Stock` | 400 expected, got 200 + an orderId |
| 3 | **Stock goes negative** (follows from bug 2) | `item["stock"] -= quantity` | `14 Stock Never Negative` | Fried Rice stock ≥ 0 expected, got -4 |
| 4 | **Quantity 0 accepted**: creates an order with a negative total (-5) | no quantity check | `10 Order Zero Quantity` | 400 expected, got 200 |
| 5 | **Negative quantity accepted**: *adds* stock back and creates an order with a negative price | no quantity check | `11 Order Negative Quantity` | 400 expected, got 200 |
| 6 | **Missing quantity crashes the server** (`None * int` → TypeError) | no input validation | `12 Order Missing Quantity` | 400 expected, got 500 |
| 7 | **Non-existent order returns 200** instead of 404 | `return jsonify({"error": ...}), 200` | `15 Get Non-existent Order` | 404 expected, got 200 |
| 8 | **Order details have no auth check**: anyone with an orderId can read it | `get_order` never calls `require_auth()` | `16 Get Order Without Token` | 401 expected, got 200 |

Also noticed but not tested automatically: tokens never expire even though login says `expiresIn: 3600` (`createdAt` is stored but never checked). Testing that would need a request to wait an hour.

### How the tests detect the bugs

- **Auth flow:** `01 Login Success` saves `accessToken` to the environment with `bru.setEnvVar`, and every order request sends it as `Bearer {{accessToken}}`. Requests 04 and 05 check that a missing or fake token gets a 401.
- **Environment variables:** `baseUrl`, `accessToken` and `orderId` live in `environments/local.bru`. No URL or token is hard-coded.
- **Data integrity:** `03 Get Menu` saves each item's price and stock. `07` checks `totalPrice === price × quantity` against that saved price. `08` checks that GET returns the same order as POST did. `09` checks that stock went down by exactly the ordered quantity.
- **Business rules, not just status codes:** the tests check prices, stock levels and that no `orderId` comes back on a rejected order, so a `200 OK` with the wrong data still fails.
- **Edge cases:** ordering more than the stock, quantity 0, negative and missing quantity, an unknown item, and an unknown order.
- **Order of requests:** the valid-order checks (07–09) run before the bad-quantity tests (10–13), because those tests corrupt the stock on the buggy server and would make the earlier checks unreliable.
