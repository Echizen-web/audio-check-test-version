<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AudioCheck - 專業操作版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root { --bg: #131314; --sidebar: #1e1f20; --border: #444746; --blue: #8ab4f8; }
        body { background-color: var(--bg); color: #e3e3e3; margin: 0; font-family: sans-serif; overflow: hidden; }
        .app-wrapper { display: flex; height: 100vh; width: 100vw; }
        #sidebar { width: 260px; background: var(--sidebar); display: flex; flex-direction: column; border-right: 1px solid var(--border); transition: transform 0.3s ease; z-index: 1000; }
        #mainContainer { flex: 1; overflow-y: auto; height: 100%; display: flex; flex-direction: column; }
        #mainContent { padding: 40px 20px; max-width: 900px; margin: 0 auto; width: 100%; padding-bottom: 100px; }
        
        @media (max-width: 1024px) {
            #sidebar { position: absolute; left: 0; top: 0; bottom: 0; transform: translateX(-100%); }
            #sidebar.active { transform: translateX(0); }
        }

        .done-text { text-decoration: line-through; color: #8e918f; font-weight: 300; }
        input[type="checkbox"] { width: 20px; height: 20px; accent-color: var(--blue); cursor: pointer; }
        
        /* 自定義捲軸 */
        ::-webkit-scrollbar { width: 5px; }
        ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 10px; }
    </style>
</head>
<body>

    <div class="app-wrapper">
        <aside id="sidebar">
            <div class="p-6 flex justify-between items-center">
                <span class="text-xl font-bold text-[#8ab4f8] italic">AudioCheck</span>
                <button onclick="toggleSidebar()" class="lg:hidden text-2xl">✕</button>
            </div>
            <div class="px-4 mb-6">
                <button onclick="openNewProjectModal()" class="w-full bg-[#1a1b1c] border border-[#444746] py-3 rounded-full text-sm font-medium hover:bg-[#333537] transition-all flex items-center justify-center gap-2">
                    <span class="text-lg">+</span> 新案子
                </button>
            </div>
            <nav id="projectList" class="flex-1 overflow-y-auto px-2 space-y-1"></nav>
            <div class="p-4 border-t border-[#333537]">
                <button onclick="showAdmin()" class="w-full text-left px-4 py-3 rounded-lg hover:bg-[#333537] text-sm flex items-center gap-3">⚙️ 系統設定</button>
            </div>
        </aside>

        <main id="mainContainer">
            <div class="lg:hidden bg-[#1e1f20] p-4 border-b border-[#444746] flex items-center gap-4">
                <button onclick="toggleSidebar()" class="text-2xl">☰</button>
                <span class="font-bold">AudioCheck</span>
            </div>

            <div id="mainContent">
                </div>
        </main>
    </div>

    <div id="projectModal" class="hidden fixed inset-0 bg-black/80 flex items-center justify-center z-[2000] p-4">
        <div class="bg-[#1e1f20] border border-[#444746] rounded-2xl w-full max-w-xl max-h-[90vh] flex flex-col p-8 shadow-2xl">
            <h2 class="text-xl font-bold mb-6">建立新案子</h2>
            <div class="overflow-y-auto flex-1 space-y-8 pr-2">
                <input type="text" id="newProjName" placeholder="案子名稱..." class="w-full bg-transparent border-b border-[#444746] py-3 text-2xl outline-none focus:border-[#8ab4f8]">
                <div>
                    <p class="text-xs text-gray-500 mb-3 font-bold tracking-widest uppercase">選擇基礎範例</p>
                    <div id="templatePicker" class="flex flex-wrap gap-2"></div>
                </div>
                <div id="templatePreview" class="hidden pt-6 border-t border-[#333537] space-y-6"></div>
            </div>
            <div class="pt-8 flex justify-end gap-4">
                <button onclick="closeProjectModal()" class="text-gray-400">取消</button>
                <button onclick="confirmCreateProject()" class="px-8 py-2 bg-[#8ab4f8] text-black rounded-full font-bold hover:bg-[#a1c2fa]">建立</button>
            </div>
        </div>
    </div>

    <div id="editNameModal" class="hidden fixed inset-0 bg-black/80 flex items-center justify-center z-[2500] p-4">
        <div class="bg-[#1e1f20] border border-[#444746] rounded-2xl w-full max-w-md p-8">
            <h2 class="text-lg font-bold mb-4">修改案子名稱</h2>
            <input type="text" id="editNameInput" class="w-full bg-[#131314] border border-[#444746] p-3 rounded-lg outline-none focus:border-[#8ab4f8] mb-6">
            <div class="flex justify-end gap-4">
                <button onclick="closeEditNameModal()" class="text-gray-400 text-sm">取消</button>
                <button onclick="confirmEditName()" class="bg-[#8ab4f8] text-black px-6 py-2 rounded-full text-sm font-bold">儲存</button>
            </div>
        </div>
    </div>

    <div id="adminPanel" class="hidden fixed inset-0 bg-[#131314] flex flex-col z-[3000] p-6 md:p-12">
        <div class="max-w-4xl mx-auto w-full h-full flex flex-col">
            <div class="flex justify-between items-center mb-6">
                <h2 class="text-2xl font-bold text-[#8ab4f8]">系統範例 (JSON)</h2>
                <button onclick="hideAdmin()" class="text-3xl">✕</button>
            </div>
            <textarea id="jsonInputArea" class="flex-1 bg-[#1e1f20] text-emerald-400 p-6 rounded-xl border border-[#444746] font-mono text-sm outline-none focus:border-[#8ab4f8] mb-6"></textarea>
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
                <div class="flex items-center group px-4 py-3 rounded-lg cursor-pointer ${activeId === p.id ? 'bg-[#004a77] text-[#c2e7ff]' : 'hover:bg-[#333537] text-gray-400'} transition-all mb-1">
                    <span onclick="switchProject(${p.id})" class="flex-1 truncate text-sm font-medium">${p.name}</span>
                    <button onclick="deleteProject(${p.id})" class="opacity-0 group-hover:opacity-100 px-2 hover:text-red-400 transition-all">✕</button>
                </div>`).join('');
        }

        function renderContent() {
            const container = document.getElementById('mainContent');
            const p = projects.find(proj => proj.id === activeId);
            if (!p) { container.innerHTML = `<div class="text-center py-40 text-gray-500"><p class="text-6xl mb-6">✨</p><p>請在左側建立或選擇一個案子</p></div>`; return; }

            container.innerHTML = `
                <div class="mb-12 flex items-center justify-between gap-4">
                    <div>
                        <h1 class="text-3xl md:text-5xl font-bold text-white flex items-center gap-4">
                            ${p.name}
                            <button onclick="openEditNameModal()" class="text-sm bg-[#333537] text-gray-400 px-3 py-1 rounded-full font-normal hover:text-white transition-all">🖊️ 編輯名稱</button>
                        </h1>
                        <p class="text-gray-500 text-xs mt-3 font-mono">Project ID: ${p.id}</p>
                    </div>
                </div>
                <div class="space-y-16">
                    ${renderPhase('行前準備', 'pre', p.checklist.pre)}
                    ${renderPhase('現場執行', 'mid', p.checklist.mid)}
                    ${renderPhase('撤場歸庫', 'post', p.checklist.post)}
                </div>`;
        }

        function renderPhase(title, key, items) {
            return `
                <div>
                    <h3 class="text-xs text-[#8ab4f8] font-bold tracking-[0.2em] uppercase border-b border-[#333537] pb-4 mb-8 flex justify-between">
                        ${title} <span class="bg-[#1e1f20] px-2 rounded text-gray-500">${items.filter(i=>i.done).length}/${items.length}</span>
                    </h3>
                    <div class="space-y-6">
                        ${items.map((it, idx) => `
                            <div class="flex items-start gap-4 group cursor-pointer" onclick="toggleItem('${key}', ${idx})">
                                <input type="checkbox" class="mt-1" ${it.done?'checked':''} onclick="event.stopPropagation(); toggleItem('${key}', ${idx})">
                                <span class="text-base md:text-lg leading-relaxed ${it.done?'done-text':''} flex-1">${it.text}</span>
                                <button onclick="event.stopPropagation(); removeItem('${key}', ${idx})" class="opacity-0 group-hover:opacity-100 text-gray-600 hover:text-red-500">✕</button>
                            </div>`).join('')}
                    </div>
                    <div class="mt-8">
                        <button onclick="promptAddItem('${key}')" class="bg-[#1e1f20] border border-[#444746] text-[#8ab4f8] px-6 py-2 rounded-full text-sm font-bold hover:bg-[#333537] transition-all flex items-center gap-2">
                            <span>+</span> 新增項目
                        </button>
                    </div>
                </div>`;
        }

        // --- 名稱編輯功能 ---
        function openEditNameModal() {
            const p = projects.find(proj => proj.id === activeId);
            document.getElementById('editNameInput').value = p.name;
            document.getElementById('editNameModal').classList.remove('hidden');
        }
        function closeEditNameModal() { document.getElementById('editNameModal').classList.add('hidden'); }
        function confirmEditName() {
            const newName = document.getElementById('editNameInput').value;
            if(newName.trim()) {
                projects.find(proj => proj.id === activeId).name = newName;
                saveData(); renderSidebar(); renderContent(); closeEditNameModal();
            }
        }

        // --- 新增項目按鈕功能 ---
        function promptAddItem(key) {
            const text = prompt("請輸入新檢查項目：");
            if (text && text.trim()) {
                projects.find(p => p.id === activeId).checklist[key].push({ text: text, done: false });
                saveData(); renderContent();
            }
        }

        // --- 範例選擇預覽 ---
        function selectTemplate(name) {
            const t = config.templates[name];
            tempItems = { pre: t.pre.map(text => ({ text, active: true })), mid: t.mid.map(text => ({ text, active: true })), post: t.post.map(text => ({ text, active: true })) };
            document.getElementById('templatePreview').classList.remove('hidden');
            renderPreviewList();
        }

        function renderPreviewList() {
            const preview = document.getElementById('templatePreview');
            let html = "";
            ['pre', 'mid', 'post'].forEach(k => {
                const label = k==='pre'?'行前':k==='mid'?'執行':'撤場';
                html += `<div class="bg-[#1a1b1c] p-4 rounded-xl border border-[#333537]">
                    <div class="flex justify-between items-center mb-4"><span class="text-xs text-blue-400 font-bold">${label}</span><button onclick="addTempItem('${k}')" class="text-[10px] bg-[#333537] px-2 py-1 rounded">+ 項目</button></div>`;
                tempItems[k].forEach((item, i) => {
                    html += `<div class="flex items-center gap-3 mb-2">
                        <input type="checkbox" ${item.active?'checked':''} onchange="tempItems.${k}[${i}].active=this.checked">
                        <input type="text" value="${item.text}" oninput="tempItems.${k}[${i}].text=this.value" class="bg-transparent border-b border-transparent focus:border-[#8ab4f8] outline-none text-sm flex-1">
                    </div>`;
                });
                html += `</div>`;
            });
            preview.innerHTML = html;
        }

        function addTempItem(k) { tempItems[k].push({ text: "新項目", active: true }); renderPreviewList(); }

        // --- 核心工具 ---
        function switchProject(id) { activeId = id; renderSidebar(); renderContent(); if(window.innerWidth < 1024) toggleSidebar(); }
        function toggleSidebar() { document.getElementById('sidebar').classList.toggle('active'); }
        function openNewProjectModal() { document.getElementById('projectModal').classList.remove('hidden'); document.getElementById('newProjName').value=''; document.getElementById('templatePreview').classList.add('hidden'); }
        function closeProjectModal() { document.getElementById('projectModal').classList.add('hidden'); }
        function showAdmin() { document.getElementById('adminPanel').classList.remove('hidden'); }
        function hideAdmin() { document.getElementById('adminPanel').classList.add('hidden'); }
        function toggleItem(k, i) { projects.find(p=>p.id===activeId).checklist[k][i].done = !projects.find(p=>p.id===activeId).checklist[k][i].done; saveData(); renderContent(); }
        function removeItem(k, i) { if(confirm('刪除此項？')){ projects.find(p=>p.id===activeId).checklist[k].splice(i, 1); saveData(); renderContent(); } }
        function saveData() { localStorage.setItem('audio_projects', JSON.stringify(projects)); localStorage.setItem('audio_config', JSON.stringify(config)); }
        function deleteProject(id) { if(confirm('刪除案子？')){ projects=projects.filter(p=>p.id!==id); activeId=projects.length?projects[0].id:null; saveData(); renderSidebar(); renderContent(); } }
        function confirmCreateProject() {
            const name = document.getElementById('newProjName').value || "未命名";
            const newP = { id: Date.now(), name, checklist: { pre: tempItems.pre.filter(i=>i.active).map(i=>({text:i.text, done:false})), mid: tempItems.mid.filter(i=>i.active).map(i=>({text:i.text, done:false})), post: tempItems.post.filter(i=>i.active).map(i=>({text:i.text, done:false})) } };
            projects.unshift(newP); activeId = newP.id; saveData(); closeProjectModal(); renderSidebar(); renderContent();
        }
        function saveJsonConfig() { try { config=JSON.parse(document.getElementById('jsonInputArea').value); saveData(); renderTemplatePicker(); hideAdmin(); } catch(e){alert('格式錯誤');} }
        function renderTemplatePicker() {
            const picker = document.getElementById('templatePicker');
            picker.innerHTML = Object.keys(config.templates).map(name => `<button onclick="selectTemplate('${name}')" class="px-4 py-2 border border-[#444746] rounded-full text-xs hover:bg-[#333537] transition-all">${name}</button>`).join('');
        }
        function resetToDefault() { if(confirm('恢復？')){ config=initialDefault; document.getElementById('jsonInputArea').value=JSON.stringify(config,null,4); saveData(); renderTemplatePicker(); } }

        init();
    </script>
</body>
</html>
