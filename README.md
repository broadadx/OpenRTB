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
        * [3.2.2.1 Object: Ext](#3221-object-ext)
      * [3.2.3 Object: Banner](#323-object-banner)
      * [3.2.4 Object: Native](#325-object-native)
      * [3.2.5 Object: Site](#326-object-site)
      * [3.2.6 Object: App](#327-object-app)
      * [3.2.7 Object: Device](#3210-object-device)
      * [3.2.8 Object: Geo](#3211-object-geo)
      * [3.2.9 Object: User](#3212-object-user)
        * [3.2.9.1 Object: Ext](#32121-object-ext)
        * [3.2.9.2 Object: Eids](#32122-object-eids)
        * [3.2.9.3 Object: Uids](#32123-object-uids)
      * [3.2.10 Object: Data](#3213-object-data)
  * [4. 입찰 응답(Bid Response Specification)](#4-입찰-응답bid-response-specification)
    * [4.1 Object Model](#41-object-model)
    * [4.2 Object Specifications](#42-object-specifications)
      * [4.2.1 Object: BidResponse](#421-object-bidresponse)
      * [4.2.2 Object: SeatBid](#422-object-seatbid)
      * [4.2.3 Object: Bid](#423-object-bid)
      * [4.2.4 Object: Ext](#424-object-ext)
      * [4.2.5 Opt-out 설정](#425-opt-out-설정)
    * [4.3 치환(Substitution Macros)](#43-치환substitution-macros)
  * [5. Native 규격](#5-native-규격)
    * [5.1 입찰 요청](#51-입찰-요청)
    * [5.2 입찰 응답](#52-입찰-응답)
  * [6. 참조 목록표(Enumerated Lists Specification)](#6-참조-목록표enumerated-lists-specification)
  * [7. 비딩 요청/응답 예제(Bid Request/Response Samples)](#7-비딩-요청응답-예제bid-requestresponse-samples)
    * [7.1 Bid Requests](#71-bid-requests)
      * [7.1.1 Example 1 (이미지 광고 요청)](#711-example-1-(이미지-광고-요청))
      * [7.1.2 Example 2 (Native 광고 요청)](#712-example-2-(native-광고-요청))
    * [7.2 Bid Responses](#72-bid-responses)
      * [7.2.1 Example 1 (이미지 광고 응답)](#721-example-1-(이미지-광고-응답))
      * [7.2.2 Example 2 (Native 광고 응답)](#722-example-2-(native-광고-응답))
  
### 1. broadadx 소개

#### 1.1 broadadx RTB 

broadadx는 광고 구매자(DSP)와 퍼블리셔 인벤토리 판매자(SSP) 사이에서 OpenRTB 를 기반으로한 광고 경매 시스템입니다.
본 문서에서는 broadadx와의 실시간 인벤토리 거래를 위한 DSP/SSP 연동 가이드를 안내 합니다.

***OpenRTB Specification version 2.4, OpenRTB-Native-Ads-Specification 1.2*** 을 기반으로 합니다. 다만, 해당 Object를 완전히 지원하지 않으며, 현 규격에 정의된 범위에 한정합니다.



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

OpenRTB 버전을 입찰 요청의 헤더에 포합니다. broadadx OpenRTB 2.4버전을 포함합니다.

```
x-openrtb-version: 2.3
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
 banner            | object  | 필수           | banner, native 오브젝트중 하나 이상을 포함
 native            | object  | 필수           | banner, native 오브젝트중 하나 이상을 포함
 --instl             | integer | 기본값 0        | 전면광고 여부. 1일 경우 전면                               
 --tagid             | string  |                 | 노출 인벤토리(해당 지면, 유닛)의 고유한 식별자               
 bidfloor          | integer | 기본값 0        | Impression의 입찰 최저가 
 secure            |integer  |                | 광고 노출에 HTTPS URL 소재 마크업이 필요한지 여부를 나타내는 플래그 0 = 비보안(http), 1 = 보안(https)
        
        
##### 3.2.3 Object: Banner

디스플레이 광고. native, video가 아닌 일반 광고일 경우 반드시 포함되어야 합니다.

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
 id         | string       | 필수         | broadadx와 연결된 사이트 ID.                                    
 name       | string       | 필수         | 사이트 이름                                                    
 domain     | string       | 필수         | 사이트 도메인. 광고주에서 블럭 처리하는데 사용 할 수 있음
 cat        | string array |              | 전체 IAB 카테고리 리스트                                       
 sectioncat | string array |              | 현재 섹션의 IAB 카테고리 리스트                                
 pagecat    | string array |             | 사이트의 현재 페이지 또는 뷰를 설명하는 IAB 콘텐츠 카테고리
 publisher	 | object       |		           |publisher 상세 정보
                                     
##### 3.2.6 Object: App

광고가 전송될 지면이 어플리케이션일 경우 반드시 포함되어야 합니다. site오브젝트와 app오브젝트와 동시에 포함 할 수 없습니다.

 Name       | Type         | 필수, 기본값 | Description                                               
:-----------|:-------------|:-------------|:----------------------------------------------------------
 id         | string       | 필수         | broadadx와 연결된 앱ID.                                    
 name       | string       | 필수         | 앱 이름                                                   
 bundle     | string       | 필수         | Android 패키지명, IOS에서는 패키지명 혹은 어플리케이션 ID
 domain     | string       |              | 어플리케이션 도메인                                      
 storeurl   | string       |              | 앱스토어 URL                                              
 cat        | string array |              | 전체 IAB 카테고리 리스트                                  
 sectioncat | string array |              | 현재 섹션의 IAB 카테고리 리스트  
 pagecat    | string array |             | 사이트의 현재 페이지 또는 뷰를 설명하는 IAB 콘텐츠 카테고리
 ver        | integer      |              | 어플리케이션 버전       
 privacypolicy |integer |                |앱의 개인 정보 보호 정책 여부 0 = 아니요, 1 = 예.
 paid       | integer      |              | 0 = 무료, 1 = 유료                                        
 keywords  | string       |           | 앱에 대한 키워드를 쉼표로 구분한 목록  
 publisher	 | object       |		           |publisher 상세 정보
         

##### 3.2.7 Object: Device

하드웨어, 플랫폼, 위치, 통신사등 해당 기기와 관련되 정보를 제공합니다.

 Name           | Type    | 필수, 기본값 | Description                                                     
:---------------|:--------|:-------------|:----------------------------------------------------------------
 ua             | string  |              | User Agent                                                      
 geo            | object  | 필수         | 위치 정보                                                       
 dnt            | integer |              | 브라우에 do-not-track 이 false로 세팅이면 0, true이면 1로 전달  
 ip             | string  |              | IPv4 주소                                                       
 devicetype     | integer | 필수         | 디바이스 종류. IAB OpenRTB Spec 2.3 > 표 5.17 참조              
 make           | string  |              | 디바이스 제조사                                                 
 model          | string  |              | 디바이스 모델                                                   
 os             | string  | 필수         | 디바이스 운영체제 (android, ios)                                
 osv            | string  |              | 디바이스 운영체제 버전                                          
 h              | integer |              | 디바이스 넓이(pixel)                                            
 w              | integer |              | 디바이스 높이(pixel)                                            
 language       | string  |              | 브라우저 언어. ISO-639-1-alpha-2                                
 carrier        | string  |              | 통신사 또는 IP 어드레스로 부터 유도된 ISP(인터넷 서비스 제공자)
 connectiontype | string  |              | 네트워크 연결 종류 IAB OpenRTB Spec 2.3 > 표 5.18 참조          
 ifa            | string  | 권장         | 광고 트래킹 아이디(ex) android = gaid, ios = idfa)                  

##### 3.2.8 Object: Geo

Device 오브젝트와 User 오브젝트 두 군데, 혹은 한 군데 모두 적용 될 수 있습니다.

 Name    | Type    | 필수, 기본값 | Description                                      
:--------|:--------|:-------------|:-------------------------------------------------
 lat     | float   |              | 위도                                             
 lon     | float   |              | 경도                                             
 type    | integer |              | 데이터 출처. IAB OpenRTB Spec 2.3 > 표 5.16 참조
 country | string  |              | 국가 코드. ISO-3166-1-alpha-3.                   

##### 3.2.9 Object: User

디바이스 사용자의 정보를 나타냅니다.

 Name     | Type    | 필수, 기본값 | Description                                                                        
:---------|:--------|:-------------|:-----------------------------------------------------------------------------------
 id       | string  |              | broadadx에서 사용자를 구분하는 고유한 아이디                                        
 yob      | integer |              | 4자리 출생년도                                                                     
 gender   | string  |              | 성별 남자 = "M", 여자는 "F", 그외는 "O"                                            
 keywords | string  |              | key:value 형식을 콤마로 구분한 사용자 관련 키워드 리스트(ex = e_age:40,e_gender:F)
 geo      | object  |              | 위치 정보        
 ext      | object  |              | eids 포함. 3.2.12.1 Object: Ext 참조

##### 3.2.11.1 Object: Ext

 Name | Type          | 필수, 기본값     | Description                                                  
:-----|:--------------|:----------------|:-------------------------------------------------------------
 eids | object array  |                 | source & uids 포함 3.2.12.2 Object: Eids 참조

##### 3.2.11.2 Object: Eids

 Name   | Type          | 필수, 기본값             | Description                                                  
:-------|:--------------|:------------------------|:-------------------------------------------------------------
 source | string        | 기본값 "broadadx.com"     | 엑셀비드 주소
 uids   | object array  |                         | uids 포함. 3.2.12.3 Object: Uids 참조

##### 3.2.11.3 Object: Uids

 Name   | Type    | 필수, 기본값  | Description                                                  
:-------|:--------|:-------------|:-------------------------------------------------------------
 id     | string  |              | 사용자 고유 식별자
 atype  | integer | 기본값 3      | 식별자 타입. 3.2.12.4 식별자 타입 참조

##### 3.2.11.4 List:Agent Type

 Value  | Description                                                  
:-------|:-------------------------------------------------------------
 1      | 특정 웹 브라우저(쿠키 기반) 아이디
 2      | 사용자 기기 아이디 (device ID 등)
 3      | 사용자 개인 아이디 (이메일, 전화번호 등)
 500+   | 추가 코드                                                                    

##### 3.2.12 Object: Data

다양한 출처(ex : exchange 자체, 제 3의 제공자 등)로 제공되는 추가적인 사용자 데이터를 그룹화 하여 하나의 Data로 제공합니다.

 Name    | Type         | 필수, 기본값 | Description                                 
:--------|:-------------|:-------------|:--------------------------------------------
 id      | string       |              | broadadx에서 사용자를 구분하는 고유한 아이디
 name    | string       |              | 데이터 제공자                               
 segment | object array |              | 데이터를 포함하는 segment 오브젝트 배열     

##### 3.2.13 Object: Segment

상위 Data오브젝트에서 지정된 제공자로 부터의 일정 정보를 포함합니다.

 Name  | Type   | 필수, 기본값 | Description                       
:------|:-------|:-------------|:----------------------------------
 id    | string |              | 데이터를 구분하는 유니크한 아이디
 name  | string |              | 데이터 이름                       
 value | string |              | 데이터      

##### 3.2.14 Object: Pmp

 Imp오브젝트에 포함되며, Private MarketPlace 또는 직거래에서 RTB 프로토콜을 사용하기 위해 필요한
 정보를 포함합니다.

  Name              | Type   | 필수, 기본값 | Description                       
 :------------------|:-------|:------------|:----------------------------------
  private_auction   | string | 기본값 0     | 데이터를 구분하는 유니크한 아이디
  deals             | object |             | Imp에 해당하는 직거래 리스트를 내포하고 있는 deal 오브젝트 배열    

##### 3.2.15 Object: Deal

  구매자와 판매자간에 사전 협약된 거래로서, 상위 Imp가 이 계약 조건하에 가능하다는 것을 명시합니다.

   Name              | Type    | 필수, 기본값    | Description                                                  
  :------------------|:--------|:----------------|:-------------------------------------------------------------
   id                | string  | 필수            | BidRequest 오브젝트 안에서 imp를 구분하기 위한 고유 식별자   
   bidfloor          | integer | 기본값 0        | Impression의 입찰 최저가                                     
   bidfloorcur       | string  | 기본값 "USD"    | ISO–4217 알파벳 코드를 사용하여 명시되어야 합니다         
   wseat             | string array |            | 경매에 참여할 수 있는 입찰자격 코드 리스트.
   wadomain          | string array |            | 입찰할 수 있는 광고주 도메인 리스트.
   at                | integer |                 | 낙찰 타입. 입찰 금액들 중 낙찰 금액 결정 순위                     

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
 nbr     | integer      |              | IAB OpenRTB Spec 2.3 > 표 5.19 참조                                                      
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
 cat     | string array |              | 광고물의 컨텐츠 카테고리 목록. IAB OpenRTB Spec 2.3 > 표 5.1 참조                                                                                                                            
 attr    | integer array |              | 광고물 속성. OpenRTB Spec 2.3 > 표 5.3 참조                                                                                                                                                  
 w       | integer      |              | 광고물 넓이(pixel)                                                                                                                                                                           
 h       | integer      |              | 광고물 높이(pixel)                                                                                                                                                                           
 ext     | object       |              | 규격 표준을 벗어나는 OpenRTB 주체가 동의한 경우 이 오브젝트로 규격의 유연성을 제공합니다 동의한 경우 이 오브젝트로 규격의 유연성을 제공합니다.<br>broadadx에서는 주로 Native 응답에 사용한다.


##### 4.2.4 Object: Ext

Bid Object의 Ext. 네이티브 요청시의 응답객체, optouturl(광고 정보 표시 링크 URL)등이 포함 됩니다.

  Name    | Type         | 필수, 기본값 | Description                                                                                                                                                                                  
 :--------|:-------------|:-------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  native      | object       |          | 네이티브 요청시의 응답 오브젝트                                                                                                         
  optouturl   | string       | 권장         | 맞춤형 광고일 경우의 opt-out 설정 url.

##### 4.2.5 Opt-out 설정

  ※ 온라인 맞춤형 광고 개인정보보호 가이드(방송통신위원회-2017)에 따라서 온라인 맞춤형 광고 사업자는 _**맞춤형 광고의 경우**_ 해당 광고에 대해 opt-out 설정할 수 있는 방법을 제공하여야 합니다.
  - 이미지 배너의 경우는 Response bid->adm에 직접 아이콘및 링크가 적용되어야 합니다.
  - Native의 경우는 bid-ext에 optouturl을 응답하면 sdk에서 표시 하도록 가이드합니다.


#### 4.3 치환(Substitution Macros)

Win Noti(낙찰통보) URL은 입찰자에 의 해 결정됩니다. 입찰자는 낙찰시 특정 정보(ex: 낙찰 가격)를 전달 받기 위해서 몇가지 치환 매크로를 URL에 삽입합니다.<br> broadadx는 해당 매크로들에 적합한 데이터가 발견되면 치환하여 낙찰 통보를 합니다. 대체할 값이 없다면 URL에서 삭제됩니다.

 Macro                   | Description                                               
:------------------------|:----------------------------------------------------------
 ${AUCTION_ID}           | BidRequest의 id                                           
 ${AUCTION_ID:B64}       | BidRequest의 id. (Base64로 인코딩)                        
 ${AUCTION_BID_ID}       | BidResponse의 id                                          
 ${AUCTION_BID_ID:B64}   | BidResponse의 id (Base64로 인코딩)                        
 ${AUCTION_IMP_ID}       | 입찰 요청 imp의 ID. Imp의 id                              
 ${AUCTION_IMP_ID:B64}   | 입찰 요청 imp의 ID. Imp의 id (Base64로 인코딩)            
 ${AUCTION_SEAT_ID}      | 입찰자의 입찰자 코드. seatbid의 seat                      
 ${AUCTION_SEAT_ID:B64}  | 입찰자의 입찰자 코드. seatbid의 seat (Base64로 인코딩)    
 ${AUCTION_AD_ID}        | 입찰자가 제공하는 광고의 ID bid의 adid.                   
 ${AUCTION_AD_ID:B64}    | 입찰자가 제공하는 광고의 ID bid의 adid. (Base64로 인코딩)
 ${AUCTION_PRICE}        | 낙찰 가격                                                 
 ${AUCTION_PRICE:B64}    | 낙찰 가격 (Base64로 인코딩)                               
 ${AUCTION_CURRENCY}     | 입찰에 사용한 통화                                        
 ${AUCTION_CURRENCY:B64} | 입찰에 사용한 통화 (Base64로 인코딩)                      

특정 매크로가 인코딩 되어야 함을 나타내기 위해서는 이름 뒤에 :X를 추가하는데 이는 익스체인지와 비더간에 상호 동의가 되어야 합니다. broadadx에서는 Base64로 인코딩을 지원합니다.

### 5. Native 규격

broadadx Native는 OpenRTB-Native-Ads-Specification 1.0을 기본으로 구성되었습니다. 상기 버전에 일부 제외(video 관련 요청, 응답)하고 충실히 적용되었습니다.

#### 5.1 입찰 요청

입찰 요약 규격(상세 정보는 OpenRTB-Native-Ads-Specification 1.0 참조)

<table>
<tr>
  <th>Field</th>
  <th>Scope</th>
  <th>Field</th>
  <th>Scope</th>
  <th>Field</th>
  <th>Scope</th>
</tr>
<tr>
  <td>ver</td>
  <td>string; optional</td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>layout</td>
  <td>integer; recommended</td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>adunit</td>
  <td>integer; recommended</td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>plcmtcnt</td>
  <td>integer; optional</td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>seq</td>
  <td>integer; optional</td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td rowspan="16">assets</td>
  <td rowspan="16">array of objects;required</td>
  <td>id</td>
  <td>integer; required</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>required</td>
  <td>integer; optional</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>title</td>
  <td>object; optional</td>
  <td>len</td>
  <td>integer; required</td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
</tr>
<tr>
  <td rowspan="7">img</td>
  <td rowspan="7">object; optional</td>
  <td>type</td>
  <td>integer; optional</td>
</tr>
<tr>
  <td>w</td>
  <td>integer; optional</td>
</tr>
<tr>
  <td>wmin</td>
  <td>integer; recommended</td>
</tr>
<tr>
  <td>h</td>
  <td>integer; optional</td>
</tr>
<tr>
  <td>hmin</td>
  <td>integer; recommended</td>
</tr>
<tr>
  <td>mimes</td>
  <td>array of strings; required</td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
</tr>
<tr>
  <td><strike>video</strike></td>
  <td><strike>object; optional</strike></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td rowspan="3">data</td>
  <td rowspan="3">object; optional</td>
  <td>type</td>
  <td>integer; required</td>
</tr>
<tr>
  <td>len</td>
  <td>integer; optional</td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
</table>

#### 5.2 입찰 응답

입찰 응답 규격(상세 정보는 OpenRTB-Native-Ads-Specification 1.0 참조)<br>
broadadx에서는 두가지 입찰 옵션 규격을 제공합니다. 기본적으로 adm 필드 안에 serialized string 으로 포함하거나, 또는 bid->ext->native 형식으로 native object는 ext object 아래에 포함합니다.(7.2.3 Example 2 – 네이티브 광고 응답 참조)

<table>
<tr>
  <th>Field</th>
  <th>Scope</th>
  <th>Field</th>
  <th>Scope</th>
  <th>Field</th>
  <th>Scope</th>
  <th>Field</th>
  <th>Scope</th>
</tr>
<tr>
  <td rowspan="25">native</td>
  <td rowspan="25">object;required</td>
  <td>ver</td>
  <td>integer; optional</td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td rowspan="17">assets</td>
  <td rowspan="17">array of objects;required</td>
  <td>id</td>
  <td>integer; required</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>required</td>
  <td>integer; optional</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td rowspan="2">title</td>
  <td rowspan="2">object; optional</td>
  <td>text</td>
  <td>string; required</td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
</tr>
<tr>
  <td rowspan="4">img</td>
  <td rowspan="4">object; optional</td>
  <td>url</td>
  <td>string; required</td>
</tr>
<tr>
  <td>w</td>
  <td>integer; recommended</td>
</tr>
<tr>
  <td>h</td>
  <td>integer; recommended</td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
</tr>
<tr>
  <td><strike>video</strike></td>
  <td><strike>object; optional</strike></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td rowspan="3">data</td>
  <td rowspan="3">object; optional</td>
  <td>label</td>
  <td>string; optional</td>
</tr>
<tr>
  <td>value</td>
  <td>string; required</td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
</tr>
<tr>
  <td rowspan="4">link</td>
  <td rowspan="4">object; optional</td>
  <td>url</td>
  <td>string; required</td>
</tr>
<tr>
  <td>clicktrackers[]</td>
  <td>array of strings; required</td>
</tr>
<tr>
  <td><strike>fallback</strike></td>
  <td><strike>string; optional</strike></td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
  <td></td>
  <td></td>
</tr>

<tr>
  <td rowspan="4">link</td>
  <td rowspan="4">object; required</td>
  <td>url</td>
  <td>string; required</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>clicktrackers[]</td>
  <td>array of strings; required</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td><strike>fallback</strike></td>
  <td><strike>string; optional</strike></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>imptrackers[]</td>
  <td>array of strings;optional</td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td><strike>jstracker</strike></td>
  <td><strike>string; optional</strike></td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
<tr>
  <td>ext</td>
  <td>object; optional</td>
  <td></td>
  <td></td>
  <td></td>
  <td></td>
</tr>
</table>

### 6. 참조 목록표(Enumerated Lists Specification)

*OpenRTB 2.3 Specification 참조*

### 7. 비딩 요청/응답 예제(Bid Request/Response Samples)

#### 7.1 Bid Requests

##### 7.1.1 Example 1 (이미지 광고 요청)

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

##### 7.1.2 Example 2 (Native 광고 요청)

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

#### 7.2 Bid Responses

##### 7.2.1 Example 1 (이미지 광고 응답)

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

##### 7.2.2 Example 2 (Native 광고 응답)

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

