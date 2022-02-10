# Motivation

The `Administrator` role currently combines both project administration functionality and system administration functionality. Giving a user access to do things like delete a signal or create a shared API key cannot be done without also giving them access to do things like installing apps and modifying the Seq license. 

The Seq permissions system currently has four permissions, Read, Write, Setup, and Ingest. These are allocated to four built in roles, Administrator, User (read-only), User (read/write) and User (read/write/ingest).

The Administrator role currently encompasses both project administration, and host system access, through functionality such as Seq App installation.

This RFC proposes splitting the Setup permission into Project (project administration) and System (host system administration), so that these two responsibilities can be separated. Team leaders often need access to project administration features such as API keys and retention policies, but don't necessarily need privileged access to the host system.

The built in Administrator role will include both Project and System, i.e., retaining the functionality of this role today. A new role Project Owner will have Project permissions, but not System.

# Summary

The `System` permission is intended to cover things relating to the Seq installation, such as app feeds, app installation, diagnostics, system settings, data privacy, licensing, backups, and cluster configuration.

The `Project` permission is intended to cover the settings required to configure Seq for a set of users, such as users, and retention policies.

# Detailed Design

## Roles to Permission Mapping

| Role | 2021.4 permissions | 2022.1 permissions |
| -- | -- | -- |
| User (read-only) | Read | Read |
| User (read/write) | Read, Write | Read, Write |
| User (read/write/ingest) | Read, Write, Ingest | Read, Write, Ingest |
| Project Owner | | Read, Write, Ingest, Project |
| Administrator | Read, Write, Ingest, Setup | Read, Write, Ingest, Project, System |

## Permission to API Endpoint Changes

Changes are highlighted. 

### `api/alerts` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/alerts/`  | `GET`  | `Read`  | Users can only read shared alerts, and those that they own. |  |
| `api/alerts/`  | `POST`  | `Write`  | _**Users can only create shared alerts or personally owned ones. The additional `Project` permission is required to create protected alerts.**_ |  |
| `api/alerts/{id}`  | `DELETE`  | `Write`  | _**Users can only remove shared alerts and those that they own. The additional `Project` permission is required to remove protected alerts.**_ |  |
| `api/alerts/{id}`  | `GET`  | `Read`  | Users can only read shared alerts, and those that they own. |  |
| `api/alerts/{id}`  | `PUT`  | `Write`  | _**Users can only update shared alerts and those that they own. The additional `Project` permission is required to update protected alerts.**_ |  |
| `api/alerts/resources`  | `GET`  | `Public`  |  |  |
| `api/alerts/template`  | `GET`  | `Write`  |  |  |

### `api/alertstate` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/alertstate/`  | `GET`  | _**`Project`**_  |  |  |
| `api/alertstate/{id}`  | `DELETE`  | _**`Project`**_  |  |  |
| `api/alertstate/{id}`  | `GET`  | _**`Project`**_  |  |  |
| `api/alertstate/resources`  | `GET`  | `Public`  |  |  |

### `api/apikeys` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/apikeys/`  | `GET`  | `Read`  | _**Only `Project` principals can view all keys; others can only request and view keys that they own.**_ |  |
| `api/apikeys/`  | `POST`  | `Write`  | _**A principal can only set the owner of a key to themselves. Only `Project` principals can create keys without an owner. Principals can only delegate permissions to a key that they themselves hold.**_ |  |
| `api/apikeys/{id}`  | `DELETE`  | `Write`  | _**Only `Project` principals can remove any key; others can only remove keys that they own.**_ |  |
| `api/apikeys/{id}`  | `GET`  | `Read`  | _**Only `Project` principals can view all keys; others can only request and view keys that they own.**_ |  |
| `api/apikeys/{id}`  | `PUT`  | `Write`  | _**A principal can only set the owner of a key to themselves. Only `Project` principals can create keys without an owner. Principals can only delegate permissions to a key that they themselves hold.**_ |  |
| `api/apikeys/{id}/metrics/{measurement}`  | `GET`  | `Read`  | _**Only `Project` principals can view metrics for all keys; others can only request and view metrics for keys that they own.**_ |  |
| `api/apikeys/resources`  | `GET`  | `Public`  |  |  |
| `api/apikeys/template`  | `GET`  | `Read`  |  |  |

