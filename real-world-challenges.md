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

Because `JSON.parse()` converts JSON numbers to JavaScript `Number`, the trailing digits can be changed due to precision loss.

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
Wrong transaction ID
```

---

### 🔍 Approach

1. Check the number of digits.
2. Compare it with `Number.MAX_SAFE_INTEGER`.
3. If it exceeds the safe limit, don't treat it as a `Number`.
4. Preserve it as a `String`.

---

### ✅ Solution

Send the transaction ID as a string:

```json
{
  "txnId": "1564378960978654367"
}
```

### 💡 Key Learning

> 🧠 **IDs are identifiers, not numbers.**
>
> If an ID can exceed **16 digits**, use a **string** to avoid precision loss.
