# SMS Messaging Program & Consent — Charging Systems Inc.

**Effective date:** June 23, 2026
**Company:** Charging Systems Inc.
**Address:** 4000 Pleasant Grove, Norman, OK 73072, US
**Contact:** admin@chargingsystemsinc.com
**Messaging number (toll-free):** +1-855-552-7708
**Opt-in form:** https://chargingsystemsinc.com/sms

---

> **Status:** In development, pending approval. This rewrite replaces the prior
> internal-staff-alert consent description, which Twilio Toll-Free Verification
> **rejected on 2026-06-05** (error `30513` — the prior `opt_in_type=WEB_FORM`
> pointed at a Markdown doc, and the "Verified Caller IDs" phone-code mechanism
> is caller-ID verification, not messaging consent). This document describes a
> real customer-facing opt-in via a web form with an unchecked consent checkbox.

## Program overview

Charging Systems Inc. (brands: Charging Systems / Arc Reactor, and the
EpartsGlobal eBay store) sends two categories of SMS/MMS message to customers
who have **explicitly opted in**:

1. **Operational route notifications** — for delivery-route customers, brief
   service texts such as "Do you need a stop today?" and order/route status.
2. **Promotional messages** — occasional offers, new-product announcements, and
   promotional flyers sent as MMS images.

Consent to receive messages is **never a condition of any purchase**.

## Who receives these messages

Only individuals who complete the opt-in described below. A customer may opt in
to operational messages, promotional messages, or both — the opt-in form lets
them choose. We do **not** message anyone who has not affirmatively opted in.

## How customers opt in (proof of consent)

**Primary method — web form.** At **https://chargingsystemsinc.com/sms** the
customer enters their mobile number and checks an **unchecked-by-default**
consent box. The form displays, at the point of consent, all of the following:

- The business name (**Charging Systems Inc.**).
- A description of the message types (operational route notifications and/or
  promotional offers) and approximate frequency.
- "**Message and data rates may apply.**"
- "**Reply STOP to opt out, HELP for help.**"
- Links to this **SMS program / Privacy Policy** and **Terms**.
- A statement that consent is **not a condition of purchase**.

Submitting the form with the box checked records the customer's mobile number,
the consent timestamp, and the exact disclosure text shown. A screenshot of this
form is the `opt_in_image_urls` proof submitted to Twilio (it replaces the prior
Markdown-doc URL).

**Secondary method — route-customer agreement.** Delivery-route customers may
also opt in to operational notifications via a checkbox on their route service
agreement, capturing the same consent fields. Promotional messaging always
requires the web-form opt-in above.

## Message frequency

- **Operational route notifications:** as needed on active route days — a few
  messages per week for a given customer.
- **Promotional messages:** infrequent — no more than a few per month.

## How to opt out

Reply **STOP** to any message to be removed immediately. Reply **HELP** for
contact information. Standard STOP / HELP / UNSTOP keyword handling applies.
**Message and data rates may apply.**

## Sample messages

```
[Charging Systems] Hi — we're running your route today. Need a stop?
Reply YES to be added. Reply STOP to opt out, HELP for help.
```

```
[Charging Systems] Your alternator order #10482 ships today from the USA.
Tracking to follow. Reply STOP to opt out.
```

```
[Charging Systems] This month: 10% off remanufactured starters (image attached).
Msg & data rates may apply. Reply STOP to opt out, HELP for help.
```

## Privacy

We do **not** sell or share customer mobile numbers or SMS opt-in data with any
third party for their own marketing. Numbers and message history are stored only
in our internal database and in Twilio's standard message logs, and are used
solely to deliver the messages the customer opted in to. SMS opt-in is handled
separately from any other data sharing described in our Privacy Policy.

## Contact

For questions about this SMS program:

**Email:** admin@chargingsystemsinc.com
**Charging Systems Inc.**
4000 Pleasant Grove
Norman, OK 73072
United States
