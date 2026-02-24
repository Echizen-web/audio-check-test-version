<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AudioCheck Pro - 全功能流暢版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .sidebar-item-active { background: #1e293b; border-left: 4px solid #3b82f6; color: #f8fafc; }
        .checkbox-item:checked + span { text-decoration: line-through; color: #64748b; }
        /* 修正捲軸顯示 */
        .custom-scrollbar::-webkit-scrollbar { width: 6px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: #0f172a; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #334155; border-radius: 10px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #475569; }
    </style>
</head>
<body class="bg-gray-950 text-gray-100 font-sans flex h-screen overflow-hidden">

    <aside id="sidebar" class="w-64 bg-gray-900 border-r border-gray-800 flex flex-col shrink-0 transition-all duration-300 z-50">
        <div class="p-4 border-b border-gray-800 flex justify-between items-center">
            <span class="text-xl font-bold text-blue-400 italic">AudioCheck</span>
            <button onclick="toggleSidebar()" class="lg:hidden text-gray-400 text-xl">✕</button>
        </div>
        
        <div class="p-4">
            <button onclick="openNewProjectModal()" class="w-full bg-blue-600 hover:bg-blue-500 py-3 rounded-xl font-bold text-sm transition-all shadow-lg shadow-blue-900/20">
                + 新增案子
            </button>
        </div>

        <nav class="flex-1 overflow-y-auto custom-scrollbar px-2 space-y-1" id="projectList">
            </nav>

        <div class="p-4 border-t border-gray-800">
            <button onclick="showAdmin()" class="w-full text-left px-3 py-2 text-sm text-gray-400 hover:bg-gray-800 rounded-lg transition-all flex items-center gap-2">
                ⚙️ 管理範例與設定
            </button>
        </div>
    </aside>

    <main class="flex-1 flex flex-col min-w-0">
        <header class="lg:hidden bg-gray-900 p-4 border-b border-gray-800 flex items-center gap-4">
            <button onclick="toggleSidebar()" class="text-gray-400 text-2xl">☰</button>
            <span class="font-bold text-blue-400">AudioCheck</span>
        </header>

        <div id="mainContent" class="flex-1 overflow-y-auto p-6 lg:p-10 custom-scrollbar">
            </div>
    </main>

    <div id="projectModal" class="hidden fixed inset-0 bg-black/90 backdrop-blur-sm flex items-center justify-center p-4 z-[100]">
        <div class="bg-gray-800 border border-gray-700 rounded-2xl p-6 w-full max-w-2xl shadow-2xl max-h-[90vh] flex flex-col">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-2xl font-bold">建立新案子</h2>
                <button onclick="closeProjectModal()" class="text-gray-400 hover:text-white text-2xl">✕</button>
            </div>
            
            <div class="space-y-6 flex-1 overflow-y-auto pr-2 custom-scrollbar">
                <div>
                    <label class="text-xs text-gray-400 uppercase font-bold tracking-widest">案子名稱</label>
                    <input type="text" id="newProjName" class="w-full bg-gray-900 border border-gray-700 rounded-lg p-3 mt-1 focus:ring-2 focus:ring-blue-500 outline-none">
                </div>

                <div>
                    <label class="text-xs text-gray-400 uppercase font-bold tracking-widest">1. 選擇基礎範例</label>
                    <div class="grid grid-cols-2 md:grid-cols-3 gap-2 mt-2" id="templatePicker"></div>
                </div>

                <div id="templatePreview" class="hidden space-y-4 bg-gray-900 p-5 rounded-xl border border-gray-700">
                    <label class="text-xs text-amber-400 font-bold uppercase tracking-widest">2. 預覽並自定義本次清單</label>
                    <div id="previewList" class="space-y-6"></div>
                </div>
            </div>

            <div class="pt-6 flex gap-3">
                <button onclick="confirmCreateProject()" class="flex-1 bg-blue-600 hover:bg-blue-500 py-3 rounded-xl font-bold transition-all text-white">確認建立案子</button>
            </div>
        </div>
    </div>

    <div id="adminPanel" class="hidden fixed inset-0 bg-gray-950 z-[110] p-4 md:p-8 flex flex-col">
        <div class="max-w-4xl mx-auto w-full flex flex-col h-full bg-gray-900 p-6 rounded-2xl border border-gray-800 shadow-2xl">
            <div class="flex justify-between items-center mb-6">
                <div>
                    <h2 class="text-2xl font-bold text-blue-400">管理預設範例 (JSON)</h2>
                    <p class="text-xs text-gray-500">修改後的範例將永久保存在此裝置</p>
                </div>
                <button onclick="hideAdmin()" class="text-gray-400 hover:text-white text-2xl">✕</button>
            </div>
            <textarea id="jsonInputArea" class="flex-1 bg-black text-emerald-500 p-4 font-mono text-sm border border-gray-800 rounded-lg mb-4 outline-none focus:border-emerald-500 custom-scrollbar"></textarea>
            <div class="flex gap-3">
                <button onclick="saveJsonConfig()" class="flex-1 bg-emerald-600 hover:bg-emerald-500 py-3 rounded-xl font-bold transition-all">儲存變更</button>
                <button onclick="resetToDefault()" class="px-6 py-3 border border-gray-700 text-gray-400 hover:bg-gray-800 rounded-xl transition-all">還原初始</button>
            </div>
        </div>
    </div>

    <script>
        // --- 初始化資料 ---
        const initialDefault = {
            templates: {
                "樂團演出": { pre: ["鼓組收音", "DI Box 數量確認"], mid: ["Line Check", "Monitor 平衡"], post: ["線材收納"] },
                "小型講座": { pre: ["麥克風電池", "PPT 音軌測試"], mid: ["主持人音量監控"], post: ["器材歸庫"] }
            }
        };

        let config = JSON.parse(localStorage.getItem('audio_config')) || initialDefault;
        let projects = JSON.parse(localStorage.getItem('audio_projects')) || [];
        let activeId = projects.length > 0 ? projects[0].id : null;
        let selectedItems = { pre: [], mid: [], post: [] };

        function init() {
            renderSidebar();
            renderContent();
            renderTemplatePicker();
            document.getElementById('jsonInputArea').value = JSON.stringify(config, null, 4);
        }

        // --- 側欄管理 ---
        function renderSidebar() {
            const list = document.getElementById('projectList');
            list.innerHTML = projects.map(p => `
                <div class="group flex items-center justify-between rounded-xl transition-all mb-1 ${activeId === p.id ? 'sidebar-item-active' : 'hover:bg-gray-800/50 text-gray-400 hover:text-gray-200'}">
                    <button onclick="switchProject(${p.id})" class="flex-1 text-left px-4 py-3 text-sm font-medium truncate">${p.name}</button>
                    <button onclick="deleteProject(${p.id})" class="opacity-0 group-hover:opacity-100 p-3 text-gray-500 hover:text-red-500">✕</button>
                </div>
            `).join('');
        }

        function switchProject(id) { activeId = id; renderSidebar(); renderContent(); if(window.innerWidth < 1024) toggleSidebar(); }

        // --- 案子建立流程 ---
        function openNewProjectModal() {
            document.getElementById('projectModal').classList.remove('hidden');
            document.getElementById('newProjName').value = `案子 ${new Date().toLocaleDateString()}`;
            document.getElementById('templatePreview').classList.add('hidden');
        }
        function closeProjectModal() { document.getElementById('projectModal').classList.add('hidden'); }

        function renderTemplatePicker() {
            const picker = document.getElementById('templatePicker');
            picker.innerHTML = Object.keys(config.templates).map(name => `
                <button onclick="applyTemplateToPreview('${name}')" class="p-3 bg-gray-900 border border-gray-700 rounded-lg hover:border-blue-500 text-xs transition-all text-gray-300">
                    ${name}
                </button>
            `).join('');
        }

        function applyTemplateToPreview(name) {
            const temp = config.templates[name];
            selectedItems = {
                pre: temp.pre.map(t => ({ text: t, active: true })),
                mid: temp.mid.map(t => ({ text: t, active: true })),
                post: temp.post.map(t => ({ text: t, active: true }))
            };
            document.getElementById('templatePreview').classList.remove('hidden');
            renderPreviewList();
        }

        function renderPreviewList() {
            const list = document.getElementById('previewList');
            let html = "";
            ['pre', 'mid', 'post'].forEach(k => {
                const label = k==='pre'?'行前':k==='mid'?'行中':'行後';
                html += `<div class="bg-gray-800/50 p-3 rounded-lg border border-gray-700">
                    <div class="flex justify-between items-center mb-3">
                        <p class="text-[11px] text-blue-400 font-black uppercase tracking-widest">${label}</p>
                        <button onclick="addPreviewItem('${k}')" class="text-xs bg-gray-700 hover:bg-gray-600 px-2 py-0.5 rounded text-gray-300 transition-all">+ 新增</button>
                    </div>`;
                selectedItems[k].forEach((item, i) => {
                    html += `<div class="flex items-center gap-3 mb-2 group">
                        <input type="checkbox" ${item.active?'checked':''} onchange="selectedItems.${k}[${i}].active=this.checked" class="w-4 h-4 accent-blue-500">
                        <input type="text" value="${item.text}" oninput="selectedItems.${k}[${i}].text=this.value" class="bg-transparent text-sm flex-1 border-b border-gray-700 focus:border-blue-500 outline-none p-0 text-gray-200">
                        <button onclick="removePreviewItem('${k}', ${i})" class="text-gray-600 hover:text-red-500 opacity-0 group-hover:opacity-100 transition-all">✕</button>
                    </div>`;
                });
                html += `</div>`;
            });
            list.innerHTML = html;
        }

        function addPreviewItem(k) { selectedItems[k].push({ text: "新項目", active: true }); renderPreviewList(); }
        function removePreviewItem(k, i) { selectedItems[k].splice(i, 1); renderPreviewList(); }

        function confirmCreateProject() {
            const newProj = {
                id: Date.now(),
                name: document.getElementById('newProjName').value,
                checklist: {
                    pre: selectedItems.pre.filter(i=>i.active).map(i=>({text:i.text, done:false})),
                    mid: selectedItems.mid.filter(i=>i.active).map(i=>({text:i.text, done:false})),
                    post: selectedItems.post.filter(i=>i.active).map(i=>({text:i.text, done:false}))
                }
            };
            projects.unshift(newProj);
            activeId = newProj.id;
            saveData(); closeProjectModal(); renderSidebar(); renderContent();
        }

        // --- 主畫面渲染 ---
        function renderContent() {
            const container = document.getElementById('mainContent');
            const p = projects.find(proj => proj.id === activeId);
            if (!p) { container.innerHTML = `<div class="h-full flex flex-col items-center justify-center text-gray-600 italic">
                <div class="text-6xl mb-4">🎚️</div>
                請在側邊欄選擇或建立一個新的音訊案子
            </div>`; return; }

            container.innerHTML = `
                <div class="max-w-6xl mx-auto pb-20">
                    <div class="flex flex-col md:flex-row md:items-end justify-between gap-4 mb-12">
                        <h1 class="text-4xl md:text-5xl font-black text-white tracking-tighter">${p.name}</h1>
                        <p class="text-gray-500 text-sm font-mono">ID: ${p.id}</p>
                    </div>
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                        ${renderColumn('行前準備', 'pre', p.checklist.pre)}
                        ${renderColumn('現場執行', 'mid', p.checklist.mid)}
                        ${renderColumn('撤場歸庫', 'post', p.checklist.post)}
                    </div>
                </div>`;
        }

        function renderColumn(title, key, items) {
            return `
                <div class="flex flex-col bg-gray-900/30 p-1 rounded-2xl">
                    <h3 class="text-[11px] font-black text-blue-500 mb-5 flex justify-between uppercase tracking-[0.2em] px-2">
                        ${title} <span class="bg-blue-950 text-blue-400 px-2 py-0.5 rounded-full text-[9px]">${items.filter(i=>i.done).length}/${items.length}</span>
                    </h3>
                    <div class="space-y-2 mb-6">
                        ${items.map((it, idx) => `
                            <div class="flex items-start gap-4 bg-gray-900 border border-gray-800 p-4 rounded-2xl hover:border-gray-600 transition-all cursor-pointer group" onclick="toggleItem('${key}', ${idx})">
                                <div class="w-6 h-6 rounded-lg border-2 border-gray-700 flex-shrink-0 flex items-center justify-center transition-all ${it.done?'bg-blue-600 border-blue-600':'group-hover:border-blue-500'}">
                                    ${it.done?'<svg class="w-4 h-4 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"></path></svg>':''}
                                </div>
                                <span class="text-sm font-medium leading-relaxed ${it.done?'text-gray-600 line-through':'text-gray-200'}">${it.text}</span>
                            </div>
                        `).join('')}
                    </div>
                    <div class="px-2">
                        <input type="text" placeholder="+ 快速新增項目..." onkeypress="if(event.key==='Enter') addItem('${key}', this)" 
                            class="w-full bg-gray-800/50 border border-gray-800 rounded-xl px-4 py-3 text-xs focus:border-blue-500 focus:bg-gray-800 outline-none transition-all text-gray-300">
                    </div>
                </div>`;
        }

        // --- 工具 ---
        function toggleItem(key, idx) { projects.find(p=>p.id===activeId).checklist[key][idx].done = !projects.find(p=>p.id===activeId).checklist[key][idx].done; saveData(); renderContent(); }
        function addItem(key, input) { if(input.value.trim()){ projects.find(p=>p.id===activeId).checklist[key].push({text:input.value, done:false}); input.value=''; saveData(); renderContent(); } }
        function deleteProject(id) { if(confirm('確定要永久刪除此案子嗎？')){ projects=projects.filter(p=>p.id!==id); activeId=projects.length?projects[0].id:null; saveData(); renderSidebar(); renderContent(); } }
        function saveData() { localStorage.setItem('audio_projects', JSON.stringify(projects)); localStorage.setItem('audio_config', JSON.stringify(config)); }
        function saveJsonConfig() { try { config = JSON.parse(document.getElementById('jsonInputArea').value); saveData(); renderTemplatePicker(); hideAdmin(); alert('範例已成功更新並儲存！'); } catch(e){ alert('JSON 格式錯誤，請檢查符號'); } }
        function resetToDefault() { if(confirm('確定還原為原始範例？')){ config=initialDefault; document.getElementById('jsonInputArea').value=JSON.stringify(config,null,4); saveData(); renderTemplatePicker(); } }
        function showAdmin() { document.getElementById('adminPanel').classList.remove('hidden'); }
        function hideAdmin() { document.getElementById('adminPanel').classList.add('hidden'); }
        function toggleSidebar() { document.getElementById('sidebar').classList.toggle('-ml-64'); }

        init();
    </script>
</body>
</html>
