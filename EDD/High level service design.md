# STX Climate Tech Solutions – High level service architecture & security model.

### Context

STX Climate Tech Solutions (SCTS) is building a single portal for Strive (by STX) counterparties to help them understand their trading activity and carbon commodities portfolio, and to facilitate new trading activities. In our initial scope we will focus on Energy Attribute Certificates (EACs). Each counterparty may have one or more identities that can access the account.

Additionally, we will build an internal portal for Strive Corporate Account Managers (CAMs) that can access a subset of data from selected counterparties.

### Functional product requirements

Functional requirements have been documented in the [MVP
scope](https://stxservices.sharepoint.com/:w:/r/sites/StriveDigital/_layouts/15/Doc.aspx?sourcedoc=%7B2c2fa6f1-3b3d-4fe2-a737-ef9cbb09022b%7D&action=edit&wdPreviousSession=c7af800e-4757-0cdd-f908-ce9678b54b89).

### Considerations

- Accounts are provisioned by CAMs using an SCTS managed identity
  provider. Onboarding external IDPs is not currently a requirement,
  but this may change in future milestones.

- Because the portal is accessible over the public internet it needs
  to be resilient against denial-of-service attacks and implement
  proper access controls.

- Information stored on the portal is considered _business sensitive
  information._ This includes information directly obtained from
  counterparties (_energy usage, net zero target dates)_ as well as
  internally (STX) sourced _trade and financial data_.

- External customers expect a modern SaaS solution including table
  stakes such as the ability to delegate responsibilities within an
  account to different users (RBAC/ABAC), (push) notifications,
  account recovery and personalization.

- STX is growing rapidly, and in the process of updating and
  modernizing IT infrastructure and tooling. To avoid any tight
  coupling on internal systems we will isolate and separate the portal
  from internal systems and define our data model working backwards
  from our requirements.

### Non-functional requirements

- Security first: we make no assumptions about network security and
  operate under the zero-trust model. This entails no direct access to
  production infrastructure, encryption in transit and encryption at
  rest.

- High availability, observability, disaster recovery. With customers
  expecting 24/7 availability of our service, we risk eroding trust
  otherwise. As STSC we build and operate with a DevOps mindset.

- Limited maintenance by leveraging Infrastructure as Code (IaC) and
  AWS (Amazon Web Services) managed services for authentication,
  authorization, CDN and API-Gateway.

### Explicit not-requirements (Assumptions)

- Enterprise features: customer managed encryption keys, data
  residency, custom identity pool (IDP) provider.

- Self-service onboarding and KYC (MVP only) (Our customers are STX
  customers that have been KYC’d

# High level system architecture (draft).

![Fig. 1. C2 infrastructure diagram](https://github.com/dittodev/diagrams/blob/main/drawio/CounterPartyPortalHighLevelSystemArchitecture.png)
Fig. 1. C2 infrastructure diagram

High level system diagram (C2) with actors and entry points:

(1) Anonymous users load static assets from CloudFront (Single page
app)  
(2) App authenticates user with Cognito pool (login)  
(3) App downloads information from the API and renders interface  
(4) App authenticates with CloudFront to download company assets
(documents)  
(5) SCTS client pushes trade data into the portal using provisioning
endpoint  
(6) STX CAMs login to backend portal (App) using internal STX M365
login.

## Data Model: High level entity relationship diagram (ERD)

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><p><img src="https://github.com/dittodev/diagrams/blob/main/plantuml/Climate%20Tech%20Solutions%20-%20CounterPartyPortal%20ERD.png" style="width:5.77084in;height:6.86458in" /></p>
<p>Fig 2. Entity Relationship Diagram (ERD)</p></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

### Account Data

Account data is created on the counterparty portal itself. Strive
accounts are not represented by STX internal data sources because we
trade with business entities that are represented by these accounts, and
neither do we store other customer data internally.

While user logins may be loosely coupled to internal CRM data, we will
manually onboard these users during the project's initial phase.

### Trade & Financial data

Trade & Financial data originates from STX internal systems. We have
intentionally deviated from internal naming conventions like “position”
and “position group”. We will allow internal systems to populate this
data using a dedicated provisioning API.

## Security architecture

We will implement a simplified version of the [multi-account strategy](https://aws.amazon.com/blogs/mt/multi-account-strategy-for-small-and-medium-businesses/) on AWS, using a central tooling account to deploy infrastructure and host our deployment pipelines. See [Appendix I](#appendix-ii-details-multi-account-diagram) for an in-depth overview of roles and permissions.

---

![Fig. 3. Simplified view of our mutli-account architecture annotated with access methods.](https://github.com/dittodev/diagrams/blob/main/drawio/ActorsAndSecurityBoundaries.png)
---
Fig. 3. Simplified view of our mutli-account architecture annotated with access methods.


### Authentication methods

- **_AWS Connector for Github_** this is a one-time setup needed to
  authenticate our AWS management account with Github. The AWS account
  has read permissions from any repository in our Github organization.

- **_MFA/Cognito Session_ Users interacting with the portal (UI) are
  authenticated using AWS Cognito, through a user-pool that we
  configure.**

- _Cross account roles_ **are deployed to infrastructure accounts,
  granting the tooling accounts permissions upload assets, trigger
  CloudFormation stacks and configure services like lambda and
  CloudFront.**

- _API Key_ **STX trade data is uploaded into the portal using a
  public API, authenticated by an API key issued by SCTS.**

## Threat Model (Draft)

### Trust levels

| ID  | Name                         | Description                                                                                                                                                                                                                                  |
| --- | ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Anonymous Web User           | A user who has not (yet) authenticated.                                                                                                                                                                                                      |
| 2   | Counterparty user            | A user who has authenticated with a portal account.                                                                                                                                                                                          |
| 3   | STX account manager          | An STX employee who has authenticated with a portal account, using their STX identity (Microsoft account)                                                                                                                                    |
| 4   | Third party engineer         | Third party engineer who writes, reviews, and deploys software for the portal, and who can access pre-production resources for testing and debugging.                                                                                        |
| 5   | SCTS engineer                | An STX employee who is a software engineer for Climate tech solutions, who writes, reviews and deploys software for the portal, and can access pre-production resources for testing and debugging, as well as manage production deployments. |
| 6   | SCTS administrator           | An STX employee who can manage climate tech infrastructure and AWS accounts, and the resources contained in these accounts.                                                                                                                  |
| 7   | STX IT Administrator         | An STX employee who can administrate the climate tech AWS root account, as well as other STX resources.                                                                                                                                      |
| 8   | SCTS management account.     | An AWS account that can manage infrastructure and deploy code to other infrastructure accounts.                                                                                                                                              |
| 9   | SCTS infrastructure account. | An AWS account that hosts infrastructure for a CI/CD stage of one or more portal services.                                                                                                                                                   |
| 10  | SCTS Internal API Client     | An API client who can publish information to the internal portal API, populating the system with STX financial and trade data.                                                                                                               |
|     |                              |                                                                                                                                                                                                                                              |

### Entry Points

_Entry points define the interfaces through which potential attackers
can interact with the application or supply it with data. For a
potential attacker to attack an application, entry points must exist
(OWASP)._

| ID  | Name                                     | Trust levels |
| --- | ---------------------------------------- | ------------ |
| 1   | CloudFront CDN                           | 1,2,3        |
| 2   | CloudFront CDN (Authenticated)           | 2            |
| 3   | Public Portal API                        | 2            |
| 4   | Public Data Provisioning API             | 10           |
| 5   | Public Portal Backend API                | 6            |
| 7   | Infrastructure provisioning role         | 8            |
| 8   | Deploy & configuration roles             | 8            |
| 9   | Route53 hosted Zone delegation           | 9            |
| 10  | SCTS management account _AdminAccess_    | 6            |
| 11  | SCTS management account _ViewOnlyAccess_ | 4,5,6        |
| 12  | Pre-prod account _PowerUserAccess_       | 4,5,6        |
| 13  | Pre-prod account _AdminAccess_           | 6            |
| 14  | Prod account _PowerUserAccess_           | 5,6          |
| 15  | Prod account _AdminAccess_               | 6            |
| 16  | AWS Root account access                  | 7            |

---

# Appendix I: details multi-account diagram

The following diagram details the roles and permissions required in our
multi-account setup. Note that this diagram is for illustrative purposes
– as we expand our system, each deployment target or service will
configure access roles for our management account.

Management account access is bootstrapped manually once, on a developer
machine, using a CDK bootstrap command. After bootstrapping we can
automate provisioning other dependencies. Increasing the amount of
access roles and deployment target does not necessarily increase our
security footprint in this way.

![](https://github.com/dittodev/diagrams/blob/main/drawio/CrossAccountInfrastructureDeployment.png)
