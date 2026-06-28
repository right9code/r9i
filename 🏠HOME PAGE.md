---
cssclass: homepage
---

# 🏠 Home

> [!quote] `= "Today is " + dateformat(date(today), "EEEE, MMMM dd, yyyy")`

---

## ⚡ Quick Actions
| | | | |
|---|---|---|---|
| 📓 [[11 - PRIVATE/1102 - Journal/2026/Daily Notes/\|Daily Note]] | 📝 New Note `Alt+N` | ✏️ Excalidraw `Alt+D` | 🔍 Search `Ctrl+O` |

---

## 🗂️ Vault Navigator

```dataviewjs
// --- Inject/Update Styles for Both Navigators ---
const existingStyle = document.getElementById("hp-tree-styles");
if (existingStyle) {
    existingStyle.remove();
}
const styleEl = document.createElement("style");
styleEl.id = "hp-tree-styles";
styleEl.textContent = `
    .hp-navigator-section-title {
        font-size: 1.1em;
        font-weight: bold;
        margin-top: 20px;
        margin-bottom: 10px;
        color: var(--text-normal);
        border-bottom: 1px solid var(--background-modifier-border);
        padding-bottom: 6px;
    }

    /* Column Styles */
    .hp-nav-container {
        display: flex;
        border: 1px solid var(--background-modifier-border);
        border-radius: 12px;
        background-color: var(--background-secondary);
        overflow-x: auto;
        min-height: 320px;
        max-height: 480px;
        margin-bottom: 20px;
        scroll-behavior: smooth;
    }
    
    .hp1-nav-column {
        display: flex;
        flex-direction: column;
        width: 200px;
        min-width: 200px;
        border-right: 1px solid var(--background-modifier-border);
        background-color: var(--background-primary);
        transition: width 0.3s cubic-bezier(0.25, 0.8, 0.25, 1), 
                    min-width 0.3s cubic-bezier(0.25, 0.8, 0.25, 1), 
                    opacity 0.25s ease,
                    padding 0.3s ease;
        overflow-y: auto;
        overflow-x: hidden;
        box-sizing: border-box;
        flex-shrink: 0;
        opacity: 1;
    }
    .hp1-nav-column.hp1-column-collapsed {
        width: 45px !important;
        min-width: 45px !important;
        background-color: var(--background-secondary-alt);
        overflow-x: hidden;
        overflow-y: hidden;
    }
    .hp1-nav-column.hp1-column-hidden {
        width: 0 !important;
        min-width: 0 !important;
        opacity: 0 !important;
        pointer-events: none !important;
        border-right: none !important;
        overflow-x: hidden;
        overflow-y: hidden;
    }
    .hp1-nav-column:last-child {
        border-right: none;
    }

    /* Common Column Items (Namespaced) */
    .hp1-col-item {
        padding: 6px 12px;
        font-size: 0.9em;
        cursor: pointer;
        display: flex;
        flex-direction: column;
        align-items: stretch;
        justify-content: center;
        border-radius: 6px;
        margin: 2px 6px;
        transition: background-color 0.2s ease, color 0.2s ease, padding 0.2s ease, margin 0.2s ease;
        user-select: none;
        color: var(--text-normal);
        box-sizing: border-box;
        min-height: 48px;
    }
    .hp1-col-item:hover {
        background-color: var(--background-modifier-hover);
        color: var(--interactive-accent);
    }
    .hp1-col-item.active {
        background-color: var(--background-modifier-active-hover);
        color: var(--interactive-accent);
        font-weight: bold;
    }
    
    .hp1-col-item-row {
        display: flex;
        align-items: center;
        gap: 8px;
        width: 100%;
    }
    
    /* Sizing fix for folder text display */
    .hp1-col-item-text {
        display: block !important;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
        flex-grow: 1;
        min-width: 0;
    }
    .hp1-col-item-text a {
        text-decoration: none !important;
        color: inherit !important;
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
    
    .hp1-col-item-icon {
        flex-shrink: 0;
    }
    
    /* Collapsed Column Items for NAV 1 */
    .hp1-column-collapsed .hp1-col-item {
        padding: 8px 0 !important;
        justify-content: center !important;
        margin: 2px 2px !important;
        min-height: 0 !important;
        height: 36px !important;
    }
    .hp1-column-collapsed .hp1-col-item-row {
        justify-content: center !important;
    }
    .hp1-column-collapsed .hp1-col-item-icon {
        margin: 0 !important;
    }
    .hp1-column-collapsed .hp1-col-item-text {
        display: none !important;
    }
    .hp1-column-collapsed .hp1-col-controls {
        display: none !important;
    }
    
    /* Common Controls and Buttons */
    .hp1-col-controls {
        display: flex;
        align-items: center;
        gap: 10px;
        margin-top: 4px;
        padding-left: 20px; /* aligns under the text */
        opacity: 0;
        transition: opacity 0.2s ease;
    }
    .hp1-col-item:hover .hp1-col-controls {
        opacity: 0.8;
    }
    .hp1-col-item:hover .hp1-col-controls:hover {
        opacity: 1;
    }
    .hp1-col-controls:has(.active) {
        opacity: 0.5;
    }
    .hp1-col-btn {
        cursor: pointer;
        padding: 2px;
        font-size: 0.95em;
        transition: transform 0.1s ease;
        user-select: none;
    }
    .hp1-col-btn:hover {
        transform: scale(1.2);
    }
    .hp1-col-btn.hp-pin-btn.active {
        opacity: 1 !important;
        color: var(--interactive-accent);
    }
    
    /* Unified Search Tree Results (Vertical) */
    .hp-search-active {
        display: block !important;
        padding: 16px;
    }
    .hp-tree-list {
        list-style-type: none !important;
        padding-left: 0 !important;
        margin: 0 !important;
    }
    .hp-tree-folder {
        margin-bottom: 4px;
        list-style-type: none !important;
    }
    .hp-tree-folder-header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 6px 8px;
        font-weight: 600;
        font-size: 0.9em;
        cursor: pointer;
        border-radius: 6px;
        transition: background-color 0.2s ease, color 0.2s ease;
        user-select: none;
        color: var(--text-normal);
    }
    .hp-tree-folder-header:hover {
        background-color: var(--background-modifier-hover);
        color: var(--interactive-accent);
    }
    .hp-tree-folder-title {
        display: flex;
        align-items: center;
        gap: 6px;
    }
    .hp-tree-folder-toggle {
        font-size: 0.8em;
        width: 12px;
        display: inline-block;
        transition: transform 0.2s ease;
        color: var(--text-muted);
    }
    .hp1-tree-folder.is-open > .hp-tree-folder-header .hp-tree-folder-toggle {
        transform: rotate(90deg);
    }
    .hp-tree-folder-content {
        display: none;
        list-style-type: none !important;
        padding-left: 16px !important;
        margin-left: 6px !important;
        border-left: 1px dashed var(--background-modifier-border);
        margin-top: 2px;
    }
    .hp1-tree-folder.is-open > .hp-tree-folder-content {
        display: block;
    }
    .hp-tree-file {
        padding: 4px 8px !important;
        font-size: 0.88em;
        display: flex;
        justify-content: space-between;
        align-items: center;
        border-radius: 4px;
        transition: background-color 0.2s ease;
        list-style-type: none !important;
    }
    .hp-tree-file:hover {
        background-color: var(--background-modifier-hover);
    }
    .hp-tree-file-title {
        display: flex;
        align-items: center;
        gap: 6px;
        flex-grow: 1;
        overflow: hidden;
    }
    .hp-tree-file-title a {
        text-decoration: none !important;
        color: var(--text-muted) !important;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
    }
    .hp-tree-file-title a:hover {
        color: var(--interactive-accent) !important;
    }
    .hp-tree-controls {
        display: flex;
        align-items: center;
        gap: 8px;
        opacity: 0;
        transition: opacity 0.2s ease;
    }
    .hp-tree-folder-header:hover .hp-tree-controls,
    .hp-tree-file:hover .hp-tree-controls {
        opacity: 0.6;
    }
    .hp-tree-folder-header:hover .hp-tree-controls:hover,
    .hp-tree-file:hover .hp-tree-controls:hover {
        opacity: 1;
    }
    .hp-tree-btn {
        cursor: pointer;
        padding: 2px;
        font-size: 0.95em;
        transition: transform 0.1s ease;
        user-select: none;
    }
    .hp-tree-btn:hover {
        transform: scale(1.2);
    }
    .hp-tree-btn.hp-pin-btn.active {
        opacity: 1 !important;
        color: var(--interactive-accent);
    }

    /* INLINE Search dropdown styling (restricted to 5-item max-height, scrollable) */
    #hp-search-results-dropdown {
        display: none;
        width: 100%;
        background-color: var(--background-primary);
        border: 1px solid var(--background-modifier-border);
        border-radius: var(--radius-s);
        max-height: 235px; /* Restrict height to 5 items, allow scrolling for more */
        overflow-y: auto;
        margin-top: 5px;
        margin-bottom: 15px;
        box-shadow: inset 0 2px 4px rgba(0,0,0,0.05);
    }
    .hp-search-result-item {
        padding: 8px 12px;
        border-bottom: 1px solid var(--background-modifier-border);
        cursor: pointer;
        display: flex;
        flex-direction: column;
        transition: background-color 0.2s ease;
    }
    .hp-search-result-item:hover {
        background-color: var(--background-secondary-alt);
    }
