<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AudioCheck Pro - 靈活編輯版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .tab-active { border-bottom: 4px solid #3b82f6; color: #3b82f6; background: rgba(59, 130, 246, 0.1); }
        .checkbox-item:checked + span { text-decoration: line-through; color: #9ca3af; }
        .drag-ghost { opacity: 0.5; background: #2d3748; }
    </style>
</head>
<body class="bg-gray-900 text-gray-100 min-h-screen font-sans">

    <nav class="bg-gray-800 border-b border-gray-700 p-4 sticky top-0 z-50 flex justify-between items-center">
        <div class="flex items-center gap-6 overflow-x-auto" id="tabBar">
            <span class="text-xl font-bold text-blue-400 mr-4 italic">AudioCheck</span>
        </div>
        <div class="flex gap-2 shrink-0">
            <button onclick="showAdmin()" class="bg-gray-700 hover:bg-gray-600 px-3 py-1 rounded text-sm border border-gray-600">⚙️ 設定</button>
            <button onclick="openNewProjectModal()" class="bg-blue-600 hover:bg-blue-500 px-4 py-1 rounded text-sm font-bold shadow-lg">+ 新案子</button>
        </div>
    </nav>

    <main id="mainContent" class="p-6 max-w-7xl mx-auto">
        </main>

    <div id="projectModal" class="hidden fixed inset-0 bg-black/80 backdrop-blur-sm flex items-center justify-center p-4 z-[100]">
        <div class="bg-gray-800 border border-gray-700 rounded-2xl p-6 w-full max-w-2xl shadow-2xl max-h-[90vh] overflow-y-auto">
            <h2 class="text-2xl font-bold mb-4 text-white">建立新案子</h2>
            
            <label class="block text-sm text-gray-400 mb-2">案子名稱</label>
            <input type="text" id="newProjName" class="w-full bg-gray-900 border border-gray-700 rounded-lg p-3 mb-6 focus:border-blue-500 outline-none">

            <label class="block text-sm text-gray-400 mb-2">選擇範例並調整內容</label>
            <div class="grid grid-cols-2 md:grid-cols-3 gap-2 mb-6" id="templatePicker">
                </div>

            <div id="templatePreview" class="bg-gray-900 rounded-xl p-4 mb-6 hidden border border-gray-700">
                <p class="text-xs text-blue-400 mb-3 font-bold uppercase">預覽待建立項目 (可編輯文字或勾選排除)</p>
                <div id="previewList" class="space-y-2 text-sm"></div>
            </div>

            <div class="flex gap-3">
                <button onclick="confirmCreateProject()" class="flex-1 bg-blue-600 hover:bg-blue-500 py-3 rounded-xl font-bold transition-all">確認建立案子</button>
                <button onclick="closeProjectModal()" class="px-6 py-3 text-gray-400 hover:text-white transition-all">取消</button>
            </div>
        </div>
    </div>

    <div id="adminPanel" class="hidden fixed inset-0 bg-gray-950 z-[110] p-8 overflow-y-auto">
        <div class="max-w-4xl mx-auto">
            <h2 class="text-2xl font-bold text-blue-400 mb-6">管理員設定</h2>
            <textarea id="jsonInputArea" class="w-full h-96 bg-black text-emerald-500 p-4 font-mono text-sm border border-gray-700 rounded-lg mb-4"></textarea>
            <button onclick="saveJsonConfig()" class="bg-emerald-600 px-6 py-2 rounded-lg font-bold">更新範例資料</button>
            <button onclick="hideAdmin()" class="ml-4 text-gray-500">關閉</button>
        </div>
    </div>

    <script>
        // --- 預設範例 ---
        const defaultConfig = {
            templates: {
                "基礎講座": { pre: ["電池檢查", "線材測試"], mid: ["試音"], post: ["撤收"] },
                "樂團演出": { pre: ["鼓組收音", "DI測試"], mid: ["Line Check"], post: ["捲線"] }
            }
        };

        let config = JSON.parse(localStorage.getItem('audio_config')) || defaultConfig;
        let projects = JSON.parse(localStorage.getItem('audio_projects')) || [];
        let activeId = projects.length > 0 ? projects[0].id : null;
        let selectedTemplateItems = { pre: [], mid: [], post: [] };

        function init() {
            renderTabs();
            renderContent();
            renderTemplatePicker();
            document.getElementById('jsonInputArea').value = JSON.stringify(config, null, 4);
        }

        // --- 案子建立邏輯 ---
        function openNewProjectModal() {
            document.getElementById('projectModal').classList.remove('hidden');
            document.getElementById('newProjName').value = `案子-${new Date().toLocaleDateString()}`;
        }

        function closeProjectModal() {
            document.getElementById('projectModal').classList.add('hidden');
            document.getElementById('templatePreview').classList.add('hidden');
        }

        function renderTemplatePicker() {
            const picker = document.getElementById('templatePicker');
            picker.innerHTML = Object.keys(config.templates).map(name => `
                <button onclick="selectTemplate('${name}')" class="p-3 bg-gray-700/50 border border-gray-600 rounded-lg hover:border-blue-500 text-sm">
                    ${name}
                </button>
            `).join('');
        }

        function selectTemplate(name) {
            const template = config.templates[name];
            selectedTemplateItems = {
                pre: template.pre.map(t => ({ text: t, active: true })),
                mid: template.mid.map(t => ({ text: t, active: true })),
                post: template.post.map(t => ({ text: t, active: true }))
            };
            
            const preview = document.getElementById('templatePreview');
            const list = document.getElementById('previewList');
            preview.classList.remove('hidden');
            
            let html = "";
            ['pre', 'mid', 'post'].forEach(phase => {
                html += `<div class="mb-2 text-blue-300 font-bold">${phase === 'pre' ? '行前' : phase === 'mid' ? '行中' : '行後'}</div>`;
                selectedTemplateItems[phase].forEach((item, idx) => {
                    html += `
                        <div class="flex items-center gap-2 bg-gray-800 p-2 rounded mb-1">
                            <input type="checkbox" checked onchange="selectedTemplateItems.${phase}[${idx}].active = this.checked">
                            <input type="text" value="${item.text}" onchange="selectedTemplateItems.${phase}[${idx}].text = this.value" class="bg-transparent flex-1 border-none focus:ring-0">
                        </div>`;
                });
            });
            list.innerHTML = html;
        }

        function confirmCreateProject() {
            const name = document.getElementById('newProjName').value;
            const newProj = {
                id: Date.now(),
                name: name,
                checklist: {
                    pre: selectedTemplateItems.pre.filter(i => i.active).map(i => ({ text: i.text, done: false })),
                    mid: selectedTemplateItems.mid.filter(i => i.active).map(i => ({ text: i.text, done: false })),
                    post: selectedTemplateItems.post.filter(i => i.active).map(i => ({ text: i.text, done: false }))
                }
            };
            projects.push(newProj);
            activeId = newProj.id;
            saveData(); closeProjectModal(); renderTabs(); renderContent();
        }

        // --- 自由新增項目功能 ---
        function addNewItem(phase) {
            const input = document.getElementById(`new-input-${phase}`);
            const text = input.value.trim();
            if (text) {
                const project = projects.find(p => p.id === activeId);
                project.checklist[phase].push({ text: text, done: false });
                input.value = "";
                saveData();
                renderContent();
            }
        }

        // --- 渲染 UI ---
        function renderContent() {
            const container = document.getElementById('mainContent');
            const p = projects.find(proj => proj.id === activeId);
            if (!p) { container.innerHTML = `<div class="text-center py-40 text-gray-500 italic">點擊上方「+ 新案子」開始</div>`; return; }

            container.innerHTML = `
                <div class="flex justify-between items-center mb-8">
                    <h2 class="text-3xl font-bold">${p.name}</h2>
                </div>
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    ${renderPhaseColumn('行前準備', 'pre', p.checklist.pre)}
                    ${renderPhaseColumn('現場執行', 'mid', p.checklist.mid)}
                    ${renderPhaseColumn('撤場歸庫', 'post', p.checklist.post)}
                </div>`;
        }

        function renderPhaseColumn(title, phaseKey, items) {
            return `
                <div class="bg-gray-800/50 border border-gray-700 rounded-2xl p-6 shadow-xl flex flex-col h-full">
                    <h3 class="text-blue-400 font-bold mb-4 flex justify-between items-center">
                        <span>${title}</span>
                        <span class="text-xs bg-blue-900/40 px-2 py-1 rounded text-blue-300">${items.filter(i => i.done).length} / ${items.length}</span>
                    </h3>
                    <ul class="space-y-3 mb-6 flex-1">
                        ${items.map((it, idx) => `
                            <li class="flex items-start gap-3 group cursor-pointer" onclick="toggleItem('${phaseKey}', ${idx})">
                                <input type="checkbox" class="w-5 h-5 mt-1 accent-blue-500 pointer-events-none" ${it.done ? 'checked' : ''}>
                                <span class="${it.done ? 'text-gray-500 line-through' : 'text-gray-200'}">${it.text}</span>
                            </li>`).join('')}
                    </ul>
                    <div class="flex gap-2 mt-auto pt-4 border-t border-gray-700">
                        <input type="text" id="new-input-${phaseKey}" placeholder="新增${title}項目..." 
                            class="bg-gray-900 border border-gray-700 rounded-lg px-3 py-2 text-sm flex-1 focus:border-blue-500 outline-none"
                            onkeypress="if(event.key==='Enter') addNewItem('${phaseKey}')">
                        <button onclick="addNewItem('${phaseKey}')" class="bg-gray-700 hover:bg-gray-600 px-3 py-2 rounded-lg text-sm">＋</button>
                    </div>
                </div>`;
        }

        // --- 基礎邏輯 ---
        function toggleItem(phase, idx) {
            projects.find(p => p.id === activeId).checklist[phase][idx].done = !projects.find(p => p.id === activeId).checklist[phase][idx].done;
            saveData(); renderContent();
        }
        function saveData() { localStorage.setItem('audio_projects', JSON.stringify(projects)); localStorage.setItem('audio_config', JSON.stringify(config)); }
        function showAdmin() { document.getElementById('adminPanel').classList.remove('hidden'); }
        function hideAdmin() { document.getElementById('adminPanel').classList.add('hidden'); }
        function renderTabs() {
            const tabBar = document.getElementById('tabBar');
            tabBar.innerHTML = `<span class="text-xl font-bold text-blue-400 mr-4 italic">AudioCheck</span>` + projects.map(p => `
                <div class="flex items-center shrink-0">
                    <button onclick="activeId=${p.id}; renderTabs(); renderContent();" class="px-4 py-2 transition-all rounded-t-lg ${activeId === p.id ? 'tab-active' : 'text-gray-500'}">${p.name}</button>
                    <button onclick="deleteProject(${p.id})" class="p-2 text-gray-600 hover:text-red-400">✕</button>
                </div>`).join('');
        }
        function deleteProject(id) { if(confirm('確定刪除？')) { projects = projects.filter(p => p.id !== id); activeId = projects.length > 0 ? projects[0].id : null; saveData(); renderTabs(); renderContent(); } }
        function saveJsonConfig() { config = JSON.parse(document.getElementById('jsonInputArea').value); saveData(); renderTemplatePicker(); hideAdmin(); }

        init();
    </script>
</body>
</html>
