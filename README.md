# web-of-trust

This is a design/discussion document for the Web of Trust used in the secure-scuttlebutt network.

A WoT is a toolset for constructing and collecting credentials to authenticate pubkey identities, and authorize permissions.
Unlike Certificate-authority PKI, the WoT does not appoint global authorities for identity; instead, users choose their own authorities, and issue thir own credentials.


## Glossary

 - **User**: An agent in the network. In the current version of SSB, a user is identified by a single keypair and log.
 - **Identity**: All data which describes a user. Includes information like name, email address, bio, etc.
 - **Rights**: Permissions to use some resource or behavior.
 - **Credential**: A datum which attributes identity information or permissions to a user. Each credential's validity depends on the trust placed in the credential's creator.
 - **Credential policy**: The rules for validating a specific credential, including who is allowed to create the credential, and what kind of credentials are allowed.
 - **Introducer**: A user who has been trusted to issue identity credentials.
 - **Delegation**: A declaration of trust in another user, to issue some kind of credential.
 - **Stranger**: A user with no trusted credentials. Strangers occur when two users are trying to introduce to each other, but there's not yet any overlap in their WoT.

## Background

The Web-of-Trust was first used by PGP, and [can be read about here](http://www.pgpi.org/doc/pgpintro/#p17) (recommended).
Not all of PGP's concepts will translate to SSB, but many will. 
Here are some useful excerpts:

> ### Digital certificates

> In a public key environment, it is vital that you are assured that the public key to which you are encrypting data is in fact the public key of the intended recipient and not a forgery. Digital certificates, or certs, simplify the task of establishing whether a public key truly belongs to the purported owner.

> ### Validity and trust

> Every user in a public key system is vulnerable to mistaking a phony key (certificate) for a real one. *Validity* is confidence that a public key certificate belongs to its purported owner.

> Some companies designate one or more Certification Authorities (CAs) to indicate certificate validity. It is the job of the CA to issue certificates to users — a process which generally entails responding to a user's request for a certificate.

> #### Establishing trust

> You validate certificates. You trust people. More specifically, you trust people to validate other people' certificates. Typically, unless the owner hands you the certificate, you have to go by someone else's word that it is valid.

> **Direct Trust**
Direct trust is the simplest trust model. In this model, a user trusts that a key is valid because he or she knows where it came from. All cryptosystems use this form of trust in some way. For example, in web browsers, the root Certification Authority keys are directly trusted because they were shipped by the manufacturer. If there is any form of hierarchy, it extends from these directly trusted certificates.

> **Hierarchical Trust**
In a hierarchical system, there are a number of "root" certificates from which trust extends. These certificates may certify certificates themselves, or they may certify certificates that certify still other certificates down some chain. Consider it as a big trust "tree." The "leaf" certificate's validity is verified by tracing backward from its certifier, to other certifiers, until a directly trusted root certificate is found.

> **Web of Trust**
A web of trust encompasses both of the other models, but also adds the notion that trust is in the eye of the beholder (which is the real-world view) and the idea that more information is better. It is thus a cumulative trust model. A certificate might be trusted directly, or trusted in some chain going back to a directly trusted root certificate (the meta-introducer), or by some group of introducers.

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

### Introducing strangers

The WoT has a natural scaling-boundary for each user, at the edge of their personal network.
To be introduced to a new person, you need a trusted credential for them; if the user's a stranger, then none of your introducers will have issued one.
Trusting the identity of a total stranger, in this case, would be like accepting a self-signed certificate for them.

This is a bootstrapping problem.
If there's no existing connection, then how do you create one?

Until you are well-connected, this will be a common problem.
A new user in the network has no connections at all; all users are strangers.

How can this be solved?

**Invite codes**

Invite codes are capability-strings which can be emailed, IMed, etc, to a target user.
They allow any two users to credential each other.

**Automated CREdentialing Services (ACREs)**

There are some kinds of credentials that can be verified automatically, such as ownership of an email, phone number, or web host.
(Most web services and SSL cert-issuers do this during signup.)

Having bot-users on ssb which do this, and publish the verifications, would be handy for introducing strangers.
With a trusted ACRE, you'd be able to lookup users by these verifications.

**Introducer-discovery through ssb logs**

The hieararchical CA model benefits from being a static shared configuration.
But, in the Web-of-Trust, the introducers are unique to each user.
So, you need to discover which introducers a stranger uses to be properly credentialed.

This can be solved by publishing the trust-delegations on the ssb log.
A stranger would download the log, read the delegations, and then get credentialed by those users.

(This is only useful if you delegate to an ACRE, or to somebody who's business it is to credential strangers.
Otherwise, the delegations will just be a list of your friends, who dont really want to be bothered.)