---
title: "Risk Management Lessons from the SKT USIM Hacking Incident"
layout: single
Typora-root-url: ../
categories: EST371
tag: [management, risk]
use_math: true
---

In April 2025, SK Telecom faced a major security breach involving unauthorized access to USIM data. This article analyzes the timeline, response gaps, and key risk management lessons from the incident.

## Hacking Incident Happens

{% include video id="4qtxISCu2Fg" provider="youtube"%}

At the end of 2024, SK Telecom’s HSS (Home Subscriber Server) was hacked. As a result, customer information was leaked, and USIM cards could be copied. Since USIMs are very important for mobile security and user identification, this hack caused serious problems.

## Incident Details

On April 18 at 6:09 PM, SKT first noticed strange activity in their internal system.
- Around 11:20 PM the same day, they found malware and realized it could be a hacking attack.
- At 1:40 AM on April 19, they started analyzing which data had been leaked.

However, the official hacking report was delayed more than 24 hours.
- This may have broken the law, which requires reporting within 24 hours of detection.

> “It is judged that the 24-hour report rule was violated.” (Source: Yonhap News, April 24, 2025)

## The Importance and Role of the USIM

![]({{site.url}}/images/2025-05-02-data-science-4/usim.png){: .align-center .img-width-half}

A USIM is not just a simple SIM card. It stores the **user’s ID** and **network authentication keys**.
The HSS (Home Subscriber Server) is the core system that keeps *subscriber information*. It saves phone numbers, location, authentication keys, and service plans.

If the HSS server is hacked:
- Hackers can make fake (cloned) USIM cards
- They can steal network access rights
- Personal information can be leaked, leading to more crimes

## Problems in SKT’s Response Process After the Hack

### Delayed Notification

![]({{site.url}}/images/2025-05-02-data-science-4/law.png){: .align-center}

One of the biggest issues was the delayed notification to customers. It took several days for SKT to inform users about the hack. This delay is a clear sign of failure in responding quickly to the situation, which is critical when dealing with security breaches. Customers were left in the dark for too long, which could have caused further concern and anxiety.

### Passive Communication Despite Late Notification

![]({{site.url}}/images/2025-05-02-data-science-4/newroom.png){: .align-center}

Even after the delayed notice, SKT’s communication remained passive. Rather than proactively reaching out to affected users with direct messages or alerts, they chose to post updates only through a notice on their official newsroom website. This method reflects a reactive stance, prioritizing public image management over active customer protection. It seemed that SKT was more focused on cleaning up the aftermath than on minimizing damage or reassuring their users in real time.

### No Proper Preparation for SIM Card Replacements

![]({{site.url}}/images/2025-05-02-data-science-4/stock.webp){: .align-center}

To make matters worse, when SKT finally decided to offer SIM card replacements, they didn’t have enough stock ready. This unpreparedness caused further confusion and frustration. It felt like they were scrambling to come up with a solution without thinking through the logistics and making sure they had the necessary resources in place.

## Risk Management Issues in SKT's Hack Response
SKT’s response to the recent hack revealed several critical issues from a risk management perspective:

### Delayed Response
SKT detected the breach on the 18th but didn’t officially report it within the required time frame. This delayed action highlights a failure in swift response, which is crucial to mitigate risks.

### Lack of Transparency
The company didn’t actively inform customers about the breach, leading to a decline in trust. Transparency is key in risk management, especially when customer data is involved.

### Disrupted Information Flow
There were significant delays in communication between internal teams, from detection to customer notification. This shows a lack of a coordinated security response system within the company.

## Conclusion

![]({{site.url}}/images/2025-05-02-data-science-4/risk.jpeg){: .align-center .img-width-seventy}

This incident was not just a technical breach but a failure in managing the entire process, from security awareness to response. While the hack was detected quickly, the slow internal processes and failure to comply with regulations created even greater risks.

### Improvement Proposal
To prevent similar issues, SKT should streamline internal communication, enforce quicker response times, and ensure compliance with established regulations to better manage security threats.