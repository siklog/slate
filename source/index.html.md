---
title: KSPAY API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
  - shell: 예시
  
toc_footers:
  - <a href='https://www.ksnet.co.kr'>KSNET</a>

includes:
  - errors

search: false

code_clipboard: true

meta:
  - name: description
    content: Documentation for the KSPAY API
---

# Introduction

KSPAY결제 연동을 위해 제공되는 API입니다.

* **문의**
    * 기술문의: pgmodule@ksnet.co.kr
    * 사업문의: ksnet001@ksnet.co.kr

# API 사용 안내

API 사용을 위해서는 KSNET PG사업부에 가맹점 등록 및 인증키 발급이 필요합니다.

https프로토콜만 허용되며, 공식적으로 지원하는 TLS버전은 1.2버전 이상입니다. (TLS1.2/1.3)

요청 파라미터의 Method는 POST, Content-Type은 application/json; charset=utf-8을 사용하셔야 합니다.

KSNET과 카드사 및 일부 은행의 경우 문자 인코딩을 EUC-KR로 사용합니다. 따라서, 파라미터를 UTF-8을 전달하더라도 EUC-KR로 표현 가능한 문자여야 합니다.

API URL 형식은 {API 서버}{API URI} 입니다.

* **API 서버**
    * 운영 서버: https://pay.ksnet.co.kr
    * 개발 서버: https://paydev.ksnet.co.kr

# API 인증

> 인증키 설정 예시

```예시
curl -v -X POST https://pgdev.ksnet.co.kr/kspay/webfep//api/v1/card/cancel \
  -H 'Content-Type: application/json' \
  -H 'Authorization: pgapi Mjk5OTE5OTk5OTpNQTAxOkE0RTc2QkRBMzM3RENDQTk1Mjk4RkI0OTVBODREMzY5'  \
  -D ...
```

API 사용을 위해 HTTP Request Header에 인증키 설정이 필요

**인증키 획득 절차**

