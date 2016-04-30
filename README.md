# web-of-trust

This is a design/discussion document for the Web of Trust used in the secure-scuttlebutt network.

A WoT is a toolset for constructing and collecting credentials to authenticate pubkey identities, and authorize permissions.
Unlike Certificate-authority PKI, the WoT does not appoint global authorities for identity; instead, users choose their own authorities, and issue thir own credentials.


## Glossary

 - **User**: An agent in the network. In the current version of SSB, a user is identified by a single keypair and log.
 - **Identity**: All data which describes a user. Usually meant to identify who they are, and make them discoverable to friends/colleagues. Includes information like name, email address, bio, etc.
 - **Rights**: Permission to use some resource or behavior.
 - **Credential**: A datum used to validate identity or rights. Each credential's validity depends on the trust placed in the credential's creator. A credential could be a signed statement about the user's pubkey, or a capability string.
 - **Credential policy**: The rules for validating a specific credential, including who is allowed to create the credential, and how many credentials are required.
 - **Introducer**: A user who has been trusted to issue credentials. "Certificate Authorities" are introducers. In the WoT, there are no global authorities, so "Introducer" feels like a better term to use.
 - **Delegation**: Attribution of authority to another user. "I trust X to do Y."

Some example usages:

 - "Based on these credentials, I am confident that this pubkey is Bob Robertson's."
 - "I'm updating my credential policy to trust Alice to identify users correctly."
   - aka "I'm delegating introductions to Alice."

## Background

