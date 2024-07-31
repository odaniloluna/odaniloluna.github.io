---
title: Extracting personal phone numbers linked to Aadhaar
date: 2018-05-03T17:53:54+05:30
author: Karan Saini
excerpt: 'This article demonstrates how the personal phone number linked to any given Aadhaar can be extrapolated due to problems in implementation of the text-based authentication mechanism which websites offering Aadhaar authentication rely on. '
layout: post
guid: https://www.karansaini.com/?p=337
permalink: /extracting-aadhaar-linked-phone-numbers/
image: /wp-content/uploads/may_6_2018_b_g.png
categories:
  - Privacy
tags:
  - Aadhaar
  - Phone Numbers
  - OSINT
---
The purpose of this article is to demonstrate how the personal phone number linked to any given Aadhaar can be extrapolated due to problems in implementation of the text-based authentication mechanism which websites offering Aadhaar authentication rely on.

Websites which make use of text-based (OTP) Aadhaar authentication display to the user only the last four digits of the phone number linked to the provided Aadhaar. This is ostensibly a privacy safeguard put in place by the UIDAI to limit personally identifiable information from being revealed prior to authentication.

This privacy safeguard can be circumvented using websites which poorly implement text-based Aadhaar authentication, specifically, through websites which allow the user to input both the Aadhaar number as well as the verified phone number linked to the same.

If the provided phone number doesn&#8217;t match the verified phone number linked with the corresponding Aadhaar; the user is notified and has the ability to try again. This flaw in implementation introduces a surface for computer-aided guessing and enumeration attacks which can be abused to reveal the entire phone number linked to a given Aadhaar.

Several websites were analyzed and tested for this article out of which three were found to be vulnerable to enumeration attacks which could be used to retrieve the complete phone number linked to an Aadhaar.

### Obtaining the last four digits

The [digilocker.gov.in](http://digilocker.gov.in) website reveals the last four digits of linked phone number prior to successful authentication. This means that a user would simply have to input an Aadhaar number when signing up for DigiLocker and the last four digits of the linked phone number would then be displayed.

![](/media/may32018.png){: width="250" }

It should be clarified that DigiLocker is one of many websites which can be used to retrieve the last four digits of the Aadhaar-linked phone number. It should also be clarified that the websites mentioned in this post were selected for no reasons other than their general availability.

Another website which was found to share the characteristic of disclosing the last four digits was [nfsm.gov.in](http://nfsm.gov.in/dbt/aadhaarverification.aspx) &#8211; where after entering an Aadhaar number the last four digits of the phone number linked to it were displayed.

![](/media/may620183423.png){: width="250" }

While most sites were found to have properly implemented text-based Aadhaar authentication, several other websites &#8211; both in the public and private sector &#8211; were found to be susceptible to the implementation flaws and vulnerabilities mentioned earlier.

### Obtaining the first six digits

The Indian mobile telephone numbering system defines that the first digit of a mobile phone number can (currently) only be one of the following: nine (9), eight (8) or seven (7).

This information will prove to be immensely useful during the enumeration process as it allows for eliminating the entire number space for numbers 1-6 when going through combinations of possible phone numbers.

Let us say that the following partial phone number is retrieved from DigiLocker or any other website which performs Aadhaar authentication:

![](/media/may6201802.png){: width="250" }

Here, the digits which are not currently known are represented using **X**, and the digits which are known are represented using **Y**.

Simply for the sake of convenience; the enumeration process should ideally begin from the 9XXX series of phone numbers, as it is the [most populated mobile telephone number series](https://en.wikipedia.org/wiki/Mobile_telephone_numbering_in_India) in India.

![](/media/may620180101.png){: width="250" }

Discovery of the five unknown digits can be automated with the use of a script, or through an intercepting proxy such as the OWASP Zed Attack Proxy &#8211; which features a tool to send automated requests to a web server with a given set of payloads.

(In this context, the term payload refers to the entire five digit number space which contains [all numbers ranging from 00000 to 99999](https://github.com/karsaini/enumerate-list/blob/master/files/5_digits_00000-99999.txt))

![](/media/may62018432.png){: width="250" }

<p class="graf graf--p">
  To reiterate; the script or tool would query any of the vulnerable websites repeatedly to extract the full phone number pertaining to any given Aadhaar.
</p>

The video below demonstrates this process through the use of Burp Suite &#8211; which is another intercepting proxy capable of sending automated requests.<figure class="wp-block-embed">

<div class="wp-block-embed__wrapper">
  https://www.youtube.com/watch?v=obbJEvy6h0M
</div></figure> 

In the video above, the valid phone number combination can be distinguished on the basis of its response length. A response length of 8081 signifies a match, as opposed to a response length of 1389 or 1375 which signifies an authentication failure.

If no valid results are found in the 9XXX series of phone numbers, enumeration could then be carried out on the 8XXX and 7XXX number series respectively &#8211; until a match has been found.

In most cases &#8211; a positive match should be found in the 9XXX series simply due to its size &#8211; but this ultimately depends on several factors such as the state where the phone number in question was issued, cellular carrier (as well as whether the number has been ported to another carrier) and others.

![](/media/may62018014.png){: width="250" }

An attack like the one described in this post would take around 17 minutes for each number series &#8211; given that there are 100 requests each second, or 6000 per minute. It would take 17 minutes in total for each number series to be exhausted with 99,999 possible combinations in each.

This means that exhausting all current number series (9XXX, 8XXX, 7XXX) would take around 51 minutes with the above-mentioned conditions.

NOTE: A text-message containing a one-time-pin for Aadhaar authentication is sent to the linked phone number as soon as a match is found.

### Possible fixes

The masking scheme used in the authentication process could be strengthened by disclosing only the least number of digits needed to verify the correctness of the linked phone number.

An example of such a masking scheme could be something similar to this:

![](/media/may620185395.png){: width="250" }

A masked phone number like the one displayed above would hinder attempts at enumeration and/or guessing while still allowing for verification of the correctness of the linked phone number.

### Conclusion

A new attack surface is introduced when websites which make use of Aadhaar authentication allow users to enter an Aadhaar number as well as the phone number linked to the same. Any website which shares this characteristic could potentially be abused in such a way which allows for accurate deduction of the phone number linked to a given Aadhaar.

The privacy implications of being able to map an Aadhaar number to its corresponding phone number are grave &#8211; as a lot can become known about a person given their phone number, such as their full name through a reverse lookup, their social media profiles through &#8216;Find your friends&#8217; features on social media websites, and more information which one should not be able to glean from a 12 digit UID.

Additionally, knowledge of a phone number and the Aadhaar card number to which it is linked could be used by malicious parties to aid SIM swap attacks, phishing schemes and other illicit activities which require knowledge of the same.
