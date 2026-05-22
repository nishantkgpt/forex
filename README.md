<!DOCTYPE html>
<html lang="en">

<head>

  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>Live Forex Dashboard</title>

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <style>

    body{
      margin:0;
      font-family:Arial, sans-serif;
      background:#0f172a;
      color:white;
    }

    header{
      background:#111827;
      padding:20px;
      text-align:center;
      box-shadow:0 2px 10px rgba(0,0,0,0.4);
    }

    h1{
      margin:0;
      font-size:34px;
    }

    .subtitle{
      margin-top:8px;
      color:#94a3b8;
    }

    .grid{
      display:grid;
      grid-template-columns:repeat(auto-fit,minmax(220px,1fr));
      gap:20px;
      padding:20px;
    }

    .card{
      background:#1e293b;
      padding:20px;
      border-radius:16px;
      transition:0.3s;
      box-shadow:0 0 10px rgba(0,0,0,0.3);
    }

    .card:hover{
      transform:translateY(-5px);
    }

    .currency{
      font-size:22px;
      font-weight:bold;
    }

    .rate{
      margin-top:10px;
      font-size:30px;
      color:#38bdf8;
    }

    .chart-box{
      background:#1e293b;
      margin:20px;
      padding:20px;
      border-radius:16px;
    }

    select{
      padding:10px;
      border:none;
      border-radius:8px;
      margin-bottom:20px;
      font-size:16px;
    }

    canvas{
      background:white;
      border-radius:12px;
      padding:10px;
    }

    footer{
      text-align:center;
      padding:20px;
      color:#94a3b8;
    }

    .loading{
      text-align:center;
      padding:20px;
      color:#fbbf24;
    }

  </style>

</head>

<body>

<header>

  <h1>Live Global FX Dashboard</h1>

  <div class="subtitle">
    Top Economies Currency Value vs USD
  </div>

</header>

<div id="loading" class="loading">
  Fetching live forex data...
</div>

<div class="grid" id="cards"></div>

<div class="chart-box">

  <h2>1-Year Currency Trend</h2>

  <select id="currencySelect">

    <option value="INR">Indian Rupee (INR)</option>
    <option value="CNY">Chinese Yuan (CNY)</option>
    <option value="JPY">Japanese Yen (JPY)</option>
    <option value="EUR">Euro (EUR)</option>
    <option value="GBP">British Pound (GBP)</option>
    <option value="BRL">Brazilian Real (BRL)</option>
    <option value="CAD">Canadian Dollar (CAD)</option>

  </select>

  <canvas id="chart"></canvas>

</div>

<footer>
  Live Forex Dashboard using Alpha Vantage API
</footer>

<script>

const API_KEY = "NQY281BTK9QQRWMV";

const currencies = [
  "INR",
  "CNY",
  "JPY",
  "EUR",
  "GBP",
  "BRL",
  "CAD"
];

const cards = document.getElementById("cards");

const loading =
  document.getElementById("loading");

let chart;

// LOAD LIVE FOREX CARDS
async function loadCards(){

  cards.innerHTML = "";

  for(const currency of currencies){

    try{

      const url =
      `https://www.alphavantage.co/query?function=CURRENCY_EXCHANGE_RATE&from_currency=USD&to_currency=${currency}&apikey=${API_KEY}`;

      const response = await fetch(url);

      const data = await response.json();

      const exchangeData =
        data["Realtime Currency Exchange Rate"];

      if(!exchangeData){

        console.log(data);

        continue;
      }

      const rate =
        exchangeData["5. Exchange Rate"];

      const card =
        document.createElement("div");

      card.className = "card";

      card.innerHTML = `

        <div class="currency">
          USD / ${currency}
        </div>

        <div class="rate">
          ${parseFloat(rate).toFixed(2)}
        </div>

      `;

      cards.appendChild(card);

    }

    catch(error){

      console.error(error);

    }

  }

  loading.style.display = "none";

}

// LOAD HISTORICAL TREND
async function loadChart(currency){

  try{

    const url =
    `https://www.alphavantage.co/query?function=FX_DAILY&from_symbol=USD&to_symbol=${currency}&outputsize=full&apikey=${API_KEY}`;

    const response = await fetch(url);

    const data = await response.json();

    const series =
      data["Time Series FX (Daily)"];

    if(!series){

      console.log(data);

      alert("Trend data unavailable.");

      return;
    }

    const labels = [];

    const values = [];

    const dates =
      Object.keys(series)
      .slice(0,365)
      .reverse();

    dates.forEach(date => {

      labels.push(date);

      values.push(
        parseFloat(series[date]["4. close"])
      );

    });

    const ctx =
      document
      .getElementById("chart")
      .getContext("2d");

    if(chart){

      chart.destroy();

    }

    chart = new Chart(ctx, {

      type:"line",

      data:{

        labels:labels,

        datasets:[{

          label:`USD/${currency}`,

          data:values,

          borderWidth:2,

          tension:0.3

        }]

      },

      options:{

        responsive:true,

        plugins:{

          legend:{
            display:true
          }

        },

        scales:{

          y:{
            beginAtZero:false
          }

        }

      }

    });

  }

  catch(error){

    console.error(error);

  }

}

// DROPDOWN EVENT
document
.getElementById("currencySelect")
.addEventListener("change",(e)=>{

  loadChart(e.target.value);

});

// INITIAL LOAD
loadCards();

loadChart("INR");

</script>

</body>

</html>
