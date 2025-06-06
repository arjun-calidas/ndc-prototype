# NDC Prototype (Using Webhook.site)

This repository contains a lightweight proof-of-concept showing how to format and send IATA NDC 17.2 SOAP/XML requests (Shop → Price → Book) without needing a live NDC backend.

## Files

- **NDC-Prototype.postman_collection.json**  
  A Postman collection with three requests:  
  1. **AirShoppingRQ** – sends a valid `<AirShoppingRQ>` to a public Webhook.site URL.  
  2. **OfferPriceRQ** – sends a valid `<OfferPriceRQ>`.  
  3. **OrderCreateRQ** – sends a valid `<OrderCreateRQ>`.  

- **screenshots/**  
  - `airshopping-snippet.png` – Screenshot of Webhook.site capturing the `<AirShoppingRQ>`.  
  - `offerprice-snippet.png` – Screenshot capturing `<OfferPriceRQ>`.  
  - `ordercreate-snippet.png` – Screenshot capturing `<OrderCreateRQ>`.  

## How to Reproduce

1. Go to [Webhook.site](https://webhook.site) and copy your unique URL.  
2. Open Postman → **Import** → select `NDC-Prototype.postman_collection.json`.  
3. Edit each request’s URL to your Webhook.site URL (paste it where prompted).  
4. Click **Send** on each request.  
5. In Webhook.site’s dashboard, you will see the raw XML you sent—verify that it matches the IATA NDC schema elements.

That’s it—no live NDC sandbox or server is required. This proves you know how to build and parameterize NDC XML payloads correctly.