### `api/appinstances` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/appinstances/`  | `GET`  | `Write`  | Principals that can invoke apps can access the basic details of available app instances. |  |
| `api/appinstances/`  | `POST`  | _**`System`**_  |  |  |
| `api/appinstances/{id}`  | `DELETE`  | _**`System`**_  |  |  |
| `api/appinstances/{id}`  | `GET`  | `Write`  | Principals that can invoke apps can access the basic details of available app instances. |  |
| `api/appinstances/{id}`  | `PUT`  | _**`System`**_  |  |  |
| `api/appinstances/{id}/icon`  | `GET`  | `Write`  | Principals that can invoke apps can access the basic details of available app instances. |  |
| `api/appinstances/{id}/invoke`  | `POST`  | `Write`  | The app instance `id` must allow manual input. |  |
| `api/appinstances/{id}/metrics/{measurement}`  | `GET`  | _**`System`**_  |  |  |
| `api/appinstances/resources`  | `GET`  | `Public`  |  |  |
| `api/appinstances/template`  | `GET`  | _**`System`**_  |  |  |

### `api/apps` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/apps/`  | `GET`  | _**`System`**_  |  |  |
| `api/apps/`  | `POST`  | _**`System`**_  |  |  |
| `api/apps/{id}`  | `DELETE`  | _**`System`**_  |  |  |
| `api/apps/{id}`  | `GET`  | _**`System`**_  |  |  |
| `api/apps/{id}`  | `PUT`  | _**`System`**_  |  |  |
| `api/apps/{id}/icon`  | `GET`  | _**`System`**_  |  |  |
| `api/apps/{id}/update`  | `POST`  | _**`System`**_  |  |  |
| `api/apps/install`  | `POST`  | _**`System`**_  |  |  |
| `api/apps/resources`  | `GET`  | `Public`  |  |  |
| `api/apps/template`  | `GET`  | _**`System`**_  |  |  |

### `api/backups` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/backups/`  | `GET`  | _**`System`**_  |  |  |
| `api/backups/{id}`  | `GET`  | _**`System`**_  |  |  |
| `api/backups/files/{filename}`  | `GET`  | _**`System`**_  |  |  |
| `api/backups/immediate`  | `POST`  | _**`System`**_  |  |  |
| `api/backups/resources`  | `GET`  | `Public`  |  |  |

### `api/clusternodes` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/clusternodes/`  | `GET`  | _**`System`**_  |  |  |
| `api/clusternodes/{id}`  | `GET`  | _**`System`**_  |  |  |
| `api/clusternodes/{id}/demote`  | `POST`  | _**`System`**_  |  |  |
| `api/clusternodes/resources`  | `GET`  | `Public`  |  |  |

### `api/dashboards` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/dashboards/`  | `GET`  | `Read`  | Users can only read shared dashboards, and those that they own. |  |
| `api/dashboards/`  | `POST`  | `Write`  | _**Users can only create shared dashboards or personally owned ones. The additional `Project` permission is required to create protected dashboards.**_ |  |
| `api/dashboards/{id}`  | `DELETE`  | `Write`  | _**Users can only remove shared dashboards and those that they own. The additional `Project` permission is required to remove protected dashboards.**_ |  |
| `api/dashboards/{id}`  | `GET`  | `Read`  | Users can only read shared dashboards, and those that they own. |  |
| `api/dashboards/{id}`  | `PUT`  | `Write`  | _**Users can only update shared dashboards and those that they own. The additional `Project` permission is required to update protected dashboards.**_ |  |
| `api/dashboards/query/template`  | `GET`  | `Write`  |  |  |
| `api/dashboards/resources`  | `GET`  | `Public`  |  |  |
| `api/dashboards/template`  | `GET`  | `Write`  |  |  |

