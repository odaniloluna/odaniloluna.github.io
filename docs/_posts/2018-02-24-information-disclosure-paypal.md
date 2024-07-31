---
id: 170
title: Disclosure of account balance and recent transactions on PayPal
date: 2018-02-24T01:00:17+05:30
author: Karan Saini
excerpt: This article details an issue which allows for enumeration of the last four digits of payment method (such as a credit or debit card) and for the disclosure of account balance and recent transactions of any given PayPal account.
layout: post
guid: https://www.karansaini.com/?p=170
permalink: /information-disclosure-paypal/
image: /media/feb_24_2018_paypal.jpg
categories:
  - Privacy
tags:
  - Open Source Intelligence
  - OSINT
  - PayPal
---
This article details an issue which allows for enumeration of the last four digits of payment method (such as a credit or debit card) and for the disclosure of account balance and recent transactions of any given PayPal account.

This attack was submitted to PayPal&#8217;s <a href="https://www.paypal.com/bugbounty/" target="_blank" rel="noopener noreferrer">bug bounty program</a> where it was classified as being out of scope, which is something that would admittedly be unavailing to refute, since their program scope does not mention anything about attacks on their interactive voice response system.

The issue still exists as of February 24, 2018.

### Prerequisites and Reconnaissance

In order to get started, the attacker would require knowledge of two pieces of information pertaining to an account, which would be the e-mail address and phone number linked to it.

Armed with knowledge of the e-mail address and phone number linked to an account, the attacker would visit the <a href="https://www.paypal.com/authflow/password-recovery/" target="_blank" rel="noopener noreferrer"><em>Forgot Password</em></a> page on PayPal&#8217;s website, and enter the e-mail address associated with the targeted account.

The attacker would then be presented with the type of card linked to the account, as well as the last two digits of the same.

![](/media/feb_24_2018_1.png){: width="250" }

### Attacking the Interactive Voice Response System

**Note:** You can listen to the audio of the IVR interaction here <a href="https://soundcloud.com/sainikaran/sets/disclosure-of-account-balance-and-recent-transactions-on-paypal">here</a>

On first glance, the interactive voice response system on PayPal&#8217;s phone-based customer support seemingly allows for a maximum of three attempts at submitting the correct last four digits per phone call.

However, if the first attempt at submission is incorrect, the caller will not be notified of a successful submission in subsequent attempts made during the same phone call. This makes any additional attempts given to a caller during the same phone call completely cosmetic.

To get around this presumed limitation, the attacker would have to make only one attempt at submitting a possible combination of the last four digits per phone call.

![](/media/feb_24_2018_2.png){: width="250" }

Additionally, limiting the number of attempts to one submission per phone call makes the task of enumerating the correct combination much more time-efficient, and not to mention, it allows for easily distinguishing between a correct attempt and an incorrect one.

Furthermore, upon have tested this theory with my own account, I have been able to conclude that there is no limit on the number of submission attempts which can be made in this manner, meaning that hypothetically, an attacker could even call 10,000 times to enumerate the last four digits entirely on their own.

That would, however, be disregarding the last two digits retrieved from the _Forgot Password_ page, the knowledge of which effectively makes the attack much more feasible–by reducing the number of possible combinations from 10,000 to just 100.<figure></figure> 

Once the correct combination of the last four digits has been found, the attacker would simply have to use the interactive voice response system to retrieve information about the account.

After having entered the correct last four digits, the account&#8217;s current balance will automatically read off by the machine.<figure></figure> 

Additionally, to retrieve information about recent transactions, an attacker would simply have to say &#8220;recent transactions&#8221;, and the same would then be read off.<figure></figure> 

### Attack Efficacy and Efficiency

If the aforementioned prerequisites have been met, an attacker would without fail have the ability to enumerate the correct last four digits of the payment method linked to an account. This information could then further be used to retrieve the account&#8217;s current balance and recent transactions as well.

Moreover, after having timed various attempts at submission of the last four digits, it was found that an attempt at submission would on average take around 30 seconds. The fastest possible time was found to be exactly 27 seconds per phone call.

If we take the fastest possible time as our average, enumerating all possible combinations from 00XX to 99XX would take at most around 45 minutes. This time could then be halved by adding another phone in the mix to consecutively make calls with.

### Possible Fixes

Users should be allowed to opt for privacy settings which keep the amount of data revealed on the _Forgot Password_ page to a minimum. This would be similar to how Twitter allows its users to hide information about the email address and/or phone number linked to their account when attempting to reset its password.

It would also be similar to how Facebook allows users to choose whether their full names show up or not when their e-mail address is entered on the password reset page.

Perhaps some measures could be deployed where the last two digits of credit or debit card, if they need to be shown at all, are only shown when the request matches a certain criteria, such as if/when the request has been made from a recognizable device or location.

### Conclusion

This issue allows for enumeration of the last four digits of the payment method on an account, which then allows for the disclosure of the account&#8217;s current balance and recent transactions.

An attacker with knowledge of the targeted account&#8217;s email address and phone number would first use PayPal&#8217;s _Forgot Password_ page to retrieve the last two digits of the payment method linked to the account.

The attacker would then be able to accurately enumerate the last four&#8211;or rather the first two of the last four digits&#8211;of the payment method on the account by making phone calls to PayPal&#8217;s phone-based customer support and interacting with the interactive voice response system.

Once the attacker has successfully enumerated the last four digits of credit/debit card or bank account linked to the account, they would then be able to query the current account balance and recent transaction information at will.

I would like to mention that, since there is no human interaction required or involved in this attack, it is essentially a backdoor into PayPal accounts–allowing attackers to query current account balance and recent transaction information of any given account, at any time.
