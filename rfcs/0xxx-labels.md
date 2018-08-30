# Labels

Reduce clutter and improve focus when working with the signal bar, dashboards, and other lists in large installations.

## Motivation

Heavily-used Seq installations accumulate large numbers of signals, dashboards, and other items. These reflect teams and developers working on different parts of large systems, projects beginning and ending over time, multiple projects operating concurrently, and people using Seq in different job roles and functions.

The contents of the signal bar and dashboard list are currently global for a user: there's no way to show or hide selected signals or dashboards based on tasks, job roles, or projects.

This RFC proposes a flexible mechanism for showing and hiding related items in these lists.

## Detailed design

_Labels_ are named, colored markers that group a set of related list items. Labels are applied to items, and then lists of those items can be filtered by selecting labels.

The canonical usage scenarios for labels are:

 * Subsystems, architectural, or functional areas, such as `online-ordering` or `database`
 * Role-oriented labels such as `infrastructure`, `business-metrics` or `testing`
 * Organizational labels - `security-team` or `queensland-team`

### Labeled items

The following items will be able to be labelled:

 * Signals
 * Saved queries
 * Dashboards

These items may have labelling applied in the future:

 * API keys
 * App streams (forthcoming)

### Defining labels

The set of usable labels will be managed in a single, admin-controlled collection. Users with sufficient permissions will be able to create and edit labels inline, when editing labelled items, or open the _Settings > Labels_ screen to see and edit all labels.

The API will only permit defined labels to be applied to items, removing any that are not present in the global label collection. When labels are edited or deleted, labelled items will be updated accordingly.

### Anatomy of a label

In its definition, a label comprises:

 * A short, single-token name, for example `backend` or `project:online-ordering`; this alone will uniquely identify the label. Because of its unique, identifying role, the label name may be referred to as just the "label".
 * A free-text description, limited to comfortable tooltip length (for example, 140 chars).
 * A color, interchangeable with a colorblind-friendly pattern fill.

Punctuation is permitted within label names. This does not have any scheme or semantic meaning applied by Seq, so teams can use different identifier/punctuation schemes to organize labels as needed.

Spaces are not permitted in label names. This will simplify token-based autocomplete/type-to-select in search boxes.

The label color will initially be chosen from a predefined set, chosen for being visible in both light- and dark-background themes. Arbitrary color selection may be added in future.

### Applying labels

In the editor for the labelled item, a control will allow zero or more labels to be chosen for that item.

When labels are associated with an item, markers using the labels' assigned colors will be displayed next to the item name when appropriate.

### Filtering based on labels

Lists of labelled items can be filtered by selecting labels.

Initially, lists show all items visible to the user. When a label is selected from the list filter box, only items with that label applied will be shown. As more labels are selected, only items with all of the selected labels will be shown.

Label filters applied on a screen will persist in the application session, so that when switching between parts of the Seq application, label filters are preserved. It's expected that the project-oriented nature of labels will mean users frequently have a set of label filters applied for long periods of time.

### Client library changes

_Seq.Api_ will need to be extended to allow listing and editing of label definitions. Labelled entity types will need to support associations with labels.

`seqcli` may benefit from permitting labels to be specified as filters to various `list` commands, e.g. `seqcli signal list -l online-ordering`.

## Open questions

 * How can a set of signals, such as _Errors_, _Warnings_ and _Exceptions_, be configured to always display even in the presence of other label filters?
 * Should items in the "active" state, such as an applied signal, or the currently-viewed dashboard, remain visible when filtering is applied? If so, do we need any additional visual indication that this is occurring?

## Drawbacks

 * Labelling schemes are very non-prescriptive; it's possible that the flexibility provided by labels will result in the labels themselves being inconsistent and cluttered.
 * Creating large numbers of labels may in itself lead to interface scalability problems.
 * Visual confusion with level indicators (colored dot to the left), signal expressions (blocks of colored text) or signal "tags" (block of colored text) may occur if labels are not given distinctive visual treatment.
 * Room for label display in the signal bar; this is already visually tight/compact - label indicators won't be able to show full label names.

## Alternatives

**Projects:** This proposal grew out of a design for [Projects](#440), which sought to introduce a grouping/hierarchy of these entities along with a permissions/event visibility boundary. The current proposal takes the position that, in future, a more manageable instance-like concept should deliver reliable whole-system partitioning. The usability benefits afforded by labels will continue to be apply within the context of any large single "project".

**Hierarchical groups:** Instead of an arbitrary labeling scheme, a hierarchy could be created, say, projects/sub-projects, etc. This scheme was considered, but discarded because of the limited flexibility for accommodating different styles of organization.
 
**An application-wide label selector:** Rather than using per-list label filters, a single application-wide label filter in the app's title bar was considered. The lack of obvious connection between this and the lists being filtered caused this option to be discarded.

**Item-specific label collections:** 

## References

 * [#440/Projects (enhancement)](#440)
