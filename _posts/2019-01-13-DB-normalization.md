---
title: Database Normalization
categories: DB
tags: DB 정규화 Normalization Anomaly
---

# Overview 

**데이터베이스 정규화**는 RDB에서 중복을 최소화 하기 위해서 데이터를 구조화 하는 작업을 말한다.  
즉, 불필요한 데이터를 어떻게 제거하여 데이터를 논리적으로 저장할 것인지에 대한 이야기이다.  

# Data Abnormalities

아래 테이블은 아직 정규화 과정이 이뤄지지 않았으며, `EmployeeID`가 primary key이다.  

**Employee**
|  <center>EmployeeID</center> |  <center>SalesPerson</center> |  <center>SalesOffice</center> |<center>OfficeNumber</center>|<center>Customer1</center>|<center>Customer2</center>|<center>Customer3</center>
|:--------:|:--------:|--------:|:--------:|:--------:|:--------:|:--------:|:--------:|
| 1003 | Mary Smith | Chicago | 312-555-1212 | Ford | GM | |
| 1004 | John Hunt | New York | 212-555-1212 | Dell | HP | Apple |
| 1005 | Martin Hap | Chicago | 312-555-1212 | Boeing | | |

위 테이블을 통해 데이터가 중복되어 있을 때 relation 조작을 하게 되면 어떤 문제점이 생기는지 알아보려고 한다.

## Insert Anomaly
**삽입이상**이란 새로운 데이터를 추가할 때 원하지 않는 값도 함께 추가되는 현상을 말한다.  
- - -
예를들어, SalesOffice 값으로 현재 테이블에 없는 Atlanta를 추가하는 상황을 가정해보자.  

Atlanta를 추가하기 위해서는 그 SalesOffice에서 누가 일하는지 SalesPerson의 값을 알아야 하며, OfficeNumber도 알아야 한다. 즉, Atlanta 하나만 추가할 수는 없으며 EmployeeID,SalesPerson, SalesOffice... 등등의 값을 알아야 레코드를 추가 할 수 있다는 것이다.  
만약 이를 지키지 않고 SalesOffice에 Atlanta 값을 추가하려고 하면, 아직 정해지지 않은 나머지 속성인 SalesPerson, OfficeNumber... 등등에 `NULL`값과 같이 의도하지 않은 값들이 들어가게 된다.

## Delete Anomaly
**삭제이상**이란 어떤 값을 삭제할 때 원하지 않는 값도 함께 삭제되는 현상을 말한다.
- - -

예를들어, SalesPerson의 값이 John Hunt라는 데이터를 삭제한다고 가정해보자.

이를 위해서는 SalesPerson의 값이 John Hunt인 튜플 전체를 삭제해야 하는데, 그렇게 되면 SalesOffice의 값 중 하나인 New York도 함께 사라지게 된다.

## Update Anomaly
**갱신이상**이란 어떤 값을 갱신했을 때 다른 중복된 속성 값과 불일치 하는 현상을 말한다. 
- - - 

예를들어, OfficeNumber 속성 중 312-555-1212라는 데이터를 412-555-1212라고 바꾼다고 가정해보자.  

그럴 경우 첫번째 튜플의 값만 바꾸면 안 되고 첫번째 튜플과 세번째 튜플의 OfficeNumber 속성 값을 모두 바꾸어 주어야 한다. 만약, 두 튜플 중 하나만 바꾸게 된다면 동일한 속성값을 가져야 함에도 불구하고 다른 속성값을 가지게 된다(inconsistency).

# Database Normalization 
위에서 말한 이상 현상들을 피해서 데이터를 논리적으로 저장하기 위해 정규화 과정이 필요하다.  
정규화 방식에 따라 그 종류가 다양하다.  

## 1NF(First Normal Form)
> 모든 행에는 하나의 값(single value)이 있어야 하며, 중복 기능 열이 없어야 한다(unique).

1. 모든 행에는 하나의 값(single value)이 있어야 한다. 

|Customer ID|	First Name	|Surname	|Telephone Number|
|:--------:|:--------:|:--------:|:--------:|
|123|	Robert|	Ingram|	555-861-2025|
|456|	Jane|Wright|	555-403-1659 555-776-4100|
|789|	Maria|Fernandez|	555-808-9633|

위 테이블에서 두번째 튜플의 Telephone Number의 속성값을 보면 	555-403-1659 555-776-4100로 두 개가 존재한다. 행 도메인은 정확하게 한 개의 값만을 허용하기 때문에 위 테이블은 1NF를 만족하지 않는다.

2. 중복 기능 열이 없어야 한다(unique)

|Customer ID|	First Name	|Surname	| Tel. No.1	| Tel. No.2 | Tel. No.3 | 
|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|
|123	| Robert	| Ingram	| 555-861-2025 | | |
|456	|Jane	|Wright	|555-403-1659|555-776-4100|	555-403-1659|
|789	|Maria|	Fernandez	|555-808-9633	|

