NIP-XX
======

Zap Tags
--------

`draft` `optional`

This NIP defines a set of tags to set zap parameters for public/social payments. Users can add optional conditions and instructions for their notes like a minimum and maximum zap amount, a redirect of a zap to an npub or a note, and specific LNURL to handle zaps from a specific note. This will enable nostr enabled lightning wallets and supporting clients to better emulate LNURL behaviour via zaps.

## Zap as Social Payments

Extending zaps to interaction outside of NOSTR for users that want easily associate zaps to events outside of NOSTR and wish to have social prove that the payer and payee have settled a transaction.

Vendors, creditors, and payees need to verify payments outside of NOSTR, but third party observers can cryptographically prove at least that payer and payee have both agreed that payment is complete by validating zap receipt event.

Although not strictly enforceable, these tags will help users request specific amounts and redirect the payment to a different note and or public key, while also expediting the payment of the zap requests by providing a one-tap solution to an arbitrary zap amount. The functionality could be mapped to specific LNURLs that can generate invoices using the provided parameters.

These parameter aim at minimising the friction of having to input custom amounts and reduces the risk of input incorrect values, while keeping the payment information contained in a kind1.


## Tags

The following tags are defined as OPTIONAL.

- `zap-min`       - Minimum target amount in milisats.
- `zap-max`       - Minimum target amount in milisats.
- `zap-uses`      - Number of expected zaps with an amount equal or greater than zap-min.
- `zap-payer`     - Public key hex of expected zapper
- `zap-target`    - Total amount of sats to be raised
- `zap-split`     - List of public keys and split weight
- `zap-note`      - Note id for zap forwarding (what happens if zap-redirect and zap-lnurl are defined? Which has priority)
- `zap-lnurl`     - Custom LNURL override on a note by note basis (if lnurl not responsive, fallback to kind0 of author?)


## Example events and use cases

### Purchase Charge
A restaurant gives the patron the option to pay privately via a lightning invoice, or publicly via a zap. If patron agrees, the restaurant will have social proof of the patronage while the patron has social proof of the expense.

```json
{
  "kind": 1,
  "tags": [
    ["zap-min", "18938000"]
  ],
  "content": "Thank you for shopping at @CornerStore. Your total is 18,938.",
  ...
}
```


This note pays to nostr:npub1vp8fdcyejd4pqjyrjk9sgz68vuhq7pyvnzk8j0ehlljvwgp8n6eqsrnpsw's LN-address.\nExperimenting with zap-lnurl tag, available only on https://pubpay.me 😉


### Debt Collection
A user wishes to publicly announce that a specific user owes a specific amount. Debtor can be tagged on the note and zap the event for a specific amount via one single tap triggering a specific amount zap. Social proof of payment is recorded on the note.

```json
{
  "kind": 1,
  "tags": [
    ["zap-min", "210000000"],
    ["zap-payer", "c4633ff9e5e26f8dae0ab752f31a75d829ec9a7af9c62a3d949a08355e32a661"]
  ],
  "content": "It's been 45 days and @NostrBum still owes me 210,000.",
  ...
}
```




### Bill Split
A business wishes to breakdown a bill in multiple parts for ease of payment by a group of users. Each of the users can find the note on their feed and pay an amount split in equal parts.

```json
{
  "kind": 1,
  "tags": [
    ["zap-min", "45534000"],
    ["zap-uses", "5"]
  ],
  "content": "Thank you for dining at @Summer_Cafe. Your total is 45,534 of a total of 227,670 split by 5.",
  ...
}
```




### Crowd Pay
User A wishes to crowd fund a zap payment request from another note while still giving credit to the user B who actually pays. Zap is redirected to the root note carrying both `user A` and `user B`.

```json
{
  "kind": 1,
  "tags": [
    ["zap-min", "45534000"],
    ["zap-forward", "e6921ff965e26f81ae0abfecf31a75d829ec9a7af9c62a3d949a04e75e32ae51"]
  ],
  "content": "Just had a great dinner at @Summer_Cafe. nostr:note1feypu6wke3eys7yrvwahd6n73psjpmpsscrrau3f2s86g43f2rhspfgnz3 Who want's to pay my part? Help me out, it's only 45,534 sats.",
  ...
}
```

`User A` who requested crowed payment is credited along side `user B` who actually pays the event.
`User B` has social proof it payed for `user A`.
`User A` has social proof bill was settled.
`User C` who created the original payment request has social proof of interaction with `User A`.

```json
{
  "kind": 9734,
  "tags": [
    ["amount", "21000"],
    ["lnurl", "lnurl1dp68gurn8ghj7um5v93kketj9ehx2amn9uh8wetvdskkkmn0wahz7mrww4excup0dajx2mrv92x9xp"],
    ["p", "recipientID"],
    ["e", "root_EventID"],
    ["req-e", "requesting_EventID"],
    ["req-p", "requesting_UserA_publicKey"],

  ],
  "content": "Happy to pay for your dinner!",
  "pubkey": "userB_publicKey",
  ...
}
```




### Specified LNURL
A user wishes zaps to go to a LNURL with specific parameters that react and adjust as zaps accomulate.

```json
{
  "kind": 1,
  "tags": [
    ["zap-min", "1000000"],
    ["zap-max", "7000000"],
    ["zap-uses", "30"],
    ["zap-increments", "200000"],
    ["zap-lnurl", "lnurle6921ff965e26f81ae0abfecf31a75d829ec9a7af9c62a3d949a04e75e32ae51"]
  ],
  "content": "We have 30 tickets selling starting at 1000 sats each. First 30 zaps get the tickets. Price goes up 200 sats for every sell. Payment closes after selling out",
  ...
}
```


## Client Behaviour

Clients MAY give users additional options when creating notes with fields to determine zap parameters.

Clients MAY display zap parameters when rendering notes by including CTAs that indicate expected zap reactions.

When zapping an event, users can use default zapping interface or tap zap UI parameter with specific conditions if client recognise the parameters.

Forward zaps should include the public key of the user that payed the zap and the user that created the forward so both users have social proof.

If user specifies a `zap-payer` along with a `zap-min` when creating a note, interface should display `zap-payer` user profile info along side or incorporated in CTA.

Additional zaps are always allowed unless an specific LNURL override prevents it.



## Related NIPs

- `zapsplits (multiple keys)` - split zap by multiple users, clients should ignore all `zap-x` tags if `zap` tags are defined.
- `zapsplits (single key)` - effectively a single split, redirects the zap but only to `pubkey`, `lud06` or `lud16`, not notes.
- `zapgoals` - wont show up on main social feed because of special event kind.
- `zapraises` - focused on raising zaps from a pool of users, could be used in conjunction with other zap-parameters tags.