`;
document.head.appendChild(styleEl);

// --- Configuration & Global State ---
let treeDivElement1 = null;
let searchInputElement = null;
let searchResultsContainer = null;
let currentFileSortOrder = 'name_desc';
const DEFAULT_CREATION_FOLDER = "07 - ROUGH NOTES";
let currentSearchTerm = "";
let columns1El = []; // Persistent columns array

// Pinned Items
let pinnedItems = [];
try {
    pinnedItems = JSON.parse(localStorage.getItem("r9i-pinned-items")) || [];
} catch (e) {
    pinnedItems = [];
}

function savePinnedItems() {
    localStorage.setItem("r9i-pinned-items", JSON.stringify(pinnedItems));
}

// Categories list
const categoriesList = [
  { name: "BACKEND", key: "01 - BACKEND", icon: "⚙️" },
  { name: "WIKI", key: "03 - WIKI", icon: "📖" },
  { name: "ACADEMICS", key: "05 - ACADEMICS", icon: "📚" },
  { name: "ROUGH NOTES", key: "07 - ROUGH NOTES", icon: "📝" },
  { name: "PROJECTS", key: "09 - PROJECTS", icon: "🎯" },
  { name: "PRIVATE", key: "11 - PRIVATE", icon: "🔒" }
];

// NAVIGATOR 1 States (Grandparent-Collapsing)
let activePath1 = [];
if (pinnedItems.length > 0) activePath1 = ["pinned"];
else activePath1 = ["09 - PROJECTS"];

let hoveredColumnIndex1 = 0;
let hoverTimeout1 = null;
let isLayoutLock1 = false;
let lockTimeout1 = null;

function updateActivePath1(newPath) {
    activePath1 = newPath;
    
    // Lock transitions
    isLayoutLock1 = true;
    clearTimeout(lockTimeout1);
    lockTimeout1 = setTimeout(() => {
        isLayoutLock1 = false;
    }, 350);
    
    applyFiltersAndRender1();
}

// --- Common Helper Functions ---
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}

function getMtime(path) {
    const file = app.vault.getAbstractFileByPath(path);
    if (file && file.stat) {
        return file.stat.mtime;
    }
    return 0;
}

function readFolderContents(folderPath) {
    const folder = app.vault.getAbstractFileByPath(folderPath);
    if (!folder || !folder.children) return [];
    
    const items = [];
    folder.children.forEach(child => {
        if (child.name.startsWith("00 - Unsorted") || child.name.startsWith(".")) {
            return;
        }
        if (child.children) {
            items.push({
                name: child.name,
                path: child.path,
                type: 'folder'
            });
        } else if (child.name.endsWith(".md")) {
            items.push({
                name: child.name.replace(/\.md$/, ""),
                path: child.path,
                type: 'file'
            });
        }
    });
    
    items.sort((a, b) => {
        if (a.type !== b.type) {
            return a.type === 'folder' ? -1 : 1;
        }
        if (currentFileSortOrder.startsWith("name")) {
            const comp = a.name.localeCompare(b.name, undefined, { sensitivity: 'base', numeric: true });
            return currentFileSortOrder === "name_desc" ? -comp : comp;
        } else {
            const tA = getMtime(a.path);
            const tB = getMtime(b.path);
            const comp = tB - tA;
            return currentFileSortOrder === "modified_asc" ? -comp : comp;
        }
    });
    
    return items;
}

// --- Modals ---
class CreateFolderModal extends obsidian.Modal {
    constructor(app, parentFolderPath, onSubmitCallback) {
        super(app);
        this.parentFolderPath = parentFolderPath;
        this.onSubmitCallback = onSubmitCallback;
        this.folderName = "";
    }
    onOpen() {
        const { contentEl } = this;
        contentEl.empty();
        contentEl.createEl('h3', { text: `Create Folder in ${this.parentFolderPath.split('/').pop()}` });
        
        contentEl.createEl('label', { text: 'Folder Name:', attr: { style: 'font-weight: bold; display: block; margin-top: 12px;' } });
        const inputEl = contentEl.createEl('input', { 
            type: 'text', 
            attr: { style: 'width: 100%; margin-bottom: 20px; padding: 8px; border-radius: var(--radius-s); border: 1px solid var(--background-modifier-border);' } 
        });
        inputEl.placeholder = "Enter folder name...";
        inputEl.addEventListener('input', (e) => this.folderName = e.target.value);
        inputEl.addEventListener('keydown', (e) => { if (e.key === 'Enter') { e.preventDefault(); this.submit(); } });
        
        const btnRow = contentEl.createDiv({ attr: { style: 'display: flex; justify-content: flex-end; margin-top: 20px;' } });
        const createBtn = btnRow.createEl('button', { text: 'Create', cls: 'mod-cta' });
        createBtn.style.marginRight = '8px';
        createBtn.addEventListener('click', () => this.submit());
        
        const cancelBtn = btnRow.createEl('button', { text: 'Cancel' });
        cancelBtn.addEventListener('click', () => this.close());
        
        setTimeout(() => inputEl.focus(), 50);
    }
    submit() {
        if (!this.folderName.trim()) {
            new obsidian.Notice("Folder name cannot be empty.");
            return;
        }
        this.onSubmitCallback(this.folderName.trim());
        this.close();
    }
    onClose() {
        this.contentEl.empty();
    }
}

class CreateNoteModal extends obsidian.Modal {
    constructor(app, folderPath, templates, onSubmitCallback) {
        super(app);
        this.folderPath = folderPath;
        this.templates = templates;
        this.onSubmitCallback = onSubmitCallback;
        this.fileName = "";
        this.selectedTemplate = "";
    }
    onOpen() {
        const { contentEl } = this;
        contentEl.empty();
        contentEl.createEl('h3', { text: `Create Note in ${this.folderPath.split('/').pop()}` });
        
        contentEl.createEl('label', { text: 'Note Title:', attr: { style: 'font-weight: bold; display: block; margin-top: 12px;' } });
        const inputEl = contentEl.createEl('input', { 
            type: 'text', 
            attr: { style: 'width: 100%; margin-bottom: 12px; padding: 8px; border-radius: var(--radius-s); border: 1px solid var(--background-modifier-border);' } 
        });
        inputEl.placeholder = "Enter title...";
        inputEl.addEventListener('input', (e) => this.fileName = e.target.value);
        inputEl.addEventListener('keydown', (e) => { if (e.key === 'Enter') { e.preventDefault(); this.submit(); } });
        
        contentEl.createEl('label', { text: 'Apply Template (Optional):', attr: { style: 'font-weight: bold; display: block; margin-top: 10px;' } });
        const selectEl = contentEl.createEl('select', {
            attr: { style: 'width: 100%; margin-bottom: 15px; padding: 8px; border-radius: var(--radius-s); border: 1px solid var(--background-modifier-border); background-color: var(--background-primary); color: var(--text-normal);' }
        });
        
        const blankOpt = selectEl.createEl('option', { text: '(Blank Note)' });
        blankOpt.value = "";
        
        this.templates.forEach(t => {
            const opt = selectEl.createEl('option', { text: t.name });
            opt.value = t.path;
            selectEl.appendChild(opt);
        });
        selectEl.addEventListener('change', (e) => this.selectedTemplate = e.target.value);
        
        const btnRow = contentEl.createDiv({ attr: { style: 'display: flex; justify-content: flex-end; margin-top: 20px;' } });
        const createBtn = btnRow.createEl('button', { text: 'Create', cls: 'mod-cta' });
        createBtn.style.marginRight = '8px';
        createBtn.addEventListener('click', () => this.submit());
        
        const cancelBtn = btnRow.createEl('button', { text: 'Cancel' });
        cancelBtn.addEventListener('click', () => this.close());
        
        setTimeout(() => inputEl.focus(), 50);
    }
    submit() {
        if (!this.fileName.trim()) {
            new obsidian.Notice("File name cannot be empty.");
            return;
        }
        this.onSubmitCallback(this.fileName.trim(), this.selectedTemplate);
        this.close();
    }
    onClose() {
        this.contentEl.empty();
    }
}

// --- Search Tree helper functions ---
function buildTree(folder) {
    const node = {
        name: folder.name,
        path: folder.path,
        type: 'folder',
        children: []
    };
    if (folder.children) {
        folder.children.forEach(child => {
            if (child.name.startsWith("00 - Unsorted") || child.name.startsWith(".")) return;
            if (child.children) {
                node.children.push(buildTree(child));
            } else {
                if (child.name.endsWith(".md")) {
                    node.children.push({
                        name: child.name.replace(/\.md$/, ""),
                        path: child.path,
                        type: 'file'
                    });
                }
            }
        });
    }
    node.children.sort((a, b) => {
        if (a.type !== b.type) return a.type === 'folder' ? -1 : 1;
        if (currentFileSortOrder.startsWith("name")) {
            const comp = a.name.localeCompare(b.name, undefined, { sensitivity: 'base', numeric: true });
            return currentFileSortOrder === "name_desc" ? -comp : comp;
        } else {
            const tA = getMtime(a.path);
            const tB = getMtime(b.path);
            const comp = tB - tA;
            return currentFileSortOrder === "modified_asc" ? -comp : comp;
        }
    });
    return node;
}

function filterTree(node, query, parentMatched = false) {
    const isMatch = parentMatched || node.name.toLowerCase().includes(query);
    if (node.type === 'file') return isMatch ? node : null;
    
    const filteredChildren = [];
    node.children.forEach(child => {
        const filteredChild = filterTree(child, query, isMatch);
        if (filteredChild) filteredChildren.push(filteredChild);
    });
    
    if (isMatch || filteredChildren.length > 0) {
        return {
            ...node,
            children: filteredChildren,
            isExpanded: true
        };
    }
    return null;
}

function renderTree(node, namespace, indentLevel = 0) {
    const ns = namespace; // 'hp1' or 'hp2'
    if (node.type === 'file') {
        const isFilePinned = pinnedItems.some(item => item.path === node.path && item.type === "file");
        const liEl = document.createElement("li");
        liEl.className = "hp-tree-file";
        
        liEl.innerHTML = `
            <div class="hp-tree-file-title">
                <span>📄</span>
                <a class="internal-link" href="${node.path}">${node.name}</a>
            </div>
            <span class="hp-tree-controls">
                <span class="hp-tree-btn hp-pin-btn ${isFilePinned ? 'active' : ''}" data-path="${node.path}" data-name="${node.name}" data-type="file" title="${isFilePinned ? 'Unpin' : 'Pin'} Note">📌</span>
            </span>
        `;
        return liEl;
    } else {
        const isFolderPinned = pinnedItems.some(item => item.path === node.path && item.type === "folder");
        const folderEl = document.createElement("li");
        folderEl.className = `hp-tree-folder ${ns}-tree-folder`;
        if (node.isExpanded) {
            folderEl.classList.add("is-open");
        }
        
        const headerEl = document.createElement("div");
        headerEl.className = "hp-tree-folder-header";
        
        headerEl.innerHTML = `
            <div class="hp-tree-folder-title">
                <span class="hp-tree-folder-toggle">▶</span>
                <span class="hp-folder-icon">📁</span>
                <span>${node.name}</span>
            </div>
            <span class="hp-tree-controls">
                <span class="hp-tree-btn hp-add-note-btn" data-path="${node.path}" title="Add note to folder">📄➕</span>
                <span class="hp-tree-btn hp-add-folder-btn" data-path="${node.path}" title="Add folder here">📁➕</span>
                <span class="hp-tree-btn hp-pin-btn ${isFolderPinned ? 'active' : ''}" data-path="${node.path}" data-name="${node.name}" data-type="folder" title="${isFolderPinned ? 'Unpin' : 'Pin'} Folder">📌</span>
            </span>
        `;
        
        headerEl.onclick = (e) => {
            if (e.target.closest('.hp-tree-controls')) return;
            folderEl.classList.toggle("is-open");
            const iconEl = folderEl.querySelector(".hp-folder-icon");
            if (iconEl) {
                iconEl.textContent = folderEl.classList.contains("is-open") ? "📂" : "📁";
            }
        };
        
        folderEl.appendChild(headerEl);
        
        const contentEl = document.createElement("ul");
        contentEl.className = "hp-tree-folder-content";
        
        if (node.children && node.children.length > 0) {
            node.children.forEach(child => {
                const childEl = renderTree(child, ns, indentLevel + 1);
                contentEl.appendChild(childEl);
            });
        } else {
            const emptyEl = document.createElement("li");
            emptyEl.className = "hp-tree-file";
            emptyEl.style.fontStyle = "italic";
            emptyEl.style.color = "var(--text-faint)";
            emptyEl.textContent = "No notes/subfolders";
            contentEl.appendChild(emptyEl);
        }
        
        folderEl.appendChild(contentEl);
        
        const iconEl = folderEl.querySelector(".hp-folder-icon");
        if (iconEl && folderEl.classList.contains("is-open")) {
            iconEl.textContent = "📂";
        }
        
        return folderEl;
    }
}

// --- Render NAVIGATOR 1 (Grandparent Collapsing) ---
function applyFiltersAndRender1() {
    clearTimeout(hoverTimeout1);
    if (!treeDivElement1) return;
    
    // A. Search Active Mode -> Unified vertical tree
    if (currentSearchTerm) {
        treeDivElement1.innerHTML = '';
        columns1El = [];
        
        const containerEl = document.createElement("div");
        containerEl.id = "tree-navigator-container-1";
        containerEl.className = "hp-nav-container hp-search-active";
        
        const panelsContainerEl = document.createElement("div");
        panelsContainerEl.className = "hp-nav-panels-container hp-search-active";
        
        const panelEl = document.createElement("div");
        panelEl.className = "hp-nav-content-panel active";
        const ulEl = document.createElement("ul");
        ulEl.className = "hp-tree-list";
        
        categoriesList.forEach(cat => {
            const categoryFolder = app.vault.getAbstractFileByPath(cat.key);
            if (categoryFolder) {
                const treeNode = buildTree(categoryFolder);
                const filteredNode = filterTree(treeNode, currentSearchTerm);
                if (filteredNode && filteredNode.children && filteredNode.children.length > 0) {
                    const categoryEl = renderTree(filteredNode, 'hp1', 0);
                    ulEl.appendChild(categoryEl);
                }
            }
        });
        
        if (ulEl.children.length === 0) {
            const emptyEl = document.createElement("div");
            emptyEl.style.padding = "20px";
            emptyEl.style.textAlign = "center";
            emptyEl.style.color = "var(--text-faint)";
            emptyEl.style.fontStyle = "italic";
            emptyEl.textContent = "No matches found.";
            panelEl.appendChild(emptyEl);
        } else {
            panelEl.appendChild(ulEl);
        }
        panelsContainerEl.appendChild(panelEl);
        containerEl.appendChild(panelsContainerEl);
        treeDivElement1.appendChild(containerEl);
        return;
    }
    
    // B. Normal Mode -> Persistent Columns (smooth width scaling animations)
    let container = document.getElementById("tree-navigator-container-1");
    if (!container || container.classList.contains("hp-search-active")) {
        treeDivElement1.innerHTML = '';
        
        container = document.createElement("div");
        container.id = "tree-navigator-container-1";
        container.className = "hp-nav-container";
        
        columns1El = [];
        for (let c = 0; c <= 5; c++) {
            const colEl = document.createElement("div");
            colEl.className = `hp1-nav-column hp1-column-${c}`;
            
            // Set initial hidden state for deep columns
            if (c > 1) {
                colEl.classList.add("hp1-column-hidden");
            }
            
            // Global column enter handler (hover intent logic)
            colEl.addEventListener("mouseenter", () => {
                if (isLayoutLock1) return;
                
                clearTimeout(hoverTimeout1);
                hoverTimeout1 = setTimeout(() => {
                    if (hoveredColumnIndex1 !== c) {
                        hoveredColumnIndex1 = c;
                        
                        // Lock layouts temporarily to avoid jumps undercursor
                        isLayoutLock1 = true;
                        setTimeout(() => { isLayoutLock1 = false; }, 350);
                        
                        if (c === 0) {
                            updateActivePath1([activePath1[0]]);
                        } else {
                            if (activePath1.length > c) {
                                updateActivePath1(activePath1.slice(0, c));
                            } else {
                                applyFiltersAndRender1();
                            }
                        }
                    }
                }, 150); // 150ms delay to verify intentional navigation
            });
            
            container.appendChild(colEl);
            columns1El.push(colEl);
        }
        
        treeDivElement1.appendChild(container);
    }
    
    // Update classes and contents of columns
    for (let c = 0; c <= 5; c++) {
        const colEl = columns1El[c];
        if (!colEl) continue;
        
        let isHidden = false;
        let isCollapsed = false;
        
        if (c === 0) {
            isCollapsed = (hoveredColumnIndex1 > 1);
        } else {
            isHidden = (c > activePath1.length);
            isCollapsed = (!isHidden && c < hoveredColumnIndex1 - 1);
        }
        
        // Transition classes
        colEl.className = `hp1-nav-column hp1-column-${c}`;
        if (isHidden) {
            colEl.classList.add("hp1-column-hidden");
        } else if (isCollapsed) {
            colEl.classList.add("hp1-column-collapsed");
        }
        
        // Render or update items (Hidden columns preserve their old elements to collapse smoothly)
        if (!isHidden) {
            colEl.innerHTML = '';
            
            const listEl = document.createElement("div");
            listEl.style.display = "flex";
            listEl.style.flexDirection = "column";
            listEl.style.padding = "10px 0";
            
            if (c === 0) {
                // Sidebar category menu
                const activeCat = activePath1[0];
                const availableCategories = [];
                if (pinnedItems.length > 0) {
                    availableCategories.push({ name: "PINNED", key: "pinned", icon: "📌" });
                }
                categoriesList.forEach(cat => availableCategories.push(cat));
                
                availableCategories.forEach(cat => {
                    const itemEl = document.createElement("div");
                    itemEl.className = "hp1-col-item";
                    if (cat.key === activeCat) {
                        itemEl.classList.add("active");
                    }
                    
                    itemEl.innerHTML = `
                        <div class="hp1-col-item-row">
                            <span class="hp1-col-item-icon">${cat.icon}</span>
                            <span class="hp1-col-item-text">${cat.name}</span>
                        </div>
                    `;
                    
                    // Hover intent delay for main menu items
                    itemEl.addEventListener("mouseenter", () => {
                        if (isLayoutLock1) return;
                        
                        clearTimeout(hoverTimeout1);
                        hoverTimeout1 = setTimeout(() => {
                            if (activePath1[0] !== cat.key) {
                                hoveredColumnIndex1 = 0;
                                updateActivePath1([cat.key]);
                            }
                        }, 150); // 150ms delay prevents accidental trigger when mouse sweeps backwards
                    });
                    
                    listEl.appendChild(itemEl);
                });
            } else {
                // Normal columns: Folders and Notes list
                const currentPath = activePath1[c - 1];
                let items = [];
                if (currentPath === "pinned") {
                    pinnedItems.forEach(item => {
                        items.push({ name: item.name, path: item.path, type: item.type });
                    });
                } else {
                    items = readFolderContents(currentPath);
                }
                
                if (items.length === 0) {
                    const emptyEl = document.createElement("div");
                    emptyEl.className = "hp1-col-item";
                    emptyEl.style.fontStyle = "italic";
                    emptyEl.style.color = "var(--text-faint)";
                    
                    emptyEl.innerHTML = `
                        <div class="hp1-col-item-row">
                            <span class="hp1-col-item-text">Empty</span>
                        </div>
                    `;
                    listEl.appendChild(emptyEl);
                } else {
                    const nextActivePath = activePath1[c];
                    
                    items.forEach(item => {
                        const itemEl = document.createElement("div");
                        itemEl.className = "hp1-col-item";
                        if (item.path === nextActivePath) {
                            itemEl.classList.add("active");
                        }
                        
                        const isPinned = pinnedItems.some(p => p.path === item.path && p.type === item.type);
                        const icon = item.type === 'folder' ? '📁' : '📄';
                        
                        let controlsHtml = "";
                        if (item.type === 'folder') {
                            controlsHtml = `
                                <div class="hp1-col-controls">
                                    <span class="hp1-col-btn hp-add-note-btn" data-path="${item.path}" title="Create note inside">📄➕</span>
                                    <span class="hp1-col-btn hp-add-folder-btn" data-path="${item.path}" title="Create folder inside">📁➕</span>
                                    <span class="hp1-col-btn hp-pin-btn ${isPinned ? 'active' : ''}" data-path="${item.path}" data-name="${item.name}" data-type="folder" title="${isPinned ? 'Unpin' : 'Pin'} Folder">📌</span>
                                </div>
                            `;
                        } else {
                            controlsHtml = `
                                <div class="hp1-col-controls">
                                    <span class="hp1-col-btn hp-pin-btn ${isPinned ? 'active' : ''}" data-path="${item.path}" data-name="${item.name}" data-type="file" title="${isPinned ? 'Unpin' : 'Pin'} Note">📌</span>
                                </div>
                            `;
                        }
                        
                        let titleHtml = "";
                        if (item.type === 'file') {
                            titleHtml = `
                                <div class="hp1-col-item-row">
                                    <span class="hp1-col-item-icon">${icon}</span>
                                    <span class="hp1-col-item-text"><a class="internal-link" href="${item.path}">${item.name}</a></span>
                                </div>
                            `;
                        } else {
                            titleHtml = `
                                <div class="hp1-col-item-row">
                                    <span class="hp1-col-item-icon">${icon}</span>
                                    <span class="hp1-col-item-text">${item.name}</span>
                                </div>
                            `;
                        }
                        
                        itemEl.innerHTML = `
                            ${titleHtml}
                            ${controlsHtml}
                        `;
                        
                        if (item.type === 'folder') {
                            itemEl.addEventListener("mouseenter", () => {
                                if (isLayoutLock1) return;
                                
                                clearTimeout(hoverTimeout1);
                                hoverTimeout1 = setTimeout(() => {
                                    if (activePath1.length > c && activePath1[c] === item.path) {
                                        return;
                                    }
                                    
                                    const newPath = activePath1.slice(0, c);
                                    newPath.push(item.path);
                                    updateActivePath1(newPath);
                                }, 150); // 150ms hover delay
                            });
                        }
                        
                        listEl.appendChild(itemEl);
                    });
                }
            }
            colEl.appendChild(listEl);
        }
    }
    
    // Auto scroll right
    setTimeout(() => {
        const container = document.getElementById("tree-navigator-container-1");
        if (container) {
            container.scrollTo({
                left: container.scrollWidth,
                behavior: 'smooth'
            });
        }
    }, 50);
}

// --- Create Control UI Elements ---
const commonMutedButtonStyle = {
    padding: '6px 12px', border: '1px solid var(--background-modifier-border, #ccc)',
    borderRadius: 'var(--radius-s, 4px)', backgroundColor: 'var(--background-secondary-alt, var(--background-secondary, #f0f0f0))',
    color: 'var(--text-muted, var(--text-normal, #333))', cursor: 'pointer',
    marginRight: '5px', marginBottom: '5px'
};
function applyStyles(element, styles) { for (const property in styles) { element.style[property] = styles[property]; } }

const treeNavWrapper = document.createElement('div');
treeNavWrapper.id = "tree-navigator-wrapper";

// Reset Row
const homeIconContainer = document.createElement('div');
homeIconContainer.style.marginBottom = '10px';
homeIconContainer.style.paddingBottom = '5px';
homeIconContainer.style.textAlign = 'left';

const homeIconButton = document.createElement('span');
homeIconButton.textContent = '🏠';
homeIconButton.title = 'Reset Views & Search';
homeIconButton.style.fontSize = '1.8em';
homeIconButton.style.cursor = 'pointer';
homeIconButton.style.padding = '0 5px';
homeIconButton.style.lineHeight = '1';

homeIconButton.onclick = () => {
    if (searchInputElement) searchInputElement.value = '';
    if (searchResultsContainer) searchResultsContainer.style.display = 'none';
    currentSearchTerm = '';
    currentFileSortOrder = 'name_desc';
    if (nameSortButtonElement) nameSortButtonElement.textContent = 'Sort by Name (Z-A)';
    if (modifiedSortButtonElement) modifiedSortButtonElement.textContent = 'Sort by Modified (Newest)';
    
    // Reset Nav 1
    hoveredColumnIndex1 = 0;
    const path1 = pinnedItems.length > 0 ? ["pinned"] : ["09 - PROJECTS"];
    updateActivePath1(path1);
};
homeIconContainer.appendChild(homeIconButton);
treeNavWrapper.appendChild(homeIconContainer);

// Search Input Bar
const searchInputContainer = document.createElement('div');
searchInputContainer.style.display = 'flex';
searchInputContainer.style.alignItems = 'center';
searchInputContainer.style.position = 'relative';
searchInputContainer.style.marginBottom = '10px';
searchInputContainer.style.paddingBottom = '10px';
searchInputContainer.style.borderBottom = '1px solid var(--text-faint)';

searchInputElement = document.createElement('input');
searchInputElement.type = 'text';
searchInputElement.placeholder = 'Search folders / global files...';
searchInputElement.style.flexGrow = '1';
searchInputElement.style.padding = '8px';
searchInputElement.style.border = '1px solid var(--background-modifier-border)';
searchInputElement.style.borderRadius = 'var(--radius-s)';

// Autocomplete Dropdown (INLINE, same level)
searchResultsContainer = document.createElement('div');
searchResultsContainer.id = "hp-search-results-dropdown";

searchInputElement.addEventListener('input', debounce((e) => {
    const query = e.target.value.trim().toLowerCase();
    currentSearchTerm = query;
    
    applyFiltersAndRender1();
    
    if (query.length >= 1) {
        // Search all pages but slice to top 30 scrollable list entries (restricted in height by CSS)
        const matches = dv.pages()
            .filter(p => p.file.name.toLowerCase().includes(query) || p.file.path.toLowerCase().includes(query))
            .slice(0, 30);
            
        if (matches.length > 0) {
            searchResultsContainer.innerHTML = "";
            matches.forEach(p => {
                const item = document.createElement("div");
                item.className = "hp-search-result-item";
                
                const title = document.createElement("span");
                title.style.fontWeight = "bold";
                title.style.color = "var(--text-normal)";
                title.textContent = p.file.name;
                
                const path = document.createElement("span");
                path.style.fontSize = "0.78em";
                path.style.color = "var(--text-muted)";
                path.textContent = p.file.path;
                
                item.appendChild(title);
                item.appendChild(path);
                
                item.addEventListener("click", () => {
                    app.workspace.getLeaf(true).openFile(p.file.toAbstractFile());
                    searchResultsContainer.style.display = "none";
                    searchInputElement.value = "";
                    currentSearchTerm = "";
                    
                    applyFiltersAndRender1();
                });
                searchResultsContainer.appendChild(item);
            });
            searchResultsContainer.style.display = "block";
        } else {
            searchResultsContainer.innerHTML = `<div style="padding: 10px; color: var(--text-faint); font-style: italic; text-align: center;">No matches found</div>`;
            searchResultsContainer.style.display = "block";
        }
    } else {
        searchResultsContainer.style.display = "none";
    }
}, 250));

searchInputContainer.appendChild(searchInputElement);

const clearSearchButton = document.createElement('button');
clearSearchButton.textContent = '✖';
applyStyles(clearSearchButton, { ...commonMutedButtonStyle, padding: '6px 8px', marginLeft: '5px', marginRight: '0', marginBottom: '0' });
clearSearchButton.onclick = () => {
    if (searchInputElement) searchInputElement.value = '';
    if (searchResultsContainer) searchResultsContainer.style.display = 'none';
    currentSearchTerm = '';
    
    applyFiltersAndRender1();
};
searchInputContainer.appendChild(clearSearchButton);
treeNavWrapper.appendChild(searchInputContainer);
treeNavWrapper.appendChild(searchResultsContainer); // Autocomplete inserted here inline

// Sort Row
const sortButtonsContainer = document.createElement('div');
sortButtonsContainer.style.marginBottom = '5px';
sortButtonsContainer.style.paddingBottom = '5px';
sortButtonsContainer.style.borderBottom = '1px solid var(--text-faint)';
sortButtonsContainer.style.display = 'flex';
sortButtonsContainer.style.flexWrap = 'wrap';
treeNavWrapper.appendChild(sortButtonsContainer);

let nameSortButtonElement = document.createElement('button');
nameSortButtonElement.textContent = 'Sort by Name (Z-A)';
applyStyles(nameSortButtonElement, commonMutedButtonStyle);
nameSortButtonElement.onclick = () => {
    if (currentFileSortOrder === 'name_asc') { currentFileSortOrder = 'name_desc'; nameSortButtonElement.textContent = 'Sort by Name (Z-A)'; }
    else { currentFileSortOrder = 'name_asc'; nameSortButtonElement.textContent = 'Sort by Name (A-Z)'; }
    applyFiltersAndRender1();
};
sortButtonsContainer.appendChild(nameSortButtonElement);

let modifiedSortButtonElement = document.createElement('button');
modifiedSortButtonElement.textContent = 'Sort by Modified (Newest)';
applyStyles(modifiedSortButtonElement, commonMutedButtonStyle);
modifiedSortButtonElement.onclick = () => {
    if (currentFileSortOrder === 'modified_desc') { currentFileSortOrder = 'modified_asc'; modifiedSortButtonElement.textContent = 'Sort by Modified (Oldest)'; }
    else { currentFileSortOrder = 'modified_desc'; modifiedSortButtonElement.textContent = 'Sort by Modified (Newest)'; }
    applyFiltersAndRender1();
};
sortButtonsContainer.appendChild(modifiedSortButtonElement);

// Default Creation Handlers
async function handleCreateFile() {
    const path = DEFAULT_CREATION_FOLDER;
    const templates = dv.pages('"01 - BACKEND/0102 - Templates"').map(p => ({
        name: p.file.name,
        path: p.file.path
    }));
    
    const modal = new CreateNoteModal(app, path, templates, async (fileName, templatePath) => {
        if (!fileName.endsWith(".md")) fileName += ".md";
        const fullPath = `${path}/${fileName}`;
        
        if (app.vault.getAbstractFileByPath(fullPath)) {
            new obsidian.Notice("A note with that name already exists in Rough Notes!");
            return;
        }
        
        try {
            const templater = app.plugins.plugins["templater-obsidian"];
            let newFile;
            
            if (templatePath && templater) {
                newFile = await app.vault.create(fullPath, "");
                const tFile = app.vault.getAbstractFileByPath(templatePath);
                await templater.templater.write_template_to_file(tFile, newFile);
            } else if (templatePath) {
                const tFile = app.vault.getAbstractFileByPath(templatePath);
                const tContent = await app.vault.read(tFile);
                newFile = await app.vault.create(fullPath, tContent);
            } else {
                newFile = await app.vault.create(fullPath, "");
            }
            
            new obsidian.Notice(`Note Created: ${fileName}`);
            app.workspace.getLeaf(true).openFile(newFile);
            applyFiltersAndRender1();
        } catch (err) {
            console.error("Error creating note:", err);
            new obsidian.Notice(`Failed to create note: ${err.message}`);
        }
    });
    modal.open();
}

async function handleCreateFolder() {
    const path = DEFAULT_CREATION_FOLDER;
    const modal = new CreateFolderModal(app, path, async (folderName) => {
        const fullPath = `${path}/${folderName}`;
        if (app.vault.getAbstractFileByPath(fullPath)) {
            new obsidian.Notice("A folder/file with that name already exists in Rough Notes!");
            return;
        }
        try {
            await app.vault.createFolder(fullPath);
            new obsidian.Notice(`Folder Created: ${folderName}`);
            applyFiltersAndRender1();
        } catch (err) {
            console.error("Error creating folder:", err);
            new obsidian.Notice(`Failed to create folder: ${err.message}`);
        }
    });
    modal.open();
}

// Action Row
const actionButtonsContainer = document.createElement('div');
actionButtonsContainer.style.marginTop = '10px';
actionButtonsContainer.style.marginBottom = '10px';
actionButtonsContainer.style.display = 'flex';
treeNavWrapper.appendChild(actionButtonsContainer);

const createFileButton = document.createElement('button');
createFileButton.textContent = '➕ Create Note in Rough Notes';
applyStyles(createFileButton, commonMutedButtonStyle);
createFileButton.onclick = handleCreateFile;
actionButtonsContainer.appendChild(createFileButton);

const createFolderButton = document.createElement('button');
createFolderButton.textContent = '📁➕ Create Folder in Rough Notes';
applyStyles(createFolderButton, commonMutedButtonStyle);
createFolderButton.style.marginRight = '0';
createFolderButton.onclick = handleCreateFolder;
actionButtonsContainer.appendChild(createFolderButton);

// Navigator Containers
treeDivElement1 = document.createElement('div');
treeNavWrapper.appendChild(treeDivElement1);

// Handle inline creations & pinning
function setupNavigatorClickListeners(treeDivElement, renderFn) {
    treeDivElement.addEventListener("click", async (e) => {
        const pinBtn = e.target.closest(".hp-pin-btn");
        const addNoteBtn = e.target.closest(".hp-add-note-btn");
        const addFolderBtn = e.target.closest(".hp-add-folder-btn");
        
        if (pinBtn) {
            e.stopPropagation();
            const path = pinBtn.getAttribute("data-path");
            const name = pinBtn.getAttribute("data-name");
            const type = pinBtn.getAttribute("data-type");
            
            const index = pinnedItems.findIndex(item => item.path === path && item.type === type);
            if (index > -1) {
                pinnedItems.splice(index, 1);
                new obsidian.Notice(`Unpinned: ${name || path.split('/').pop()}`);
            } else {
                pinnedItems.push({ type, path, name: name || path.split('/').pop() });
                new obsidian.Notice(`Pinned: ${name || path.split('/').pop()}`);
            }
            savePinnedItems();
            applyFiltersAndRender1();
        }
        
        if (addNoteBtn) {
            e.stopPropagation();
            const path = addNoteBtn.getAttribute("data-path");
            
            const templates = dv.pages('"01 - BACKEND/0102 - Templates"').map(p => ({
                name: p.file.name,
                path: p.file.path
            }));
            
            const modal = new CreateNoteModal(app, path, templates, async (fileName, templatePath) => {
                if (!fileName.endsWith(".md")) fileName += ".md";
                const fullPath = `${path}/${fileName}`;
                
                if (app.vault.getAbstractFileByPath(fullPath)) {
                    new obsidian.Notice("A note with that name already exists in this folder!");
                    return;
                }
                
                try {
                    const templater = app.plugins.plugins["templater-obsidian"];
                    let newFile;
                    
                    if (templatePath && templater) {
                        newFile = await app.vault.create(fullPath, "");
                        const tFile = app.vault.getAbstractFileByPath(templatePath);
                        await templater.templater.write_template_to_file(tFile, newFile);
                    } else if (templatePath) {
                        const tFile = app.vault.getAbstractFileByPath(templatePath);
                        const tContent = await app.vault.read(tFile);
                        newFile = await app.vault.create(fullPath, tContent);
                    } else {
                        newFile = await app.vault.create(fullPath, "");
                    }
                    
                    new obsidian.Notice(`Note Created: ${fileName}`);
                    app.workspace.getLeaf(true).openFile(newFile);
                    applyFiltersAndRender1();
                } catch (err) {
                    console.error("Error creating note:", err);
                    new obsidian.Notice(`Failed to create note: ${err.message}`);
                }
            });
            modal.open();
        }
        
        if (addFolderBtn) {
            e.stopPropagation();
            const path = addFolderBtn.getAttribute("data-path");
            
            const modal = new CreateFolderModal(app, path, async (folderName) => {
                const fullPath = `${path}/${folderName}`;
                
                if (app.vault.getAbstractFileByPath(fullPath)) {
                    new obsidian.Notice("A folder with that name already exists!");
                    return;
                }
                
                try {
                    await app.vault.createFolder(fullPath);
                    new obsidian.Notice(`Folder Created: ${folderName}`);
                    applyFiltersAndRender1();
                } catch (err) {
                    console.error("Error creating folder:", err);
                    new obsidian.Notice(`Failed to create folder: ${err.message}`);
                }
            });
            modal.open();
        }
    });
}

setupNavigatorClickListeners(treeDivElement1, applyFiltersAndRender1);

// Close autocomplete dropdown when clicking outside
document.addEventListener('click', (e) => {
    if (searchResultsContainer && !searchInputContainer.contains(e.target)) {
        searchResultsContainer.style.display = 'none';
    }
});

// Append to Dataview container
dv.container.appendChild(treeNavWrapper);

// --- Initial Render ---
applyFiltersAndRender1();

```

