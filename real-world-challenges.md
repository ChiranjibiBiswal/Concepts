# Real-World-Challenges  

## 🚨 Large Transaction ID Precision Loss in JavaScript

### 🔴 Problem

We received a `transactionId` as an **unquoted numeric value**:

```json
{
  "txnId": 1564378960978654367
}
```

This is a **19-digit number**, which exceeds JavaScript's safe integer limit.

### ⚠️ JavaScript Number Limit

```js
Number.MAX_SAFE_INTEGER
// 9007199254740991
```

- ✅ Maximum safe integer: `9,007,199,254,740,991`
- 📏 Maximum safe size: **16 digits**
- ❌ **17+ digit integers can lose precision**

Because `JSON.parse()` converts JSON numbers to JavaScript `Number`, the value may already be corrupted before your application logic even runs.

---

### 🤔 Why?

JavaScript `Number` uses **64-bit IEEE-754 floating-point** representation and cannot precisely represent integers beyond `Number.MAX_SAFE_INTEGER`.

```text
19-digit transaction ID
        ↓
Exceeds MAX_SAFE_INTEGER
        ↓
Precision loss 💥
        ↓
Wrong transaction ID / wrong lookup / broken audit trail
```

This can silently change the value, which is dangerous for IDs used in validation, lookup, analytics, or payment flows.

---

### 🔍 Approach

1. Check the number of digits.
2. Compare it with `Number.MAX_SAFE_INTEGER`.
3. If it exceeds the safe limit, do not treat it as a `Number`.
4. Preserve it as a `String`.
5. If you need numeric operations later, convert only when safe and explicitly required.

---

### ✅ Safe Solution

Send the transaction ID as a string:

```json
{
  "txnId": "1564378960978654367"
}
```

This keeps the exact identifier intact during JSON parsing and transmission.

### 💡 When `BigInt` Helps

JavaScript has `BigInt` for large integers:

```js
const txnId = 1564378960978654367n;
```

However, `BigInt` is not always ideal for API contracts because JSON serialization, database fields, and many libraries do not handle it cleanly. For API payloads and IDs, `String` is usually the safest and most portable choice.

---

### 💬 Key Learning

> 🧠 **IDs are identifiers, not numbers.**
>
> If an ID can exceed **16 digits**, treat it as a **string** to avoid precision loss.

This is a common real-world bug in APIs, payment systems, and transaction records: values that look numeric are not always safe to store as JavaScript numbers.
