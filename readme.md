# PeopleForBikes Ridespot Integration RFC
RFC for how to extend the [Ridespot](https://www.ridespot.org) website/API to support the needs of the [PeopleForBikes' main website](https://www.peopleforbikes.org) and other properties like [PFB Keep Riding](https://www.pfbkeepriding.org/)

## Description
A long term goal for PFB is to create more consumer-facing experiences that encourages general riders to explore new rides. One of the major tools the organization is using to accomplish this goal is [Ridespot](https://www.ridespot.org), a mobile application that allows users to share exciting rides and creates cue cards for other riders. 

Currently, Ridespot does not directly integrate with any other PFB properties and has poor usability for web-based visitors. The Ridespot team has asked for guidance on the data needs for integration with the other wed-based PFB properties, so this RFC serves as a starting point in that conversation. This document outlines the baseline data needs to integrate Ridespot and PFB's main site and the 'Keep Riding' campaign, with an eye towards using Ridespot data in new applications in the future.

## High Level

The current PeopleForBikes main website has three areas with Ridespot data needs:

- Homepage 'Rides' block: https://www.peopleforbikes.org/
- Location Pages 'Rides' block, sample: https://www.peopleforbikes.org/locations/colorado
- 'Featured Rides' page: https://www.peopleforbikes.org/rides

The 'Keep Riding' site is built around Ridespot rides, with the 'Routes' section outlining categories for riding:

- https://www.pfbkeepriding.org/ride-collections

All points of integration are doing the same thing, more or less: **showing users rides from Ridespot**.

## Data

For each ride, we need to following information to be made available from Ridespot:

- Title
- City
- State
- Country
- Organization / Individual associated with ride
- Coordinates (Latitude of ride start, Longitude of ride start)
- Distance (in miles and kilometers)
- Ride Type
- Ride card image (see Homepage 'Rides' block: (https://www.peopleforbikes.org/)
- QR Code
- URL
- Featured Image
- Collection(s)
- Editor's Choice ride


## New Features for Ridespot.org backend

Three new pieces of information need to be included with each ride to make this concept a reality:

- **Featured Image** (we recommend a common aspect ratio like: 5:4 or 4:3, with the backend service sizing the image appropriately). Serving properly sized images is very hard to do correctly and THOR recommends the Ridespot team use something like [imgix](https://imgix.com/) as part of their software stack to mitigate the effort.
- **Collection(s)** - these are the categories from the PFB 'Keep Riding' campaign. See the collections on [https://www.pfbkeepriding.org/ride-collections](https://www.pfbkeepriding.org/ride-collections). **Please Note** these have not been finalized by the PFB editorial team.
- **Editors Choice** - this is an option for Ridespot administrators to indicate a ride is of exceptional quality. 

**There probably needs to be enhancements to the Ridespot administrative dashboard to do batch collection and editor choice metadata modifications. This would be an excellent place to get feedback from the Ridespot team on how to procede.**


## Sample Endpoint Documentation - REST

The following is an outline of what THOR would expect from the Ridespot team for a set of REST-based endpoints.

| Endpoint | READ | WRITE | Request | Description
|----------|:--------:|:--------:|:--------:|--------|
| /data/ridespot/v1/ride/{id} | yes | no | GET | Returns single ride based on ride ID
| /data/ridespot/v1/ridesLatest | yes | no | GET | Returns latest 20 rides on platform
| /data/ridespot/v1/ridesEC | yes | no | GET | Returns latest 20 Editor's Choice rides
| /data/ridespot/v1/ridesByState/{state} | yes | no | GET | Returns 20 most viewed rides for a state
| /data/ridespot/v1/collection/{name} | yes | no | GET | Returns 20 most viewed rides for a collection
| /data/ridespot/v1/collectionEC/{name} | yes | no | GET | Returns 20 most viewed, Editor's Choice rides for a collection
| /data/ridespot/v1/collections | yes | no | GET | Returns an array of all collection names

The endpoints map to the views needed on the main PFB site and Keep Riding campaign sites, but the endpoints are flexible enough to be repurposed by other applications in the future.


### Environments
From what has been provided to THOR by the Ridespot team, we understand the Ridespot infrastructure is hosted on [Azure](https://azure.microsoft.com/en-us/), which is Microsoft's cloud offering. As Azure is endlessly customizable, we expect a professional development environment for the API that has at least development and production environments, and should also include a staging environment for future endpoint enhancement and testing.

Example environments:

* DEV: [https://dev.ridespot.org/data](https://dev.ridespot.org/data)
* STAGING: [https://staging.ridespot.org/data](https://staging.ridespot.org/data)
* PROD: [https://www.ridespot.org/data](https://www.ridespot.org/data)


### Namespace
The full namespace of the API:
* `/data/ridespot/v1`

We could potentially use `/data` as the base of the endpoint URLs, as an example. We can extend that with our own namespace of `ridespot` or `rs` and then version with `v1`. You shouldn't be able to change `/data` and should not change the namespace. 

However, `v1` can be changed inside of each endpoint - and should be if the Ridespot enhances any endpoint. All documenation needs to be updated whenever a new endpoint iteration is deployed.


## Example Endpoints

### Ride

Description: Return a JSON object of an individual ride from the Ridespot platform

Endpoint: `/data/ridespot/v1/ride/{id}` 

Parameters:

| Name | Type | Description
|----------|:--------:|:--------:|
| id | int | This is the unique ID number of the ride inside of the Ridespot system (ie, `224871`) |

Schema:

```
{
  "#######": {
    "title": string,
    "city": string,
    "state": string,
    "stateAbbreviation": string,
    "country": string,
    "story": [
      {
        "type": "paragraph",
        "text": string
      },
      {
        "type": "image",
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      }
    ],
    "date": string,
    "url": string,
    "organization" : {
      "name": string,
      "url": string
    },
    "coordinates": {
      "latitude": float,
      "longitude": float
    },
    "distance": {
      "miles": float,
      "kilometers": float
    },
    "rideType": string
    "rideCardImage": {
      "url": string,
      "alt": string,
      "caption": string,
      "license": string,
      "dimensions: {
        "width": int,
        "height": int
      }
    },
    "qrCode": {
      "url": string,
      "alt": string,
      "caption": string,
      "license": string,
      "dimensions: {
        "width": int,
        "height": int
      }
    },    
    "featuredImage": {
      "url": string,
      "alt": string,
      "caption": string,
      "license": string,
      "dimensions: {
        "width": int,
        "height": int
      }    
    },
    "collections": [ string ],
    "ecRide": boolean
  }
}
````

Response example :
TBD

### Latest Rides

Description: Return a JSON object of the twenty latest rides added to the system

Endpoint: `/data/ridespot/v1/ridesLatest` 

Parameters: None

Schema:

```
{
  "total": int (should always be 20),
  "rides": [
    {
      "title": string,
      "city": string,
      "state": string,
      "stateAbbreviation": string,
      "country": string,
      "story": [
        {
          "type": "paragraph",
          "text": string
        },
        {
          "type": "image",
          "url": string,
          "alt": string,
          "caption": string,
          "license": string,
          "dimensions: {
            "width": int,
            "height": int
          }
        }
      ],
      "date": string,
      "url": string,
      "organization" : {
        "name": string,
        "url": string
      },
      "coordinates": {
        "latitude": float,
        "longitude": float
      },
      "distance": {
        "miles": float,
        "kilometers": float
      },
      "rideType": string
      "rideCardImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },
      "qrCode": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },    
      "featuredImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }    
      },
      "collections": [ string ],
      "ecRide": boolean
    },
    ...
  ]
}
````

Response example :
TBD


### Editor's Choice Rides

Description: Return a JSON object of the twenty latest Editor's Choice rides

Endpoint: `/data/ridespot/v1/ridesEC` 

Parameters: None

Schema:

```
{
  "total": int (should always be 20),
  "rides": [
    {
      "title": string,
      "city": string,
      "state": string,
      "stateAbbreviation": string,
      "country": string,
      "story": [
        {
          "type": "paragraph",
          "text": string
        },
        {
          "type": "image",
          "url": string,
          "alt": string,
          "caption": string,
          "license": string,
          "dimensions: {
            "width": int,
            "height": int
          }
        }
      ],
      "date": string,
      "url": string,
      "organization" : {
        "name": string,
        "url": string
      },
      "coordinates": {
        "latitude": float,
        "longitude": float
      },
      "distance": {
        "miles": float,
        "kilometers": float
      },
      "rideType": string
      "rideCardImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },
      "qrCode": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },    
      "featuredImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }    
      },
      "collections": [ string ],
      "ecRide": boolean
    },
    ...
  ]
}
````

Response example :
TBD

### Rides By State

Description: Return a JSON object of the 20 most viewed rides for each state

Endpoint: `/data/ridespot/v1/rideByState/{state}` 

Parameters:

| Name | Type | Description
|----------|:--------:|:--------:|
| state | string | This is the full text name of the state (ie, `Texas`). Ideally endpoint should accept USPS abbreviation code (ie, `TX`) |

Schema:

```
{
  "total": int (should always be 20),
  "state": {
    "full": string,
    "abbreviation": string
  },
  "rides": [
    {
      "title": string,
      "city": string,
      "state": string,
      "stateAbbreviation": string,
      "country": string,
      "story": [
        {
          "type": "paragraph",
          "text": string
        },
        {
          "type": "image",
          "url": string,
          "alt": string,
          "caption": string,
          "license": string,
          "dimensions: {
            "width": int,
            "height": int
          }
        }
      ],
      "date": string,
      "url": string,
      "organization" : {
        "name": string,
        "url": string
      },
      "coordinates": {
        "latitude": float,
        "longitude": float
      },
      "distance": {
        "miles": float,
        "kilometers": float
      },
      "rideType": string
      "rideCardImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },
      "qrCode": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },    
      "featuredImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }    
      },
      "collections": [ string ],
      "ecRide": boolean
    },
    ...
  ]
}
````

Response example :
TBD

### Collection

Description: Return a JSON object of the twenty most viewed rides associated with a collection

Endpoint: `/data/ridespot/v1/collection/{name}` 

Parameters:

| Name | Type | Description
|----------|:--------:|:--------:|
| name | string | This is the full text name of the collection (ie, `Adventure`). |

Schema:

```
{
  "total": int (should always be 20),
  "collection": string,
  "rides": [
    {
      "title": string,
      "city": string,
      "state": string,
      "stateAbbreviation": string,
      "country": string,
      "story": [
        {
          "type": "paragraph",
          "text": string
        },
        {
          "type": "image",
          "url": string,
          "alt": string,
          "caption": string,
          "license": string,
          "dimensions: {
            "width": int,
            "height": int
          }
        }
      ],
      "date": string,
      "url": string,
      "organization" : {
        "name": string,
        "url": string
      },
      "coordinates": {
        "latitude": float,
        "longitude": float
      },
      "distance": {
        "miles": float,
        "kilometers": float
      },
      "rideType": string
      "rideCardImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },
      "qrCode": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },    
      "featuredImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }    
      },
      "collections": [ string ],
      "ecRide": boolean
    },
    ...
  ]
}
````

Response example :
TBD


### Editor's Choice Collection

Description: Return a JSON object of the twenty latest, Editor's Choice rides associated with a collection

Endpoint: `/data/ridespot/v1/collectionEC/{name}` 

Parameters:

| Name | Type | Description
|----------|:--------:|:--------:|
| name | string | This is the full text name of the collection (ie, `Adventure`). |

Schema:

```
{
  "total": int (should always be 20),
  "collection": string,
  "rides": [
    {
      "title": string,
      "city": string,
      "state": string,
      "stateAbbreviation": string,
      "country": string,
      "story": [
        {
          "type": "paragraph",
          "text": string
        },
        {
          "type": "image",
          "url": string,
          "alt": string,
          "caption": string,
          "license": string,
          "dimensions: {
            "width": int,
            "height": int
          }
        }
      ],
      "date": string,
      "url": string,
      "organization" : {
        "name": string,
        "url": string
      },
      "coordinates": {
        "latitude": float,
        "longitude": float
      },
      "distance": {
        "miles": float,
        "kilometers": float
      },
      "rideType": string
      "rideCardImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },
      "qrCode": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }
      },    
      "featuredImage": {
        "url": string,
        "alt": string,
        "caption": string,
        "license": string,
        "dimensions: {
          "width": int,
          "height": int
        }    
      },
      "collections": [ string ],
      "ecRide": boolean
    },
    ...
  ]
}
````