튜플 중에 Null 값을 가지는 속성값이 있기 때문에 1NF의 정의에 맞지 않고 Tel. No.1, Tel. No.2, Tel. No.3 속성은 완전히 동일한 도메인과 의미를 갖으므로 중복된 기능을 가진다.
따라서 두 가지 이유로 위 테이블은 1NF를 만족하지 않는다. 

맨 처음에 제시했던 테이블 역시 2번째 이유로 1NF를 만족하지 않는다.

### 1NF Design
1NF를 충족하지 않는 테이블은 2개의 테이블을 이용하여 나누도록 한다. 1NF에서 예시를 든 테이블의 경우 전화번호 테이블과 고객 테이블로 나눌 수 있을 것이다.

**Customer**
|Customer ID|	First Name	|Surname	|
|:--------:|:--------:|:--------:|
|123	| Robert	| Ingram	|
|456	|Jane	|Wright	|
|789	|Maria|	Fernandez	|

**Customer Telephone Number**
|Customer ID|	Telephone Number	|
|:--------:|:--------:|
|123	| 555-861-2025 | 
|456	| 555-403-1659 |
|456	| 555-776-4100 |
|789	| 555-808-9633 |

이렇게 테이블을 나눌 경우 Customer 테이블과 Customer Telephone Number 테이블을 연결시켜 주는 속성, 즉 Customer ID가 존재하게 된다. 그리고 한 명의 Customer가 전화번호 여러개를 가질 수 있으므로 두 테이블은 1:N의 관계가 존재하게 된다. 

같은 방식으로 맨 처음에 들었던 Employee 테이블을 1NF로 디자인하면 다음과 같다.

**Employee**
|  <center>EmployeeID</center> |  <center>SalesPerson</center> |  <center>SalesOffice</center> |<center>OfficeNumber</center>|
|:--------:|:--------:|:--------:|:--------:|
| 1003 | Mary Smith | Chicago | 312-555-1212 |
| 1004 | John Hunt | New York | 212-555-1212 |
| 1005 | Martin Hap | Chicago | 312-555-1212 |

**Employee Customer**
|EmployeeID|Customer|
|:--------:|:--------:|
|1003	| Ford | 
|1003	| GM |
|1004	| Dell |
|1004	| HP |
|1004	| Apple |
|1004	| Apple |
|1004	| Boeing |

그리고 **Employee Customer** 테이블의 속성값을 수정 및 새로 추가했다.
|CustomerID|EmployeeID|CustomerName|CustomerCity|PostalCode|
|:--------:|:--------:|:--------:|:--------:|:--------:|
|C1000|1003|Ford|Dearborn|48123|
|C1010|1003|GM|Detroit|48213|
|C1020|1004|Dell|Austin|78720|
|C1030|1004|HP|Palo Alto|94303|
|C1040|1004|Apple|Cupertino|95014|
|C1050|1005|Boeing|Chicago|60601|


## 2NF(Second Normal Form)
> 1NF를 만족하면서 후보키에 속하지 않은 속성들이 후보키에 속한 속성들 전체에 **함수 종속**인 경우이다.

추가적으로, 1NF 테이블에서 복합 후보키가 없으면 자동으로 2NF를 만족한다.  

* 후보키에 속하지 않은 속성들이 후보키에 속한 속성들 전체에 함수 종속인 경우

|종업원|기술|근무지|
|:--------:|:--------:|:--------:|
|Jones|Typing|114 Main Street|
|Jones|Shorthand|114 Main Street|
|Jones|Whittling|114 Main Street|
|Bravo|Light Cleaning|73 Industrial Way|
|Ellis|Alchemy|73 Industrial Way|
|Ellis|Flying|73 Industrial Way|
|Harrison|Light|Cleaning|73 Industrial Way|

우선 {종업원,기술}이라는 복합키가 튜플을 유일하게 구분하는 동시에 그 값을 유일하게 식별하는 최소한의 키이기 때문에 후보키가 될 수 있다. 

그러나 {근무지} 속성의 경우 후보키에 속하지 않은 속성이고 이 속성은 {종업원, 기술}이라는 두 복합키의 속성이 아닌 **{종업원}이라는 한 가지 속성**에만 영향을 받는다.

영향을 받는다는 말을 좀 더 자세하게 이야기 해보자면, 종업원 각각에 대해 오직 하나의 근무지가 정해진다. 즉 근무지에서 일하는 종업원은 여러명이 될 수 있지만, 종업원은 근무지가 하나로 결정된다.
이런 현상을 **함수 종속**이라 하며 위의 테이블에서 근무지 속성은 종업원 속성에 함수 종속적이라고 할 수 있다.

