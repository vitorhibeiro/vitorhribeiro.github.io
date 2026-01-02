---
title: "My running data analytics dashboard" 
tags: ["Strava","Running"]
summary: "A personal data visualization project that integrates the Strava API to track and analyze my running performance. Combining my background in data science with my fitness goals, this dashboard provides insights into my pace, distance, and training consistency. It’s a way to apply the same analytical rigor I use in my research to my personal health and race preparations." 
hidemeta: true
---

<div style="margin-bottom: 20px; font-family: sans-serif;">
    <label for="yearSelect" style="font-weight: bold; margin-right: 10px;">Select Year:</label>
    <select id="yearSelect" style="padding: 5px 10px; border-radius: 5px; border: 1px solid #ccc;">
        <option value="all">All Years</option>
    </select>
</div>

<div style="position: relative; height:40vh; width:100%">
  <canvas id="runningChart"></canvas>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
let runningData = []; // Variável global para armazenar os dados brutos
let myChart;          // Variável para a instância do gráfico

async function initDashboard() {
    const response = await fetch('/data/running_data.json');
    runningData = await response.json();

    // A. Popular o seletor de anos dinamicamente
    const yearSelect = document.getElementById('yearSelect');
    const years = [...new Set(runningData.map(d => d.year))].sort((a, b) => b - a);

    years.forEach(year => {
        let option = document.createElement('option');
        option.value = year;
        option.text = year;
        yearSelect.appendChild(option);
    });

    // B. Escutar mudanças no seletor
    yearSelect.addEventListener('change', (e) => {
        updateChart(e.target.value);
    });

    // C. Renderizar gráfico inicial (ex: ano mais recente ou todos)
    updateChart('all');
}

function updateChart(selectedYear) {
    // 1. Filtrar os dados
    const filteredData = selectedYear === 'all' 
        ? runningData 
        : runningData.filter(d => d.year.toString() === selectedYear.toString());

    // 2. Preparar labels e valores (ex: Distância por data)
    // Ordenar por data para garantir que o gráfico de linha faça sentido
    filteredData.sort((a, b) => new Date(a.date) - new Date(b.date));
    
    const labels = filteredData.map(d => new Date(d.date).toLocaleDateString());
    const distances = filteredData.map(d => d.distance_km);

    // 3. Criar ou Atualizar o Gráfico
    const ctx = document.getElementById('runningChart').getContext('2d');

    if (myChart) {
        // Se o gráfico já existe, apenas atualizamos os dados
        myChart.data.labels = labels;
        myChart.data.datasets[0].data = distances;
        myChart.update();
    } else {
        // Se é a primeira vez, criamos a instância
        myChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [{
                    label: 'Distance (km)',
                    data: distances,
                    borderColor: '#e6550d',
                    backgroundColor: '#fee6ce',
                    fill: true,
                    tension: 0.3,
                    pointRadius: 4
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: { display: true }
                }
            }
        });
    }
}

initDashboard();
</script>