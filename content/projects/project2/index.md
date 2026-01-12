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
        <h3 style="text-align:center; color:#444;">Monthly Volume Over Time</h3>
        <div id="allTimeChart"></div>
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
            charts.general = echarts.init(document.getElementById('allTimeChart'));
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
        // Group data by Month-Year for the bar chart
        // (Assuming your data has "date" as YYYY-MM-DD)
        const monthlyData = {};
        rawData.forEach(r => {
            const monthKey = r.date.substring(0, 7); // "YYYY-MM"
            monthlyData[monthKey] = (monthlyData[monthKey] || 0) + r.distance_km;
        });
        const sortedMonths = Object.keys(monthlyData).sort();
        charts.general.setOption({
            tooltip: { trigger: 'axis' },
            xAxis: { type: 'category', data: sortedMonths },
            yAxis: { type: 'value', name: 'km' },
            series: [{
                data: sortedMonths.map(m => monthlyData[m]),
                type: 'bar',
                itemStyle: { color: '#ff8a3d' }
            }]
        });
    }
    function renderAnnual(year) {
        const filtered = rawData.filter(d => d.year.toString() === year.toString());
        // --- Calendar Heatmap ---
        charts.calendar.setOption({
            visualMap: { min: 0, max: 15, show: false, inRange: { color: ['#ebedf0', '#e65100'] } },
            calendar: { range: year, cellSize: ['auto', 12], left: 40, right: 10, top: 40, dayLabel: {firstDay: 1} },
            series: { type: 'heatmap', coordinateSystem: 'calendar', data: filtered.map(d => [d.date, d.distance_km]) }
        });
        // --- Hourly Heatmap (The Punch Card) ---
        let punchData = [];
        for (let d = 0; d < 7; d++) for (let h = 0; h < 24; h++) punchData.push([d, h, 0]);
        filtered.forEach(r => {
            const idx = (r.weekday_num * 24) + r.hour;
            if(punchData[idx]) punchData[idx][2]++;
        });
        charts.hourly.setOption({
            xAxis: { type: 'category', data: ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'] },
            yAxis: { type: 'category', data: Array.from({length: 24}, (_, i) => i + ':00'), inverse: true },
            visualMap: { show: false, min: 0, max: 5, inRange: { color: ['#ebedf0', '#ff8a3d', '#e65100'] } },
            grid: { top: '5%', bottom: '10%' },
            series: [{ type: 'heatmap', data: punchData }]
        });
    }
    start();
})();
</script>