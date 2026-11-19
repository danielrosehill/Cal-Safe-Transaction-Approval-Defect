# Screen contents (from `uiautomator dump`, redacted)

The Cal app sets `FLAG_SECURE`, so screenshots come out black. These are the text and view IDs
read from the accessibility hierarchy during attempt 4 (2026-11-19). Amounts, dates and card
digits are redacted.

## 1. Approval screen: `CALSafeTransactionActivity`

| View ID | Text |
|---|---|
| (title) | ביצוע עסקה בטוחה |
| `safeTransactionApproveTopTitle` | אישור עסקה בטוחה |
| `approveTransactionSubTitleText` | פרטי העסקה |
| `safeTransactionAmount` | $##.## |
| `safeTransactionDate` | ##/11/2026 |
| `safeTransactionMerchantName` | (merchant) |
| `safeTransactionBranchAndCard` | ויזה #### |
| `safeTransactionApproveQuestion` | האם לאשר את העסקה? |
| `disapproveAnswerIv` / `disapproveAnswerTv` | לא, תדחו את העסקה |
| `approveAnswerIv` / `approveAnswerTv` | כן, לאשר |
| `biometricsNote` | צריך לאשר עם זיהוי ביומטרי נוסף |
| `button` | המשך לזיהוי ביומטרי |
| close (content-desc) | סגירה |

## 2. Fingerprint prompt (system `BiometricPrompt`, app-supplied strings)

| Text |
|---|
| Touch ID עבור כאל |
| לצורך הזדהות למערכת נדרשת הזדהות בטביעת אצבע – יש לסרוק את האצבע בכפתור הבית |
| ביטול |

The device has an in-display sensor and no home button.

## 3. After approval

| Text |
|---|
| ביצוע עסקה בטוחה |
| סיימנו פה :) |
| כדי לסיים את הרכישה, צריך לחזור לאפליקציה או האתר בו בוצעה העסקה ולאשר את הקנייה |
| הבנתי, תודה |

## 4. Merchant 3DS page (Cardinal StepUp, Cal-branded)

- Pending: "לא סיימנו!", which asks the cardholder to enter the Cal app via the message sent to their
  phone, identify, approve with a click, and return to the page.
- After attempts 2 and 3: "העסקה לא אושרה", "העסקה נדחתה באפליקציית כאל", "אפשר תמיד לבצע אותה שוב", button "המשך".
