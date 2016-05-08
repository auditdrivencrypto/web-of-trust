# web-of-trust

This is a design/discussion document for the Web of Trust used in the secure-scuttlebutt network.

A "Web of Trust" (WoT) is a toolset for creating identities in a decentralized network.
Unlike Certificate-authority PKI, the WoT does not appoint global authorities for identity; instead, users choose their own authorities, and can issue their own credentials.

## Overview

In SSB, users enter into the network by joining Pub servers.
Users are encouraged to join more than one.
Pubs rehost user data, and exchange messages between members.
Their membership is exclusive (invite-only).

When users join a Pub, they do so with an invite-code that must be sent in a private trusted-enough channel, such as email.
The invite code has a name attached to it, indicating who the admin *believes* they are inviting.
This information is then published on the Pub's log.
The member directory is built out of these published (believed) identities.
By aggregating the memberships of a given user, we can get a picture of who they are.
(Or, at least, who their Pub operators believe they are.)

Pubs are not the only identity-authorities in the network, but they are the default.
They make good defaults because:

 1. They are followed by their members, meaning identity changes will be noticed.
 2. Identity-verification is a natural step of the invite process.
 3. They have an incentive to manage their community effectively, and maintain good directories.
 4. They are easy to replace, in the event of mismanagement (member devices operate independently of them).

Users can configure authorities which are not Pubs.
This is useful for users that want extra protection for key distribution, or for groups that only use wifi (and therefore dont have any Pubs).

## Glossary

 - **User**: In SSB, a user consists of a keypair and log.
 - **Pub**: A server in SSB. Pubs are users.
 - **Identity**: The data which describes a user (name, bio, etc).
 - **Verification**: Identity data which was verified through a trusted channel.
 - **Trust**: A measure of whether data published by a user will be used without outside verification.
 - **Authority**: A user which has been trusted.

## Verifications

Verifications are a tool for confirming somebody's identity within *reasonable* certainty.
Specifically, a verification is used to tie a name (or some other identity datum) to a pubkey.
They are not fool-proof, which is why it's a good idea to accumulate verifications.

### Invite-code verifications

"Invite-codes" are a tool used in SSB to give Pub-membership to users.
They include connection information about the Pub, as well as the seed for producing a One-Time-Use Keypair.
The OTU Keypair is used as a [capability string](https://en.wikipedia.org/wiki/Capability-based_security).
The code-recipient uses the keypair to authenticate with the Pub, and then is given rights to command the Pub to follow their main Keypair.

Invite-codes are sent using a trusted-enough medium, such as email, SMS, or a web service PM.
This is a similar flow to most Web-service signups, which send confirmation links to the new user's email inbox.
The sender should have reasonable certainty that the code is going to its intended recipient.
IRC's private messaging, for instance, is riskier than email or SMS, because IRC is very lax about its username ownership.

To use Invites as a verification, the Pub admin should publish an "Invite Announcement" message on the Pub log noting the following information:

 - The OTU Kepair's public key.
 - The name of the recipient of the Invite code.

The recipient will download that message, confirm that the name is correct, and use the Invite to join the Pub.
After joining the Pub, the recipient will publish a "Invite Accepted" message on their own log with the following information:

 - The ID of the Pub's Invite Announcement message.
 - The name which the recipient confirmed.
 - A signature proving ownership of the OTU Keypair on the Invite-code.

Likewise, the Pub will publish a "Invite Used" message with the following information:

 - The ID of the Pub's Invite Announcement message.
 - The public key of the recipient who used the Invite code.

We now have overlapping confirmations of the new identity.
The Pub asserts the OTU Keypair, recipient's name, and recipient's pubkey; the recipient confirms ownership of the OTU Keypair, and confirms the asserted name.
Users who trust the Pub to handle Invites can now add the identity information to their database.