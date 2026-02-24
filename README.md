<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>AudioCheck - Gemini Style Fixed</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { background-color: #131314; color: #e3e3e3; margin: 0; padding: 0; height: 100vh; display: flex; overflow: hidden; }
        .sidebar { background-color: #1e1f20; transition: all 0.3s ease; }
        .sidebar-item:hover { background-color: #333537; }
        .sidebar-item-active { background-color: #004a77; color: #c2e7ff; }
        
        /* 核心滾動修正 */
        #mainContentContainer { flex: 1; overflow-y: auto; -webkit-overflow-scrolling: touch; }
        .custom-scrollbar::-webkit-scrollbar { width: 6px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #444746; border-radius: 10px; }
        
        /* 彈窗滾動修正 */
        .modal-content { max-height: 80vh; overflow-y: auto; }
        
        input[type="checkbox"] { accent-color: #8ab4f8; width: 1.2rem; height: 1.2rem; cursor: pointer; }
        .line-through-text { text-decoration: line-through; color: #8e918f; }
        
        /* 手機端側邊欄隱藏邏輯 */
        @media (max-width: 1024px) {
            .sidebar-hidden { margin-left: -256px; }
            .sidebar-overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 40; }
            .sidebar-overlay.active { display: block; }
        }
    </style>
</head>
<body class="font-sans">

    <div id="sidebarOverlay" class="sidebar-overlay" onclick="toggleSidebar()"></div>

    <aside id="sidebar" class="sidebar w-64 flex flex-col shrink-0 z-50 h-full lg:ml-0 sidebar-hidden">
        <div class="p-4 flex justify-between items-center">
            <span class="text-xl font-bold text-blue-400 italic px-2">AudioCheck</span>
            <button onclick="toggleSidebar()" class="lg:hidden text-gray-400 p-2">✕</button>
        </div>

        <div class="px-4 mb-4">
            <button onclick="openNewProjectModal()" class="flex items-center gap-3 w-full px-4 py-3 rounded-full bg-[#1a1b1c] hover:bg-[#333537] text-gray-400 transition-all border border-[#444746]">
                <span class="text-xl">+</span>
                <span class="text-sm font-medium">新案子</span>
            </button>
        </div>
        
        <nav class="flex-1 overflow-y-auto custom-scrollbar px-3 space-y-1" id="projectList"></nav>

        <div class="p-4 border-t border-[#444746]">
            <button onclick="showAdmin()" class="flex items-center gap-3 w-full px-4 py-3 rounded-lg hover:bg-[#333537] text-gray-300 transition-all">
                <span>⚙️</span>
                <span class="text-sm">設定</span>
            </button>
        </div>
    </aside>

    <main class="flex-1 flex flex-col min-w-0 bg-[#131314]">
        <header class="lg:hidden p-4 flex items-center justify-between border-b border-[#444746] bg-[#1e1f20]">
            <div class="flex items-center gap-4">
                <button onclick="toggleSidebar()" class="text-2xl">☰</button>
                <span class="font-bold text-blue-400">AudioCheck</span>
            </div>
        </header>

        <div id="mainContentContainer" class="custom-scrollbar">
            <div id="mainContent" class="p-6 lg:p-12 max-w-6xl mx-auto">
                </div>
        </div>
    </main>

    <div id="projectModal" class="hidden fixed inset-0 bg-black/80 backdrop-blur-sm flex items-center justify-center p-4 z-[100]">
        <div class="bg-[#1e1f20] border border-[#444746] rounded-2xl p-6 md:p-8 w-full max-w-2xl shadow-2xl flex flex-col">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-medium">建立新案子</h2>
                <button onclick="closeProjectModal()" class="text-gray-400 hover:text-white">✕</button>
            </div>
            
            <div class="modal-content custom-scrollbar pr-2 space-y-8">
                <input type="text" id="newProjName" placeholder="案子名稱..." class="w-full bg-transparent border-b border-[#444746] py-2 text-2xl outline-none focus:border-[#8ab4f8] transition-all">

                <div>
                    <p class="text-xs text-gray-500 font-bold mb-3 uppercase tracking-widest">選擇範例</p>
                    <div id="templatePicker" class="flex flex-wrap gap-2"></div>
                </div>

                <div id="templatePreview" class="hidden space-y-8 pt-4 border-t border-[#333537]">
                    <div id="previewList" class="space-y-8"></div>
                </div>
            </div>

            <div class="pt-8 flex justify-end gap-4">
                <button onclick="closeProjectModal()" class="px-6 py-2 text-sm text-gray-400">取消</button>
                <button onclick="confirmCreateProject()" class="px-8 py-2 text-sm bg-[#8ab4f8] text-[#131314] rounded-full font-bold hover:bg-[#a1c2fa]">建立並儲存</button>
            </div>
        </div>
    </div>

    <div id="adminPanel" class="hidden fixed inset-0 bg-[#131314]/98 z-[110] p-4 md:p-10 flex flex-col items-center overflow-y-auto">
        <div class="w-full max-w-4xl h-full flex flex-col">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold">系統範例設定</h2>
                <button onclick="hideAdmin()" class="text-3xl">✕</button>
            </div>
            <textarea id="jsonInputArea" class="flex-1 min-h-[300px] bg-[#1e1f20] text-emerald-400 p-6 font-mono text-sm border border-[#444746] rounded-xl outline-none focus:border-[#8ab4f8] custom-scrollbar shadow-inner"></textarea>
            <div class="mt-6 flex gap-4">
                <button onclick="saveJsonConfig()" class="bg-[#8ab4f8] text-black px-10 py-3 rounded-full font-bold shadow-lg">儲存變更</button>
                <button onclick="resetToDefault()" class="border border-[#444746] px-8 py-3 rounded-full text-gray-400">重設</button>
            </div>
        </div>
    </div>

    <script>
        const initialDefault = {
            templates: {
                "小型講座": { pre: ["麥克風電池測試", "PPT 聲音測試"], mid: ["主持人音量監看"], post: ["設備歸庫"] },
                "樂團演出": { pre: ["鼓組 Mic 設置", "DI Box 確認"], mid: ["Sound Check", "多軌錄音確認"], post: ["線材整理"] }
            }
        };

        let config = JSON.parse(localStorage.getItem('audio_config')) || initialDefault;
        let projects = JSON.parse(localStorage.getItem('audio_projects')) || [];
        let activeId = projects.length > 0 ? projects[0].id : null;
        let tempEditItems = { pre: [], mid: [], post: [] };

        function init() {
            renderSidebar();
            renderContent();
            renderTemplatePicker();
            document.getElementById('jsonInputArea').value = JSON.stringify(config, null, 4);
        }

        function renderSidebar() {
            const list = document.getElementById('projectList');
            list.innerHTML = projects.map(p => `
                <div class="group flex items-center justify-between px-4 py-3 rounded-lg cursor-pointer transition-all ${activeId === p.id ? 'sidebar-item-active' : 'sidebar-item'}">
                    <div onclick="switchProject(${p.id})" class="flex-1 truncate text-sm">${p.name}</div>
                    <button onclick="deleteProject(${p.id})" class="opacity-0 group-hover:opacity-100 px-2 text-gray-500 hover:text-red-400">✕</button>
                </div>`).join('');
        }

        function renderTemplatePicker() {
            const picker = document.getElementById('templatePicker');
            picker.innerHTML = Object.keys(config.templates).map(name => `
                <button onclick="selectTemplate('${name}')" class="px-4 py-2 rounded-full border border-[#444746] text-xs hover:bg-[#333537] transition-all text-gray-300">
                    ${name}
                </button>`).join('');
        }

        function selectTemplate(name) {
            const t = config.templates[name];
            tempEditItems = {
                pre: t.pre.map(text => ({ text, active: true })),
                mid: t.mid.map(text => ({ text, active: true })),
                post: t.post.map(text => ({ text, active: true }))
            };
            document.getElementById('templatePreview').classList.remove('hidden');
            renderPreviewList();
        }

        function renderPreviewList() {
            const list = document.getElementById('previewList');
            let html = "";
            ['pre', 'mid', 'post'].forEach(k => {
                const label = k==='pre'?'行前準備':k==='mid'?'現場執行':'撤場歸庫';
                html += `<div class="bg-[#1a1b1c] p-4 rounded-xl border border-[#333537]">
                    <div class="flex justify-between items-center mb-4">
                        <span class="text-xs text-[#8ab4f8] font-bold uppercase tracking-widest">${label}</span>
                        <button onclick="addTempItem('${k}')" class="text-[10px] bg-[#333537] px-2 py-1 rounded text-gray-400 hover:text-white">+ 增加選項</button>
                    </div>`;
                tempEditItems[k].forEach((item, i) => {
                    html += `
                    <div class="flex items-center gap-3 mb-3 group">
                        <input type="checkbox" ${item.active?'checked':''} onchange="tempEditItems.${k}[${i}].active=this.checked">
                        <input type="text" value="${item.text}" oninput="tempEditItems.${k}[${i}].text=this.value" class="bg-transparent border-b border-transparent focus:border-[#444746] outline-none text-sm flex-1 text-gray-200">
                        <button onclick="tempEditItems.${k}.splice(${i},1); renderPreviewList();" class="text-gray-600 hover:text-red-400 text-xs px-2">✕</button>
                    </div>`;
                });
                html += `</div>`;
            });
            list.innerHTML = html;
        }

        function addTempItem(k) { tempEditItems[k].push({ text: "新檢查項", active: true }); renderPreviewList(); }

        function renderContent() {
            const container = document.getElementById('mainContent');
            const p = projects.find(proj => proj.id === activeId);
            if (!p) { container.innerHTML = `<div class="py-40 flex flex-col items-center justify-center text-gray-500 text-center"><div class="text-6xl mb-6">✨</div><p>請點擊左側「新案子」<br>或選擇一個現有的工程</p></div>`; return; }

            container.innerHTML = `
                <div class="mb-12">
                    <h1 class="text-3xl md:text-5xl font-medium text-white mb-2">${p.name}</h1>
                    <p class="text-gray-500 font-mono text-xs">ID: ${p.id}</p>
                </div>
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-10">
                    ${renderColumn('行前準備', 'pre', p.checklist.pre)}
                    ${renderColumn('現場執行', 'mid', p.checklist.mid)}
                    ${renderColumn('撤場歸庫', 'post', p.checklist.post)}
                </div>`;
        }

        function renderColumn(title, key, items) {
            return `
                <div class="flex flex-col">
                    <div class="text-[11px] text-[#8ab4f8] font-black tracking-[0.2em] mb-6 uppercase flex justify-between items-center border-b border-[#333537] pb-2">
                        ${title} <span class="text-gray-500">${items.filter(i=>i.done).length}/${items.length}</span>
                    </div>
                    <div class="space-y-5 mb-8">
                        ${items.map((it, idx) => `
                            <div class="flex items-start gap-4 cursor-pointer group" onclick="toggleItem('${key}', ${idx})">
                                <input type="checkbox" class="mt-0.5 pointer-events-none" ${it.done?'checked':''}>
                                <span class="text-sm md:text-base leading-relaxed ${it.done?'line-through-text font-light':'text-gray-200'}">${it.text}</span>
                            </div>`).join('')}
                    </div>
                    <input type="text" placeholder="+ 快速新增..." onkeypress="if(event.key==='Enter') addItem('${key}', this)" class="bg-[#1e1f20] border border-[#444746] rounded-lg px-4 py-3 text-sm outline-none focus:border-[#8ab4f8] transition-all">
                </div>`;
        }

        // 基本邏輯
        function switchProject(id) { activeId = id; renderSidebar(); renderContent(); if(window.innerWidth < 1024) toggleSidebar(); }
        function toggleSidebar() { 
            document.getElementById('sidebar').classList.toggle('sidebar-hidden'); 
            document.getElementById('sidebarOverlay').classList.toggle('active');
        }
        function openNewProjectModal() { document.getElementById('projectModal').classList.remove('hidden'); document.getElementById('newProjName').value=''; document.getElementById('templatePreview').classList.add('hidden'); }
        function closeProjectModal() { document.getElementById('projectModal').classList.add('hidden'); }
        function showAdmin() { document.getElementById('adminPanel').classList.remove('hidden'); }
        function hideAdmin() { document.getElementById('adminPanel').classList.add('hidden'); }
        function toggleItem(k, i) { projects.find(p=>p.id===activeId).checklist[k][i].done = !projects.find(p=>p.id===activeId).checklist[k][i].done; saveData(); renderContent(); }
        function addItem(k, input) { if(input.value.trim()){ projects.find(p=>p.id===activeId).checklist[k].push({text:input.value, done:false}); input.value=''; saveData(); renderContent(); } }
        function saveData() { localStorage.setItem('audio_projects', JSON.stringify(projects)); localStorage.setItem('audio_config', JSON.stringify(config)); }
        function deleteProject(id) { if(confirm('刪除案子？')){ projects=projects.filter(p=>p.id!==id); activeId=projects.length?projects[0].id:null; saveData(); renderSidebar(); renderContent(); } }
        function confirmCreateProject() {
            const name = document.getElementById('newProjName').value || "未命名案子";
            const newP = { id: Date.now(), name, checklist: { pre: tempEditItems.pre.filter(i=>i.active).map(i=>({text:i.text, done:false})), mid: tempEditItems.mid.filter(i=>i.active).map(i=>({text:i.text, done:false})), post: tempEditItems.post.filter(i=>i.active).map(i=>({text:i.text, done:false})) } };
            projects.unshift(newP); activeId = newP.id; saveData(); closeProjectModal(); renderSidebar(); renderContent();
        }
        function saveJsonConfig() { try { config=JSON.parse(document.getElementById('jsonInputArea').value); saveData(); renderTemplatePicker(); hideAdmin(); alert('設定已儲存'); }catch(e){alert('JSON 格式有誤');} }
        function resetToDefault() { if(confirm('重設？')){ config=initialDefault; document.getElementById('jsonInputArea').value=JSON.stringify(config,null,4); saveData(); renderTemplatePicker(); } }

        init();
    </script>
</body>
</html>
