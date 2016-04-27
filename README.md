# web-of-trust

This is a design/discussion document for the Web of Trust used in the secure-scuttlebutt network.

A WoT is a toolset for constructing and collecting identity proofs to authenticate pubkey identities.
Unlike Certificate-authority PKI, the WoT does not appoint any total authorities for identity; instead, users generate a "social graph" of certifications which improve the certainty for a keypair/identity pairing.
The proofs are aggregated to determine a formal (or informal) probability of authentication, which is accepted after reaching some threshold.

The purpose of this document is to discuss interface designs, cryptography protocols, and network protocols.

## Identity signals

### Profile information

Every user asserts profile information for their ssb identity.
This includes name, bio, image, etc.

It's possible for users to assert matching (or different) profile information for each other.
This looks like:

```js
{
  type: 'about',
  about: pubkey,
  name: 'bob',
  desc: 'My good friend bob'
}
```

There's no source-evidence involved in profile information.
It does reveal who users *believe* they're viewing, which makes it a good social signal, but a mediocre identity proof signal.

**Interface**: The self-profile is obvious. Assigning a custom name/profile/image to users may still be useful?

### Ownership proofs

A user creates an assertion that they own an identity on another channel.
The assertion is signed by the ssb identity, then published on that channel and on the ssb log.
Other users receive the assertion by the log, and then verify it on the other channel -- or visa-versa.

This can be applied to: websites/domains, email, twitter, github, reddit, hackernews.

The verifications can be published, by other ssb users, to improve the confidence in a proof.

**Interface**: This can be provided as a tool to "bind" your identity to some other account.

### Flow proofs

A user creates a secret token with a recipient in mind.
Evidence of the token, and the recipient's name, is published on the user's ssb log.
The user sends the token through any medium they like.

They recipient publishes evidence that they received the token.
If the sender accepts the evidence, then this becomes an identity proof.

### Upvotes

A user publishes an "upvote" for another user, signalling confidence in their identity.
This looks like:

```js
{
  type: 'vote',
  vote: {
    link: pubkey,
    value: 1,
    reason: 'This is my pub. I set it up myself.'
  }
}
```

Upvotes are a generic way to signal confidence without limiting users to a specific flow.
This makes them good to have for expert users, but slightly risky to consider as identity proofs, because they could be misused by inexperienced users.

**Interface**: Perhaps as an advanced tool for users to confirm an identity.
(Advanced meaning, behind a dropdown somewhere.)

### Trust delegations

Some identity signals are hard to trust because they could be misused by amateur or malicious users.
(Profile information, upvotes.)
However, from trust-worthy users, these signals could be very useful.

Delegations would tell the authentication algorithms to accept signals from specific users which are typically considered too soft to act as identity-proofs.
They could look like:

```js
{
  type: 'delegate',
  delegate: pubkey,
  trust: ['about', 'vote']
}
```

Published delegations are, like about and upvote, poor identity proofs, but strong social signals.

**Interface**: Perhaps a profile button, "Appoint as moderator," or something like that.
Other delegations in your social graph would be indicated as well.

### Flags

Flags are the inverse of recommendations.
They're a generic way to signal a lack-of-confidence in a user.

They look like:

```js
{
  type: 'vote',
  vote: {
    link: pubkey,
    value: -1,
    reason: 'Im extremely certain this is a fraud, because I know the real bob.'
  }
}
```

## Interface

### Onboarding: Joining Pubs

Joining a Pub is currently handled with an invite code.
We can improve the pub-invite to be a [flow-proof](#flow-proofs), or bake in email transmission to make it an [ownership-proof](#ownership-proofs).

A web ui for Pub signups could use the [email-ownership-proof](#ownership-proofs).

### Onboarding: after joining a Pub

Joining a Pub will cause the user to follow it, and therefore receive any identity signals the Pub has published.
The user could be shown the signals on the Pub's profile page, as a way to suggest first contacts.

This isn't terribly useful, though, without [unsolicited friend requests](#unsolicited-friend-requests).

### Unsolicited "friend" requests

SSB doesn't have any means for unsolicited messages; you have to follow somebody to receive messages from them.
Could there be a protocol, specifically for creating two-way connections, which allows a user to send their contact info to someone without it being solicited?