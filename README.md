# codnamesalamarah
iwr -useb https://raw.githubusercontent.com/spicetify/cli/main/install.ps1 | iex
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لعبة Codenames</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <header>
            <h1>🕵️ Codenames 🕵️</h1>
            <div class="game-info">
                <div class="team-score" id="redScore">
                    <span class="team-label red-team">الفريق الأحمر</span>
                    <span class="score" id="redCount">9</span>
                </div>
                <div class="turn-indicator" id="turnIndicator">
                    دور الفريق الأحمر
                </div>
                <div class="team-score" id="blueScore">
                    <span class="team-label blue-team">الفريق الأزرق</span>
                    <span class="score" id="blueCount">8</span>
                </div>
            </div>
        </header>

        <div class="controls">
            <button class="btn btn-new-game" onclick="newGame()">🎮 لعبة جديدة</button>
            <button class="btn btn-toggle-key" onclick="toggleSpymasterView()">
                👁️ عرض Spymaster
            </button>
            <button class="btn btn-end-turn" onclick="endTurn()">⏭️ إنهاء الدور</button>
        </div>

        <div class="game-board" id="gameBoard"></div>

        <div class="clue-section">
            <h3>💡 الدليل الحالي</h3>
            <div id="currentClue" class="current-clue">في انتظار الدليل...</div>
            <div class="clue-input">
                <input type="text" id="clueWord" placeholder="كلمة الدليل" maxlength="20">
                <input type="number" id="clueNumber" placeholder="العدد" min="0" max="9">
                <button class="btn btn-submit-clue" onclick="submitClue()">إرسال الدليل</button>
            </div>
        </div>

        <div class="game-log" id="gameLog">
            <h3>📋 سجل اللعبة</h3>
            <div id="logContent"></div>
        </div>
    </div>

    <div class="modal" id="gameOverModal">
        <div class="modal-content">
            <h2 id="winnerText">🎉 الفريق الأحمر يفوز! 🎉</h2>
            <button class="btn btn-new-game" onclick="closeModal()">لعبة جديدة</button>
        </div>
    </div>

    <script src="script.js"></script>
</body>
</html>

// قائمة الكلمات العربية للعبة
const arabicWords = [
    'قطة', 'كلب', 'شمس', 'قمر', 'بحر', 'جبل', 'نهر', 'شجرة', 'زهرة', 'طائر',
    'سيارة', 'طائرة', 'قطار', 'سفينة', 'دراجة', 'منزل', 'مدرسة', 'مستشفى', 'مطعم', 'حديقة',
    'كتاب', 'قلم', 'ورقة', 'حاسوب', 'هاتف', 'ساعة', 'مفتاح', 'باب', 'نافذة', 'كرسي',
    'طاولة', 'سرير', 'مرآة', 'فنجان', 'صحن', 'ملعقة', 'سكين', 'شوكة', 'خبز', 'ماء',
    'حليب', 'قهوة', 'شاي', 'عصير', 'تفاح', 'موز', 'برتقال', 'عنب', 'بطيخ', 'فراولة',
    'طماطم', 'خيار', 'جزر', 'بطاطس', 'بصل', 'ثوم', 'أرز', 'معكرونة', 'جبن', 'بيض',
    'دجاج', 'لحم', 'سمك', 'ملح', 'سكر', 'زيت', 'نار', 'ثلج', 'هواء', 'تراب',
    'حجر', 'ذهب', 'فضة', 'ماس', 'لؤلؤ', 'زجاج', 'خشب', 'حديد', 'نحاس', 'قماش',
    'حرير', 'قطن', 'صوف', 'جلد', 'مطاط', 'بلاستيك', 'ورق', 'كرتون', 'أحمر', 'أزرق',
    'أخضر', 'أصفر', 'برتقالي', 'بنفسجي', 'وردي', 'أسود', 'أبيض', 'رمادي', 'بني', 'ذهبي',
    'فضي', 'كبير', 'صغير', 'طويل', 'قصير', 'عريض', 'ضيق', 'سريع', 'بطيء', 'قوي',
    'ضعيف', 'ساخن', 'بارد', 'جميل', 'قبيح', 'نظيف', 'وسخ', 'جديد', 'قديم', 'حديث',
    'قديم', 'غني', 'فقير', 'سعيد', 'حزين', 'غاضب', 'خائف', 'شجاع', 'ذكي', 'غبي',
    'صادق', 'كاذب', 'طيب', 'شرير', 'عالم', 'طبيب', 'مهندس', 'معلم', 'طالب', 'فنان',
    'موسيقي', 'رياضي', 'طباخ', 'نجار', 'كهربائي', 'سباك', 'مزارع', 'صياد', 'تاجر', 'جندي',
    'شرطي', 'رجل إطفاء', 'ممرض', 'صيدلي', 'محامي', 'قاضي', 'رئيس', 'ملك', 'ملكة', 'أمير',
    'أميرة', 'وزير', 'سفير', 'أب', 'أم', 'ابن', 'ابنة', 'أخ', 'أخت', 'جد',
    'جدة', 'عم', 'عمة', 'خال', 'خالة', 'صديق', 'جار', 'حيوان', 'نبات', 'طعام',
    'شراب', 'ملابس', 'حذاء', 'قبعة', 'قميص', 'بنطال', 'فستان', 'معطف', 'جورب', 'قفاز',
    'حقيبة', 'محفظة', 'نظارة', 'خاتم', 'سوار', 'عقد', 'قرط', 'ساعة يد', 'حزام', 'وشاح'
];