그러나 이런 테이블은 2NF라 할 수 없다. 1NF의 조건은 만족하지만 {근무지} 속성은 후보키의 일부인 {종업원}에 대해서만 함수 종속적이기 때문이다. 위에서 말했듯 2NF를 만족하려면 후보키 전체 속성에 대해서 함수 종속적이어야 한다.

위에서 1NF로 나누었던 Employee Customer 테이블의 경우 {CustomerID, EmployeeID}라는 복합키가 곧 후보키가 되고 나머지 속성인 CustomerName, CustomerCity, CustomerPostalCode는 후보키의 일부인 {CustomerID}에 대해서만 함수 종속적이므로 2NF를 만족하지 않는다.

그리고 Employee 테이블의 경우 후보키인 {EmployeeID}에 의해 구별되지만, 

### 2NF Design

2NF를 충족하지 않는 테이블은 **동일한 데이터**를 2개의 테이블을 이용하여 나누도록 한다. 2NF에서 예시를 든 테이블의 경우 {종업원} 속성을 후보키로 갖는 테이블 하나와 {종업원, 기술} 속성을 후보키로 갖는 테이블로 나눌 수 있을 것이다.

**종업원**
|**종업원**| 근무지 |
|:--------:|:--------:|
|Jones|114 Main Street|
|Bravo|73 Industrial Way|
|Ellis|73 Industrial Way|
|Harrison|73 Industrial Way|

**종업원의 기술**
|**종업원**|**기술**|
|:--------:|:--------:|
|Jones|Typing|
|Jones|Shorthand|
|Jones|Whittling|
|Bravo|Light Cleaning|
|Ellis|Alchemy|
|Ellis|Flying|
|Harrison|Light Cleaning|

이렇게 테이블을 나눌 경우 {근무지} 속성에 중복으로 인해 발생한 갱신 이상 문제를 해결할 수 있다.

같은 방식으로 1NF로 나누었던 Employee Customer 테이블을 2NF로 디자인하면 다음과 같다.

**Customer** 
|CustomerID|CustomerName|CustomerCity|CustomerPostalCode|
|:--------:|:--------:|:--------:|:--------:|:--------:|
|C1000|Ford|Dearborn|48123|
|C1010|GM|Detroit|48213|
|C1020|Dell|Austin|78720|
|C1030|HP|Palo Alto|94303|
|C1040|Apple|Cupertino|95014|
|C1050|Boeing|Chicago|60601|

**SalesStaffCustomer**
|CustomerID|EmployeeID|
|:--------:|:--------:|
|C1000|1003|
|C1010|1003|
|C1020|1004|
|C1030|1004|
|C1040|1004|
|C1050|1005|

## 3NF(Third Normal Form)
> 2NF를 만족하면서 테이블 내의 모든 속성이 기본 키에만 의존하며, 다른 후보키에 의존하지 않는다.

* 테이블 내의 모든 속성이 기본 키에만 의존하며, 다른 후보키에 의존하지 않는다.

|대회|연도|우승자|우승자 생년월일|
|:--------:|:--------:|:--------:|:--------:|
|Des Moines Masters|1998|Chip Masterson|14 March 1977|
|Indiana Invitational|1998|Al Fredrickson|21 July 1975|
|Cleveland Open|1999|Bob Albertson|28 September 1968|
|Des Moines Masters|1999|Al Fredrickson|21 July 1975|
|Indiana Invitational|1999|Chip Masterson|14 March 1977|

우선 이 테이블의 경우 2NF의 충족 조건을 만족하므로 2NF 정규화된 테이블이라고 할 수 있다. 
그리고 {대회, 연도}가 각 튜플을 유일하게 결정하므로 이 복합키는 곧 후보키(기본키)가 된다.

그러나 우승자가 {대회, 연도}에 종속되는 속성인지에 대해선 확신할 수 있을까?  
몰론 우승자 속성만 보았을 때는 {대회, 연도}에 따라 우승자가 달라지니까 함수 종속적이라고 할 수 있다. 하지만 우승자와 우승자 생년월일이 함께 있는 위 테이블의 경우 다시 생각해 볼 필요가 있다. 우승자의 생년월일의 경우 **몇 년도 무슨 대회에서 우승했는지**에 따라 달라지는 것이 아니라 **우승자가 누구냐**에 따라 달라지기 때문이다. 

이 테이블의 경우 우승자 생년월일이라는 속성이 기본키인 {대회, 연도} 라는 복합키에 의존하지 않고 우승자라는 속성에 의존하기 때문에 3NF라고 할 수 없다.

한편 위에서 1NF로 나누었던 Employee 테이블의 경우 기본키인 {EmployeeID}에 SalesOffice 속성과 OfficeNumber 속성이 의존하지 않기 때문에 3NF의 정의를 만족하지 않는다.
그리고 위에서 2NF로 나누었던 Customer 테이블에서도 CustomerPostalCode 속성의 경우 기본키인 {CustomerID}가 아니라 CustomerCity 속성에 의존하기 때문에 이 또한 3NF를 만족하지 않는다.

