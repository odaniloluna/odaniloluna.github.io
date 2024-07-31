---
id: 650
title: Attacking a weak 3D Secure implementation
date: 2019-08-25T03:50:21+05:30
author: Karan Saini
excerpt: This article documents a security flaw in the mechanism through which the 3D Secure implementation of Wibmo andles the generation and processing of the one-time PIN used for performing cardholder verification.
layout: post
guid: https://www.karansaini.com/?p=650
permalink: /3d-secure-flawed-otp-handling/
image: /wp-content/uploads/TT.png
categories:
  - Security
tags:
  - 3D Secure
  - Cardholder Verification
  - OTP Relay
---
# Attacking a weak 3D Secure implementation
This article documents a security flaw in the mechanism through which the 3D Secure implementation of Wibmo — which is the primary integrator of the 3D Secure protocol for banks in India — handles the generation and processing of the one-time PIN used for performing cardholder verification. The mechanism used by the Wibmo 3D Secure Access Control Server for generating the one-time PIN improperly handles transactions that have been initiated within a 180 second time-frame of each other, resulting in the generation and delivery of a singular, non-unique one-time PIN for both of the transactions. The flaw allows an attacker to re-appropriate a one-time PIN which has been generated for a low-value transaction (e.g., 10 INR) for successfully completing another transaction of an arbitrary amount (e.g., 1000 INR). 

## 3D Secure 1.0

