```java
import java.util.HashMap;
import java.util.Random;

// 레거시 외부 결제 시스템 (변경 불가능한 라이브러리)
class OldPayPalProcessor {
    public void initiatePayment(double dollars, String merchantCode) {
        System.out.println("Processing $" + dollars + " via PayPal to " + merchantCode);
    }

    public boolean checkStatus(long transactionId) {
        // 실제로는 REST API 호출
        return new Random().nextBoolean();
    }
}

class NewStripeGateway implements Pattern.PaymentAdepter{

    @Override
    public PaymentResult executePayment(PaymentRequest request) {
        return chargeInKRW(request.amountKRW(), resolveMerchantAccount(request.merchant()));
    }

    public PaymentResult chargeInKRW(int wonAmount, MerchantAccount account) {
        System.out.println("Charging ₩" + wonAmount + " to " + account.getAccountId());
        return new PaymentResult(new Random().nextLong(), true);
    }

    private MerchantAccount resolveMerchantAccount(Merchant merchant) {
        return new MerchantAccount(merchant.id(), "AccountFor_" + merchant.id());
    }
}

// 서비스 내부 도메인 모델
record PaymentRequest(int amountKRW, String currencyType, Merchant merchant) {} // 결제 요청
record PaymentResult(long transactionId, boolean isSuccess) {} // 결제 결과 값
record Merchant(String id, String name, String paymentType) {} // 가맹점정보 및 결제 타입을 받음


class OldPayPalAdepter implements Pattern.PaymentAdepter {

    OldPayPalProcessor processor ;

    public OldPayPalAdepter(OldPayPalProcessor processor) {
        this.processor=processor;
    }

    @Override
    public PaymentResult executePayment(PaymentRequest request) {
        if ("USD".equals(request.currencyType())) {
            double usdAmount = convertToUSD(request.amountKRW());
            processor.initiatePayment(usdAmount, "OUR_MERCHANT_CODE");
            return new PaymentResult(System.currentTimeMillis(), true);
        } else {
            return new PaymentResult(System.currentTimeMillis(), false);
        }
    }
    private double convertToUSD(int won) {
        return won / 1200.0;
    }
}


class PaymentHandler  {

    HashMap<String, Pattern.PaymentAdepter> payList = new HashMap<>();

    public PaymentHandler() {
        // 레거시 감싼 객체 생성
        payList.put("PAYPAL", new OldPayPalAdepter(new OldPayPalProcessor()));
        payList.put("STRIPE", new NewStripeGateway());
        // 카카오도 추가된다면 어댑터 생성
//        payList.put("KAKAOPAY", new KakaoPayAdapter());
    }

    public PaymentResult processPayment(PaymentRequest request) {

        Pattern.PaymentAdepter processor = payList.get(request.merchant().paymentType());

        if (processor == null) {
            throw new UnsupportedOperationException("지원하지 않는 결제 방식");
        }
        return processor.executePayment(request);
    }

}

public class Pattern {
    public static void main(String[] args) {
        // 결제 핸들러 생성
        PaymentHandler handler = new PaymentHandler();

        // PayPal 결제 요청 생성 (KRW -> USD 변환)
        PaymentRequest paypalRequest = new PaymentRequest(
                5000, "USD", new Merchant("1", "이마트", "PAYPAL")
        );

        // Stripe 결제 요청 생성 (KRW -> KRW)
        PaymentRequest stripeRequest = new PaymentRequest(
                20000, "KRW", new Merchant("2", "올리브영", "STRIPE")
        );

        // KakaoPay 결제 요청 생성 (새로 추가된 결제 방식)
        PaymentRequest kakaoPayRequest = new PaymentRequest(
                3300, "KRW", new Merchant("3", "다이소", "KAKAO_PAY")
        );

        // 결제 처리
        PaymentResult paypalResult = handler.processPayment(paypalRequest);
        System.out.println("PayPal 결제 결과: " + paypalResult);

        PaymentResult stripeResult = handler.processPayment(stripeRequest);
        System.out.println("Stripe 결제 결과: " + stripeResult);

//        PaymentResult kakaoPayResult = handler.processPayment(kakaoPayRequest);
//        System.out.println("KakaoPay 결제 결과: " + kakaoPayResult);
    }

    public interface PaymentAdepter {

        public PaymentResult executePayment(PaymentRequest request);

    }


}

```


### 사용 패턴 : 어댑터 패턴 
### 사용 이유 : 결제 시스템 추가 시 확장성지 좋고, 기존 클래스를 바꾸지 않고 어댑터로 감싸 그것을 클라이언트는 모르고 있어 기존 코드와으 결합도가 낮음









