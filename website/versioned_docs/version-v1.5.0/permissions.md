---
id: version-v1.5.0-permissions
title: Permissions
original_id: permissions
---
    
[alanning:roles](https://github.com/alanning/meteor-roles) package provides Reaction permissions support.

## Packages

**Permissions are grouped by `shopId`.**

Package specific roles can be defined in `register.js`, by adding custom permissions to registry entries with:

```js
  permissions: [
    {
      label: "Custom Permission"
      permission: "custom/permission"
    }
  ]
```

Permission of the current route and user are compared against the package route by default, adding specific permissions to the registry entry is optional.

For using shop permissions in some packages you must add it into register directive. If we add this package then permissions will be available in Shop Accounts Settings.

Another example:

```js
import { Reaction } from "/server/api";

Reaction.registerPackage({
  label: "Dashboard",
  name: "reaction-dashboard",
  icon: "fa fa-th",
  autoEnable: true,
  settings: {
    name: "Dashboard"
  },
  registry: [{
    provides: "dashboard",
    workflow: "coreDashboardWorkflow",
    template: "dashboardPackages",
    name: "dashboardPackages",
    label: "Core",
    description: "Reaction core shop configuration",
    icon: "fa fa-th",
    priority: 0,
    container: "core",
    permissions: [{
      label: "Dashboard",
      permission: "dashboard"
    }]
   });
```

At the point where Packages are published in the app, each registry item permissions are collected and put on the
package registry [(source)](https://github.com/reactioncommerce/reaction/blob/master/server/publications/collections/packages.js#L31-L56).
Based on these permissions, we can enable or disable functionality depending on user roles.

## Owner

The initial setup user was added to the shops 'owner' permission group with the 'owner' permission.

Users with "owner" role are full-permission, app-wide users.

**To check if user has owner access:**

```js
    # client / server
    import { Logger, Reaction } from "/server/api";

    if ( Reaction.hasOwnerAccess() ) {
      Logger.info("The Reaction user has Owner Access");
    }
```

`/client/modules/core/helpers/templates.js` exports the `hasOwnerAccess` helper.

```html
    # template
    {{#if hasOwnerAccess}}
      <strong>This has owner access</strong>
    {{/if}}
```

## Admin

Users with "admin" role are full-permission, site-wide users.

**To check if user has admin access:**

```js
// client / server
import { Logger, Reaction } from "/server/api";

if (Reaction.hasAdminAccess()) {
  Logger.info("The Reaction user has Admin Access");
}
```

`/client/modules/core/helpers/templates.js` exports the `hasAdminAccess` helper.

```handlebars
<!-- template -->
{{#if hasAdminAccess}}
  <strong>This has admin access</strong>
{{/if}}
```

## Dashboard

Users with "dashboard" role are limited-permission, site-wide users.

**To check if user has Dashboard access:**

```js
// client
import { Logger, Reaction } from "/client/api";
// server
import { Logger, Reaction } from "/server/api";

if (Reaction.hasDashboardAccess()) {
  Logger.info("The Reaction user has Owner Access");
}
```

`/client/modules/core/helpers/templates.js` exports the `hasDashboardAccess` helper.

```handlebars
{{#if hasDashboardAccess}}
  <strong>This has dashboard access</strong>
{{/if}}
```

To check if user has some specific permissions:

## hasPermission

Client

```js
// client
import { Reaction } from "/client/api";

// can be a String or Array of strings
const permissions = ["guest", "profile"];

Reaction.hasPermission(permissions);
```

Server

Uses the current shop and current user if either is not defined

```js
// server
import { Reaction } from "/server/api";

// can be a String or Array of strings
const permissions = ["guest", "profile"];

Reaction.hasPermission(permissions, shop, userId);
```

## hasPermission helper

Helpers in template in templates:

```html
{{#if hasPermission permissions}}
  <strong> has permission </strong>
{{/if}}
```

`/client/modules/core/helpers/permissions.js` exports the `hasPermission` helper.

## Reaction.Apps()

This core helper method gets all package apps that match the filter passed in. You can use this as in the example below to
get all enabled packages for payments.

```js
  Reaction.Apps({
    provides: "paymentMethod",
    enabled: true
  });
```

You can also pass in an `audience` field to filter returned apps based on assigned roles for the user.
[(source)](https://github.com/reactioncommerce/reaction/blob/master/client/modules/core/helpers/apps.js#L106-L127)

```js
  Reaction.Apps({
    provides: "settings",
    enabled: true,
    audience: Roles.getRolesForUser(Meteor.userId(), Reaction.getShopId())
  })
```
