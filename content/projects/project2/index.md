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
    /* Chart Containers */
    #calendarHeatmap { width: 100%; height: 280px; margin-bottom: 30px; }
    #hourlyHeatmap { width: 100%; height: 450px; }
    
    .chart-title {
        text-align: center;
        font-size: 1rem;
        font-weight: bold;
        color: #444;
        margin-bottom: 10px;
    }

</style>

<div class="dashboard-card">
    <div class="dashboard-header">
        <h1>Strava data</h1>
        <div>
            <label for="yearSelect" style="font-size: 0.8rem; font-weight: bold; color: #666;">YEAR:</label>
            <select id="yearSelect"></select>
        </div>
    </div>
    <div class="chart-title">Annual Consistency</div>
    <div id="calendarHeatmap"></div>
    <div style="margin-top: 10px;" class="chart-title">Weekly Routine (Frequency by Hour)</div>
    <div id="hourlyHeatmap"></div>

</div>

<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>

<script>
    (function() {
        let rawData = [];
        let myHeatmap;
        let hourlyChart;

        async function start() {
            try {
                // Hugo: ensure running_data.json is in your /static/ folder
                const response = await fetch('running_data.json');
                if (!response.ok) throw new Error("Could not load running_data.json");
                
                rawData = await response.json();

                // Initialize Charts
                calendarChart = echarts.init(document.getElementById('calendarHeatmap'));
                hourlyChart = echarts.init(document.getElementById('hourlyHeatmap'));

                // Populate Year Selector
                const yearSelect = document.getElementById('yearSelect');
                const years = [...new Set(rawData.map(d => d.year))].sort((a, b) => b - a);
                
                years.forEach(year => {
                    const opt = document.createElement('option');
                    opt.value = year;
                    opt.text = year;
                    yearSelect.appendChild(opt);
                });

                // Initial Render
                renderDashboards(years[0]);

                // Listen for changes
                yearSelect.addEventListener('change', (e) => renderDashboards(e.target.value));
                
                // Responsiveness
                window.addEventListener('resize', () => {
                    calendarChart.resize();
                    hourlyChart.resize();
                });

            } catch (err) {
                console.error(err);
                document.getElementById('calendarHeatmap').innerHTML = `<p style="color:red">Error loading data. Check console.</p>`;
            }
        }

        function renderDashboards(year) {
            const filteredData = rawData.filter(d => d.year.toString() === year.toString());

            // --- 1. CALENDAR HEATMAP CONFIG ---
            const calData = filteredData.map(d => [d.date, d.distance_km]);
            const calOption = {
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
                    data: calData
                }
            };

            // --- 2. HOURLY PUNCH CARD CONFIG ---
            const days = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];
            const hours = Array.from({length: 24}, (_, i) => i + ':00');
            
            // Initialize 7x24 grid
            let punchData = [];
            for (let d = 0; d < 7; d++) {
                for (let h = 0; h < 24; h++) {
                    punchData.push([d, h, 0]);
                }
            }

            // Fill grid
            filteredData.forEach(run => {
                const index = run.weekday_num * 24 + run.hour;
                if (punchData[index]) {
                    punchData[index][2]++;
                }
            });

            const hourlyOption = {
                tooltip: {
                    position: 'top',
                    formatter: (p) => `${days[p.data[0]]} @ ${p.data[1]}:00 <br/><b>${p.data[2]} runs</b>`
                },
                grid: { height: '75%', top: '5%', right: '5%' },
                xAxis: { type: 'category', data: days, splitArea: { show: true } },
                yAxis: { type: 'category', data: hours, inverse: true, splitArea: { show: true } },
                visualMap: {
                    show: false, // This hides the color bar
                    min: 0,
                    max: 5,
                    calculable: true,
                    orient: 'horizontal',
                    left: 'center',
                    bottom: 0,
                    inRange: { color: ['#ebedf0', '#ffbd8b', '#e65100'] }
                },
                series: [{
                    name: 'Frequency',
                    type: 'heatmap',
                    data: punchData,
                    label: { show: false },
                    emphasis: {
                        itemStyle: { shadowBlur: 10, shadowColor: 'rgba(0, 0, 0, 0.5)' }
                    }
                }]
            };

            // Apply options
            calendarChart.setOption(calOption);
            hourlyChart.setOption(hourlyOption);
        }

        start();
    })();
</script>