## 📅 Today's Snapshot

```dataviewjs
const today = dv.date("today").toFormat("yyyy-MM-dd");
const year = dv.date("today").toFormat("yyyy");
const dailyPath = `11 - PRIVATE/1102 - Journal/${year}/Daily Notes`;

const todayNote = dv.pages(`"${dailyPath}"`)
    .where(p => p.file.name === today);

if (todayNote.length > 0) {
    const n = todayNote[0];
    dv.table(
        ["", ""],
        [
            ["🔗 Note", `[[${n.file.path}|${n.file.name}]]`],
            ["😴 Sleep", `${n.sleep_time ?? "—"} → ${n.wakeup_time ?? "—"} (${n.sleep_duration ?? "—"})`],
            ["⚡ Energy", `${n.energy_lvl ?? "—"} / 10`],
            ["🍅 Pomos", `${n.pomos ?? 0}`],
            ["📆 Day Type", `${(n.Day_type ?? []).join(", ") || "—"}`]
        ]
    );
} else {
    dv.paragraph("> _No daily note for today. Use `Alt+N` or open Daily Notes to create one._");
}
```

---

## 📊 This Week's Stats

```dataviewjs
const year = dv.date("today").toFormat("yyyy");
const currentWeek = dv.date("today").toFormat("WW");
const dailyPath = `11 - PRIVATE/1102 - Journal/${year}/Daily Notes`;

const weekNotes = dv.pages(`"${dailyPath}"`)
    .where(p => p.week_num == currentWeek && p.year_num == year);

if (weekNotes.length > 0) {
    let totalPomos = 0;
    let totalSleepMins = 0;
    let sleepCount = 0;
    let totalEnergy = 0;
    let energyCount = 0;

    for (let day of weekNotes) {
        totalPomos += parseInt(day.pomos ?? 0) || 0;

        if (day.sleep_duration) {
            const durStr = String(day.sleep_duration);
            const hMatch = durStr.match(/(\d+)\s*h/i);
            const mMatch = durStr.match(/(\d+)\s*m/i);
            let mins = 0;
            if (hMatch) mins += parseInt(hMatch[1]) * 60;
            if (mMatch) mins += parseInt(mMatch[1]);
            if (mins > 0) { totalSleepMins += mins; sleepCount++; }
        }

        if (day.energy_lvl) {
            const e = parseInt(day.energy_lvl);
            if (!isNaN(e)) { totalEnergy += e; energyCount++; }
        }
    }

    const avgSleepH = sleepCount ? Math.floor((totalSleepMins / sleepCount) / 60) : 0;
    const avgSleepM = sleepCount ? Math.floor((totalSleepMins / sleepCount) % 60) : 0;
    const avgEnergy = energyCount ? (totalEnergy / energyCount).toFixed(1) : "—";
    const pomoDurH = Math.floor((totalPomos * 25) / 60);
    const pomoDurM = (totalPomos * 25) % 60;

    dv.table(
        ["Metric", "Value"],
        [
            ["📅 Days logged", `${weekNotes.length} / 7`],
            ["😴 Avg Sleep", `${avgSleepH}h ${avgSleepM}m`],
            ["⚡ Avg Energy", `${avgEnergy} / 10`],
            ["🍅 Total Pomos", `${totalPomos} (${pomoDurH}h ${pomoDurM}m)`],
        ]
    );
} else {
    dv.paragraph("_No daily notes found for this week yet._");
}
```

