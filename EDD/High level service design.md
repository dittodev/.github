# STX Climate Tech Solutions -- High level service architecture & security model.

### Context

## STX Climate Tech Solutions (SCTS) is building a single portal for Strive (by STX) counterparties to help them understand their trading activity and carbon commodities portfolio, and to facilitate new trading activities. In our initial scope we will focus on Energy Attribute Certificates (EACs). Each counterparty may have one or more identities that can access the account. 

## Additionally, we will build an internal portal for Strive Corporate Account Managers (CAMs) that can access a subset of data from selected counterparties. 

### Functional product requirements

Functional requirements have been documented in the [MVP
scope](https://stxservices.sharepoint.com/:w:/r/sites/StriveDigital/_layouts/15/Doc.aspx?sourcedoc=%7B2c2fa6f1-3b3d-4fe2-a737-ef9cbb09022b%7D&action=edit&wdPreviousSession=c7af800e-4757-0cdd-f908-ce9678b54b89).

### Considerations

-   Accounts are provisioned by CAMs using an SCTS managed identity
    provider. Onboarding external IDPs is not currently a requirement,
    but this may change in future milestones.

-   Because the portal is accessible over the public internet it needs
    to be resilient against denial-of-service attacks and implement
    proper access controls.

-   Information stored on the portal is considered *business sensitive
    information.* This includes information directly obtained from
    counterparties (*energy usage, net zero target dates)* as well as
    internally (STX) sourced *trade and financial data*.

-   External customers expect a modern SaaS solution including table
    stakes such as the ability to delegate responsibilities within an
    account to different users (RBAC/ABAC), (push) notifications,
    account recovery and personalization.

-   STX is growing rapidly, and in the process of updating and
    modernizing IT infrastructure and tooling. To avoid any tight
    coupling on internal systems we will isolate and separate the portal
    from internal systems and define our data model working backwards
    from our requirements.

### Non-functional requirements

-   Security first: we make no assumptions about network security and
    operate under the zero-trust model. This entails no direct access to
    production infrastructure, encryption in transit and encryption at
    rest.

-   High availability, observability, disaster recovery. With customers
    expecting 24/7 availability of our service, we risk eroding trust
    otherwise. As STSC we build and operate with a DevOps mindset.

-   Limited maintenance by leveraging Infrastructure as Code (IaC) and
    AWS (Amazon Web Services) managed services for authentication,
    authorization, CDN and API-Gateway.

### Explicit not-requirements (Assumptions)

-   Enterprise features: customer managed encryption keys, data
    residency, custom identity pool (IDP) provider.

-   Self-service onboarding and KYC (MVP only) (Our customers are STX
    customers that have been KYC'd

# High level system architecture (draft). 

![](/media/image3.png){width="6.5in" height="4.6875in"}

Fig. 1. C2 infrastructure diagram

High level system diagram (C2) with actors and entry points:\
\
(1) Anonymous users load static assets from CloudFront (Single page
app)\
(2) App authenticates user with Cognito pool (login)\
(3) App downloads information from the API and renders interface\
(4) App authenticates with CloudFront to download company assets
(documents)\
(5) SCTS client pushes trade data into the portal using provisioning
endpoint\
(6) STX CAMs login to backend portal (App) using internal STX M365
login.

## Data Model: High level entity relationship diagram (ERD)

+-----------------------------------------------------------------------+
| ![](/media/image8.png){width="5.77083552055993in"                     |
| height="6.864584426946632in"}                                         |
|                                                                       |
| Fig 2. Entity Relationship Diagram (ERD)                              |
+=======================================================================+
+-----------------------------------------------------------------------+

### Account Data

Account data is created on the counterparty portal itself. Strive
accounts are not represented by STX internal data sources because we
trade with business entities that are represented by these accounts, and
neither do we store other customer data internally.

While user logins may be loosely coupled to internal CRM data, we will
manually onboard these users during the project\'s initial phase.

### Trade & Financial data

Trade & Financial data originates from STX internal systems. We have
intentionally deviated from internal naming conventions like "position"
and "position group". We will allow internal systems to populate this
data using a dedicated provisioning API.

## Security architecture

We will implement a simplified version of the [multi-account
strategy](https://aws.amazon.com/blogs/mt/multi-account-strategy-for-small-and-medium-businesses/)
on AWS, using a central tooling account to deploy infrastructure and
host our deployment pipelines. See [Appendix
I](#appendix-ii-details-multi-account-diagram) for an in-depth overview
of roles and permissions.

+-----------------------------------------------------------------------+
| ![](/media/image5.png){width="6.4377701224846895in"                   |
| height="8.802082239720034in"}                                         |
|                                                                       |
| Fig. 3. Simplified view of our mutli-account architecture annotated   |
| with access methods.                                                  |
+=======================================================================+
+-----------------------------------------------------------------------+

### Authentication methods

-   ***AWS Connector for Github*** this is a one-time setup needed to
    authenticate our AWS management account with Github. The AWS account
    has read permissions from any repository in our Github organization.

-   ***MFA/Cognito Session* Users interacting with the portal (UI) are
    authenticated using AWS Cognito, through a user-pool that we
    configure.**

-   *Cross account roles* **are deployed to infrastructure accounts,
    granting the tooling accounts permissions upload assets, trigger
    CloudFormation stacks and configure services like lambda and
    CloudFront.**

-   *API Key* **STX trade data is uploaded into the portal using a
    public API, authenticated by an API key issued by SCTS.**

## Threat Model (Draft)

### Trust levels

  -----------------------------------------------------------------------
  ID      Name                    Description
  ------- ----------------------- ---------------------------------------
  1       Anonymous Web User      A user who has not (yet) authenticated.

  2       Counterparty user       A user who has authenticated with a
                                  portal account.

  3       STX account manager     An STX employee who has authenticated
                                  with a portal account, using their STX
                                  identity (Microsoft account)

  4       Third party engineer    Third party engineer who writes,
                                  reviews, and deploys software for the
                                  portal, and who can access
                                  pre-production resources for testing
                                  and debugging.

  5       SCTS engineer           An STX employee who is a software
                                  engineer for Climate tech solutions,
                                  who writes, reviews and deploys
                                  software for the portal, and can access
                                  pre-production resources for testing
                                  and debugging, as well as manage
                                  production deployments.

  6       SCTS administrator      An STX employee who can manage climate
                                  tech infrastructure and AWS accounts,
                                  and the resources contained in these
                                  accounts.

  7       STX IT Administrator    An STX employee who can administrate
                                  the climate tech AWS root account, as
                                  well as other STX resources.

  8       SCTS management         An AWS account that can manage
          account.                infrastructure and deploy code to other
                                  infrastructure accounts.

  9       SCTS infrastructure     An AWS account that hosts
          account.                infrastructure for a CI/CD stage of one
                                  or more portal services.

  10      SCTS Internal API       An API client who can publish
          Client                  information to the internal portal API,
                                  populating the system with STX
                                  financial and trade data.

                                  
  -----------------------------------------------------------------------

### Entry Points

*Entry points define the interfaces through which potential attackers
can interact with the application or supply it with data. For a
potential attacker to attack an application, entry points must exist
(OWASP).*

  -----------------------------------------------------------------------
  ID       Name                                   Trust levels
  -------- -------------------------------------- -----------------------
  1        CloudFront CDN                         1,2,3

  2        CloudFront CDN (Authenticated)         2

  3        Public Portal API                      2

  4        Public Data Provisioning API           10

  5        Public Portal Backend API              6

  7        Infrastructure provisioning role       8

  8        Deploy & configuration roles           8

  9        Route53 hosted Zone delegation         9

  10       SCTS management account *AdminAccess*  6

  11       SCTS management account                4,5,6
           *ViewOnlyAccess*                       

  12       Pre-prod account *PowerUserAccess*     4,5,6

  13       Pre-prod account *AdminAccess*         6

  14       Prod account *PowerUserAccess*         5,6

  15       Prod account *AdminAccess*             6

  16       AWS Root account access                7
  -----------------------------------------------------------------------

# APPENDIX I: STSC Trade & financial data provisioning API

The Trade & Financial data provisioning API (TFD API) allows STX
internal systems to populate the counterparty portal with data. The TFD
API acts as a source of truth for internal systems to maintain a mapping
between internal entities and portal identities: TFD entities contain a
metadata field to store immutable internal identifiers. TODO: describe
this in more detail in an appendix, together with an example mapping
between ERD and internal data.

SCTS will provide libraries and standard clients to integrate with the
TFD API.

### System architecture

We will build a lightweight serverless endpoint using AWS API Gateway
and Lambda. Endpoint authentication uses standard IAM credentials scoped
to execute-api:Invoke and may be further hardened using AWS WAF and AWS
Resource Policies to limit potential traffic sources. Optionally we may
further harden security by creating a [VPC
Privatelink](https://aws.amazon.com/blogs/compute/architecture-patterns-for-consuming-private-apis-cross-account/).

Assets will be uploaded to S3 directly using Signed URLs. This provides
a scalable, secure, and stable data plane independent of the original
underlying datastore. We do not allow retrieving these assets or
modifying them.

### Bookkeeping

To facilitate bookkeeping we will provide an interface to attach
arbitrary metadata to objects in the portal. The metadata will be in the
shape of simple key-value pairs. We will allow up to N key-value pairs
per item, with an X limit in key length and a Y max length per value.
All Get\* operation will return this metadata verbatim.

  -----------------------------------------------------------------------
  ![](/media/image6.png){width="6.34375in"
  height="3.8631813210848645in"}Fig. 4. STSC Trade & Financial data
  provisioning API system architecture
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

## Endpoints

The following endpoints have been identified following the ERD (fig. 2).

TODO

-   Map use-cases and to functional requirements

-   ***Wed Jun 5 -- MVH current naming is outdated, See ERD***

  -------------------------------------------------------------------------------------
  ID    Path                               GET   POST   PUT   DELETE   description
  ----- ---------------------------------- ----- ------ ----- -------- ----------------
  1.1   /products                          x                           List products
                                                                       defined in the
                                                                       portal

  2.1   /quote                                   x                     Create a quote
                                                                       in the portal

  3.1   /orders                            x     x                     Create a new
                                                                       order in the
                                                                       portal.

  3.2   /orders/\[oid\]/contract                 x      ?              The contract
                                                                       associated with
                                                                       the portal.

  3.3   /orders/\[oid\]/addendums          x     x                     Addendums
                                                                       associated with
                                                                       the contract.

  3.4   /orders/\[oid\]/items              x     x                     Line items that
                                                                       are part of the
                                                                       order
                                                                       (positions).

  3.5   /orders/\[oid\]/items/\[itemid\]   x                  x        A single line
                                                                       item associated
                                                                       with an order.

  3.6   /orders/\[oid\]/invoices                 x                     Invoices sent
                                                                       for an order.

  3.7   /orders/\[oid\]/delivery                 x                     Delivery made
                                                                       for a line item.
                                                                       (certificates)

  4.1   /entities                          x                           List all
                                                                       entities in the
                                                                       portal
  -------------------------------------------------------------------------------------

### Products

This endpoint exists to expose an internal catalog used for display
purposes. Products SHOULD align to internal STX products. ***Further
investigation is needed.***

#### Use cases:

1.  As an STX engineer, I want to list all TFD product identifiers with
    metadata used in the UI, so that I can retrieve internal
    identifiers, and send the TFD identifier as part of a new quote.

  -------------------------------------------------------------------------
  ID             name           input          output
  -------------- -------------- -------------- ----------------------------
  1.1            ListPoducts                   Products\[\]

  -------------------------------------------------------------------------

### Quotes

Quotes are firm offers given by Strive. Customers want to see firm
offers received through the portal. We think quotes CAN and SHOULD
originate from internal systems. ***Further investigation is needed.***

#### Use cases:

1.  As a Strive CAM, I want to provide a firm offer a counterparty in
    the portal. I want to use existing internal processes and tooling so
    that I reduce manual churn and speed up the sales process.

  -------------------------------------------------------------------------
  ID             name           input          output
  -------------- -------------- -------------- ----------------------------
  2.1            CreateQuote    Price, amount, QuoteID
                                technology,    
                                origin, TFD    
                                product        
                                identifier     

  -------------------------------------------------------------------------

### Orders

Firm offers (quotes) turn into orders. In the portal orders contain
numbers for visualization and act as the main container for financial
and trade: contracts, invoices, addendums, certificates.

  -------------------------------------------------------------------------------
  ID     name                          input                  output
  ------ ----------------------------- ---------------------- -------------------
  3.1    ListOrders                    Entity,                Orders\[{orderId,
                                       paginationToken        reference, create
                                                              date, metadata}\]

  3.1    CreateOrder                   Create date, reference OrderId

  3.2    CreateContract                Create date, reference uploadURL

  3.2    UpdateContract                OrderId                uploadURL

  3.3    ListOrderContractAddendums    OrderId                Addendums \[{create
                                                              date, metadata}\]

  3.3    CreateOrderContractAddendum   OrderId                uploadURL

  3.4    ListOrderItems                OrderId                Item\[{Id, created,
                                                              metadata}\]

  3.4    CreateOrderItem               OrderId                itemId

  3.5    GetOrderItem                  OrderId, ItemId        Item

  3.5    DeleteOrderItem               OrderId, ItemId        

  3.6    CreateInvoice                 OrderId                uploadURL

  3.7    CreateDelivery                OrderId, ItemId        uploadURL
  -------------------------------------------------------------------------------

### Entities

Reverse lookup for sync purposes. List all known entities in the portal
with external metadata.

#### Use cases:

1.  As an STX engineer, I want portal data to be my source of truth for
    account data, so that I don't need to keep a record of this myself
    and can build reverse lookups.

  -------------------------------------------------------------------------
  ID             name           input          output
  -------------- -------------- -------------- ----------------------------
  4.1            ListEntities                  Entity\[{Id, acountId,
                                               metadata}\]

  -------------------------------------------------------------------------

# Appendix II: details multi-account diagram

The following diagram details the roles and permissions required in our
multi-account setup. Note that this diagram is for illustrative purposes
-- as we expand our system, each deployment target or service will
configure access roles for our management account.

Management account access is bootstrapped manually once, on a developer
machine, using a CDK bootstrap command. After bootstrapping we can
automate provisioning other dependencies. Increasing the amount of
access roles and deployment target does not necessarily increase our
security footprint in this way.

![](/media/image7.png){width="6.5in" height="3.5729166666666665in"}
