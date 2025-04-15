# hw.6
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Simulatore Payoff Opzioni</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #f7f7f7;
      margin: 20px;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    .form-container, .options-table {
      text-align: center;
      margin-bottom: 20px;
    }
    .form-container input, .form-container select {
      margin: 5px;
      padding: 8px;
      font-size: 14px;
    }
    .form-container button {
      padding: 8px 16px;
      background-color: #007bff;
      color: white;
      border: none;
      cursor: pointer;
    }
    .form-container button:hover {
      background-color: #0056b3;
    }
    table {
      margin: 0 auto;
      border-collapse: collapse;
      width: 90%;
    }
    th, td {
      padding: 8px;
      border: 1px solid #ccc;
    }
    canvas {
      display: block;
      margin: 20px auto;
    }
  </style>
</head>
<body>

  <h1>Simulatore Payoff Opzioni (Call)</h1>

  <div class="form-container">
    <input type="number" id="strike" placeholder="Strike">
    <input type="number" id="premio" placeholder="Premio">
    <input type="number" id="quantita" placeholder="Quantità">
    <input type="number" id="scadenza" placeholder="Scadenza (giorni)">
    <select id="posizione">
      <option value="long">Long</option>
      <option value="short">Short</option>
    </select>
    <button onclick="aggiungiOpzione()">Aggiungi Opzione</button>
  </div>

  <div class="options-table">
    <table id="tabellaOpzioni">
      <thead>
        <tr>
          <th>Strike</th>
          <th>Premio</th>
          <th>Quantità</th>
          <th>Scadenza</th>
          <th>Posizione</th>
          <th>Azioni</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <div style="text-align: center;">
    <label for="prezzo">Prezzo Attuale Sottostante:</label>
    <input type="number" id="prezzo" value="100">
  </div>

  <div style="text-align: center; margin-top: 10px;">
    <label for="dataRiferimento">Data di Riferimento:</label>
    <input type="date" id="dataRiferimento" value="2025-04-12">
  </div>

  <div style="text-align: center; margin-top: 10px;">
    <button onclick="aggiornaGrafico()">Aggiorna Grafico</button>
  </div>

  <canvas id="graficoPayoff" width="800" height="400"></canvas>

  <script>
    let opzioni = [];
    let colori = ['green', 'red', 'blue', 'orange', 'purple', 'brown', 'cyan', 'magenta'];

    function aggiungiOpzione() {
      const strike = parseFloat(document.getElementById("strike").value);
      const premio = parseFloat(document.getElementById("premio").value);
      const quantita = parseInt(document.getElementById("quantita").value);
      const scadenza = parseInt(document.getElementById("scadenza").value);
      const posizione = document.getElementById("posizione").value;

      if (isNaN(strike) || isNaN(premio) || isNaN(quantita) || isNaN(scadenza)) {
        alert("Inserisci tutti i valori numerici!");
        return;
      }

      opzioni.push({ strike, premio, quantita, scadenza, tipo: "call", posizione });
      aggiornaTabella();
    }

    function rimuoviOpzione(index) {
      opzioni.splice(index, 1);
      aggiornaTabella();
    }

    function aggiornaTabella() {
      const tbody = document.querySelector("#tabellaOpzioni tbody");
      tbody.innerHTML = "";
      opzioni.forEach((opzione, i) => {
        const row = `<tr>
          <td>${opzione.strike}</td>
          <td>${opzione.premio}</td>
          <td>${opzione.quantita}</td>
          <td>${opzione.scadenza} gg</td>
          <td>${opzione.posizione}</td>
          <td><button onclick="rimuoviOpzione(${i})">Rimuovi</button></td>
        </tr>`;
        tbody.innerHTML += row;
      });
    }

    function calcolaPayoffOpzione(opzione, prezzi) {
      return prezzi.map(p => {
        let payoff = Math.max(0, p - opzione.strike) - opzione.premio;
        if (opzione.posizione === "short") payoff = -payoff;
        return payoff * opzione.quantita;
      });
    }

    function aggiornaGrafico() {
      const prezzoAttuale = parseFloat(document.getElementById("prezzo").value);
      const dataRiferimento = document.getElementById("dataRiferimento").value;

      if (isNaN(prezzoAttuale) || !dataRiferimento) {
        alert("Inserisci un prezzo attuale e una data di riferimento validi.");
        return;
      }

      const prezzoDate = new Date(dataRiferimento);
      const oggi = new Date();
      const giorniRiferimento = Math.floor((prezzoDate - oggi) / (1000 * 60 * 60 * 24));  // Differenza in giorni

      const prezzi = [];
      const min = prezzoAttuale - 50, max = prezzoAttuale + 50, step = 5;
      for (let p = min; p <= max; p += step) prezzi.push(p);

      const datasets = [];
      let payoffTotale = prezzi.map(() => 0);

      opzioni.forEach((opzione, idx) => {
        if (opzione.scadenza >= giorniRiferimento) {
          const payoff = calcolaPayoffOpzione(opzione, prezzi);
          payoffTotale = payoffTotale.map((val, i) => val + payoff[i]);

          datasets.push({
            label: `Call ${opzione.posizione} @${opzione.strike} (${opzione.scadenza}gg)`,
            data: payoff,
            borderColor: colori[idx % colori.length],
            borderWidth: 2,
            fill: false
          });
        }
      });

      datasets.push({
        label: `Payoff Totale (≥ ${giorniRiferimento}gg)`,
        data: payoffTotale,
        borderColor: "black",
        borderWidth: 2,
        borderDash: [5, 5],
        fill: false
      });

      const ctx = document.getElementById("graficoPayoff").getContext("2d");
      if (window.chart) window.chart.destroy();

      window.chart = new Chart(ctx, {
        type: "line",
        data: {
          labels: prezzi,
          datasets: datasets
        },
        options: {
          responsive: true,
          plugins: {
            legend: {
              display: true
            }
          },
          scales: {
            x: {
              title: {
                display: true,
                text: "Prezzo Sottostante"
              }
            },
            y: {
              title: {
                display: true,
                text: "Payoff"
              }
            }
          }
        }
      });
    }
  </script>

</body>
</html>
