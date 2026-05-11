# Recorder CPR
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ACLS 2025 Smart Recorder</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @keyframes pulse-red {
            0%, 100% { background-color: rgba(220, 38, 38, 0.1); }
            50% { background-color: rgba(220, 38, 38, 0.6); }
        }
        .timer-critical { animation: pulse-red 0.5s infinite; border-color: #ef4444 !important; color: white !important; }
        body { 
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; 
            -webkit-tap-highlight-color: transparent;
            overscroll-behavior-y: contain; /* ป้องกันการ pull-to-refresh บนมือถือ */
        }
        /* ล็อคความสูงของ Log และทำให้เลื่อนได้ */
        .log-container {
            flex: 1;
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
        }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            word-break: break-word; min-height: 55px; padding: 4px 8px; line-height: 1.1;
        }
        .modal-overlay { display: none; position: fixed; inset: 0; z-index: 100; background: rgba(0,0,0,0.85); backdrop-filter: blur(6px); align-items: center; justify-content: center; padding: 1rem; }
    </style>
</head>
<body class="bg-slate-900 h-screen flex flex-col overflow-hidden">

    <div id="summary-modal" class="modal-overlay">
        <div class="bg-white rounded-3xl p-6 w-full max-w-md shadow-2xl flex flex-col max-h-[90vh]">
            <h2 class="text-2xl font-black text-slate-800 mb-4 border-b pb-2 uppercase">Case Summary</h2>
            <div class="overflow-y-auto no-scrollbar mb-4">
                <div class="grid grid-cols-2 gap-3 mb-6">
                    <div class="bg-slate-100 p-3 rounded-2xl text-center"><p class="text-[10px] font-bold text-slate-500 uppercase">Duration</p><p id="sum-time" class="text-xl font-black">00:00</p></div>
                    <div class="bg-rose-100 p-3 rounded-2xl text-center"><p class="text-[10px] font-bold text-rose-500 uppercase">Shocks</p><p id="sum-shock" class="text-xl font-black text-rose-600">0</p></div>
                    <div class="bg-blue-100 p-3 rounded-2xl text-center"><p class="text-[10px] font-bold text-blue-500 uppercase">Epinephrine</p><p id="sum-epi" class="text-xl font-black text-blue-600">0</p></div>
                    <div class="bg-purple-100 p-3 rounded-2xl text-center"><p class="text-[10px] font-bold text-purple-500 uppercase">Amiodarone</p><p id="sum-amio" class="text-xl font-black text-purple-600">0</p></div>
                </div>
                <h3 class="text-xs font-black text-slate-400 mb-2 uppercase">Rhythm Timeline</h3>
                <div id="rhythm-timeline" class="space-y-2"></div>
            </div>
            <button onclick="document.getElementById('summary-modal').style.display='none'" class="w-full bg-slate-800 text-white font-black py-4 rounded-2xl">Close</button>
        </div>
    </div>

    <div class="shrink-0 p-4 bg-slate-900 border-b border-slate-800">
        <div class="flex justify-between items-start mb-4">
            <div>
                <h1 class="text-xl font-black text-red-500 leading-none">ACLS 2025</h1>
                <p class="text-[10px] text-slate-400 uppercase font-bold tracking-widest mt-1">Nirun Suwannachot</p>
            </div>
            <div class="text-right">
                <p class="text-[10px] text-slate-500 uppercase font-bold">Total Time</p>
                <p id="total-timer" class="text-2xl font-mono font-bold text-emerald-400">00:00</p>
            </div>
        </div>

        <div class="grid grid-cols-2 gap-3 mb-4">
            <div id="cpr-card" class="bg-slate-800 p-3 rounded-xl border-2 border-blue-500 text-center">
                <p class="text-[10px] text-blue-400 font-bold uppercase mb-1">Rhythm Check In</p>
                <p id="cpr-timer" class="text-4xl font-mono font-black text-white">02:00</p>
            </div>
            <div class="bg-slate-800 p-2 rounded-xl border border-slate-700 overflow-hidden">
                <p class="text-[9px] text-yellow-500 font-bold uppercase mb-1">Algorithm Guide</p>
                <div id="guidance-text" class="text-[10px] text-slate-400 italic leading-tight">Standby...</div>
            </div>
        </div>

        <div class="grid grid-cols-3 gap-2">
            <button onclick="toggleStart()" id="btn-start" class="bg-emerald-600 py-3 rounded-lg font-black text-xs text-white uppercase shadow-lg active:scale-95">Start</button>
            <button onclick="setRhythm('Shockable')" class="bg-rose-700 py-3 rounded-lg font-black text-xs text-white uppercase active:scale-95">Shockable</button>
            <button onclick="setRhythm('Non-Shockable')" class="bg-blue-700 py-3 rounded-lg font-black text-xs text-white uppercase active:scale-95">Non-Shock</button>
        </div>
    </div>

    <div class="shrink-0 p-3 bg-white border-b border-slate-200">
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordAction('💓 Start CPR')" class="drug-btn bg-emerald-50 border-2 border-emerald-500 text-emerald-700 rounded-lg text-[10px] font-black uppercase">Start CPR</button>
            <button onclick="recordAction('💉 Access IV/IO')" class="drug-btn bg-sky-50 border-2 border-sky-600 text-sky-700 rounded-lg text-[10px] font-black uppercase">Access IV/IO</button>
            <button onclick="recordAction('💧 IV Fluid')" class="drug-btn bg-blue-50 border-2 border-blue-400 text-blue-700 rounded-lg text-[10px] font-black uppercase">IV Fluid</button>
        </div>
        <div class="grid grid-cols-3 gap-2 mb-2">
            <button onclick="recordDefib()" class="drug-btn bg-rose-50 border-2 border-rose-500 text-rose-600 rounded-lg text-[10px] font-black uppercase italic tracking-tighter">Defib 200J</button>
            <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white rounded-lg text-[10px] font-black uppercase shadow-md active:bg-blue-700">Epinephrine</button>
            <button onclick="recordAmio()" class="drug-btn bg-white border-2 border-purple-500 text-purple-600 rounded-lg text-[10px] font-black uppercase">Amiodarone</button>
        </div>
        <div class="grid grid-cols-2 gap-2">
            <button onclick="recordAction('💊 Amiodarone 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700 rounded-lg text-[9px] font-bold uppercase">Amio 150</button>
            <button onclick="recordAction('🌬️ Consider Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-300 text-orange-700 rounded-lg text-[9px] font-bold uppercase italic">Adv. Airway</button>
        </div>
    </div>

    <div class="log-container bg-slate-50 relative">
        <table class="w-full text-left border-separate border-spacing-y-1 px-4">
            <thead class="sticky top-0 bg-slate-50 z-10">
                <tr class="text-[10px] text-slate-400 uppercase font-black"><th class="py-2 w-16 text-center">Time</th><th class="py-2 px-2">Action Log</th></tr>
            </thead>
            <tbody id="log-body" class="text-sm"></tbody>
        </table>

        <div class="fixed bottom-20 right-4 flex flex-col gap-2 pointer-events-none">
            <div class="bg-rose-600 text-white px-3 py-1 rounded-xl shadow-xl flex flex-col items-center border-2 border-white">
                <span class="text-[8px] font-bold uppercase">Shock</span>
                <span id="dash-shock" class="text-xl font-black leading-none">0</span>
            </div>
            <div class="bg-blue-600 text-white px-3 py-1 rounded-xl shadow-xl flex flex-col items-center border-2 border-white">
                <span class="text-[8px] font-bold uppercase">Epi</span>
                <span id="dash-epi" class="text-xl font-black leading-none">0</span>
            </div>
        </div>
    </div>

    <div class="shrink-0 p-3 bg-white border-t flex justify-between gap-4">
        <button onclick="recordAction('🏁 ROSC Achieved')" class="flex-1 bg-emerald-500 text-white py-3 rounded-xl font-black text-xs uppercase shadow-md">ROSC</button>
        <button id="btn-close-case" onclick="closeCase()" class="flex-1 bg-slate-800 text-white py-3 rounded-xl font-black text-xs uppercase shadow-md">Close Case</button>
    </div>

    <div id="alert-modal" class="modal-overlay">
        <div class="bg-white rounded-3xl p-8 w-full max-w-sm text-center shadow-2xl border-4 border-red-500">
            <h2 class="text-3xl font-black text-slate-900 mb-2 uppercase italic">Time Up!</h2>
            <p class="text-lg font-bold text-slate-600 mb-6">CHECK PULSE / EKG</p>
            <button onclick="closeModal()" class="w-full bg-red-600 text-white font-black py-4 rounded-2xl text-xl">Resume CPR</button>
        </div>
    </div>

    <script>
        let totalSec = 0, cprSec = 120, isRunning = false, mainInterval = null;
        let rhythmCheckCount = 0, defibCount = 0, epiCount = 0, amioCount = 0;
        let isCaseClosed = false, rhythmHistory = [];

        const ALGO_TEXTS = {
            'Shockable': `<b class="text-rose-400">⚡ VF/pVT:</b><br>1. SHOCK 200J<br>2. CPR 2m<br>3. Epi 3-5m<br>4. Amio/Lido`,
            'Non-Shockable': `<b class="text-blue-400">💉 PEA/Asy:</b><br>1. CPR 2m<br>2. EPI ASAP<br>3. Epi 3-5m<br>4. H's & T's`
        };

        function formatTime(s) {
            const m = Math.floor(s / 60).toString().padStart(2, '0');
            const sec = (s % 60).toString().padStart(2, '0');
            return `${m}:${sec}`;
        }

        function toggleStart() {
            if (isCaseClosed) return;
            const btn = document.getElementById('btn-start');
            if (!isRunning) {
                isRunning = true;
                btn.innerText = 'Pause';
                btn.classList.replace('bg-emerald-600', 'bg-amber-600');
                if (totalSec === 0) recordAction('🚀 Code Blue Activated');
                if (mainInterval) clearInterval(mainInterval);
                mainInterval = setInterval(ticking, 1000);
            } else {
                isRunning = false;
                btn.innerText = 'Resume';
                btn.classList.replace('bg-amber-600', 'bg-emerald-600');
                clearInterval(mainInterval);
                mainInterval = null;
            }
        }

        function ticking() {
            totalSec++;
            cprSec--;
            document.getElementById('total-timer').innerText = formatTime(totalSec);
            document.getElementById('cpr-timer').innerText = formatTime(cprSec);
            if (cprSec <= 10 && cprSec > 0) document.getElementById('cpr-card').classList.add('timer-critical');
            else document.getElementById('cpr-card').classList.remove('timer-critical');
            if (cprSec <= 0) {
                clearInterval(mainInterval);
                mainInterval = null;
                document.getElementById('alert-modal').style.display = 'flex';
            }
        }

        function setRhythm(type) {
            if (isCaseClosed) return;
            document.getElementById('guidance-text').innerHTML = ALGO_TEXTS[type];
            rhythmHistory.push({ time: formatTime(totalSec), type: type });
            recordAction(`🔍 Rhythm: ${type}`);
            resetCprCycle();
            if (!isRunning) toggleStart(); 
        }

        function recordDefib() {
            if (isCaseClosed) return;
            defibCount++;
            document.getElementById('dash-shock').innerText = defibCount;
            recordAction(`⚡ Defibrillation #${defibCount} (200J)`);
            resetCprCycle();
            if (!isRunning) toggleStart();
        }

        function recordEpi() {
            if (isCaseClosed) return;
            epiCount++;
            document.getElementById('dash-epi').innerText = epiCount;
            recordAction(`💉 Epinephrine Dose #${epiCount} (1 mg)`);
        }

        function recordAmio() {
            if (isCaseClosed) return;
            amioCount++;
            recordAction(`💊 Amiodarone #${amioCount} (300 mg)`);
        }

        function resetCprCycle() {
            cprSec = 120;
            document.getElementById('cpr-timer').innerText = formatTime(cprSec);
            document.getElementById('cpr-card').classList.remove('timer-critical');
            document.getElementById('alert-modal').style.display = 'none';
        }

        function closeModal() {
            document.getElementById('alert-modal').style.display = 'none';
            cprSec = 120;
            if (isRunning && !isCaseClosed) {
                if (mainInterval) clearInterval(mainInterval);
                mainInterval = setInterval(ticking, 1000);
            }
        }

        function recordAction(msg) {
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 mb-1 shadow-sm";
            row.innerHTML = `<td class="py-2.5 font-mono text-[11px] text-center font-bold bg-slate-100">${formatTime(totalSec)}</td><td class="py-2.5 px-3 font-bold text-slate-800">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
        }

        function closeCase() {
            if (totalSec === 0) return;
            isRunning = false;
            clearInterval(mainInterval);
            isCaseClosed = true;
            recordAction('🏁 CASE CLOSED');
            document.getElementById('sum-time').innerText = formatTime(totalSec);
            document.getElementById('sum-shock').innerText = defibCount;
            document.getElementById('sum-epi').innerText = epiCount;
            document.getElementById('sum-amio').innerText = amioCount;

            const timeline = document.getElementById('rhythm-timeline');
            timeline.innerHTML = rhythmHistory.map(h => 
                `<div class="flex justify-between p-2 rounded-lg text-xs font-bold ${h.type === 'Shockable' ? 'bg-rose-50 text-rose-600' : 'bg-blue-50 text-blue-600'}">
                    <span>${h.type}</span><span>${h.time}</span>
                </div>`).join('');

            document.getElementById('btn-close-case').innerText = "Summary";
            document.getElementById('btn-close-case').onclick = () => document.getElementById('summary-modal').style.display='flex';
            document.getElementById('summary-modal').style.display = 'flex';
        }
    </script>
</body>
</html>
