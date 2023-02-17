>내가 쇼핑몰을 설립했다고 가정해보자. 결제 수단이 무통장 입금밖에 없다면 아무도 내 쇼핑몰을 이용하지 않을 것이다. 그렇다고 카드결제를 하기위해 수많은 카드기업들과 계약을 할수도 없는 노릇이다.
>쉽게 카드결제를 도입하기 위해서 존재하는것이 결제대행사 PG(payment gateway)이다. 우리는 PG와 계약해 카드 결제서비스를 손쉽게 구축할 수 있다.

## PG(결제 대행사)

> PG사는 여러 카드사와 계약을 채결하고 개인 사업자에게 수수료를 받으며 결제 및 지불을 대행하는 회사를 말한다. 카드 결제 뿐만 아니라 계좌이체, 소액결제 등 다양한 결제를 지원하며, 매출을 집계해 사업체 관리를 용이하게 해준다.

## 포트원(아임포트)

>카드회사가 많은것처럼 PG회사또한 많이 있고, 요즘에는 간편결제(카카오페이, 네이버페이)와 같이 카드사를 직접 경유하지 않는 서비스 등 결제에 신경쓸게 많아졌다.
>포트원은 모든 PG회사와 간편결제를 쉽게 계약과 연동을 지원해주고, 테스트또한 쉽게 구현할수 있다.
>최근 회사명을 포트원으로 변경했는데 구글검색에 다른회사가 나오고 있다. 전 회사 이름인 아임포트로 검색해주자

## PG직접 계약과 포트원의 결제 FLOW 차이

### PG직접 계약

