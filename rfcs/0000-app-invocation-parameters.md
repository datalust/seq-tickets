# App invocation parameters

Allow specific app parameters, like a destination email address or Slack channel, to be set when configuring an alert instead of when configuring the app instance.

## Motivation

Alerts use _Seq apps_ to send notifications. To do that, an _instance_ of the app needs to be set up, with information like destination addresses, URLs, usernames and passwords provided by a Seq administrator.

```
Alert #1 [count > 1] ---> Email app instance [ops@example.com, smtp.example.com, pa$$word]
                             ^
                             |
Alert #2 [count < 5] --------+
```

This effectively prevents non-administrative users from making full use of alerts, since "alert-specific" app configuration, like a recipient email address, is bundled up with inaccessible infrastructure details like SMTP server hostnames and passwords.

By marking specific app settings as user-configurable, this feature will make it possible for (unprivileged) users to set up their own alerts, and reduce the amount of duplicated app instance configuration.

```
Alert #1 [count > 1, user1@example.com] ---> Email app instance [_, smtp.example.com, pa$$word]
                                                ^
                                                |
Alert #2 [count < 5, user2@example.com] --------+
```

## Detailed design

As implemented today, apps expose _available settings_, which each carry a unique name (like `"EmailAddress"`), an input type (e.g. `Text`) and other details. When an app instance is created, it is given a dictionary of _settings values_, keyed by setting name. For example, `("EmailAddress", "user@example.com")`.

The design proposes that app instances carry a new list of _invocation parameters_. For each app setting, the setting name will **either** be given a setting value, **or** appear in the instance's invocation parameters. An app instance that has one or more invocation parameters is said to be _parameterized_.

From a user's perspective, parameterized app instances are just normal app instances, but they behave slightly differently from existing apps.

1. When a notification is configured to use a parameterized app instance, values must be provided for the instance's invocation parameters
1. Likewise, when sending an event manually to a parameterized app instance, invocation parameter values must be supplied
1. A parameterized app instance cannot receive events directly from the stream

### App instance configuration

There are two requirements imposed by this feature:

1. Parameterized app instances can't be configured to receive events directly from the stream
1. Individual settings need to be marked as invocation parameters

Rather than use the presence of invocation parameters (2.) to drive the application of rule (1.), this design proposes that we allow parameterization to occur only when the app is marked as "manual input only". In 4.2, this setting is under the _Input_ heading, and called _Stream incoming events_.

This adds additional semantics to the existing field, but these are compatible with the current behavior of this setting.

The setting label and help text will be:

> **Stream incoming events**
> App instances can be triggered by alerts set on dashboards, and events can be sent manually to any app from the drop-down menu in the event details. Check this box to send events as they arrive.

Initially, the checkbox is unchecked, so the app instance will have parameterization enabled. If the user checks the checkbox, the option to mark settings as invocatoin parameters (further down the screen) will be greyed out and show an explanatory tool-tip _App instances that receive events directly from the stream cannot be parameterized_.

Under the _Settings_ heading, each setting will gain an additional icon-style button to the right of its input control. Pressing this button will toggle the setting between being a regular setting, and being an input parameter.

In the input parameter state, the setting's input control will be replaced with static text stating that the parameter value will be supplied when the app is invoked.

### Alert configuration

When an alert is configured, selecting a parameterized app instance will show the input controls for the invocation parameters.

**Important:** special handling is required for the `Password` setting input type, which is never echoed back from the server, and must be stored using master key encryption in the server's database.

### Manual send-to-app

In the events screen, if _Send to app_ is selected and a paramerized app instance is chosen, a pop-up dialog will display the input controls for invocation parameters.

To maintain a consistent user experience, we may wish to show this dialog (with OK/Cancel buttons) even when no invocation parameters are required. This would also provide an opportunity to advise the user that app activation is asynchronous.

### Edge cases and error conditions

* A missing or invalid parameter value will be handled in the same way as a missing or invalid setting

### API client changes

The _Seq.Api_ package will need to be updated with new types/fields/methods.

## Drawbacks

* There is a mild inconsistency between how parameterized apps are configured and used, and the way that current apps in-the-wild use event properties to emulate invocation parameters. For example, the _Seq.App.EmailPlus_ app accepts property placeholders in its `Subject` setting: a subject might be `"Error raised in {Environment}"`, where `Environment` is an event property. This mode of parameterization is different from marking the `Subject` property as an invocation parameter. This is accepted as a trade-off in the design.
* Creation of an app instance per invocation may be very expensive (order of seconds). We'll need to mitigate the risk of performance problems by ensuring a) that invocations remain asynchronous and parallel, allowing load to be spread out, and b) no features stream events directly to parameterized app instances. The design and use of alert notifications already discourages high-frequency alerts.

## Alternatives

The major alternative to this design is to use some kind of templating/binding scheme for parameterization, with invocation parameters pulled from the events themselves. This would have the advantage of being usable with direct event streaming, but this would bring with it the need to extend the plug-in app mechanism so that a single running instance could efficiently handle a stream of events with different parameter values. Any design that requires opt-in from the app ecosystem will be much slower to realize benefit from.

## References

_Left blank._
