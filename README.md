# web-of-trust

This is a design/discussion document for the Web of Trust used in the secure-scuttlebutt network.

A WoT is a toolset for constructing and collecting identity proofs to authenticate pubkey identities.
Unlike Certificate-authority PKI, the WoT does not appoint any total authorities for identity; instead, users generate a "social graph" of certifications which improve the certainty for a keypair/identity pairing.
The proofs are aggregated to determine a formal (or informal) probability of authentication, which is accepted after reaching some threshold.

The purpose of this document is to discuss interface designs, cryptography protocols, and network protocols.

## Glossary

 - **User**: An agent in the network. In the current version of SSB, a user is identified by a single keypair, and its log. (In future versions, users may be identified by multiple keypairs and logs.)
 - **Identity**: The data assigned to a user that's meant to identify who they are, and make them discoverable to friends/colleagues. Includes information like name, email address, bio, etc. Identity data must be verified (see Authentication).
 - **Right**: A permission to use some resource or trigger some behavior.
 - **Credential**: A datum used to confirm an identity or authorize a right. Each credential's validity depends on the trust placed in the credential's creator. A credential could be a signed statement about the user's pubkey, or a capability string.
 - **Credentialing policy**: The rules for validating a specific credential, including who is allowed to create the credential, and how many credentials are required.

## Applications of the WoT

### Authentication

Authentication is the process of verifying the identity of a user via trusted credentials.

The credential policy in CA PKI is hierarchical, and eminates down from the root CAs.
A new credential is considered valid if it presents an unbroken signature chain between the new credential, (optionally) through intermediary CAs, and terminating at a root CA.

In a WoT, the credential policy is set by each user; there are no global authorities.
You appoint other users as "CAs," and can assign varying levels of trust.
See: [Validity and trust in PGP](http://www.pgpi.org/doc/pgpintro/#p17).

> For example, suppose your key ring contains Alice's key. You have validated Alice's key and you indicate this by signing it. You know that Alice is a real stickler for validating others' keys. You therefore assign her key with Complete trust. This makes Alice a Certification Authority. If Alice signs another's key, it appears as Valid on your keyring.

> PGP requires one Completely trusted signature or two Marginally trusted signatures to establish a key as valid. PGP's method of considering two Marginals equal to one Complete is similar to a merchant asking for two forms of ID. You might consider Alice fairly trustworthy and also consider Bob fairly trustworthy. Either one alone runs the risk of accidentally signing a counterfeit key, so you might not place complete trust in either one. However, the odds that both individuals signed the same phony key are probably small.

### Authorization

Authorization is the process of giving rights over a resource or behavior to a user via trusted credentials.

The credential-policy for authorization may depend on the resource/behavior, or the specific user.
Some example policies:

 - **Requires successful authentication**. If the requester can prove to have an accountable identity, that's good enough. This is the policy of websites that require a valid email or facebook-account to signup, for instance.
 - **Requires trusted rights assignment**. If the resource-owner has assigned rights to the requester, or a trusted peer of the reosurce owner has assigned rights to the requester, that's good enough.
 - **Requires a capability string.** If the requester holds a valid capability string, that's good enough.

An example of when WoT authorization is useful, is for an introduction protocol, when a user wants to establish contact with a new friend.
This is effectively a matter of spam-prevention.

## PGP vs SSB

### Discovery during introductions

One of the challenges for handling introductions in a WoT is discovering a pathway between two strangers.
If there is no trusted verification already available, then it can be unclear how to get the verifications you need.

In the hieararchical CA model, this is handled by using (mostly) universal root CAs.
If you need an intro to a stranger, you can have one of the shared roots (or their intermediaries) provide a new verification.
In a WoT, each user can choose their own trusted introducers; so, the "CA" must be discovered on a case-by-case basis.

One advantage of SSB over PGP is, SSB can distribute the validations and trust-policies of each user.
A user looking for an intro can download the log of their target user.
This gives them a way to discover the introducers or policies needed to make a new connection.


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

### Invite codes

A button on the top right of the SSBMail UI could generate an "invite code."

The user would be provided a form, for filling out who the code is for (making it a [flow-proof](#flow-proofs)).
The form would then generate a code, which includes a capability to send a "friend request" via the creator's pub.

A recipient would press a button on the top right, "Add contact," then input the invite code there.
The creator would be pinged, to let them know the invite was used.
Both users would now follow each other.

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