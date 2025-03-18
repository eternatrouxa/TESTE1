# TESTE1
<!DOCTYPE html>
<html lang="pt-BR">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Calculadora de Ciclos Termodinâmicos</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #f4f6f9;
      color: #333;
      padding: 20px;
    }

    .container {
      max-width: 900px;
      margin: auto;
      background-color: #fff;
      padding: 30px;
      border-radius: 15px;
      box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
    }

    h1 {
      text-align: center;
      color: #4a90e2;
      margin-bottom: 30px;
    }

    label {
      display: block;
      margin-bottom: 8px;
      font-weight: bold;
    }

    select,
    input {
      width: 100%;
      padding: 12px;
      margin-bottom: 20px;
      border: 1px solid #ccc;
      border-radius: 8px;
      background-color: #f9f9f9;
    }

    button {
      background-color: #4a90e2;
      color: #fff;
      padding: 14px 28px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      width: 100%;
      font-size: 16px;
      transition: background-color 0.3s ease;
    }

    button:hover {
      background-color: #357ab8;
    }

    .result {
      margin-top: 30px;
      padding: 20px;
      background-color: #e8f7e8;
      border-left: 5px solid #4caf50;
      border-radius: 10px;
    }

    canvas {
      margin-top: 30px;
      width: 100%;
      max-width: 700px;
    }

    .equations {
      background-color: #f7f7f7;
      padding: 15px;
      border-radius: 8px;
      font-family: monospace;
      white-space: pre-wrap;
    }

    .error {
      color: #d9534f;
      font-weight: bold;
      margin-bottom: 15px;
    }
  </style>
</head>

<body>
  <div class="container">
    <h1>Calculadora de Ciclos Termodinâmicos</h1>

    <div id="error" class="error"></div>

    <label for="fluido">Escolha o fluído:</label>
    <select id="fluido">
      <option value="Amonia">Amônia</option>
      <option value="Vapor">Vapor d'água</option>
      <option value="R134">R134</option>
      <option value="CO2">CO2</option>
    </select>

    <label for="Ti">Temperatura Inicial (°C):</label>
    <input type="number" id="Ti" placeholder="Ex: 300">

    <label for="Tf">Temperatura Final (°C):</label>
    <input type="number" id="Tf" placeholder="Ex: 500">

    <label for="Pi">Pressão Inicial (Pa):</label>
    <input type="number" id="Pi" placeholder="Ex: 100000">

    <label for="Pf">Pressão Final (Pa):</label>
    <input type="number" id="Pf" placeholder="Ex: 500000">

    <button id="btnCalcular">Calcular</button>

    <div class="result" id="result" style="display: none;">
      <h3>Resultados:</h3>
      <div class="equations" id="equacoes"></div>
      <p id="rendimento"></p>
      <p id="trabalho"></p>
      <p id="calor"></p>
      <canvas id="grafico"></canvas>
    </div>
  </div>

  <script>
    const fluidos = {
      "Amonia": { cp: 2.09, Ti: 300, Tf: 500, Pi: 100000, Pf: 500000 },
      "Vapor": { cp: 1.86, Ti: 350, Tf: 600, Pi: 150000, Pf: 600000 },
      "R134": { cp: 0.85, Ti: 280, Tf: 450, Pi: 120000, Pf: 480000 },
      "CO2": { cp: 0.84, Ti: 250, Tf: 400, Pi: 110000, Pf: 450000 }
    };

    let chartInstance = null;

    document.getElementById('btnCalcular').addEventListener('click', calcular);

    function calcular() {
      const fluidoSelecionado = document.getElementById("fluido").value;
      const TiInput = parseFloat(document.getElementById("Ti").value);
      const TfInput = parseFloat(document.getElementById("Tf").value);
      const PiInput = parseFloat(document.getElementById("Pi").value);
      const PfInput = parseFloat(document.getElementById("Pf").value);
      const errorDiv = document.getElementById("error");

      let { cp, Ti, Tf, Pi, Pf } = fluidos[fluidoSelecionado];

      // Substitui valores se o usuário preencheu
      if (!isNaN(TiInput)) Ti = TiInput;
      if (!isNaN(TfInput)) Tf = TfInput;
      if (!isNaN(PiInput)) Pi = PiInput;
      if (!isNaN(PfInput)) Pf = PfInput;

      // Validação básica
      if (Tf <= Ti) {
        errorDiv.textContent = "A Temperatura Final deve ser maior que a Inicial!";
        document.getElementById("result").style.display = "none";
        return;
      }

      errorDiv.textContent = ""; // Limpa erro anterior

      const rendimento = 1 - (Ti / Tf);
      const trabalho = cp * (Tf - Ti);
      const calor = trabalho * rendimento;

      document.getElementById("equacoes").innerText =
        `Equações utilizadas:\n` +
        `Rendimento = 1 - (Ti / Tf)\n` +
        `Trabalho = cp * (Tf - Ti)\n` +
        `Calor = Trabalho * Rendimento`;

      document.getElementById("rendimento").innerHTML = `<strong>Rendimento:</strong> ${(rendimento * 100).toFixed(2)}%`;
      document.getElementById("trabalho").innerHTML = `<strong>Trabalho:</strong> ${trabalho.toFixed(2)} J`;
      document.getElementById("calor").innerHTML = `<strong>Calor:</strong> ${calor.toFixed(2)} J`;

      document.getElementById("result").style.display = "block";

      desenharGrafico(["Início", "Meio", "Fim"], [0, rendimento * 50, rendimento * 100]);
    }

    function desenharGrafico(labels, data) {
      const ctx = document.getElementById("grafico").getContext("2d");

      if (chartInstance) {
        chartInstance.destroy();
      }

      chartInstance = new Chart(ctx, {
        type: 'line',
        data: {
          labels: labels,
          datasets: [{
            label: 'Evolução do Rendimento (%)',
            data: data,
            borderColor: '#4a90e2',
            backgroundColor: 'rgba(74, 144, 226, 0.2)',
            borderWidth: 2,
            fill: true,
            tension: 0.3
          }]
        },
        options: {
          responsive: true,
          plugins: {
            legend: { display: true },
            tooltip: { mode: 'index', intersect: false }
          },
          scales: {
            y: {
              beginAtZero: true,
              ticks: { callback: value => value + '%' }
            }
          }
        }
      });
    }
  </script>
</body>

</html>
