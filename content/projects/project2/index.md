---
title: "My running data analytics dashboard" 
tags: ["Strava","Running"]
summary: "A personal data visualization project that integrates the Strava API to track and analyze my running performance. Combining my background in data science with my fitness goals, this dashboard provides insights into my pace, distance, and training consistency. It’s a way to apply the same analytical rigor I use in my research to my personal health and race preparations." 
hidemeta: true
---

<style>
    .dashboard-card { 
        width: 100%;
        background: white; 
        padding: 20px; 
        border-radius: 12px; 
        box-shadow: 0 4px 15px rgba(0,0,0,0.08); 
        margin: 20px 0;
        font-family: sans-serif;
    }
    .dashboard-header { 
        display: flex; 
        justify-content: space-between; 
        align-items: left; 
        margin-bottom: 20px;
        padding-bottom: 10px;
        border-bottom: 1px solid #eee;
    }
    .dashboard-header h1 { margin: 0; font-size: 1.0rem; color: #333; }
    #yearSelect { 
        padding: 5px 10px; 
        border-radius: 5px; 
        border: 1px solid #ccc;
    }
    /* Critical: ECharts needs a defined height to render */
    #calendarHeatmap { width: 100%; height: 280px; }
    #hourlyHeatmap { width: 100%; height: 300px; margin-top: 20px; }

</style>

<div class="dashboard-card">
    <div class="dashboard-header">
        <h1>Running Consistency</h1>
        <div>
            <label for="yearSelect" style="font-size: 0.8rem; font-weight: bold; color: #666;">YEAR:</label>
            <select id="yearSelect"></select>
        </div>
    </div>
    <div id="calendarHeatmap"></div>
    
</div>

<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>

<script>
    (function() {
        let rawData = [];
        let myHeatmap;
        const heatmapDom = document.getElementById('calendarHeatmap');

        function renderHeatmap(year) {
            const heatmapData = rawData
                .filter(d => d.year.toString() === year.toString())
                .map(d => [d.date, d.distance_km]);

            const option = {
                title: {
                    text: 'Distance (km)',
                    left: 'center',
                    top: 0,
                    textStyle: { fontSize: 14, color: '#444' }
                },
                tooltip: {
                    position: 'top',
                    formatter: (p) => `<b>${p.data[0]}</b><br/>${p.data[1]} km`
                },
                visualMap: {
                    min: 0,
                    max: 15,
                    type: 'piecewise',
                    orient: 'horizontal',
                    left: 'center',
                    top: 30,
                    inRange: { color: ['#ebedf0', '#ffbd8b', '#ff8a3d', '#e65100'] }
                },
                calendar: {
                    top: 100,
                    left: 40,
                    right: 10,
                    cellSize: ['auto', 13],
                    range: year,
                    itemStyle: { borderWidth: 0.5, borderColor: '#fff' },
                    yearLabel: { show: false },
                    dayLabel: { firstDay: 1, nameMap: 'en' },
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

        start();
    })();
</script>