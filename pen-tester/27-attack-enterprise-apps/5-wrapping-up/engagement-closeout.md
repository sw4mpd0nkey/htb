# Engagement Closeout

---

We've reached the end of the engagement, and there are some tasks we must perform before wrapping everything up. The first thing we should do is send our client an email notifying them that the testing period has ended and a specific time frame they can expect us to deliver the report. At this time, you can also ask them to give potential times for a report review meeting, but this is usually better to do later when delivering the report. Some firms will also provide a summary of findings after the assessment, but this differs from company to company. If you have communicated clearly with your client throughout the assessment, they should already have a good idea of what to expect. Bottom line, we don't want them to hear about the highest risk findings for the first time when they receive the report and be completely blindsided.

![text](https://academy.hackthebox.com/storage/modules/163/end_testing.png)

---

## Attack Path Recap

Writing out your attack path from start to finish is a good idea to help you visualize the path taken and what findings to pull out of it. This also helps to add to the report to summarize the attack chain walkthrough. This recap is a good way to ensure you didn't miss anything critical in the chain from initial access up to domain compromise. This recap should only show the path of least resistance and not include the extra steps taken or the entire thought process of testing, as this can clutter things up and make it difficult to follow.

Some penetration testing forms will structure their report in narrative form, given a step-by-step walkthrough of every action performed from start to finish and calling out the specific findings along the way. The approach here will differ from company to company. Regardless, it's a good idea to have this list handy to help with reporting, and if the client reaches out to ask how you achieved X.

---

## Structuring our Findings

Ideally, we have noted down findings as we test, including as many command outputs and evidence in our notetaking tool as possible. This should be done in a structured way, so it's easy to drop into the report. If we haven't been doing this, we should ensure we have a prioritized finding list and all necessary command output and screenshots before we lose access to the internal network or cease any external testing. We don't want to be in the position of asking the client to grant us access again to gather some evidence or run additional scans. We should have been structuring our findings list from `highest to lowest risk` as we test because this list can be beneficial to send to the client at the end of testing and is very helpful when drafting our report.

For more on notetaking and reporting, see the [Documentation & Reporting module](https://academy.hackthebox.com/module/162/section/1533). It's worth following the tips in that module for setting up your testing and notetaking environment and approaching the network in this module (Attacking Enterprise Networks) as a real-world penetration test, documenting and logging everything we do along the way. It's also great practice to use the sample report from the `Documentation & Reporting` module and create a report based on this network. This network has many opportunities to practice all aspects of report documentation and report writing.

---

## Post-Engagement Cleanup

If this were a real engagement, we should be noting down:

- `Every scan`
- `Attack attempt`
- `File placed on a system`
- `Changes made` (accounts created, minor configuration changes, etc.)

Before the engagement closes, we should delete any files we uploaded (tools, shells, payloads, notes) and restore everything to the way we found it. Regardless of if we were able to clean everything up, we should still note down in our report appendices every change, file uploaded, account compromise, and host compromise, along with the methods used. We should also retain our logs and a detailed activity log for a period after the assessment ends in case the client needs to correlate any of our testing activities with some alerts. Treat the network in this module like a real-world customer network. `Go back through a second time` and `pentest it as if it were an actual production network`, taking minimally invasive actions, noting down all actions that may require cleanup, and `clean up after yourself at the end!` This is a great habit to develop.

---

## Client Communication

We absolutely need to let the client know when testing has ended so they know when any abnormal activities they may be seeing are no longer related to our testing. We should also stay in clear communication during the reporting phase, providing the client with a precise delivery date for the report and a brief recap of our findings if asked. At this time, our manager or the Project Manager may be reaching out to the client to schedule a report review meeting, which we should expect to attend to walk through the results of our testing. If retesting is part of the Scope of Work, we should work with the client on a timeline for their remediation activities to be complete. However, they may not have an idea yet, so they may reach out regarding post-remediation testing at a later date. The client may reach out periodically over the next few days or weeks to correlate alerts they received, so we should have our notes and logs handy in case we need to justify or clarify any of our actions.

---

## Internal Project Closeout

Once the report has been delivered, and the closeout meeting is completed, your company will take various activities to close out the project, such as:

- Archiving the report and associated project data on a company share drive
- Hold a lessons learned debriefing
- Potentially fill out a post-engagement questionnaire for the sales team
- Perform administrative tasks such as invoicing

While it is best to have the original tester perform post-remediation testing, schedules may not align. We may need to do an internal knowledge transfer to another teammate. Now we should sit back, think about what went well and could be improved on during the assessment, and prepare for the next one.

---

## Next Steps

Now that you've completed this module, it's worth going back through the lab without the guide or minimal guidance to test your skills. Make a list of all the skills and associated modules covered in this lab and revisit topics you still have trouble with. Use this lab to hone your craft, try out different tools and techniques to complete the lab, practice documentation and reporting, and even prepare a briefing for a colleague or friend to practice your oral presentation skills. The following section provides more insight into additional steps we can take after finishing this module (and path). You may also want to consider working on one or more Pro Labs, which we recommend also approaching as a pentest to practice your engagement skills as much as possible.