---

## 🎯 Active Projects — BASE 3

> Core commitments from the weekly note.

| Priority | Project | Area |
|---|---|---|
| 🔴 S | [[09 - PROJECTS/0901 - Health/OPTIMISING SLEEP PROTOCOL/OPTIMISING SLEEP PROTOCOL 2026 - mainnote\|OPTIMISING SLEEP PROTOCOL]] | Health |
| 🟠 A | [[09 - PROJECTS/0901 - Health/OPTIMISING WORKOUT PROTOCOL/OPTIMISING WORKOUT PROTOCOL-basenote\|OPTIMISING WORKOUT PROTOCOL]] | Health |
| 🔴 S | [[05 - ACADEMICS/0501 - PGT/Uttarakhand Lecturer/Syllabus for Uttarakhand Lecturer\|ACE UTTARAKHAND LECTURER]] | Academics |
| 🟠 A | [[05 - ACADEMICS/0501 - PGT/PGT KVS/Syllabus for PGT KVS\|BEST SHOT AT KVS PGT]] | Academics |
| 🔴 S | [[09 - PROJECTS/0903 - Writing/BAL BHADRA CHRONICLES/BAL BHADRA CHRONICLES-basenote\|BAL BHADRA CHRONICLES]] | Writing |
| 🔴 S | [[09 - PROJECTS/0904 - Learn/LEARN TOUCHTYPE/LEARN TOUCHTYPE-basenote\|LEARN TOUCHTYPE]] | Learn |
| 🟠 A | [[09 - PROJECTS/0904 - Learn/LEARN ASL/LEARN ASL-basenote\|LEARN ASL]] | Learn |
| 🔴 S | [[09 - PROJECTS/0905 - Home/MOTHER LIT/MOTHER LIT -basenote\|TEACHING MY MAA]] | Home |
| 🔴 S | [[09 - PROJECTS/0907 - Miscellaneous/POETRY CHANNEL/POETRY CHANNEL-basenote\|POETRY CHANNEL]] | Misc |

