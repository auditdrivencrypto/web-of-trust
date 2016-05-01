# web-of-trust

This is a design/discussion document for the Web of Trust used in the secure-scuttlebutt network.

A WoT is a toolset for constructing and collecting credentials to authenticate pubkey identities.
Unlike Certificate-authority PKI, the WoT does not appoint global authorities for identity; instead, users choose their own authorities, and issue their own credentials.

## Glossary

 - **User**: An agent in the network. In SSB, a user consists of a keypair and log.
 - **Identity**: All data which describes a user. Includes information like name, email address, bio, etc.
 - **Credential**: A datum which attributes identity information to a user. Each credential's validity depends on the trust placed in the credential's creator.
 - **Credential policy**: The rules for validating a specific credential, including who is allowed to create the credential, and what kind of credentials are accepted.
 - **Delegation**: A declaration of trust in a user to issue credentials.
 - **Introducer**: A user who has been trusted to issue credentials.
 - **Stranger**: A user with no trusted credentials.

## Background

The Web-of-Trust was first used by PGP, and [can be read about here](http://www.pgpi.org/doc/pgpintro/#p17) (recommended).
Not all of PGP's concepts will translate to SSB (for instance, SSB doesnt use "certificates") but many will. 
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

## Discussion

### Introductions

When you want to meet somebody for the first time, you need to be introduced.
Usually, you have a mutual acquaintance do it for you.
That way, you know you've found the right person.

In SSB, the same process applies for contacting new people.
You look for mutual friends who can identify users in the network.
That way, you know you're sending messages to the right person.

**Why does SSB need introductions?**

Email works without an "introduction" process.
You just find the person's address and send the email.
Why does SSB need it?

The answer is that SSB has a higher standard of security than email has.
Email entrusts the email hosts to correctly identify its users and send their messages privately.
But there's no builtin protection against hosts making mistakes, getting hacked, or selling you out.
If the host wants to send your email straight to Russia, it can.

(Somewhat more troubling is, [the email transport protocol is not secure](https://blog.filippo.io/the-sad-state-of-smtp-encryption/).
Most messages can be passively read over the wire.
[Gmail is taking steps to improve this](https://support.google.com/mail/answer/6330403?p=tls&hl=en&rd=1), but it's not solved.)

SSB is designed to let you transmit sensitive information, such as system credentials, legal documents, and medical information.
It entrusts *nobody except the recipient* with the message content, using independent confirmations of the recipient's identity, and end-to-end encryption.

SSB's Web-of-Trust is a tool for confirming a recipient's identity.
It gathers multiple overlapping confirmations from your contact list, most of whom you should know personally.
It provides a reasonable expectation of privacy on the public Internet, without relying on firewalls or on-site servers.

### Introducing strangers

When you go to the bank, you need to introduce yourself to the teller.
To do this, you show them your debit card and driver's license, which are two independent trusted forms of identification.

SSB has a need for the same process.
The WoT has a natural scaling-boundary for each user, at the edge of their personal network.
To be introduced to a new person, you need a trusted credential for them; if the user's a stranger, then none of your introducers will have issued one.

This is a bootstrapping problem.
If there's no existing connection, then how do you create one?
Trusting the identity of a total stranger would be like accepting a self-signed certificate.

Until you are well-connected, this is a very common problem.
A new user in the network has no connections at all.

How can this be solved?

**Invite codes**

Invite codes are capability-strings which can be emailed or IMed to a target user.
They allow any two users to credential each other.

Invite codes are effective but tedious.
They require users to have a private, trusted channel to each other.
And, they require the users to walk through manual processes.

Invite codes are probably the best way to intro a completely new user into the network.

**Automated Introducers**

There are some kinds of credentials that can be verified automatically, such as ownership of an email, phone number, or web host.
Having bot-users on ssb which do this, and publish the verifications, would be handy for introducing strangers.
With a trusted auto-introducer, you'd be able to lookup users by these verifications.

**Introducer-discovery through ssb logs**

The hieararchical CA model benefits from being a static shared configuration.
But, in the Web-of-Trust, the introducers are unique to each user.
So, you need to discover which introducers a stranger uses to be properly credentialed.

This can be solved by publishing the trust-delegations on the ssb log.
A stranger would download the log, read the delegations, and then get credentialed by those users.

(This is only useful if you delegate to an automated introducer, or to somebody who's business it is to credential strangers.
Otherwise, the delegations will just be a list of your friends, who dont really want to be bothered.)