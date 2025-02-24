# Intro to Attacking Enterprise Networks

---

You've done it! Congratulations, you've reached the end of the `Penetration Tester` Job Role Path. This is no easy feat, and we know it has been a long journey full of many challenges, but hopefully, you have learned loads (or picked up new skills) along the way. As mentioned in the `Penetration Testing Process` module, the path was split into 27 modules to present a penetration test against the fictional organization, `Inlanefreight`, piece by piece, phase by phase, tool by tool. We structured the path based on the most important stages of the penetration testing process. We broke out individual modules based on the Tactics, Techniques, and Procedures (TTPs) that we feel are most important to have a firm grasp over to perform penetration testing engagements at an intermediate level while managing or assisting with the entire engagement lifecycle. Through the various modules, we take an in-depth tour through the following stages:

- Pre-Engagement
- Information Gathering
- Vulnerability Assessment
- Exploitation
- Post-Exploitation
- Lateral Movement
- Proof of Concept
- Post-Engagement

We aimed to keep the path as hands-on as possible while teaching the necessary prerequisite skills that we feel are necessary for an individual to succeed in a consulting environment. Yes, many things can only be learned on the job, but we've done our best to equip you with the mindset and technical know-how to progress in your current role, start a new role, or move from one technical discipline to another. We, of course, cannot guarantee that anyone who completes this path will land their dream job immediately. But, if you've put in the time, worked hard to understand the technical concepts in-depth, can complete all modules skills assessments on your own with a mix of automated and manual approaches, and focused heavily on honing your documentation and reporting skills, you will make sure yourself highly marketable.

Believe it or not, at this point (if you have finished every other module in the path), you have completed `seven mini simulated penetration tests`, each focusing on a particular area:

- All of the elements of a large pentest cut up into the 27 preceding modules
- A cross-section of an Active Directory pentest (broken down step-by-step) in the Active Directory Enumeration & Attacks module sections
- 2 mini/simulated Active Directory pentests in the Active Directory Enumeration & Attacks skills assessments
- 1 mini/simulated pentest for the Shells & Payloads module skills assessment
- 1 mini/simulated pentest for the Pivoting, Tunneling, & Port Forwarding module skills assessment
- 1 mini/simulated internal pentest in the Documentation & Reporting module (a mix of exploratory and guided learning)

Through all module sections, we have attacked over 200 targets (a mix of Windows, Linux, Web, and Active Directory targets).

Until now, we have not seen all of the topics we teach in this path together in a single network (though some combine a few parts, i.e., a web attack to gain a foothold into an AD network). In each module's skills assessment, we had a general idea of the topics and tactics that would be covered. In this lab, we will have to call on `ALL` of the knowledge we have gained thus far and be able to switch from info gathering to web attacks to network attacks back to info gathering to Active Directory attacks, to privilege escalation, to pillaging and lateral movement, call on our pivoting skills and more. We need to be comfortable whether our target is a web application, a standalone Windows or Linux host, or an Active Directory network. This can seem overwhelming at first, but successful penetration testers must be able to constantly cycle through their Rolodex of skills and quickly adapt on the fly. We never know what we'll face before an engagement starts, and every network is unique but similar in how we should approach things. Every pentester has their own methodology and way of doing things but will always come back to the core stages of the penetration testing process.

This module's purpose is to allow you to practice everything learned so far against a simulated corporate network. This module will take us step-by-step through an External Penetration Test against the Inlanefreight company, leading to internal access and, at that point, turning into a full-scope Internal Penetration Test to find all possible weaknesses. The module will take you through each step from the perspective of a penetration tester, including chasing down "dead ends" and explaining the thought process and each step along the way. This can be considered a simulated "ride-along" pentest with a Penetration Tester working alongside a more experienced tester. The module sections will take you through the target network in a guided fashion, but you must still complete the entire lab yourself and retrieve and submit each flag. To get the most out of this module, we recommend tackling the lab a second time without the walkthrough as the pentester in the driver's seat, taking detailed notes (documenting as we learned in the `Documentation and Reporting` module), and creating your own walkthrough and even practice creating a commercial-grade report.

Next, we will cover the scope of the engagement and then dig in and get our hands dirty!