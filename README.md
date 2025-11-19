<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>填字 & 單字搜尋 產生器</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        /* --- Crossword Styles (優化後) --- */
        .crossword-grid { 
            display: grid; 
            border: none; /* 移除外框 */
            gap: 1px; /* 增加格子間的細微間距 */
        }
        .grid-cell { 
            width: 32px; 
            height: 32px; 
            position: relative; 
            background-color: transparent; /* 預設為透明，隱藏未使用的格子 */
            font-weight: 500; 
            text-transform: uppercase; 
            user-select: none;
            border: none; /* 移除預設邊框 */
        }
        .grid-cell.active { 
            background-color: white; 
            border: 1px solid #9ca3af; /* 只為有效格子加上邊框 */
            box-shadow: 0 1px 2px rgba(0,0,0,0.05); /* 增加細微陰影 */
        }
        .grid-cell .char { 
            position: absolute; 
            top: 50%; 
            left: 50%; 
            transform: translate(-50%, -50%); 
            color: #111827; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            width: 100%; 
            height: 100%;
            line-height: 32px;
        }
        .grid-cell .num { 
            position: absolute; 
            top: 1px; 
            left: 2px; 
            font-size: 10px; 
            font-weight: 600; 
            color: #4b5563; 
        }
        .cell-input { 
            width: 100%; 
            height: 100%; 
            text-align: center; 
            background-color: transparent; 
            border: none; 
            outline: none; 
            color: #1f2937; 
            font-size: 16px; 
            font-weight: bold; 
            text-transform: uppercase; 
            caret-color: #3b82f6; 
        }

        /* --- Word Search Styles --- */
        .wordsearch-grid { display: grid; border: 2px solid #374151; cursor: pointer; }
        .ws-cell { 
            width: 32px; 
            height: 32px; 
            border: 1px solid #d1d5db; 
            background-color: white; 
            display: flex; 
            justify-content: center; 
            align-items: center; 
            font-weight: 500; 
            text-transform: uppercase; 
            user-select: none; 
            position: relative; 
            line-height: 32px;
        }
        .ws-cell.selecting { background-color: #bfdbfe; /* blue-200 */ }
        .ws-cell.found { background-color: #a7f3d0; /* green-200 */ color: #065f46; /* green-800 */ }
        .wordsearch-list-item.found { text-decoration: line-through; color: #6b7280; }
    </style>
</head>
<body class="bg-gray-100 text-gray-800 flex flex-col items-center justify-center min-h-screen p-4">

    <div class="w-full max-w-6xl mx-auto bg-white rounded-lg shadow-xl p-6">
        <header class="text-center mb-6">
            <h1 class="text-3xl font-bold text-gray-800">謎題產生器</h1>
            <p class="text-gray-600 mt-1">從您的單字列表建立填字遊戲或單字搜尋遊戲。</p>
        </header>

        <main class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <!-- Left Panel: Input -->
            <div class="lg:col-span-1 bg-gray-50 p-4 rounded-lg border">
                <h2 class="text-xl font-semibold mb-3">1. 輸入單字</h2>
                <textarea id="vocab-input" class="w-full h-48 p-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500" placeholder="輸入單字，一行一個...&#10;格式：&#10;單字: 解釋"></textarea>
                
                <h2 class="text-xl font-semibold mb-2 mt-4">2. 選擇遊戲</h2>
                <div class="flex border-b border-gray-200">
                    <button id="tab-crossword" class="flex-1 py-2 px-4 text-sm font-medium text-center text-blue-600 bg-white border-b-2 border-blue-600 rounded-t-lg focus:outline-none">填字遊戲</button>
                    <button id="tab-wordsearch" class="flex-1 py-2 px-4 text-sm font-medium text-center text-gray-500 hover:text-gray-700 focus:outline-none">單字搜尋</button>
                </div>
                
                <h2 class="text-xl font-semibold mb-2 mt-4">3. 設定格子大小 (選填)</h2>
                <input type="number" id="grid-size-input" class="w-full p-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500" placeholder="留空則自動計算">


                <button id="generate-btn" class="mt-6 w-full bg-blue-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 transition-colors">
                    產生遊戲
                </button>
                
                <!-- Download Buttons -->
                <button id="download-puzzle-btn" class="mt-4 w-full bg-teal-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-teal-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-teal-500 transition-colors hidden">
                    下載題目 (PNG)
                </button>
                <button id="download-answer-btn" class="mt-2 w-full bg-purple-600 text-white font-bold py-2 px-4 rounded-lg hover:bg-purple-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-purple-500 transition-colors hidden">
                    下載解答 (PNG)
                </button>
            </div>

            <!-- Right Panel: Game Output -->
            <div class="lg:col-span-2">
                <div id="status-message" class="text-center p-4 mb-4 rounded-lg hidden"></div>
                
                <!-- Crossword Container -->
                <div id="crossword-container">
                    <div id="puzzle-wrapper" class="flex flex-col xl:flex-row gap-6">
                        <div id="puzzle-container" class="overflow-auto w-full"><div id="puzzle-grid" class="crossword-grid mx-auto"></div></div>
                        <div id="clues-container" class="w-full xl:w-2/3">
                            <div id="clues-across-wrapper" class="hidden"><h3 class="text-lg font-semibold mb-2 border-b pb-1">橫向提示</h3><ul id="clues-across" class="space-y-1 text-sm"></ul></div>
                            <div id="clues-down-wrapper" class="mt-4 hidden"><h3 class="text-lg font-semibold mb-2 border-b pb-1">縱向提示</h3><ul id="clues-down" class="space-y-1 text-sm"></ul></div>
                        </div>
                    </div>
                </div>

                <!-- Word Search Container -->
                <div id="wordsearch-container" class="hidden">
                     <div class="flex flex-col xl:flex-row gap-6">
                        <div id="wordsearch-grid-container" class="overflow-auto"><div id="wordsearch-grid" class="wordsearch-grid mx-auto"></div></div>
                        <div id="wordsearch-list-container" class="w-full xl:w-1/3">
                            <h3 class="text-lg font-semibold mb-2 border-b pb-1">找出這些單字:</h3>
                            <ul id="wordsearch-list" class="space-y-1 text-sm"></ul>
                        </div>
                    </div>
                </div>

            </div>
        </main>

        <!-- Answer Key Section -->
        <div id="answer-key-container" class="hidden mt-8 p-6 bg-gray-50 rounded-lg border">
            <h2 class="text-2xl font-bold text-center mb-4">解答</h2>
            <div id="answer-grid-wrapper" class="flex justify-center items-center overflow-auto">
                 <div id="answer-grid"></div>
            </div>
        </div>

    </div>

<script>
// --- UTILITY: A simple shuffle function ---
const shuffle = (array) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
};

// --- CROSSWORD GENERATOR CLASS (OPTIMIZED) ---
class CrosswordGenerator {
    constructor(size, vocabulary) {
        this.size = size;
        this.vocabulary = this.cleanAndSortVocab(vocabulary);
        this.grid = Array.from({ length: size }, () => Array(size).fill(null));
        this.placedWords = [];
        this.clueMap = {}; // Maps {row, col} to a clue number details
    }
    cleanAndSortVocab(vocab) {
        return vocab.map(line => {
            const parts = line.split(/:(.+)/); // Split on the first colon only
            if (parts.length < 2) return null;
            const word = parts[0].trim().toUpperCase().replace(/[^A-Z]/g, '');
            const definition = parts[1].trim();
            if (!word || !definition) return null;
            return { word, definition };
        }).filter(Boolean).sort((a,b) => b.word.length - a.word.length);
    }
    
    // --- OPTIMIZED PUZZLE GENERATION LOGIC ---
    generatePuzzle() {
        if (this.vocabulary.length < 1) return { unplaced: [] };

        let wordsToPlace = [...this.vocabulary];
        
        const firstWordObj = wordsToPlace.shift();
        const startCol = Math.floor((this.size - firstWordObj.word.length) / 2);
        const startRow = Math.floor(this.size / 2);
        this.placeWord(firstWordObj, startRow, startCol, 'across');

        let placedThisCycle = true;
        let attempts = 0;
        while (wordsToPlace.length > 0 && placedThisCycle && attempts < 20) {
            placedThisCycle = false;
            for (let i = wordsToPlace.length - 1; i >= 0; i--) {
                const wordObj = wordsToPlace[i];
                const bestPosition = this.findBestPosition(wordObj);
                
                if (bestPosition) {
                    this.placeWord(bestPosition.wordObj, bestPosition.row, bestPosition.col, bestPosition.direction);
                    wordsToPlace.splice(i, 1);
                    placedThisCycle = true;
                }
            }
            attempts++;
        }

        this.assignClueNumbers();
        const unplacedWords = wordsToPlace.map(w => w.word);
        return { unplaced: unplacedWords };
    }

    findBestPosition(wordObj) {
        let bestPosition = null;
        let maxScore = -1;
        const word = wordObj.word;

        for (let i = 0; i < word.length; i++) {
            const letter = word[i];
            for (const placed of this.placedWords) {
                for (let j = 0; j < placed.word.length; j++) {
                    if (placed.word[j] !== letter) continue;
                    
                    let row, col, direction;
                    if (placed.direction === 'across') {
                        row = placed.row - i;
                        col = placed.col + j;
                        direction = 'down';
                    } else {
                        row = placed.row + j;
                        col = placed.col - i;
                        direction = 'across';
                    }

                    if (placed.direction === direction || !this.isValidPlacement(word, row, col, direction)) {
                        continue;
                    }
                    
                    const score = this.calculateScore(word, row, col, direction);
                    if (score > maxScore) {
                        maxScore = score;
                        bestPosition = { wordObj, row, col, direction };
                    }
                }
            }
        }
        return bestPosition;
    }

    isValidPlacement(word, row, col, direction) {
        if (row < 0 || col < 0) return false;
        if (direction === 'across') {
            if (col + word.length > this.size) return false;
            if ((col > 0 && this.grid[row][col - 1] !== null) || (col + word.length < this.size && this.grid[row][col + word.length] !== null)) return false;
            for (let i = 0; i < word.length; i++) {
                const cell = this.grid[row][col + i];
                if (cell !== null && cell !== word[i]) return false;
                if (cell === null && ((row > 0 && this.grid[row - 1][col + i] !== null) || (row < this.size - 1 && this.grid[row + 1][col + i] !== null))) return false;
            }
        } else { // 'down'
            if (row + word.length > this.size) return false;
            if ((row > 0 && this.grid[row - 1][col] !== null) || (row + word.length < this.size && this.grid[row + word.length][col] !== null)) return false;
            for (let i = 0; i < word.length; i++) {
                const cell = this.grid[row + i][col];
                if (cell !== null && cell !== word[i]) return false;
                if (cell === null && ((col > 0 && this.grid[row + i][col - 1] !== null) || (col < this.size - 1 && this.grid[row + i][col + 1] !== null))) return false;
            }
        }
        return true;
    }

    calculateScore(word, row, col, direction) { 
        let score = 0; 
        for(let i = 0; i < word.length; i++) {
            if ((direction === 'across' && this.grid[row]?.[col + i] === word[i]) || (direction === 'down' && this.grid[row + i]?.[col] === word[i])) {
                score++;
            }
        } 
        return score;
    }

    placeWord(wordObj, row, col, direction) {
        const word = wordObj.word;
        const startKey = `${row},${col}`;
        if (!this.clueMap[startKey]) this.clueMap[startKey] = { num: 0, across: false, down: false };
        if (direction === 'across') { this.clueMap[startKey].across = true; for (let i = 0; i < word.length; i++) this.grid[row][col+i] = word[i]; }
        else { this.clueMap[startKey].down = true; for (let i = 0; i < word.length; i++) this.grid[row+i][col] = word[i]; }
        this.placedWords.push({ ...wordObj, row, col, direction });
    }
    assignClueNumbers() {
        const sortedStarts=Object.keys(this.clueMap).sort((a,b)=>{const[r1,c1]=a.split(',').map(Number);const[r2,c2]=b.split(',').map(Number);return r1!==r2?r1-r2:c1-c2;});
        sortedStarts.forEach((key,index)=>{this.clueMap[key].num=index+1;});
        this.placedWords.forEach(w=>{w.clueNum=this.clueMap[`${w.row},${w.col}`].num;});
    }
    getGrid() { return this.grid; }
    getClues() { return { across: this.placedWords.filter(p=>p.direction==='across').sort((a,b)=>a.clueNum-b.clueNum), down: this.placedWords.filter(p=>p.direction==='down').sort((a,b)=>a.clueNum-b.clueNum) }; }
}

// --- WORD SEARCH GENERATOR CLASS ---
class WordSearchGenerator {
    constructor(size, vocabulary) {
        this.size = size;
        this.vocabulary = this.cleanAndSortVocab(vocabulary);
        this.grid = Array.from({ length: size }, () => Array(size).fill(''));
        this.placedWords = [];
        this.directions = [[-1,0], [-1,1], [0,1], [1,1], [1,0], [1,-1], [0,-1], [-1,-1]];
    }
    cleanAndSortVocab(vocab) {
        return vocab.map(line => {
            const parts = line.split(/:(.+)/);
            if (parts.length < 2) return null;
            const word = parts[0].trim().toUpperCase().replace(/[^A-Z]/g, '');
            const definition = parts[1].trim();
            if (!word || !definition) return null;
            return { word, definition };
        }).filter(Boolean).sort((a, b) => b.word.length - a.word.length);
    }
    generate() {
        const unplaced = this.vocabulary.filter(wordObj => !this.tryPlaceWord(wordObj)).map(w=>w.word);
        this.fillEmptyCells();
        return { unplaced, placed: this.placedWords };
    }
    tryPlaceWord(wordObj) {
        const word = wordObj.word;
        const locations = [];
        for (let r = 0; r < this.size; r++) for (let c = 0; c < this.size; c++) locations.push({ r, c });
        shuffle(locations);
        for (const { r, c } of locations) {
            for (const [dr, dc] of shuffle([...this.directions])) {
                if (this.checkFit(word, r, c, dr, dc)) { this.placeWord(wordObj, r, c, dr, dc); return true; }
            }
        }
        return false;
    }
    checkFit(word, r, c, dr, dc) {
        for (let i = 0; i < word.length; i++) {
            const nr = r + i * dr, nc = c + i * dc;
            if (nr < 0 || nr >= this.size || nc < 0 || nc >= this.size) return false;
            const cell = this.grid[nr][nc];
            if (cell !== '' && cell !== word[i]) return false;
        }
        return true;
    }
    placeWord(wordObj, r, c, dr, dc) {
        const word = wordObj.word;
        for (let i = 0; i < word.length; i++) this.grid[r + i * dr][c + i * dc] = word[i];
        this.placedWords.push({ ...wordObj, r, c, dr, dc });
    }
    fillEmptyCells() {
        const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
        for (let r = 0; r < this.size; r++) for (let c = 0; c < this.size; c++) if (this.grid[r][c] === '') this.grid[r][c] = alphabet[Math.floor(Math.random() * alphabet.length)];
    }
    getGrid() { return this.grid; }
}


document.addEventListener('DOMContentLoaded', () => {
    // --- STATE MANAGEMENT ---
    let activeGame = 'crossword';
    let currentGenerator = null;

    // --- DOM ELEMENT REFERENCES ---
    const elements = {
        vocabInput: document.getElementById('vocab-input'),
        gridSizeInput: document.getElementById('grid-size-input'),
        generateBtn: document.getElementById('generate-btn'),
        statusMessage: document.getElementById('status-message'),
        tabCrossword: document.getElementById('tab-crossword'),
        tabWordSearch: document.getElementById('tab-wordsearch'),
        crosswordContainer: document.getElementById('crossword-container'),
        puzzleGrid: document.getElementById('puzzle-grid'),
        cluesAcrossList: document.getElementById('clues-across'),
        cluesDownList: document.getElementById('clues-down'),
        cluesAcrossWrapper: document.getElementById('clues-across-wrapper'),
        cluesDownWrapper: document.getElementById('clues-down-wrapper'),
        wordsearchContainer: document.getElementById('wordsearch-container'),
        wordsearchGrid: document.getElementById('wordsearch-grid'),
        wordsearchList: document.getElementById('wordsearch-list'),
        answerKeyContainer: document.getElementById('answer-key-container'),
        answerGrid: document.getElementById('answer-grid'),
        downloadPuzzleBtn: document.getElementById('download-puzzle-btn'),
        downloadAnswerBtn: document.getElementById('download-answer-btn'),
    };
    
    // --- INITIAL SETUP ---
    elements.vocabInput.value = "PUZZLE: A game, toy, or problem designed to test ingenuity or knowledge.\nALGORITHM: A process or set of rules to be followed in calculations.\nGENERATOR: Something that produces or creates something.";

    // --- EVENT LISTENERS ---
    elements.tabCrossword.addEventListener('click', () => switchGame('crossword'));
    elements.tabWordSearch.addEventListener('click', () => switchGame('wordsearch'));
    elements.generateBtn.addEventListener('click', handleGenerate);
    elements.downloadPuzzleBtn.addEventListener('click', handleDownloadPuzzle);
    elements.downloadAnswerBtn.addEventListener('click', handleDownloadAnswer);


    // --- GAME SWITCHING LOGIC ---
    function switchGame(game) {
        activeGame = game;
        elements.crosswordContainer.classList.toggle('hidden', game !== 'crossword');
        elements.wordsearchContainer.classList.toggle('hidden', game !== 'wordsearch');
        
        elements.tabCrossword.classList.toggle('text-blue-600', game === 'crossword');
        elements.tabCrossword.classList.toggle('border-blue-600', game === 'crossword');
        elements.tabCrossword.classList.toggle('bg-white', game === 'crossword');
        elements.tabCrossword.classList.toggle('text-gray-500', game !== 'crossword');
        elements.tabWordSearch.classList.toggle('text-blue-600', game === 'wordsearch');
        elements.tabWordSearch.classList.toggle('border-blue-600', game === 'wordsearch');
        elements.tabWordSearch.classList.toggle('bg-white', game === 'wordsearch');
        elements.tabWordSearch.classList.toggle('text-gray-500', game !== 'wordsearch');
        
        elements.answerKeyContainer.classList.add('hidden');
        elements.downloadPuzzleBtn.classList.add('hidden');
        elements.downloadAnswerBtn.classList.add('hidden');
    }

    // --- MAIN GENERATION HANDLER ---
    function handleGenerate() {
        elements.answerKeyContainer.classList.add('hidden');
        const vocabulary = elements.vocabInput.value.split('\n').filter(w => w.trim() !== '');
        
        if (vocabulary.length < 2) { 
            showStatus("請至少輸入兩個單字。", 'error'); 
            return; 
        }

        const gridSizeInput = elements.gridSizeInput.value;
        let gridSize = parseInt(gridSizeInput, 10);

        if (isNaN(gridSize) || gridSize <= 0) { // If input is empty or invalid, calculate automatically
            let totalCharacters = 0;
            let longestWordLength = 0;
            vocabulary.forEach(line => {
                const parts = line.split(/:(.+)/);
                if (parts.length > 0) {
                    const word = parts[0].trim().toUpperCase().replace(/[^A-Z]/g, '');
                    if (word.length > 0) {
                        totalCharacters += word.length;
                        if (word.length > longestWordLength) {
                            longestWordLength = word.length;
                        }
                    }
                }
            });
            
            gridSize = Math.ceil(Math.sqrt(totalCharacters * 2.8));
            gridSize = Math.max(gridSize, longestWordLength + 4);
            gridSize = Math.min(Math.max(gridSize, 15), 50);
        } else { // Use user-provided size, but with validation
             if (gridSize < 10 || gridSize > 100) {
                 showStatus("格子大小請介於 10 到 100 之間。", 'error');
                 return;
            }
        }


        if (activeGame === 'crossword') {
            currentGenerator = new CrosswordGenerator(gridSize, vocabulary);
            const { unplaced } = currentGenerator.generatePuzzle();
            renderCrossword(currentGenerator.getGrid(), currentGenerator.getClues());
            renderCrosswordAnswer(currentGenerator);
            updateStatus(unplaced, '填字遊戲');
        } else { // Word Search
            currentGenerator = new WordSearchGenerator(gridSize, vocabulary);
            const { unplaced, placed } = currentGenerator.generate();
            renderWordSearch(currentGenerator.getGrid(), placed);
            renderWordSearchAnswer(currentGenerator);
            updateStatus(unplaced, '單字搜尋遊戲');
        }
        elements.answerKeyContainer.classList.remove('hidden');
        elements.downloadPuzzleBtn.classList.remove('hidden');
        elements.downloadAnswerBtn.classList.remove('hidden');
    }
    
    // --- RENDER FUNCTIONS ---
    function renderCrossword(grid, clues) {
        const { puzzleGrid, cluesAcrossList, cluesDownList, cluesAcrossWrapper, cluesDownWrapper } = elements;
        puzzleGrid.innerHTML = '';
        puzzleGrid.style.gridTemplateColumns = `repeat(${grid.length}, 32px)`;
        const clueNumbers={}; clues.across.forEach(c=>clueNumbers[`${c.row},${c.col}`]=c.clueNum); clues.down.forEach(c=>clueNumbers[`${c.row},${c.col}`]=clueNumbers[`${c.row},${c.col}`]||c.clueNum);
        grid.forEach((row,r)=>{row.forEach((cell,c)=>{
            const cellDiv=document.createElement('div'); cellDiv.className='grid-cell';
            if(cell!==null){
                cellDiv.classList.add('active');
                if(clueNumbers[`${r},${c}`]){const n=document.createElement('span'); n.className='num'; n.textContent=clueNumbers[`${r},${c}`]; cellDiv.appendChild(n);}
                const inp=document.createElement('input'); inp.type='text'; inp.maxLength=1; inp.className='cell-input'; cellDiv.appendChild(inp);
            }
            puzzleGrid.appendChild(cellDiv);
        })});
        const renderClues=(list,ul,wrap)=>{ul.innerHTML='';list.forEach(c=>{const li=document.createElement('li');li.innerHTML=`<span class="font-semibold">${c.clueNum}.</span> ${c.definition}`;ul.appendChild(li);});wrap.classList.toggle('hidden',list.length===0);};
        renderClues(clues.across,cluesAcrossList,cluesAcrossWrapper); renderClues(clues.down,cluesDownList,cluesDownWrapper);
    }

    function renderWordSearch(grid, placedWordObjs) {
        const { wordsearchGrid, wordsearchList } = elements;
        wordsearchGrid.innerHTML = ''; wordsearchGrid.style.gridTemplateColumns = `repeat(${grid.length}, 32px)`;
        grid.forEach((row,r)=>{row.forEach((cell,c)=>{const d=document.createElement('div');d.className='ws-cell';d.textContent=cell;d.dataset.row=r;d.dataset.col=c;wordsearchGrid.appendChild(d);});});
        wordsearchList.innerHTML='';placedWordObjs.sort((a,b) => a.word.localeCompare(b.word)).forEach(wObj=>{const l=document.createElement('li');l.innerHTML=`<span class="font-semibold">${wObj.word}:</span> ${wObj.definition}`;l.id=`ws-word-${wObj.word}`;l.className="wordsearch-list-item";wordsearchList.appendChild(l);});
        let isSelecting=false,startCell=null,selectedCells=[];
        wordsearchGrid.addEventListener('mousedown',e=>{if(!e.target.classList.contains('ws-cell'))return;isSelecting=true;startCell=e.target;selectedCells=[startCell];startCell.classList.add('selecting');});
        wordsearchGrid.addEventListener('mousemove',e=>{if(!isSelecting||!e.target.classList.contains('ws-cell'))return;const endCell=e.target,startRow=parseInt(startCell.dataset.row),startCol=parseInt(startCell.dataset.col),endRow=parseInt(endCell.dataset.row),endCol=parseInt(endCell.dataset.col);document.querySelectorAll('.ws-cell.selecting').forEach(c=>c.classList.remove('selecting'));selectedCells=[];const dr=Math.sign(endRow-startRow),dc=Math.sign(endCol-startCol);let currRow=startRow,currCol=startCol;while(true){const cell=wordsearchGrid.querySelector(`[data-row='${currRow}'][data-col='${currCol}']`);if(cell){selectedCells.push(cell);cell.classList.add('selecting');}if(currRow===endRow&&currCol===endCol)break;if(Math.abs(endRow-startRow)>0)currRow+=dr;if(Math.abs(endCol-startCol)>0)currCol+=dc;}});
        document.addEventListener('mouseup',()=>{if(!isSelecting)return;isSelecting=false;const selectedWord=selectedCells.map(c=>c.textContent).join(''),reversedWord=selectedWord.split('').reverse().join('');const wordListItem=document.getElementById(`ws-word-${selectedWord}`)||document.getElementById(`ws-word-${reversedWord}`);if(wordListItem&&!wordListItem.classList.contains('found')){const word=wordListItem.id.replace('ws-word-','');wordListItem.classList.add('found');selectedCells.forEach(cell=>{cell.classList.remove('selecting');cell.classList.add('found');});}else{selectedCells.forEach(cell=>cell.classList.remove('selecting'));}startCell=null;selectedCells=[];});
    }

    function renderCrosswordAnswer(generator) {
        const { answerGrid } = elements;
        const grid = generator.getGrid();
        const clues = generator.getClues();
        const clueNumbers = {};
        clues.across.forEach(c => clueNumbers[`${c.row},${c.col}`] = c.clueNum);
        clues.down.forEach(c => clueNumbers[`${c.row},${c.col}`] = clueNumbers[`${c.row},${c.col}`] || c.clueNum);

        answerGrid.innerHTML = '';
        answerGrid.className = 'crossword-grid';
        answerGrid.style.gridTemplateColumns = `repeat(${grid.length}, 32px)`;
        grid.forEach((row, r) => { row.forEach((cell, c) => {
            const cellDiv = document.createElement('div');
            cellDiv.className = 'grid-cell';
            if (cell !== null) {
                cellDiv.classList.add('active');
                 if(clueNumbers[`${r},${c}`]){
                    const n = document.createElement('span'); 
                    n.className='num'; 
                    n.textContent=clueNumbers[`${r},${c}`]; 
                    cellDiv.appendChild(n);
                }
                const charSpan = document.createElement('span');
                charSpan.className = 'char';
                charSpan.textContent = cell;
                cellDiv.appendChild(charSpan);
            }
            answerGrid.appendChild(cellDiv);
        }); });
    }

    function renderWordSearchAnswer(generator) {
        const { answerGrid } = elements;
        const grid = generator.getGrid();
        const placedWords = generator.placedWords;
        answerGrid.innerHTML = '';
        answerGrid.className = 'wordsearch-grid';
        answerGrid.style.gridTemplateColumns = `repeat(${grid.length}, 32px)`;

        const solutionCoords = new Set();
        placedWords.forEach(({ word, r, c, dr, dc }) => {
            for (let i = 0; i < word.length; i++) {
                solutionCoords.add(`${r + i * dr},${c + i * dc}`);
            }
        });
        grid.forEach((row, r) => { row.forEach((cell, c) => {
            const cellDiv = document.createElement('div');
            cellDiv.className = 'ws-cell';
            cellDiv.textContent = cell;
            if (solutionCoords.has(`${r},${c}`)) {
                cellDiv.classList.add('found'); // Highlight solution cells
            }
            answerGrid.appendChild(cellDiv);
        }); });
    }


    // --- HELPER & DOWNLOAD FUNCTIONS ---
    function handleDownloadPuzzle() {
        const isTransparent = activeGame === 'crossword';
        const element = isTransparent ? elements.puzzleGrid : elements.wordsearchGrid;
        downloadGridAsImage(element, `${activeGame}_puzzle.png`, isTransparent);
    }

    function handleDownloadAnswer() {
        const isTransparent = activeGame === 'crossword';
        downloadGridAsImage(elements.answerGrid, `${activeGame}_answer.png`, isTransparent);
    }

    // --- UPDATED: Robust download function ---
    function downloadGridAsImage(element, filename, isTransparent = false) {
        const parentContainer = element.parentElement;
        const originalParentStyles = {
            overflow: parentContainer.style.overflow,
            width: parentContainer.style.width,
            height: parentContainer.style.height,
        };

        parentContainer.style.overflow = 'visible';
        parentContainer.style.width = 'auto';
        parentContainer.style.height = 'auto';

        const options = {
            scale: 3,
            backgroundColor: isTransparent ? null : '#ffffff',
            useCORS: true,
        };

        if (!isTransparent) {
            element.style.padding = '16px';
            element.style.backgroundColor = '#ffffff';
        }
        
        // --- FIX: Add short delay ---
        // This gives the browser time to re-render the layout
        // with overflow: 'visible' before html2canvas tries to capture it.
        setTimeout(() => {
            html2canvas(element, options).then(canvas => {
                const link = document.createElement('a');
                link.download = filename;
                link.href = canvas.toDataURL('image/png');
                link.click();
            }).catch(err => {
                console.error("圖片轉換失敗:", err);
                showStatus('圖片下載失敗，請再試一次。', 'error');
            }).finally(() => {
                // Always restore original styles
                if (!isTransparent) {
                    element.style.padding = '';
                    element.style.backgroundColor = '';
                }
                parentContainer.style.overflow = originalParentStyles.overflow;
                parentContainer.style.width = originalParentStyles.width;
                parentContainer.style.height = originalParentStyles.height;
            });
        }, 100); // 100ms delay
    }

    function updateStatus(unplaced, gameName) {
        if (unplaced.length > 0) { showStatus(`無法放置: ${unplaced.join(', ')}`, 'warning'); } 
        else { showStatus(`${gameName} 產生成功！`, 'success'); }
    }

    function showStatus(message, type = 'info') {
        const { statusMessage } = elements;
        statusMessage.textContent = message;
        const typeClasses = type==='error'?'bg-red-100 border border-red-300 text-red-800':type==='success'?'bg-green-100 border border-green-300 text-green-800':'bg-yellow-100 border border-yellow-300 text-yellow-800';
        statusMessage.className = `text-center p-4 mb-4 rounded-lg ${typeClasses}`;
        setTimeout(() => statusMessage.classList.add('hidden'), 5000);
    }
});
</script>
</body>
</html>
