# Variables

Assign values to names that can be referenced in Seq searches and queries.

## Motivation

Every time I go hunting a bug - one that isn't immediately obvious thanks to some blaring exception, error message, or stack trace - my desk ends up covered in scribbled notes, and my browser window filled with open tabs.

Seq should help organize more of this information, and bear a bit more of the cognitive load. Debugging production issues is difficult and stressful enough, and the more Seq can do to help, the better.

When I'm poking at an issue in a web page, I often assign little snippets of information to variables in the browser's developer tools console, or by clicking an element of interest and assigning its DOM node to a variable that I can poke at in JavaScript.

Life would be made easier if Seq could offer a similar experience for server-side application debugging:

 * Save the id of an interesting HTTP request to e.g. `$suspiciousRequestId`, and then easily return to searching for `RequestId = $suspiciousRequestId`
 * Save a fragment of text like a problematic machine name to e.g. `$slowServer`, and then refer to it multiple times within a query `select count(*) from stream where contains(@Message, $slowServer) or contains(@Exception, $slowServer)`
 * Assign a large/complex property value to a variable, e.g. `$failingRequestBody`, and then interactively examine elements of it, for example `select $failingRequestBody.items[0].description`

A new _Variables_ tool window can list current variable names and their values, avoiding at least some of the scribbling, copying, pasting, and double-handling that I'd otherwise need to do.

## Detailed design

### Variable names

A variable is a named value. In the Seq query language, variable names start with `$`, so that there's no risk of getting incorrect results when the name of a variable conflicts with the name of an event property or column (neither of these things can use names that start with `$`).

Formally, as a regular expression, variables names match the pattern `\$[A-Za-z_0-9]+`.

Examples of variable names:

 * `$a`
 * `$TEST`
 * `$_`
 * `$1`
 * `$event99`

### Variable values

Variables can hold any Seq runtime value. For practical purposes, this means JSON-style `null`, Booleans, numbers, strings, arrays, and objects can all be stored in a variable.

### Variable scope

Variables are defined and used within a single Seq browser window. Variables are not persisted or tracked by the Seq server.

The _Variables_ tool window (see below) provides a simple "copy" function that can be used to replicate a variable from one Seq browser session into another.

### Setting, editing, and clearing variables

#### In the _Variables_ tool window

In the right-hand (signal bar) panel, a new tool window called _Variables_ shows the list of variables, and their values, held in the current browser session.

> Some reorganization of the signal bar is planned so that tool windows can be supported.

The _Variables_ tool window has a "+" button, allowing a new variable to be added, alongside features to edit, remove, and clear current variables.

A user of the Seq interface can click the add button, give the variable a name (`$` prefix is enforced), and choose to give the variable a literal text or numeric value.

Variables in the tool window will provide a context menu with _Copy name_, _Copy value_, and _Copy assignment_, where the assignment is a query language statement that sets the variable value.

#### Direcly, in the query language

Variables can be set in the Seq search box using `select into`:

```sql
select 'Brisbane' into $city
```

Values can be computed using all of Seq's usual operators and functions:


```sql
select OffsetIn('Australia/Brisbane', now()) into $offset
```

The `into` clause can only appear in scalar queries (initially, only those without a `from stream` clause), and only when there is exactly one column in the `select` clause column list.

#### With shortcuts or context menus

Various UI elements will assign values to variables. For example, the green tick drop-down beside property values will include an _Assign to variable_ item that will assign the property's value to a variable.

When one of these UI elements is clicked, the _Variables_ tool window will show (if hidden), and the new variable will appear with an automatically-generated name, derived from the context that provided the value. The user can choose to click and rename the variable, or accept the system-generated name.

UI elements that may include variable assignments:

 * The "event actions" _Type_ menu
 * The green tick context menu on properties, including all levels of JSON value viewer
 * Cells in result set tables (possibly)

### Using variables

#### In searches and queries

Variables can be referred to by name in any search:

```sql
RequestId = $failingRequestId
```

or query:

```sql
select count(*) from stream where EndsWith(Email, $mistypedDomain) ci
```

The Seq search box shows defined variables in auto-complete suggestions.

#### Undefined variables

Any reference to a variable name that has not been defined in the current session is a hard error.

#### In signals, dashboards, and alerts

Variable references cannot be used in signals, dashboards, or alerts. Any variable reference appearing in one of these contexts will be treated as if it is undefined (causing a hard error, as above).

## Drawbacks

### Existing uses of `$` syntax

Early Seq versions used hexadecimal numeric literals like `$00000000` as a shorthand syntax for event type comparisons, today written as `@EventType = 0x00000000`. While Seq has discouraged, and only partially supported, the old syntax for some time, saved signals may still use it. These will need to be automatically migrated when upgrading to a version that uses `$` as a variable name prefix.

It's also possible that some external integrations/API users will make calls with `$` event type comparisons; these will fail once variables are implemented. The likelihood of encountering these cases is judged to be fairly low, however.

## Alternatives

Some use cases for variables are covered by different query/search/result history mechanisms, but these seem complimentary.

## References

_Currently blank_