* Step.1 - [KSTA Login](https://ksta.ksnet.co.kr)
* Step.2 - PG Shop info : PG상점정보 > 상점정보 > 상점아이디 입력 > Search > 해당 상점아이디 Click
* Step.3 - Check 'pgapi' key info : 하단 pgapi항목에서 키값을 획득하실 수 있습니다.

<aside class="notice">
{apiKey}가 유출될 경우 해커에 의한 부정취소 등이 발생할 수 있으므로 javascript 같은 client에 노출된 형태로 사용하지 않도록 주의하여야 합니다.
</aside>

# Response 공통

> 응답 예시

```cUrl
{
  "aid": "API 요청 고유값",
  "code": "API 응답 코드",
  "message": "API 응답 메시지",
  "data": {
    ...
  }
}
```

API 표준 응답은 다음과 같습니다.

http status : 200 OK

Content-Type: application/json;charset=utf-8

1. aid: 매 요청에 대하여 KSNET에서 부여하는 고유한 문자열입니다.
2. code: 요청에 대한 응답 코드입니다.
    * A0200: 처리 성공 (유일한 성공값으로, A0200외 다른 값은 실패를 의미합니다.)
    * A0201: 처리 실패 (처리 실패의 경우 반드시 data 항목의 응답코드와 응답메시지를 확인하시기 바랍니다.)
    * A0400: 요청 파라미터 오류
    * A0401: 인증 오류
    * A0403: 프로토콜 오류
    * A0500: 서버 오류 (KSNET 기술팀 문의)
    * A0999: 기타 오류
3. message: 코드보다 상세한 내용이 담긴 응답값입니다.
4. data: API별 세부 응답 값으로, JSON객체로 구성됩니다. 일부 API는 null값을 응답할 수도 있습니다.

# 1. 카드 API

## 1.1. 카드결제 취소 

카드 결제를 취소할 수 있는 API입니다.

매입 전 취소는 승인 취소로 처리되어 바로 환불이 이루어지지만, 매입된 거래건 또는 부분취소 거래건에 대해서는 +1~3영업일이 소요될 수 있습니다.

부분취소는 최대 '9회’까지 가능한 점을 참고해 주시기 바랍니다.

취소는 결제일 기준 6개월까지만 가능합니다.

**HTTP Request**

POST /kspay/webfep/api/v1/card/cancel <code><button type="button" onclick="javascript:window.open('https://pgdev.ksnet.co.kr/kspay/webfep//api/test/x.jsp?api=card_cancel');">테스트요청</button></code>

### 요청항목

> 신용카드 결제 취소 요청 예시

```예시
{
    "mid": "2999199999",
    "payload": "KST20240604S031",
    "cancelType": "FULL",
    "orgTradeKeyType": "TID",
    "orgTradeKey": "189189008016",
    "orgTradeDate": "",
    "cancelTotalAmount": "",
    "cancelTaxFreeAmount": "",
    "cancelSeq": ""
}
```

Name | Size | Description
--------- |---- | ------------------
mid*  | 10 |**상점아이디**<br>- 계약 완료 후 사업부를 통해 전달받은 상점아이디
payload  | * | **가맹점데이터**<br>- API 응답에 돌려받을 가맹점의 데이터입니다.
cancelType*  | 10  | **취소처리구분**<br>- FULL : 전체취소, PARTIAL : 부분취소
orgTradeKeyType*  | 10 | **거래키구분**<br>- TID : 거래번호취소, ORDER_NUMB : 주문번호취소
orgTradeKey*  | 50 | **원거래 키**<br>- 원거래 구분에 해당하는 TID 혹은 ORDER_NUMB 값
orgTradeDate  | 8  | **원거래일자**<br>- 주문번호 취소시 설정필요(yyyyMMdd)
cancelTotalAmount  | 9  | **취소 총금액**<br>- 선택 항목으로 부분취소 시 사용하는 필수값입니다.
cancelTaxFreeAmount  | 9  | **취소 면세금액**<br>- 부분취소 시 취소 총금액 중 면세금액이 있을 경우 설정
cancelSeq  | 1 | **취소일련번호**<br>- 선택 항목으로 부분취소 시 사용하는 필수값이며 해당 회차 부분취소 일련번호입니다.(1~9)


### 응답항목(Body > data)
> 신용카드 결제 취소 응답 예시

```예시
# 성공응답
{
  "aid":"WFVCCSE00000000000054321",
  "code":"A0200",
  "message":"Success",
  "data": {
    "payload": "KST20240604S031",
	"tid":"189189008016",
    "tradeDateTime":"20240604120147",
    "cancelAmount":"50004",
    "respCode":"0000",
    "respMessage":"승인취소완료/매입취소요청",
    "issuerCardType":"HYUNDAI",
    "issuerCardName":"현대카드",
    "purchaseCardType":"HYUNDAI",
    "purchaseCardName":"현대카드",
    "approvalNumb":"00876543",
    "cardNumb":"40176201XXXX825X",
    "expiryDate":"",
    "installMonth":"03"
}}

# 실패응답
{
  "aid":"WFVCCSE00000000000054320",
  "code":"A0201",
  "message":"Fail",
  "data": {
    "payload": "KST20240604S027",
	"tid":"Lbd124090990",
    "tradeDateTime":"20240604102151",
    "cancelAmount":"",
    "respCode":"7003",
    "respMessage":"취소거절기간경과/Tel:1544-6030",
    "issuerCardType":"UNDEFINED",
    "issuerCardName":"미등록카드사",
    "purchaseCardType":"UNDEFINED",
    "purchaseCardName":"미등록카드사",
    "approvalNumb":"7003",
    "cardNumb":"",
    "expiryDate":"",
    "installMonth":""
}}
```

Name | Size | Description
--------- | ---- | -----------------------
payload  | *  | 가맹점데이터
tid  | 12  | PG거래번호
tradeDateTime | 14 | 거래일시(yyyyMMddHHmmss)
cancelAmount  | 9  | 취소된금액
respCode  | 4  | 응답코드
respMessage  | 40  | 응답메시지
issuerCardType  | 20  | 발급사타입
issuerCardName  | 20  | 발급사명
purchaseCardType  | 20  | 매입사타입
purchaseCardName  | 20  | 매입사명
approvalNumb  | 12  | 승인번호
cardNumb  | 20  | 카드번호
expiryDate  | 4  | 유효기간
installMonth  | 2  | 할부개월수

## 1.2. 카드 인증

카드번호/유효기간/생년월일/비밀번호로 카드 소유자 인증을 진행하는 API입니다.

결제가 아닌 단순 인증만 진행하므로 유의하여 사용하시기 바랍니다.

서비스를 위해서는 사업부를 통해 별도의 계약이 필요합니다.

**HTTP Request**

POST /kspay/webfep/api/v1/card/cert <code><button type="button" onclick="javascript:window.open('https://pgdev.ksnet.co.kr/kspay/webfep//api/test/x.jsp?api=card_cert');">테스트요청</button></code>

### 요청항목

> 카드 인증 요청 예시

```예시
{
    "mid": "2999199999",
    "payload": "KST20240604S011",
    "orderNumb": "M20240604103105",
    "userName": "홍길동",
    "productName": "핑크테디",
    "cardNumb": "4017620101234567",
    "expiryDate": "3012",
    "password2": "99",
    "userInfo": "680102"
}
```

Name | Size | Description
--------- |---- | ------------------
mid*  | 10 |**상점아이디**<br>- 계약 완료 후 사업부를 통해 전달받은 상점아이디
payload  | * | **가맹점데이터**<br>- API 응답에 돌려받을 가맹점의 데이터입니다.
orderNumb*  | 50 |**가맹점 주문번호**
userName*  | 50 |**주문자명**
productName*  | 50 |**상품명**
cardNumb*  | 20 |**카드번호**
expiryDate*  | 4 |**유효기간**<br>- 연월(yyMM) 형태의 4자리 포맷을 사용합니다.
password2*  | 2 |**카드비밀번호 앞2자리**
userInfo*  | 10 |**사용자정보**<br>- 개인명의카드: 카드 소유자 생년월일 6자리(yyMMdd)<br>- 법인명의 법인카드: 사업자번호 10자리

### 응답항목(Body > data)
> 카드 인증 응답 예시

```예시
# 성공응답
{
  "aid":"WFVCCSE00000000000054111",
  "code":"A0200",
  "message":"Success",
  "data": {
    "payload": "KST20240604S011",
	"tid":"189189011016",
    "tradeDateTime":"20240604103147",
    "totalAmount":"50004",
    "respCode":"0000",
    "respMessage":"현대카드/OK: 00000000",
    "issuerCardType":"HYUNDAI",
    "issuerCardName":"현대카드",
    "purchaseCardType":"HYUNDAI",
    "purchaseCardName":"현대카드",
    "cardNumb":"40176201XXXX825X",
	"approvalNumb": "00000000",
	"expiryDate": "",
	"installMonth": "",
    "cardType":"CREDIT"
}}

# 실패응답
{
  "aid":"WFVCCSE00000000000054110",
  "code":"A0201",
  "message":"Fail",
  "data": {
    "payload": "KST20240604S027",
	"tid":"Lbd124090911",
    "tradeDateTime":"20240604103120",
    "totalAmount":"",
    "respCode":"6003",
    "respMessage":"비밀번호오류/비밀번호확인요망",
    "issuerCardType":"HYUNDAI",
    "issuerCardName":"현대카드",
    "purchaseCardType":"HYUNDAI",
    "purchaseCardName":"현대카드",
    "cardNumb":"",
	"approvalNumb": "",
	"expiryDate": "",
	"installMonth": "",
    "cardType":""
}}
```

Name | Size | Description
--------- | ---- | -----------------------
payload  | *  | 가맹점데이터
tid  | 12  | PG거래번호
tradeDateTime | 14 | 거래일시(yyyyMMddHHmmss)
totalAmount  | 9  | 총금액
respCode  | 4  | 응답코드
respMessage  | 40  | 응답메시지
issuerCardType  | 20  | 발급사타입
issuerCardName  | 20  | 발급사명
purchaseCardType  | 20  | 매입사타입
purchaseCardName  | 20  | 매입사명
approvalNumb  | 12  | 승인번호
cardNumb  | 20  | 카드번호
expiryDate  | 4  | 유효기간
installMonth  | 2  | 할부개월수
cardType  | 10  | 카드타입(CREDIT/CHECK/GIFT/PREPAID)

## 1.3. 카드 비인증 결제

카드번호/유효기간으로 결제를 요청하는 비인증 승인 API입니다.

서비스를 위해서는 사업부를 통해 별도의 계약이 필요합니다.

**HTTP Request**

POST /kspay/webfep/api/v1/card/pay/noncert <code><button type="button" onclick="javascript:window.open('https://pgdev.ksnet.co.kr/kspay/webfep//api/test/x.jsp?api=card_pay_noncert');">테스트요청</button></code>

### 요청항목

> 카드 비인증 결제 예시

```예시
{
    "mid": "2999199999",
    "payload": "KST20240604S021",
    "orderNumb": "M20240604103125",
    "userName": "홍길동",
    "userEmail": "test@test.com",
    "productType": "REAL",
    "productName": "핑크테디",
	"totalAmount": "1004",
    "cardNumb": "4017620101234567",
    "expiryDate": "3012",
	"installMonth": "00",
	"currencyType": "KRW"
}
```

Name | Size | Description
--------- |---- | ------------------
mid*  | 10 |**상점아이디**<br>- 계약 완료 후 사업부를 통해 전달받은 상점아이디
payload  | * | **가맹점데이터**<br>- API 응답에 돌려받을 가맹점의 데이터입니다.
orderNumb*  | 50 |**가맹점 주문번호**
userName*  | 50 |**주문자명**
userEmail  | 50 |**주주문자이메일**
productType*  | 10 |**상품구분**<br>- REAL : 실물상품, DIGITAL : 디지털컨텐츠
productName*  | 50 |**상품명**
totalAmount*  | 9 |**총금액**
taxFreeAmount  | 9 |**면세금액**<br>- 면세금액이 없으면 총금액 전체 과세처리
tax  | 9 |**세금**<br>- 세금항목이 없으면 일반과세 상점의 경우 10% 부가세 계산 = (총금액-면세금액) / 11
cardNumb*  | 20 |**카드번호**
expiryDate*  | 4 |**유효기간**<br>- 연월(yyMM) 형태의 4자리 포맷을 사용합니다.
installMonth*  | 2  | **할부개월수**
currencyType*  | 3  | **통화타입**<br>- KRW : 원화, USD : 달러(모든 금액을 1달러=1000으로 표기) 

### 응답항목(Body > data)
> 카드 비인증 결제 응답 예시

```예시
# 성공응답
{
  "aid":"WFVCCSE00000000000060121",
  "code":"A0200",
  "message":"Success",
  "data": {
    "payload": "KST20240604S021",
	"tid":"189189011016",
    "tradeDateTime":"20240604103121",
    "totalAmount":"1004",
    "respCode":"0000",
    "respMessage":"현대카드/OK: 00112345",
    "issuerCardType":"HYUNDAI",
    "issuerCardName":"현대카드",
    "purchaseCardType":"HYUNDAI",
    "purchaseCardName":"현대카드",
    "cardNumb":"40176201XXXX825X",
	"approvalNumb": "00112345",
	"expiryDate": "",
	"installMonth": "",
    "cardType":"CREDIT",
	"partCancelYn":"Y"
}}

# 실패응답
{
  "aid":"WFVCCSE00000000000060120",
  "code":"A0201",
  "message":"Fail",
  "data": {
    "payload": "KST20240604S020",
	"tid":"Lbd124090920",
    "tradeDateTime":"20240604103120",
    "totalAmount":"",
    "respCode":"8326",
    "respMessage":"승인거절/월사용한도초과",
    "issuerCardType":"HYUNDAI",
    "issuerCardName":"현대카드",
    "purchaseCardType":"HYUNDAI",
    "purchaseCardName":"현대카드",
    "cardNumb":"",
	"approvalNumb": "",
	"expiryDate": "",
	"installMonth": "",
    "cardType":"",
	"partCancelYn":""
}}
```

Name | Size | Description
--------- | ---- | -----------------------
payload  | *  | 가맹점데이터
tid  | 12  | PG거래번호
tradeDateTime | 14 | 거래일시(yyyyMMddHHmmss)
totalAmount  | 9  | 총금액
respCode  | 4  | 응답코드
respMessage  | 40  | 응답메시지
issuerCardType  | 20  | 발급사타입
issuerCardName  | 20  | 발급사명
purchaseCardType  | 20  | 매입사타입
purchaseCardName  | 20  | 매입사명
approvalNumb  | 12  | 승인번호
cardNumb  | 20  | 카드번호
expiryDate  | 4  | 유효기간
installMonth  | 2  | 할부개월수
cardType  | 10  | 카드타입(CREDIT/CHECK/GIFT/PREPAID)
partCancelYn  | 1  | 부분취소가능여부

## 1.4. 카드 구인증 결제

카드번호/유효기간으로 결제를 요청하는 비인증 승인 API입니다.

서비스를 위해서는 사업부를 통해 별도의 계약이 필요합니다.

**HTTP Request**

POST /kspay/webfep/api/v1/card/pay/oldcert <code><button type="button" onclick="javascript:window.open('https://pgdev.ksnet.co.kr/kspay/webfep//api/test/x.jsp?api=card_pay_oldcert');">테스트요청</button></code>

### 요청항목

> 카드 구인증 결제 예시

```예시
{
    "mid": "2999199999",
    "payload": "KST20240604S022",
    "orderNumb": "M20240604103126",
    "userName": "홍길동",
    "userEmail": "test@test.com",
    "productType": "REAL",
    "productName": "핑크테디",
	"totalAmount": "1004",
    "cardNumb": "4017620101234567",
    "expiryDate": "3012",
	"installMonth": "00",
	"currencyType": "KRW",
    "password2": "99",
    "userInfo": "680102"
}
```

Name | Size | Description
--------- |---- | ------------------
mid*  | 10 |**상점아이디**<br>- 계약 완료 후 사업부를 통해 전달받은 상점아이디
payload  | * | **가맹점데이터**<br>- API 응답에 돌려받을 가맹점의 데이터입니다.
orderNumb*  | 50 |**가맹점 주문번호**
userName*  | 50 |**주문자명**
userEmail  | 50 |**주주문자이메일**
productType*  | 10 |**상품구분**<br>- REAL : 실물상품, DIGITAL : 디지털컨텐츠
productName*  | 50 |**상품명**
totalAmount*  | 9 |**총금액**
taxFreeAmount  | 9 |**면세금액**<br>- 면세금액이 없으면 총금액 전체 과세처리
tax  | 9 |**세금**<br>- 세금항목이 없으면 일반과세 상점의 경우 10% 부가세 계산 = (총금액-면세금액) / 11
cardNumb*  | 20 |**카드번호**
expiryDate*  | 4 |**유효기간**<br>- 연월(yyMM) 형태의 4자리 포맷을 사용합니다.
installMonth*  | 2  | **할부개월수**
currencyType*  | 3  | **통화타입**<br>- KRW : 원화, USD : 달러(모든 금액을 1달러=1000으로 표기) 
password2*  | 2 |**카드비밀번호 앞2자리**
userInfo*  | 10 |**사용자정보**<br>- 개인명의카드: 카드 소유자 생년월일 6자리(yyMMdd)<br>- 법인명의 법인카드: 사업자번호 10자리

### 응답항목(Body > data)
> 카드 구인증 결제 응답 예시

```예시
# 성공응답
{
  "aid":"WFVCCSE00000000000060122",
  "code":"A0200",
  "message":"Success",
  "data": {
    "payload": "KST20240604S022",
	"tid":"189189011016",
    "tradeDateTime":"20240604103122",
    "totalAmount":"1004",
    "respCode":"0000",
    "respMessage":"현대카드/OK: 00212345",
    "issuerCardType":"HYUNDAI",
    "issuerCardName":"현대카드",
    "purchaseCardType":"HYUNDAI",
    "purchaseCardName":"현대카드",
    "cardNumb":"40176201XXXX825X",
	"approvalNumb": "00212345",
	"expiryDate": "",
	"installMonth": "",
    "cardType":"CREDIT",
	"partCancelYn":"Y"
}}

# 실패응답
{
  "aid":"WFVCCSE00000000000060123",
  "code":"A0201",
  "message":"Fail",
  "data": {
    "payload": "KST20240604S023",
	"tid":"Lbd124090923",
    "tradeDateTime":"20240604103123",
    "totalAmount":"",
    "respCode":"8326",
    "respMessage":"승인거절/월사용한도초과",
    "issuerCardType":"HYUNDAI",
    "issuerCardName":"현대카드",
    "purchaseCardType":"HYUNDAI",
    "purchaseCardName":"현대카드",
    "cardNumb":"",
	"approvalNumb": "",
	"expiryDate": "",
	"installMonth": "",
    "cardType":"",
	"partCancelYn":""
}}
```

Name | Size | Description
--------- | ---- | -----------------------
payload  | *  | 가맹점데이터
tid  | 12  | PG거래번호
tradeDateTime | 14 | 거래일시(yyyyMMddHHmmss)
totalAmount  | 9  | 총금액
respCode  | 4  | 응답코드
respMessage  | 40  | 응답메시지
issuerCardType  | 20  | 발급사타입
issuerCardName  | 20  | 발급사명
purchaseCardType  | 20  | 매입사타입
purchaseCardName  | 20  | 매입사명
approvalNumb  | 12  | 승인번호
cardNumb  | 20  | 카드번호
expiryDate  | 4  | 유효기간
installMonth  | 2  | 할부개월수
cardType  | 10  | 카드타입(CREDIT/CHECK/GIFT/PREPAID)
partCancelYn  | 1  | 부분취소가능여부

# 2. 실시간 계좌이체 API

## 2.1. 실시간 계좌이체 취소 

계좌이체 결제를 취소할 수 있는 API입니다.

결제 당일 취소는 즉시 처리되지만 결제 당일이 아닌 경우 계좌이체 원천사에 따라 +1~3영업일이 소요될 수 있습니다.

취소가능 기간은 결제기관별로 상이(3~6개월)합니다. 

**HTTP Request**

POST /kspay/webfep/api/v1/account/cancel <code><button type="button" onclick="javascript:window.open('https://pgdev.ksnet.co.kr/kspay/webfep//api/test/x.jsp?api=account_cancel');">테스트요청</button></code>

### 요청항목

> 실시간 계좌이체 취소 요청 예시

```예시
{
    "mid": "2999199990",
    "payload": "KST20240604S221",
    "cancelType": "FULL",
    "orgTradeKeyType": "TID",
    "orgTradeKey": "289189002031",
    "orgTradeDate": "",
    "cancelTotalAmount": "",
    "cancelSeq": ""
}
```

Name | Size | Description
--------- |---- | ------------------
mid*  | 10 |**상점아이디**<br>- 계약 완료 후 사업부를 통해 전달받은 상점아이디
payload  | * | **가맹점데이터**<br>- API 응답에 돌려받을 가맹점의 데이터입니다.
cancelType*  | 10  | **취소처리구분**<br>- FULL : 전체취소, PARTIAL : 부분취소
orgTradeKeyType*  | 10 | **거래키구분**<br>- TID : 거래번호취소, ORDER_NUMB : 주문번호취소
orgTradeKey*  | 50 | **원거래 키**<br>- 원거래 구분에 해당하는 TID 혹은 ORDER_NUMB 값
orgTradeDate  | 8  | **원거래일자**<br>- 주문번호 취소시 설정필요(yyyyMMdd)
cancelTotalAmount  | 9  | **취소 총금액**<br>- 선택 항목으로 부분취소 시 사용하는 필수값입니다.
cancelSeq  | 1 | **취소일련번호**<br>- 선택 항목으로 부분취소 시 사용하는 필수값이며 해당 회차 부분취소 일련번호입니다.(1~9)

### 응답항목(Body > data)
> 실시간 계좌이체 취소 응답 예시

```예시
# 성공응답
{
  "aid":"WFVCCSE00000000000054221",
  "code":"A0200",
  "message":"Success",
  "data": {
    "payload": "KST20240604S221",
	"tid":"289189002031",
    "tradeDateTime":"20240604120227",
    "cancelAmount":"1004",
    "respCode":"000",
    "respMessage":"농협은행/OK:이체취소",
    "agencyType":"5",
    "agencyName":"금융결제원",
    "approvalNumb":"011"
}}

# 실패응답
{
  "aid":"WFVCCSE00000000000054220",
  "code":"A0201",
  "message":"Fail",
  "data": {
    "payload": "KST20240604S220",
	"tid":"Lbd124092990",
    "tradeDateTime":"20240604120220",
    "cancelAmount":"",
    "respCode":"994",
    "respMessage":"취소거절/취소원거래없음",
    "agencyType":"5",
    "agencyName":"금융결제원",
    "approvalNumb":""
}}
```

Name | Size | Description
--------- | ---- | -----------------------
payload  | *  | 가맹점데이터
tid  | 12  | PG거래번호
tradeDateTime | 14 | 거래일시(yyyyMMddHHmmss)
cancelAmount  | 9  | 취소된금액
respCode  | 4  | 응답코드
respMessage  | 40  | 응답메시지
agencyType  | 1  | 대행사구분
agencyName  | 20  | 대행사명
approvalNumb  | 12  | 승인번호