### `api/diagnostics` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/diagnostics/ingestion`  | `GET`  | _**`System`**_  |  |  |
| `api/diagnostics/metrics`  | `GET`  | _**`System`**_  |  |  |
| `api/diagnostics/metrics/{measurement}`  | `GET`  | _**`System`**_  |  |  |
| `api/diagnostics/report`  | `GET`  | _**`System`**_  |  |  |
| `api/diagnostics/resources`  | `GET`  | `Public`  |  |  |
| `api/diagnostics/status`  | `GET`  | `Read`  |  |  |
| `api/diagnostics/storage`  | `GET`  | _**`System`**_  |  |  |

### `api/events` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/events/`  | `GET`  | `Read`  |  |  |
| `api/events/{id}`  | `GET`  | `Read`  |  |  |
| `api/events/raw/`  | `POST`  | `Public`  | If the `RequireApiKeyForWritingEvents` setting is enabled, requests must be authenticated and have the `Ingest` permission. |  |
| `api/events/resources`  | `GET`  | `Public`  |  |  |
| `api/events/signal`  | `DELETE`  | _**`Project`**_  |  |  |
| `api/events/signal`  | `GET`  | `Read`  |  |  |
| `api/events/signal`  | `POST`  | `Read`  |  |  |
| `api/events/signal/{signalId}`  | `GET`  | `Read`  |  | **Obsolete**  |
| `api/events/tabulate`  | `POST`  | `Read`  |  |  |
| `api/events/tabulate/{signalId}`  | `GET`  | `Read`  |  |  |


### `api/feeds` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/feeds/`  | `GET`  | _**`System`**_  |  |  |
| `api/feeds/`  | `POST`  | _**`System`**_  |  |  |
| `api/feeds/{id}`  | `DELETE`  | _**`System`**_  |  |  |
| `api/feeds/{id}`  | `GET`  | _**`System`**_  |  |  |
| `api/feeds/{id}`  | `PUT`  | _**`System`**_  |  |  |
| `api/feeds/resources`  | `GET`  | `Public`  |  |  |
| `api/feeds/template`  | `GET`  | _**`System`**_  |  |  |

### `api/licenses` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/licenses/`  | `GET`  | _**`System`**_  |  |  |
| `api/licenses/{id}`  | `GET`  | `Read`  | `Read` principals can see license status but _**`System`**_ is required for certificate details. |  |
| `api/licenses/{id}`  | `PUT`  | _**`System`**_  |  |  |
| `api/licenses/downgrade`  | `POST`  | _**`System`**_  |  |  |
| `api/licenses/resources`  | `GET`  | `Public`  |  |  |

### `api/permalinks` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/permalinks/`  | `GET`  | `Read`  | Non-_**`Project`**_ principals can only view their own permalinks. |  |
| `api/permalinks/`  | `POST`  | `Write`  | Non-_**`Project`**_ principals can only create permalinks for themselves. |  |
| `api/permalinks/{id}`  | `DELETE`  | `Write`  | Non-_**`Project`**_ principals can only remove their own permalinks. |  |
| `api/permalinks/{id}`  | `GET`  | `Read`  | Non-_**`Project`**_ principals can only retrieve their own permalinks. |  |
| `api/permalinks/resources`  | `GET`  | `Public`  |  |  |
| `api/permalinks/template`  | `GET`  | `Write`  |  |  |

### `api/retentionpolicies` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/retentionpolicies/`  | `GET`  | _**`Project`**_  |  |  |
| `api/retentionpolicies/`  | `POST`  | _**`Project`**_  |  |  |
| `api/retentionpolicies/{id}`  | `DELETE`  | _**`Project`**_  |  |  |
| `api/retentionpolicies/{id}`  | `GET`  | _**`Project`**_  |  |  |
| `api/retentionpolicies/{id}`  | `PUT`  | _**`Project`**_  |  |  |
| `api/retentionpolicies/resources`  | `GET`  | `Public`  |  |  |
| `api/retentionpolicies/template`  | `GET`  | _**`Project`**_  |  |  |

