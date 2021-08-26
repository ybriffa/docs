---
title: User administration
slug: administration-users
section: Administration
order: 11
---

**Last updated 26th August 2021**


## Objective  

Every Web PaaS user has a role that controls access and improves security on your project. Different roles are authorized to do different things with your applications, environments and users. You can use your collection of Roles to manage how users interact with Web PaaS.

Any user added to a project or an environment on Web PaaS will need to [register for an account](https://www.ovh.com/auth/) before they can contribute. 

## User roles

At the project level:

* **Project Administrator** - A project administrator can change settings and execute actions in any environment.
* **Project Viewer** - A project reader can view all environments within a project through the management console but cannot execute any actions on them. Note that a user must be granted access to at least one environment before they can be given project-wide viewer access.

A Project Viewer can have a specific role in different environments. At the environment level:

* **Environment Administrator** - An environment administrator can change settings and execute actions on this environment.
* **Environment Contributor** - An environment contributor can push code to this environment and branch the environment.
* **Environment Viewer** - An environment reader can only view this environment on the management console.

> [!primary]  
> After a user is added to (or deleted from) an environment, it will be automatically redeployed, after which the new permissions will be fully updated.
> 
> When adding users at the **project level**, however, redeployments do not occur automatically, and you will need to trigger redeployments to update those settings for each environment using the CLI command `webpaas redeploy`. Otherwise, user access will not be updated on those environments until after the next build and deploy commit.
> 

Accessing the project through SSH may differ depending on the [configuration of the project or environment](../configuration-app/access).

------------------------------------------------------------------------

When a development team works on a project, the team leader can be the project administrator and decide which roles to give his team members.  One team member can contribute to one environment, another member can administer a different environment and the customer can be a reader of the `master` environment.

If you want your users to be able to see everything (Reader), but only commit to a specific branch, change their permission level on that environment to "Contributor".

> [!primary]  
> An environment contributor can push code to the environment and has SSH access to the environment. You can change this by [specifying user types](../configuration-app/access) with SSH access.
> 

> [!primary]  
> The project owner - the person licensed to use Web PaaS - doesn't have special powers. A project owner usually has a project administrator role.
> 

## Manage user permissions at the project level

From your list of projects, select the project where you want to view or edit user permissions. At this point, you will not have selected a particular environment. Click the Settings tab at the top of the page, then click the `Access` tab on the left to show the project-level users and their roles.

![Project user management screenshot](images/settings-project-access.png)

The `Access` tab shows project-level users and their roles.

Selecting a user will allow you either to edit that user's permissions or delete the user's access to the project entirely.

Add a new user by clicking on the `Add` button.

If you select the "Viewer" role for the user, you'll have the option of adjusting the user's permissions at the environment level.

From this view, you can assign the user's access. Selecting them to become a "Project admin" will give them "Admin" access to every environment in the project. Alternatively, you can give the user "Admin", "Viewer", "Contributor", or "No Access" to each environment separately.

If you select the "Viewer" role for the user, you'll have the option of adjusting the user's permissions at the environment level.

Once this has been done, if the user does not have a Web PaaS account, they will receive an email asking to confirm their details and register an account name and a password.

In order to push and pull code (or to SSH to one of the project's environments) the user will need to add an SSH key.

If the user already has an account, they will receive an email with a link to the project.

## Manage user permissions at the environment level

From within a project select an environment from the `ENVIRONMENT` pull-down menu.

Click the `Settings` tab at the top of the screen and then click the `Access` tab on the left hand side.

![Project user management screenshot](images/settings-environment-access.png)

The `Access` tab shows environment-level users and their roles.

Selecting a user will allow you either to edit that user's permissions or delete the user's access to the environment entirely.

Add a new user by clicking on the `Add` button.

> [!primary]  
> Remember the user will only be able to access the environment once it has been rebuilt (after a `git push`).
> 

## Manage users with the CLI

You can use the Web PaaS command line client to fully manage your users and integrate this with any other automated system.

Available commands:

* `user:add`
  * Add a user to the project
* `user:delete`
  * Delete a user
* `user:list` (`users`)
  * List project users
* `user:role`
  * View or change a user's role

For example, the following command would add the 'admin' role to `alice@example.com` in the current project.

```bash
webpaas user:add
```

This will present you with an interactive wizard that will allow you to choose precisely what rights you want to give the new user.

```bash
$ webpaas user:add

Email address: alice@example.com
The user's project role can be 'viewer' ('v') or 'admin' ('a').
Project role [V/a]:
The user's environment-level roles can be 'viewer', 'contributor', or 'admin'.
development environment role [V/c/a]:
sprint1 environment role [V/c/a]:
hot-fix environment role [V/c/a]:
master environment role [V/c/a]:
pr-2 environment role [V/c/a]:
pr-3 environment role [V/c/a]:
Summary:
    Email address: alice@example.com
    Project role: viewer
    development: viewer
    sprint1: viewer
    hot-fix: viewer
    pr-2: viewer
    pr-3: viewer
Adding users can result in additional charges.
Are you sure you want to add this user? [Y/n]
User alice@example.com created
```

Once this has been done, the user will receive an email asking her to confirm her details and register an account name and a password.

To give Alice the 'contributor' role on the environment 'development', you could run:

```bash
webpaas user:role alice@example.com --level environment --environment development --role contributor
```

Use `webpaas list` to get the full list of commands.

## User access and integrations

If you have setup an [external integration](../integrations-source) to GitHub, GitLab, or Bitbucket, this adds an additional layer of access control to the project that you will need to be aware of. It is, for example, possible that a user that has been given admininstrator privileges to a project on Web PaaS is [unable to clone the project](../administration-web#git) locally if they have not also been given access to the repository on GitHub. 

They could use the CLI

```bash
$ webpaas get <projectID>
```

or the command visible from the "Git" dropdown in the management console

```bash
$ git clone git@github.com:user/github-repo.git Project Name
```

and both would give

```bash
Failed to connect to the Git repository: git@github.com:user/github-repo.git

Please make sure you have the correct access rights and the repository exists.
```

despite their `Admin` access to the project.

This is a good thing, as the project functions as a read-only mirror of your remote repository. Otherwise, changes pushed directly to the project would be overwritten or deleted when commits are pushed via the integration. Web PaaS considers your integrated remote repository to be the "source of truth" as soon as it has been configured, and this caveat ensures that all commits go through the integration.

The best course of action is to have your access updated on the integrated repository. If for some reason that is not a quick change, you can still clone through the project using the legacy pattern (which will set the *project* as its remote), but again, it is not recommended that you commit to the project once you have done so:

```bash
$ git clone <project>@git.<region>.platform.sh:<project>.git
```


