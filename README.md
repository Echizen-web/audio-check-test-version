# audio-check-test-version
audio-check
<測試模式>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AudioCheck Pro - 雙重更新模式</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .tab-active { border-bottom: 4px solid #3b82f6; color: #3b82f6; background: rgba(59, 130, 246, 0.1); }
        .admin-tab-active { background: #334155; color: #fbbf24; border-left: 4px solid #fbbf24; }
        .checkbox-item:checked + span { text-decoration: line-through; color: #9ca3af; }
    </style>
</head>
<body class="bg-gray-900 text-gray-100 min-h-screen">

    <nav class="bg-gray-800 border-b border-gray-700 p-4 sticky top-0 z-50 flex justify-between items-center">
        <div class="flex items-center gap-6 overflow-x-auto" id="tabBar">
            <span class="text-xl font-bold text-blue-400 mr-4 italic">AudioCheck</span>
        </div>
        <div class="flex gap-2 shrink-0">
            <button onclick="showAdmin()" class="bg-gray-700 hover:bg-gray-600 px-3 py-1 rounded text-sm border border-gray-600">⚙️ 管理員設定</button>
            <button onclick="openModal()" class="bg-blue-600 hover:bg-blue-500 px-4 py-1 rounded text-sm font-bold shadow-lg">+ 新案子</button>
        </div>
    </nav>

    <main id="mainContent" class="p-6 max-w-7xl mx-auto">
        </main>

    <div id="modal" class="hidden fixed inset-0 bg-black/80 backdrop-blur-sm flex items-center justify-center p-4 z-[100]">
        <div class="bg-gray-800 border border-gray-700 rounded-2xl p-6 w-full max-w-md shadow-2xl">
            <h2 class="text-xl font-bold mb-4">選擇案子範例</h2>
            <div id="templateList" class="space-y-3"></div>
            <button onclick="closeModal()" class="w-full text-center p-2 text-gray-400 mt-4">取消</button>
        </div>
    </div>

    <div id="adminPanel" class="hidden fixed inset-0 bg-gray-950 z-[110] flex flex-col md:flex-row">
        <div class="w-full md:w-64 bg-gray-900 border-r border-gray-800 p-4">
            <h2 class="text-xl font-bold text-blue-400 mb-8 px-4">系統管理員</h2>
            <nav class="space-y-2">
                <button onclick="switchAdminTab('content')" id="btn-admin-content" class="w-full text-left p-4 rounded-lg admin-tab-active">📄 檢查表內容更新</button>
                <button onclick="switchAdminTab('system')" id="btn-admin-system" class="w-full text-left p-4 rounded-lg hover:bg-gray-800">💻 網頁功能更新</button>
            </nav>
            <button onclick="hideAdmin()" class="mt-auto w-full p-4 text-gray-500 hover:text-white text-center">✕ 關閉設定</button>
        </div>
        
        <div class="flex-1 p-8 overflow-y-auto">
            <div id="admin-content-view">
                <h3 class="text-2xl font-bold mb-2">更新檢查表內容 (JSON)</h3>
                <p class="text-gray-400 mb-6">請在下方貼上新的檢查項目資料。這不會影響網頁功能。</p>
                <textarea id="jsonInputArea" class="w-full h-96 bg-black text-emerald-500 p-4 font-mono text-sm border border-gray-700 rounded-lg mb-4"></textarea>
                <button onclick="saveJsonConfig()" class="bg-emerald-600 hover:bg-emerald-500 px-8 py-3 rounded-xl font-bold">確認更新內容項目</button>
            </div>

            <div id="admin-system-view" class="hidden">
                <h3 class="text-2xl font-bold mb-2 text-amber-400">更新網頁核心程式碼 (HTML/JS)</h3>
                <p class="text-gray-400 mb-6">注意：貼上錯誤的代碼可能導致系統無法運作。更新後網頁會自動下載新版備份並重整。</p>
                <textarea id="systemCodeArea" class="w-full h-96 bg-black text-amber-500 p-4 font-mono text-sm border border-gray-700 rounded-lg mb-4" placeholder="在此貼上全段 HTML 程式碼..."></textarea>
                <button onclick="saveSystemCode()" class="bg-amber-600 hover:bg-amber-500 px-8 py-3 rounded-xl font-bold text-black">確認更新核心功能</button>
            </div>
        </div>
    </div>

    <script>
        // --- 初始化配置 ---
        const defaultConfig = {
            templates: {
                "小型講座": { pre: ["電池檢查", "線材測試"], mid: ["試音"], post: ["撤收"] },
                "樂團演出": { pre: ["鼓組收音", "DI測試"], mid: ["Line Check"], post: ["捲線"] }
            }
        };

        let config = JSON.parse(localStorage.getItem('audio_config')) || defaultConfig;
        let projects = JSON.parse(localStorage.getItem('audio_projects')) || [];
        let activeId = projects.length > 0 ? projects[0].id : null;

        function init() {
            renderTabs();
            renderContent();
            renderTemplateButtons();
            document.getElementById('jsonInputArea').value = JSON.stringify(config, null, 4);
        }

        // --- 設定分頁切換 ---
        function switchAdminTab(type) {
            document.getElementById('admin-content-view').classList.toggle('hidden', type !== 'content');
            document.getElementById('admin-system-view').classList.toggle('hidden', type !== 'system');
            document.getElementById('btn-admin-content').classList.toggle('admin-tab-active', type === 'content');
            document.getElementById('btn-admin-system').classList.toggle('admin-tab-active', type === 'system');
        }

        // --- 核心功能函數 ---
        function renderTemplateButtons() {
            const list = document.getElementById('templateList');
            list.innerHTML = Object.keys(config.templates).map(name => `
                <button onclick="addProject('${name}')" class="w-full text-left p-4 bg-gray-700/50 border border-gray-600 rounded-xl hover:bg-blue-600/30 transition-all">📁 ${name}</button>
            `).join('');
        }

        function addProject(templateName) {
            const id = Date.now();
            projects.push({
                id: id,
                name: `${templateName}-${new Date().toLocaleDateString()}`,
                template: templateName,
                checklist: {
                    pre: config.templates[templateName].pre.map(text => ({ text, done: false })),
                    mid: config.templates[templateName].mid.map(text => ({ text, done: false })),
                    post: config.templates[templateName].post.map(text => ({ text, done: false }))
                }
            });
            activeId = id;
            saveData(); closeModal(); renderTabs(); renderContent();
        }

        // --- 更新保存邏輯 ---
        function saveJsonConfig() {
            try {
                config = JSON.parse(document.getElementById('jsonInputArea').value);
                saveData();
                renderTemplateButtons();
                alert("內容更新成功！");
            } catch (e) { alert("JSON 格式錯誤！"); }
        }

        function saveSystemCode() {
            const newCode = document.getElementById('systemCodeArea').value;
            if (!newCode.includes('<html') || !newCode.includes('</html')) {
                alert("這看起來不像是完整的 HTML 程式碼，請確認後再貼上。");
                return;
            }
            if (confirm("更新核心程式碼後，系統將嘗試以新版本重新開啟。建議您先儲存一份目前的 index.html 作為備份。確定要更新嗎？")) {
                // 實務上 HTML 檔案無法從瀏覽器直接覆寫本地檔案，
                // 這裡我們提示使用者手動存檔，或下載新檔案。
                const blob = new Blob([newCode], { type: 'text/html' });
                const a = document.createElement('a');
                a.href = URL.createObjectURL(blob);
                a.download = 'index_updated.html';
                a.click();
                alert("新版本已下載為 index_updated.html，請用該檔案取代目前的檔案。");
            }
        }

        // 輔助函數
        function saveData() {
            localStorage.setItem('audio_projects', JSON.stringify(projects));
            localStorage.setItem('audio_config', JSON.stringify(config));
        }
        function openModal() { document.getElementById('modal').classList.remove('hidden'); }
        function closeModal() { document.getElementById('modal').classList.add('hidden'); }
        function showAdmin() { document.getElementById('adminPanel').classList.remove('hidden'); }
        function hideAdmin() { document.getElementById('adminPanel').classList.add('hidden'); }
        function toggleCheck(phase, idx) { projects.find(p => p.id === activeId).checklist[phase][idx].done = !projects.find(p => p.id === activeId).checklist[phase][idx].done; saveData(); renderContent(); }
        function deleteProject(id) { if(confirm('刪除？')) { projects = projects.filter(p => p.id !== id); activeId = projects.length > 0 ? projects[0].id : null; saveData(); renderTabs(); renderContent(); } }
        
        function renderTabs() {
            const tabBar = document.getElementById('tabBar');
            tabBar.innerHTML = `<span class="text-xl font-bold text-blue-400 mr-4 italic">AudioCheck</span>` + projects.map(p => `
                <div class="flex items-center shrink-0">
                    <button onclick="activeId=${p.id}; renderTabs(); renderContent();" class="px-4 py-2 transition-all rounded-t-lg ${activeId === p.id ? 'tab-active' : 'text-gray-500'}">${p.name}</button>
                    <button onclick="deleteProject(${p.id})" class="text-xs text-gray-600 hover:text-red-400 ml-1">✕</button>
                </div>`).join('');
        }

        function renderContent() {
            const container = document.getElementById('mainContent');
            const p = projects.find(proj => proj.id === activeId);
            if (!p) { container.innerHTML = `<div class="text-center py-40 text-gray-500">尚無案子</div>`; return; }
            container.innerHTML = `<div class="flex justify-between items-center mb-8"><div><h2 class="text-3xl font-bold">${p.name}</h2><p class="text-gray-400">範例：${p.template}</p></div></div>
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    ${renderSec('行前', 'pre', p.checklist.pre)} ${renderSec('行中', 'mid', p.checklist.mid)} ${renderSec('行後', 'post', p.checklist.post)}
                </div>`;
        }

        function renderSec(t, k, items) {
            return `<div class="bg-gray-800/50 border border-gray-700 rounded-2xl p-6">
                <h3 class="text-blue-400 font-bold mb-4">${t}</h3>
                <ul class="space-y-4">${items.map((it, i) => `
                    <li class="flex items-start gap-3 cursor-pointer group" onclick="toggleCheck('${k}', ${i})">
                        <input type="checkbox" class="checkbox-item w-5 h-5 accent-blue-500 pointer-events-none" ${it.done ? 'checked' : ''}>
                        <span class="${it.done ? 'text-gray-500 line-through' : 'text-gray-300'} group-hover:text-white">${it.text}</span>
                    </li>`).join('')}</ul></div>`;
        }

        init();
    </script>
</body>
</html>
