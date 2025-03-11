System.out.println("Getting InputType--->" + type);

kcontext.setVariable("inputType", type);
kcontext.setVariable("teamId", inout.getTeamId());

com.barclays.sw_entitlement.RuleOutputResponse outputResponse = inout.getRuleOutputResponse();

if (outputResponse != null) {
    System.out.println("Getting RequestStatus--->" + outputResponse.getRequestStatus());
    kcontext.setVariable("action", outputResponse.getRequestStatus());
}

boolean isAppSub = false;

java.util.List<String> approvers_List = new java.util.ArrayList<>();
java.util.List<String> approver_ntlm = new java.util.ArrayList<>();

System.out.println("In Human Task-->");

java.util.List<Users> app = new java.util.ArrayList<>();
java.util.List<Users> delegList = new java.util.ArrayList<>();

if (outputResponse != null) {
    app = outputResponse.getUsers();
    delegList = outputResponse.getDelegators();
}

String app_role = null;

try {
    if (app == null || app.isEmpty()) {
        System.out.println("No approvers found in the process variable 'approverList'.");
        return;
    }

    // Fix: Fetch submitter as a Users object and cast it correctly
    Object submitterObj = kcontext.getVariable("submitterId");
    Users submitterUser = null;

    if (submitterObj instanceof Users) {
        submitterUser = (Users) submitterObj;
    } else {
        System.out.println("Error: submitterId is not an instance of Users class.");
    }

    for (Users approver : app) {
        app_role = "{ \"UserId\":\"" + approver.getUserID() + "\", \"UserRole\":\"" + approver.getUserRole() + "\"}";

        if (!approver_ntlm.contains(approver.getUserID())) {
            approver_ntlm.add(approver.getUserID());
            approvers_List.add(app_role);
        }

        String userinfo = approver.getUserID();
        
        // Fix: Compare user IDs only if submitterUser is not null
        if (submitterUser != null && userinfo.equalsIgnoreCase(submitterUser.getUserID())) {
            isAppSub = true;
        }
    }

    // Fix: Check if submitterUser exists in delegate list
    if (!isAppSub && submitterUser != null) {
        isAppSub = delegList.contains(submitterUser);
    }

    kcontext.setVariable("isAutoApprove", isAppSub);
    kcontext.setVariable("approverList", approvers_List);
    kcontext.setVariable("approverIds", approver_ntlm);

} catch (Exception e) {
    e.printStackTrace();
    kcontext.setVariable("exceptionLog", e.toString());
}



System.out.println("Getting InputType-outputResponse.getRequestStatus());");
kcontext.setVariable("action", outputResponse.getRequestStatus());

boolean IsApp = false;

java.util.List<String> approversList = new java.util.ArrayList<>();
java.util.List<String> approverList = new java.util.ArrayList<>();

System.out.println("In Has Task");

java.util.List<Users> app = new java.util.ArrayList<>();
java.util.List<Users> delegList = new java.util.ArrayList<>();

if (outputResponse != null) {
    app = outputResponse.getUsers();
    delegList = outputResponse.getDelegators();
}

String app_role = null;

if (app == null || app.isEmpty()) {
    System.out.println("No approvers found in the process variable 'approverList'.");
    return;
}

con.barclays.entitlement.InputProcessRequest inputRequest = 
    (con.barclays.entitlement.InputProcessRequest) kcontext.getVariable("inputRequest");

Submitter submitter = (Submitter) kcontext.getVariable("submitter");

for (Users approver : app) {
    String app_role_entry = "{ \"UserId\": \"" + approver.getUserID() + "\", \"sarkale\": \"" + approver.getUserRole() + "\" }";
    
    if (!approverList.contains(app_role_entry)) {
        approverList.add(app_role_entry);
    }

    approversList.add(app_role_entry);
}

String userinfo = submitter.getUserId(); // Extract user ID from Submitter object

for (Users approver : app) {
    if (approver.getUserID().equalsIgnoreCase(userinfo)) {
        IsApp = true;
        break;
    }
}

if (!IsApp) {
    for (Users delegator : delegList) {
        if (delegator.getUserID().equalsIgnoreCase(userinfo)) {
            IsApp = true;
            break;
        }
    }
}

kcontext.setVariable("IsApp", IsApp);
kcontext.setVariable("approversList", approversList);
kcontext.setVariable("approverList", approverList);

} catch (Exception e) {
    e.printStackTrace();
    kcontext.setVariable("exception", e.toString());
}


