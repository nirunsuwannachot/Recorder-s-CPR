# Recorder CPR
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS 2025 Smart Recorder</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* ล็อคความสูงหน้าจอแบบ Dynamic ป้องกันปัญหามือถือมีแถบ URL */
        :root {
            --app-height: 100dvh;
        }

        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden; /* ห้ามเลื่อนทั้งหน้าจอ */
            display: flex;
            flex-direction: column;
            background-color: #0f172a; /* slate-900 */
        }

        /* ส่วนที่เลื่อนได้ (Action Log) */
        .scrollable-content {
            flex: 1; /* กินพื้นที่ที่เหลือทั้งหมด */
            overflow-y: auto; /* เลื่อนในแนวดิ่งได้ */
            -webkit-overflow-scrolling: touch;
            background-color: #f8fafc; /* slate-50 */
        }

        /* ตกแต่งปุ่มกดให้พอดี */
        .control-panel {
            flex-shrink: 0; /* ห้ามถูกบีบขนาด */
        }

        .no-scrollbar::-webkit-scrollbar { display: none; }
        
        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            word-break: break-word; height: 50px; font-size: 10px; font-weight: 900;
            border-radius: 8px; transition: all 0.1s;
        }
        .drug-btn:active { transform: scale(0.95); }
    </style>
