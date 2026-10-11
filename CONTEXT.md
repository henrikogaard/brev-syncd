# brev-syncd context

This glossary defines the product and protocol language for the single
`brev-syncd` context. It deliberately describes trust and ownership boundaries,
not implementation details.

## Language

**Workspace**:
A user-owned privacy boundary containing authorized devices and one encrypted
metadata history.
_Avoid_: account, tenant, user

**Device**:
A specific Brev installation authorized to read and contribute to a Workspace.
_Avoid_: client, session, user

**Membership record**:
A signed statement that grants or withdraws a Device's authority in a
Workspace.
_Avoid_: login, account record

**Relay**:
The self-hosted `brev-syncd` service that authenticates Devices and durably
stores opaque Workspace data.
_Avoid_: mail server, sync authority, cloud

**Opaque envelope**:
An authenticated ciphertext container the Relay can validate and store without
interpreting its metadata payload or mail identity.
_Avoid_: message, email record

**Metadata operation**:
A client-created, encrypted change to Brev-owned state for a locally resolved
mail item.
_Avoid_: mail operation, IMAP operation

**Locator**:
A client-only keyed reference that connects metadata to a locally resolved mail
item and is never exposed to the Relay.
_Avoid_: server key, message ID

**Snapshot**:
An opaque encrypted representation of a Workspace's compacted metadata state.
_Avoid_: backup, server state