### 3NF Design
2NF를 충족하지 않는 테이블은 **별도의 종속성을 갖는 속성을 분리하여** 2개의 테이블을 이용하여 나누도록 한다. 3NF에서 예시를 든 테이블의 경우 대회 테이블과 우승자에 관한 정보 테이블로 나누어 두 테이블을 우승자로 연결시켜 준다.

**대회**
|대회|연도|우승자|
|:--------:|:--------:|:--------:|
|Des Moines Masters|1998|Chip Masterson|
|Indiana Invitational|1998|Al Fredrickson|
|Cleveland Open|1999|Bob Albertson|
|Des Moines Masters|1999|Al Fredrickson|
|Indiana Invitational|1999|Chip Masterson|

**우승자**
|우승자|우승자 생년월일|
|:--------:|:--------:|
|Chip Masterson|14 March 1977|
|1998|Al Fredrickson|21 July 1975|
|Bob Albertson|28 September 1968|
|1999|Al Fredrickson|21 July 1975|
|Chip Masterson|14 March 1977|

같은 방식으로 2NF로 나누었던 Customer 테이블을 3NF로 디자인하면 다음과 같다.

**Customer**
|CustomerID|CustomerName|CustomerPostalCode|
|:--------:|:--------:|:--------:|
|C1000|Ford|48123|
|C1010|GM|48123|
|C1020|Dell|78720|
|C1030|HP|94303|
|C1040|Apple|95014|
|C1050|Boeing|60601|

**PostalCode**
|CustomerPostalCode|CustomerCity|
|:--------:|:--------:|
|48123|Dearborn|
|48213|Detroit|
|78720|Austin|
|94303|Palo Alto|
|95014|Cupertino|
|60601|Chicago|

## BCNF(Boyce and Codd Normal Form)
3NF를 만족하는 테이블 중에서도 특별한 경우에만 BCNF를 만족하지 않는다.
> 3NF를 만족하면서 모든 결정자가 후보키이다.
*  모든 결정자가 후보키이다.

솔직히 BCNF랑 3NF 구분이 잘 안 된다.
나중에 DB 책 참고해서 추가하자...

This is because of the dependency Rate Type → Court in which the determining attribute Rate Type - on which Court depends - (1.) is neither a candidate key nor a superset of a candidate key and (2.) Court Type is no subset of Rate Type.

Dependency Rate Type → Court is respected since a Rate Type should only ever apply to a single Court.

### BCNF Design

## 4NF(fourth Normal Form)
> BCNF를 만족하면서 다치 종속 (multi-value dependent)가 존재하지 않는 경우를 말한다.

**Teaching database**
|수업|교재|강사|
|:--------:|:--------:|:--------:|
|AHA|Silberschatz|John D|
|AHA|Nederpelt|John D|
|AHA|Silberschatz|William M|
|AHA|Nederpelt|William M|
|AHA|Silberschatz|Christian G|
|AHA|Nederpelt|Christian G|
|OSO|Silberschatz|John D|
|OSO|Silberschatz|William M|

수업을 하는 강사들과 수업에서 사용하는 교재가 서로 독립적이므로 이 테이블은 다치 종속이 존재한다. 만약 AHA 수업에 교재 하나를 추가해야 한다면 John D, William M, Christian G 세 강사들에 대한 로우를 추가해 주어야 한다.
이러한 관계를 다치 종속을 갖는다고 한다.

### 4NF Design
따라서 위의 테이블의 경우 수업을 기준으로 다차 종속을 없애서 각각의 속성에 따라 테이블을 생성한다.

|수업|교재|
|:--------:|:--------:|
|AHA|Silberschatz|
|AHA|Nederpelt|
|AHA|Silberschatz|
|AHA|Nederpelt|
|AHA|Silberschatz|
|AHA|Nederpelt|
|OSO|Silberschatz|
|OSO|Silberschatz|

|수업|강사|
|:--------:|:--------:|
|AHA|John D|
|AHA|William M|
|AHA|William M|
|AHA|Christian G|
|AHA|Christian G|
|OSO|John D|
|OSO|William M|

## 5NF(fifth Normal Form)
> 4NF를 만족하면서 어떤 릴레이션에 존재하는 모든 조인 종속성이 릴레이션의 후보키를 통해서만 성립한다.

만약, 어떤 테이블 T가 T에 포함된 필드들의 부분집합을 포함하는 테이블들을 JOIN 하여 생성될 수 있다면 테이블 T는 join dependency를 갖는다.

# 참고자료
https://ko.wikipedia.org/wiki/%EC%A0%9C3%EC%A0%95%EA%B7%9C%ED%98%95