</head>
<body class="font-sans antialiased">

    <div class="control-panel p-4 pb-2 bg-slate-900 text-white shadow-xl">
        <div class="flex justify-between items-start mb-3">
            <div>
                <h1 class="text-xl font-black text-red-500 leading-none">ACLS 2025</h1>
                <p class="text-[9px] text-slate-400 uppercase font-bold tracking-widest mt-1">Nirun Suwannachot</p>
            </div>
            <div class="text-right">
                <p class="text-[9px] text-slate-500 uppercase font-bold">Total Time</p>
                <p id="total-timer" class="text-2xl font-mono font-bold text-emerald-400 leading-none">00:00</p>
            </div>
        </div>

        <div class="grid grid-cols-2 gap-2 mb-3">
            <div id="cpr-card" class="bg-slate-800 p-2 rounded-xl border-2 border-blue-500 text-center">
                <p class="text-[9px] text-blue-400 font-bold uppercase">Rhythm Check In</p>
                <p id="cpr-timer" class="text-3xl font-mono font-black">02:00</p>
            </div>
            <div class="bg-slate-800 p-2 rounded-xl border border-slate-700 text-[10px]">
                <p class="text-yellow-500 font-bold uppercase mb-1">Guide</p>
                <div id="guidance-text" class="text-slate-400 italic leading-tight">Standby...</div>
            </div>
        </div>

        <div class="grid grid-cols-3 gap-2">
            <button onclick="toggleStart()" id="btn-start" class="bg-emerald-600 py-3 rounded-lg font-black text-xs uppercase shadow-lg">Start</button>
            <button onclick="setRhythm('Shockable')" class="bg-rose-700 py-3 rounded-lg font-black text-xs uppercase">Shockable</button>
            <button onclick="setRhythm('Non-Shockable')" class="bg-blue-700 py-3 rounded-lg font-black text-xs uppercase text-[9px]">Non-Shock</button>
        </div>
    </div>

    <div class="control-panel p-3 bg-white border-b border-slate-200">
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordAction('💓 Start CPR')" class="drug-btn border-2 border-emerald-500 text-emerald-700 bg-emerald-50">START CPR</button>
            <button onclick="recordAction('💉 Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700 bg-sky-50 uppercase">Access IV/IO</button>
            <button onclick="recordAction('💧 IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700 bg-blue-50">IV FLUID</button>
        </div>
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordDefib()" class="drug-btn border-2 border-rose-500 text-rose-600 bg-rose-50 italic">DEFIB 200J</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPINEPHRINE</button>
            <button onclick="recordAmio()" class="drug-btn border-2 border-purple-500 text-purple-600 bg-white">AMIODARONE</button>
        </div>
        <div class="grid grid-cols-2 gap-2">
            <button onclick="recordAction('💊 Amiodarone 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700">AMIO 150</button>
            <button onclick="recordAction('🌬️ Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-300 text-orange-700 italic uppercase">Adv. Airway</button>
        </div>
    </div>

    <div class="scrollable-content relative">
        <table class="w-full text-left border-separate border-spacing-y-1 p-3">
            <thead class="sticky top-0 bg-slate-50 z-10">
                <tr class="text-[10px] text-slate-400 uppercase font-black">
                    <th class="py-2 w-16 text-center">Time</th>
                    <th class="py-2 px-2">Action Log</th>
                </tr>
            </thead>
            <tbody id="log-body" class="text-xs font-bold text-slate-700">
                </tbody>
        </table>

        <div class="fixed bottom-20 right-4 flex flex-col gap-2 z-20">
            <div class="bg-rose-600 text-white px-3 py-1 rounded-xl shadow-2xl border-2 border-white text-center">
                <p class="text-[8px] font-black uppercase leading-none">Shock</p>
                <p id="dash-shock" class="text-xl font-black">0</p>
            </div>
            <div class="bg-blue-600 text-white px-3 py-1 rounded-xl shadow-2xl border-2 border-white text-center">
                <p class="text-[8px] font-black uppercase leading-none">Epi</p>
                <p id="dash-epi" class="text-xl font-black">0</p>
            </div>
        </div>
    </div>

    <div class="control-panel p-3 bg-white border-t flex gap-3">
        <button onclick="recordAction('🏁 ROSC Achieved')" class="flex-1 bg-emerald-500 text-white py-3 rounded-xl font-black text-xs uppercase shadow-lg">ROSC</button>
        <button id="btn-close-case" onclick="closeCase()" class="flex-1 bg-slate-800 text-white py-3 rounded-xl font-black text-xs uppercase">Close Case</button>
    </div>

    <div id="summary-modal" class="fixed inset-0 z-50 hidden bg-black/90 backdrop-blur-sm p-4 flex items-center justify-center">
        <div class="bg-white w-full max-w-sm rounded-3xl p-6 shadow-2xl overflow-hidden flex flex-col max-h-[80vh]">
            <h2 class="text-2xl font-black mb-4 uppercase border-b pb-2">Summary</h2>
            <div id="summary-content" class="overflow-y-auto mb-4">
                <div class="grid grid-cols-2 gap-2 mb-4">
                    <div class="bg-slate-100 p-2 rounded-xl text-center"><p class="text-[9px] font-bold text-slate-500">Duration</p><p id="sum-time" class="text-xl font-black">00:00</p></div>
                    <div class="bg-rose-100 p-2 rounded-xl text-center"><p class="text-[9px] font-bold text-rose-500">Shock</p><p id="sum-shock" class="text-xl font-black text-rose-600">0</p></div>
                </div>
                <div id="rhythm-timeline" class="space-y-1 text-xs"></div>
            </div>
            <button onclick="document.getElementById('summary-modal').style.display='none'" class="w-full bg-slate-800 text-white py-4 rounded-2xl font-black">CLOSE</button>
        </div>
    </div>

    <script>
        // --- Logic Start ---
        let totalSec = 0, cprSec = 120, isRunning = false, mainInterval = null;
        let defibCount = 0, epiCount = 0, rhythmHistory = [];

        function formatTime(s) {
            const m = Math.floor(s / 60).toString().padStart(2, '0');
            const sec = (s % 60).toString().padStart(2, '0');
            return `${m}:${sec}`;
        }

        function toggleStart() {
            const btn = document.getElementById('btn-start');
            if (!isRunning) {
                isRunning = true;
                btn.innerText = 'PAUSE'; btn.classList.replace('bg-emerald-600', 'bg-amber-600');
                if (totalSec === 0) recordAction('🚀 Code Blue Activated');
                mainInterval = setInterval(() => {
                    totalSec++; cprSec--;
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    document.getElementById('cpr-timer').innerText = formatTime(cprSec);
                    if (cprSec <= 0) { clearInterval(mainInterval); isRunning = false; alert("Check Rhythm!"); }
                }, 1000);
            } else {
                isRunning = false;
                btn.innerText = 'RESUME'; btn.classList.replace('bg-amber-600', 'bg-emerald-600');
                clearInterval(mainInterval);
            }
        }

        function setRhythm(type) {
            document.getElementById('guidance-text').innerHTML = type === 'Shockable' ? '<b>⚡ Shockable:</b> Shock 200J -> CPR 2m' : '<b>💉 Non-Shock:</b> Epi ASAP -> CPR 2m';
            rhythmHistory.push({ time: formatTime(totalSec), type });
            recordAction(`🔍 Rhythm: ${type}`);
            cprSec = 120;
        }

        function recordDefib() {
            defibCount++; document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ Defibrillation #${defibCount} (200J)`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++; document.getElementById('dash-epi').innerText = epiCount;
            recordAction(`💉 Epinephrine Dose #${epiCount} (1 mg)`);
        }

        function recordAction(msg) {
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm";
            row.innerHTML = `<td class="py-2 text-center bg-slate-100 font-mono">${formatTime(totalSec)}</td><td class="py-2 px-3">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
        }

        function closeCase() {
            isRunning = false; clearInterval(mainInterval);
            document.getElementById('sum-time').innerText = formatTime(totalSec);
            document.getElementById('sum-shock').innerText = defibCount;
            const timeline = document.getElementById('rhythm-timeline');
            timeline.innerHTML = rhythmHistory.map(h => `<div class="p-2 bg-slate-50 rounded border-l-2 border-slate-300 flex justify-between"><span>${h.type}</span><span>${h.time}</span></div>`).join('');
            document.getElementById('summary-modal').style.display = 'flex';
        }
    </script>
</body>
</html>
