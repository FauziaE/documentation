# A Client service
In a delivery-hub that distributes orders over vendors and registers the subsequent deliveries, we define a service for clients. This service allows clients to change their name and address and shows them status information of their orders.

```
INTERFACE ClientInfo FOR Client,Vendor : I[Client]
BOX [ "Name" : clientName
    , "Street" : clientAddress
    , "City" : clientCity
    , "All orders" : orderedBy~
      BOX [ vendor :orderedAt
          , product : orderOf
          ]
    , "Orders to be accepted by provider" : orderedBy~ - V;orderAccepted~
    , "Orders pending delivery" : orderedBy~ /\ (V;orderAccepted~ - orderReceived~)
    , "Received orders" : orderReceived~
    ]
```

# Structure
The service has a **header**, which is the first line in this example:
```
INTERFACE ClientInfo FOR Client,Vendor : I[Client]
```
The word `ClientInfo` is the **name** of this service. This name identifies the service throughout the entire context, so it must be unique.

This service is shown only to users with roles `Client` or `Vendor`. That is indicated by the restriction `FOR Client,Vendor`. Without that restriction, the interface is available for every user in any role.

The expression to the right of the colon (`:`) symbol is called the **interface expression**. An interface is called from an atom which must be in the domain of this expression. Let, for example, `Peter` be a `Client`. As `Peter` is an element of the domain of `I[Client]`, the interface can be  called from that atom.

The same expression, `I[Client]`, is also used as **box expression** for the box that follows the header. For every element in the codomain of the box expression, a container (in HTML: `<div>`) will be drawn on the user screen. That box serves as a subinterface, which is called with precisely one atom. With `I[Client]` as box expression, the codomain will contain just one atom, which is precisely the atom from which the interface was called.

In this example, the outermost box contains seven **box items** and the innermost box two. Each box item has a label and an expression. For example the box item `"Name" : clientName` has `"Name"` as its label and `clientName` as expression.  The atom `a` from which the box was called is used to select the pairs (`a`,`x`) from the expression. All `x`-es for which (`a`,`x`) is in `clientName` will be displayed. Supposing that the relation `clientName` associates only one name to a client, this specific box item displays just one name. However, in the fifth box item, the expression  `orderedBy~ - V; orderAccepted~` may contain an arbitrary number of orders to be accepted by provider, all of which are shown.