### `api/runningtasks` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/runningtasks/`  | `GET`  | _**`System`**_  |  |  |
| `api/runningtasks/{id}`  | `DELETE`  | _**`System`**_  |  |  |
| `api/runningtasks/{id}`  | `GET`  | _**`System`**_  |  |  |
| `api/runningtasks/resources`  | `GET`  | `Public`  |  |  |

### `api/settings` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/settings/{id}`  | `GET`  | _**`System`**_  |  |  |
| `api/settings/{id}`  | `PUT`  | _**`System`**_  |  |  |
| `api/settings/internal-error-reporting`  | `GET`  | _**`System`**_  |  |  |
| `api/settings/internal-error-reporting`  | `PUT`  | _**`System`**_  |  |  |
| `api/settings/resources`  | `GET`  | `Public`  |  |  |
| `api/settings/setting-authenticationprovider`  | `GET`  | `Public`  |  |  |
| `api/settings/setting-instancetitle`  | `GET`  | `Public`  |  |  |
| `api/settings/setting-isactivedirectoryauthentication`  | `GET`  | `Public`  |  |  |
| `api/settings/setting-isauthenticationenabled`  | `GET`  | `Public`  |  |  |

### `api/signals` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/signals/`  | `GET`  | `Read`  | Users can only read shared signals, and those that they own. |  |
| `api/signals/`  | `POST`  | `Write`  | Users can only create shared signals or personally owned ones. The additional _**`Project`**_ permission is required to create protected signals. |  |
| `api/signals/{id}`  | `DELETE`  | `Write`  | Users can only remove shared signals and those that they own. The additional _**`Project`**_ permission is required to remove protected signals. |  |
| `api/signals/{id}`  | `GET`  | `Read`  | Users can only read shared signals, and those that they own. |  |
| `api/signals/{id}`  | `PUT`  | `Write`  | Users can only update shared signals and those that they own. The additional _**`Project`**_ permission is required to update protected signals. |  |
| `api/signals/resources`  | `GET`  | `Public`  |  |  |
| `api/signals/template`  | `GET`  | `Write`  |  |  |

### `api/sqlqueries` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/sqlqueries/`  | `GET`  | `Read`  | Users can only read shared SQL queries, and those that they own. |  |
| `api/sqlqueries/`  | `POST`  | `Write`  | Users can only create shared SQL queries or personally owned ones. The additional _**`Project`**_ permission is required to create protected SQL queries. |  |
| `api/sqlqueries/{id}`  | `DELETE`  | `Write`  | Users can only remove shared SQL queries and those that they own. The additional _**`Project`**_ permission is required to remove protected SQL queries. |  |
| `api/sqlqueries/{id}`  | `GET`  | `Read`  | Users can only read shared SQL queries, and those that they own. |  |
| `api/sqlqueries/{id}`  | `PUT`  | `Write`  | Users can only update shared SQL queries and those that they own. The additional _**`Project`**_ permission is required to update protected SQL queries. |  |
| `api/sqlqueries/resources`  | `GET`  | `Public`  |  |  |
| `api/sqlqueries/template`  | `GET`  | `Write`  |  |  |

### `api/updates` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/updates/`  | `GET`  | _**`System`**_  |  |  |
| `api/updates/{id}`  | `GET`  | _**`System`**_  |  |  |
| `api/updates/resources`  | `GET`  | `Public`  |  |  |

