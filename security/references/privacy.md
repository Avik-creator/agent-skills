# Privacy

Read when a change touches personal, payment, or health data. Record the classification and the retention in the design doc. Do not claim the system is compliant. Do not write a privacy policy, consent wording, or a data-processing agreement.

## Each new field

Classify it: non-personal, personal, or sensitive (health, finance, precise location, biometrics, government identifiers, data about minors). State the purpose. "We might want it later" is not a purpose. Set how long it is kept.

The deletion path has to reach backups, caches, search indexes, analytics, and third-party processors. An export path has to return what you hold about that person. A flag that hides the row is not deletion. Why: a request you cannot find the rows for is a request you cannot finish.

## Hard rules

- No direct identifiers (name, email, phone, address, government id, card data) or free text from the person in logs, traces or a model context. An opaque internal id (user id, tenant id) is allowed as a log field; audit events need it. Why: the id joins records without exposing the person, and it is deleted with them.
- Sending personal data to a vendor (analytics, a model, ads) is sharing. Ask first.
- Card verification codes and PINs: PCI DSS v4.0.1 requirements 3.3.1.2 and 3.3.1.3 say they are not stored after authorization, even encrypted. They may be used for that authorization and then dropped. An issuer is the exception named in the standard; if the owner might be one, ask. Do not store the card number yourself. Take a token from the payment provider.
- Audit events to emit, and how to log them, are in the observability skill: login success and failure, role and permission changes, data export, data delete.

## Which regulations might apply

Name the ones that might, say why you think so, and leave the answer as an open question for the owner or for legal. Checked on 2026-11-02:

- **GDPR** (Regulation (EU) 2016/679, Article 3). It can apply to processing in the context of an establishment in the Union, and to processing of people who are in the Union by someone outside it when that processing is about offering them goods or services, or about monitoring their behaviour in the Union.
- **CCPA.** Ask when the people are California residents. The statute's tests were not re-checked for this skill; do not state that it applies.
- **HIPAA.** Ask when the data is health information. Who counts as a covered entity was not re-checked for this skill; do not state that it applies.
- **PCI DSS.** Ask when the system stores, processes, or transmits cardholder data or sensitive authentication data. The scope sentence is the standard's own (v4.0.1).

## A suspected leak

Tell the user at once that a legal deadline may apply. Stop and let them take it to the owner or to legal.

If GDPR may apply, quote Article 33 rather than a round number. The controller notifies the supervisory authority without undue delay and, where feasible, not later than 72 hours after becoming aware, unless the breach is unlikely to result in a risk to the rights and freedoms of natural persons. A later notice includes the reasons for the delay. The processor notifies the controller without undue delay; the 72 hours are the controller's. Source: the regulation text, checked on 2026-11-02.
