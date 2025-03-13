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
