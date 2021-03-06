---
id: authorization
title: Authorization
sidebar_label: Authorization
description: Authorization
keywords:
  - rmuif
  - docs
  - authorization
image: img/illustrations/authorization.png
hide_title: true
---

import useBaseUrl from '@docusaurus/useBaseUrl';

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<div style={{ textAlign: "center" }}>
  <img style={{ width: "75%", marginBottom: "32px" }} alt="Illustration" src={useBaseUrl('img/illustrations/authorization.svg')} />
  <h1>Authorization</h1>
  <p>
    There are mechanisms in place to protect content from unauthorized users.
  </p>
</div>

## Accessibility of roles

Roles are accessible through the user’s authentication token, in the custom claims object. The actual roles is an array of strings, denoting which roles the user has. If the user belongs to the `admin` role, the roles array looks like: `["admin"]`.

In the app, the resolution of roles happens in `App`:

```js
authentication
  .getRoles()
  .then((value) => {
    this.setTheme(data.theme, () => {
      this.setState({
        ready: true,
        user: user,
        userData: data,
        roles: value || [],
      });
    });
  })
  .catch((reason) => {
    this.resetState(() => {
      const code = reason.code;
      const message = reason.message;

      switch (code) {
        default:
          this.openSnackbar(message);
          return;
      }
    });
  });
```

:::note

It exposes the roles of a user in the `roles` state variable, which passes down as a prop to child components.

:::

## Default roles

There are two roles by default: `admin` and `premium`. The `admin` role grants the user unrestricted access to all data in Cloud Firestore, i.e., users with the role can view and manage other users. The `premium` role, however, doesn’t do anything at the moment. It serves as a placeholder for both RMUIF and you. If any premium functionality comes, this role deserves some attention.

## Protecting content

If the `roles` array is accessible, you can check if it contains a role using `includes()`. For example, checking for the `admin` role in `Bar`:

```jsx
{
  roles.includes("admin") && (
    <Box mr={1}>
      <Button color="inherit" component={Link} to="/admin" variant="outlined">
        Admin
      </Button>
    </Box>
  );
}
```

## Protecting routes

You can also use the API to restrict users from accessing a specific route. Take a look at `Router` for the `/admin` route:

```jsx
<Route path="/admin">
  {user && roles.includes("admin") ? <AdminPage /> : <Redirect to="/" />}
</Route>
```

:::note

The `Redirect` part sends the user to the `/` route if the user isn’t signed in and an administrator.

:::

## The `roles` tool

Usually, you would need to set up the Firebase Admin SDK and hack together something that enables you to change the roles of a user on demand. Luckily, for you, we have already done that. Much like the `user` and `users` tool, there exists a `roles` tool. You can use it to manage a user’s roles from the command-line.

### Setting a user’s roles

If you need to set a user’s roles, we’ve got you covered. Although, note that the word _set_ was used instead of _add_. You are **setting** the user’s roles, not **adding** a role, i.e., this command overwrites the user’s existing roles:

<Tabs
groupId="package-managers"
defaultValue="yarn"
values={[
{ label: 'yarn', value: 'yarn' },
{ label: 'npm', value: 'npm' }
]
}>
<TabItem value="yarn">

```sh
yarn roles set <uid> <roles>
```

</TabItem>
<TabItem value="npm">

```sh
npm run roles set <uid> <roles>
```

</TabItem>
</Tabs>

Also, like the `user` tool, you can use the `--email` or `-e` option to use an e-mail address instead of a user ID:

<Tabs
groupId="package-managers"
defaultValue="yarn"
values={[
{ label: 'yarn', value: 'yarn' },
{ label: 'npm', value: 'npm' }
]
}>
<TabItem value="yarn">

```sh
yarn roles -e set <email> <roles>
```

</TabItem>
<TabItem value="npm">

```sh
npm run roles -e set <email> <roles>
```

</TabItem>
</Tabs>

An example of setting the `admin` role for `john`:

<Tabs
groupId="package-managers"
defaultValue="yarn"
values={[
{ label: 'yarn', value: 'yarn' },
{ label: 'npm', value: 'npm' }
]
}>
<TabItem value="yarn">

```sh
yarn roles set john admin
```

</TabItem>
<TabItem value="npm">

```sh
npm run roles set john admin
```

</TabItem>
</Tabs>

By the way, the `roles` argument is comma-separated, i.e., you can set multiple roles at once:

<Tabs
groupId="package-managers"
defaultValue="yarn"
values={[
{ label: 'yarn', value: 'yarn' },
{ label: 'npm', value: 'npm' }
]
}>
<TabItem value="yarn">

```sh
yarn roles set john admin,premium
```

</TabItem>
<TabItem value="npm">

```sh
npm run roles set john admin,premium
```

</TabItem>
</Tabs>

:::warning

As stated before, the `set` command overwrites any existing roles. There might come an update later, adding an `add` command that takes into account any existing roles, but it’s not a priority at the moment. As the saying goes: know what you do before you do.

:::

### Getting a user’s roles

You can also use the `roles` tool to get the existing roles of a user. They can come in handy when using `set`, making you aware of the current roles before you overwrite them. The command is simple and only requires a `uid` (or `email`):

<Tabs
groupId="package-managers"
defaultValue="yarn"
values={[
{ label: 'yarn', value: 'yarn' },
{ label: 'npm', value: 'npm' }
]
}>
<TabItem value="yarn">

```sh
yarn roles get <uid>
```

</TabItem>
<TabItem value="npm">

```sh
npm run roles get <uid>
```

</TabItem>
</Tabs>

Getting the roles of a user `john` would be:

<Tabs
groupId="package-managers"
defaultValue="yarn"
values={[
{ label: 'yarn', value: 'yarn' },
{ label: 'npm', value: 'npm' }
]
}>
<TabItem value="yarn">

```sh
yarn roles get john
```

</TabItem>
<TabItem value="npm">

```sh
npm run roles get john
```

</TabItem>
</Tabs>

Which in turn would yield a result looking like:

```sh
┌───────┬───────┐
│ roles │ admin │
└───────┴───────┘
```

### Removing a user’s roles

You can clear a user’s roles entirely by using the `remove` command. This command sets the `roles` array in the custom claims object to `null`. It looks like the `get` command:

<Tabs
groupId="package-managers"
defaultValue="yarn"
values={[
{ label: 'yarn', value: 'yarn' },
{ label: 'npm', value: 'npm' }
]
}>
<TabItem value="yarn">

```sh
yarn roles remove <uid>
```

</TabItem>
<TabItem value="npm">

```sh
npm run roles remove <uid>
```

</TabItem>
</Tabs>

Cleaning up the `john` user with:

<Tabs
groupId="package-managers"
defaultValue="yarn"
values={[
{ label: 'yarn', value: 'yarn' },
{ label: 'npm', value: 'npm' }
]
}>
<TabItem value="yarn">

```sh
yarn roles remove john
```

</TabItem>
<TabItem value="npm">

```sh
npm run roles remove john
```

</TabItem>
</Tabs>

:::warning

Depending on your setup, removing a user’s roles can make them lose access to protected content. Make sure that this is your intention.

:::
