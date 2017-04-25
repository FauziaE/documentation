# Working with rules in RAP2

Consider the following script. You can compile and run it in [RAP2](http://is.cs.ou.nl/rap2).

```
CONTEXT Delivery IN ENGLISH



clientName :: Client -> Name
 =  [ ("Client_1"      , "Martijn")
    ; ("Client_2"      , "Stef")
    ]

clientAddress :: Client -> Address
  = [ ("Client_1"      , "Kerkstraat")
    ; ("Client_2"      , "Dorpsstraat")
    ]

clientCity :: Client -> City
  = [ ("Client_1"      , "Utrecht")
    ; ("Client_2"      , "Enschede")
    ]

-- Vendor

vendorName :: Vendor -> Name
  = [ ("Vendor_1", "Rubber inc.")
    ; ("Vendor_2", "Mario's pizzas")
    ]

sells :: Vendor * Product
  = [ ("Vendor_1", "Product_1")
    ; ("Vendor_1", "Product_2")
    ; ("Vendor_1", "Product_3")
    ; ("Vendor_2", "Product_4")
    ; ("Vendor_2", "Product_5")
    ; ("Vendor_2", "Product_3")
    ]

-- Product

productName :: Product -> Name
  = [ ("Product_1", "Inner tube")
    ; ("Product_2", "Bouncing ball")
    ; ("Product_3", "Rubber chicken")
    ; ("Product_4", "Pizza Margherita")
    ; ("Product_5", "Broodje Mario")
    ]

productPrice :: Product -> Price
  = [ ("Product_1", "10,00 euro")
    ; ("Product_2", "0,75 euro")
    ; ("Product_3", "6,95 euro")
    ; ("Product_4", "8,50 euro")
    ; ("Product_5", "4,50 euro")
    ]

-- Order
orderTotal :: Order * Price [UNI]

orderedBy :: Order -> Client
--  = [ ("Order_1", "Client_2") ]
orderedAt :: Order -> Vendor
--  = [ ("Order_1", "Vendor_1") ]
orderOf :: Order * Product [TOT]
--  = [ ("Order_1", "Product_1") ]

-- Rules

PROCESS Bestellen

orderAccepted :: Order * Vendor [UNI] -- an order may not be accepted by multiple vendors
--  = [ ("Order_1", "Vendor_1") ]

orderReceived :: Order * Client [UNI] -- an order may not be received by multiple clients
--  = [ ("Order_1", "Client_1") ]


RULE orderInAssortment : orderOf |- orderedAt; sells

PURPOSE RULE allAccepted
{+To remind vendors of orders that are not yet accepted, we introduce a process rule.
-}
RULE allAccepted: orderedAt |- (I/\orderAccepted; orderAccepted~); orderedAt -- == TOT extended to allow hyperlinking to vendor in violation
MEANING "All orders have been accepted"
MESSAGE "Not all orders have been accepted"
VIOLATION (TGT I, TXT " has not accepted the order ",SRC I,TXT " by ", SRC orderedBy; clientName)

RULE allPriced: orderAccepted |- (orderTotal;orderTotal~/\I ) ;orderAccepted
MEANING "The order's total price must be calculated for each accepted order."
MESSAGE "Not all accepted orders have been priced."
VIOLATION (SRC I, TXT " has been accepted by ", TGT I, TXT " but hasn't been priced.")

ROLE Vendor MAINTAINS allAccepted
ROLE OPA MAINTAINS allPrized


ROLE Client MAINTAINS dummy
RULE dummy: orderedAt |- orderedAt

ENDPROCESS

-- Interfaces
INTERFACE Overview  : I[ONE]
BOX[ "All clients"  : V[ONE*Client]
   , "All vendors"  : V[ONE*Vendor]
   , "All products" : V[ONE*Product]
   , "All orders"   : V[ONE*Order]
     BOX [ product : orderOf;productName
         , client  : orderedBy;clientName
         , vendor  :orderedAt;vendorName
         ]
   ]

INTERFACE Client (clientName, clientAddress, clientCity
                 , orderedBy,orderOf,orderedAt
                 , orderReceived) FOR Client : I[Client]
BOX [ "Name"   : clientName
    , "Street" : clientAddress
    , "City"   : clientCity
    , "All orders" : orderedBy~
     BOX [ vendor  :orderedAt
         , product : orderOf
         ]
    , "Orders to be accepted by provider" : orderedBy~ /\ -(V; orderAccepted~)
    , "Orders pending delivery" : orderedBy~ /\ (V; orderAccepted~) /\ -orderReceived~
    , "Received orders"       : orderReceived~
    ]

INTERFACE ClientInfo (orderAccepted) FOR Vendor : I[Client]
BOX [ "Name"   : clientName
    , "Street" : clientAddress
    , "City"   : clientCity
    , "All orders" : orderedBy~
     BOX [ product : orderOf;productName
         , client  : orderedBy;clientName
         , vendor  :orderedAt;vendorName
         ]
    , "Orders to be accepted by provider" : orderedBy~ /\ -(V; orderAccepted~)
     BOX [ product : orderOf;productName
         , client  : orderedBy;clientName
         , vendor  :orderedAt;vendorName
         ]
    , "Orders pending delivery" : orderedBy~ /\ (V; orderAccepted~) /\ -orderReceived~
     BOX [ product : orderOf;productName
         , client  : orderedBy;clientName
         , vendor  :orderedAt;vendorName
         ]
    , "Received orders"       : orderReceived~
     BOX [ product : orderOf;productName
         , client  : orderedBy;clientName
         , vendor  :orderedAt;vendorName
         ]
    ]

INTERFACE Vendor (vendorName, sells, productName, productPrice, orderAccepted) FOR Vendor: I[Vendor]

BOX [ "Name"     : vendorName
    , "Products" : sells
      BOX [ "Name"  : productName
          , "Price" : productPrice
          ]
    , "Orders to be accepted" : orderedAt~ /\ -orderAccepted~
     BOX [ product : orderOf;productName
         , client  : orderedBy;clientName
         , vendor  :orderedAt;vendorName
         ]
    , "Orders to be delivered"       : orderAccepted~ /\ -(orderAccepted~;orderReceived;V)
     BOX [ product : orderOf;productName
         , client  : orderedBy;clientName
         , vendor  :orderedAt;vendorName
         ]
    , "Past orders"       : orderAccepted~ /\ orderAccepted~;orderReceived;V
     BOX [ product : orderOf;productName
         , client  : orderedBy;clientName
         , vendor  :orderedAt;vendorName
         ]

    ]
INTERFACE Order (orderTotal) FOR OPA : I[Order]
BOX [ "Client"   : orderedBy
    , "Vendor"   : orderedAt
    , "Products" : orderOf
      BOX [ Product : I
          , Price : productPrice
          ]
    , "Total price" : orderTotal
    ]
INTERFACE Product (productName, productPrice) FOR Vendor : I[Product]
BOX [ "Name"    : productName
    , "Price"   : productPrice
    , "Vendors" : sells~
    ]

INTERFACE AcceptOrderByVendor (orderAccepted) FOR Vendor : I[Order] /\ -(orderAccepted;orderAccepted~)
BOX [ "Client"  : orderedBy
    , "Vendor"  : orderedAt
    , "Product" : orderOf
    , "sign here to accept" : orderAccepted
    ]

INTERFACE ViewOrderByVendor FOR Vendor : I[Order]
BOX [ "Client"  : orderedBy
    , "Vendor"  : orderedAt
    , "Products" : orderOf
    , "sign here to accept" : orderAccepted
    ]

INTERFACE OrdersForClient (orderedBy, orderedAt, orderOf, orderReceived) FOR Client : I[Order]
BOX [ "Client"  : orderedBy
    , "Vendor"  : orderedAt
    , "Product" : orderOf
    , "accepted by"  : orderAccepted
    , "sign here when received" : orderReceived
    ]

ENDCONTEXT
```

## Assignment

This script contains a `RULE` called `orderInAssortment`. Describe the meaning and the purpose of this rule. It may help to play with the script and discussing this rule with your peers. Add your meaning and purpose to the script and make sure it compiles and runs.
