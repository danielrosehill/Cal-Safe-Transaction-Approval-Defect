# Cal "safe transaction" (עסקה בטוחה) approval: a biometric-only gate that silently declines

A defect report on the out-of-band 3-D Secure approval flow in the **Cal** (Israel Credit Cards Ltd,
`com.onoapps.cal4u`) Android app, written for Cal's application-security and mobile development
teams. It is traced end to end with Android system logs and UI hierarchy dumps from a real purchase.

Personal details (cardholder, card number, merchant order, exact amounts, phone identifiers) are
redacted. The unredacted evidence exists and can be shared with Cal on request.

**Summary:** online purchases on a Cal card trigger an SMS asking the cardholder to approve in the
app. The approval screen exists but is **fingerprint-only, has no fallback, is visible for 2-3
seconds when the fingerprint is unavailable, and is recorded as a decline when it closes**. The app
then shows the transactions list with no message, so the cardholder sees "nothing to approve" while
the merchant says "declined in the Cal app". The same symptom was reported to Cal customer service
in July 2026 and was not resolved.

## Environment

| | |
|---|---|
| App | Cal, `com.onoapps.cal4u`, versionName 6.26.20 |
| Device | OnePlus Nord 3 5G (CPH2493), Android 16, in-display optical fingerprint sensor |
| Card / 3DS | Cal-issued Visa, challenge via Cardinal Commerce (`client.cardinaltrusted.com/centinelapi/V2/Cruise/StepUp`) |
| Merchant | a large international marketplace, USD purchase under $75 |
| Date | 2026-11-19 |
| Capture | wireless adb: `logcat -v time`, `uiautomator dump`; SMS read over KDE Connect |

## What the cardholder sees

1. The merchant's 3DS page (Cal-branded, inside Cardinal) says, in summary: "We're not done! We sent
   you a message; open the Cal app from it, identify yourself, approve, and come back to this page."
2. SMS from "Cal": a request to approve a transaction at the merchant, card ending ####, amount;
   "approval by identifying in the Cal app"; link `https://dl.cal-online.co.il/mobile?link=2042&ts=cal&tm=sms&tc=app2615`.
   A Cal push notification says the same.
3. Tapping the SMS or the push opens the app, asks for login, and ends on the **transactions list**.
   No approval request is visible.
4. The merchant page changes to "העסקה לא אושרה: העסקה נדחתה באפליקציית כאל" (transaction not
   approved: declined in the Cal app). The cardholder never pressed decline.

## What actually happens (from the logs)

Full sanitised timeline: [`evidence/logcat-timeline.txt`](evidence/logcat-timeline.txt).
Screen contents: [`evidence/screens.md`](evidence/screens.md).

**The deep link works.** `dl.`, `www.` and `api.cal-online.co.il` are verified App Links for
`com.onoapps.cal4u` (`pm get-app-links`). The tap starts `.ui.CALDeepLinksActivity`, then
`.ui.CALMainActivity`, then `.ui.safe_transaction.container.CALSafeTransactionActivity`.

**The approval screen exists** ("אישור עסקה בטוחה"): amount, date, merchant, card, "האם לאשר את
העסקה?", buttons "כן, לאשר" / "לא, תדחו את העסקה", note "צריך לאשר עם זיהוי ביומטרי נוסף",
primary button "המשך לזיהוי ביומטרי", close X.

Four attempts on the same order:

| # | Time | Condition | Approval screen | Fingerprint requested | Outcome |
|---|---|---|---|---|---|
| 1 | 18:40 | stale saved password; privacy screen protector over the sensor | never reached (stuck at login) | at login, failed | request expired / declined |
| 2 | 18:47 | logged in with a new password | **open 2 s** (18:47:13 to 18:47:15), one tap | no | **declined** |
| 3 | 18:49 | session already live, protector still on | **open 2.8 s** (18:49:57 to 18:50:00), error "biometric identification didn't work", two taps | **no `BiometricPrompt` was requested at all** | **declined** |
| - | 18:52 | link reopened, no live request | open 0.3 s, then transactions list, no message | n/a | n/a |
| 4 | 18:54 | protector removed, fingerprint working | stays open until approved | yes: 1st scan rejected, 2nd accepted (18:55:16) | **approved**, "סיימנו פה :)", merchant payment successful |

## Defects

1. **No fallback when biometrics are unavailable.** Approval requires a fingerprint. Passwords, an
   SMS OTP, or device credential (PIN/pattern via `BIOMETRIC_STRONG | DEVICE_CREDENTIAL`) are not
   offered. Screen protectors, wet or injured fingers, and phones without enrolled biometrics are
   common; for them the flow cannot succeed.
2. **Dismissal is recorded as a decline.** Closing the error, pressing back or the X ends the
   challenge as a refusal. A failed or impossible authentication is not a customer decision; it
   should leave the request pending until timeout, or offer a retry.
3. **Silent failure in the app.** After the screen closes the app shows the transactions list. It never
   says "you declined this transaction" or "approval failed", so the customer cannot tell what
   happened and reports "there was nothing to approve".
4. **Attempt 3 did not request a fingerprint before showing a biometric failure.** No
   `BiometricPrompt` request appears in the system log in that window; the app showed the error
   without trying. A likely trigger is the password change minutes earlier invalidating the app's
   biometric key, but the app gave no hint and no way to re-enrol from that screen.
5. **No pending-request entry point.** With no live request, the link flashes the screen for 0.3 s.
   The app has no place where a cardholder can see a pending approval, or a declined one, after the
   fact.
6. **Wrong prompt text on Android.** The fingerprint prompt reads "Touch ID עבור כאל ... יש לסרוק את
   האצבע בכפתור הבית" ("Touch ID for Cal ... scan your finger on the home button") on a phone with
   an in-display sensor and no home button.
7. **Short, unannounced window.** The SMS gives no time limit. Opening it a few minutes late lands
   on the transactions list with no "expired" message.

## Suggested fixes

- Offer device credential or an SMS OTP when biometrics are unavailable or fail.
- Treat cancel / error / back as "not answered", not "declined". Keep the request pending until the
  ACS timeout and let the cardholder return to it.
- Show the outcome in the app ("approved", "declined", "expired") and keep a list of recent 3DS
  requests with their state.
- Before showing the approval, check `BiometricManager.canAuthenticate()`. If biometrics are not
  usable, explain why and route to the fallback instead of failing.
- Platform-correct prompt text on Android.
- State the validity window in the SMS ("valid for N minutes").

## Reproduce

1. Cal card, Android phone where the fingerprint cannot be read (screen protector, or no fingerprint
   enrolled in Cal), start any online purchase that triggers 3DS.
2. Tap the Cal SMS, log in with the password.
3. Observe the approval screen appear briefly with a biometric error, close, and the transactions
   list. The merchant reports "declined in the Cal app".

## Contact

Filed by a Cal cardholder. Reply via the address this report was sent from.
