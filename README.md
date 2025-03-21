Here’s a professional closing comment for your JIRA:

package com.sw.sw_limits;

public enum CreditSanctionTeams {
    CRMD_INDIA,
    CRMD_SECURITISTN_EUR,
    CRMDNY_HEDGE_FUNDS,
    CRMONY_SECURITIZATN,
    CRMDNY_SAM_IB_US_LEG,
    BUK_CCR_FI_SOV,
    IB_BE_SECURITISATN;

    public static boolean contains(String team) {
        for (CreditSanctionTeams cst : values()) {
            if (cst.name().equalsIgnoreCase(team.replace(" ", "_"))) {
                return true;
            }
        }
        return false;
    }
}

-----


package com.sw.sw_limits;

import java.util.*;
import com.sw.sw_limits.CreditSanctionTeams;

rule "Check and Set Securitization Methodology Flag False"
salience 15

when 
    $rio: CSTLimitsRuleInputOutput(cstRequest != null && cstRequest.size() > 0)
then
    List<CSTLimits> cstLimitsList = $rio.getCstRequest(); 
    CSTLimitsRuleOutput output = new CSTLimitsRuleOutput();
    boolean temp = false;

    for (CSTLimits cst : cstLimitsList) {
        if (cst.getCreditSanctionTeam() != null && CreditSanctionTeams.contains(cst.getCreditSanctionTeam())) {
            temp = true;
            break;  // No need to check further once found
        }
    }

    output.setSecFlag(temp);
    $rio.setCSTLimitsRuleOutput(output);
end

⸻

JIRA Closing Comment:

The rule for setting the Securitization Methodology Flag has been successfully implemented and tested. The testing covered the following scenarios:
	1.	Primary CST is one of the predefined seven teams – The SEC flag is correctly set as expected.
	2.	Additional CST is one of the predefined seven teams – The SEC flag is correctly set as expected.
	3.	Neither Primary nor Additional CST belongs to the predefined seven teams – The SEC flag remains unset, as expected.
	4.	Both Primary and Additional CSTs belong to the predefined seven teams – The SEC flag is correctly set as expected.

All test cases have passed, and the results align with the expected outcomes. The implementation is now complete and ready for closure.

⸻

Let me know if you’d like any refinements!



package com.sw.sw_limits;

rule "Check and Set Securitization Methodology Flag"
    salience 15
    ruleflow-group "securitization-methodology"

when
    $input: CSTLimitsRuleInputOutput(cstDetails != null)

    SmatchingCST: CSTLimits (
        creditSanctionTeam == "CRMD INDIA" || 
        creditSanctionTeam == "CRPD SECURITISTH EUR" || 
        creditSanctionTeam == "CRMONY HEDGE FUNDS" || 
        creditSanctionTeam == "CRMONY SECURITIZAIN" || 
        creditSanctionTeam == "CRMONY SAN IB US LEG" || 
        creditSanctionTeam == "BUK-CCR-FI S850V" || 
        creditSanctionTeam == "IB-BE SECURITISATN"
    ) from $input.cstDetails

then
    CSTLimitsRuleOutput output = new CSTLimitsRuleOutput();
    output.setSecFlag(true);

    insert(output);
    update(output);

    $input.setCstLimitsRuleOutput(output);
end


-------
package com.rules.securitization;

import com.model.CSTLimitsRuleInputOutput;
import com.model.CSTLimitsRuleOutput;

rule "Check and Set Securitization Methodology Flag"
when
    // Loop over all CST entries in the input list
    $input : CSTLimitsRuleInputOutput(cstList != null)
    // Check if any CST has a matching creditSanctionTeam
    $matchingCST : CSTLimitsRuleInputOutput.CSTInput(
        creditSanctionTeam in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                               "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                               "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
    )
then
    // If any match is found, set secFlag to true
    CSTLimitsRuleOutput output = new CSTLimitsRuleOutput();
    output.setSecFlag(true);
    insert(output);
    
    // If no match is found, set secFlag to false
    // This is achieved by the not condition
    not CSTLimitsRuleInputOutput.CSTInput(
        creditSanctionTeam in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                               "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                               "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
    )
    // Create an output object with secFlag as false
    CSTLimitsRuleOutput outputFalse = new CSTLimitsRuleOutput();
    outputFalse.setSecFlag(false);
    insert(outputFalse);
end




package com.rules.securitization;

import com.model.CSTLimitsRuleInputOutput;
import com.model.CSTLimitsRuleOutput;

rule "Check and Set Securitization Methodology Flag"
when
    // Loop over all CST entries in the input list
    $input : CSTLimitsRuleInputOutput(cstList != null)
    // Check if any CST has a matching creditSanctionTeam
    $matchingCST : CSTLimitsRuleInputOutput.CSTInput(
        creditSanctionTeam in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                               "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                               "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
    )
