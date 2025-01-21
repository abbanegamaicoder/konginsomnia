
Scenario

When the submitter is the approver of the workflow process.
Given

The submitter initiates a process, and their role is verified as the approver.
When

The system identifies that the submitter and approver are the same person.
Then

The process is auto-approved, a notification is sent to the entitlement team for auto-approval, and the child process is closed with enum code 12.


Scenario

When the submitter is the approver, and the system integrates the auto-approval workflow with the SWBPM gateway for verification.
Given

The submitter initiates a process, and their role is verified as the approver in the JBPM system.
When

The system auto-approves the process, sends a notification to the entitlement team, closes the child process with the approved state, and integrates the workflow with the SWBPM gateway.
Then

The integration with the SWBPM gateway is verified, ensuring correct notifications, approvals, and process closures function as expected.





Scenario

When the workflow requires sequential two-level approval to approve the request and close the process.
Given

The BBS Profile Workflow is configured in the Rule Engine to handle two-level approvals, where the second approver is tasked only after the first approver approves the request.
When

The first approver approves the request, triggering the task assignment to the second approver for further action.
Then

The process is closed with an Approved State only if both approvers approve the request sequentially, and the request is marked as approved in the system.



