# How to Choose an Identity Authentication Service?
English version for [如何选择一款身份认证服务？](1-CN-如何选择一款身份认证服务.md)translated by GPT-4o

OAuth 2.0 is a protocol that allows users to authorize clients to access protected resources on their behalf. For example, when using some WeChat mini-programs, a dialog box may pop up asking for your authorization. Once authorized through the OAuth 2.0 protocol, the mini-program can access the resources you allow, such as your profile picture, username, etc. Many companies have built identity authentication management systems based on this protocol to enable users to quickly set up authentication services. Common examples include Amazon Cognito, Auth0, Firebase Authentication, Ping Identity, and Azure Active Directory.

I have worked with two cloud-based identity management services with significant market shares—Auth0 and Cognito—in my projects. Both provide rapid setup for identity authentication and authorization services, including client creation, user pool maintenance, multiple login methods, and multi-factor authentication (MFA).

But what are the differences between them? How should you choose a provider if you want to build an authentication service? This article compares them from both user and developer perspectives.

## User Perspective

### Customizable Interface

When users interact with an authentication system, they first notice whether the interface is user-friendly and whether the login methods meet their needs. Clients often require custom UI designs, such as adding company logos, and defining color schemes. Both Cognito and Auth0 offer customizable UI features for different clients.

![Cognito Hosted UI](pictures/1-01.png)

Cognito’s UI customization options are limited, allowing changes only to colors, logos, buttons, and input field styles.

![Auth0 Hosted UI](pictures/1-02.png)

Auth0 offers a wider range of customization. Using its universal login feature, you can tailor the colors, fonts, logos, and layout of login, registration, password recovery, and MFA interfaces. It also provides Liquid templates to load different UI styles for different clients. Additionally, if more advanced customization is required, Auth0 offers three templates: Lock, Lock (passwordless), and Custom Login Form, via the Lock and auth0.js SDKs.

### Login Methods

Cognito supports the following login methods:

* Username and password
* Social login (Google, Apple, Facebook, Amazon)
* Enterprise login (SAML, OIDC)

![Cognito supported login methods](pictures/1-03.png)

Auth0 expands on this with:

* 52 types of social logins
* Enterprise logins like Google Workspace, Okta, Azure, Ping, ADFS, and LDAP
* Passwordless and biometrics login for enhanced security and a password-free user experience.

Auth0’s configuration is more convenient and provides a wider range of options compared to Cognito.

![Auth0 supported login methods](pictures/1-04.png)

![Auth0 supported social login methods](pictures/1-05.png)

### User Security

Both Auth0 and Cognito provide various security measures to protect user accounts:

* Cognito Security: MFA and IP restrictions.
* Auth0 Security: Bot Detection, IP throttling, Brute-force Protection, Breached Password Detection, and MFA.
Auth0’s MFA options are more extensive, including:

* FIDO security keys
* FIDO device biometrics
* OTP
* Auth0 Guardian
* SMS
* Email
* Recovery codes
* DUO security

This variety ensures higher security and usability compared to Cognito’s SMS and OTP options.

![Auth0 supported MFA](pictures/1-06.png)

### User Preferences

Auth0 provides a more flexible way to define user attributes with three types:

* Standard attributes
* user_metadata: Customizable without modifying the user pool schema (e.g., display preferences or user nicknames)
* app_metadata: System-level attributes (e.g., enabling MFA for specific users)

Cognito, on the other hand, uses standard and custom attributes. However, adding a new custom attribute requires creating a new user pool, which complicates user management, especially since Cognito user pools don’t support data export.

## Developer Perspective

### User Management

When users authenticate using different methods (e.g., Google social login and email/password with the same email), identity systems should manage them effectively.

* Auth0: Uses connection to distinguish users from different sources, such as social login, password login, and passwordless login. Accounts can be linked across different connections, making it easier for users and admins to manage multiple logins. Auth0 provides an Auth0 Account Link plugin for this functionality.
* Cognito: Groups users by external login source but does not provide an account linking feature, requiring custom implementation.

### Custom Logic

Customizing authentication logic (e.g., enabling MFA for specific users) can be achieved using:

Cognito Triggers: Implemented as Lambda functions that can be triggered at different authentication stages. Since they are Lambda functions, they can be written in any Lambda-supported language.


| cognito                        | auth0                  |
|--------------------------------|------------------------|
| Pre sign-up                    | Login                  |
| Pre authentication             | Machine to Machine     |
| Post authentication            | Pre User Registration  |
| Post confirmation              | Post User Registration |
| Custom message                 | Post Change Password   |
| Define Auth Challenge          | Send Phone Message     |
| Create Auth Challenge          |                        |
| Verify Auth Challenge Response |                        |
| User Migration                 |                        |
| Pre Token Generation           |                        |

Auth0: Offers Rules, Hooks, and Actions, all built on Node.js. Actions are the latest and most recommended approach, supporting external libraries and visual workflow management. Each deployment creates a new version, allowing quick rollback if needed.

![Auth0 Actions Visual Editing Interface](pictures/1-07.png)

### System Monitoring

System monitoring ensures reliability, availability, and performance:

* Cognito: Integrates with AWS services like CloudTrail and CloudWatch.
* Auth0: Provides:
    Logs: Records all tenant activities, with filtering options.
    Streams: Exports logs for further processing, integrating with Amazon EventBridge, Datadog, Splunk, Sumo Logic, etc.
    User History: Allows direct tracking of individual user actions.

### Data Migration

As requirements change, developers may need to switch identity providers or migrate data:

Auth0:

* Supports Client ID and Secret Import, maintaining client access during platform transitions.
* Allows Bulk migration and Lazy migration for user data.
* Offers plugins for data import/export with password encryption.
* Supports Session migration, ensuring seamless user experience during provider changes.
* Provides External Database Connections, allowing users to maintain custom user pools.

Cognito:

* Does not support data export.
* Only supports data import through Bulk migration or Lazy migration.。

![Auth0 lazy migration](pictures/1-08.png)

### SDK Support

Cognito: Uses Amplify SDK, supporting:

Authentication UI Components for React, Vue, Angular, and React Native.
Amplify Auth for authentication flows like sign-up and sign-in.
Auth0: Offers SDKs for:

SPA, Web apps, APIs, and Mobile apps.
Multi-language support, including Express, Java, Next.js, PHP, Python, Ruby, etc.
A well-supported community and responsive support team.

## Conclusion

* User Experience: Auth0 excels with highly customizable UI and diverse authentication options, including passwordless login, enhancing user experience and security.
* Developer Experience: Auth0 offers more built-in features and templates, faster customization, and a plugin-rich marketplace, reducing development time. In contrast, Cognito requires more custom implementation.
* Documentation and Support: Auth0 provides clearer documentation and quicker support feedback, with shorter cycles from feature requests to launch.

Recommendation:

* For basic authentication needs: Both Auth0 and Cognito are suitable, with Cognito offering a cost advantage.
* For rich user experiences or advanced features: Auth0 is the better choice.
