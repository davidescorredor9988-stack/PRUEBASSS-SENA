<!-- ... existing code ... -->
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SIMULADOR DE SEGURIDAD INDUSTRIAL INTELIGENTE – YOLOv8n + IoT + RED</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons CDN -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        industrial: {
                            50: '#f8fafc',
                            100: '#f1f5f9',
                            200: '#e2e8f0',
                            700: '#15803d',
                            800: '#0f5132',
                            900: '#064e3b',
                        },
                        accent: {
                            orange: '#ea580c',
                            orangeBright: '#f97316',
                            red: '#dc2626',
                            green: '#16a34a'
                        }
                    }
                }
            }
        }
    </script>
    
    <style>
        body {
            font-family: 'Inter', system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: #f3f4f6;
            color: #1f2937;
        }

        .pulse-green {
            box-shadow: 0 0 0 0 rgba(22, 163, 74, 0.7);
            animation: pulse-green-anim 2s infinite;
        }
        @keyframes pulse-green-anim {
            0% { box-shadow: 0 0 0 0 rgba(22, 163, 74, 0.7); }
            70% { box-shadow: 0 0 0 8px rgba(22, 163, 74, 0); }
            100% { box-shadow: 0 0 0 0 rgba(22, 163, 74, 0); }
        }

        .pulse-red {
            box-shadow: 0 0 0 0 rgba(220, 38, 38, 0.7);
            animation: pulse-red-anim 1.5s infinite;
        }
        @keyframes pulse-red-anim {
            0% { box-shadow: 0 0 0 0 rgba(220, 38, 38, 0.7); }
            70% { box-shadow: 0 0 0 8px rgba(220, 38, 38, 0); }
            100% { box-shadow: 0 0 0 0 rgba(220, 38, 38, 0); }
        }

        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col p-3 md:p-5 gap-4 text-slate-800">

    <!-- ==================== ENCABEZADO PRINCIPAL ==================== -->
    <header class="bg-industrial-900 text-white rounded-xl p-4 shadow-md flex flex-col lg:flex-row justify-between items-center gap-4 border-b-4 border-accent-orange">
        <div class="flex items-center gap-3">
            <div class="p-3 bg-industrial-800 rounded-lg border border-emerald-600 text-accent-orangeBright text-2xl">
                <i class="fa-solid fa-shield-halved"></i>
            </div>
            <div>
                <h1 class="text-lg md:text-xl font-bold tracking-wide flex flex-wrap items-center gap-2">
                    SIMULADOR DE SEGURIDAD INDUSTRIAL INTELIGENTE
                    <span class="text-xs font-semibold bg-accent-orange text-white px-2 py-0.5 rounded uppercase">YOLOv8n + IoT + RED</span>
                </h1>
                <p class="text-xs text-emerald-200">Panel de Monitoreo Integral de Visión Artificial e Infraestructura Teleco</p>
            </div>
        </div>

        <div class="flex flex-wrap items-center gap-3 text-xs md:text-sm">
            <div class="bg-industrial-800 border border-emerald-700/50 rounded-lg px-3 py-1.5 flex items-center gap-2">
                <span id="global-status-dot" class="w-3 h-3 rounded-full bg-emerald-400 pulse-green"></span>
                <span class="text-slate-300">Estado:</span>
                <span id="global-status-text" class="font-bold text-emerald-300">SISTEMA OPTIMO</span>
            </div>

            <div class="flex items-center gap-2">
                <button id="btn-toggle-sim" onclick="toggleSimulation()" class="bg-emerald-600 hover:bg-emerald-500 text-white font-semibold px-3 py-1.5 rounded-lg transition-all flex items-center gap-1.5 shadow">
                    <i id="btn-sim-icon" class="fa-solid fa-pause"></i>
                    <span id="btn-sim-text">Pausar</span>
                </button>
                <button id="btn-toggle-congestion" onclick="toggleManualCongestion()" class="bg-amber-600 hover:bg-amber-500 text-white font-semibold px-3 py-1.5 rounded-lg transition-all flex items-center gap-1.5 shadow">
                    <i class="fa-solid fa-triangle-exclamation"></i>
                    <span>Congestión Router</span>
                </button>
                <button onclick="resetStats()" class="bg-slate-700 hover:bg-slate-600 text-slate-200 font-semibold px-3 py-1.5 rounded-lg transition-all flex items-center gap-1.5 shadow">
                    <i class="fa-solid fa-rotate-right"></i>
                    <span class="hidden sm:inline">Reiniciar</span>
                </button>
            </div>
        </div>
    </header>

    <!-- ==================== PANEL DE CONFIGURACIÓN DE VIDEO Y RED ==================== -->
    <section class="bg-white rounded-xl p-4 shadow-sm border border-slate-200">
        <div class="text-xs font-bold text-slate-500 uppercase tracking-wider mb-3 pb-1 border-b border-slate-100 flex items-center gap-2">
            <i class="fa-solid fa-sliders text-industrial-800"></i> Parámetros de Transmisión de Video & Enlace de Red
        </div>

        <div class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-6 gap-3">
            <div>
                <label class="text-[11px] font-semibold text-slate-600 block mb-1">Resolución</label>
                <select id="cfg-resolution" onchange="updateParameters()" class="w-full bg-slate-50 border border-slate-300 rounded p-1.5 text-xs font-medium focus:ring-2 focus:ring-industrial-800 focus:outline-none">
                    <option value="640x480">640x480</option>
                    <option value="1280x720" selected>1280x720</option>
                    <option value="1920x1080">1920x1080</option>
                </select>
            </div>

            <div>
                <label class="text-[11px] font-semibold text-slate-600 block mb-1">FPS</label>
                <select id="cfg-fps" onchange="updateParameters()" class="w-full bg-slate-50 border border-slate-300 rounded p-1.5 text-xs font-medium focus:ring-2 focus:ring-industrial-800 focus:outline-none">
                    <option value="15">15 FPS</option>
                    <option value="30" selected>30 FPS</option>
                    <option value="60">60 FPS</option>
                </select>
            </div>

            <div>
                <label class="text-[11px] font-semibold text-slate-600 block mb-1">Códec</label>
                <select id="cfg-codec" onchange="updateParameters()" class="w-full bg-slate-50 border border-slate-300 rounded p-1.5 text-xs font-medium focus:ring-2 focus:ring-industrial-800 focus:outline-none">
                    <option value="H.264" selected>H.264</option>
                    <option value="H.265">H.265</option>
                </select>
            </div>

            <div>
                <label class="text-[11px] font-semibold text-slate-600 block mb-1">QoS para video</label>
                <select id="cfg-qos" onchange="updateParameters()" class="w-full bg-slate-50 border border-slate-300 rounded p-1.5 text-xs font-medium focus:ring-2 focus:ring-industrial-800 focus:outline-none">
                    <option value="AF41" selected>DSCP AF41</option>
                    <option value="AF31">DSCP AF31</option>
                    <option value="BestEffort">Best Effort</option>
                </select>
            </div>

            <div>
                <div class="flex justify-between items-center text-[11px] font-semibold text-slate-600 mb-1">
                    <span>Tráfico de otros equipos</span>
                    <span id="slider-bg-val" class="text-industrial-800 font-bold">40 %</span>
                </div>
                <input id="cfg-bg-traffic" type="range" min="0" max="100" value="40" oninput="updateParameters()" class="w-full accent-industrial-800 cursor-pointer h-1.5 bg-slate-200 rounded mt-1">
            </div>

            <div>
                <label class="text-[11px] font-semibold text-slate-600 block mb-1">Enlace router</label>
                <select id="cfg-bandwidth" onchange="updateParameters()" class="w-full bg-slate-50 border border-slate-300 rounded p-1.5 text-xs font-medium focus:ring-2 focus:ring-industrial-800 focus:outline-none">
                    <option value="5">5 Mbps</option>
                    <option value="10">10 Mbps</option>
                    <option value="20" selected>20 Mbps</option>
                    <option value="50">50 Mbps</option>
                    <option value="100">100 Mbps</option>
                </select>
            </div>
        </div>
    </section>

    <!-- ==================== MONITOREO COMPARATIVO LADO A LADO ==================== -->
    <section class="grid grid-cols-1 lg:grid-cols-2 gap-4">
        
        <!-- PANEL IZQUIERDO: 1-5 · CÁMARA + YOLOV8N (SERVIDOR EN VIVO) -->
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 flex flex-col overflow-hidden">
            <div class="bg-slate-100 px-3 py-2 flex justify-between items-center border-b border-slate-200">
                <span class="font-bold text-xs text-slate-700 tracking-wider">1-5 · CÁMARA + YOLOV8N</span>
                <div class="flex items-center gap-3 text-xs">
                    <div class="bg-white px-2 py-0.5 rounded border border-slate-300 font-mono text-[11px] text-slate-700">
                        personas: <span id="cam-live-persons" class="font-bold text-slate-900">2</span> | autos: <span id="cam-live-cars" class="font-bold text-slate-900">1</span>
                    </div>
                    <span class="flex items-center gap-1 font-bold text-[11px] text-red-600">
                        EN VIVO <span class="w-2 h-2 rounded-full bg-red-600 inline-block pulse-red"></span>
                    </span>
                </div>
            </div>

            <!-- Canvas Cámara Live -->
            <div class="relative bg-slate-200 flex-grow min-h-[280px] w-full flex items-center justify-center overflow-hidden">
                <canvas id="canvas-cam-live" class="w-full h-full block object-cover"></canvas>
            </div>
            
            <div class="p-2 bg-slate-50 border-t border-slate-200 text-[11px] text-slate-500 flex justify-between items-center">
                <span>Captura local directa en el servidor de ingesta</span>
                <span class="font-mono text-[10px]">COCO: yolov8n.pt</span>
            </div>
        </div>

        <!-- PANEL DERECHO: 9 · LO QUE VE EL CLIENTE (REMOTO VIA RED) -->
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 flex flex-col overflow-hidden">
            <div class="bg-slate-100 px-3 py-2 flex justify-between items-center border-b border-slate-200">
                <span class="font-bold text-xs text-slate-700 tracking-wider">9 · LO QUE VE EL CLIENTE</span>
                <div class="flex items-center gap-3 text-xs">
                    <div class="bg-white px-2 py-0.5 rounded border border-slate-300 font-mono text-[11px] text-slate-700">
                        personas: <span id="cam-client-persons" class="font-bold text-slate-900">2</span> | autos: <span id="cam-client-cars" class="font-bold text-slate-900">1</span>
                    </div>
                    <span class="font-mono text-[11px] font-bold text-slate-700">
                        retardo <span id="client-delay-lbl" class="text-industrial-800">18 ms</span>
                    </span>
                </div>
            </div>

            <!-- Canvas Cliente VLC -->
            <div class="relative bg-slate-200 flex-grow min-h-[280px] w-full flex items-center justify-center overflow-hidden">
                <canvas id="canvas-cam-client" class="w-full h-full block object-cover"></canvas>
                <!-- Indicador visual de perdida/glitch -->
                <div id="glitch-overlay" class="absolute inset-0 bg-red-500/10 pointer-events-none hidden flex items-center justify-center text-red-700 font-bold text-xs bg-stripes">
                    [PÉRDIDA DE TRAMAS DE VIDEO]
                </div>
            </div>

            <div class="p-2 bg-slate-50 border-t border-slate-200 text-[11px] text-slate-500 flex justify-between items-center">
                <span>Streaming RTSP/RTP recibido en estación de cliente remoto</span>
                <span class="font-mono text-[10px]" id="client-fps-lbl">30.0 FPS</span>
            </div>
        </div>

    </section>

    <!-- ==================== ARQUITECTURA DE RED ==================== -->
    <section class="bg-white rounded-xl p-4 shadow-sm border border-slate-200">
        <div class="text-xs font-bold text-slate-500 uppercase tracking-wider mb-2 pb-1 border-b border-slate-100 flex items-center gap-2">
            <i class="fa-solid fa-diagram-project text-industrial-800"></i> 6-8 · RED: SERVIDOR → SWITCH → ROUTER → CLIENTE
        </div>

        <div class="relative bg-slate-50 rounded-lg p-2 border border-slate-200 overflow-hidden min-h-[140px]">
            <canvas id="canvas-network" class="w-full h-full block"></canvas>
        </div>

        <div class="mt-2 flex flex-wrap justify-between items-center text-xs text-slate-600 gap-2 pt-1 border-t border-slate-100">
            <div class="flex items-center gap-4 text-[11px]">
                <span class="flex items-center gap-1.5"><span class="w-2.5 h-2.5 rounded-full bg-accent-orange inline-block"></span> paquete de video</span>
                <span class="flex items-center gap-1.5"><span class="w-2.5 h-2.5 rounded-full bg-slate-500 inline-block"></span> otro tráfico</span>
                <span class="flex items-center gap-1.5"><span class="w-2.5 h-2.5 rounded-full bg-red-600 inline-block"></span> descartado en la cola del router</span>
            </div>
            <div class="font-mono text-[11px] text-slate-500">
                cola del router: <span id="lbl-router-queue" class="font-bold text-slate-800">0 ms</span>
            </div>
        </div>
    </section>

    <!-- ==================== INDICADORES EN TIEMPO REAL (6 TARJETAS PRINCIPALES) ==================== -->
    <section class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-6 gap-3">
        <!-- 1. Bitrate del video -->
        <div class="bg-white rounded-xl p-3 shadow-sm border border-slate-200 flex flex-col justify-between">
            <span class="text-[11px] font-semibold text-slate-500">Bitrate del video</span>
            <div class="my-1">
                <span id="kpi-bitrate" class="text-2xl font-black text-slate-800">2.80</span>
                <span class="text-xs font-bold text-slate-500">Mbps</span>
            </div>
            <span class="text-[10px] text-slate-400">H.264 / 1280x720</span>
        </div>

        <!-- 2. Disponible para video -->
        <div class="bg-white rounded-xl p-3 shadow-sm border border-slate-200 flex flex-col justify-between">
            <span class="text-[11px] font-semibold text-slate-500">Disponible para video</span>
            <div class="my-1">
                <span id="kpi-available-bw" class="text-2xl font-black text-slate-800">12.00</span>
                <span class="text-xs font-bold text-slate-500">Mbps</span>
            </div>
            <span class="text-[10px] text-slate-400">Tras tráfico de fondo</span>
        </div>

        <!-- 3. Uso del enlace -->
        <div class="bg-white rounded-xl p-3 shadow-sm border border-slate-200 flex flex-col justify-between">
            <span class="text-[11px] font-semibold text-slate-500">Uso del enlace</span>
            <div class="my-1">
                <span id="kpi-utilization" class="text-2xl font-black text-slate-800">68</span>
                <span class="text-xs font-bold text-slate-500">%</span>
            </div>
            <div class="w-full bg-slate-100 h-1.5 rounded-full overflow-hidden">
                <div id="bar-utilization" class="bg-industrial-700 h-full transition-all duration-300" style="width: 68%;"></div>
            </div>
        </div>

        <!-- 4. Pérdida de paquetes -->
        <div class="bg-white rounded-xl p-3 shadow-sm border border-slate-200 flex flex-col justify-between">
            <span class="text-[11px] font-semibold text-slate-500">Pérdida de paquetes</span>
            <div class="my-1">
                <span id="kpi-packet-loss" class="text-2xl font-black text-emerald-600">0.0</span>
                <span class="text-xs font-bold text-slate-500">%</span>
            </div>
            <div class="w-full bg-slate-100 h-1.5 rounded-full overflow-hidden">
                <div id="bar-packet-loss" class="bg-emerald-500 h-full transition-all duration-300" style="width: 0%;"></div>
            </div>
        </div>

        <!-- 5. Inferencia YOLOv8n -->
        <div class="bg-white rounded-xl p-3 shadow-sm border border-slate-200 flex flex-col justify-between">
            <span class="text-[11px] font-semibold text-slate-500">Inferencia YOLOv8n</span>
            <div class="my-1">
                <span id="kpi-inference-time" class="text-2xl font-black text-slate-800">12</span>
                <span class="text-xs font-bold text-slate-500">ms</span>
            </div>
            <span class="text-[10px] text-slate-400">yolov8n.pt (COCO)</span>
        </div>

        <!-- 6. Latencia extremo a extremo -->
        <div class="bg-white rounded-xl p-3 shadow-sm border border-slate-200 flex flex-col justify-between">
            <span class="text-[11px] font-semibold text-slate-500">Latencia extremo a extremo</span>
            <div class="my-1">
                <span id="kpi-latency" class="text-2xl font-black text-emerald-700">18</span>
                <span class="text-xs font-bold text-slate-500">ms</span>
            </div>
            <span class="text-[10px] text-slate-400">Red + Decodificación</span>
        </div>
    </section>

    <!-- PIE DE NOTA DIDÁCTICO -->
    <div class="text-xs text-slate-500 italic px-1">
        Valores aproximados para fines didácticos. En el laboratorio se reemplazan por mediciones reales con Wireshark e iperf3.
    </div>

    <!-- ==================== SECCIÓN INFERENCIA IA & GRÁFICAS & LOGS ==================== -->
    <section class="grid grid-cols-1 lg:grid-cols-3 gap-4">
        
        <!-- PANEL DE CONTROL IA YOLOV8 -->
        <div class="bg-white rounded-xl p-4 shadow-sm border border-slate-200 flex flex-col justify-between">
            <div>
                <div class="flex items-center justify-between pb-2 mb-3 border-b border-slate-100">
                    <span class="font-bold text-xs text-slate-700 uppercase tracking-wider">Flujo de Inferencia YOLOv8n</span>
                    <span class="text-[10px] font-mono bg-emerald-50 text-emerald-800 font-semibold px-2 py-0.5 rounded border border-emerald-200">Ultralytics PyTorch</span>
                </div>

                <div class="grid grid-cols-5 gap-1 text-center text-[9px] font-mono mb-3 bg-slate-900 text-slate-200 p-2 rounded">
                    <div class="bg-slate-800 p-1 rounded">Entrada</div>
                    <div class="bg-slate-800 p-1 rounded">Resize</div>
                    <div class="bg-industrial-800 text-emerald-300 font-bold p-1 rounded">yolov8n</div>
                    <div class="bg-slate-800 p-1 rounded">NMS</div>
                    <div class="bg-slate-800 p-1 rounded">BBoxes</div>
                </div>

                <div class="space-y-2 text-xs">
                    <div class="flex justify-between p-1.5 bg-slate-50 rounded border border-slate-200">
                        <span class="text-slate-600">Modelo Activo:</span>
                        <span class="font-bold font-mono">yolov8n.pt</span>
                    </div>
                    <div class="flex justify-between p-1.5 bg-slate-50 rounded border border-slate-200">
                        <span class="text-slate-600">Clases COCO:</span>
                        <span class="font-bold text-accent-orange font-mono">persona, auto</span>
                    </div>
                    <div class="flex justify-between p-1.5 bg-slate-50 rounded border border-slate-200">
                        <span class="text-slate-600">Confianza Promedio:</span>
                        <span id="lbl-avg-conf" class="font-bold text-emerald-700">93.5%</span>
                    </div>
                </div>
            </div>

            <div class="mt-3 pt-2 border-t border-slate-100 flex items-center justify-between text-xs">
                <span class="text-[11px] text-slate-500">Backend Python: <strong id="backend-status-lbl" class="text-slate-400">SIMULADO</strong></span>
                <button onclick="openBackendModal()" class="bg-slate-800 hover:bg-slate-700 text-white font-medium px-2.5 py-1 rounded text-[11px] transition-colors">
                    <i class="fa-solid fa-plug text-accent-orange"></i> Conectar Backend
                </button>
            </div>
        </div>

        <!-- PANEL DE GRÁFICAS DINÁMICAS (CHART.JS) -->
        <div class="bg-white rounded-xl p-4 shadow-sm border border-slate-200 flex flex-col">
            <div class="flex items-center justify-between pb-2 mb-2 border-b border-slate-100">
                <span class="font-bold text-xs text-slate-700 uppercase tracking-wider">Métricas de Red en Tiempo Real</span>
                <span class="text-[10px] text-slate-400">Ventana: 30s</span>
            </div>

            <div class="grid grid-cols-1 gap-2 flex-grow h-48">
                <canvas id="chart-teleco"></canvas>
            </div>
        </div>

        <!-- PANEL DE DISPARO DE EVENTOS Y REGISTRO DE ALERTAS -->
        <div class="bg-white rounded-xl p-4 shadow-sm border border-slate-200 flex flex-col justify-between">
            <div>
                <div class="flex items-center justify-between pb-2 mb-2 border-b border-slate-100">
                    <span class="font-bold text-xs text-slate-700 uppercase tracking-wider">Simulación de Eventos & Log</span>
                    <button onclick="clearLogs()" class="text-[10px] text-slate-400 hover:text-slate-600 underline">Limpiar</button>
                </div>

                <div class="flex flex-wrap gap-1 mb-2">
                    <button onclick="triggerEvent('PERSON_DETECTED')" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-[10px] px-2 py-1 rounded border font-medium">
                        <i class="fa-solid fa-user-check text-emerald-600"></i> Persona
                    </button>
                    <button onclick="triggerEvent('CAR_DETECTED')" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-[10px] px-2 py-1 rounded border font-medium">
                        <i class="fa-solid fa-car text-blue-600"></i> Auto
                    </button>
                    <button onclick="triggerEvent('TRAFFIC_SPIKE')" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-[10px] px-2 py-1 rounded border font-medium">
                        <i class="fa-solid fa-chart-simple text-amber-600"></i> Pico Tráfico
                    </button>
                    <button onclick="triggerEvent('PACKET_DROP_EVENT')" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-[10px] px-2 py-1 rounded border font-medium">
                        <i class="fa-solid fa-dumpster-fire text-red-600"></i> Caída Pkts
                    </button>
                </div>

                <div id="log-container" class="bg-slate-900 rounded p-2 text-slate-200 font-mono text-[10px] h-36 overflow-y-auto space-y-1.5 custom-scrollbar">
                </div>
            </div>
        </div>
    </section>

    <!-- MODAL CONFIGURACIÓN BACKEND PYTHON -->
    <div id="backend-modal" class="fixed inset-0 bg-black/60 backdrop-blur-xs flex items-center justify-center hidden z-50 p-4">
        <div class="bg-white rounded-xl shadow-xl max-w-md w-full p-5 border border-slate-200">
            <div class="flex justify-between items-center pb-3 border-b border-slate-100">
                <h3 class="font-bold text-slate-800 text-sm flex items-center gap-2">
                    <i class="fa-brands fa-python text-blue-600 text-lg"></i> Conexión Backend Python YOLOv8
                </h3>
                <button onclick="closeBackendModal()" class="text-slate-400 hover:text-slate-600">
                    <i class="fa-solid fa-xmark text-lg"></i>
                </button>
            </div>
            
            <div class="my-4 space-y-3 text-xs">
                <p class="text-slate-600">
                    Conecta este frontend con una instancia real de Ultralytics YOLOv8 ejecutándose en Python mediante WebSocket.
                </p>

                <div>
                    <label class="block font-semibold text-slate-700 mb-1">URL WebSocket:</label>
                    <input id="ws-url-input" type="text" value="ws://localhost:8000/ws/yolo" class="w-full bg-slate-50 border border-slate-300 rounded p-2 font-mono text-xs focus:outline-none">
                </div>
            </div>

            <div class="flex justify-end gap-2 pt-2 border-t border-slate-100">
                <button onclick="closeBackendModal()" class="px-3 py-1.5 bg-slate-200 hover:bg-slate-300 text-slate-700 font-semibold rounded text-xs">
                    Cancelar
                </button>
                <button onclick="connectBackend()" class="px-3 py-1.5 bg-industrial-800 hover:bg-industrial-900 text-white font-semibold rounded text-xs flex items-center gap-1">
                    <i class="fa-solid fa-plug"></i> Probar Conexión
                </button>
            </div>
        </div>
    </div>

    <!-- ==================== SCRIPT DE SIMULACIÓN Y CRISTALINIDAD CANVAS ==================== -->
    <script>
        // ESTADO DE LA SIMULACIÓN
        const simState = {
            running: true,
            manualCongestion: false,
            
            resolution: '1280x720',
            fps: 30,
            codec: 'H.264',
            qos: 'AF41',
            bandwidthMbps: 20,
            bgTrafficPercent: 40,

            videoBitrate: 2.80,
            availableBw: 12.00,
            utilizationPct: 68,
            packetLossPct: 0.0,
            inferenceTimeMs: 12,
            latencyMs: 18,
            jitterMs: 2.1,
            throughputMbps: 10.5,

            totalPktsTx: 0,
            totalPktsDropped: 0
        };

        // OBJETOS ANIMADOS NATIVOS COCO (SOLO PERSONAS Y AUTOS)
        const sceneObjects = [
            { type: 'persona', x: 80, y: 155, vx: 0.7, label: 'persona', conf: 0.94, w: 22, h: 48 },
            { type: 'persona', x: 280, y: 160, vx: -0.5, label: 'persona', conf: 0.89, w: 22, h: 48 },
            { type: 'auto', x: 120, y: 195, vx: 1.8, label: 'auto', conf: 0.91, w: 90, h: 42 },
            { type: 'persona', x: 420, y: 150, vx: 0.6, label: 'persona', conf: 0.96, w: 22, h: 48 }
        ];

        // Partículas de red para la animación
        const netParticles = [];

        // Frame Buffer para simular retardo visual en el cliente
        const clientFrameBuffer = [];

        let chartTeleco;

        // FUNCIÓN CLAVE PARA ULTRA-NÍTIDEZ EN CANVAS (devicePixelRatio)
        function setupSharpCanvas(canvas) {
            const dpr = window.devicePixelRatio || 1;
            const rect = canvas.getBoundingClientRect();
            
            if (rect.width === 0 || rect.height === 0) return { ctx: canvas.getContext('2d'), w: canvas.width, h: canvas.height };

            const targetW = Math.round(rect.width * dpr);
            const targetH = Math.round(rect.height * dpr);

            if (canvas.width !== targetW || canvas.height !== targetH) {
                canvas.width = targetW;
                canvas.height = targetH;
            }

            const ctx = canvas.getContext('2d');
            ctx.resetTransform();
            ctx.scale(dpr, dpr);
            return { ctx, w: rect.width, h: rect.height };
        }

        window.onload = function() {
            initChart();
            updateParameters();
            
            requestAnimationFrame(renderLoop);
            setInterval(physicsLoop, 400);

            logEvent('INFO', 'Simulador de Seguridad Industrial iniciado correctamente.');
            logEvent('INFO', 'Modelo yolov8n.pt cargado (Clases: persona, auto).');
        };

        function updateParameters() {
            simState.resolution = document.getElementById('cfg-resolution').value;
            simState.fps = parseInt(document.getElementById('cfg-fps').value);
            simState.codec = document.getElementById('cfg-codec').value;
            simState.qos = document.getElementById('cfg-qos').value;
            simState.bandwidthMbps = parseFloat(document.getElementById('cfg-bandwidth').value);
            simState.bgTrafficPercent = parseInt(document.getElementById('cfg-bg-traffic').value);

            document.getElementById('slider-bg-val').innerText = `${simState.bgTrafficPercent} %`;

            // Cálculo dinámico de bitrate
            let baseBitrate = 3.0;
            if (simState.resolution === '640x480') baseBitrate = 1.3;
            if (simState.resolution === '1920x1080') baseBitrate = 7.0;

            let fpsFactor = simState.fps / 30.0;
            let codecFactor = simState.codec === 'H.265' ? 0.6 : 1.0;

            simState.videoBitrate = baseBitrate * fpsFactor * codecFactor;
            
            let bgMbps = simState.bandwidthMbps * (simState.bgTrafficPercent / 100.0);
            simState.availableBw = Math.max(0, simState.bandwidthMbps - bgMbps);

            let totalDemand = simState.videoBitrate + bgMbps;
            simState.utilizationPct = Math.min(100, Math.round((totalDemand / simState.bandwidthMbps) * 100));
        }

        function physicsLoop() {
            if (!simState.running) return;

            const bw = simState.bandwidthMbps;
            const bgMbps = bw * (simState.bgTrafficPercent / 100.0);
            const totalRequested = simState.videoBitrate + bgMbps;
            let ratio = totalRequested / bw;

            if (simState.manualCongestion) ratio += 0.55;

            let qosFactor = simState.qos === 'AF41' ? 0.4 : (simState.qos === 'AF31' ? 0.7 : 1.2);

            if (ratio <= 0.85) {
                simState.latencyMs = Math.round(14 + Math.random() * 5);
                simState.jitterMs = parseFloat((1.0 + Math.random() * 2).toFixed(1));
                simState.packetLossPct = 0.0;
            } else if (ratio <= 1.0) {
                simState.latencyMs = Math.round(25 + (ratio - 0.85) * 80 * qosFactor);
                simState.jitterMs = parseFloat((3.0 + Math.random() * 4).toFixed(1));
                simState.packetLossPct = parseFloat(((ratio - 0.85) * 1.5 * qosFactor).toFixed(1));
            } else {
                let overflow = ratio - 1.0;
                simState.latencyMs = Math.round(50 + overflow * 250 * qosFactor + Math.random() * 20);
                simState.jitterMs = parseFloat((8.0 + overflow * 40 * qosFactor).toFixed(1));
                simState.packetLossPct = parseFloat(Math.min(30.0, overflow * 35.0 * qosFactor + Math.random() * 2).toFixed(1));
            }

            simState.throughputMbps = parseFloat((Math.min(simState.videoBitrate, simState.availableBw) * (1 - simState.packetLossPct / 100)).toFixed(2));
            simState.inferenceTimeMs = Math.round(10 + Math.random() * 4);

            updateKPICards();
            updateChartData();
        }

        function updateKPICards() {
            document.getElementById('kpi-bitrate').innerText = simState.videoBitrate.toFixed(2);
            document.getElementById('kpi-available-bw').innerText = simState.availableBw.toFixed(2);
            document.getElementById('kpi-utilization').innerText = simState.utilizationPct;
            document.getElementById('kpi-packet-loss').innerText = simState.packetLossPct.toFixed(1);
            document.getElementById('kpi-inference-time').innerText = simState.inferenceTimeMs;
            document.getElementById('kpi-latency').innerText = simState.latencyMs;

            document.getElementById('client-delay-lbl').innerText = `${simState.latencyMs} ms`;
            document.getElementById('lbl-router-queue').innerText = `${Math.round(simState.latencyMs * 0.4)} ms`;

            document.getElementById('bar-utilization').style.width = `${simState.utilizationPct}%`;
            document.getElementById('bar-packet-loss').style.width = `${Math.min(100, simState.packetLossPct * 3.3)}%`;

            const statusDot = document.getElementById('global-status-dot');
            const statusText = document.getElementById('global-status-text');

            if (simState.packetLossPct > 1.0 || simState.manualCongestion) {
                statusDot.className = "w-3 h-3 rounded-full bg-red-600 pulse-red";
                statusText.innerText = "CONGESTIÓN / PÉRDIDA";
                statusText.className = "font-bold text-red-400";
            } else {
                statusDot.className = "w-3 h-3 rounded-full bg-emerald-400 pulse-green";
                statusText.innerText = "SISTEMA OPTIMO";
                statusText.className = "font-bold text-emerald-300";
            }

            // Mostrar u ocultar overlay de glitch en cliente
            const glitchOverlay = document.getElementById('glitch-overlay');
            if (simState.packetLossPct > 0.5) {
                glitchOverlay.classList.remove('hidden');
            } else {
                glitchOverlay.classList.add('hidden');
            }
        }

        function renderLoop() {
            if (simState.running) {
                animateSceneObjects();
                
                // Capturar fotograma local para la cámara en vivo
                const liveData = drawAvenueScene('canvas-cam-live', true);
                
                // Guardar en el buffer de retardo de cliente
                clientFrameBuffer.push({
                    timestamp: Date.now(),
                    objects: JSON.parse(JSON.stringify(sceneObjects))
                });

                // Renderizar cliente según latencia
                drawClientView();

                // Animación arquitectura de red
                drawNetworkTopology();
            }
            requestAnimationFrame(renderLoop);
        }

        function animateSceneObjects() {
            sceneObjects.forEach(obj => {
                obj.x += obj.vx;
                if (obj.x > 520) obj.x = -60;
                if (obj.x < -70) obj.x = 510;
            });
        }

        // DIBUJAR ESCENA INDUSTRIAL ESTILIZADA Y NÍTIDA
        function drawAvenueScene(canvasId, isLive, overrideObjects = null) {
            const canvas = document.getElementById(canvasId);
            const { ctx, w, h } = setupSharpCanvas(canvas);

            const objectsToDraw = overrideObjects || sceneObjects;

            // Fondo: Cielo industrial suave
            ctx.fillStyle = '#f1f5f9';
            ctx.fillRect(0, 0, w, h);

            // Estructura Nave Industrial limpia al fondo
            ctx.fillStyle = '#e2e8f0';
            ctx.fillRect(40, 30, 160, 90);
            ctx.fillRect(280, 20, 180, 100);

            // Franjas decorativas verde industrial
            ctx.fillStyle = '#0f5132';
            ctx.fillRect(40, 50, 160, 8);
            ctx.fillRect(280, 45, 180, 8);

            // Ventanales industriales
            ctx.fillStyle = '#94a3b8';
            ctx.fillRect(60, 70, 35, 25);
            ctx.fillRect(110, 70, 35, 25);
            ctx.fillRect(300, 65, 45, 30);
            ctx.fillRect(360, 65, 45, 30);

            // Árboles geométricos limpios
            ctx.fillStyle = '#15803d';
            ctx.beginPath(); ctx.arc(230, 95, 18, 0, Math.PI * 2); ctx.fill();
            ctx.fillStyle = '#78350f'; ctx.fillRect(228, 110, 4, 15);

            // Andén Peatonal
            ctx.fillStyle = '#cbd5e1';
            ctx.fillRect(0, 125, w, 45);
            ctx.fillStyle = '#94a3b8';
            ctx.fillRect(0, 125, w, 3); // Borde andén

            // Pavimento de Avenida
            ctx.fillStyle = '#334155';
            ctx.fillRect(0, 170, w, 80);

            // Marcas viales discontinuas
            ctx.strokeStyle = '#fef08a';
            ctx.lineWidth = 2;
            ctx.setLineDash([15, 12]);
            ctx.beginPath();
            ctx.moveTo(0, 210); ctx.lineTo(w, 210);
            ctx.stroke();
            ctx.setLineDash([]); // Reset dash

            // Contar clases
            let personCount = 0;
            let carCount = 0;

            // Dibujar Objetos con Bounding Boxes Nítidos
            objectsToDraw.forEach(obj => {
                if (obj.type === 'persona') {
                    personCount++;
                    // Sombra
                    ctx.fillStyle = 'rgba(0,0,0,0.15)';
                    ctx.beginPath(); ctx.ellipse(obj.x + 11, obj.y + 45, 10, 4, 0, 0, Math.PI * 2); ctx.fill();

                    // Silueta Humana limpia
                    ctx.fillStyle = '#1e293b';
                    ctx.beginPath(); ctx.arc(obj.x + 11, obj.y + 8, 6, 0, Math.PI * 2); ctx.fill(); // Cabeza
                    ctx.fillRect(obj.x + 5, obj.y + 15, 12, 22); // Torso/Piernas

                    // Bounding Box YOLO Naranja Vivo
                    const bx = obj.x;
                    const by = obj.y - 2;
                    const bw = obj.w;
                    const bh = obj.h;

                    ctx.strokeStyle = '#ea580c';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(bx, by, bw, bh);

                    // Etiqueta
                    const labelText = `persona ${obj.conf.toFixed(2)}`;
                    ctx.font = 'bold 10px monospace';
                    const textWidth = ctx.measureText(labelText).width;

                    ctx.fillStyle = '#ea580c';
                    ctx.fillRect(bx, by - 14, textWidth + 6, 14);

                    ctx.fillStyle = '#ffffff';
                    ctx.fillText(labelText, bx + 3, by - 3);

                } else if (obj.type === 'auto') {
                    carCount++;
                    // Sombra
                    ctx.fillStyle = 'rgba(0,0,0,0.2)';
                    ctx.fillRect(obj.x + 4, obj.y + 32, obj.w - 8, 8);

                    // Vehículo Estilizado
                    ctx.fillStyle = '#0284c7'; // Azul metálico
                    ctx.fillRect(obj.x + 10, obj.y + 8, obj.w - 20, 16); // Cabina
                    ctx.fillRect(obj.x, obj.y + 20, obj.w, 15); // Cuerpo

                    // Ruedas
                    ctx.fillStyle = '#0f172a';
                    ctx.beginPath(); ctx.arc(obj.x + 18, obj.y + 33, 7, 0, Math.PI * 2); ctx.fill();
                    ctx.beginPath(); ctx.arc(obj.x + obj.w - 18, obj.y + 33, 7, 0, Math.PI * 2); ctx.fill();

                    // Bounding Box YOLO
                    const bx = obj.x - 2;
                    const by = obj.y + 4;
                    const bw = obj.w + 4;
                    const bh = obj.h;

                    ctx.strokeStyle = '#ea580c';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(bx, by, bw, bh);

                    // Etiqueta
                    const labelText = `auto ${obj.conf.toFixed(2)}`;
                    ctx.font = 'bold 10px monospace';
                    const textWidth = ctx.measureText(labelText).width;

                    ctx.fillStyle = '#ea580c';
                    ctx.fillRect(bx, by - 14, textWidth + 6, 14);

                    ctx.fillStyle = '#ffffff';
                    ctx.fillText(labelText, bx + 3, by - 3);
                }
            });

            if (isLive) {
                document.getElementById('cam-live-persons').innerText = personCount;
                document.getElementById('cam-live-cars').innerText = carCount;
            } else {
                document.getElementById('cam-client-persons').innerText = personCount;
                document.getElementById('cam-client-cars').innerText = carCount;
            }
        }

        // DIBUJAR VISTA DEL CLIENTE CON RETARDO DE RED
        function drawClientView() {
            const delayTarget = simState.latencyMs;
            const now = Date.now();
            
            // Buscar fotograma retenido en buffer según latencia
            let bufferedFrame = null;
            for (let i = clientFrameBuffer.length - 1; i >= 0; i--) {
                if (now - clientFrameBuffer[i].timestamp >= delayTarget) {
                    bufferedFrame = clientFrameBuffer[i];
                    // Limpiar antiguos
                    clientFrameBuffer.splice(0, i);
                    break;
                }
            }

            const frameToDraw = bufferedFrame ? bufferedFrame.objects : sceneObjects;
            drawAvenueScene('canvas-cam-client', false, frameToDraw);
        }

        // DIAGRAMA DE ARQUITECTURA DE RED HORIZONTAL Y NÍTIDO
        function drawNetworkTopology() {
            const canvas = document.getElementById('canvas-network');
            const { ctx, w, h } = setupSharpCanvas(canvas);

            ctx.clearRect(0, 0, w, h);

            const nodes = [
                { id: 'server', name: 'Servidor', sub: 'YOLO · VLAN10', x: w * 0.12, y: h * 0.45, color: '#0f5132', icon: '\uf233' },
                { id: 'switch', name: 'Switch', sub: '2960 · 802.1Q', x: w * 0.38, y: h * 0.45, color: '#0284c7', icon: '\uf6ff' },
                { id: 'router', name: 'Router', sub: 'R-o-a-S · NAT', x: w * 0.64, y: h * 0.45, color: '#8b5cf6', icon: '\uf012' },
                { id: 'client', name: 'Cliente', sub: 'VLC · VLAN20', x: w * 0.88, y: h * 0.45, color: '#06b6d4', icon: '\uf108' }
            ];

            // Líneas de enlace
            ctx.strokeStyle = '#cbd5e1';
            ctx.lineWidth = 3;
            ctx.beginPath();
            for (let i = 0; i < nodes.length - 1; i++) {
                ctx.moveTo(nodes[i].x, nodes[i].y);
                ctx.lineTo(nodes[i + 1].x, nodes[i + 1].y);
            }
            ctx.stroke();

            // Generar partículas de paquetes
            if (Math.random() < 0.5) {
                netParticles.push({
                    x: nodes[0].x,
                    y: nodes[0].y,
                    speed: 2.8 + Math.random() * 1.5,
                    type: 'video',
                    color: '#ea580c'
                });

                if (simState.bgTrafficPercent > 0) {
                    netParticles.push({
                        x: nodes[1].x,
                        y: nodes[1].y,
                        speed: 2.2 + Math.random() * 1.5,
                        type: 'bg',
                        color: '#64748b'
                    });
                }
            }

            // Animar paquetes
            for (let i = netParticles.length - 1; i >= 0; i--) {
                const p = netParticles[i];
                p.x += p.speed;

                // Descarte en Router si hay pérdida
                if (p.x >= nodes[2].x && p.x <= nodes[2].x + 12) {
                    if (Math.random() * 100 < simState.packetLossPct) {
                        ctx.fillStyle = '#dc2626';
                        ctx.beginPath(); ctx.arc(p.x, p.y + 12, 4, 0, Math.PI * 2); ctx.fill();
                        netParticles.splice(i, 1);
                        continue;
                    }
                }

                ctx.fillStyle = p.color;
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.type === 'video' ? 4 : 3, 0, Math.PI * 2);
                ctx.fill();

                if (p.x >= nodes[nodes.length - 1].x) {
                    netParticles.splice(i, 1);
                }
            }

            // Dibujar Nodos
            nodes.forEach(node => {
                ctx.fillStyle = '#ffffff';
                ctx.strokeStyle = node.color;
                ctx.lineWidth = 2.5;
                ctx.beginPath();
                ctx.arc(node.x, node.y, 18, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();

                ctx.fillStyle = node.color;
                ctx.font = '900 12px "Font Awesome 6 Free"';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(node.icon, node.x, node.y);

                ctx.fillStyle = '#1e293b';
                ctx.font = 'bold 11px sans-serif';
                ctx.fillText(node.name, node.x, node.y + 28);

                ctx.fillStyle = '#64748b';
                ctx.font = '9px monospace';
                ctx.fillText(node.sub, node.x, node.y + 40);
            });
        }

        // GRÁFICAS CHART.JS
        function initChart() {
            const ctx = document.getElementById('chart-teleco').getContext('2d');
            const labels = Array(20).fill('');

            chartTeleco = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: labels,
                    datasets: [
                        { label: 'Latencia (ms)', data: Array(20).fill(18), borderColor: '#0f5132', borderWidth: 2, tension: 0.3, pointRadius: 0 },
                        { label: 'Pérdida (%)', data: Array(20).fill(0), borderColor: '#dc2626', borderWidth: 1.5, tension: 0.3, pointRadius: 0 }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    animation: false,
                    plugins: { legend: { labels: { boxWidth: 8, font: { size: 10 } } } },
                    scales: {
                        x: { display: false },
                        y: { ticks: { font: { size: 9 } }, grid: { color: '#f1f5f9' } }
                    }
                }
            });
        }

        function updateChartData() {
            if (!chartTeleco) return;
            chartTeleco.data.datasets[0].data.shift();
            chartTeleco.data.datasets[0].data.push(simState.latencyMs);
            chartTeleco.data.datasets[1].data.shift();
            chartTeleco.data.datasets[1].data.push(simState.packetLossPct);
            chartTeleco.update();
        }

        // EVENTOS Y LOGS
        function logEvent(severity, text) {
            const container = document.getElementById('log-container');
            const time = new Date().toTimeString().split(' ')[0];

            let color = 'text-slate-300';
            if (severity === 'WARN') color = 'text-amber-400';
            if (severity === 'CRITICAL') color = 'text-red-400 font-bold';

            const div = document.createElement('div');
            div.className = 'flex gap-2';
            div.innerHTML = `<span class="text-slate-500">[${time}]</span> <span class="${color}">${text}</span>`;

            container.prepend(div);
            if (container.children.length > 25) container.removeChild(container.lastChild);
        }

        function clearLogs() {
            document.getElementById('log-container').innerHTML = '';
        }

        function toggleSimulation() {
            simState.running = !simState.running;
            document.getElementById('btn-sim-text').innerText = simState.running ? 'Pausar' : 'Reanudar';
            document.getElementById('btn-sim-icon').className = simState.running ? 'fa-solid fa-pause' : 'fa-solid fa-play';
        }

        function toggleManualCongestion() {
            simState.manualCongestion = !simState.manualCongestion;
            logEvent(simState.manualCongestion ? 'CRITICAL' : 'INFO', simState.manualCongestion ? 'Congestión forzada en Router.' : 'Congestión desactivada.');
        }

        function resetStats() {
            logEvent('INFO', 'Estadísticas reiniciadas.');
        }

        function triggerEvent(type) {
            if (type === 'PERSON_DETECTED') logEvent('INFO', 'YOLOv8n: Detección de persona (Confianza: 96.2%).');
            if (type === 'CAR_DETECTED') logEvent('INFO', 'YOLOv8n: Vehículo detectado en carril principal.');
            if (type === 'TRAFFIC_SPIKE') {
                document.getElementById('cfg-bg-traffic').value = 90;
                updateParameters();
                logEvent('WARN', 'Inyección de tráfico de fondo al 90%.');
            }
            if (type === 'PACKET_DROP_EVENT') logEvent('CRITICAL', 'Caída masiva de paquetes en cola del Router.');
        }

        function openBackendModal() { document.getElementById('backend-modal').classList.remove('hidden'); }
        function closeBackendModal() { document.getElementById('backend-modal').classList.add('hidden'); }
        function connectBackend() {
            logEvent('INFO', 'Conectando con Backend Python WebSocket...');
            closeBackendModal();
        }
    </script>
</body>
</html>
