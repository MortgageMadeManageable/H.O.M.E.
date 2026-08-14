<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="mobile-web-app-capable" content="yes">
<title>Mortgage Made Manageable</title>
<link rel="manifest" href="data:application/manifest+json,{%22name%22:%22ROSE%20Budget%20Tool%22,%22short_name%22:%22ROSE%22,%22start_url%22:%22.%22,%22display%22:%22standalone%22,%22background_color%22:%22%23111827%22,%22theme_color%22:%22%23111827%22}">
<style>
:root {
  --bg: #111827;
  --card: #1f2937;
  --text: #f3f4f6;
  --accent: #38bdf8;
  --border: #374151;
}
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  background: var(--bg);
  color: var(--text);
  padding: env(safe-area-inset-top, 20px) 20px 20px 20px;
  margin: 0;
  display: flex;
  justify-content: center;
}
.calc-box {
  width: 100%;
  max-width: 440px;
  background: var(--card);
  padding: 25px;
  border-radius: 16px;
  border: 1px solid var(--border);
  box-sizing: border-box;
  margin-top: 10px;
}
h2 {
  margin: 0 0 5px;
  font-size: 22px;
  font-weight: 700;
  color: #fff;
  text-align: center;
}
  .home-title {
  font-size: 32px; 
  display: block;  
  margin-top: 5px; 
}
.subtitle {
  font-size: 13px;
  color: #9ca3af;
  text-align: center;
  margin-bottom: 25px;
}
label {
  display: block;
  margin: 18px 0 6px;
  font-weight: 600;
  font-size: 14px;
  color: #e5e7eb;
}
.input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}
.prefix {
  position: absolute;
  left: 12px;
  color: #9ca3af;
  font-size: 16px;
  pointer-events: none;
}
input[type="number"], select {
  const downPaymentCashAmount = (price * downPercent) / 100;
  const formattedCash = downPaymentCashAmount.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
  
  // This updates JUST the inner text, preserving the container properties so it stacks vertically
  const pmiDisplayEl = document.getElementById('pmiDisplay');
  if (pmiDisplayEl) {
    pmiDisplayEl.innerText = `Down Payment: $${formattedCash} | PMI Breakdown: $0.00 / mo`;
  }
  // =========================================

// 1. Calculate Core Loan Mechanics
  const downDollarAmount = price * (downPercent / 100);
  const loanAmount = Math.max(0, price - downDollarAmount);
  const ltvRatio = price > 0 ? (loanAmount / price) * 100 : 0;

  // Format the down payment dollar amount for display
  const formattedDownCash = downDollarAmount.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});

  // 2. Process PMI Flags (With VA Exception Overrides)
  const isVALoan = document.getElementById('vaLoanToggle').checked;
  let monthlyPMI = 0;

  if (isVALoan) {
    // VA loans explicitly carry $0 monthly PMI penalties
    document.getElementById('pmiDisplay').innerText = `Down Payment: $${formattedDownCash} | PMI Breakdown: $0.00 / mo (VA Loan)`;
    document.getElementById('pmiDisplay').style.color = '#38bdf8'; // Theme blue accent for approved status
  } else if (ltvRatio > 80 && loanAmount > 0) {
    const annualPMIRate = 0.007; 
    monthlyPMI = (loanAmount * annualPMIRate) / 12;
    const formattedPMI = monthlyPMI.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
    
    document.getElementById('pmiDisplay').innerText = `Down Payment: $${formattedDownCash} | PMI Breakdown: $${formattedPMI} / mo (LTV: ${ltvRatio.toFixed(1)}%)`;
    document.getElementById('pmiDisplay').style.color = '#fbbf24'; 
  } else {
    document.getElementById('pmiDisplay').innerText = `Down Payment: $${formattedDownCash} | PMI Breakdown: $0.00 / mo (LTV: ${ltvRatio.toFixed(1)}% - No PMI)`;
    document.getElementById('pmiDisplay').style.color = '#9ca3af'; 
  }

  // 3. Insurance Processing
  const monthlyIns = annualIns / 12;
  document.getElementById('monthlyInsDisplay').innerText = 'Monthly breakdown: $' + monthlyIns.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2}) + ' / mo';

  // 4. Principal & Interest Formula Mechanics
  const monthlyRate = (annualRate / 100) / 12;
  const totalPayments = 360;
  let monthlyPI = 0;
  
  if (monthlyRate === 0) {
    monthlyPI = loanAmount / totalPayments;
  } else {
    monthlyPI = loanAmount * (monthlyRate * Math.pow(1 + monthlyRate, totalPayments)) / (Math.pow(1 + monthlyRate, totalPayments) - 1);
  }

   // 5. Dynamic Loop Engine with Unified ISD Homestead Logic
  const activeTaxEntities = TAX_DATABASE[selectedZip] || [];
  const isHomesteadExempt = document.getElementById('homesteadToggle').checked;
  let annualTaxSum = 0;
  let breakdownHTML = '';

  activeTaxEntities.forEach(entity => {
    // Check if the entity is a School District and toggle is active
    let taxableValue = price;
    if (isHomesteadExempt && entity.name.toUpperCase().includes("ISD")) {
      taxableValue = Math.max(0, price - 140000);
    }

    const entityTaxAmount = taxableValue * entity.rate;
    annualTaxSum += entityTaxAmount;
    
    breakdownHTML += `
      <div class="tax-row">
        <span>${entity.name} (${(entity.rate * 100).toFixed(4)}%)</span>
        <span>$${Math.round(entityTaxAmount).toLocaleString('en-US')}/yr</span>
      </div>
<style>
  /* Animates and colors both tracks when checked */
  #homesteadToggle:checked + .slider,
  #vaLoanToggle:checked + .slider {
    background-color: #38bdf8 !important; /* Soft accent blue */
  }

  /* Moves both white knob circles to the right when checked */
  #homesteadToggle:checked + .slider:before,
  #vaLoanToggle:checked + .slider:before {
    transform: translateX(20px) !important; 
  }
  /* Creates the white physical circle knob on the slider */
  .slider:before {
    position: absolute;
    content: "";
    height: 18px;
    width: 18px;
    left: 3px;
    bottom: 3px;
    background-color: white;
    transition: .2s;
    border-radius: 50%;
  }
</style>
    `;
  });

  const monthlyTax = annualTaxSum / 12;
  const combinedPercentage = activeTaxEntities.reduce((sum, item) => sum + item.rate, 0) * 100;

  breakdownHTML += `
    <div class="tax-row total-row">
      <span>Est. Combined Rate</span>
      <span>${combinedPercentage.toFixed(3)}%</span>
    </div>
  `;
  document.getElementById('taxBreakdownContainer').innerHTML = breakdownHTML;

  // 6. Final Computations
  const totalPITI = monthlyPI + monthlyTax + monthlyIns + monthlyPMI;
  const lowRange = Math.floor((totalPITI - 100) / 100) * 100;
  const highRange = Math.ceil((totalPITI + 100) / 100) * 100;

   // 7. Render Output to DOM
  document.getElementById('totalMonthly').innerText = '$' + totalPITI.toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2});
  document.getElementById('ballparkRange').innerText = 'Safe Budget Range: $' + lowRange.toLocaleString('en-US') + ' to $' + highRange.toLocaleString('en-US');
}

calculatePITI();
function handleVAToggle() {
  const isVA = document.getElementById('vaLoanToggle').checked;
  if (isVA) {
    document.getElementById('downPayment').value = 0;
  } else {
    // Automatically restore your 20% default when the VA toggle is turned off
    document.getElementById('downPayment').value = 20;
  }
  calculatePITI();
}
</script>

</body>
</html>
