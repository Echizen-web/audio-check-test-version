<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AudioCheck - Gemini Style</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { background-color: #131314; color: #e3e3e3; }
        .sidebar { background-color: #1e1f20; }
        .sidebar-item:hover { background-color: #333537; }
        .sidebar-item-active { background-color: #004a77; color: #c2e7ff; }
        .main-container { background-color: #131314; }
        .card { background-color: #1e1f20; border: 1px solid #444746; }
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #444746; border-radius: 10px; }
        input[type="checkbox"] { accent-color: #8ab4f8; width: 1.2rem; height: 1.2rem; }
        .line-through-text { text-decoration: line-through; color: #8e918f; }
    </style>
</head>
<body class="flex h-screen overflow-hidden font-sans">

    <aside id="sidebar" class="sidebar w-64 flex flex-col shrink-0 transition-all duration-300 z-50">
        <div class="p-4 mb-4">
            <button onclick="openNewProjectModal()" class="flex items-center gap-3 px-4 py-3 rounded-full bg-[#1a1b1c] hover:bg-[#333537] text-gray-400 transition-all border border-[#444746]">
                <span class="text-xl">+</span>
                <span class="text-sm font-medium">新案子</span>
            </button>
        </div>
        
        <div class="flex-1 overflow-y-auto custom-scrollbar px-3 space-y-1" id="projectList"></div>

        <div class="p-4 border-t border-[#444746]">
            <button onclick="showAdmin()" class="flex items-center gap-3 w-full px-4 py-3 rounded-lg hover:bg-[#333537] text-gray-300 transition-all">
                <span>⚙️</span>
                <span class="text-sm">設定</span>
            </button>
        </div>
    </aside>

    <main class="flex-1 flex flex-col min-w-0 main-container">
        <header class="p-4 flex items-center gap-4 lg:hidden border-b border-[#444746]">
            <button onclick="toggleSidebar()" class="text-2xl">☰</button>
            <span class="font-bold">AudioCheck</span>
        </header>

        <div id="mainContent" class="flex-1 overflow-y-auto p-6 lg:p-12 custom-scrollbar">
            </div>
    </main>

    <div id="projectModal" class="hidden fixed inset-0 bg-black/70 backdrop-blur-md flex items-center justify-center p-4 z-[100]">
        <div class="card rounded-2xl p-8 w-full max-w-2xl shadow-2xl max-h-[85vh] flex flex-col">
            <h2 class="text-xl font-medium mb-6">建立新案子</h2>
            
            <div class="flex-1 overflow-y-auto pr-2 custom-scrollbar space-y-6">
                <input type="text" id="newProjName" placeholder="輸入案子名稱..." class="w-full bg-transparent border-b border-[#444746] py-2 text-2xl outline-none focus:border-[#8ab4f8] transition-all">

                <div id="templatePicker" class="flex flex-wrap gap-2"></div>

                <div id="templatePreview" class="hidden space-y-4">
                    <div id="previewList" class="space-y-6"></div>
                </div>
            </div>

            <div class="pt-8 flex justify-end gap-4">
                <button onclick="closeProjectModal()" class="px-6 py-2 text-sm text-gray-400 hover:text-white">取消</button>
                <button onclick="confirmCreateProject()" class="px-8 py-2 text-sm bg-[#8ab4f8] text-[#131314] rounded-full font-bold hover:bg-[#a1c2fa]">建立</button>
            </div>
        </div>
    </div>

    <div id="adminPanel" class="hidden fixed inset-0 bg-[#131314]/95 z-[110] p-6 flex flex-col items-center">
        <div class="w-full max-w-4xl h-full flex flex-col">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold">系統設定</h2>
                <button onclick="hideAdmin()" class="text-3xl">✕</button>
            </div>
            <textarea id="jsonInputArea" class="flex-1 bg-[#1e1f20] text-emerald-400 p-6 font-mono text-sm border border-[#444746] rounded-xl outline-none focus:border-[#8ab4f8] custom-scrollbar"></textarea>
            <div class="mt-6 flex gap-4">
                <button onclick="saveJsonConfig()" class="bg-[#8ab4f8] text-black px-8 py-3 rounded-full font-bold">儲存範例變更</button>
                <button onclick="resetToDefault()" class="border border-[#444746] px-8 py-3 rounded-full">重設</button>
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
                <div class="group flex items-center justify-between px-4 py-2 rounded-lg cursor-pointer transition-all ${activeId === p.id ? 'sidebar-item-active' : 'sidebar-item'}">
                    <div onclick="switchProject(${p.id})" class="flex-1 truncate text-sm">${p.name}</div>
                    <button onclick="deleteProject(${p.id})" class="opacity-0 group-hover:opacity-100 px-2 text-xs hover:text-red-400">✕</button>
                </div>`).join('');
        }

        function renderTemplatePicker() {
            const picker = document.getElementById('templatePicker');
            picker.innerHTML = Object.keys(config.templates).map(name => `
                <button onclick="selectTemplate('${name}')" class="px-4 py-1.5 rounded-full border border-[#444746] text-xs hover:bg-[#333537] transition-all text-gray-300">
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
                const label = k==='pre'?'行前':k==='mid'?'行中':'行後';
                html += `<div><div class="flex justify-between items-center mb-2"><span class="text-xs text-[#8ab4f8] font-bold uppercase">${label}</span><button onclick="addTempItem('${k}')" class="text-xs text-gray-500 hover:text-white">+ 新增</button></div>`;
                tempEditItems[k].forEach((item, i) => {
                    html += `
                    <div class="flex items-center gap-3 mb-2 group">
                        <input type="checkbox" ${item.active?'checked':''} onchange="tempEditItems.${k}[${i}].active=this.checked">
                        <input type="text" value="${item.text}" oninput="tempEditItems.${k}[${i}].text=this.value" class="bg-transparent border-b border-transparent focus:border-[#444746] outline-none text-sm flex-1 text-gray-300">
                        <button onclick="tempEditItems.${k}.splice(${i},1); renderPreviewList();" class="opacity-0 group-hover:opacity-100 text-gray-600 hover:text-red-400 text-xs">✕</button>
                    </div>`;
                });
                html += `</div>`;
            });
            list.innerHTML = html;
        }

        function addTempItem(k) { tempEditItems[k].push({ text: "新檢查項", active: true }); renderPreviewList(); }

        function confirmCreateProject() {
            const name = document.getElementById('newProjName').value || "未命名案子";
            const newP = {
                id: Date.now(),
                name: name,
                checklist: {
                    pre: tempEditItems.pre.filter(i=>i.active).map(i=>({text:i.text, done:false})),
                    mid: tempEditItems.mid.filter(i=>i.active).map(i=>({text:i.text, done:false})),
                    post: tempEditItems.post.filter(i=>i.active).map(i=>({text:i.text, done:false}))
                }
            };
            projects.unshift(newP);
            activeId = newP.id;
            saveData(); closeProjectModal(); renderSidebar(); renderContent();
        }

        function renderContent() {
            const container = document.getElementById('mainContent');
            const p = projects.find(proj => proj.id === activeId);
            if (!p) { container.innerHTML = `<div class="h-full flex flex-col items-center justify-center text-gray-500"><div class="text-5xl mb-4">✨</div>在此開始管理你的音訊工程</div>`; return; }

            container.innerHTML = `
                <div class="max-w-6xl mx-auto">
                    <h1 class="text-3xl font-medium mb-12">${p.name}</h1>
                    <div class="grid grid-cols-1 lg:grid-cols-3 gap-12">
                        ${renderColumn('行前準備', 'pre', p.checklist.pre)}
                        ${renderColumn('現場執行', 'mid', p.checklist.mid)}
                        ${renderColumn('撤場歸庫', 'post', p.checklist.post)}
                    </div>
                </div>`;
        }

        function renderColumn(title, key, items) {
            return `
                <div class="flex flex-col">
                    <div class="text-[11px] text-gray-500 font-bold tracking-[0.2em] mb-6 uppercase flex justify-between items-center">
                        ${title} <span>${items.filter(i=>i.done).length}/${items.length}</span>
                    </div>
                    <div class="space-y-4 mb-8">
                        ${items.map((it, idx) => `
                            <div class="flex items-start gap-4 cursor-pointer group" onclick="toggleItem('${key}', ${idx})">
                                <input type="checkbox" class="mt-1 pointer-events-none" ${it.done?'checked':''}>
                                <span class="text-sm leading-relaxed ${it.done?'line-through-text':'text-gray-200'}">${it.text}</span>
                            </div>`).join('')}
                    </div>
                    <input type="text" placeholder="新增..." onkeypress="if(event.key==='Enter') addItem('${key}', this)" class="bg-transparent border-b border-[#444746] py-2 text-xs outline-none focus:border-[#8ab4f8] transition-all">
                </div>`;
        }

        function switchProject(id) { activeId = id; renderSidebar(); renderContent(); if(window.innerWidth < 1024) toggleSidebar(); }
        function toggleItem(k, i) { projects.find(p=>p.id===activeId).checklist[k][i].done = !projects.find(p=>p.id===activeId).checklist[k][i].done; saveData(); renderContent(); }
        function addItem(k, input) { if(input.value.trim()){ projects.find(p=>p.id===activeId).checklist[k].push({text:input.value, done:false}); input.value=''; saveData(); renderContent(); } }
        function saveData() { localStorage.setItem('audio_projects', JSON.stringify(projects)); localStorage.setItem('audio_config', JSON.stringify(config)); }
        function deleteProject(id) { if(confirm('刪除案子？')){ projects=projects.filter(p=>p.id!==id); activeId=projects.length?projects[0].id:null; saveData(); renderSidebar(); renderContent(); } }
        function saveJsonConfig() { try { config=JSON.parse(document.getElementById('jsonInputArea').value); saveData(); renderTemplatePicker(); hideAdmin(); alert('設定已儲存'); }catch(e){alert('JSON 格式有誤');} }
        function openNewProjectModal() { document.getElementById('projectModal').classList.remove('hidden'); document.getElementById('newProjName').value=''; document.getElementById('templatePreview').classList.add('hidden'); }
        function closeProjectModal() { document.getElementById('projectModal').classList.add('hidden'); }
        function showAdmin() { document.getElementById('adminPanel').classList.remove('hidden'); }
        function hideAdmin() { document.getElementById('adminPanel').classList.add('hidden'); }
        function toggleSidebar() { document.getElementById('sidebar').classList.toggle('-ml-64'); }
        function resetToDefault() { if(confirm('重設？')){ config=initialDefault; document.getElementById('jsonInputArea').value=JSON.stringify(config,null,4); saveData(); renderTemplatePicker(); } }

        init();
    </script>
</body>
</html>
