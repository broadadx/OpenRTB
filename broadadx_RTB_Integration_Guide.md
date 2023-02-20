# broadadx RTB Integration Guide

  * [1. Introduction](#1-Introduction)
    * [1.1 broadadx RTB?](#11-broadadx-rtb)
  * [2. OpenRTB Basics](#2-openrtb-basics)
    * [2.1 Transport](#21-Transport)
    * [2.2 Data Format](#22-data-format)
    * [2.3 OpenRTB Version HTTP Header](#23-openrtb-version-http-header)
  * [3. Bid Request Specification](#3-Bid-Request-Specification)
    * [3.1 Object Model](#31-object-model)
    * [3.2 Object Specifications](#32-object-specifications)
      * [3.2.1 Object: BidRequest](#321-object-bidrequest)
      * [3.2.2 Object: Imp](#322-object-imp)
      * [3.2.3 Object: Banner](#323-object-banner)
      * [3.2.4 Object: Native](#325-object-native)
      * [3.2.5 Object: Site](#326-object-site)
      * [3.2.6 Object: App](#327-object-app)
      * [3.2.7 Object: Device](#3210-object-device)
      * [3.2.8 Object: Geo](#3211-object-geo)
      * [3.2.9 Object: User](#3212-object-user)
  * [4. Bid Response Specification](#4-bid-response-specification)
    * [4.1 Object Model](#41-object-model)
    * [4.2 Object Specifications](#42-object-specifications)
      * [4.2.1 Object: BidResponse](#421-object-bidresponse)
      * [4.2.2 Object: SeatBid](#422-object-seatbid)
      * [4.2.3 Object: Bid](#423-object-bid)
    * [4.3 Substitution Macros](#43-치환substitution-macros)
  * [5. Native Specification](#5-native-Specification)
    * [5.1 Native markup Request Object](#51-Native-markup-Request-Object)
    * [5.2 Native markup Response Object](#52-Native-markup-Response-Object)
  * [6. Bid Request/Response Samples](#6-bid-requestresponse-samples)
    * [6.1 Bid Requests](#71-bid-requests)
      * [6.1.1 Example 1 (Image Banner Creative)](#711-example-1-(Image-Banner-Creative))
      * [6.1.2 Example 2 (Native Creative)](#712-example-2-(Native-Creative))
    * [6.2 Bid Responses](#72-bid-responses)
      * [6.2.1 Example 1 (Image Banner Creative Response)](#721-example-1-(Image-Banner-Creative-Response))
      * [6.2.2 Example 2 (Native Markup Returned Inline)](#722-example-2-(Native-Markup-Returned-Inline))
  
### 1. Introduction

#### 1.1 broadadx RTB 

broadadx is an OpenRTB protocol–based advertisement auction system operating between Demand-Side Providers (DSP) and Supply-Side Providers (SSP). 
broadadx currently supports OpenRTB 2.5 and OpenRTB Native-Ads-Specification 1.2. However, we do not fully support the Object from the spec but is limited to our definition in this document.


### 2. OpenRTB Basics

The diagram below shows interaction between exchange and bidder. Bid requests are created from the publisher’s mobile, and all requests will be available for all bidders. Once bidders return bid responses, auction algorithm will determine the winner of the bids and notify the winning bidder.
<img src="https://user-images.githubusercontent.com/125242569/218430210-f6a99f53-867f-4043-84f8-380085ce6a1e.png"/>

#### 2.1 전송

HTTP is a protocol for data transfer. Bid requests will be sent via POST requests, and winner notification will be delivered by GET response. All requests should be returned with HTTP 200, with a single exception of No Bid, which will be returned with 204 code.

#### 2.2 Data Format

JSON is a data format for bid requests and responses. Bid Request’s header must have Content-Type : application/json, and the same format must be used for bid responses

```
Content-Type: application/json
```

#### 2.3 OpenRTB Version HTTP Header

Include OpenRTB version to request header. broadadx is on OpenRTB 2.5

```
x-openrtb-version: 2.5
```

### 3. Bid Request Specification

RTB transactions are initiated when an exchange or other supply source sends a bid request to a bidder. The bid request consists of the top-level bid request object, at least one impression object, and may optionally include additional objects providing impression context

#### 3.1 Object Model

#### 3.2 Object Specifications

##### 3.2.1 Object: BidRequest

The top-level bid request object contains a globally unique bid request, auction ID, imp, app, or site. It is optional recommended to include attributes such as device, cur, and test.

 Name   | Type         | Required, Default  | Description                                                               
:-------|:-------------|:--------------|:--------------------------------------------------------------------------
 id     | string       | Required          | Unique ID of the bid request                                             
 imp    | object array | Required          | Array of Imp objects                                                  
 site   | object       | Required(or app)  | Either one of app or site must is required            
 app    | object       | Required(or site) | Either one of app or site must is required           
 device | object       | Required          | Details via a Device object about the user’s device to which the impression will be delivered.                        
 user   | object       |               | Details via a User object about the human user of the device; the advertising audience.                                                
 test   | integer      | Default 0      | Indicator of test mode in which auctions are not billable, where 0 = live mode, 1 = test mode.                                 
 at     | integer      | Default 2      | Auction type, where 1 = First Price, 2 = Second Price Plus. Exchange-specific auction types can be defined using values greater than 500.               
 tmax   | integer      |               | Maximum time in milliseconds to submit a bid to avoid timeout. This value is commonly communicated offline.                        
 cur    | string array | Default "USD"  | Array of allowed currencies for bids on this bid request using ISO-4217 alpha codes. Recommended only if the exchange accepts multiple currencies.                                    
 bcat   | string array |               | Blocked advertiser categories using the IAB content categories.
 badv   | string array |               | Block list of advertisers by their domains.                         

##### 3.2.2 Object: Imp

This object describes an ad placement or impression being auctioned.

 Name              | Type    | Required, Default    | Description                                                  
:------------------|:--------|:----------------|:-------------------------------------------------------------
 id                | string  | Required            | A unique identifier for this impression within the context of the bid request.   
 banner            | object  | Required           | At least one of banner, video, or native is required
 native            | object  | Required           | At least one of banner, video, or native is required
 instl             | integer | Default 0        | 1 = the ad is interstitial or full screen, 0 = not interstitial.                            
 tagid             | string  |                 | Identifier for specific ad placement or ad tag that was used to initiate the auction.            
 bidfloor          | integer | Default 0        | Minimum bid for this impression expressed in CPM.
 secure            |integer  |                | Flag to indicate if the impression requires secure HTTPS URL creative assets and markup, where 0 = non-secure, 1 = secure.</br> If omitted, the secure state is unknown, but non-secure HTTP support can be assumed.
        
        
##### 3.2.3 Object: Banner

 Name     | Type          | Required, Default | Description                                                                          
:---------|:--------------|:-------------|:---------------------------------------------------------------------------------------------------------
 w        | integer       | Required         | Width of the impression in pixels.                                                              
 h        | integer       | Required         | Height of the impression in pixels.                                                    
 btype    | integer array |              | Blocked banner ad types.            
 battr    | integer array |              | Blocked banner ad types.            
 pos      | integer       | Default 0     | Ad position on screen.                               
 mimes    | string array |         | Content MIME types supported. Popular MIME types may include “application/x-shockwave-flash”,“image/jpg”, and “image/gif”.  


##### 3.2.4 Object: Native

This object represents a native type impression. 

 Name    | Type          | Required, Default | Description                                                                                        
:--------|:--------------|:-------------|:---------------------------------------------------------------------------------------------------
 request | string        | Required         |Request payload complying with the Native Ad Specification.
 battr   | integer array |              | Blocked creative attributes.                    

##### 3.2.5 Object: Site

This object should be included if the ad supported content is a website as opposed to a non-browser application. A bid request must not contain both a Site and an App object.

 Name       | Type         | Required, Default | Description                                                    
:-----------|:-------------|:-------------|:---------------------------------------------------------------
 id         | string       | Required      | Exchange-specific site ID.                                
 name       | string       | Required         | name string Site name (may be aliased at the publisher’s request).                                                
 domain     | string       | Required         | Domain of the site (e.g., “mysite.foo.com”).
 pagecat    | string array |             |Array of IAB content categories that describe the current page
or view of the site. Refer to List 5.1.
 page| string       |          |  URL of the page where the impression will be shown.
 cat        | string array |              | Array of IAB content categories of the site. Refer to List 5.1.                                   
 sectioncat | string array |              | Array of IAB content categories that describe the current
section of the site. Refer to List 5.1.                               
 
                                     
##### 3.2.6 Object: App

This object should be included if the ad supported content is a non-browser application (typically in mobile) as opposed to a website. A bid request must not contain both an App and a Site object.


 Name       | Type         | Required, Default | Description                                               
:-----------|:-------------|:-------------|:----------------------------------------------------------
 id         | string       | Required         |   Exchange-specific app ID.                                  
 name       | string       | Required         | name string App name (may be aliased at the publisher’s request).                                               
 bundle     | string       | Required         | Application bundle or package name.
 domain     | string       |              | Domain of the app.                                    
 storeurl   | string       |              | App store URL for an installed app.                                        
 cat        | string array |              | Array of IAB content categories of the site. Refer to OpenRTB List 5.1.                            
 sectioncat | string array |              | Array of IAB content categories that describe the current
section of the app. Refer to List 5.1
 pagecat    | string array |             | Array of IAB content categories that describe the current page
or view of the app. Refer to List 5.1.
 ver        | string      |              | Application version. 
 privacypolicy |integer |                |Indicates if the app has a privacy policy, where 0 = no, 1 = yes.
 paid       | integer      |              | 0 = app is free, 1 = the app is a paid version.                               
 keywords  | string       |           | Comma separated list of keywords about the app.
         

##### 3.2.7 Object: Device

This object describes the publisher of the media in which the ad will be displayed.

 Name           | Type    | Required, Default | Description                                                     
:---------------|:--------|:-------------|:----------------------------------------------------------------
 ua             | string  |    Required          | Browser user agent string.                                                 
 geo            | object  |          | Location of the device assumed to be the user’s current location.                                               
 ip             | string  | Required            | IPv4 address closest to device.                                                    
 devicetype     | integer |          | The general type of device. Refer to List 5.21      
 make           | string  |              | make string Device make (e.g., “Apple”).                                       
 model          | string  |              | Device model (e.g., “iPhone”).                                     
 os             | string  |          | Device operating system (e.g., “iOS”).                         
 osv            | string  |              | Device operating system version (e.g., “3.1.2”).                                 
 h              | integer |              | Physical height of the screen in pixels.                                
 w              | integer |              | Physical width of the screen in pixels.                                   
 language       | string  |              | Browser language using ISO-639-1-alpha-2.                         
 carrier        | string  |              | Carrier or ISP (e.g., “VERIZON”)
 connectiontype | string  |              | Network connection type. Refer to List 5.22.
 ifa            | string  | recommended         | ID sanctioned for advertiser use in the clear.

##### 3.2.8 Object: Geo

This object encapsulates various methods for specifying a geographic location. When subordinate to a Device object, it indicates the location of the device which can also be interpreted as the user’s current location. When subordinate to a User object, it indicates the location of the user’s home base (i.e., not necessarily their current location).
 Name    | Type    | Required, Default | Description                                      
:--------|:--------|:-------------|:-------------------------------------------------
 lat     | float   |              | Latitude from -90.0 to +90.0, where negative is south.                                             
 lon     | float   |              | Longitude from -180.0 to +180.0, where negative is west.                                             
 type    | integer |              | Source of location data; recommended when passing lat/lon. Refer to List 5.20.
 country | string  |              | Country code using ISO-3166-1-alpha-3.           

##### 3.2.9 Object: User

This object contains information known or derived about the human user of the device.

 Name     | Type    | Required, Default | Description                                                                        
:---------|:--------|:-------------|:-----------------------------------------------------------------------------------
 id       | string  |              | Exchange-specific ID for the user.
 yob      | integer |              | Year of birth as a 4-digit integer                                                                   
 buyeruid | string  |              |Buyer-specific ID for the user as mapped by the exchange for the buyer. 


                                                                              
### 4. Bid Response Specification

#### 4.1 Object Model

At least one seatbid object is required, which contains at least one bid for an impression.

 Name        | Type   | Required, Default | Description                                                                                                                                           
:------------|:-------|:-------------|:------------------------------------------------------------------------------------------------------------------------------------------------------
 BidResponse | object | Required         | Top-level object.                                                                                                                                              
 seatbid     | object | Required         | Collection of bids made by the bidder on behalf of a specific seat.                                                                                              
 bid         | object | Required         | An offer to buy a specific impression under certain business terms.                                                                                             
 ext         | object |              | Extension of the OpenRTB specification. Exelbid uses ext field for Native creative response using JSON.


#### 4.2 Object Specifications

##### 4.2.1 Object: BidResponse

 Name    | Type         | Required, Default | Description                                                                              
:--------|:-------------|:-------------|:-----------------------------------------------------------------------------------------
 id      | string       | Required         | Bid Request의 ID                                                                 
 seatbid | object array | Required         |                                                                                          
 bidid   | string       |              | Bidder generated response ID to assist with logging/tracking.        
 cur     | string       | Required         | Bid currency using ISO-4217 alpha codes                                                          
 nbr     | integer      |              | Reason for not bidding. Refer to List 5.24.                                                    
 ext     |              |              | Extension of the OpenRTB specification. Exelbid uses ext field for Native creative response using JSON.

Top-level object that uses ID congruent to BidRequest’s ID. At least one seatbid object is required for a single imp object.

##### 4.2.2 Object: SeatBid

 Name | Type         | Required, Default | Description                                                                                                                                   
:-----|:-------------|:-------------|:----------------------------------------------------------------------------------------------------------------------------------------------
 bid  | object array | Required         | Response object to imp object.                                                                                                               
 seat | string       |              | ID of the bidder seat on whose behalf this bid is made.                                                                                                              
 ext  |              |              | Extension of the OpenRTB specification. Exelbid uses ext field for Native creative response using JSON.

##### 4.2.3 Object: Bid

 Name    | Type         | Required, Default | Description                                                                                                                                                                                 
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 id      | string       | Required         | Bidder generated bid ID to assist with logging/tracking.                                                                                                                                                  
 impid   | string       | Required         | ID of the Imp object in the related bid request.                                                                                                                                                          
 price   | float        | Required         | Bid price expressed as CPM.                                                                                                                                                                       
 adid    | string       |              | ID of a preloaded ad to be served if the bid wins.                                                                                                                                                                     
 nurl    | string       |              | Win notice URL called by the exchange if the bid wins.                                                                                                                                                                             
 adm     | string       |              | Optional means of conveying ad markup in case the bid wins.                                                                                                                                           
 adomain | string array | Required         | Advertiser domain for block list checking.                                                                                           
 bundle  | string       |              | Package name in case of application.                                                                                                                  
 iurl    | string       | Required         | URL without cache-busting to an image that is representative of the content of the campaign for ad quality/safety checking.                                                                                                                                                      
 cid     | string       | Required         | Campaign ID to assist with ad quality checking; the collection of creatives for which iurl should be representative.                                                                                                                                                                         
 crid    | string       | Required         | Creative ID to assist with ad quality checking.                                                                                                                                      
 cat     | string array |              | cat string array IAB content categories of the creative. Refer to List 5.1.                                                                                       
 attr    | integer array |              | Set of attributes describing the creative. Refer to List 5.3.                                                                                                           
 w       | integer      |              |Width of the creative in pixels.                                                                                                                                                                      
 h       | integer      |              | Height of the creative in pixels.                                                                                                                                         
 ext     | object       |              | Extension of the OpenRTB specification. Exelbid uses ext field for Native creative response using JSON.




#### 4.3 Win notice

1. OpenRTB without “nurl” (default).
In this scenario, “nurl” field is absent in bid responses and the system will record wins automatically when impression happens. </br>SSP has to replace ${AUCTION_PRICE} before the ad markup is being rendered.
2. OpenRTB with “nurl”
Win URL is represented as a separate bid response field. In case of 2nd price auction usage, “nurl” has to be called before the actual impression happens.

 Macro                   | Description                                               
:------------------------|:----------------------------------------------------------
 ${AUCTION_PRICE}        | Settlement price using the same currency and units as the bid.
                                           
                     

### 5. Native Specification

broadadx Native is based on OpenRTB Native Ads Specification 1.2. With few exceptions of video-related requests and responses, ExelBid complies with the standard.

#### 5.1 Native markup Request Object

Bid Request Specification. Refer to OpenRTB-Native-Ads-Specification 1.2 for more detais.


##### 5.1.1 Native Request Markup Object
Name    | Type         | Required, Default | Description                                                                     
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ver      | string       |          |Version	of	the	Native	Markup version	in	use.                
 layout   | integer       |         | The Layout ID of the native ad unit.
 adunit   | integer        |         |        The Ad unit ID of the native ad unit.
 plcmtcnt    | integer       |   1      |  The number of identical placements in this Layout.                      
 seq    | integer       |     0        | 0 for the first ad, 1 for the second ad, and so on.
 assets    | an array of </br>Asset Objects|      Required        | Any bid must comply with the array of elements expressed by the Exchange.

##### 5.1.2 Asset Object
Name    | Type         | Required, Default | Description    
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 id      | integer       | Required | Unique asset ID, assigned by exchange. Typically a counter for the array.     
 required   | integer     |     | Set to 1 if asset is required (exchange will not accept a bid without it).
 title   | object       |  recommended   | Title object for title assets.       
 img      | object       | recommended    | Image object for image assets. 
 data    | object       |     | Data object for ratings, prices etc. 

##### 5.1.2.1 Title Object
Name    | Type         | Required, Default | Description                       
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 len      | integer       | Required | Maximum length of the text in the title element.

##### 5.1.2.2 Image Asset Object
Name    | Type         | Required, Default | Description    
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 type      | integer       | optional | Type ID of the image element supported by the publisher.    
 w   | integer             |     | Width of the image in pixels
 wmin   | integer          |  recommended   | The minimum requested width of the image in pixels. 
 h      | integer          |     | Height of the image in pixels.
 hmin    | integer         | recommended  | The minimum requested height of the image in pixels. 
 mimes    | array of strings   |All types allowed| Whitelist of content MIME types supported. Popular MIME types include, but are not limited to “image/jpg” “image/gif”. 

##### 5.1.2.3 Data Asset Object
Name    | Type         | Required, Default | Description    
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 type      | integer       | Required | Type ID of the element supported by the publisher. The publisher
can display this information in an appropriate format. 
 len   | integer             |     | len optional integer - Maximum length of the text in the element’s response.


### 6. Bid Request/Response Samples

#### 6.1 Bid Requests

##### 6.1.1 Example 1 (Image Banner Creative)

```json
{
    "id": "4487159888663217854",
    "imp": [
        {
            "id": "1",
            "banner": {}
        }
    ],
    "site": {
        "page": "http://test.com/page1?param=value"
    },
    "device": {
        "ua": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8) AppleWebKit/537.13 (KHTML, like Gecko) Version/5.1.7 Safari/534.57.2",
        "ip": "1.1.1.1"
    }
}
```

##### 6.1.2 Example 2  (Native Creative)


```json
{
    "id": "4487159888663217854",
    "imp": [
        {
            "id": "1",
            "native": {
                "request": "{ \"native\": { \"plcmtcnt\": 5, \"assets\": [ { \"id\": 1, \"title\": { \"len\": 25 } }, { \"id\": 2, \"data\": {\u000a\"type\": 2, \"len\": 35 } }, { \"id\": 3, \"data\": { \"type\": 10, \"len\": 35 } }, { \"id\": 4, \"data\": { \"type\": 11, \"len\":\u000a35 } }, { \"id\": 5, \"img\": { \"wmin\": 50, \"hmin\": 50 } } ] } }"
            }
        }, {
            "id": "2",
            "native": {
                "request": "{ \"native\": { \"plcmtcnt\": 3, \"assets\": [ { \"id\": 1, \"title\": { \"len\": 40 } }, { \"id\": 2,\u000a\"required\": 1, \"data\": { \"type\": 2, \"len\": 30 } }, { \"id\": 3, \"data\": { \"type\": 8, \"len\": 35 } }, { \"id\": 4,\u000a\"video\": { \"mimes\": [\"video/mp4\"], \"minduration\": 10, \"maxduration\": 30, \"protocols\": [2] } } ] } }"
            }
        }
    ],
    "site": {
        "page": "http://test.com/page1?param=value"
    },
    "device": {
        "ua": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8) AppleWebKit/537.13 (KHTML, like Gecko) Version/5.1.7 Safari/534.57.2",
        "ip": "123.145.167.13"
    }
}
```

#### 6.2 Bid Responses

##### 6.2.1 Example 1 (Image Banner Creative Response)

```json
{
    "id": "4487159888663217854",
    "seatbid": [{
        "bid": [{
            "id": "QRh2T-YNIFk_0",
            "impid": "1",
            "price": 0.01,
            "adid": "823011",
            "nurl": "http://rtb.broadadx.com/win?i=QRh2T-YNIFk_0&price=${AUCTION_PRICE}",
            "adm": "<a href=\"http://rtb.broadadx.comclick?i=QRh2T-YNIFk_0\" target=\"_blank\"><img src=\"http://rtb.broadadx.com/n1/ad/300x250_EUNqbCsW.png\"
            width = \"300\" height=\"250\" border=\"0\" ></a><img src='http://rtb.broadadx.com/pixel?i=QRh2T-YNIFk_0' alt=' ' style='display:none'>",
            "adomain": ["broadadx.com"],
            "iurl": "http://xs.wowconversions.com/n1/ad/300x250_EUNqbCsW.png",
            "cid": "28734",
            "crid": "823011",
            "cat": ["IAB3-1"]

        }]

    }]

}
```

##### 6.2.2 Example 2 (Native Markup Returned Inline)

```json
{
    "id": "4487159888663217854",
    "seatbid": [
        {
            "bid": [
                {
                    "id": "890784660",
                    "impid": "1",
                    "price": 0.75,
                    "nurl": "http://ads.broadadx.com/win?i=zlyFxRMRjBk_0",
                    "adm": "{\"native\":{\"assets\":[{\"id\":1,\"title\":{\"text\":\"TextAd 1 Title\" }},{\"id\":2,\"data\":{\"value\":\"TextAd 1\u000aDescriprion\" }},{\"id\":3,\"data\":{\"value\":\"TextAd 1 Descriprion Line\u000a2\" }},{\"id\":4,\"data\":{\"value\":\"www.textad1.com\" }},{\"id\":5,\"img\":{\"url\":\"http://ads.broadadx.com/thumbnail?i=zlyFxRMRjBk_0\" }}],\"link\":{\"url\\u000a": "http://ads.broadadx.com/click?i=zlyFxRMRjBk_0\"},\"imptrackers\":[\"http://ads.broadadx.com/impression?i=zlyFxRMRjBk_0\"]}",
                    "adomain": ["broadadx.com"],
                    "iurl": "http://ads.broadadx.com/1.png"
                }, {
                    "id": "1309277383",
                    "impid": "1",
                    "price": 0.7,
                    "nurl": "http://ads.broadadx.com/win?i=zlyFxRMRjBk_1",
                    "adm": "{\"native\":{\"assets\":[{\"id\":1,\"title\":{\"text\":\"TextAd 2 Title\" }},{\"id\":2,\"data\":{\"value\":\"TextAd 2\u000aDescriprion\" }},{\"id\":3,\"data\":{\"value\":\"TextAd 2 Descriprion Line\u000a2\" }},{\"id\":4,\"data\":{\"value\":\"www.textad2.com\" }},{\"id\":5,\"img\":{\"url\":\"http://ads.abroadadx.com/thumbnail?i=zlyFxRMRjBk_1\" }}],\"link\":{\"url\\u000a": "http://ads.broadadx.com/click?i=zlyFxRMRjBk_1\"},\"imptrackers\":[\"http://ads.broadadx.com/impression?i=zlyFxRMRjBk_1\"]}",
                    "adomain": ["broadadx.com"],
                    "iurl": "http://ads.broadadx.com/1.png"
                }
            ]
        }
    ]
}
```
