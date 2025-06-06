# NDC Shop → Price → Book User Stories

Below are the core user stories and acceptance criteria for each step of the IATA NDC 17.2 flow. These were captured as part of a JIRA epic, but have been extracted here for anyone who wants to see exactly what needed to be built.

---

## Epic: NDC One‐Way Flight Shopping, Pricing & Booking

**As a** travel app developer  
**I want** to implement IATA NDC 17.2  
**So that** our users can search, price, and book one‐way economy flights from LHR → JFK

---

### User Story 1: AirShoppingRQ → AirShoppingRS

**As a** consumer‐facing web/mobile app  
**I want** to send a valid `<AirShoppingRQ>` (Shop) for one‐way economy from Heathrow to JFK  
**So that** I receive an `<AirShoppingRS>` containing at least one offer with `<OfferID>` and `<OfferItems>`

#### Acceptance Criteria

1. **Request**  
   - `<AirShoppingRQ>` MUST have:
     - `xmlns="http://www.iata.org/IATA/EDIST"`
     - `Version="17.2"`
     - `PartyID="TESTAGENT"`
   - Child nodes:
     ```xml
     <Document>
       <Name>ShopTest</Name>
       <ReferenceVersion>1</ReferenceVersion>
     </Document>
     <Travelers>
       <Traveler>
         <AnonymousTraveler>
           <PTC>ADT</PTC>
         </AnonymousTraveler>
       </Traveler>
     </Travelers>
     <CoreQuery>
       <OriginDestinations>
         <OriginDestination>
           <Departure>
             <AirportCode>LHR</AirportCode>
             <Date>2025-09-01</Date>
           </Departure>
           <Arrival>
             <AirportCode>JFK</AirportCode>
           </Arrival>
         </OriginDestination>
       </OriginDestinations>
     </CoreQuery>
     <Preferences>
       <CabinPreferences>
         <CabinType>Economy</CabinType>
       </CabinPreferences>
     </Preferences>
     ```

2. **Response**  
   - Must return HTTP 200.  
   - `<AirShoppingRS>` MUST contain:
     - `<Document>` with `<ResponseID>` and `<CreationDateTime>`.
     - `<DataLists><OfferList>` with at least one `<Offer>` element.
       - Each `<Offer>` must have an `<OfferID>` (e.g. `Type="14xID"`).
       - Each `<Offer>` must include `<OfferItems>` with at least one `<OfferItem><OfferItemID>`.

3. **Error Handling**  
   - If `<Travelers>` is missing or `<PTC>` is invalid, return a SOAP `<Error Type="Business">` with code `144` (Invalid PTC).  
   - If `<CoreQuery>` is missing or invalid (e.g., date in the past), return `Type="Business"` code `145`.

---

### User Story 2: OfferPriceRQ → OfferPriceRS

**As a** consumer‐facing web/mobile app  
**I want** to send a valid `<OfferPriceRQ>` referencing the selected `<OfferID>`  
**So that** I receive an `<OfferPriceRS>` with the detailed price breakdown

#### Acceptance Criteria

1. **Request**  
   - `<OfferPriceRQ>` MUST have:
     - `xmlns="http://www.iata.org/IATA/EDIST"`
     - `Version="17.2"`
     - `PartyID="TESTAGENT"`
   - Child nodes:
     ```xml
     <Document>
       <Name>PriceTest</Name>
       <ReferenceVersion>1</ReferenceVersion>
     </Document>
     <Query>
       <OfferID Owner="AA">ABC-123</OfferID>
       <PaxReferences>
         <PaxReference>
           <TravelerID>1</TravelerID>
         </PaxReference>
       </PaxReferences>
     </Query>
     ```

2. **Response**  
   - Must return HTTP 200.  
   - `<OfferPriceRS>` MUST contain:
     - `<Document>` with `<ResponseID>` and `<CreationDateTime>`.
     - `<DataLists><OfferList><Offer>` matching `OfferID="ABC-123"`.
       - `<OfferItems><OfferItem><OfferItemID>` (e.g. `AA-ECONOMY-001`).
       - `<Price><TotalAmount>` and `<CurrencyCode>`.

