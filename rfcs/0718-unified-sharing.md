# Unified Sharing

Support a consistent, flexible model of "personal" vs "shared" item visibility in Signals, Queries, and Dashboards.

## Motivation

Seq 4.2 has two distinct models for sharing common items in the main events/dashboards screens:

| Signals & Queries | Dashboards |
| --- | --- |
| Shared only | Shared or personal |
| Shared by default | Personal by default |
| Shared items openly editable | Shared items admin-only |
| "Restricted" editing option | - |
| Explicit visibility selection | All items visible |

This leaves gaps that have been noted by users:

 * It's not possible to "hide" items from a large dashboard list
 * It's not possible for non-admin users to share dashboards, or edit shared dashboards
 * New signals created by one user don't appear/aren't discoverable by other users
 * Throwaway/temporary "personal" signals clutter the global shared signal list

This RFC seeks to unify these two sharing models, so that one well-aligned mental model applies for all shared resources dealt with day-to-day in Seq.

## Detailed design

The following guiding principles have been established:

 1. Sharing is an explicit choice. It should be easy and comfortable to work in Seq without worrying about cluttering up shared resources. Creating a new shared resource, or modifying an existing one, should be an obvious and deliberate action.
 1. Collaboration is the default. While administrative users may need to be able to restrict editing of sensitive signals and dashboards, this is the exception rather than the rule. Users should be free to collaborate and improve these things by default.
 1. [Labels]() and [Signal Groups]() are the primary means of dealing with clutter. It's unwieldy to rely on "hiding" items to keep lists clean, as personal roles/focus may change regularly.

Applying these, the model proposed by this RFC is:

 * All items can be marked with an "owner", in which case they will be regarded as "personal"
 * All newly-created items will initially be owned by their creator (except in API-driven/automation scenarios)
 * Any user can make a "personal" item into a "shared" item
 * Administrative users can mark shared items as "restricted", in which case they can only be modified by administrative users
 * Any user can edit, delete, or un-share (convert to personal) any unrestricted "shared" item
 * Users will by default see all of their personal items, and all shared items, in each category
 * Shared items can be "hidden", which corresponds with the current "Remove" function on signals
 * Personal items can only be deleted, not hidden

### New permissions

The "restricted" option on dashboards and signals will be implemented by introducing a new permission, `ModifyRestrictedItem`, which will be granted to the built-in administrators group.

### Migration

**Dashboards:** All shared dashboards will be marked "restricted" by the migration process, so that new rights are not conferred; administrators can remove this restriction as needed.

**Signals:** The current "user signal list" will be inverted by the migration, so that it tracks hidden rather than shown signals.

### Client changes

_Seq.Api_ and `seqcli` will need updates to work with the new sharing model.

## Drawbacks

None raised so far.

## Alternatives

The space of possible sharing models is open. This proposal maps as closely as possible to the current system, under the assumption that these changes won't impose any significant additional restrictions on what improvements may be made in the future.

## References

 * [#623/Personal (vs Shared) Signals (enhancement)](#623)
 * [#582/Allow non-administrative users to share dashboards (enhancement)](#582)
 * [#626/Hide items from the dashboard list (enhancement)](#626)