The 3D Secure (&#8220;_3_&#8211;_domain structure&#8221;_) protocol was originally developed by Arcot Systems for adding an additional layer of authentication for card-based transactions on the Internet. 3D Secure 1.0 (3DS1) has been the de-facto standard for online cardholder verification and has been adopted by several major card schemes, such as Visa through _[Verified by Visa](https://usa.visa.com/pay-with-visa/featured-technologies/verified-by-visa.html)_ and Mastercard through _[SecureCode](https://www.mastercard.us/en-us/consumers/payment-technologies/securecode.html)_.

The first iteration of the protocol has been criticized over time by various parties — including online merchants who have argued that implementation has an affect on conversion rates — as well as academic researchers, who have stated that the protocol may not be as effective in preventing card fraud as it is purported to be. 3DS1 has also been repeatedly criticized for non-uniformity, and implementations of the protocol have been [noted](https://www.adyen.com/blog/3d-secure-20-a-new-authentication-solution) as being &#8220;inconsistent from bank to bank and country to country.&#8221;

The Reserve Bank of India has long mandated that banks within the country implement [_&#8220;additional factor authentication&#8221;_](https://m.rbi.org.in/CommonPerson/english/Scripts/FAQs.aspx?Id=1498) for all transactions where the physical presence of a payment instrument cannot be verified, e.g., for transactions which do not involve card interaction with a point-of sale-system. Indian banks traditionally perform online cardholder verification through confirmation of an additional authentication step, such as a user-defined PIN, passphrase, or a dynamic one-time PIN (OTP). While mandatory cardholder verification is generally perceived as a step in the right direction, it should noted that the Reserve Bank of India has not issued guidelines or best practices on how banks should implement secure additional factor authentication for their customers.

## Wibmo 3DS 1.0

Wibmo — previously known as enStage — is a financial software and services provider based in Bangalore, India. The company acts as a 3DS1 integrator for a large number of significant financial institutions in India, including banks like Punjab National Bank, Kotak Mahindra Bank, Corporation Bank, and others.

The mechanism used by the Wibmo 3DS1 Access Control Server for generating the one-time PIN improperly handles transactions that have been initiated within a 180-second time-frame of each other, resulting in the generation and delivery of a singular, non-unique one-time PIN for both of the transactions.

The flaw can be observed by initiating a transaction for a low arbitrary amount (e.g., 1 INR), then consequently — within 180 seconds — initiating a parallel for a higher arbitrary amount (e.g., 1000 INR). The one-time PIN mechanism implemented on the 3DS Access Control Server will generate and deliver the same OTP to the customer in both cases. The delivered OTP can then be used for successfully authenticating either of the two transactions.

![INR 366](/media/366.png){: width="250" }

The two transactions need not be limited to the same merchant, meaning that, the OTP generated for transacting at Store ABC will be identically generated for transacting at Store XYZ — as long as the second transaction was initiated within the 180-second window. Another factor to note is that, the two transactions need not be initiated from the same IP address for the same OTP to be delivered twice. If that, however, were the case, perhaps not generating a fresh one-time PIN for the second transaction could be excused.

![INR 999](/media/999.png){: width="250" }

## Insecure OTP Generation

The non-uniqueness of the one-time PIN can be attributed to the OTP generation mechanism which has been employed by the 3DS Access Control Server — which relies solely on the customer&#8217;s credit or debit card number — in addition to a 180-second timer which is set by the Access Control Server during which requests for generating of a fresh OTP are not entertained. This essentially means that a fresh one-time PIN will _not be generated until_ 180 seconds have elapsed from the generation of a previous OTP — without consideration of parameters such as _TXN\_ID, TXN\_VALUE, IP_ADDR_, etc.

![Insecure OTP generation method](/media/otp_gen_insecure.png){: width="250" }

![Ideal OTP Generation method](/media/otp_gen_secure.png){: width="250" }

It can be assumed that the 180-second timer on OTP generation was put in place for reducing rigidity and friction, like in cases where a customer may have subpar cellular coverage and may repeatedly attempt to _resend OTP_ several times in quick succession. Regardless, even if such a use case were given consideration, the design choice which dictates that an OTP should be generated solely on the basis of the card number — without accounting for dynamic parameters such as _TXN_ID_ — which, if they were employed, would result in the generated OTP being unique across transactions — would perhaps still not be justifiable.

In an email response, Wibmo&#8217;s security team confirmed that OTP generation mechanism does not account for _&#8220;a specific session or transaction_,_&#8220;_ stating that only the payment card number is treated as an input parameter. The security team further went on to state that the weak handling of OTP generation _&#8220;is not a flaw in the system, but a known functionality,&#8221;_ which is certainly not true, as other banks which make use of the 3D Secure protocol, such as HDFC Bank, generate a unique one-time PIN for each separate transaction, regardless of time elapsed from the generation of a previous OTP.

The email from the Wibmo security team further went on to remark that, since the validity of an OTP expires after being used once, that _&#8220;an OTP generated for two parallel transactions can only ultimately still only be used in one.&#8221;_ The following section will demonstrate how the discovered flaw can be used for carrying out a convincing OTP __relay or real-time phishing attack, similar to the type of attack which is [employed for defeating Google&#8217;s 2FA](https://www.zdnet.com/article/new-tool-automates-phishing-attacks-that-bypass-2fa/) — through [automated phishing](https://www.vice.com/en_us/article/evye3a/hackers-publish-list-of-discord-email-addresses-passwords-login-credentials) of the 2FA token.

## Exploitation

An attacker can carry out an OTP relay or real-time phishing attack to automate siphoning of user funds. The first step of such an attack involves the customer submitting an OTP to the attacker&#8217;s domain, which is masquerading as a legitimate 3D Secure authentication page — but is instead actually only a wrapper for acquiring the OTP from the victim. The wrapper will call a payment API for initiating the first transaction with a non-suspicious value — and upon submission of the OTP to the wrapper page — would immediately initiate _and_ successfully complete a transaction for a predefined high-value amount.

A tool such as Piotr Duszynski&#8217;s [_Modlishka_](https://github.com/drk1wi/Modlishka) — or Mike Felch&#8217;s [_CredSniper_](https://github.com/ustayready/CredSniper) — or FireEye&#8217;s [ReelPhish](https://github.com/fireeye/ReelPhish) — can be configured for provisioning an automated phishing setup which targets the Wibmo 3D Secure ACS. A key advantage for an attacker here would be that, in most cases, 3D Secure ACS URLs are not the most memorable, and thus would not be immediately recognizable to a victim — unlike web addresses for popular services such as Google or Yahoo. Alternatively, an unsophisticated actor may quite simply just choose to remain _low-technology_ and perform a tried and trusted social engineering attack.

## Conclusion 

The scope of affected banks is somewhat difficult to deduce in this case. Potentially affected banks can be identified _prima facie_ through the presence of a “Powered by Wibmo” footer on the Access Control Server webpage. At the same time however, banks can choose to modify their implementation of 3D Secure 1.0, which means that particular banks _may_ or _may not_ employ the flawed OTP mechanism. Nevertheless, it can authoritatively be said that this flaw does affect _all payment cards_ issued by Kotak Mahindra Bank and Punjab National Bank, at the very least.

An extended list of banks which have used Wibmo for integration with 3D Secure 1.0 can be found [here](https://pastebin.com/raw/QP5VbtC2).

This flaw was disclosed to Wibmo on August 3, 2019. The security team at Wibmo closed the issue and marked it as known functionality on August 12, 2019. The flaw was publicly disclosed on August 25, 2019. I would like to thank [captn3m0](https://twitter.com/captn3m0) for the assistance he provided in helping ascertain the scope of this issue.
