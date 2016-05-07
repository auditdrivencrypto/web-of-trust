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
 - **Verification**: Identity data which was verified through a trusted channel. A verification's validity depends on the trust placed in its publisher.
 - **Trust**: A measure of whether data published by a user will be used without outside verification.
 - **Authority**: A user which has been trusted.