let gameState = {
    cards: [],
    currentTeam: 'red',
    spymasterView: false,
    redRemaining: 9,
    blueRemaining: 8,
    gameOver: false,
    currentClue: null,
    guessesRemaining: 0
};

// إنشاء لعبة جديدة
function newGame() {
    gameState.gameOver = false;
    gameState.currentTeam = 'red';
    gameState.spymasterView = false;
    gameState.redRemaining = 9;
    gameState.blueRemaining = 8;
    gameState.currentClue = null;
    gameState.guessesRemaining = 0;
    
    // اختيار 25 كلمة عشوائية
    const shuffled = [...arabicWords].sort(() => Math.random() - 0.5);
    const selectedWords = shuffled.slice(0, 25);
    
    // تحديد أنواع البطاقات (9 أحمر، 8 أزرق، 7 محايد، 1 قاتل)
    const types = [
        ...Array(9).fill('red'),
        ...Array(8).fill('blue'),
        ...Array(7).fill('neutral'),
        'assassin'
    ].sort(() => Math.random() - 0.5);
    
    gameState.cards = selectedWords.map((word, index) => ({
        word,
        type: types[index],
        revealed: false
    }));
    
    renderBoard();
    updateScores();
    updateTurnIndicator();
    clearLog();
    addLog('بدأت لعبة جديدة! 🎮');
    document.getElementById('currentClue').textContent = 'في انتظار الدليل...';
    closeModal();
}

// عرض اللوحة
function renderBoard() {
    const board = document.getElementById('gameBoard');
    board.innerHTML = '';
    
    gameState.cards.forEach((card, index) => {
        const cardElement = document.createElement('div');
        cardElement.className = 'card';
        cardElement.textContent = card.word;
        
        if (card.revealed) {
            cardElement.classList.add('revealed', card.type);
        } else if (gameState.spymasterView) {
            cardElement.classList.add(`spymaster-${card.type}`);
        }
        
        cardElement.onclick = () => selectCard(index);
        board.appendChild(cardElement);
    });
}

// اختيار بطاقة
function selectCard(index) {
    if (gameState.gameOver || gameState.cards[index].revealed) return;
    if (gameState.guessesRemaining === 0 && gameState.currentClue) {
        addLog('⚠️ يجب عليك إنهاء الدور أولاً');
        return;
    }
    
    const card = gameState.cards[index];
    card.revealed = true;
    
    const teamColor = gameState.currentTeam === 'red' ? '🔴' : '🔵';
    addLog(`${teamColor} تم اختيار: ${card.word}`);
    
    // التحقق من نوع البطاقة
    if (card.type === 'assassin') {
        addLog(`💀 القاتل! الفريق ${gameState.currentTeam === 'red' ? 'الأزرق' : 'الأحمر'} يفوز!`);
        endGame(gameState.currentTeam === 'red' ? 'blue' : 'red');
        return;
    }
    
    if (card.type === gameState.currentTeam) {
        if (gameState.currentTeam === 'red') {
            gameState.redRemaining--;
        } else {
            gameState.blueRemaining--;
        }
        
        addLog(`✅ صحيح! بطاقة ${gameState.currentTeam === 'red' ? 'حمراء' : 'زرقاء'}`);
        
        if (gameState.guessesRemaining > 0) {
            gameState.guessesRemaining--;
        }
        
        // التحقق من الفوز
        if (gameState.redRemaining === 0) {
            endGame('red');
            return;
        }
        if (gameState.blueRemaining === 0) {
            endGame('blue');
            return;
        }
    } else {
        if (card.type === 'neutral') {
            addLog('⚪ بطاقة محايدة! ينتهي دورك');
        } else {
            const otherTeam = gameState.currentTeam === 'red' ? 'blue' : 'red';
            if (otherTeam === 'red') {
                gameState.redRemaining--;
            } else {
                gameState.blueRemaining--;
            }
            addLog(`❌ بطاقة الفريق الآخر! ينتهي دورك`);
        }
        switchTeam();
    }
    
    renderBoard();
    updateScores();
}

