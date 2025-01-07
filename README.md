# konginsomnia
Closing Comment for Request Approved:
Implemented and tested the “Request Approved” functionality by identifying approvers, generating task IDs via the task generation API, and ensuring task creation and completion through the workflow. Verified the solution across scenarios and inputs (e.g., CA, additional role, watch list) for multiple teams, confirming approval tasks work as expected. Used enum 12 for approval. Exposed all relevant task APIs to the gateway, meeting the acceptance criteria.

Closing Comment for Request Rejected:
Implemented and tested the “Request Rejected” functionality by identifying approvers, generating task IDs via the task generation API, and ensuring task creation and completion through the workflow. Verified the solution across scenarios and inputs (e.g., CA, additional role, watch list) for multiple teams, confirming rejection tasks work as expected. Used enum 45 for rejection. Exposed all relevant task APIs to the gateway, meeting the acceptance criteria.
Closing Comment:
Implemented a new “Withdraw” state for the submitter in the child workflow, allowing them to withdraw a request. Added logic to dynamically abort or suspend the approver’s or submitter’s user task based on actions taken on either task. After analysis, found that the Suspend API caused the process instance to remain active despite reaching the end node. Resolved the issue by ensuring proper task completion and process termination.
Conference Page Documentation for New Joiners: Introduction to jBPM (Business Central)

Welcome to the jBPM (Business Central) platform! This document serves as a guide for new joiners to understand and get started with jBPM, which is deployed on BAM. It includes details about accessing the system, creating projects, and understanding key components like DRL files, business processes, and data objects.

Accessing jBPM (Business Central)

Before accessing jBPM, you must ensure the following:
	1.	Access Requirements:
	•	You need to raise access requests for the following functional groups:
	•	FGLB, BCP, PROD (Production Environment)
	•	FGLB, BCP, DEV (Development Environment)
	2.	Login to BAM:
	•	Once access is granted, log in to BAM using the provided credentials.
	3.	Open jBPM (Business Central):
	•	Navigate to the jBPM/Business Central interface within BAM.

Creating a Project in jBPM
	1.	Select a Space:
	•	Spaces are organizational units in jBPM where projects are grouped. Choose an existing space or create a new one.
	2.	Create a Project:
	•	Inside the selected space, click on “Add Project” and provide a name for your project.
	3.	Define Components:
	•	Under the project, you can create the following:
	•	Business Processes: Graphical workflows representing business logic.
	•	Data Objects: Java classes used to pass data in workflows.
	•	DRL Files (Rules): Decision logic written in Drools Rule Language.

Key Components of jBPM

1. Data Object
	•	A Data Object is a Java class representing business entities in a workflow.
	•	Example: If a workflow processes customer orders, the Order object can have fields like orderId, customerName, and orderAmount.

2. DRL (Drools Rule Language)
	•	A DRL file contains rules written in the Drools Rule Language, which helps define decision logic.
	•	Example Rule:

rule "Discount Rule"
when
    Order(orderAmount > 1000)
then
    System.out.println("Apply 10% discount");
end



3. Business Process
	•	A Business Process is a graphical workflow comprising various nodes (tasks, decisions, etc.).
	•	Example Nodes:
	•	Start Node: Entry point of the process.
	•	Script Task: Executes a script (e.g., Java code).
	•	End Node: Marks the completion of the process.

Example: Simple Business Process

Objective:

Create a basic workflow with:
	1.	A Start Node.
	2.	A Script Task that logs “Hello, jBPM!”.
	3.	An End Node.

Steps:
	1.	Open the project in Business Central.
	2.	Click on “New Process” and name it (e.g., HelloWorldProcess).
	3.	Drag and drop the following nodes:
	•	Start Node: Represents the start of the process.
	•	Script Task: Add a script task and configure it to print “Hello, jBPM!”.
	•	Script example (Java):

System.out.println("Hello, jBPM!");


	•	End Node: Represents the end of the process.

	4.	Connect the nodes in sequence:
	•	Start Node → Script Task → End Node.
	5.	Save and deploy the process.

Example Process Diagram

Node	Description
Start Node	Entry point of the process.
Script Task	Prints “Hello, jBPM!” to the console.
End Node	Marks the end of the process.

Conclusion

This guide provides a starting point for working with jBPM (Business Central). With the ability to define business processes, rules, and data objects, you can model complex workflows efficiently. For further queries, refer to the detailed jBPM documentation or reach out to your team lead.
