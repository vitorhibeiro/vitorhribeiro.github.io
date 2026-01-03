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
    <div id="hourlyHeatmap"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>

<script>
    (function() {
        let rawData = [];
        let myHeatmap;
        let myHourlyChart;
        const heatmapDom = document.getElementById('calendarHeatmap');

        async function start() {
            try {
                // Hugo note: ensure running_data.json is in your /static/ folder
                const response = await fetch('running_data.json');
                if (!response.ok) throw new Error("Could not load running_data.json");
                
                rawData = await response.json();
                myHeatmap = echarts.init(heatmapDom);
                myHourlyChart = echarts.init(document.getElementById('hourlyHeatmap'));

                const yearSelect = document.getElementById('yearSelect');
                const years = [...new Set(rawData.map(d => d.year))].sort((a, b) => b - a);
                
                years.forEach(year => {
                    const opt = document.createElement('option');
                    opt.value = year;
                    opt.text = year;
                    yearSelect.appendChild(opt);
                });

                renderHeatmap(years[0]);

                yearSelect.addEventListener('change', (e) => {
                    renderHeatmap(e.target.value);
                });

                window.addEventListener('resize', () => {
                                                            myHeatmap.resize();
                                                            myHourlyChart.resize();
                                                        });

            } catch (err) {
                console.error(err);
                heatmapDom.innerHTML = `<p style="color:red; padding: 20px;">Error: Ensure 'running_data.json' is in your static folder and the site is running on a server.</p>`;
            }
        }

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

            // --- Hourly Heatmap Logic ---
            const days = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
            const hours = Array.from({length: 24}, (_, i) => i + ':00');

            // Initialize a 7x24 grid with zeros
            let punchData = [];
            for (let d = 0; d < 7; d++) {
                for (let h = 0; h < 24; h++) {
                    punchData.push([d, h, 0]);
                }
            }

            // Fill grid with counts from your data
            rawData.filter(d => d.year.toString() === year.toString()).forEach(run => {
                // This assumes your JSON has 'day_of_week' (0-6) and 'hour' (0-23)
                const index = run.day_of_week * 24 + run.hour;
                if (punchData[index]) punchData[index][2]++;
            });

            const hourlyOption = {
                title: { text: 'Most Frequent Running Hours', left: 'center', textStyle: { fontSize: 14, color: '#444' } },
                tooltip: { position: 'top' },
                grid: { height: '70%', top: '15%' },
                xAxis: { type: 'category', data: days, splitArea: { show: true } },
                yAxis: { type: 'category', data: hours, splitArea: { show: true } },
                visualMap: {
                    min: 0,
                    max: 10, // Adjust based on your frequency
                    calculable: true,
                    orient: 'horizontal',
                    left: 'center',
                    bottom: '0%',
                    inRange: { color: ['#ebedf0', '#ffbd8b', '#e65100'] }
                },
                series: [{
                    name: 'Runs',
                    type: 'heatmap',
                    data: punchData,
                    label: { show: false },
                    emphasis: { itemStyle: { shadowBlur: 10, shadowColor: 'rgba(0, 0, 0, 0.5)' } }
                }]
            };

            myHourlyChart.setOption(hourlyOption);
        }

        start();
    })();
</script>