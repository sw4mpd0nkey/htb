# Scenario & Kickoff

---

Our client, Inlanefreight, has contracted our company, Acme Security, Ltd., to perform a full-scope External Penetration Test to assess their perimeter security. The customer has asked us to identify as many vulnerabilities as possible; therefore, evasive testing is not required. They would like to see what sort of access can be achieved by an anonymous user on the Internet. Per the Rules of Engagement (RoE), if we can breach the DMZ and gain a foothold into the internal network, they would like us to see how far we can take that access, up to and including Active Directory domain compromise. The client has not provided web application, VPN, or Active Directory user credentials. The following domain and network ranges are in scope for testing:

| **External Testing** | **Internal Testing** |
| --- | --- |
| 10.129.x.x ("external" facing target host) | 172.16.8.0/23 |
| \*.inlanefreight.local (all subdomains) | 172.16.9.0/23 |
|  | INLANEFREIGHT.LOCAL (Active Directory domain) |

The customer has provided the primary domain and internal networks but has not given specifics on the exact subdomains within this scope nor the "live" hosts we will encounter within the network. They would like us to perform discovery to see what type of visibility an attacker can gain against their external network (and internal if a foothold is achieved).

Automated testing techniques such as enumeration and vulnerability scanning are permitted, but we must work carefully not to cause any service disruptions. The following are out of scope for this assessment:

- Phishing/Social Engineering against any Inlanefreight employees or customers
- Physical attacks against Inlanefreight facilities
- Destructive actions or Denial of Service (DoS) testing
- Modifications to the environment without written consent from authorized Inlanefreight IT staff

---

## Project Kickoff

At this point, we have a Scope of Work (SoW) signed by both our company management and an authorized member of the Inlanefreight IT department. This SoW document lists the specifics of the testing, our methodology, the timeline, and agreed-upon meetings and deliverables. The client also signed a separate Rules of Engagement (RoE) document, commonly known as an Authorization to Test document. This document is crucial to have in hand before beginning testing and lists out the scope for all assessment types (URLs, individual IP addresses, CIDR network ranges, and credentials, if applicable). This document also lists key personnel from the testing company and Inlanefreight (a minimum of two contacts for each side, including their cell phone number and email address). The document also lists out specifics such as the testing start and stop date, and the allowed testing window.

We have been given one week for testing and two additional days to write our draft report (which we should be working on as we go). The client has authorized us to test 24/7 but asked us to run any heavy vulnerability scans outside regular business hours (after 18:00 London time). We have checked all necessary documents and have the required signatures from both sides, and the scope is filled in entirely, so we are good to go from an administrative perspective.

---

## Start of Testing

It is first thing Monday morning, and we are ready to begin testing. Our testing VM is set up and ready to go, and we've set up a skeleton notetaking and directory structure to take notes using our favorite notetaking tool. While our initial discovery scans run, as always, we will fill in as much of the report template as possible. This is one small efficiency we can gain while waiting for scans to complete to optimize the time we have for testing. We have drafted the following email to signal the start of testing and copied all necessary personnel.

![text](../imgs/163/start_testing.webp)

We click send on the email and kick off our external information gathering.