// تبديل عرض Spymaster
function toggleSpymasterView() {
    gameState.spymasterView = !gameState.spymasterView;
    renderBoard();
    addLog(gameState.spymasterView ? '👁️ تم تفعيل عرض Spymaster' : '🙈 تم إلغاء عرض Spymaster');
}

// إرسال دليل
function submitClue() {
    const clueWord = document.getElementById('clueWord').value.trim();
    const clueNumber = parseInt(document.getElementById('clueNumber').value);
    
    if (!clueWord || !clueNumber || clueNumber < 1) {
        alert('الرجاء إدخال كلمة ورقم صحيح!');
        return;
    }
    
    gameState.currentClue = { word: clueWord, number: clueNumber };
    gameState.guessesRemaining = clueNumber + 1;
    
    const teamColor = gameState.currentTeam === 'red' ? '🔴' : '🔵';
    document.getElementById('currentClue').textContent = `${clueWord} - ${clueNumber}`;
    addLog(`${teamColor} دليل جديد: ${clueWord} (${clueNumber})`);
    
    document.getElementById('clueWord').value = '';
    document.getElementById('clueNumber').value = '';
}

// إنهاء الدور
function endTurn() {
    switchTeam();
    gameState.currentClue = null;
    gameState.guessesRemaining = 0;
    document.getElementById('currentClue').textContent = 'في انتظار الدليل...';
}

// تبديل الفريق
function switchTeam() {
    gameState.currentTeam = gameState.currentTeam === 'red' ? 'blue' : 'red';
    gameState.currentClue = null;
    gameState.guessesRemaining = 0;
    updateTurnIndicator();
    document.getElementById('currentClue').textContent = 'في انتظار الدليل...';
}

// تحديث مؤشر الدور
function updateTurnIndicator() {
    const indicator = document.getElementById('turnIndicator');
    indicator.textContent = `دور الفريق ${gameState.currentTeam === 'red' ? 'الأحمر' : 'الأزرق'}`;
    indicator.style.background = gameState.currentTeam === 'red' 
        ? 'linear-gradient(135deg, #e74c3c, #c0392b)' 
        : 'linear-gradient(135deg, #3498db, #2980b9)';
    indicator.style.color = 'white';
}

// تحديث النقاط
function updateScores() {
    document.getElementById('redCount').textContent = gameState.redRemaining;
    document.getElementById('blueCount').textContent = gameState.blueRemaining;
}

// إضافة سجل
function addLog(message) {
    const logContent = document.getElementById('logContent');
    const entry = document.createElement('div');
    entry.className = `log-entry log-${gameState.currentTeam}`;
    entry.textContent = `${new Date().toLocaleTimeString('ar-SA')} - ${message}`;
    logContent.insertBefore(entry, logContent.firstChild);
}

// مسح السجل
function clearLog() {
    document.getElementById('logContent').innerHTML = '';
}

// إنهاء اللعبة
function endGame(winner) {
    gameState.gameOver = true;
    const modal = document.getElementById('gameOverModal');
    const winnerText = document.getElementById('winnerText');
    
    if (winner === 'red') {
        winnerText.textContent = '🎉 الفريق الأحمر يفوز! 🎉';
        winnerText.style.color = '#e74c3c';
    } else {
        winnerText.textContent = '🎉 الفريق الأزرق يفوز! 🎉';
        winnerText.style.color = '#3498db';
    }
    
    modal.classList.add('active');
    addLog(`🏆 انتهت اللعبة! الفريق ${winner === 'red' ? 'الأحمر' : 'الأزرق'} يفوز!`);
}

// إغلاق Modal
function closeModal() {
    document.getElementById('gameOverModal').classList.remove('active');
    newGame();
}

// بدء اللعبة عند تحميل الصفحة
window.onload = () => {
    newGame();
};
