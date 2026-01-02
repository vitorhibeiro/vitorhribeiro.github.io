---
title: "My running data analytics dashboard" 
tags: ["Strava","Running"]
summary: "A personal data visualization project that integrates the Strava API to track and analyze my running performance. Combining my background in data science with my fitness goals, this dashboard provides insights into my pace, distance, and training consistency. It’s a way to apply the same analytical rigor I use in my research to my personal health and race preparations." 
hidemeta: true
---

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<div style="position: relative; height:40vh; width:100%">
  <canvas id="runningChart"></canvas>
</div>

<script>
async function buildDashboard() {
    // 1. Buscar os dados que você exportou
    const response = await fetch('running_data.json');
    const data = await response.json();

    // 2. Processar os dados (ex: pegar os últimos 10 treinos)
    const lastTen = data.slice(-10);
    const labels = lastTen.map(d => new Date(d.date).toLocaleDateString());
    const distances = lastTen.map(d => d.distance_km); // Ajuste para o nome da sua coluna
    const paces = lastTen.map(d => d.pace_min_km);

    // 3. Criar o gráfico
    const ctx = document.getElementById('runningChart').getContext('2d');
    new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Distância (km)',
                data: distances,
                borderColor: '#2ecc71',
                backgroundColor: 'rgba(46, 204, 113, 0.2)',
                fill: true,
                tension: 0.3
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: { beginAtZero: true }
            }
        }
    });
}

buildDashboard();
</script>