---
title: "My running analytics dashboard" 
tags: ["Strava","Running"]
summary: "A personal data visualization project that integrates the Strava API to track and analyze my running performance. Combining my background in data science with my fitness goals, this dashboard provides insights into my pace, distance, and training consistency. It’s a way to apply the same analytical rigor I use in my research to my personal health and race preparations." 
hidemeta: true
---


<style>
    .dashboard-hub { max-width: 1000px; margin: 20px auto; font-family: sans-serif; }
    
    /* Menu Styling */
    .dashboard-nav {
        display: flex;
        gap: 5px;
        margin-bottom: 20px;
        background: #f0f0f0;
        padding: 5px;
        border-radius: 10px;
    }
    .nav-btn {
        flex: 1;
        padding: 12px;
        text-align: center;
        cursor: pointer;
        border-radius: 8px;
        font-weight: bold;
        color: #666;
        transition: 0.3s;
    }
    input[type="radio"] { display: none; }
    
    #tab1:checked ~ .dashboard-nav label[for="tab1"],
    #tab2:checked ~ .dashboard-nav label[for="tab2"] {
        background: #e65100;
        color: white;
    }

    /* Tab Switching Logic */
    .tab-content { display: none; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); }
    #tab1:checked ~ #general-content,
    #tab2:checked ~ #annual-content { display: block; }

    /* General Stats Cards */
    .stats-row { display: flex; gap: 20px; margin-bottom: 20px; }
    .stat-card { flex: 1; background: #fff5ed; padding: 20px; border-radius: 10px; text-align: center; border: 1px solid #ffbd8b; }
    .stat-value { font-size: 2rem; font-weight: bold; color: #e65100; }
    .stat-label { font-size: 0.9rem; color: #666; }

    /* Chart Heights */
    #allTimeChart { width: 100%; height: 350px; }
    #calendarHeatmap { width: 100%; height: 250px; }
    #hourlyHeatmap { width: 100%; height: 400px; }
</style>

<div class="dashboard-hub">
    <input type="radio" name="main-tabs" id="tab1" checked>
    <input type="radio" name="main-tabs" id="tab2">
    <div class="dashboard-nav">
        <label for="tab1" class="nav-btn">📈 General Analysis</label>
        <label for="tab2" class="nav-btn">📅 Annual Analysis</label>
    </div>
    <div id="general-content" class="tab-content">
        <div class="activity-timeline" style="margin-top: 30px; padding: 20px; background: #fff; border-radius: 10px; border-left: 4px solid #e65100; box-shadow: inset 0 0 10px rgba(0,0,0,0.02);">
        <p style="margin: 5px 0; font-size: 1.1rem; color: #444;">
            <strong>First Activity:</strong> <span id="firstDate" style="color: #e65100; font-weight: bold;">-</span>
        </p>
        <p style="margin: 5px 0; font-size: 1.1rem; color: #444;">
            <strong>Last Activity:</strong> <span id="lastDate" style="color: #e65100; font-weight: bold;">-</span>
        </p>
    </div>
    <div class="stats-row">
        <div class="stat-card">
            <div class="stat-value" id="totalKm">0</div>
            <div class="stat-label">Total Kilometers</div>
        </div>
        <div class="stat-card">
            <div class="stat-value" id="totalRuns">0</div>
            <div class="stat-label">Total Activities</div>
        </div>
    </div>
    <h3 style="text-align:center; color:#444; margin-top: 40px;">Distance Frequency Distribution</h3>
    <div id="distFreqChart" style="width: 100%; height: 350px;"></div>
    </div>
    <div id="annual-content" class="tab-content">
        <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
            <h2 style="margin:0; font-size: 1.2rem;">Annual Drill-down</h2>
            <select id="yearSelect" style="padding: 5px 10px; border-radius: 5px;"></select>
        </div>
        <p style="text-align:center; font-weight:bold; color:#666;">Distance Heatmap</p>
        <div id="calendarHeatmap"></div>
        <p style="text-align:center; font-weight:bold; color:#666; margin-top:30px;">Hourly Frequency</p>
        <div id="hourlyHeatmap"></div>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>
<script>
(function() {
    let rawData = [];
    let charts = { general: null, calendar: null, hourly: null };
    async function start() {
        try {
            const response = await fetch('running_data.json');
            rawData = await response.json();
            // Init General Chart
            charts.distFreq = echarts.init(document.getElementById('distFreqChart'));
            // Init Annual Charts
            charts.calendar = echarts.init(document.getElementById('calendarHeatmap'));
            charts.hourly = echarts.init(document.getElementById('hourlyHeatmap'));
            // Setup Year Select
            const yearSelect = document.getElementById('yearSelect');
            const years = [...new Set(rawData.map(d => d.year))].sort((a, b) => b - a);
            years.forEach(y => {
                let opt = document.createElement('option'); opt.value = y; opt.text = y;
                yearSelect.appendChild(opt);
            });
            // Initial Renders
            renderGeneral();
            renderAnnual(years[0]);
            // Events
            yearSelect.addEventListener('change', (e) => renderAnnual(e.target.value));
            // Critical: Resize when switching tabs
            document.querySelectorAll('.nav-btn').forEach(btn => {
                btn.addEventListener('click', () => {
                    setTimeout(() => { 
                        Object.values(charts).forEach(c => c && c.resize());
                    }, 50);
                });
            });
            window.addEventListener('resize', () => Object.values(charts).forEach(c => c && c.resize()));
        } catch (e) { console.error(e); }
    }
    function renderGeneral() {
        // Calculate Totals
        const totalKm = rawData.reduce((sum, r) => sum + r.distance_km, 0);
        document.getElementById('totalKm').innerText = totalKm.toFixed(1);
        document.getElementById('totalRuns').innerText = rawData.length;
        // 2. Find First and Last Activities
        if (rawData.length > 0) {
            // Extract all dates and sort them
            const dates = rawData.map(r => r.date).sort();     
            const first = dates[0];
            const last = dates[dates.length - 1];
            // Format function (transforms YYYY-MM-DD to DD/MM/YYYY for better reading)
            const formatDate = (dateStr) => {
                // Extract day (before "T")
                const day = dateStr.split('T')[0];
                // Extract month and year (after "/")
                const [, month, year] = dateStr.match(/\/(\d{2})\/(\d{4})/);
                return date.toLocaleDateString('en-US', {
                    day: 'numeric',
                    month: 'long',
                    year: 'numeric'
                });
            };
            document.getElementById('firstDate').innerText = formatDate(first);
            document.getElementById('lastDate').innerText = formatDate(last);
            // 2. SCATTERPLOT LOGIC: Frequency vs Distance
            const freqMap = {};
            rawData.forEach(r => {
                // Round to 1 decimal place to group similar runs (e.g., 5.0km)
                const d = parseFloat(r.distance_km).toFixed(1);
                freqMap[d] = (freqMap[d] || 0) + 1;
            });
            // Convert Map to [x, y] coordinates: [Number of Activities, Distance]
            const scatterData = Object.keys(freqMap).map(dist => [
            freqMap[dist],       // x: Number of activities
            parseFloat(dist)     // y: Distance in km
            ]);
            const scatterOption = {
                tooltip: {
                    formatter: (p) => `Distance: <b>${p.data[1]} km</b><br/>Frequency: <b>${p.data[0]} times</b>`
                },
                grid: { left: '10%', right: '10%', bottom: '15%', top: '10%' },
                xAxis: { 
                    name: 'Activities (Count)', 
                    nameLocation: 'middle', 
                    nameGap: 30,
                    type: 'value',
                    splitLine: { lineStyle: { type: 'dashed' } }
                },
                yAxis: { 
                    name: 'Distance (km)', 
                    type: 'value',
                    splitLine: { lineStyle: { type: 'dashed' } }
                },
                series: [{
                    symbolSize: 15,
                    data: scatterData,
                    type: 'scatter',
                    itemStyle: {
                        color: '#e65100',
                        opacity: 0.6,
                        borderColor: '#fff',
                        borderWidth: 1
                    },
                    emphasis: {
                        itemStyle: { opacity: 1, shadowBlur: 10, shadowColor: 'rgba(0,0,0,0.3)' }
                    }
                }]
            };
            charts.distFreq.setOption(scatterOption);
        }
    }
    function renderAnnual(year) {
        const filteredData = rawData.filter(d => d.year.toString() === year.toString());
        // --- Calendar Heatmap ---
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
        charts.calendar.setOption(calOption);
        // --- Hourly Heatmap (The Punch Card) ---
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
        charts.hourly.setOption(hourlyOption);
    }
    start();
})();
</script>