![](https://3026939543-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FwWX2hlvRZLZrXeH1aacF%2Fuploads%2Fgit-blob-b2adfb934072237d6fca4fddc0cba90027cea5b5%2Fimage.png?alt=media)

PG와 직접 계약했을때의 결제 FLOW이다.
결제에 필요한 정보(유저 정보, 물품 정보)를 가지고 PG사에 결제 요청을 하면 PG사는 해당 결제에 대한 정보와 결제에 사용할 url 주소, 인증키를 반환해준다. 해당 url주소로 들어가 결제를 마치면 PG사에서 카드사로 결제 정보가 전달되고, 결제가 완료되었다면 결제 정보를 반환받는다.

### 포트원과 계약

![](https://3026939543-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FwWX2hlvRZLZrXeH1aacF%2Fuploads%2Fgit-blob-23e3d8d8040deedc6974922657d8efa6850a8a3d%2Fimage.png?alt=media)

포트원을 이용했을때의 결제 FLOW이다.
결제에 필요한 정보를 가지고 클라이언트가 포트원측이 만들어준 SDK를 이용해 결제 요청을 한다. 결제가 완료되면, 포트원측에서 해당 데이터를 전달 받고, 클라이언트와 서버에 각각 데이터를 송신한다.

포트원과 PG직접계약의 큰 차이점은 결제에 사용하는 인증키를 포트원에서 처리해줘 결제 FLOW가 줄어든다는 장점이 있다.

## 포트원 테스트

포트원서비스를 하려면 회원가입을 해야한다. 회원가입 후 테스트를 진행할 결제 대행사를 추가 해야한다. 테스트 > NHN KCP > NHN KCP 를 선택 추가 후 결제채널 이름은 마음에 들도록 지으면 된다.
결제대행사를 등록한뒤엔 식별코드와 API Key가 필요하다. 내 아래 코드에서 식별코드를 내 식별코드로 변경해주고, API Key는 미리 기록해 두자

### 식별코드, API 키 발급
![](https://velog.velcdn.com/images/kisuk623/post/1fe3ceb5-05ef-447d-a368-e3bace90af6c/image.png)
![](https://velog.velcdn.com/images/kisuk623/post/483c4011-9dac-4773-bdf0-c12ef32a4d9e/image.png)


### 클라이언트 결제 예시 코드

```html
// 결제 예시 코드
<!DOCTYPE html>
<html lang="en">
<head>
    <!-- jQuery -->
    <script type="text/javascript" src="https://code.jquery.com/jquery-1.12.4.min.js" ></script>
    <!-- iamport.payment.js -->
    <script type="text/javascript" src="https://cdn.iamport.kr/js/iamport.payment-1.2.0.js"></script>
    <script>
        var IMP = window.IMP; 
        IMP.init("imp67011510"); // 가맹점 식별코드
      
        var today = new Date();   
        var hours = today.getHours(); // 시
        var minutes = today.getMinutes();  // 분
        var seconds = today.getSeconds();  // 초
        var milliseconds = today.getMilliseconds();
        var makeMerchantUid = hours +  minutes + seconds + milliseconds;
        

        function requestPay() {
            IMP.request_pay({
                pg : 'kcp', // PG사 코드표에서 선택
                pay_method : 'card', // 결제 방식
                merchant_uid: "IMP"+makeMerchantUid, // 결제 고유 번호
                name : '당근 10kg', // 제품명
                amount : 1004, // 가격
                buyer_email : 'Iamport@chai.finance',
                buyer_name : '아임포트 기술지원팀',
                buyer_tel : '010-1234-5678',
                buyer_addr : '서울특별시 강남구 삼성동',
                buyer_postcode : '123-456'
            }, function (rsp) { // callback
                if (rsp.success) {
                    console.log(rsp);
                } else {
                    console.log(rsp);
                }
            });
        }
    </script>
    <meta charset="UTF-8">
    <title>Sample Payment</title>
</head>
<body>
    <button onclick="requestPay()">결제하기</button> <!-- 결제하기 버튼 생성 -->
</body>
</html>
```

포트원 서비스를 이용하기 위한 클라이언트에서 작성할 코드의 예시이다. 요청시 PG사 코드표, 결제방식, 결제 고유번호, 제품에 대한 정보, 유저에 대한 정보를 기입 후 포트원측에서 지원하는 SDK를 이용해 정보를 송신하면 결제창이 출력 된다. 
실제로 결제를 진행하더라도 당일 23시 30분쯤에 거래취소가 이뤄지니 부담없이 테스트를 진행해도 된다.
신용카드의 경우 자동 취소가 이뤄지지만 체크카드는 포트원에 문의를 해야한다. 신용카드로 테스트를 진행하자

![](https://velog.velcdn.com/images/kisuk623/post/754b8f03-61e8-45e8-94a9-85a3bdfc6eb3/image.png)



### PG사 코드표, 결제 방식, 결제 고유 번호

#### PG사 코드표 링크
[PG사 코드표](https://api.iamport.kr/)

#### 결제 방식 코드

| 결제 방식        | 코드  |
| --------------- | ----- |
| 카드            | card  |
| 실시간 계좌이체 | trans |
| 가상계좌        | vbank |
| 소액결제        | phone |
| 삼성페이        | samsung      |

### 결제 테스트, 결제 내역 확인

결제 테스트를 마쳤다면 결제 내역이 추가되었을 것이다. 결제내역을 통해 결제단계, 결제 수단과 같은것을 알 수 있다.

![](https://velog.velcdn.com/images/kisuk623/post/a4a03661-f42b-47a6-9232-b0bc649efe7b/image.png)


### 결제 검증

결제가 정상적으로 완료되었다면 해당 결제건이 내가 요구한 결제정보와 일치하는가 확인을 해야한다.
아임포트는 웹훅(WebHook)을 통해 결제, 가상계좌 발급, 가상계좌 입금등의 서비스가 이뤄질때 마다 엔드포인트로 결제고유 id와 요청한 결제id를 수신할수 있게 해준다. 해당 결제id를 이용해 아임포트API를 호출하면 결제에 대한 상세정보를 알 수 있다.
아임포트 측에서 수신한 정보와 DB에 존재하는 데이터를 비교해 유효한 데이터일 경우 결제완료처리를 하면 아임포트를 이용한 결제가 끝나게 된다.

#### 결제 검증 코드(Node.js)

```js
app.use(bodyParser.json());
    app.post("/payments/complete", async (req, res) => {
      try {
        // req의 body에서 imp_uid, merchant_uid 추출
        const { imp_uid, merchant_uid } = req.body; 
        ...
        // 액세스 토큰(access token) 발급 받기
        const getToken = await axios({
          url: "https://api.iamport.kr/users/getToken",
          method: "post", // POST method
          headers: { "Content-Type": "application/json" }, 
          data: {
            imp_key: "imp_apikey", // REST API 키
            imp_secret: "ekKoeW8RyKuT0zgaZsUtXXTLQ4AhPFW3ZGseDA6bkA5lamv9OqDMnxyeB9wqOsuO9W3Mx9YSJ4dTqJ3f" // REST API Secret
          }
        });
        const { access_token } = getToken.data; // 인증 토큰
        ...
        // imp_uid로 포트원 서버에서 결제 정보 조회
        const getPaymentData = await axios({
          // imp_uid 전달
          url: `https://api.iamport.kr/payments/${imp_uid}`, 
          // GET method
          method: "get", 
          // 인증 토큰 Authorization header에 추가
          headers: { "Authorization": access_token } 
        });
        const paymentData = getPaymentData.data; // 조회한 결제 정보
				/*
				비즈니스 로직
				*/
        ...
      } catch (e) {
        res.status(400).send(e);
      }
    });
```

결제 정보 검증에는 Access Token이 필요한데, 해당 토큰을 발급받으려면 회원가입후 발급받은 API KEY를 이용해 포트원 API를 호출해야 한다. 해당 토큰의 유효기간은 30분이며, 30분안에 재 요청시 같은 토큰을 반환해주고 유효기간은 기존 토큰의 유효기간과 같다. 토큰 재발급시 기존 토큰의 유효기간이 1분 이하일 경우 토큰의 유효기간을 5분 연장시킨뒤 재발급 해준다.


#### 포트원 제공 모듈 사용

```js
app.use(bodyParser.json());
    app.post("/payments/complete", async (req, res) => {
      try {
        // req의 body에서 imp_uid, merchant_uid 추출
        const { imp_uid, merchant_uid } = req.body; 
        ...
        // API KEY 세팅하기
        const iamport = new Iamport({
					apiKey: "1520702154347286",
		      apiSecret: "oPt7RwbMUBXANPrILHBto235HxRMy3RUHujGWBCqtoVKkID5i0FFPCcVfT0GzpIDbCwXKVSkzxhKeB2I",
		    });
        // imp_uid로 결제정보 할당
        const setPaymentsUid = Payments.getByImpUid(req.body);
				// 할당된 값으로 포트원에 결제 정보 요청
		    const getPayment = await setPaymentsUid.request(iamport); // 결제 정보
				/*
				비즈니스 로직
				*/
        ...
      } catch (e) {
        res.status(400).send(e);
      }
    });
```

포트원 측에선 언어별 REST API모듈을 지원하며 해당 모듈을 사용해 손쉽게 포트원 API 를 호출할 수 있다.
코드가 상당히 줄어든 모습을 볼 수 있다.

```bash
$ npm install iamport-rest-client-nodejs
```

[원포트 Nodejs Github](https://github.com/iamport/rest-client-nodejs)

## 토스페이먼츠 PG

>포트원을 통한 API 결제흐름을 살펴보았으니 PG사와 직접 계약했을때의 결제 FLOW의 차이를 알아보도록 하자

### 토스페이먼츠 결제 FLOW

![](https://static.tosspayments.com/docs/window-flow.png)

토스페이먼츠의 경우 결제창을 출력하는 방법이 SDK를 이용하는것, API를 이용하는것이 2가지가 있으며, 방법이 하나 더 있긴 하지만, 간편결제창을 띄우는 방법이라 출력되는 화면이 다르다는 차이가 있다.

#### SDK이용

토스페이먼츠에서 제공하는 SDK를 이용헀을 경우 결제창의 호출 과정이 포트원을 사용했을 때와 동일하다. 

```js
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>결제 위젯</title>
    <script src="https://js.tosspayments.com/v1/payment-widget"></script>
    <style>
      #payment-button {
        width: 100%;
        padding: 15px;
        background-color: #3065ac;
        color: white;
        border-radius: 3px;
        font-size: 16px;
        border: none;
        margin-top: 10px;
      }
      .title {
        margin: 0 0 4px;
        font-size: 24px;
        font-weight: 600;
        color: #4e5968;
      }
    </style>
  </head>
  <body>
    <!-- 상품 정보 영역-->
    <div class="title">상품 정보</div>
    <p>토스 티셔츠</p>
    <p>결제 금액: 10,000원</p>
    <form id="discount-coupon"><input type="checkbox" id="coupon" />5,000원 할인받기</form>
    <hr />

    <!-- 결제 방법 영역-->
    <div class="title">결제 방법</div>
    <div id="payment-method"></div>
    <div id="agreement"></div>
    <button id="payment-button">결제하기</button>
  </body>

  <script>
    const clientKey = "test_ck_D5GePWvyJnrK0W0k6q8gLzN97Eoq"; // 상점을 특정하는 키
    const customerKey = "YbX2HuSlsC9uVJW6NMRMj"; // 결제 고객을 특정하는 키
    const amount = 15_000; // 결제 금액
    const couponAmount = 5_000; // 할인 쿠폰 금액

    /*결제위젯 영역 렌더링*/
    const paymentWidget = PaymentWidget(clientKey, customerKey); // 회원 결제
    // const paymentWidget = PaymentWidget(clientKey, PaymentWidget.ANONYMOUS) // 비회원 결제
    paymentMethods = paymentWidget.renderPaymentMethods("#payment-method", amount);

    /*약관 영역 렌더링*/
    const paymentAgreement = paymentWidget.renderAgreement("#agreement");

    /*결제창 열기*/
    document.querySelector("#payment-button").addEventListener("click", () => {
      paymentWidget
        .requestPayment({
          orderId: "AD8aZDpbzXs4EQa-UkIX6",
          orderName: "토스 티셔츠",
          successUrl: "http://localhost:8080/success",
          failUrl: "http://localhost:8080/fail",
          customerEmail: "customer123@gmail.com",
          customerName: "김토스",
        })
        .catch(function (error) {
          if (error.code === "USER_CANCEL") {
            // 결제 고객이 결제창을 닫았을 때 에러 처리
          }
          if (error.code === "INVALID_CARD_COMPANY") {
            // 유효하지 않은 카드 코드에 대한 에러 처리
          }
        });
    });

    /*할인 쿠폰 적용*/
    document.querySelector("#coupon").addEventListener("click", applyDiscount);

    function applyDiscount(e) {
      if (e.target.checked) {
        paymentMethods.updateAmount(amount - couponAmount, "쿠폰");
      } else {
        paymentMethods.updateAmount(amount);
      }
    }
  </script>
</html>
```

![](https://velog.velcdn.com/images/kisuk623/post/0cf053d9-b691-43a8-b961-7949989c110e/image.png)


#### API 호출

```bash
curl --request POST \
  --url https://api.tosspayments.com/v1/payments \
  --header 'Authorization: Basic dGVzdF9za196WExrS0V5cE5BcldtbzUwblgzbG1lYXhZRzVSOg==' \
  --header 'Content-Type: application/json' \
  --data '{"method":"카드","amount":15000,"orderId":"a4CWyWY5m89PNh7xJwhk1","orderName":"토스 티셔츠 외 2건","successUrl":"http://localhost:8080/success","failUrl":"http://localhost:8080/fail"}'
```

해당 코드를 터미널에서 호출해보면 응답값에 결제를 진행할 수 있는 url를 수신할 수 있다. 해당 url을 들어가면 요청했던 결제 정보에 대한 결제창이 호출 된다.

``` json
  {
    "mId": "tosspayments",
    "version": "2022-11-16",
    "checkout": {
        "url": "https://api.tosspayments.com/v1/payments/0jPR7DvYpNk6bJXmgo28e1QkmPjlE8LAnGKWx4qMl91aEwB5/checkout"
    }
		//...
}
```

SDK를 이용한 방법은 프론트에서 결제신청의 과정을 모두 진행해야 했지만, API호출을 백엔드 서버를 통해서도 진행이 가능하다.
백엔드 서버를 통해 결제창을 호출할 경우 클라이언트에서 백엔드 서버로 결제 요청을 보내면 서버는 클라이언트가 준 정보 혹은 데이터베이스에 저장된 정보를 통해 토스페이먼츠에 결제URL을 요청하고 해당 URL을 클라이언트로 전송해 결제를 진행할 수 있다.
클라이언트에서 결제요청을 할것인지, 백엔드 서버에서 결제요청을 할것인지는 개발자의 몫이다.

[토스페이먼츠 블로그](https://velog.io/@tosspayments/%EA%B2%B0%EC%A0%9C%EC%B0%BD%EC%9D%84-%EC%97%AC%EB%8A%94-%EB%AA%A8%EB%93%A0-%EB%B0%A9%EB%B2%95#%EB%B0%A9%EB%B2%95-3-%EC%B9%B4%EB%93%9C%EC%82%AC-%EA%B0%84%ED%8E%B8%EA%B2%B0%EC%A0%9C%EC%82%AC-%EA%B2%B0%EC%A0%9C%EC%B0%BD-%EB%B0%94%EB%A1%9C-%EC%97%B4%EA%B8%B0)

### 결제 승인

토스페이먼츠의 경우 결제가 끝난뒤 결제 승인이라는 절차가 하나더 남아있다. 결제가 종료되면 결제의 주문id와 결제금액을 받게 되는데 요청한 결제 금액과 실제 결제된 금액이 동일한지 확인하는 과정이다.
결제 금액이 동일할 경우 결제 승인 API를 호출하게 될 경우 결제에 대한 상세 정보를 반환받게 되고 결제 FLOW가 종료된다.

```js
var axios = require("axios").default;

var options = {
  method: 'POST',
  url: 'https://api.tosspayments.com/v1/payments/confirm',
  headers: {
    Authorization: 'Basic dGVzdF9za196WExrS0V5cE5BcldtbzUwblgzbG1lYXhZRzVSOg==',
    'Content-Type': 'application/json'
  },
  data: {
    paymentKey: 'bjSP--2imeaVqdoTSyR1Z',
    amount: 15000,
    orderId: 'ys2_Amx_igMeEkuduuYel'
  }
};

axios.request(options).then(function (response) {
  console.log(response.data);
}).catch(function (error) {
  console.error(error);
});
```

| 키값       | 설명               |
| ---------- | ------------------ |
| paymentKey | 결제의 키 값       |
| orderId    | 주문할때 송신한 id |
| amount     | 결제 금액                   |

## 마치며

오늘은 가장민감하게 다뤄야할 결제에 대해 알아 보았다. 서비스를 제공하면서 제일 중요하게 여겨야할것은 개인정보와 돈에 관련된것이라고 생각한다.
이번 결제 FLOW를 살펴보면서 왜 PG사가 생길수 밖에 없었는지에 대해 알아 보았고, 앞으로 결제시스템을 어떤식으로 도입해야할지 알수 있었다.
