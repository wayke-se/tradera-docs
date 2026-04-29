
# Tradera - Wayke | Leads documentation
Tradera will be given two API keys pointing towards the production and test environments respectively. When doing an API call, one of these must be set as a key inside the header value `x-api-key` 
to authenticate the request.

The functionality described in this document pertains towards generating a phone number for a specified vehicle and creating a dedicated lead for a dealer.

## Generating a phone number for use on a vehicle
When the user wants to phone the dealer for a vehicle a corresponding number has to be generated. This is done through a GraphQL request at the endpoint `https://test-api.wayke.com/graphql`

### Example GraphQL Request
Endpoint: `https://test-api.wayke.com/graphql`

Requires the `x-api-key` header to be set to an API key on the HTTP request.

```graphql
query {
  dynamicNumber {
    generateNumber(input: {
      adId: "3d329e3b-f744-4d73-a54e-919caaf8458d"
    }) {
      success
      number
    }
  }
}
```

`adId`: The identification number for the specific Wayke vehicle 

**No additional fields other than `adId` should be set by Tradera**

## Creating a dedicated lead
If the user is interested in a car and wants to message the dealer, a "lead" can be created. This is done through a REST POST call to the endpoint `https://test-api.wayke.com/lead`


### Example REST Request body
Endpoint: `https://test-api.wayke.com/lead`

Requires the `x-api-key` header to be set to an API key on the HTTP request.
```json
{
    "branchId": "a42a9d8b-a22e-4c1c-87e3-7113e27b46d9",
    "firstName": "Martin",
    "lastName": "Bergqlin",
    "email": "martin@ourstudio.se",
    "phoneNumber": "+46423424434",
    "type": "registrationOfInterestToBuy",
    "metadata": [
        {
            "key": "itemForSaleId",
            "value": "5d6dc4f6-2016-43ea-93ad-9babb24bd250"
        },
        {
            "key": "message",
            "value": "Sample text"
        },
        {
            "key": "sourceMechanism",
            "value": "cta.email"
        },
        {
            "key": "readingUnit",
            "value": "ScandinavianMile"
        },
        {
            "key": "registrationNumber",
            "value": "ONP33D"
        },
        {
            "key": "tradeInCarMileage",
            "value": "6500"
        },
    ]
}
```

`branchId`: The branch the vehicle is registered at

`firstName`: First name of user

`lastName`: Last name of user

`email`: The user email

`itemForSaleId`: The identification number for the specific Wayke vehicle

`message`: The user message