### `api/users` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/users/`  | `GET`  | `Project`  | _**Users can retrieve their own records. Only `Project` principals can retrieve other principals.**_ |  |
| `api/users/`  | `POST`  | `Project`  | _**The `System` permission is required to create users with the `System` permission**_ |  |
| `api/users/{id}`  | `DELETE`  | `Project`  | _**The `System` permission is required to delete users with the `System` permission**_ |  |
| `api/users/{id}`  | `GET`  | `Public`  | _**Users can retrieve their own records. Only `Project` principals can retrieve other principals.**_ |  |
| `api/users/{id}`  | `PUT`  | `Write`  | _**Users can update limited fields on their own records. Only `Project` principals can update other principals. Only `System` principals can update other principals with the `System` permission.**_ |  |
| `api/users/{id}/searches`  | `GET`  | `Read`  | Only the principal's own search history can be retrieved. |  |
| `api/users/{id}/searches/update`  | `POST`  | `Write`  | Only the principal's own search history can be updated. |  |
| `api/users/{id}/unlinkauthenticationprovider`  | `POST`  | _**`System`**_  |  |  |
| `api/users/current`  | `GET`  | `Public`  | Can only return logged-in user information. |  |
| `api/users/login`  | `POST`  | `Public`  |  |  |
| `api/users/logout`  | `POST`  | `Public`  |  |  |
| `api/users/providers`  | `GET`  | `Public`  |  |  |
| `api/users/resources`  | `GET`  | `Public`  |  |  |
| `api/users/template`  | `GET`  | _**`Project`**_  |  |  |

### `api/workspaces` 

| Path | HTTP method | Permission demand | Additional requirements | Notes |
| --- | --- | --- | --- | --- |
| `api/workspaces/`  | `GET`  | `Read`  | Users can only read shared workspaces, and those that they own. |  |
| `api/workspaces/`  | `POST`  | `Write`  | _**Users can only create shared workspaces or personally owned ones. The additional `Project` permission is required to create protected workspaces.**_ |  |
| `api/workspaces/{id}`  | `DELETE`  | `Write`  | _**Users can only remove shared workspaces and those that they own. The additional `Project` permission is required to remove protected workspaces.**_ |  |
| `api/workspaces/{id}`  | `GET`  | `Read`  | Users can only read shared workspaces, and those that they own. |  |
| `api/workspaces/{id}`  | `PUT`  | `Write`  | _**Users can only update shared workspaces and those that they own. The additional `Project` permission is required to update protected workspaces.**_ |  |
| `api/workspaces/resources`  | `GET`  | `Public`  |  |  |
| `api/workspaces/template`  | `GET`  | `Write`  |  |  |

## Breaking Changes

The API will no longer allow creation of API keys with the `Setup` permission. Instead it will respond with a message indicating that the `Setup` permission is no longer valid. 

## Migration

### Users

Administrators will continue to be administrators. The Administrator role will gain the `Project` and `System` permissions.

### API Keys

API keys with the `Setup` permission will gain the `Project` and `System` permissions.

## Backwards Compatibility

To be impacted by this change an installation mostly needs to opt-in by allocating some users to the new `Project Owner` role. Existing `Administrators` will get the new `Project` permission and so will not notice a change. 

API keys are created, via the UI and API, with a subset of the set of permissions. Thus any existing processes that create API keys with the `Setup` permission will fail with a helpful error message (see Breaking Changes). 

## UI Changes

### API Keys

The API keys create and edit views will need to be changed to show the set of possible permissions as 'Read', 'Write', 'Ingest', 'Project', 'System'. 

### Users

The add/edit user and new user defaults views will need to reflect the additional user role `Project Owner`. 

## Seq API

[Seq API](https://github.com/datalust/seq-api) will need to be updated with the new permission and role.

## `seqcli` Changes

[Seq CLI](https://github.com/datalust/seqcli) will need to be updated with the new permission and role. Possible just a documentation change. 

# Future Directions

NA.

# Drawbacks & Limitations

1. Keeping Seq instances running requires migrating all existing administrators to the `Project` + `System` permissions, potentially giving them more access than they need. To get the benefit of the new role and new permissions administrators will need to reallocate user roles and API key permissions. This limitation does not apply to new Seq installations.

# Alternatives

An alternative approach is to make roles configurable. This would require many more permissions, which the `Administrator` could then map to whatever roles they wish. This would be more flexible, but also more complicated. 

# References