---

## 📝 Recently Modified Notes

```dataviewjs
dv.table(
    ["Note", "Folder", "Modified"],
    dv.pages()
        .where(p => !p.file.path.startsWith("01 - BACKEND"))
        .where(p => !p.file.path.startsWith("."))
        .where(p => p.file.name !== "HOME PAGE")
        .sort(p => p.file.mtime, "desc")
        .limit(10)
        .map(p => [
            p.file.link,
            p.file.folder,
            dv.date(p.file.mtime).toFormat("MMM dd, HH:mm")
        ])
);
```

---

## 📚 Recent Journal Entries

```dataviewjs
const year = dv.date("today").toFormat("yyyy");
const dailyPath = `11 - PRIVATE/1102 - Journal/${year}/Daily Notes`;

dv.table(
    ["Date", "Day", "Energy", "Sleep", "Pomos"],
    dv.pages(`"${dailyPath}"`)
        .sort(p => p.file.name, "desc")
        .limit(7)
        .map(p => [
            p.file.link,
            p.weekday ?? "—",
            `${p.energy_lvl ?? "—"}/10`,
            p.sleep_duration ?? "—",
            p.pomos ?? 0
        ])
);
```

---

## 🎬 Media Corner

```dataviewjs
const ccPath = "11 - PRIVATE/1101 - CC";
const categories = ["Anime", "Books", "Movies", "Shows", "AsianDramas", "GraphicReads"];
const icons = {"Anime": "🎌", "Books": "📖", "Movies": "🎬", "Shows": "📺", "AsianDramas": "🎭", "GraphicReads": "📚"};

let rows = [];
for (let cat of categories) {
    const pages = dv.pages(`"${ccPath}/${cat}"`);
    const watching = pages.where(p => p.status === "watching" || p.status === "reading" || p.status === "ongoing");
    rows.push([
        `${icons[cat] ?? "📁"} [[${ccPath}/${cat}|${cat}]]`,
        pages.length,
        watching.length
    ]);
}
dv.table(["Category", "Total", "Active"], rows);
```
