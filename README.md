# 🍏 Apple Pay Integration on iOS

Apple Pay is a **digital wallet** that allows users to pay with cards stored in Wallet.  
It is **not a payment gateway**; it only generates a **payment token**, which must be sent to a backend/payment gateway (like Stripe or Paymob) to process the payment.

---

## 1️⃣ What is Apple Pay & Difference from Payment Gateway

Apple Pay is a **secure way to authorize payments** with a user's card.  
Payment Gateway is the **service that actually processes the money**.

| Feature | Apple Pay | Payment Gateway |
|---------|-----------|----------------|
| Generates token | ✅ | ❌ |
| Processes money | ❌ | ✅ |
| UI / authorization | ✅ | ❌ |
| Sends token to backend | ✅ | ✅ |

> Without a payment gateway, Apple Pay token alone does nothing.

---

## 2️⃣ Requirements

- iOS 15+  
- Xcode 15+  
- Paid Apple Developer Account  
- Merchant ID created on Apple Developer Portal  
- Real device for testing (Simulator does not support Apple Pay)  

---

## 3️⃣ Setup Steps in Xcode

1. Enable Apple Pay in your Apple Developer Account  
2. Create a **Merchant ID**  
3. Add **Apple Pay Capability** to your Xcode target  
4. Set up `PKPaymentRequest` (products, amounts, currency, country, networks)  
5. Implement `PKPaymentAuthorizationViewControllerDelegate` to handle results  

---

## 4️⃣ Summary

Apple Pay is **only a digital wallet**, it **does not process money**.  
It generates a **token** which **must** be sent to a backend/payment gateway to complete the payment.  

- Apple Pay = **Token Generator + UI Authorization**  
- Payment Gateway = **Actual Payment Processing**  

Without a gateway, you just get a token and nothing happens.

---

## 5️⃣ Swift Sample File

```swift
//
//  ApplePayIntegration.swift
//  Example of Apple Pay Integration
//
//  Created by Mohamed Abd ElHakam.
//

import UIKit
import PassKit

class ApplePayManager: NSObject {

    // MARK: - Check if device can use Apple Pay
    static func canUseApplePay() -> Bool {
        return PKPaymentAuthorizationViewController.canMakePayments()
    }

    // Check if device can use specific networks
    static func canUseApplePayWithNetworks() -> Bool {
        return PKPaymentAuthorizationViewController.canMakePayments(usingNetworks: [.visa, .masterCard, .amex, .mada])
    }

    // MARK: - Start Apple Pay Payment
    func startApplePay(from viewController: UIViewController, cartItems: [CartItem], tax: Double = 0, shipping: Double = 0) {
        guard ApplePayManager.canUseApplePay() else {
            print("Device does not support Apple Pay")
            return
        }

        let request = PKPaymentRequest()
        request.merchantIdentifier = "merchant.your.company.id"
        request.countryCode = "US"
        request.currencyCode = "USD"
        request.supportedNetworks = [.visa, .masterCard, .amex, .mada]
        request.merchantCapabilities = .capability3DS

        // Build dynamic payment summary items from cart
        var summaryItems: [PKPaymentSummaryItem] = cartItems.map {
            PKPaymentSummaryItem(label: $0.name, amount: NSDecimalNumber(value: $0.price))
        }

        if tax > 0 {
            summaryItems.append(PKPaymentSummaryItem(label: "Tax", amount: NSDecimalNumber(value: tax)))
        }

        if shipping > 0 {
            summaryItems.append(PKPaymentSummaryItem(label: "Shipping", amount: NSDecimalNumber(value: shipping)))
        }

        let totalAmount = cartItems.reduce(0) { $0 + $1.price } + tax + shipping
        summaryItems.append(PKPaymentSummaryItem(label: "Total", amount: NSDecimalNumber(value: totalAmount)))

        request.paymentSummaryItems = summaryItems

        if let paymentVC = PKPaymentAuthorizationViewController(paymentRequest: request) {
            paymentVC.delegate = self
            viewController.present(paymentVC, animated: true)
        } else {
            print("Unable to present Apple Pay authorization.")
        }
    }
}

// MARK: - PKPaymentAuthorizationViewControllerDelegate
extension ApplePayManager: PKPaymentAuthorizationViewControllerDelegate {

    func paymentAuthorizationViewControllerDidFinish(_ controller: PKPaymentAuthorizationViewController) {
        controller.dismiss(animated: true)
    }

    func paymentAuthorizationViewController(
        _ controller: PKPaymentAuthorizationViewController,
        didAuthorizePayment payment: PKPayment,
        handler completion: @escaping (PKPaymentAuthorizationResult) -> Void
    ) {
        let token = payment.token
        print("Payment Token:", token)

        sendTokenToServer(token) { success in
            let status: PKPaymentAuthorizationStatus = success ? .success : .failure
            completion(PKPaymentAuthorizationResult(status: status, errors: nil))
        }
    }

    private func sendTokenToServer(_ token: PKPaymentToken, completion: @escaping (Bool) -> Void) {
        // Send token.paymentData to backend / Stripe / Paymob
        DispatchQueue.main.asyncAfter(deadline: .now() + 2.0) {
            completion(true) // change to false to simulate failure
        }
    }
}

// MARK: - Example CartItem model
struct CartItem {
    let name: String
    let price: Double
}
```

<br><br>

## Important Note
- Apple Pay does not decide if the payment was successful

  - Authorizes the payment on the device
  - Generates a payment token
  - Sends the token to your backend or payment gateway

<br>

- The payment gateway (Stripe, Paymob, etc.) is responsible for:
  - Processing the payment
  - Confirming success or failure
  - Sending the response back to your app

<br>

Without a payment gateway, generating the token does not complete the transaction.

<br><br><br>
