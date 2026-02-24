<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AudioCheck - 穩定修復版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* 基礎配色與結構 */
        :root { --bg: #131314; --sidebar: #1e1f20; --border: #444746; }
        body { background-color: var(--bg); color: #e3e3e3; margin: 0; font-family: sans-serif; }
        
        /* 佈局修正 */
        .app-wrapper { display: flex; height: 100vh; width: 100vw; overflow: hidden; }
        
        /* 側邊欄 */
        #sidebar { width: 260px; background: var(--sidebar); display: flex; flex-direction: column; transition: transform 0.3s ease; z-index: 1000; border-right: 1px solid var(--border); }
        
        /* 主內容區：確保滾動 */
        #mainContainer { flex: 1; overflow-y: auto; height: 100%; position: relative; display: flex; flex-direction: column; }
        #mainContent { padding: 40px 20px; max-width: 900px; margin: 0 auto; width: 100%; }

        /* 手機端側欄自動隱藏 */
        @media (max-width: 1024px) {
            #sidebar { position: absolute; left: 0; top: 0; bottom: 0; transform: translateX(-100%); }
            #sidebar.active { transform: translateX(0); }
            .mobile-header { display: flex; }
        }
        @media (min-width: 1025px) { .mobile-header { display: none; } }

        /* 滾動條樣式 */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-thumb { background: #444746; border-radius: 10px; }
        
        /* 選項勾選特效 */
        .done-text { text-decoration: line-through; color: #8e918f; font-weight: 300; }
        input[type="checkbox"] { width: 18px; height: 18px; accent-color: #8ab4f8; cursor: pointer; }
    </style>
</head>
<body>

    <div class="app-wrapper">
        <aside id="sidebar">
            <div class="p-5 flex justify-between items-center border-b border-[#333537]">
                <span class="text-xl font-bold text-blue-400">AudioCheck</span>
                <button onclick="toggleSidebar()" class="lg:hidden text-2xl">✕</button>
            </div>

            <div class="p-4">
                <button onclick="openNewProjectModal()" class="w-full bg-[#1a1b1c] border border-var(--border) py-3 rounded-full text-sm font-medium hover:bg-[#333537] transition-all">
                    + 新案子
                </button>
            </div>

            <nav id="projectList" class="flex-1 overflow-y-auto px-2 space-y-1"></nav>

            <div class="p-4 border-t border-[#333537]">
                <button onclick="showAdmin()" class="w-full text-left px-4 py-3 rounded-lg hover:bg-[#333537] text-sm flex items-center gap-3">
                    ⚙️ 設定
                </button>
            </div>
        </aside>

        <main id="mainContainer">
            <div class="mobile-header bg-[#1e1f20] p-4 border-b border-[#444746] items-center gap-4">
                <button onclick="toggleSidebar()" class="text-2xl">☰</button>
                <span class="font-bold">AudioCheck</span>
            </div>

            <div id="mainContent">
                </div>
        </main>
    </div>

    <div id="projectModal" class="hidden fixed inset-0 bg-black/80 flex items-center justify-center z-[2000] p-4">
        <div class="bg-[#1e1f20] border border-[#444746] rounded-2xl w-full max-w-xl max-h-[90vh] flex flex-col p-6 shadow-2xl">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-xl font-bold">建立新案子</h2>
                <button onclick="closeProjectModal()" class="text-2xl">✕</button>
            </div>
            
            <div class="overflow-y-auto flex-1 space-y-6 pr-2">
                <input type="text" id="newProjName" placeholder="案子名稱..." class="w-full bg-transparent border-b border-[#444746] py-3 text-2xl outline-none focus:border-blue-400 transition-all">
                
                <div>
                    <p class="text-xs text-gray-500 mb-2 uppercase font-bold tracking-widest">選擇基礎範例</p>
                    <div id="templatePicker" class="flex flex-wrap gap-2"></div>
                </div>

                <div id="templatePreview" class="hidden pt-6 border-t border-[#333537] space-y-8">
                    </div>
            </div>

            <div class="pt-6 flex justify-end gap-4">
                <button onclick="closeProjectModal()" class="px-6 py-2 text-gray-400">取消</button>
                <button onclick="confirmCreateProject()" class="px-8 py-2 bg-[#8ab4f8] text-black rounded-full font-bold">建立案子</button>
            </div>
        </div>
    </div>

    <div id="adminPanel" class="hidden fixed inset-0 bg-[#131314] flex flex-col z-[3000] p-6 md:p-12">
        <div class="max-w-4xl mx-auto w-full h-full flex flex-col">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold">系統範例設定 (JSON)</h2>
                <button onclick="hideAdmin()" class="text-3xl">✕</button>
            </div>
            <textarea id="jsonInputArea" class="flex-1 bg-[#1e1f20] text-emerald-400 p-6 rounded-xl border border-[#444746] font-mono text-sm outline-none focus:border-blue-400 mb-6"></textarea>
            <div class="flex gap-4">
                <button onclick="saveJsonConfig()" class="bg-[#8ab4f8] text-black px-10 py-3 rounded-full font-bold">儲存變更</button>
                <button onclick="resetToDefault()" class="border border-[#444746] px-6 py-3 rounded-full text-gray-400">恢復預設</button>
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
        let tempItems = { pre: [], mid: [], post: [] };

        function init() {
            renderSidebar();
            renderContent();
            renderTemplatePicker();
            document.getElementById('jsonInputArea').value = JSON.stringify(config, null, 4);
        }

        function renderSidebar() {
            const list = document.getElementById('projectList');
            list.innerHTML = projects.map(p => `
                <div class="flex items-center group px-3 py-2 rounded-lg cursor-pointer ${activeId === p.id ? 'bg-[#004a77] text-white' : 'hover:bg-[#333537] text-gray-400'} transition-all">
                    <span onclick="switchProject(${p.id})" class="flex-1 truncate text-sm">${p.name}</span>
                    <button onclick="deleteProject(${p.id})" class="opacity-0 group-hover:opacity-100 px-2 hover:text-red-400">✕</button>
                </div>`).join('');
        }

        function renderTemplatePicker() {
            const picker = document.getElementById('templatePicker');
            picker.innerHTML = Object.keys(config.templates).map(name => `
                <button onclick="selectTemplate('${name}')" class="px-4 py-2 border border-[#444746] rounded-full text-xs hover:bg-[#333537] transition-all">
                    ${name}
                </button>`).join('');
        }

        function selectTemplate(name) {
            const t = config.templates[name];
            tempItems = {
                pre: t.pre.map(text => ({ text, active: true })),
                mid: t.mid.map(text => ({ text, active: true })),
                post: t.post.map(text => ({ text, active: true }))
            };
            document.getElementById('templatePreview').classList.remove('hidden');
            renderPreviewList();
        }

        function renderPreviewList() {
            const preview = document.getElementById('templatePreview');
            let html = "";
            ['pre', 'mid', 'post'].forEach(k => {
                const label = k==='pre'?'行前準備':k==='mid'?'現場執行':'撤場歸庫';
                html += `<div class="bg-[#1a1b1c] p-4 rounded-xl border border-[#333537] mb-4">
                    <div class="flex justify-between items-center mb-4"><span class="text-xs text-blue-400 font-bold uppercase">${label}</span><button onclick="addTempItem('${k}')" class="text-[10px] bg-[#333537] px-2 py-1 rounded">+ 增加項目</button></div>`;
                tempItems[k].forEach((item, i) => {
                    html += `<div class="flex items-center gap-3 mb-2">
                        <input type="checkbox" ${item.active?'checked':''} onchange="tempItems.${k}[${i}].active=this.checked">
                        <input type="text" value="${item.text}" oninput="tempItems.${k}[${i}].text=this.value" class="bg-transparent border-b border-transparent focus:border-blue-400 outline-none text-sm flex-1">
                        <button onclick="tempItems.${k}.splice(${i},1); renderPreviewList();" class="text-xs text-gray-500 hover:text-red-500">✕</button>
                    </div>`;
                });
                html += `</div>`;
            });
            preview.innerHTML = html;
        }

        function addTempItem(k) { tempItems[k].push({ text: "新檢查項", active: true }); renderPreviewList(); }

        function renderContent() {
            const container = document.getElementById('mainContent');
            const p = projects.find(proj => proj.id === activeId);
            if (!p) { container.innerHTML = `<div class="text-center py-20 text-gray-500"><p class="text-5xl mb-6">✨</p><p>選擇或建立一個音訊工程案子</p></div>`; return; }

            container.innerHTML = `
                <div class="mb-10">
                    <h1 class="text-3xl md:text-5xl font-bold mb-2">${p.name}</h1>
                    <p class="text-gray-500 text-xs font-mono">ID: ${p.id}</p>
                </div>
                <div class="space-y-12">
                    ${renderPhase('行前準備', 'pre', p.checklist.pre)}
                    ${renderPhase('現場執行', 'mid', p.checklist.mid)}
                    ${renderPhase('撤場歸庫', 'post', p.checklist.post)}
                </div>`;
        }

        function renderPhase(title, key, items) {
            return `
                <div>
                    <h3 class="text-xs text-blue-400 font-bold tracking-[0.2em] uppercase border-b border-[#333537] pb-3 mb-6 flex justify-between">
                        ${title} <span>${items.filter(i=>i.done).length}/${items.length}</span>
                    </h3>
                    <div class="space-y-6">
                        ${items.map((it, idx) => `
                            <div class="flex items-start gap-4 cursor-pointer" onclick="toggleItem('${key}', ${idx})">
                                <input type="checkbox" class="mt-1" ${it.done?'checked':''} onclick="event.stopPropagation()">
                                <span class="text-base leading-relaxed ${it.done?'done-text':''}">${it.text}</span>
                            </div>`).join('')}
                    </div>
                    <input type="text" placeholder="+ 新增..." onkeypress="if(event.key==='Enter') addItem('${key}', this)" class="mt-6 w-full bg-[#1e1f20] border border-[#444746] rounded-xl px-4 py-3 text-sm focus:border-blue-400 outline-none">
                </div>`;
        }

        function switchProject(id) { activeId = id; renderSidebar(); renderContent(); if(window.innerWidth < 1024) toggleSidebar(); }
        function toggleSidebar() { document.getElementById('sidebar').classList.toggle('active'); }
        function openNewProjectModal() { document.getElementById('projectModal').classList.remove('hidden'); document.getElementById('templatePreview').classList.add('hidden'); }
        function closeProjectModal() { document.getElementById('projectModal').classList.add('hidden'); }
        function showAdmin() { document.getElementById('adminPanel').classList.remove('hidden'); }
        function hideAdmin() { document.getElementById('adminPanel').classList.add('hidden'); }
        function toggleItem(k, i) { projects.find(p=>p.id===activeId).checklist[k][i].done = !projects.find(p=>p.id===activeId).checklist[k][i].done; saveData(); renderContent(); }
        function addItem(k, input) { if(input.value.trim()){ projects.find(p=>p.id===activeId).checklist[k].push({text:input.value, done:false}); input.value=''; saveData(); renderContent(); } }
        function saveData() { localStorage.setItem('audio_projects', JSON.stringify(projects)); localStorage.setItem('audio_config', JSON.stringify(config)); }
        function deleteProject(id) { if(confirm('確定刪除？')){ projects=projects.filter(p=>p.id!==id); activeId=projects.length?projects[0].id:null; saveData(); renderSidebar(); renderContent(); } }
        function confirmCreateProject() {
            const name = document.getElementById('newProjName').value || "未命名";
            const newP = { id: Date.now(), name, checklist: { pre: tempItems.pre.filter(i=>i.active).map(i=>({text:i.text, done:false})), mid: tempItems.mid.filter(i=>i.active).map(i=>({text:i.text, done:false})), post: tempItems.post.filter(i=>i.active).map(i=>({text:i.text, done:false})) } };
            projects.unshift(newP); activeId = newP.id; saveData(); closeProjectModal(); renderSidebar(); renderContent();
        }
        function saveJsonConfig() { try { config=JSON.parse(document.getElementById('jsonInputArea').value); saveData(); renderTemplatePicker(); hideAdmin(); } catch(e){alert('JSON 格式錯誤');} }
        function resetToDefault() { if(confirm('重設？')){ config=initialDefault; document.getElementById('jsonInputArea').value=JSON.stringify(config,null,4); saveData(); renderTemplatePicker(); } }

        init();
    </script>
</body>
</html>
