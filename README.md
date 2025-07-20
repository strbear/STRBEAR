<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>STR Squad Custom Ratio Calculator</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #1c1c1c;
      padding: 20px;
      color: #fff;
    }
    h1 {
      color: #ffffff;
      background-color: #333;
      padding: 10px;
      border-radius: 10px;
    }
    p.description {
      margin-top: 10px;
      background-color: #2e2e2e;
      padding: 10px;
      border-radius: 5px;
      color: #ccc;
    }
    label, input {
      display: block;
      margin: 10px 0;
    }
    input[type="number"] {
      width: 100%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }
    button {
      margin-top: 20px;
      padding: 10px 20px;
      background-color: #3498db;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background-color: #2980b9;
    }
    .output {
      margin-top: 20px;
      padding: 15px;
      background-color: #2e2e2e;
      border-radius: 5px;
      color: #fff;
    }
    label {
      background-color: #2e2e2e;
      padding: 5px;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <h1>STR Squad Custom Ratio Calculator</h1>
  <p class="description">
    Welcome STR commanders! Enter your troop counts and set a custom ratio (Infantry:Lancer:Marksman) for each squad. No more excuses — just precise deployment!
  </p>

  <label>How many squads do you have?</label>
  <button onclick="setSquadCount(5)">5</button>
  <button onclick="setSquadCount(6)">6</button>
  <button onclick="setSquadCount(7)">7</button>
  <p id="squadCountDisplay"></p>

  <label for="infantry">Total Infantry:</label>
  <input type="number" id="infantry" placeholder="Enter total Infantry count">

  <label for="lancer">Total Lancer:</label>
  <input type="number" id="lancer" placeholder="Enter total Lancer count">

  <label for="marksman">Total Marksman:</label>
  <input type="number" id="marksman" placeholder="Enter total Marksman count">

  <label>Do you want to use the same ratio for all squads?</label>
  <button onclick="setSameRatio(true)">Yes</button>
  <button onclick="setSameRatio(false)">No</button>
  <p id="sameRatioChoice"></p>

  <div id="customInputArea">
    <div id="squadCapacitiesContainer"></div>
    <button onclick="calculateCustomDistribution()">Calculate Custom Distribution</button>
    <div class="output" id="output"></div>
  </div>

  <script>
    let squadCount = 0;
    let useSameRatio = false;

    function setSameRatio(choice) {
      useSameRatio = choice;
      document.getElementById('sameRatioChoice').innerText = choice ? 'Using same ratio for all squads.' : 'Custom ratios for each squad.';
      setSquadCount(squadCount);
    }

    function setSquadCount(count) {
      squadCount = count;
      document.getElementById('squadCountDisplay').innerText = `Selected Squad Count: ${count}`;
      const container = document.getElementById('squadCapacitiesContainer');
      container.innerHTML = '';

      for (let i = 0; i < count; i++) {
        container.innerHTML += `
          <label>Squad ${i + 1} Capacity:</label>
          <input type="number" id="squadCap${i}" placeholder="Enter capacity">`;

        if (!useSameRatio) {
          container.innerHTML += `
            <div style="display: flex; gap: 10px; align-items: center;">
              <div style="flex: 1;">
                <label>Infantry %:</label>
                <input type="number" id="squadInf${i}" placeholder="Blank for default">
              </div>
              <div style="flex: 1;">
                <label>Lancer %:</label>
                <input type="number" id="squadLan${i}" placeholder="Blank for default">
              </div>
              <div style="flex: 1;">
                <label>Marksman %:</label>
                <input type="number" id="squadMar${i}" placeholder="Blank for default">
              </div>
            </div>`;
        }
      }
    }

    function calculateCustomDistribution() {
      let infantry = parseInt(document.getElementById('infantry').value);
      let lancer = parseInt(document.getElementById('lancer').value);
      let marksman = parseInt(document.getElementById('marksman').value);

      let output = '';

      for (let i = 0; i < squadCount; i++) {
        const cap = parseInt(document.getElementById(`squadCap${i}`).value);
        const infPct = parseInt(document.getElementById(`squadInf${i}`)?.value);
        const lanPct = parseInt(document.getElementById(`squadLan${i}`)?.value);
        const marPct = parseInt(document.getElementById(`squadMar${i}`)?.value);

        let iCount = 0, lCount = 0, mCount = 0;

        if (!isNaN(infPct) && !isNaN(lanPct) && !isNaN(marPct)) {
          const totalPct = infPct + lanPct + marPct;
          iCount = Math.floor((infPct / totalPct) * cap);
          lCount = Math.floor((lanPct / totalPct) * cap);
          mCount = cap - iCount - lCount;

          // Check troop availability for custom ratio
          if (infantry < iCount || lancer < lCount || marksman < mCount) {
            let missingInf = Math.max(0, iCount - infantry);
            let missingLan = Math.max(0, lCount - lancer);
            let missingMar = Math.max(0, mCount - marksman);
            output += `Squad ${i + 1}: ❌ Not enough troops! Train `;
            let partsMissing = [];
            if (missingInf > 0) partsMissing.push(`${missingInf} Infantry`);
            if (missingLan > 0) partsMissing.push(`${missingLan} Lancer`);
            if (missingMar > 0) partsMissing.push(`${missingMar} Marksman`);
            output += partsMissing.join(', ') + ` to meet the required ratio.<br>`;
            continue;
          }
        } else {
          // Collect all squads needing default logic
          if (!window.defaultSquads) window.defaultSquads = [];
          window.defaultSquads.push({ i, cap });
          continue;
        }

        if (infantry < iCount || lancer < lCount || marksman < mCount) {
          let missingInf = Math.max(0, iCount - infantry);
          let missingLan = Math.max(0, lCount - lancer);
          let missingMar = Math.max(0, mCount - marksman);
          output += `Squad ${i + 1}: ❌ Not enough troops! Train `;
          let partsMissing = [];
          if (missingInf > 0) partsMissing.push(`${missingInf} Infantry`);
          if (missingLan > 0) partsMissing.push(`${missingLan} Lancer`);
          if (missingMar > 0) partsMissing.push(`${missingMar} Marksman`);
          output += partsMissing.join(', ') + ` to meet the required ratio.<br>`;
        } else {
          infantry -= iCount;
          lancer -= lCount;
          marksman -= mCount;
          let squadTotal = iCount + lCount + mCount;
let squadInfPct = squadTotal > 0 ? Math.round((iCount / squadTotal) * 100) : 0;
let squadLanPct = squadTotal > 0 ? Math.round((lCount / squadTotal) * 100) : 0;
let squadMarPct = 100 - squadInfPct - squadLanPct;
output += `Squad ${i + 1}: ✅ Infantry: ${squadInfPct}%, Lancer: ${squadLanPct}%, Marksman: ${squadMarPct}% (Total: ${cap})<br>`;
        }
      }

      output += `<br><strong>Remaining Troops:</strong> Infantry: ${infantry}, Lancer: ${lancer}, Marksman: ${marksman}`;

      let usedInfantry = parseInt(document.getElementById('infantry').value) - infantry;
      let usedLancer = parseInt(document.getElementById('lancer').value) - lancer;
      let usedMarksman = parseInt(document.getElementById('marksman').value) - marksman;
      let totalUsed = usedInfantry + usedLancer + usedMarksman;
      let usedInfPct = totalUsed > 0 ? Math.round((usedInfantry / totalUsed) * 100) : 0;
      let usedLanPct = totalUsed > 0 ? Math.round((usedLancer / totalUsed) * 100) : 0;
      let usedMarPct = 100 - usedInfPct - usedLanPct;
      output += `<br><br><strong>Used Troop Ratio:</strong> Infantry: ${usedInfPct}%, Lancer: ${usedLanPct}%, Marksman: ${usedMarPct}%`;

      // Now process default squads if any
      if (window.defaultSquads && window.defaultSquads.length > 0) {
        const totalRemainingMarksman = marksman;
        const totalRemainingInfantry = infantry;
        const totalRemainingLancer = lancer;

        const totalDefaultCap = window.defaultSquads.reduce((sum, s) => sum + s.cap, 0);

        for (const s of window.defaultSquads) {
          const { i, cap } = s;

          const mCount = Math.min(marksman, Math.floor((cap / totalDefaultCap) * totalRemainingMarksman));
          let remainingCap = cap - mCount;

          let iCount = Math.min(infantry, Math.floor(remainingCap / 2));
          let lCount = Math.min(lancer, remainingCap - iCount);

          if (iCount < remainingCap / 2) {
            let needed = Math.floor(remainingCap / 2) - iCount;
            let availableLan = lancer - lCount;
            lCount += Math.min(needed, availableLan);
          }

          if (lCount < (remainingCap - iCount)) {
            let needed = remainingCap - iCount - lCount;
            let availableInf = infantry - iCount;
            iCount += Math.min(needed, availableInf);
          }

          if (infantry < iCount || lancer < lCount || marksman < mCount) {
            let missingInf = Math.max(0, iCount - infantry);
            let missingLan = Math.max(0, lCount - lancer);
            let missingMar = Math.max(0, mCount - marksman);
            output += `Squad ${i + 1}: ❌ Not enough troops! Train `;
            let partsMissing = [];
            if (missingInf > 0) partsMissing.push(`${missingInf} Infantry`);
            if (missingLan > 0) partsMissing.push(`${missingLan} Lancer`);
            if (missingMar > 0) partsMissing.push(`${missingMar} Marksman`);
            output += partsMissing.join(', ') + ` to meet the default ratio.<br>`;
            continue;
          }

          infantry -= iCount;
          lancer -= lCount;
          marksman -= mCount;

          let squadTotal = iCount + lCount + mCount;
          let squadInfPct = squadTotal > 0 ? Math.round((iCount / squadTotal) * 100) : 0;
          let squadLanPct = squadTotal > 0 ? Math.round((lCount / squadTotal) * 100) : 0;
          let squadMarPct = 100 - squadInfPct - squadLanPct;
          output += `Squad ${i + 1}: ✅ Infantry: ${squadInfPct}%, Lancer: ${squadLanPct}%, Marksman: ${squadMarPct}% (Total: ${cap})<br>`;
        }
        window.defaultSquads = [];
      }

      document.getElementById('output').innerHTML = output;
    }
  </script>
</body>
</html>
` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