Response example :
TBD


### Editor's Choice Collection

Description: Return a JSON object of the twenty latest, Editor's Choice rides associated with a collection

Endpoint: `/data/ridespot/v1/collections` 

Parameters: None

Schema:

```
{
  "collections": [
    string, string, ...
  ]
}
````

Response example :
TBD


## Technology Reference
PFB uses a mix of technologies to power its many websites. The flagship [main site](https://www.peopleforbikes.org) and new [City Ratings](https://cityratings.peopleforbikes.org) utilitze the following technologies: 

- [Next.js](https://www.nextjs.org) and [React.js](https://www.reactjs.org) power the front-end of the application
- [Prismic](https://prismic.io) is the headless CMS we're using (which provides the DB layer as well)
- All data moves between the front and back end using a single [GraphQL](https://graphql.org) endpoint that covers the entire Prismic data graph. The data is self-documenting and can be explored using using sample GraphQL queries using our [GraphiQL instance](https://peopleforbikes.prismic.io/graphql)
- Authentication is handled with [Auth0's passwordless infrastructure](https://auth0.com/passwordless), search utilitizes [Algolia](https://www.algolia.com), and payments provider [Stripe](https://www.stripe.com) handles payments.
- PFB also has campaign websites that utilize [Wix](https://www.wix.com/), [Shopify](https://www.shopify.com/), and [Webflow](https://webflow.com/) when appropriate. 

While the technical stack that powers Ridespot is not entirely known to our team, **it is expected that the forthcoming public Ridespot API will make its data available either using well-documented, legacy REST-based endpoints (outlined above) or new, self-documenting GraphQL endpoints.**

## Contribute
This is an RFC, which means any comments would be appreciated. This is a living document and needs your input to be finished!