The Web-of-Trust was first used by PGP, and [can be read about here](http://www.pgpi.org/doc/pgpintro/#p17) (recommended).
Not all of PGP's concepts will translate to SSB, but many will. 
Here are some useful excerpts:

> ### Digital certificates

> In a public key environment, it is vital that you are assured that the public key to which you are encrypting data is in fact the public key of the intended recipient and not a forgery. Digital certificates, or certs, simplify the task of establishing whether a public key truly belongs to the purported owner.

> A certificate is a form of credential. Examples might be your driver's license, your social security card, or your birth certificate. Each of these has some information on it identifying you and some authorization stating that someone else has confirmed your identity. 

> ### Validity and trust

> Every user in a public key system is vulnerable to mistaking a phony key (certificate) for a real one. *Validity* is confidence that a public key certificate belongs to its purported owner.

> When you've assured yourself that a certificate belonging to someone else is valid, you can sign the copy on your keyring to attest to the fact that you've checked the certificate and that it's an authentic one. If you want others to know that you gave the certificate your stamp of approval, you can export the signature to a certificate server so that others can see it.

> Some companies designate one or more Certification Authorities (CAs) to indicate certificate validity. It is the job of the CA to issue certificates to users — a process which generally entails responding to a user's request for a certificate. Basically, the main purpose of a CA is to bind a public key to the identification information contained in the certificate and thus assure third parties that some measure of care was taken to ensure that this binding of the identification information and key is valid.

> #### Establishing trust

> You validate certificates. You trust people. More specifically, you trust people to validate other people' certificates. Typically, unless the owner hands you the certificate, you have to go by someone else's word that it is valid.

> In relatively closed systems, such as within a small company, it is easy to trace a certification path back to the root CA. However, users must often communicate with people outside of their corporate environment, including some whom they have never met, such as vendors, customers, clients, associates, and so on. Establishing a line of trust to those who have not been explicitly trusted by your CA is difficult.

> **Direct Trust**
Direct trust is the simplest trust model. In this model, a user trusts that a key is valid because he or she knows where it came from. All cryptosystems use this form of trust in some way. For example, in web browsers, the root Certification Authority keys are directly trusted because they were shipped by the manufacturer. If there is any form of hierarchy, it extends from these directly trusted certificates.

> **Hierarchical Trust**
In a hierarchical system, there are a number of "root" certificates from which trust extends. These certificates may certify certificates themselves, or they may certify certificates that certify still other certificates down some chain. Consider it as a big trust "tree." The "leaf" certificate's validity is verified by tracing backward from its certifier, to other certifiers, until a directly trusted root certificate is found.

> **Web of Trust**
A web of trust encompasses both of the other models, but also adds the notion that trust is in the eye of the beholder (which is the real-world view) and the idea that more information is better. It is thus a cumulative trust model. A certificate might be trusted directly, or trusted in some chain going back to a directly trusted root certificate (the meta-introducer), or by some group of introducers.

> In a PGP environment, any user can act as a certifying authority. Any PGP user can validate another PGP user's public key certificate. However, such a certificate is only valid to another user if the relying party recognizes the validator as a trusted introducer. (That is, you trust my opinion that others' keys are valid only if you consider me to be a trusted introducer. Otherwise, my opinion on other keys' validity is moot.)

> Stored on each user's public keyring are indicators of
> - whether or not the user considers a particular key to be valid
> - the level of trust the user places on the key that the key's owner can serve as certifier of others' keys

> You indicate, on your copy of my key, whether you think my judgement counts. It's really a reputation system: certain people are reputed to give good signatures, and people trust them to attest to other keys' validity.

## Applications of the WoT

### Authentication

Authentication is the process of verifying the identity of a user via trusted credentials.

The credential policy in CA PKI is hierarchical, and eminates down from the root CAs.
A new credential is considered valid if it presents an unbroken signature chain between the new credential, (optionally) through intermediary CAs, and terminating at a root CA.

In a WoT, the credential policy is set by each user; there are no root CAs.
You appoint other users as "CAs," and can assign varying levels of trust.
See: [Validity and trust in PGP](http://www.pgpi.org/doc/pgpintro/#p17).

> For example, suppose your key ring contains Alice's key. You have validated Alice's key and you indicate this by signing it. You know that Alice is a real stickler for validating others' keys. You therefore assign her key with Complete trust. This makes Alice a Certification Authority. If Alice signs another's key, it appears as Valid on your keyring.

> PGP requires one Completely trusted signature or two Marginally trusted signatures to establish a key as valid. PGP's method of considering two Marginals equal to one Complete is similar to a merchant asking for two forms of ID. You might consider Alice fairly trustworthy and also consider Bob fairly trustworthy. Either one alone runs the risk of accidentally signing a counterfeit key, so you might not place complete trust in either one. However, the odds that both individuals signed the same phony key are probably small.

### Authorization

Authorization is the process of giving rights over a resource or behavior to a user via trusted credentials.

The credential-policy for authorization may depend on the resource/behavior, or the specific user.
Some example policies:

 - **Requires successful authentication**. If the requester can prove to have an accountable identity, that's good enough. This is the policy of websites that require a valid email or facebook-account to signup, for instance.
 - **Requires trusted rights assignment**. If the resource-owner has assigned rights to the requester, or a trusted peer of the resource owner has assigned rights to the requester, that's good enough.
 - **Requires a capability string.** If the requester holds a valid capability string, that's good enough.

## Discussion

### Scaling the WoT

The WoT has a natural growth-boundary, at the edge of each user's social graph.
Total strangers -- users with no overlap in who they trust -- are not able to make introductions to each other.
And, of course, every new user is a stranger to everybody, as far as the WoT is concerned.

How can this be solved?

#### Invite codes

Invite codes are capability-strings which can be emailed, IMed, etc, to a target user.
They allow any two users to credential each other.

#### Automated CREdentialing Services (ACREs)

There are some kinds of credentials that can be verified automatically, such as ownership of an email or web account.
(Consider: most web services verify an email, phone, or twitter/gh/facebook account as a prerequisite for signup.
The cheaper SSL certificates only verify control of an email, and of the host & dns entry.)

Having bot-users on ssb which verify these kinds of credentials, and publish the verification, would be handy for automating introductions between strangers.

#### Introducer-discovery through ssb logs

The hieararchical CA model benefits from being a static shared configuration.
But, in the Web-of-Trust, the "CAs" are unique to each user.
So, you need to discover which introducers a stranger uses.

This can be solved by publishing your trust-delegations on the ssb log.
A stranger would download the log, read the delegations, and then get credentialed by those users.

This is only useful if you delegate to an ACRE, or to somebody who's business it is to credential strangers.
Otherwise, the delegations will just be a list of your friends, that dont want to be bothered either.