then
    // If any match is found, set secFlag to true
    CSTLimitsRuleOutput output = new CSTLimitsRuleOutput();
    output.setSecFlag(true);
    insert(output);
    
    // If no match is found, set secFlag to false
    // This is achieved by the not condition
    not CSTLimitsRuleInputOutput.CSTInput(
        creditSanctionTeam in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                               "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                               "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
    )
    // Create an output object with secFlag as false
    CSTLimitsRuleOutput outputFalse = new CSTLimitsRuleOutput();
    outputFalse.setSecFlag(false);
    insert(outputFalse);
end







package com.rules.securitization;

import com.model.CSTLimitsRuleInputOutput;
import com.model.CSTLimitsRuleOutput;

rule "Check Securitization Methodology Eligibility"
when
    // Loop over all CST entries in the input list
    $input : CSTLimitsRuleInputOutput(cstList != null)
    $cst : CSTLimitsRuleInputOutput.CSTInput(
        creditSanctionTeam in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                               "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                               "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
    )
then
    // Create a new output object
    CSTLimitsRuleOutput output = new CSTLimitsRuleOutput();
    
    // Check if any matching CST is primary and set the secFlag accordingly
    boolean flag = false;
    if ($cst.isPrimary) {
        // Set flag to true if any matching CST is primary
        flag = true;
    }

    // Set the secFlag in the output
    output.setSecFlag(flag);

    // Insert the result back into the working memory
    insert(output);
end


package com.rules.securitization;

import com.model.CSTLimitsRuleInputOutput;
import com.model.CSTLimitsRuleOutput;

rule "Check Securitization Methodology Eligibility"
when
    // Loop over all CST entries in the input list
    $input : CSTLimitsRuleInputOutput(cstList != null)
    $cst : CSTLimitsRuleInputOutput.CSTInput(
        creditSanctionTeam in ("CRMD INDIA", "CRMD SECURITISTN EUR", "CRMDNY HEDGE FUNDS", 
                               "CRMDNY SECURITIZATN", "CRMDNY SAM IB US LEG", 
                               "BUK-CCR-FI'S&SOV", "IB-BE SECURITISATN")
    )
then
    // Create a new output object
    CSTLimitsRuleOutput output = new CSTLimitsRuleOutput();
    List<CSTLimitsRuleOutput.EntityEntitlement> entitlements = new ArrayList<>();

    // Based on the "isPrimary" flag, add to entitlement list
    if ($cst.isPrimary) {
        if ($cst.creditSanctionTeam.equals("CRMD INDIA")) {
            entitlements.add(new CSTLimitsRuleOutput.EntityEntitlement("BI", true));  // Example: Set "BI" as true for primary team
        }
        if ($cst.creditSanctionTeam.equals("IB-BE SECURITISATN")) {
            entitlements.add(new CSTLimitsRuleOutput.EntityEntitlement("BE", false)); // Example: Set "BE" as false for this team
        }
    }

    // Set the output entitlement list
    output.setEntitlement(entitlements);
    
    // Insert the result back into the working memory
    insert(output);
end
package com.example.rules;

import com.example.model.CSTData;

rule "Check Securitization Methodology for Primary CST"
    salience 10
    ruleflow-group "cstValidationGroup" // Define the rule flow group
when
    $cstData : CSTData(
        (creditSanctionTeam == "CRMD INDIA" ||
         creditSanctionTeam == "CRMD SECURITISTN EUR" ||
         creditSanctionTeam == "CRMDNY HEDGE FUNDS" ||
         creditSanctionTeam == "CRMDNY SECURITIZATN" ||
         creditSanctionTeam == "CRMDNY SAM IB US LEG" ||
         creditSanctionTeam == "BUK-CCR-FI'S&SOV" ||
         creditSanctionTeam == "IB-BE SECURITISATN") &&
        isPrimary == true
    )
then
    $cstData.setSecFlag(true);
end

rule "Check Securitization Methodology for Additional CST"
    salience 5
    ruleflow-group "cstValidationGroup" // Define the rule flow group
when
    $cstData : CSTData(
        (creditSanctionTeam == "CRMD INDIA" ||
         creditSanctionTeam == "CRMD SECURITISTN EUR" ||
         creditSanctionTeam == "CRMDNY HEDGE FUNDS" ||
         creditSanctionTeam == "CRMDNY SECURITIZATN" ||
         creditSanctionTeam == "CRMDNY SAM IB US LEG" ||
         creditSanctionTeam == "BUK-CCR-FI'S&SOV" ||
         creditSanctionTeam == "IB-BE SECURITISATN") &&
        isPrimary == false
    )
then
    $cstData.setSecFlag(true);
end
