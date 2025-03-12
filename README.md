
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
