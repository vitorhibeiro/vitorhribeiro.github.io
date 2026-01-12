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