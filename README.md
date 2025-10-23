<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>智慧之泉</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background: #e2e8f0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        /* 確保卦象符號字體夠大且不會被擠壓 */
        #hexagram-symbol {
            font-family: 'Times New Roman', serif; /* 使用襯線字體確保卦象符號顯示正常 */
        }
        /* 定義一個 class 用於圖片下載，將背景設為白色，確保輸出圖片不會是透明的 */
        .image-export-area {
            background-color: white; 
        }
    </style>
</head>
<body class="bg-gray-100 p-4 sm:p-8">
    <div class="max-w-xl w-full bg-white rounded-3xl shadow-2xl p-6 sm:p-10 text-center">
        <h1 class="text-3xl sm:text-4xl font-extrabold text-gray-800 mb-2">智慧之泉</h1>
        <p class="text-gray-500 mb-6 sm:mb-8">輸入日期、時間、事由與心中默想的文字</p>

        <div id="messageBox" class="hidden mb-4 p-4 rounded-lg text-sm transition-opacity duration-300"></div>

        <div class="space-y-4 text-left">
            <div>
                <label for="date-input" class="block text-gray-700 font-semibold mb-2">日期</label>
                <input type="date" id="date-input" class="w-full p-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors" required>
            </div>
            <div>
                <label for="time-input" class="block text-gray-700 font-semibold mb-2">時間</label>
                <input type="time" id="time-input" class="w-full p-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors" required>
            </div>
            <div>
                <label for="reason-text-input" class="block text-gray-700 font-semibold mb-2">事由 (文字)</label>
                <textarea id="reason-text-input" rows="2" class="w-full p-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors" placeholder="輸入事由，例如：今日工作進度" required></textarea>
            </div>
            <div>
                <label for="thought-char-input" class="block text-gray-700 font-semibold mb-2">心中默想的一個字</label>
                <input type="text" id="thought-char-input" class="w-full p-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors" maxlength="1" placeholder="心中默想後輸入一個字" required>
            </div>
            <div>
                <label for="manual-strokes-input" class="block text-gray-700 font-semibold mb-2">手動輸入筆畫總數 (選填)</label>
                <input type="number" id="manual-strokes-input" class="w-full p-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-purple-500 focus:border-purple-500 transition-colors" placeholder="如需手動覆蓋計算結果，請在此輸入" min="1">
            </div>
            <button id="calculate-btn" class="w-full mt-6 bg-blue-600 text-white font-bold py-3 rounded-xl shadow-lg hover:bg-blue-700 transition-all transform hover:scale-105">
                開始計算
            </button>
        </div>

        <div id="results-container" class="mt-8 hidden">
             <button id="download-btn" class="w-full mb-4 bg-green-600 text-white font-bold py-2 rounded-xl shadow hover:bg-green-700 transition-colors">
                下載卦象結果為圖片 (PNG)
            </button>

            <div id="results" class="image-export-area p-4 rounded-xl">
                <h2 class="text-2xl font-bold text-gray-800 mb-4">卦象結果</h2>
                <div class="flex flex-col sm:flex-row justify-center items-center gap-6">
                    <div class="bg-blue-50 p-6 rounded-2xl shadow-inner w-full sm:w-1/2">
                        <h3 class="text-lg font-semibold text-gray-700 mb-2">上卦</h3>
                        <p id="upper-trigram-symbol" class="text-5xl mb-2"></p>
                        <p id="upper-trigram-name" class="text-2xl font-bold text-blue-600"></p>
                        <p id="upper-trigram-info" class="text-sm text-gray-500 mt-2"></p>
                    </div>
                    <div class="bg-blue-50 p-6 rounded-2xl shadow-inner w-full sm:w-1/2">
                        <h3 class="text-lg font-semibold text-gray-700 mb-2">下卦</h3>
                        <p id="lower-trigram-symbol" class="text-5xl mb-2"></p>
                        <p id="lower-trigram-name" class="text-2xl font-bold text-blue-600"></p>
                        <p id="lower-trigram-info" class="text-sm text-gray-500 mt-2"></p>
                    </div>
                </div>
                <div class="mt-6 bg-gray-50 p-6 rounded-2xl shadow-inner w-full">
                    <h3 class="text-xl font-semibold text-gray-700 mb-2">總卦象 (上卦+下卦)</h3>
                    <p id="hexagram-symbol" class="text-6xl font-sans mb-2"></p>
                    <p id="hexagram-name" class="text-2xl font-bold text-gray-800"></p>
                    <div class="mt-4 pt-4 border-t border-gray-200">
                        <p id="hexagram-full-name" class="text-xl font-semibold text-purple-600 mb-2"></p>
                        
                        <p class="text-sm font-bold text-gray-700">卦辭 (經典)：</p>
                        <p id="hexagram-meaning" class="text-base text-gray-600 italic mb-3"></p>
                        
                        <p class="text-sm font-bold text-gray-700">白話解釋：</p>
                        <p id="hexagram-vernacular" class="text-base text-gray-600"></p>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 設定康熙字典筆畫數 (精簡版)
        const KANGXI_STROKES = {
            '一': 1, '二': 2, '三': 3, '四': 5, '五': 4, '六': 4, '七': 2, '八': 2, '九': 2, '十': 2,
            '人': 2, '子': 3, '女': 3, '大': 3, '小': 3, '山': 3, '川': 3, '工': 3, '土': 3,
            '日': 4, '月': 4, '水': 4, '火': 4, '木': 4, '金': 8, '土': 3, '石': 5, '田': 5,
            '事': 8, '由': 5, '智': 12, '慧': 15, '泉': 9, '網': 14, '頁': 9, '文': 4, '字': 6,
            '筆': 12, '畫': 12, '數': 15, '餘': 15, '乾': 14, '兌': 7, '離': 19, '震': 15, '巽': 13,
            '坎': 11, '艮': 6, '坤': 8, '年': 6, '月': 4, '日': 4, '時': 10, '刻': 10,
            '心': 4, '中': 4, '默': 15, '想': 13, '的': 11, '到': 8, '個': 11,
            '工': 3, '作': 7, '進': 15, '度': 9, '的': 11
        };

        // 八卦數字對應 (1=乾, 2=兌, 3=離, 4=震, 5=巽, 6=坎, 7=艮, 0=坤)
        const TRIGRAMS = {
            1: { name: '乾', symbol: '☰' },
            2: { name: '兌', symbol: '☱' },
            3: { name: '離', symbol: '☲' },
            4: { name: '震', symbol: '☳' },
            5: { name: '巽', symbol: '☴' },
            6: { name: '坎', symbol: '☵' },
            7: { name: '艮', symbol: '☶' },
            0: { name: '坤', symbol: '☷' }
        };

        // 六十四卦卦名及卦辭 (取自周易) - 鍵值為「上卦名+下卦名」
        const HEXAGRAMS = {
            // 純卦 (八純卦)
            '乾乾': { name: '乾為天', meaning: '元亨，利貞。', vernacular: '大吉大利，亨通順暢，只要堅守正道，就會有好結果。' },
            '坤坤': { name: '坤為地', meaning: '元亨，利牝馬之貞。君子有攸往，先迷後得主，利。西南得朋，東北喪朋。安貞，吉。', vernacular: '廣大亨通，利於像母馬一樣柔順且持之以恆。君子外出先會迷路，但最終會找到主人，有利。守住安定和正道是吉利的。' },
            '震震': { name: '震為雷', meaning: '亨。震來虩虩，笑言啞啞。震驚百里，不喪匕鬯。', vernacular: '亨通。雖然會遇到突如其來的驚嚇（像打雷），但內心保持鎮定，最後仍能談笑自若，不會因此失態或失去重要之物。' },
            '艮艮': { name: '艮為山', meaning: '艮其止，止其所也。時止則止，時行則行，動靜不失其時，其道光明。', vernacular: '停止不動，要能安於其位。該停止時就停止，該行動時就行動，動靜之間不失時機，其道光明。' },
            '離離': { name: '離為火', meaning: '利貞，亨。畜牝牛，吉。', vernacular: '利於堅守正道，亨通。像飼養母牛一樣柔順，則為吉利。' },
            '坎坎': { name: '坎為水', meaning: '習坎，有孚，維心亨，行有尚。', vernacular: '重重險難，但只要內心誠實堅定，就能亨通。行動會有價值和尊貴。' },
            '兌兌': { name: '兌為澤', meaning: '亨，利貞。', vernacular: '亨通，利於堅守正道。代表喜悅、和樂。' },
            '巽巽': { name: '巽為風', meaning: '小亨，利有攸往，利見大人。', vernacular: '小有亨通，利於前往目標。有利於會見有德行的大人物。' },

            // 雜卦
            '乾坤': { name: '天地否', meaning: '否之匪人，不利君子貞。大往小來。', vernacular: '閉塞不通的時機，不利於君子堅守正道。壞的勢力增長，好的勢力衰退。' },
            '坤乾': { name: '地天泰', meaning: '小往大來，吉亨。', vernacular: '亨通吉利。小的力量離去，大的力量到來，象徵著國泰民安、平順和諧。' },
            '震坎': { name: '屯卦 (水雷屯)', meaning: '元亨，利貞。勿用有攸往，利建侯。', vernacular: '開始亨通，利於堅守正道。此時不宜貿然行動，最好先安定下來，有利於建立根基。' },
            '坎離': { name: '既濟 (水火既濟)', meaning: '亨，小利貞。初吉終亂。', vernacular: '亨通，小的方面有利於堅守正道。事情已經成功，但要警惕，因為開始吉利，最終可能會混亂。' },
            '離坎': { name: '未濟 (火水未濟)', meaning: '亨，小狐汔濟，濡其尾，無攸利。', vernacular: '亨通，但像小狐狸快要渡河卻把尾巴弄濕一樣，沒有實質的利益。意指事情尚未完全成功，需要耐心和謹慎。' },
            '坎乾': { name: '需卦 (水天需)', meaning: '有孚，光亨，貞吉。利涉大川。', vernacular: '有誠信，光明亨通，堅守正道吉利。利於渡過大江大河，意指可以進行大的行動，但需要等待時機。' },
            '乾坎': { name: '訟卦 (天水訟)', meaning: '有孚，窒惕，中吉，終凶。利見大人，不利涉大川。', vernacular: '有誠信，但會遇到阻礙，需提高警惕。中間吉利，但最終可能凶險。有利於見到貴人，不利於冒險前進。' },
            '震離': { name: '噬嗑 (火雷噬嗑)', meaning: '亨。利用獄。', vernacular: '亨通。有利於使用刑罰，意指解決問題需要果斷和力量。' },
            '離震': { name: '豐卦 (雷火豐)', meaning: '亨。王假之。勿憂，宜日中。', vernacular: '亨通。君王應當親臨祭祀。不用憂慮，就像太陽正當中午，光明普照。' },
            '震兌': { name: '歸妹 (雷澤歸妹)', meaning: '征凶，無攸利。', vernacular: '行動會帶來凶險，沒有任何益處。比喻不合時宜、不合常理的結合或行動。' },
            '兌震': { name: '隨卦 (澤雷隨)', meaning: '元亨，利貞，無咎。', vernacular: '大為亨通，利於堅守正道，沒有災禍。意指順隨時勢、隨從正道。' },
            '艮坎': { name: '蒙卦 (山水蒙)', meaning: '蒙卦 (山水蒙)', meaning: '亨。匪我求童蒙，童蒙求我。初噬告，再三瀆，瀆則不告。利貞。', vernacular: '亨通。不是我去求教於無知者，而是無知者來求教於我。第一次詢問會告訴他，但再三詢問就是輕慢，輕慢就不會再告訴了。利於堅守正道。' },
            '坎艮': { name: '蹇卦 (水山蹇)', meaning: '利西南，不利東北。利見大人，貞吉。', vernacular: '險難在前，利於往西南方（柔順退守），不利於往東北方（剛強前進）。有利於會見貴人，堅守正道是吉利的。' },
            '乾兌': { name: '履卦 (天澤履)', meaning: '履虎尾，不咥人，亨。', vernacular: '像踩在老虎尾巴上，但老虎沒有咬人，亨通。比喻行為應當小心謹慎，雖然危險但能化解。' },
            '兌乾': { name: '夬卦 (澤天夬)', meaning: '揚於王庭，孚號有厲，告自邑，不利即戎，利有攸往。', vernacular: '在朝廷上宣揚，雖然有誠信，但會有危險。應當在自己的城邑裡警告，不利於用武力解決，利於有所前往。' },
            
            '乾離': { name: '同人 (天火同人)', meaning: '同人於野，亨。利涉大川，利君子貞。', vernacular: '與人志同道合，在廣闊的原野上結交朋友，亨通。利於渡過大江大河，利於君子堅守正道。' },
            '乾震': { name: '無妄 (天雷無妄)', meaning: '元亨，利貞。其匪正有眚，不利有攸往。', vernacular: '大為亨通，利於堅守正道。如果不是出於正當，就會有災禍，不利於有所前往。' },
            '乾巽': { name: '姤卦 (天風姤)', meaning: '女壯，勿用取女。', vernacular: '女子強勢，不適合娶這位女子。比喻陰盛陽衰，不宜採取強硬措施。' },
            '乾艮': { name: '遁卦 (天山遁)', meaning: '亨。小利貞。', vernacular: '亨通。小事有利於堅守正道。意指退隱、迴避的時機。' },
            
            '坤兌': { name: '臨卦 (地澤臨)', meaning: '元亨，利貞。至于八月有凶。', vernacular: '大為亨通，利於堅守正道。但到了八月會出現凶險，提醒事物發展有高峰期和衰落期。' },
            '坤離': { name: '明夷 (地火明夷)', meaning: '利艱貞。', vernacular: '利於在艱難中堅守正道。意指光明受到傷害或遮蔽，需要韜光養晦。' },
            '坤震': { name: '豫卦 (雷地豫)', meaning: '利建侯行師。', vernacular: '利於建立基業、發兵征戰。意指安樂、歡愉，但需防範樂極生悲。' },
            '坤巽': { name: '觀卦 (風地觀)', meaning: '盥而不薦，有孚顒若。', vernacular: '舉行祭祀時，只洗手而不獻祭，但能以誠信和莊重的態度讓人仰望。意指以德服人，需觀察和體會。' },
            '坤艮': { name: '剝卦 (山地剝)', meaning: '不利有攸往。', vernacular: '不利於有所前往。意指剝落、衰敗，陽氣漸消，應當休止。' },
            
            '兌離': { name: '革卦 (澤火革)', meaning: '巳日乃孚，元亨，利貞，悔亡。', vernacular: '到了巳日（象徵時機成熟）就能取信於人，大為亨通，利於堅守正道，後悔消除。意指變革、除舊布新。' },
            '兌巽': { name: '中孚 (風澤中孚)', meaning: '豚魚吉，利涉大川，利貞。', vernacular: '連小豬、小魚這樣不通靈性的動物都能感到誠信，是吉利的。利於渡過大江大河，利於堅守正道。' },
            '兌艮': { name: '損卦 (山澤損)', meaning: '元亨，利貞，無咎。可貞，不利有攸往。', vernacular: '大為亨通，利於堅守正道，沒有災禍。可以堅守正道，但不利於有所前往（行動）。' },

            '離乾': { name: '大有 (火天大有)', meaning: '元亨。', vernacular: '大為亨通。意指擁有豐大的財富和德行。' },
            '離兌': { name: '睽卦 (火澤睽)', meaning: '小事吉。', vernacular: '小事吉利。意指互相背離、不和，大處不合，小處可為。' },
            '離巽': { name: '鼎卦 (火風鼎)', meaning: '元吉，亨。', vernacular: '大為吉利，亨通。意指確立新的、穩固的地位或制度。' },
            '離艮': { name: '賁卦 (山火賁)', meaning: '亨，小利有攸往。', vernacular: '亨通，小事有利於有所前往。意指修飾、美化，但不能過度依賴外表。' },

            '震乾': { name: '大壯 (雷天大壯)', meaning: '利貞。', vernacular: '利於堅守正道。意指氣勢強大，但要防止過度剛猛。' },
            '震巽': { name: '益卦 (風雷益)', meaning: '利有攸往，利涉大川。', vernacular: '利於有所前往，利於渡過大江大河。意指增益、獲利。' },
            '震艮': { name: '小過 (雷山小過)', meaning: '亨，利貞。可小事，不可大事。飛鳥遺之音，不宜上宜下，大吉。', vernacular: '亨通，利於堅守正道。可以做小事，不可做大事。像飛鳥留下聲音，不宜向上發展，宜於向下保守，大吉。' },

            '巽乾': { name: '小畜 (風天小畜)', meaning: '亨。密雲不雨，自我西郊。', vernacular: '亨通。天空有密布的雲層，但還沒有下雨，雲氣從西方而來。意指小的積聚和阻礙，能量尚未釋放。' },
            '巽坎': { name: '渙卦 (風水渙)', meaning: '亨。王假有廟，利涉大川，利貞。', vernacular: '亨通。君王前往宗廟祭祀，利於渡過大江大河，利於堅守正道。意指渙散、離散，需要聚合人心。' },
            '巽艮': { name: '漸卦 (風山漸)', meaning: '女歸吉，利貞。', vernacular: '女子出嫁吉利，利於堅守正道。意指循序漸進、逐漸發展。' },

            '坎兌': { name: '困卦 (澤水困)', meaning: '亨，貞。大人吉，無咎。有言不信。', vernacular: '亨通，堅守正道。大人吉利，沒有災禍。但言語不能取信於人。意指受困、處於困境。' },
            
            '艮乾': { name: '大畜 (山天大畜)', meaning: '利貞，不家食吉。利涉大川。', vernacular: '利於堅守正道，不在家裡吃閒飯是吉利的。利於渡過大江大河。意指大的積蓄和約束。' },
            
            '艮震': { name: '頤卦 (山雷頤)', meaning: '貞吉。觀頤，自求口實。', vernacular: '堅守正道吉利。觀察自己的飲食和養生之道，自求食物。意指頤養、自給自足。' }, 
            '艮巽': { name: '漸卦 (風山漸)', meaning: '女歸吉，利貞。', vernacular: '女子出嫁吉利，利於堅守正道。意指循序漸進、逐漸發展。' },

            '坤坎': { name: '比卦 (水地比)', meaning: '吉。原筮，元永貞，無咎。不寧方來，後夫凶。', vernacular: '吉利。最初占筮結果就大吉、永遠堅守正道，沒有災禍。不安寧的國家會前來歸附，後來的君子（或遲到的追隨者）則有凶險。意指親近、輔助。' },
        };
        // 時辰數字對應
        const TIME_NUMBERS = {
            '23-01': 1, '01-03': 2, '03-05': 3, '05-07': 4, '07-09': 5, '09-11': 6, '11-13': 7,
            '13-15': 8, '15-17': 9, '17-19': 10, '19-21': 11, '21-23': 12
        };

        // 元素獲取
        const dateInput = document.getElementById('date-input');
        const timeInput = document.getElementById('time-input');
        const reasonTextInput = document.getElementById('reason-text-input');
        const thoughtCharInput = document.getElementById('thought-char-input');
        const manualStrokesInput = document.getElementById('manual-strokes-input');
        const calculateBtn = document.getElementById('calculate-btn');
        const resultsContainer = document.getElementById('results-container'); // 新增外層容器
        const resultsDiv = document.getElementById('results'); // 實際的卦象結果 DIV
        const messageBox = document.getElementById('messageBox');
        const downloadBtn = document.getElementById('download-btn'); // 下載按鈕

        // 結果顯示元素
        const upperTrigramSymbol = document.getElementById('upper-trigram-symbol');
        const upperTrigramName = document.getElementById('upper-trigram-name');
        const upperTrigramInfo = document.getElementById('upper-trigram-info');
        const lowerTrigramSymbol = document.getElementById('lower-trigram-symbol');
        const lowerTrigramName = document.getElementById('lower-trigram-name');
        const lowerTrigramInfo = document.getElementById('lower-trigram-info');
        const hexagramSymbol = document.getElementById('hexagram-symbol');
        const hexagramName = document.getElementById('hexagram-name');
        const hexagramFullName = document.getElementById('hexagram-full-name');
        const hexagramMeaning = document.getElementById('hexagram-meaning');
        const hexagramVernacular = document.getElementById('hexagram-vernacular'); // 新增白話解釋元素

        // 計算文字總筆畫數
        function calculateStrokes(text) {
            let totalStrokes = 0;
            let unknownChars = [];
            for (const char of text) {
                const strokes = KANGXI_STROKES[char];
                if (strokes !== undefined) {
                    totalStrokes += strokes;
                } else {
                    unknownChars.push(char);
                }
            }
            return { totalStrokes, unknownChars };
        }

        // 顯示訊息
        function showMessage(text, isError = false) {
            messageBox.textContent = text;
            messageBox.classList.remove('hidden');
            if (isError) {
                messageBox.classList.remove('bg-blue-100', 'text-blue-700');
                messageBox.classList.add('bg-red-100', 'text-red-700');
            } else {
                messageBox.classList.remove('bg-red-100', 'text-red-700');
                messageBox.classList.add('bg-blue-100', 'text-blue-700');
            }
            setTimeout(() => {
                messageBox.classList.add('hidden');
            }, 5000);
        }

        // === 下載圖片功能 ===
        function downloadResultImage() {
            // 要轉換成圖片的 DOM 元素
            const node = resultsDiv; 
            
            showMessage('正在生成圖片，請稍候...');

            // 使用 html2canvas 將 DOM 元素轉換為 Canvas
            html2canvas(node, {
                // 設置背景色為白色，避免輸出圖片時背景為透明
                backgroundColor: '#ffffff',
                // 增加 scale 提升圖片清晰度 (例如 2 倍)
                scale: 2, 
            }).then(canvas => {
                // 將 Canvas 轉換為圖片的 Data URL
                const image = canvas.toDataURL('image/png');

                // 創建一個虛擬的 <a> 標籤來觸發下載
                const a = document.createElement('a');
                
                // 根據卦名設定檔名，並替換卦象符號
                const dateStr = dateInput.value.replace(/-/g, '');
                const hexName = hexagramFullName.textContent.replace(/\s/g, '');
                const fileName = `智慧之泉_${dateStr}_${hexName}.png`;

                a.href = image;
                a.download = fileName; // 設定下載檔案名稱
                document.body.appendChild(a);
                a.click(); // 模擬點擊下載

                document.body.removeChild(a);
                showMessage('圖片下載完成！');
            }).catch(error => {
                console.error('圖片下載失敗:', error);
                showMessage('圖片下載失敗。請檢查瀏覽器支援或再試一次。', true);
            });
        }
        // =======================


        // 計算上卦和下卦
        function calculateHexagram() {
            const date = dateInput.value;
            const time = timeInput.value;
            const reasonText = reasonTextInput.value;
            const thoughtChar = thoughtCharInput.value;

            if (!date || !time || !reasonText || !thoughtChar) {
                showMessage('請完整輸入日期、時間、事由與心中默想的文字', true);
                return;
            }

            // Step 1: 計算上卦
            const manualStrokes = parseInt(manualStrokesInput.value, 10);
            let totalStrokes;
            let calculationSource = '計算自文字';

            if (!isNaN(manualStrokes) && manualStrokes > 0) {
                // 如果手動輸入有效數字，則使用手動輸入的筆畫數
                totalStrokes = manualStrokes;
                calculationSource = '使用手動輸入覆蓋';
                showMessage('已使用您手動輸入的筆畫總數進行計算。', false); 
                
            } else {
                // 否則，使用文字計算筆畫數
                const { totalStrokes: reasonStrokes, unknownChars: reasonUnknown } = calculateStrokes(reasonText);
                const { totalStrokes: thoughtStrokes, unknownChars: thoughtUnknown } = calculateStrokes(thoughtChar);
                
                totalStrokes = reasonStrokes + thoughtStrokes;
                
                const unknownChars = [...reasonUnknown, ...thoughtUnknown];
                // 處理未知的字
                if (unknownChars.length > 0) {
                    const unknownStr = unknownChars.join('、');
                    showMessage(`注意：下列文字的筆畫數未在字典中找到，已計為0：${unknownStr}。您可以於「手動輸入筆畫總數」欄位調整結果。`, true);
                }
            }

            if (totalStrokes <= 0) {
                showMessage('筆畫總數必須大於 0。請檢查輸入文字或手動輸入筆畫總數。', true);
                return;
            }

            const upperRemainder = totalStrokes % 8;
            const upperTrigram = TRIGRAMS[upperRemainder];
            
            // Step 2: 計算下卦
            const [hours, minutes] = time.split(':').map(Number);
            let timeNumber = 0;
            // 判斷時辰數字
            if (hours >= 23 || hours < 1) timeNumber = TIME_NUMBERS['23-01'];
            else if (hours >= 1 && hours < 3) timeNumber = TIME_NUMBERS['01-03'];
            else if (hours >= 3 && hours < 5) timeNumber = TIME_NUMBERS['03-05'];
            else if (hours >= 5 && hours < 7) timeNumber = TIME_NUMBERS['05-07'];
            else if (hours >= 7 && hours < 9) timeNumber = TIME_NUMBERS['07-09'];
            else if (hours >= 9 && hours < 11) timeNumber = TIME_NUMBERS['09-11'];
            else if (hours >= 11 && hours < 13) timeNumber = TIME_NUMBERS['11-13'];
            else if (hours >= 13 && hours < 15) timeNumber = TIME_NUMBERS['13-15'];
            else if (hours >= 15 && hours < 17) timeNumber = TIME_NUMBERS['15-17'];
            else if (hours >= 17 && hours < 19) timeNumber = TIME_NUMBERS['17-19'];
            else if (hours >= 19 && hours < 21) timeNumber = TIME_NUMBERS['19-21'];
            else if (hours >= 21 && hours < 23) timeNumber = TIME_NUMBERS['21-23'];

            // 根據您的規則，從上卦數字往後算時間數字
            const lowerRemainder = (upperRemainder + timeNumber) % 8;
            const lowerTrigram = TRIGRAMS[lowerRemainder];

            // 取得六十四卦信息，新增 vernacular 預設值
            const hexagramKey = upperTrigram.name + lowerTrigram.name;
            const hexagramData = HEXAGRAMS[hexagramKey] || { 
                name: '未定義卦象', 
                meaning: '此卦象未找到對應的經典卦辭。',
                vernacular: '由於資料庫限制，此卦象暫無白話解釋。' 
            };


            // 顯示結果
            upperTrigramSymbol.textContent = upperTrigram.symbol;
            upperTrigramName.textContent = upperTrigram.name;
            upperTrigramInfo.textContent = `筆畫總數：${totalStrokes} (${calculationSource}) | 餘數：${upperRemainder}`;

            lowerTrigramSymbol.textContent = lowerTrigram.symbol;
            lowerTrigramName.textContent = lowerTrigram.name;
            lowerTrigramInfo.textContent = `時辰數字：${timeNumber} | 餘數：${lowerRemainder}`;
            
            // 顯示總卦象
            hexagramSymbol.textContent = upperTrigram.symbol + lowerTrigram.symbol;
            hexagramName.textContent = `${upperTrigram.name}${lowerTrigram.name}卦`;
            hexagramFullName.textContent = hexagramData.name;
            hexagramMeaning.textContent = hexagramData.meaning;
            hexagramVernacular.textContent = hexagramData.vernacular; // 顯示白話解釋

            resultsContainer.classList.remove('hidden');
        }

        // 按鈕事件監聽
        calculateBtn.addEventListener('click', calculateHexagram);
        downloadBtn.addEventListener('click', downloadResultImage);
    </script>
</body>
</html># echun.githun.io
