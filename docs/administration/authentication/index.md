---
title: Authentication Providers
description: Authentication options for Octopus Deploy including our internal provider, Active Directory, Azure AD, Okta, and GoogleApps.
position: 50
---

Starting from **Octopus 3.5**, Octopus Deploy supports the most common authentication providers out-of-the-box, including special support for a Guest Login.

- [Active Directory Authentication](/docs/administration/authentication/active-directory-authentication/index.md)
- [Azure Active Directory Authentication](/docs/administration/authentication/azure-ad-authentication.md)
- [GoogleApps Authentication](/docs/administration/authentication/googleapps-authentication.md)
- [Okta Authentication](/docs/administration/authentication/okta-authentication.md)
- [Octopus ID](octopusid-authentication.md)
- [Guest Login](/docs/administration/authentication/guest-login.md)

## Configuring Authentication Providers {#AuthenticationProviders-ConfiguringAuthenticationProviders}

You can use the web user interface to configure authentication providers at {{Configuration>Settings}}.

Alternatively, you can configure authentication providers using the `Octopus.Server.exe configure` command-line interface.

## Signing in for the First Time

If you are using the UsernamePassword provider, you will need to invite your team to use Octopus. 

When a user signs in to Octopus for the first time using an external authentication provider, Octopus will automatically create a new user account for them as a convenience. If you prefer to control which users can access Octopus, you can disable auto user creation and manually invite users instead.

Learn about [managing users and teams](/docs/administration/managing-users-and-teams/index.md).

Learn about [auto user creation](/docs/administration/authentication/auto-user-creation.md).

## Managing Teams

In Octopus, you can group your users into teams and use the role-based permission system to control what they can see and do. Learn about [managing users and teams](/docs/administration/managing-users-and-teams/index.md).

You can manually manage the members of your teams, or you can configure certain external authentication providers to manage your teams for you automatically.

- Learn about [automatically managing teams with Active Directory](/docs/administration/authentication/active-directory-authentication/index.md).
- Learn about [automatically managing teams with Azure Active Directory](/docs/administration/authentication/azure-ad-authentication.md).
- Learn about [automatically managing teams with Okta](/docs/administration/authentication/azure-ad-authentication.md).

## Auto Login {#AuthenticationProviders-AutoLogin}

When using an external authentication provider, you can configure Octopus to work in one of two ways:

1. Make the user click a button on the Octopus login screen.
2. Automatically sign the user in by redirecting to the external identity provider.

Auto login is **disabled by default**, and you can enable it in {{Configuration>Settings>Authentication>Auto Login}}.

Note that even when enabled, **this functionality is only active when there is a single, non forms-based authentication provider enabled**.  If multiple providers are enabled, which includes Guest access being enabled, this setting is overridden.

### Auto Login and Active Directory

When using the Active Directory provider, auto login will only be active when the {{Configuration>Settings>Active Directory>Allow Forms Authentication For Domain Users}} setting is **false**.

## Associating Users With Multiple External Identities {#AuthenticationProviders-usersandauthprovidersUsersandAuthenticationProviders}

In versions up to 3.5, only a single Authentication Provider could be enabled at a time (either Domain or UsernamePassword).  In that scenario Users were managed based on the currently enabled provider and switching providers meant re-configuring Users.  With 3.5 comes the ability to have multiple Authentication Providers enabled simultaneously and as such the User management has been adjusted to be provider agnostic.  What does that mean?  Let's consider an example scenario.

Let's consider that we have UsernamePassword enabled and we create some users, and we've set their email address to their Active Directory domain email address.  The users can now log in with the username and password stored against their user record.  If we now enable the Active Directory authentication provider, then the users can authenticate using either their original username and password, or they can use a username of user@domain or domain\user along with their domain password, or they can use the Integrated authentication button.  In the first scenario they are actually logging in via the UsernamePassword provider, in the latter 2 scenarios they are using the Active Directory provider, but in all of the cases they end up logged in as the same user (this is the driver behind the fallback checks described in the next section).

This scenario would work equally with Azure AD or GoogleApps in place of Active Directory.

Starting from **Octopus 3.17**, there is also the ability to specify the details for multiple logins for each user. For example, you could specify that a user can log is as a specific UPN/SamAccountName from Active Directory or that they could login using a specific account/email address using GoogleApps. Whichever option is actually used to login, Octopus will identify them as the same user.

### Matching External Identities to Octopus Users {#AuthenticationProviders-Usernames,emailaddresses,UPNsandExternalIds}

When someone signs in to Octopus using an external authentication provider, Octopus will try to find their user account by looking for matching identifiers. It starts by looking for a matching identifiers from the external authentication provider, and will eventually fall back to match on email address.

## Changing Authentication Providers

In some circumstances you may want to move from one authentication provider to another. The best way to do this is have a period of time where you enable both the new and old authentication providers.

1. Make sure all your existing user accounts in Octopus are configured with the email address for the new authentication provider. This is how Octopus will recognize the new external identity and match it to the existing Octopus user account.
2. Enable the new authentication provider and configure it correctly.
3. Test the new authentication provider, making sure it correctly matches your existing users with their existing Octopus user accounts.
4. Disable the old authentication provider.

## Session Management

User sessions can be managed in two ways with Octopus:

1. Session cookies, which are persisted by the browser after a successful login and then sent with every subsequent HTTP request.
2. API Keys, which are a shared secret that identify the user, and must be sent with every HTTP request.

Session cookies are used for interactive sessions regardless of the authentication provider used to identity the user.

### Revoking Access to Octopus When Using External Authentication Providers

Octopus uses the external identity provider only to initially verify the user's identity, and then returns a session cookie to the browser. When you disable a user in your external identity provider, this will prevent that user from signing in to Octopus using that authentication provider. However, if the user already has an active session with Octopus, that session will stay active until the cookie expires.

If you want to revoke access to Octopus immediately, disable the user in Octopus as well as the external identity provider.
