# web-of-trust

This is a design/discussion document for the Web of Trust used in the secure-scuttlebutt network.

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
