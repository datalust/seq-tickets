# Telemetry Exception Content

RFC #640 describes the opt-in internal error reporting feature implemented in Seq 4.2. This RFC amends the design for error reporting to more robustly exclude potentially-sensitive information.

## Motivation

The original error reporting design includes exception contents, with the following caveat:

> Exception messages are the least-controllable data item here; currently, Seq itself avoids including senstive information in exception messages. By their nature, however, exceptions are unpredictable. Initial implementation of the feature will need to include a review current exception messages, and we will futher need to monitor the content of reported exceptions to limit the amount of user/implementation-specific data that is collected.

The impact of this is mitigated by the creation of a blacklist of excluded exception sources.

After having tested the feature in our internal environment, it's clear that exception content is too variable to be managed this way. Although rare, any component can hypothetically trigger exceptions with local filenames, URLs, account names and so-on in the message. Over the long-term we don't want to accept this risk.

While blacklisting could help, we'd reduce the value of telemetry by blanket exclusion of all exceptions from conservatively-sensitive components. It's possible to crash authentication with a `NullRefrerenceException`, for example, which will never contain sensitive information. Collecting these kinds of exceptions, when the administrator opts in, may actually help improve the security of Seq and is worth persevering with.

The proposal made in this RFC is to:

1. Stop collecting exception messages entirely, instead recording only the exception type and stack trace information, and
1. Abandon the component blacklisting mechanism.

By collecting exception information more conservatively, we remove almost all possible mechanisms of information leakage while extending the usefulness of the collected error information.

## Detailed design

### Blanket exception formatting

The change will replace the default `ToString()` collection of exception details by an equivalent formatting function that does not include exception messages, except their first five characters.

E.g. the following excerpted text, which leaks the feed URI:

```
System.InvalidOperationException: An error occurred while loading packages from'https://www.nuget.org/api/v2d/': The remote server returned an error: (404) Not Found. ---> System.Net.WebException: The remote server returned an error: (404) Not Found.
   at System.Net.HttpWebRequest.GetResponse()
   at NuGet.RequestHelper.GetResponse(Func`1 createRequest, Action`1 prepareRequest, IProxyCache proxyCache, ICredentialCache credentialCache, ICredentialProvider credentialProvider)
   at NuGet.HttpClient.GetResponse()
   at NuGet.RedirectedHttpClient.GetResponseUri(HttpClient client)
   at NuGet.RedirectedHttpClient.EnsureClient()
```

will become:

```
System.InvalidOperationException: An er... ---> System.Net.WebException: The r...
   at System.Net.HttpWebRequest.GetResponse()
   at NuGet.RequestHelper.GetResponse(Func`1 createRequest, Action`1 prepareRequest, IProxyCache proxyCache, ICredentialCache credentialCache, ICredentialProvider credentialProvider)
   at NuGet.HttpClient.GetResponse()
   at NuGet.RedirectedHttpClient.GetResponseUri(HttpClient client)
   at NuGet.RedirectedHttpClient.EnsureClient()
```

_Why five characters? It's unlikely any exception-specific information will be present in the first few characters of the exception message string, and even if usage-specific data are present, five characters is unlikley to represent anything substantial. It may, however, be enough for us to narrrow down the cause of an exception with a general type.

### Whitelisted information

Where specific exception types can be formatted with known-safe fields, such as [`ArgumentNullException.ParamName`](https://msdn.microsoft.com/en-us/library/system.argumentexception.paramname(v=vs.110).aspx), these may be included in the replacement message.

A non-exhaustive list of such fields is:

* `ArgumentException.ParamName`
* `Exception.HResult`
* `WebException.Status`

This list may be extended over time as useful data are identified.

### Blacklist removal

The entire component blacklisting mechanism proposed in #640 will be removed.

## Drawbacks

This change will make it harder to immediately diagnose the cause of an exception, or distinguish between closely-related exception types.

## Alternatives

We could stick with the status quo and pursue blacklisting, however this seems unwise.

## References

[The original error reporting RFC](https://github.com/datalust/seq-tickets/blob/master/rfcs/0640-internal-error-reporting.md)
