---
title: "My running data analytics dashboard" 
tags: ["Strava","Running"]
summary: "A personal data visualization project that integrates the Strava API to track and analyze my running performance. Combining my background in data science with my fitness goals, this dashboard provides insights into my pace, distance, and training consistency. It’s a way to apply the same analytical rigor I use in my research to my personal health and race preparations." 
hidemeta: true
---

<div id="calendarHeatmap" style="width: 100%; height: 100px; margin-top: 30px;"></div>

<script src="https://cdn.jsdelivr.net/npm/echarts/dist/echarts.min.js"></script>

    <style>
        body { 
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
            padding: 40px; 
            background-color: #f4f7f6; 
            display: flex;
            justify-content: center;
        }
        .dashboard-card { 
            width: 100%;
            max-width: 1000px; 
            background: white; 
            padding: 30px; 
            border-radius: 15px; 
            box-shadow: 0 10px 25px rgba(0,0,0,0.05); 
        }
        .header { 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            margin-bottom: 30px;
            border-bottom: 2px solid #f0f0f0;
            padding-bottom: 15px;
        }
        h1 { margin: 0; font-size: 1.5rem; color: #333; }
        select { 
            padding: 8px 15px; 
            border-radius: 8px; 
            border: 1px solid #ddd; 
            background-color: #fff;
            font-size: 0.9rem;
            cursor: pointer;
            outline: none;
        }
        select:hover { border-color: #e65100; }
        #calendarHeatmap { width: 100%; height: 300px; }
    </style>
</head>
<body>

<div class="dashboard-card">
    <div class="header">
        <h1>🏃 Running Consistency</h1>
        <div>
            <label for="yearSelect" style="font-size: 0.8rem; font-weight: bold; color: #666;">YEAR:</label>
            <select id="yearSelect"></select>
        </div>
    </div>

    <div id="calendarHeatmap"></div>
</div>

<script>
    let rawData = [];
    let myHeatmap;

    // Inicializa o gráfico
    const heatmapDom = document.getElementById('calendarHeatmap');
    myHeatmap = echarts.init(heatmapDom);

    async function start() {
        try {
            // Busca o arquivo JSON (deve estar na mesma pasta)
            const response = await fetch('running_data.json');
            if (!response.ok) throw new Error("Não foi possível carregar o JSON.");
            
            rawData = await response.json();

            // 1. Configura o Seletor de Anos
            const yearSelect = document.getElementById('yearSelect');
            const years = [...new Set(rawData.map(d => d.year))].sort((a, b) => b - a);
            
            years.forEach(year => {
                const opt = document.createElement('option');
                opt.value = year;
                opt.text = year;
                yearSelect.appendChild(opt);
            });

            // 2. Renderiza o ano mais recente por padrão
            renderHeatmap(years[0]);

            // 3. Listener para mudança de ano
            yearSelect.addEventListener('change', (e) => {
                renderHeatmap(e.target.value);
            });

        } catch (err) {
            console.error(err);
            heatmapDom.innerHTML = `<p style="color:red">Erro: Certifique-se de que 'running_data.json' está na mesma pasta e que o Live Server está ativo.</p>`;
        }
    }

    function renderHeatmap(year) {
    const heatmapData = rawData
        .filter(d => d.year.toString() === year.toString())
        .map(d => [d.date, d.distance_km]);

    const option = {
        // Usamos o título do gráfico para legendar a escala de cores
        title: {
            text: 'Distance in kilometers',
            left: 'center',
            top: 0,
            textStyle: {
                fontSize: 14,
                fontWeight: 'bold',
                color: '#444'
            }
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
            top: 30, // Posicionado logo abaixo do título interno
            itemSymbol: 'rect', // Garante que sejam quadrados
            inRange: {
                color: ['#ebedf0', '#ffbd8b', '#ff8a3d', '#e65100']
            }
        },
        calendar: {
            top: 100, // Espaço aumentado para acomodar título + legenda
            left: 30,
            right: 30,
            cellSize: ['auto', 13],
            range: year,
            itemStyle: {
                borderWidth: 0.5,
                borderColor: '#fff'
            },
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
    // Responsividade
    window.onresize = () => myHeatmap.resize();

    start();
</script>