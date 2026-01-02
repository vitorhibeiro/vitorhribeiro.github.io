---
title: "My running data analytics dashboard" 
tags: ["Strava","Running"]
summary: "A personal data visualization project that integrates the Strava API to track and analyze my running performance. Combining my background in data science with my fitness goals, this dashboard provides insights into my pace, distance, and training consistency. It’s a way to apply the same analytical rigor I use in my research to my personal health and race preparations." 
hidemeta: true
---

<div id="calendarHeatmap" style="width: 100%; height: 250px; margin-top: 30px;"></div>

<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>

<script>
let chartDom = document.getElementById('calendarHeatmap');
let myHeatmap = echarts.init(chartDom);
let allActivities = []; // Seus dados do JSON

async function loadHeatmap() {
    const response = await fetch('running_data.json');
    allActivities = await response.json();
    
    // Pegar o ano atual ou o primeiro do seletor
    const initialYear = new Date().getFullYear().toString();
    renderHeatmap(initialYear);
}

function renderHeatmap(year) {
    // 1. Filtrar e formatar dados para o ECharts: [[data, valor], ...]
    const heatmapData = allActivities
        .filter(d => d.year.toString() === year)
        .map(d => [d.date, d.distance_km]);

    const option = {
        title: {
            top: 0,
            left: 'center',
            text: `Running Distance (km) - ${year}`
        },
        tooltip: {
            formatter: function (p) {
                return `${p.data[0]}: ${p.data[1]} km`;
            }
        },
        visualMap: {
            min: 0,
            max: 15, // Ajuste baseado no seu volume máximo (ex: 15km)
            type: 'piecewise',
            orient: 'horizontal',
            left: 'center',
            top: 40,
            // Cores que lembram o Strava (tons de laranja/vermelho)
            inRange: {
                color: ['#eeeeee', '#ffbd8b', '#ff8a3d', '#e65100']
            }
        },
        calendar: {
            top: 90,
            left: 30,
            right: 30,
            cellSize: ['auto', 13],
            range: year,
            itemStyle: {
                borderWidth: 0.5
            },
            yearLabel: { show: false },
            dayLabel: { nameMap: 'en' },
            monthLabel: { nameMap: 'en' }
        },
        series: {
            type: 'heatmap',
            coordinateSystem: 'calendar',
            data: heatmapData
        }
    };

    myHeatmap.setOption(option);
}

// Integrando com o seu select de ano existente
document.getElementById('yearSelect').addEventListener('change', (e) => {
    if(e.target.value !== 'all') {
        renderHeatmap(e.target.value);
    }
});

// Responsividade
window.addEventListener('resize', () => myHeatmap.resize());

loadHeatmap();
</script>