For CST Rule Configuration, we are implementing a rule in the Rule Engine to determine whether the Securitization Methodologies field should be populated based on the Group’s Primary CST or Additional CSTs.
	•	Data Model: Defines CSTEvaluationRequest, which includes groupId, primaryCST, additionalCST1, and additionalCST2.
	•	Output Response: Defines CSTEvaluationResponse, which contains a collection of flags (e.g., secFlag) to allow future scalability.
	•	DRL Rule: Checks if any CST matches the 7 predefined CSTs and, if true, sets secFlag = true inside the response collection.

This ensures a structured and scalable approach, making it easier to extend if new flags need to be introduced later.


Here’s a concise and to-the-point comment you can add to your JIRA:

Comment:
Had a discussion with the application team (Appel and Sagar) regarding the CST Rule Configuration. They suggested that the security flag (Boolean type) should be part of a collection inside a data object to allow scalability for future flags. Based on this, I will modify the DRL rule to return the flag within a structured data object collection instead of a standalone Boolean value. Will proceed with implementing this approach in the Rule Engine.

Let me know if you need any refinements!


Stepwise Approach for Configuring the CST Rule in the Rule Engine

1. Understand the Requirements
	•	The Securitization Methodologies field should be populated only if the Primary CST or Additional CST (1 or 2) of a Group matches any of the 7 predefined CSTs.
	•	The rule should return a collection of flags inside a data object, not a standalone Boolean flag.

⸻

2. Define the Data Model (Java POJOs)

We need a structured data model to represent the incoming request and the response.

Data Object for CST Rule Evaluation (CSTEvaluationRequest.java)

public class CSTEvaluationRequest {
    private int groupId;
    private String primaryCST;
    private String additionalCST1;
    private String additionalCST2;

    // Getters and Setters
}

Data Object for CST Rule Output (CSTEvaluationResponse.java)

import java.util.HashMap;
import java.util.Map;

public class CSTEvaluationResponse {
    private int groupId;
    private Map<String, Boolean> flagCollection = new HashMap<>();

    public CSTEvaluationResponse(int groupId) {
        this.groupId = groupId;
    }

    public void addFlag(String flagName, Boolean value) {
        flagCollection.put(flagName, value);
    }

    // Getters and Setters
}



⸻

3. Write the DRL Rule (CSTRules.drl)

This rule will:
	1.	Check if any CST (Primary or Additional) matches the allowed 7 CSTs.
	2.	Set the security flag (secFlag) inside a collection and return it in the response object.

package com.rules.securitization;

import com.model.CSTEvaluationRequest;
import com.model.CSTEvaluationResponse;

rule "Determine Securitization Methodology Eligibility"
when
    $request : CSTEvaluationRequest(
        primaryCST in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                       "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                       "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
        || additionalCST1 in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                              "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                              "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
        || additionalCST2 in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                              "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                              "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
    )
then
    // Create response object and set the securitization flag
    CSTEvaluationResponse response = new CSTEvaluationResponse($request.getGroupId());
    response.addFlag("secFlag", true);

    insert(response);
    System.out.println("Security flag set to True for Group ID: " + $request.getGroupId());
end



⸻

4. Configure the Rule in jBPM

a) Add the DRL file to jBPM
	•	Place the CSTRules.drl file inside src/main/resources/rules/.

b) Set Up the KIE Session in Java

import org.kie.api.runtime.KieSession;
import org.kie.api.runtime.KieContainer;
import org.kie.api.KieServices;

public class RuleEngineExecutor {
    public CSTEvaluationResponse evaluateCSTRules(CSTEvaluationRequest request) {
        KieServices ks = KieServices.Factory.get();
        KieContainer kContainer = ks.getKieClasspathContainer();
        KieSession kSession = kContainer.newKieSession("rulesSession");

        kSession.insert(request);
        CSTEvaluationResponse response = new CSTEvaluationResponse(request.getGroupId());
        kSession.insert(response);
        kSession.fireAllRules();
        kSession.dispose();

        return response;
    }
}



⸻

5. How the Application Will Consume the Rule Engine
	•	The application will call evaluateCSTRules() with input data.
	•	The rule will process the data and return a collection of flags.
	•	The Limit Application will use this response to determine if the Securitization Methodologies field should be shown.

⸻

6. Testing the Rule

