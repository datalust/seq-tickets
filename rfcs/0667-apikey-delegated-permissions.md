# API key delegated permissions

Unify access rules between regular users and API keys by switching to a model where API keys are delegated a subset of their owning user's permissions.

## Motivation

The [current API key system](https://docs.getseq.net/docs/api-keys) was designed for ingestion management only, with limited read capability added on later, and shows a lot of quirks because of it.

* API key tokens are treated primarily as identifiers rather than as credentials
* It's not possible to create an API key that can perform admin-level tasks at all
* API keys can only be assigned all-or-nothing user-level access, where more restricted access may be desirable
* Users can't create API keys to perform tasks on their behalf that they're otherwise permitted to perform themselves

In Seq 5 we have the opportunity to modernize and redesign API keys, and to overhaul the underlying permissions model to better support them.

## Detailed design

The proposal has three major parts:

 * User _roles and permissions_ will be introduced so that API keys can have fine-grained user permissions delegated to them
 * API keys will gain the concept of an owner - the user who delegates their permissions to the API key
 * API key storage will switch to a hashed-credentials model

### Usage

The summaries in this section highlight how the major use cases for API keys will be achieved in the new version.

#### API keys for application log ingestion

It's anticipated that API keys for application log ingestion will continue to be created and maintained by a system administrator, as shared (non-personal) API keys. This will prevent interruption of application logging when users come and go.

A user may still create a personal API key and use it to ingest log events if they wish - this essentially provides "self-service" API key creation. Using shared keys for ingestion will be encouraged but not required.

API keys created for ingestion purposes will have the `Ingest` permission assigned: this will be selected by default when creating a new shared API key, as it is by far the most common use case.

#### API keys for reporting/integration

Again, for important reporting and integration tasks, shared API keys created by an administrator will be recommended. However, developers testing an integration or performing analysis may often self-serve API keys for this purpose.

Most reporting and integration tasks should be achievable using an API key with only `Read` and/or `Ingest` permissions - many integration scenarios are possible without requiring a `Write` API key.

#### API keys for scripted system setup and administration

API keys can be used for scripted setup in essentially the same manner as for reporting and integration, but with the `Write` or `Setup` permissions delegated to them.

#### Controlling ingestion rate per API key

When an administrative user browses _Settings > API keys_, by default only shared keys will be shown. By selecting the new _Show personal API keys_ checkbox, the details of API keys owned by all users will be included, showing the ingestion rate for personal as well as shared keys.

Note that this will not reveal the tokens associated with personal API keys, as these will have been hashed and therefore will not be retrievable from storage.

Administrators will be able to view and revoke personal API keys; however, to edit the personal API key of another user, the administrator must at that point make the API key shared, i.e., dissociated from the original owner. This ensures that administrative changes to otherwise personally-controlled entities is readily apparent.

### Roles & permissions

In order for users to delegate permissions to API keys, we must be able to associate permissions with a user. This will be achiveved by assigning to a user zero or more _roles_; each role will confer zero or more _permissions_. The permissions associated with a user's roles are available for delegation to API keys.

Initially, we'll start with a limited set of broad permissions, to simplify migration from Seq 4.x; later versions will extend this set by adding and splitting permissions:

 * **Setup** - access to administrative features of Seq, management of other users, app installation, backups
 * **Write** - lower-privileged write-access to signals, alerts, preferences etc.
 * **Read** - query events, dashboards, signals, app instances
 * **Ingest** - write events to the stream
 * **Public** - access to publicly-visible assets - the API root/metadata, HTML, JavaScript, CSS and so-on

Permissions are hard-coded into the application.

A role associates a user with a set of permissions. The permissions held by a user are the aggregate set of all roles held by the user. Intially there will be the following built-in, fixed roles:

 * **Administrator** - Setup + Write + Read + Ingest (+ Public)
 * **User** - Write + Read + Ingest (+ Public)
 * **Ingestion Key** - Ingest (+ Public)

Again, these are chosen to simplify forward migration, but future versions can extend this set and allow new roles to be defined and customized.

### API key creation

This proposal introduces the notion of an _owner_ of an API key.

 * General system-level keys, created in _Settings > API keys_ will have a `null` owner, and be visible to/managed by, any user with the `Setup` administrative permission
 * Personal API keys, created via the _(Username) > API keys_ preferences panel, will be associated with their owning user for their lifetime

When an API key is created, the set of permissions held by the current user will be presented in a checkbox-list; the user must choose at least one of the permissions to associat with the API key.

Note, this does not grant the specified permissions to the key directly - the permissions are only delegated, so that if the user's role changes or their permissions are revoked through some other action, the API key will no longer have the permissions in question.

### API key storage

When an API key is created, its token will be presented once only; the token will be stored using a cryptographic hash (likely PBKDF2) so that it cannot be retrieved from the server at a later date.

To simplify management, a four- or six-character prefix will be added to the token and stored in plain text. This will enable approximate identification of API keys given a full token and the list of known prefixes in the API keys management screen.

### Migration of existing API keys

It would be nice to move existing API keys to the hashed storage model automatically, however it's likely this will cause API key information to be lost unexpectedly by customers who rely on being able to look up API keys in Seq by token.

Instead, we'll migrate keys as-is, retainig a plain-text token, but use the hashed storage scheme for new keys when they are created. A similar strategy has been employed by other projects (AWS, NuGet.org) when the same change was introduced elsewhere.

### `Seq.Api` client changes

The [_Seq.Api_ library](https://github.com/datalust/seq-api) currently uses API keys for authentication. It's expected that following this change, API-key authenticated access from the client library will continue to work unchanged, and new capabilities enabled by the broader possible permissions will just work.

When used for API key and user provisioning, the client library and apps that use it will need to be updated:

 * `RoleEntity` will need to be added, along with a `WellKnownRoleIds` class/enumeration
 * `UserEntity` gains a `RoleIds` collection, and will have `IsAdministrator` removed
 * `ApiKeyEntity` gains a `DelegatedPermissions` collection, and will have `PermitUserLevelAccess` removed

## Drawbacks

There will be some slight churn introduced by this change, particularly:

 * Customers depending on Seq as a readable store for API keys will need to introduce/use an external credential management system to store them
 * Scripts/applications that use the Seq HTTP API for user account and API key creation and management will need to be updated to use role ids rather than Boolean `UserEnitity.IsAdministrator` and `ApiKeyEntity.PermitUserLevelAccess` properties.

## Alternatives

**Roles:** A further possible role, "System Adminstrator", may exist in reality - this would be the role of the person with full access to the underlying machine who would install/maintain Seq from e.g. the server's local shell. This role is not modelled because it sits outside the Seq application, but it's worth keeping in mind that the Administrator role within Seq may not correspond to a system administration role on the machine.

## References

Current API key model documentation: https://docs.getseq.net/docs/api-keys

C# HTTP API client: https://github.com/datalust/seq-api