3. **Error Handling**  
   - If `<OfferID>` is missing or not found, return SOAP `<Error Type="Business">` code `146` (Offer Not Available).  
   - If `<TravelerID>` does not exist in the original shopping result, return code `147` (Invalid TravelerID).

---

### User Story 3: OrderCreateRQ → OrderCreateRS

**As a** consumer‐facing web/mobile app  
**I want** to send a valid `<OrderCreateRQ>` with passenger details, payment, and `OfferItemID`  
**So that** I receive an `<OrderCreateRS>` confirming the booking and ticketing information

#### Acceptance Criteria

1. **Request**  
   - `<OrderCreateRQ>` MUST have:
     - `xmlns="http://www.iata.org/IATA/EDIST"`
     - `Version="17.2"`
     - `PartyID="TESTAGENT"`
   - Child nodes:
     ```xml
     <Document>
       <Name>OrderTest</Name>
       <ReferenceVersion>1</ReferenceVersion>
     </Document>
     <Query>
       <Passengers>
         <Passenger>
           <PassengerID>1</PassengerID>
           <PersonName>
             <NamePrefix>Mr</NamePrefix>
             <GivenName>John</GivenName>
             <Surname>Doe</Surname>
           </PersonName>
         </Passenger>
       </Passengers>
       <Billing>
         <PaymentForm>
           <CreditCard>
             <CardType>VI</CardType>
             <CardNumber>4111111111111111</CardNumber>
             <ExpiryDate>2025-12</ExpiryDate>
             <CardHolderName>JOHN DOE</CardHolderName>
             <CVV>123</CVV>
           </CreditCard>
         </PaymentForm>
       </Billing>
       <Travelers>
         <Traveler>
           <AnonymousTraveler>
             <PTC>ADT</PTC>
             <PassengerTypeQuantity>1</PassengerTypeQuantity>
           </AnonymousTraveler>
           <ServiceDetails>
             <OfferItemID>AA-ECONOMY-001</OfferItemID>
           </ServiceDetails>
         </Traveler>
       </Travelers>
     </Query>
     ```

2. **Response**  
   - Must return HTTP 200.  
   - `<OrderCreateRS>` MUST contain:
     - `<Document>` with `<ResponseID>` and `<CreationDateTime>`.
     - `<Order><OrderID>` (e.g. `ORD-123456`).
     - `<OfferItemID>` matching the request (e.g. `AA-ECONOMY-001`).
     - `<Ticketing><TicketNumber>` (e.g. `001-9876543210`) and `<TicketIssueDate>`.

3. **Error Handling**  
   - If `<OfferItemID>` is invalid or not priced, return SOAP `<Error Type="Business">` code `148` (Invalid OfferItem).  
   - If credit card fails basic format validation (e.g. missing 16 digits), return code `149` (Invalid PaymentForm).

---

### How These Stories Fit Together

1. **Shop (AirShoppingRQ):** Retrieve available offers for a given origin/destination/date/cabin.  
2. **Price (OfferPriceRQ):** Lock in price details for exactly one of the offers returned in step 1.  
3. **Book (OrderCreateRQ):** Collect traveler info + payment and confirm a booking (with ticket).  

Each story references the previous step (e.g. “OfferID from step 1,” “OfferItemID from step 2”) and specifies exactly which XML nodes and attributes must be present. This level of detail ensures the development and testing teams know precisely what to build and validate.

---

## 3. Why This Belongs in the Repo

- These user stories (and acceptance criteria) are the core “spec” for any NDC‐forked implementation.  
- A developer can clone the repo, open `USER_STORIES.md`, and instantly know how to build, test, and validate each operation.  
- They also demonstrate your PO skill at turning a high‐level feature (“we need NDC”) into granular tasks with clear “pass/fail” conditions.  

---

**Bottom line:** Yes—add your user stories (with XML snippets and acceptance criteria) to the GitHub repo. It makes your thinking transparent, shows exactly how you’d guide a dev team, and rounds out the POC by pairing the “how” (Postman examples) with the “why” (business requirements and validation rules).
