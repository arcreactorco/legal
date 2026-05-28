# Comms_Master SMS Opt-In Process — Charging Systems Inc.

**Effective date:** May 28, 2026
**Company:** Charging Systems Inc.
**Address:** 4000 Pleasant Grove, Norman, OK 73072, US
**Contact:** admin@chargingsystemsinc.com
**Phone number sending messages:** +1-855-552-7708

---

## Purpose

Charging Systems Inc. operates nine eBay seller accounts
(ChargingSystemsInc, Automotive-Starter-Alternator, Alt-Start-Shop,
EpartsGlobal-TX, Online-Starters-and-Alternators, Premier-Electrical-5,
RotatingElectricalNetwork, SuperStarters, Texas-Starter-Alternator)
selling aftermarket automotive electrical components.

We use an internal customer-service system called **Comms_Master** to
monitor buyer messages across our marketplaces and detect time-sensitive
cases (eBay Item Not Received cases, Amazon A-to-Z claims, negative
feedback threats, urgent damage claims). When such a case is detected,
the system sends a brief SMS alert to a pre-approved business staff
phone so the issue can be addressed quickly.

## Who receives these messages

SMS alerts from our Twilio number `+1-855-552-7708` are sent ONLY to:

- The business owners of Charging Systems Inc. (Dillon Toole, Pete Toole)
- Customer-service team members of Charging Systems Inc.

We **do not** send marketing or promotional messages from this number.
We **do not** send messages to end customers or members of the public.

## How recipients opt in (proof of consent)

Recipients are added to the alert list through a two-step verification
process that establishes affirmative consent:

1. **Administrator request.** The Comms_Master account administrator
   (Dillon Toole) requests that a given phone number receive alerts.
   The number is entered into Twilio's *Verified Caller IDs* interface
   inside our Twilio account dashboard.

2. **Recipient verification.** Twilio places a verification call or
   SMS to the recipient's number containing a six-digit code. The
   recipient — operating their own device — receives that code and
   enters it back into the Twilio Verified Caller IDs interface.

3. **Explicit consent recorded.** By entering the code themselves, the
   recipient provides explicit, demonstrable consent to receive alert
   SMS from `+1-855-552-7708` on behalf of Comms_Master. Twilio's
   Verified Caller IDs page maintains a permanent record of the
   verification timestamp.

Recipients who do not complete this two-step verification cannot receive
any messages from our system.

## Message frequency

5–20 alert messages per day across all 9 stores. Volume is bounded by
case frequency, not by promotional intent.

## How to opt out

Recipients can reply **STOP** to any message to be removed from the
alert list. Replying **HELP** returns contact information. Standard
Twilio STOP/HELP/UNSTOP keyword handling applies. **Message and data
rates may apply.**

## Sample messages

```
[Comms_Master] INR case opened on RotatingElectricalNetwork eBay store.
Buyer: missing_jordan. Priority 64. Respond within 4 business hours.
Reply STOP to opt out.
```

```
[Comms_Master] Negative feedback threat detected on Alt-Start-Shop.
Buyer: never_again. Action required within 1 business hour.
Reply STOP to opt out.
```

## Privacy

We do not share recipient phone numbers with any third party. Phone
numbers and message history are stored only in our internal Postgres
database and in Twilio's standard message logs. We retain no buyer
personally identifying information beyond what is required for the
customer-service workflow.

## Contact

For questions about Comms_Master SMS alerts:

**Email:** admin@chargingsystemsinc.com
**Charging Systems Inc.**
4000 Pleasant Grove
Norman, OK 73072
United States
