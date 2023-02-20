# broadadx RTB 연동 가이드

  * [1. broadadx 소개](#1-broadadx-소개)
    * [1.1 broadadx RTB?](#11-broadadx-rtb)
  * [2. OpenRTB Basics](#2-openrtb-basics)
    * [2.1 전송](#21-전송)
    * [2.2 Data Format](#22-data-format)
    * [2.3 OpenRTB Version HTTP Header](#23-openrtb-version-http-header)
  * [3. 입찰 요청(Bid Request Specification)](#3-입찰-요청bid-request-specification)
    * [3.1 Object Model (요청 오브젝트 모델 설명)](#31-object-model-요청-오브젝트-모델-설명)
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
  * [4. 입찰 응답(Bid Response Specification)](#4-입찰-응답bid-response-specification)
    * [4.1 Object Model](#41-object-model)
    * [4.2 Object Specifications](#42-object-specifications)
      * [4.2.1 Object: BidResponse](#421-object-bidresponse)
      * [4.2.2 Object: SeatBid](#422-object-seatbid)
      * [4.2.3 Object: Bid](#423-object-bid)
    * [4.3 치환(Substitution Macros)](#43-치환substitution-macros)
  * [5. Native 규격](#5-native-규격)
    * [5.1 입찰 요청](#51-입찰-요청)
    * [5.2 입찰 응답](#52-입찰-응답)
  * [6. 비딩 요청/응답 예제(Bid Request/Response Samples)](#6-비딩-요청응답-예제bid-requestresponse-samples)
    * [6.1 Bid Requests](#71-bid-requests)
      * [6.1.1 Example 1 (이미지 광고 요청)](#711-example-1-(이미지-광고-요청))
      * [6.1.2 Example 2 (Native 광고 요청)](#712-example-2-(native-광고-요청))
    * [6.2 Bid Responses](#72-bid-responses)
      * [6.2.1 Example 1 (이미지 광고 응답)](#721-example-1-(이미지-광고-응답))
      * [6.2.2 Example 2 (Native 광고 응답)](#722-example-2-(native-광고-응답))
  
### 1. broadadx 소개

#### 1.1 broadadx RTB 

broadadx는 광고 구매자(DSP)와 퍼블리셔 인벤토리 판매자(SSP) 사이에서 OpenRTB 를 기반으로한 광고 경매 시스템입니다.
본 문서에서는 broadadx와의 실시간 인벤토리 거래를 위한 DSP/SSP 연동 가이드를 안내 합니다.

***OpenRTB Specification version 2.5, OpenRTB-Native-Ads-Specification 1.2*** 을 기반으로 합니다. 다만, 해당 Object를 완전히 지원하지 않으며, 현 규격에 정의된 범위에 한정합니다.



### 2. OpenRTB Basics

다음 그림은 익스체인지와 비더간의 OpenRTB 상호작용을 보여줍니다.


광고 요청은 매체 사이트에서 발생됩니다. 매 광고 요청마다 비딩 요청이 모든 비더들에게 전파되고 비더로 부터 온 응답들은 일반적인 경매 룰에 의해 평가됩니다. 


당첨자(Winner)는 경매 성공을 통보받고 광고 마크업(markup)이 익스체인지로 전달됩니다. 다른 상호작용(블럭리스트 동기, 통신 제어 등)은 다음 제안에 포함되거나 이미 OpenRTB에 정의 되어있습니다.
<img src="https://user-images.githubusercontent.com/125242569/218430210-f6a99f53-867f-4043-84f8-380085ce6a1e.png"/>

#### 2.1 전송

기본 프로토콜은 HTTP입니다. HTTP POST가 입찰 요청에 사용됩니다. 낙찰 통보는 HTTP GET입니다. No Bid(광고 없음) 경우 HTTP코드 204리턴, 이 외 경우는 모든 요청에 HTTP 200 코드가 리턴 되어야 합니다.

#### 2.2 Data Format

JSON을 입찰 요청과 응답의 전문으로 사용합니다. 입찰 요청 *Content-type* 으로 *Content-Type: application/json* 이 지정되며, 입찰 응답 포맷도 같아야 합니다.

```
Content-Type: application/json
```

#### 2.3 OpenRTB Version HTTP Header

OpenRTB 버전을 입찰 요청의 헤더에 포합니다. broadadx OpenRTB 2.5버전을 포함합니다.

```
x-openrtb-version: 2.5
```

### 3. 입찰 요청(Bid Request Specification)

RTB 시작은 입찰 요청을 보내면서 시작됩니다. BidRequest는 하나 이상의 Imp(impression) Object로 구성되며, Imp에 대한 추가 정보를 추가하여 연동합니다.

#### 3.1 Object Model (요청 오브젝트 모델 설명)

#### 3.2 Object Specifications

##### 3.2.1 Object: BidRequest

입찰 최상위 오브젝트. 입찰 고유 값인 id, imp, app or site, device의 필수 오브젝트가 포함되어야 합니다. 필요에 따라 통화 종류를 나타내는 cur 값들, 또 테스트 입찰값인 test 등이 권장 사항으로 포함됩니다.

 Name   | Type         | 필수, 기본값  | Description                                                               
:-------|:-------------|:--------------|:--------------------------------------------------------------------------
 id     | string       | 필수          | 입찰에 대한 유니크 아이디                                                 
 imp    | object array | 필수          | imp 오브젝트 배열                                                         
 site   | object       | 필수(or app)  | app 개체와 site 오브젝트 둘 중 하나가 반드시 포함              
 app    | object       | 필수(or site) | app 개체와 site 오브젝트 둘 중 하나가 반드시 포함                
 device | object       | 필수          | 광고가 전송될 디바이스 특성, 종류등을 설명합니다.                         
 user   | object       |               | 사용자 정보를 나타냅니다.                                                 
 test   | integer      | 기본값 0      | 0 or 1로 기본값은 0, 1일 경우 테스트 연동                                 
 at     | integer      | 기본값 2      | 낙찰 타입. 입찰 금액들 중 낙찰 금액 결정 순위 1 = First Price, 2 = Second Price               
 tmax   | integer      |               | 입찰에 참여하기 위한 밀리세턴드 단위 최대 지연시간                        
 cur    | string array | 기본값 "USD"  | ISO–4217 코드의 단위통화 리스트                                     
 bcat   | string array |               | 제외되어야 할 광고주 카테고리 리스트
 badv   | string array |               | 제외되어야 할 광고주의 최상위 도메인 리스트                             

##### 3.2.2 Object: Imp

입찰 대상이 되는 광고의 위치나 광고 종류 등을 나타냅니다.

 Name              | Type    | 필수, 기본값    | Description                                                  
:------------------|:--------|:----------------|:-------------------------------------------------------------
 id                | string  | 필수            | BidRequest 오브젝트 안에서 imp를 구분하기 위한 고유 식별자   
 banner            | object  | 필수           | banner, native 오브젝트중 하나 이상을 포함</br>*Banner 일 경우 필수
 native            | object  | 필수           | banner, native 오브젝트중 하나 이상을 포함</br> *native 일 경우 필수
 instl             | integer | 기본값 0        | 전면광고 여부. 1일 경우 전면                               
 tagid             | string  |                 | 노출 인벤토리(해당 지면, 유닛)의 고유한 식별자               
 bidfloor          | integer | 기본값 0        | Impression의 입찰 최저가 
 secure            |integer  |                | 광고 노출에 HTTPS URL 소재 마크업이 필요한지 여부를 나타내는 플래그 0 = 비보안(http), 1 = 보안(https)
        
        
##### 3.2.3 Object: Banner

 Name     | Type          | 필수, 기본값 | Description                                                                          
:---------|:--------------|:-------------|:---------------------------------------------------------------------------------------------------------
 w        | integer       | 필수         | 광고의 넓이(pixel)                                                                   
 h        | integer       | 필수         | 광고의 높이(pixel)                                                                   
 btype    | integer array |              | 제외 되어야 할 광고물 종류                   
 battr    | integer array |              | 제외 되어야 할 광고물 속성                   
 pos      | integer       | 기본값 0     | 광고 위치                                      
 mimes    | string array |         | 지원되는 콘텐츠 MIME 유형. 인기 있는 MIME 유형은 다음과 같습니다 "응용 프로그램/x-shock wave-flash"를 포함 "image/jpg" 및 "image/gif"  


##### 3.2.4 Object: Native

Native 형식의 Impression을 나타냅니다. Open RTB Native Spec에 의해 응답 되어야 합니다.

 Name    | Type          | 필수, 기본값 | Description                                                                                        
:--------|:--------------|:-------------|:---------------------------------------------------------------------------------------------------
 request | string        | 필수         | Native Ad Spec 을 준수하는 요청 페이로드<br>Open RTB Native Spec 에 의거한 Json stirng이 반영된다.
 battr   | integer array |              | 제외 되어야 할 광고물 속성                                 

##### 3.2.5 Object: Site

광고가 전송될 지면이 웹사이트일 경우 반드시 포함되어야 합니다. site오브젝트와 app오브젝트와 동시에 포함 할 수 없습니다.

 Name       | Type         | 필수, 기본값 | Description                                                    
:-----------|:-------------|:-------------|:---------------------------------------------------------------
 id         | string       | 필수      | 사이트 ID                                   
 name       | string       | 필수         | 사이트 이름                                                    
 domain     | string       | 필수         | 사이트 도메인
 pagecat    | string array |             | 사이트의 현재 페이지 또는 뷰를 설명하는 IAB 콘텐츠 카테고리
 page| string       |          |  현재 사용자가 보고 있는 페이지
 cat        | string array |              | 전체 IAB 카테고리 리스트                                       
 sectioncat | string array |              | 현재 섹션의 IAB 카테고리 리스트                                
 
                                     
##### 3.2.6 Object: App

광고가 전송될 지면이 어플리케이션일 경우 반드시 포함되어야 합니다. site오브젝트와 app오브젝트와 동시에 포함 할 수 없습니다.

 Name       | Type         | 필수, 기본값 | Description                                               
:-----------|:-------------|:-------------|:----------------------------------------------------------
 id         | string       | 필수         |   앱 ID                                   
 name       | string       | 필수         | 앱 이름                                                   
 bundle     | string       | 필수         | Android 패키지명, IOS에서는 패키지명 혹은 어플리케이션 ID
 domain     | string       |              | 어플리케이션 도메인                                      
 storeurl   | string       |              | 앱스토어 URL                                              
 cat        | string array |              | 전체 IAB 카테고리 리스트                                  
 sectioncat | string array |              | 현재 섹션의 IAB 카테고리 리스트  
 pagecat    | string array |             | 사이트의 현재 페이지 또는 뷰를 설명하는 IAB 콘텐츠 카테고리
 ver        | string      |              | 어플리케이션 버전       
 privacypolicy |integer |                |앱의 개인 정보 보호 정책 여부 0 = 아니요, 1 = 예.
 paid       | integer      |              | 0 = 무료, 1 = 유료                                        
 keywords  | string       |           | 앱에 대한 키워드를 쉼표로 구분한 목록  
         

##### 3.2.7 Object: Device

하드웨어, 플랫폼, 위치, 통신사등 해당 기기와 관련되 정보를 제공합니다.

 Name           | Type    | 필수, 기본값 | Description                                                     
:---------------|:--------|:-------------|:----------------------------------------------------------------
 ua             | string  |    필수          | User Agent                                                      
 geo            | object  |          | 위치 정보                                                        
 ip             | string  | 필수            | IPv4 주소                                                       
 devicetype     | integer |          | 디바이스 종류 (IAB OpenRTB Spec 2.5 > 표 5.21 참조)              
 make           | string  |              | 디바이스 제조사 (e.g. "Apple")                                                 
 model          | string  |              | 디바이스 모델 (e.g. "iPhone")                                                   
 os             | string  |          | 디바이스 운영체제 (e.g. "iOS")                                
 osv            | string  |              | 디바이스 운영체제의 버전 (e.g. "9.0")                                          
 h              | integer |              | 디바이스 넓이(pixel)                                            
 w              | integer |              | 디바이스 높이(pixel)                                            
 language       | string  |              | 브라우저 언어. ISO-639-1-alpha-2                                
 carrier        | string  |              | 통신사 또는 IP 어드레스로 부터 유도된 ISP(인터넷 서비스 제공자)
 connectiontype | string  |              | 네트워크 연결 종류 IAB OpenRTB Spec 2.5 > 표 5.22 참조          
 ifa            | string  | 권장         | IDFA 또는 GAID 와 같은 값의 광고 트래킹을 위한 유저 ID

##### 3.2.8 Object: Geo

Device 오브젝트에 적용 될 수 있습니다.

 Name    | Type    | 필수, 기본값 | Description                                      
:--------|:--------|:-------------|:-------------------------------------------------
 lat     | float   |              | 위도                                             
 lon     | float   |              | 경도                                             
 type    | integer |              | 데이터 출처. IAB OpenRTB Spec 2.5 > 표 5.20 참조
 country | string  |              | 국가 코드. ISO-3166-1-alpha-3.                   

##### 3.2.9 Object: User

디바이스 사용자의 정보를 나타냅니다.

 Name     | Type    | 필수, 기본값 | Description                                                                        
:---------|:--------|:-------------|:-----------------------------------------------------------------------------------
 id       | string  |              | broadadx에서 사용자를 구분하는 고유한 아이디                                        
 yob      | integer |              | 4자리 출생년도                                                                     
 buyeruid | string  |              |DSP 또는 SSP와 매핑된 유저 ID


                                                                              
### 4. 입찰 응답(Bid Response Specification)

#### 4.1 Object Model

기본적으로 하나의 입찰 요청에 대해 하나의 입찰 응답이 되어야 합니다.

 Name        | Type   | 필수, 기본값 | Description                                                                                                                                           
:------------|:-------|:-------------|:------------------------------------------------------------------------------------------------------------------------------------------------------
 BidResponse | object | 필수         | 최상위                                                                                                                                                
 seatbid     | object | 필수         | 상위 BidResponse에 하나 이상의 seatbid가 존재해야 한다.                                                                                               
 bid         | object | 필수         | 상위 seatbid에는 하나 이상의 bid가 존재해야 한다.                                                                                                     
 ext         | object |              | 규격 표준을 벗어나는 OpenRTB 주체가 동의한 경우 이 오브젝트로 규격의 유연성을 제공합니다.<br>broadadx에서는 주로 Json 형식의 Native 응답에 사용합니다.

#### 4.2 Object Specifications

##### 4.2.1 Object: BidResponse

 Name    | Type         | 필수, 기본값 | Description                                                                              
:--------|:-------------|:-------------|:-----------------------------------------------------------------------------------------
 id      | string       | 필수         | Bid Request의 ID                                                                         
 seatbid | object array | 필수         |                                                                                          
 bidid   | string       |              | 응답의 ID로 입찰자가 응답을 추적하기 위해 사용함. 입찰자에 의해 선택됩니다.              
 cur     | string       | 필수         | ISO–4217 코드의 단위통화.                                                                
 nbr     | integer      |              | 미입찰 사유 IAB OpenRTB Spec 2.5 > 표 5.24 참조                                                      
 ext     |              |              | 규격 표준을 벗어나는 OpenRTB 주체가 동의한 경우 이 오브젝트로 규격의 유연성을 제공합니다

최상위 오브젝트로 id는 BidRequest의 ID를 그대로 사용합니다. 최소 하나의 seatbid 오브젝트가 필수이며, 하나의 imp에 대한 입찰을 포함합니다.

##### 4.2.2 Object: SeatBid

 Name | Type         | 필수, 기본값 | Description                                                                                                                                   
:-----|:-------------|:-------------|:----------------------------------------------------------------------------------------------------------------------------------------------
 bid  | object array | 필수         | imp 대한 응답 오브젝트                                                                                                                        
 seat | string       |              | 입찰하는 입찰 자격 코드                                                                                                                       
 ext  |              |              | 규격 표준을 벗어나는 OpenRTB 주체가 동의한 경우 이 오브젝트로 규격의 유연성을 제공합니다 동의한 경우 이 오브젝트로 규격의 유연성을 제공합니다

##### 4.2.3 Object: Bid

 Name    | Type         | 필수, 기본값 | Description                                                                                                                                                                                  
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 id      | string       | 필수         | 입찰자가 트래킹에 사용할 유니크한 아이디                                                                                                                                                     
 impid   | string       | 필수         | 응답에 대한 요청 Imp 오브젝트의 id                                                                                                                                                           
 price   | float        | 필수         | CPM 단위의 입찰 가격                                                                                                                                                                         
 adid    | string       |              | 낙찰시 전송될 광고 ID                                                                                                                                                                        
 nurl    | string       |              | 낙찰시 통보 URL                                                                                                                                                                              
 adm     | string       |              | 광고 마크업                                                                                                                                                                                  
 adomain | string array | 필수         | 광고주 최상위 도메인(광고주 필터링에 사용)                                                                                                                                                   
 bundle  | string       |              | 어플리케이션일 경우 패키지 명(앱광고등)                                                                                                                                                      
 iurl    | string       | 필수         | 광고 컨텐츠 확익을 위한 샘플 이미지 URL                                                                                                                                                      
 cid     | string       | 필수         | 광고 캠페인 ID                                                                                                                                                                               
 crid    | string       | 필수         | 광고물 ID                                                                                                                                                                                    
 cat     | string array |              | 광고물의 컨텐츠 카테고리 목록. IAB OpenRTB Spec 2.5 > 표 5.1 참조                                                                                                                            
 attr    | integer array |              | 광고물 속성. OpenRTB Spec 2.5 > 표 5.3 참조                                                                                                                                                  
 w       | integer      |              | 광고물 넓이(pixel)                                                                                                                                                                           
 h       | integer      |              | 광고물 높이(pixel)                                                                                                                                                                           
 ext     | object       |              | 규격 표준을 벗어나는 OpenRTB 주체가 동의한 경우 이 오브젝트로 규격의 유연성을 제공합니다 동의한 경우 이 오브젝트로 규격의 유연성을 제공합니다.<br>broadadx에서는 주로 Native 응답에 사용한다.




#### 4.3 낙찰 알림 (Win notice)

1. "nurl" 없음(기본값)
</br>입찰응답에 "nurl" 필드가 없을 경우 노출이 발생하면 자동으로 시스템에서 낙찰을 기록합니다.
</br>SSP는 광고마트업을 렌더링하기 전에 {AUCTION_PRICE}를 대체해야합니다.
2. "nurl" 있음
</br> Win Noti(낙찰통보) URL이 별도의 입찰응답 필드로 표시됩니다. 
</br>2nd price auction을 사용하는 경우 "nurl" 실제 노출이 발생하기 전에 호출되어야 합니다.

 Macro                   | Description                                               
:------------------------|:----------------------------------------------------------
 ${AUCTION_PRICE}        | 낙찰 가격                                                 
                     

### 5. Native 규격

broadadx Native는 OpenRTB-Native-Ads-Specification 1.2을 기본으로 구성되었습니다. 상기 버전에 일부 제외하고 충실히 적용되었습니다.

#### 5.1 입찰 요청

입찰 요약 규격(상세 정보는 OpenRTB-Native-Ads-Specification 1.2 참조)

##### 5.1.1 Native Request Markup Object
Name    | Type         | 필수, 기본값 | Description                                                                     
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ver      | string       |          | 네이티브 요청 마크업 버전                       
 layout   | integer       |         | 
 adunit   | integer        |         |        
 plcmtcnt    | integer       |         |                        
 seq    | integer       |             | 
 assets    | an array of </br>Asset Objects|      필수        | 입찰에 필요한 네이티브 구성요소를 정의한 assets 객체

##### 5.1.2 Asset Object
Name    | Type         | 필수, 기본값 | Description    
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 id      | integer       | 필수 | 고유한 asset id                 
 required   | integer     |     | 1인경우 필수값  
 title   | object       |  권장   | 타이틀 요청 asset 객체      
 img      | object       | 권장    | 이미지 요청 asset 객체
 data    | object       |     | 데이터 요청 asset 객체

##### 5.1.2.1 Title Object
Name    | Type         | 필수, 기본값 | Description                       
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 len      | integer       | 필수 | 최대 타이틀 길이            
##### 5.1.2.2 Image Asset Object
Name    | Type         | 필수, 기본값 | Description    
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 type      | integer       | 필수 | 이미지 타입</br>1: 로고 또는 앱 아이콘</br>3: 메인 이미지         
 w   | integer             |     | 이미지 width
 wmin   | integer          |  권장   | 이미지 height  
 h      | integer          |     | 이미지 최소 width
 hmin    | integer         | 권장  | 이미지 최소 height  
 mimes    | array of strings   |모든유형| 
##### 5.1.2.3 Data Asset Object
Name    | Type         | 필수, 기본값 | Description    
:--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 type      | integer       | 필수 | 데이터 타입
 len   | integer             |     | 데이터 최대 길이


### 6. 비딩 요청/응답 예제(Bid Request/Response Samples)

#### 6.1 Bid Requests

##### 6.1.1 Example 1 (이미지 광고 요청)

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

##### 6.1.2 Example 2 (Native 광고 요청)

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

##### 6.2.1 Example 1 (이미지 광고 응답)

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

##### 6.2.2 Example 2 (Native 광고 응답)

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
