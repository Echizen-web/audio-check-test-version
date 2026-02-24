<測試版本>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AudioCheck Pro - 側欄管理版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .sidebar-item-active { background: #1e293b; border-left: 4px solid #3b82f6; color: #f8fafc; }
        .checkbox-item:checked + span { text-decoration: line-through; color: #64748b; }
        /* 隱藏捲軸但保持功能 */
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="bg-gray-950 text-gray-100 font-sans flex h-screen overflow-hidden">

    <aside id="sidebar" class="w-64 bg-gray-900 border-r border-gray-800 flex flex-col shrink-0 transition-all duration-300">
        <div class="p-4 border-b border-gray-800 flex justify-between items-center">
            <span class="text-xl font-bold text-blue-400 italic">AudioCheck</span>
            <button onclick="toggleSidebar()" class="lg:hidden text-gray-400">✕</button>
        </div>
        
        <div class="p-4">
            <button onclick="openNewProjectModal()" class="w-full bg-blue-600 hover:bg-blue-500 py-2 rounded-lg font-bold text-sm transition-all mb-4">
                + 新增案子
            </button>
        </div>

        <nav class="flex-1 overflow-y-auto no-scrollbar px-2 space-y-1" id="projectList">
            </nav>

        <div class="p-4 border-t border-gray-800 space-y-2">
            <button onclick="showAdmin()" class="w-full text-left px-3 py-2 text-sm text-gray-400 hover:bg-gray-800 rounded-lg transition-all flex items-center gap-2">
                ⚙️ 管理範例與設定
            </button>
        </div>
    </aside>

    <main class="flex-1 flex flex-col overflow-hidden relative">
        <header class="lg:hidden bg-gray-900 p-4 border-b border-gray-800 flex items-center gap-4">
            <button onclick="toggleSidebar()" class="text-gray-400 text-2xl">☰</button>
            <span class="font-bold">案子清單</span>
        </header>

        <div id="mainContent" class="flex-1 overflow-y-auto p-6 lg:p-10 no-scrollbar">
            </div>
    </main>

    <div id="projectModal" class="hidden fixed inset-0 bg-black/90 backdrop-blur-sm flex items-center justify-center p-4 z-[100]">
        <div class="bg-gray-800 border border-gray-700 rounded-2xl p-6 w-full max-w-2xl shadow-2xl max-h-[90vh] flex flex-col">
            <h2 class="text-2xl font-bold mb-4">建立新案子</h2>
            
            <div class="space-y-4 flex-1 overflow-y-auto pr-2 no-scrollbar">
                <div>
                    <label class="text-xs text-gray-400 uppercase font-bold">案子名稱</label>
                    <input type="text" id="newProjName" class="w-full bg-gray-900 border border-gray-700 rounded-lg p-3 focus:ring-2 focus:ring-blue-500 outline-none">
                </div>

                <div>
                    <label class="text-xs text-gray-400 uppercase font-bold">選擇初始範例</label>
                    <div class="grid grid-cols-2 gap-2 mt-2" id="templatePicker"></div>
                </div>

                <div id="templatePreview" class="hidden space-y-4 bg-gray-900 p-4 rounded-xl border border-gray-700">
                    <p class="text-xs text-amber-400 font-bold">預覽並調整本次清單 (點擊文字可修改)</p>
                    <div id="previewList" class="space-y-4"></div>
                </div>
            </div>

            <div class="pt-6 flex gap-3">
                <button onclick="confirmCreateProject()" class="flex-1 bg-blue-600 hover:bg-blue-500 py-3 rounded-xl font-bold transition-all">確認建立案子</button>
                <button onclick="closeProjectModal()" class="px-6 py-3 text-gray-400 hover:text-white">取消</button>
            </div>
        </div>
    </div>

    <div id="adminPanel" class="hidden fixed inset-0 bg-gray-950 z-[110] p-8 flex flex-col">
        <div class="max-w-4xl mx-auto w-full flex flex-col h-full">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold text-blue-400">管理預設範例 (JSON)</h2>
                <button onclick="hideAdmin()" class="text-gray-400 hover:text-white">✕ 關閉</button>
            </div>
            <p class="text-gray-400 mb-4 text-sm">在這裡編輯的內容將成為未來「新案子」的預設範本：</p>
            <textarea id="jsonInputArea" class="flex-1 bg-black text-emerald-500 p-4 font-mono text-sm border border-gray-700 rounded-lg mb-4 outline-none focus:border-emerald-500"></textarea>
            <button onclick="saveJsonConfig()" class="bg-emerald-600 hover:bg-emerald-500 py-3 rounded-xl font-bold transition-all">儲存範例設定</button>
        </div>
    </div>

    <script>
        // --- 初始配置 ---
        const defaultConfig = {
            templates: {
                "樂團演出": { pre: ["鼓組收音組件", "DI Box 數量確認", "舞台電源分配"], mid: ["Line Check", "樂手 Monitor 平衡", "FOH 調音"], post: ["線材分類收納", "器材損壞標註"] },
                "小型講座": { pre: ["麥克風電池", "PPT 音軌測試"], mid: ["主持人音量監控"], post: ["器材歸庫"] }
            }
        };

        let config = JSON.parse(localStorage.getItem('audio_config')) || defaultConfig;
        let projects = JSON.parse(localStorage.getItem('audio_projects')) || [];
        let activeId = projects.length > 0 ? projects[0].id : null;
        let selectedItems = { pre: [], mid: [], post: [] };

        function init() {
            renderSidebar();
            renderContent();
            renderTemplatePicker();
            document.getElementById('jsonInputArea').value = JSON.stringify(config, null, 4);
        }

        // --- 側邊欄與案子切換 ---
        function renderSidebar() {
            const list = document.getElementById('projectList');
            list.innerHTML = projects.map(p => `
                <div class="group flex items-center justify-between rounded-lg transition-all ${activeId === p.id ? 'sidebar-item-active' : 'hover:bg-gray-800 text-gray-400 hover:text-gray-200'}">
                    <button onclick="switchProject(${p.id})" class="flex-1 text-left px-3 py-3 text-sm font-medium truncate">
                        ${p.name}
                    </button>
                    <button onclick="deleteProject(${p.id})" class="opacity-0 group-hover:opacity-100 p-3 text-gray-500 hover:text-red-500 transition-all text-xs">✕</button>
                </div>
            `).join('');
        }

        function switchProject(id) {
            activeId = id;
            renderSidebar();
            renderContent();
            if (window.innerWidth < 1024) toggleSidebar(); // 手機端點擊後收合
        }

        // --- 建立案子邏輯 ---
        function openNewProjectModal() {
            document.getElementById('projectModal').classList.remove('hidden');
            document.getElementById('newProjName').value = `案子 ${new Date().toLocaleDateString()}`;
        }

        function closeProjectModal() { document.getElementById('projectModal').classList.add('hidden'); }

        function renderTemplatePicker() {
            const picker = document.getElementById('templatePicker');
            picker.innerHTML = Object.keys(config.templates).map(name => `
                <button onclick="applyTemplateToPreview('${name}')" class="p-3 bg-gray-900 border border-gray-700 rounded-lg hover:border-blue-500 text-xs transition-all">
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
            const preview = document.getElementById('templatePreview');
            preview.classList.remove('hidden');
            renderPreviewList();
        }

        function renderPreviewList() {
            const list = document.getElementById('previewList');
            let html = "";
            ['pre', 'mid', 'post'].forEach(k => {
                const label = k==='pre'?'行前':k==='mid'?'行中':'行後';
                html += `<div><p class="text-[10px] text-gray-500 mb-1 font-bold">${label}</p>`;
                selectedItems[k].forEach((item, i) => {
                    html += `<div class="flex items-center gap-2 mb-1">
                        <input type="checkbox" checked onchange="selectedItems.${k}[${i}].active=this.checked" class="accent-blue-500">
                        <input type="text" value="${item.text}" onchange="selectedItems.${k}[${i}].text=this.value" class="bg-transparent text-sm flex-1 border-none focus:ring-0 p-0 text-gray-300">
                    </div>`;
                });
                html += `</div>`;
            });
            list.innerHTML = html;
        }

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
            projects.unshift(newProj); // 新案子放在最上面
            activeId = newProj.id;
            saveData(); closeProjectModal(); renderSidebar(); renderContent();
        }

        // --- 主內容渲染 ---
        function renderContent() {
            const container = document.getElementById('mainContent');
            const p = projects.find(proj => proj.id === activeId);
            if (!p) { container.innerHTML = `<div class="h-full flex items-center justify-center text-gray-600 italic">請在側邊欄選擇或建立案子</div>`; return; }

            container.innerHTML = `
                <div class="max-w-5xl mx-auto">
                    <h1 class="text-4xl font-bold mb-10 text-white">${p.name}</h1>
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                        ${renderColumn('行前準備', 'pre', p.checklist.pre)}
                        ${renderColumn('現場執行', 'mid', p.checklist.mid)}
                        ${renderColumn('撤場歸庫', 'post', p.checklist.post)}
                    </div>
                </div>`;
        }

        function renderColumn(title, key, items) {
            return `
                <div class="flex flex-col">
                    <h3 class="text-sm font-bold text-blue-400 mb-4 flex justify-between uppercase tracking-widest">
                        ${title} <span>${items.filter(i=>i.done).length}/${items.length}</span>
                    </h3>
                    <div class="space-y-2 mb-4">
                        ${items.map((it, idx) => `
                            <div class="flex items-start gap-3 bg-gray-900/50 border border-gray-800 p-3 rounded-xl hover:bg-gray-800 transition-all cursor-pointer" onclick="toggleItem('${key}', ${idx})">
                                <input type="checkbox" class="w-5 h-5 mt-0.5 accent-blue-500 pointer-events-none" ${it.done?'checked':''}>
                                <span class="text-sm ${it.done?'text-gray-500 line-through':'text-gray-300'}">${it.text}</span>
                            </div>
                        `).join('')}
                    </div>
                    <input type="text" placeholder="+ 新增..." onkeypress="if(event.key==='Enter') addItem('${key}', this)" 
                        class="bg-transparent border border-gray-800 rounded-lg px-3 py-2 text-xs focus:border-blue-500 outline-none transition-all text-gray-400">
                </div>`;
        }

        // --- 工具功能 ---
        function toggleItem(key, idx) { projects.find(p=>p.id===activeId).checklist[key][idx].done = !projects.find(p=>p.id===activeId).checklist[key][idx].done; saveData(); renderContent(); }
        function addItem(key, input) { if(input.value.trim()){ projects.find(p=>p.id===activeId).checklist[key].push({text:input.value, done:false}); input.value=''; saveData(); renderContent(); } }
        function deleteProject(id) { if(confirm('確定刪除？')){ projects=projects.filter(p=>p.id!==id); activeId=projects.length?projects[0].id:null; saveData(); renderSidebar(); renderContent(); } }
        function saveData() { localStorage.setItem('audio_projects', JSON.stringify(projects)); localStorage.setItem('audio_config', JSON.stringify(config)); }
        function saveJsonConfig() { try { config = JSON.parse(document.getElementById('jsonInputArea').value); saveData(); renderTemplatePicker(); hideAdmin(); alert('範例已永久儲存！'); } catch(e){ alert('JSON 格式錯誤'); } }
        function showAdmin() { document.getElementById('adminPanel').classList.remove('hidden'); }
        function hideAdmin() { document.getElementById('adminPanel').classList.add('hidden'); }
        function toggleSidebar() { document.getElementById('sidebar').classList.toggle('-ml-64'); }

        init();
    </script>
</body>
</html>