JUnit Test Case

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CSTRuleTest {
    @Test
    public void testCSTRule() {
        CSTEvaluationRequest request = new CSTEvaluationRequest();
        request.setGroupId(10339);
        request.setPrimaryCST("CRMD INDIA");

        RuleEngineExecutor executor = new RuleEngineExecutor();
        CSTEvaluationResponse response = executor.evaluateCSTRules(request);

        assertTrue(response.getFlagCollection().get("secFlag"));
    }
}



⸻

Summary of Steps

Step	Description
1. Understand Requirements	Define conditions and expected output.
2. Define Data Model	Create request and response Java objects.
3. Write the DRL Rule	Implement logic in Drools to check CSTs and set flags in a collection.
4. Configure in jBPM	Add DRL file, configure KIE session.
5. Integration with Application	Ensure the response is consumed correctly.
6. Testing	Write JUnit tests to validate rule execution.



⸻

This approach ensures the rule is scalable, reusable, and future-proof while aligning with the requirement to return the flag inside a collection.

Let me know if you need any modifications!


Here’s a Drools Rule (DRL file) to implement the rule logic for checking the Primary or Additional CST and determining whether the Securitization Methodologies field should be populated.

⸻

Rule Explanation:
	1.	Input Data:
	•	The Group ID, Primary CST, and Additional CSTs are passed from the application.
	2.	Rule Logic:
	•	If Primary CST or Additional CST 1/2 belongs to the specified list of 7 CSTs, the rule sets a flag (secFlag) as True.
	3.	Output Data:
	•	The rule will return secFlag = True when the condition is met.
	4.	Integration:
	•	This DRL rule will be configured in jBPM Rule Engine, and the application will consume its output.

⸻

DRL Rule: SecuritizationMethodology.drl

package com.rules.securitization;

import com.model.FacilityDetails;

rule "Check Securitization Methodologies"
when
    // FacilityDetails object contains the Group's CST information
    $facility : FacilityDetails( 
        primaryCST in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                       "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                       "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
        || additionalCST1 in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                              "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                              "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
        || additionalCST2 in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                              "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                              "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
    )
then
    // Set securitization flag to true
    $facility.setSecFlag(true);
    System.out.println("Securitization Methodologies field is enabled for Group ID: " + $facility.getGroupId());
end



⸻

Steps to Implement in jBPM
	1.	Define Model Class (FacilityDetails.java)
Ensure your model has appropriate fields:

public class FacilityDetails {
    private int groupId;
    private String primaryCST;
    private String additionalCST1;
    private String additionalCST2;
    private boolean secFlag;
    
    // Getters and Setters
}


	2.	Load the Rule Engine in jBPM
	•	Add the rule to your jBPM project.
	•	Use the KIE session to execute rules against incoming data.
	3.	Application Consumption
	•	The output (secFlag) is used by the Facility Details Screen to populate the field accordingly.

⸻

Expected Output

Group ID	Primary CST	Additional CST 1	Additional CST 2	SecFlag
10339	CRMD INDIA	XYZ	ABC	True
20321	ABC	DEF	XYZ	False
40567	IB-BE SECURITISATN	XYZ	ABC	True



⸻

This Drools rule ensures that the Securitization Methodologies field is enabled only when the given Primary or Additional CSTs match the allowed 7 CSTs. Let me know if you need modifications or explanations!


nalysis of Approaches for Automating jBPM Query Storage and Execution Using Jenkins and GitLab CI/CD

For automating the storage and execution of jBPM custom queries, we have two possible approaches: Jenkins Job and GitLab CI/CD Pipeline. Below is a detailed analysis of both pathways, including the implementation steps, advantages, limitations, and a recommendation on the best approach.

Approach 1: Jenkins Job

Implementation Steps
	1.	Define jBPM Query API or Database Access
	•	Identify whether jBPM provides REST APIs for storing and executing queries or if direct database access is required.
	2.	Set Up Jenkins
	•	Install required plugins like Pipeline, Git, and HTTP Request (if using REST APIs).
	•	Ensure a Jenkins Agent is available to run the job.
	3.	Create a New Jenkins Job
	•	Define it as a Freestyle Job or Pipeline Job based on flexibility needs.
	4.	Configure Query Storage and Execution
	•	If using APIs, set up cURL commands to store and execute queries.
	•	If using a database, execute SQL scripts via a shell or database plugin.
	5.	Implement the Jenkins Pipeline Script

