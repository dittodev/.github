# STSC Trade & financial data provisioning API

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
