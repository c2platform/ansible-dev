# How-To Integrate KeyCloud and Jira

This how-to describes how to integrate KeyCloud and Jira using [SSO for Atlassian Server and Data Center - Atlassian Marketplace](https://marketplace.atlassian.com/apps/1216096/sso-for-atlassian-server-and-data-center?hosting=server&tab=versions). See also [OpenID Connect for Atlassian Data Center applications](https://confluence.atlassian.com/enterprise/openid-connect-for-atlassian-data-center-applications-987142159.html])

[![SSO for Atlassian Server and Data Center ](./files/sso-atlassian-server.png)](./files/sso-atlassian-server.png)

<!-- MarkdownTOC levels="2,3" autolink="true" -->

- [Install](#install)
- [](#)

<!-- /MarkdownTOC -->

## Install

In 8.13.0 this app is pre-installed. Update it to latest version.

## 


|Name         |Property                                                           |
|-------------|-------------------------------------------------------------------|
|Base URL     |https://seetoo.tech/                                          |
|Realm        |`seetoo`                                                           |
|Client ID    |`myapp`                                                            |
|Client secret|`f847aa0a-308a-40b4-b03b-b398ec2228e8`                             |
|IDP Hint     |                                                                   |

Hint: IDP hint for Keycloak. This suggest an identity provider when there are multiple available for the configured realm. |

Note: OpenID Provider metadata  https://seetoo.tech/auth/realms/seetoo/.well-known/openid-configuration