pipeline {
    agent any
    stages {
        stage('Store Query') {
            steps {
                script {
                    sh "curl -X POST -H 'Content-Type: application/json' -d '{\"query\":\"YOUR_QUERY\"}' http://jbpm-server/storeQuery"
                }
            }
        }
        stage('Execute Query') {
            steps {
                script {
                    sh "curl -X POST -H 'Content-Type: application/json' -d '{\"queryId\": \"QUERY_ID\"}' http://jbpm-server/executeQuery"
                }
            }
        }
    }
}


	6.	Triggering Mechanism
	•	Can be manually triggered, scheduled, or triggered via webhooks.
	7.	Monitoring & Logging
	•	View console logs in Jenkins for API responses or database execution status.
	•	Implement email notifications for job success/failure.

Advantages of Jenkins Approach

✅ Flexible Scheduling: Jobs can be scheduled periodically or triggered based on events.
✅ Extensive Plugin Support: Supports integration with various tools (Git, Slack, Databases).
✅ User-Controlled Execution: Jobs can be manually triggered for better control.

Limitations of Jenkins Approach

❌ Extra Setup Required: Jenkins server setup and maintenance are necessary.
❌ Not Integrated with Git: Requires separate job configurations outside version control.
❌ Scalability Issues: If multiple queries need execution, managing jobs may become complex.

Approach 2: GitLab CI/CD Pipeline

Implementation Steps
	1.	Define GitLab CI/CD Pipeline Configuration
	•	Create a .gitlab-ci.yml file for pipeline execution.
	2.	Set Up GitLab Runner
	•	Ensure a GitLab Runner is configured for executing scripts.
	3.	Implement the Pipeline

stages:
  - store_query
  - execute_query

store_query:
  stage: store_query
  script:
    - echo "Storing query in jBPM..."
    - curl -X POST -H "Content-Type: application/json" -d '{"query":"YOUR_QUERY"}' http://jbpm-server/storeQuery
  only:
    - main

execute_query:
  stage: execute_query
  script:
    - echo "Executing stored query..."
    - curl -X POST -H "Content-Type: application/json" -d '{"queryId": "QUERY_ID"}' http://jbpm-server/executeQuery
  only:
    - main


	4.	Triggering Mechanism
	•	Automatically executes on code commit or merge requests.
	5.	Monitoring & Logging
	•	Check pipeline execution logs in GitLab CI/CD dashboard.
	•	Configure email notifications for failures.

Advantages of GitLab CI/CD Approach

✅ Version Control Integration: Pipeline scripts are stored in Git, allowing better tracking.
✅ No Extra Tooling Needed: No need for separate Jenkins setup, reducing overhead.
✅ Automated Execution: Runs automatically on code changes, ensuring consistency.
✅ Better Scalability: Can handle multiple queries more efficiently.

Limitations of GitLab CI/CD Approach

❌ Less Manual Control: Pipelines are tied to Git events, making ad-hoc executions harder.
❌ Requires GitLab Runner: A properly configured runner is necessary for execution.
❌ Limited Scheduling Options: Unlike Jenkins, does not support custom job scheduling easily.

Comparison: Jenkins vs. GitLab CI/CD

Feature	Jenkins	GitLab CI/CD
Ease of Setup	Requires Jenkins installation and configuration	No extra setup if using GitLab
Integration with Git	Requires additional setup	Seamless integration
Execution Control	Manual, scheduled, or event-based	Triggered by Git events
Scalability	Complex for multiple queries	Easier to manage in version control
Monitoring	Console logs in Jenkins	CI/CD pipeline logs in GitLab
Best For	Teams needing manual control and scheduled executions	Teams looking for automated, Git-integrated workflows

Recommendation

1️⃣ If the requirement is to execute queries on a scheduled or ad-hoc basis, Jenkins is a better choice.
	•	It provides better manual control, flexible scheduling, and plugin support.
	•	Suitable when the process is independent of Git commits.

2️⃣ If the requirement is to automate query execution based on repository changes, GitLab CI/CD is the preferred option.
	•	It eliminates extra tool setup, integrates with Git, and ensures consistency.
	•	Ideal for teams already using GitLab for CI/CD workflows.

Final Decision:
If there is a strong need for manual execution and scheduling, Jenkins is the better option. However, if automation and Git-based execution are the primary requirements, GitLab CI/CD is the way forward.

Would you like a proof-of-concept (POC) for either approach?
