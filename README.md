<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MirAI v1.0</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    <style>
        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #0f0f0f;
            margin: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            position: relative;
            transition: all 0.5s ease-in-out;
        }

        .logo-container {
            position: absolute;
            top: 25%;
            left: 50%;
            transform: translate(-50% , -50%);
            transition: visibility 0.5s ease-in-out;
            z-index: 999;
        }

        .logo-container.logo-hidden {
            opacity: 0;
            visibility: hidden;
        }

        .logo-container img {
            max-width: 450px;
            height: auto;
        }

        body.chat-active {
            justify-content: flex-start;
        }
        
        .chat-container {
            width: 100%;
            max-width: 600px;
            padding: 20px;
            box-sizing: border-box;
            display: flex;
            flex-direction: column;
            overflow-y: auto;
            flex-grow: 1;
            padding-bottom: 120px;
            opacity: 0;
            transition: opacity 0.5s ease-in-out;
        }
        
        body.chat-active .chat-container {
            opacity: 1;
        }

        #chatMessages {
            display: flex;
            flex-direction: column;
            gap: 20px;
        }
        
        .chat-bubble {
            max-width: 100%;
            padding: 8px 20px;
            border-radius: 25px;
            box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
            font-size: 16px;
            line-height: 1.5;
            word-wrap: break-word;
            margin-bottom: 10px;
        }

        .user-bubble {
            background-color: #ffce00;
            color: #000;
            align-self: flex-end;
            border-bottom-right-radius: 5px;
        }

        .ai-bubble {
            background-color: #212121;
            color: #fff;
            align-self: flex-start;
            border-bottom-left-radius: 5px;
        }

        .ai-bubble h1, .ai-bubble h2, .ai-bubble h3 {
            border-bottom: 1px solid #ffce00;
            padding-bottom: 5px;
            margin-top: 20px;
        }
        .ai-bubble ul, .ai-bubble ol {
            padding-left: 20px;
        }
        .ai-bubble li {
            margin-bottom: 5px;
        }
        .ai-bubble p {
            margin-bottom: 10px;
        }
        .ai-bubble strong {
            color: #ffce00;
        }

        .typing-indicator {
            align-self: flex-start;
            margin-left: 10px;
        }

        .typing-indicator span {
            display: inline-block;
            width: 8px;
            height: 8px;
            margin: 0 2px;
            background-color: #ffce00;
            border-radius: 50%;
            animation: bounce 0.6s infinite alternate;
        }
        
        .typing-indicator span:nth-child(2) {
            animation-delay: 0.2s;
        }
        
        .typing-indicator span:nth-child(3) {
            animation-delay: 0.4s;
        }
        
        @keyframes bounce {
            from {
                transform: translateY(0);
            }
            to {
                transform: translateY(-5px);
            }
        }

        .input-container {
            position: fixed;
            bottom: 0;
            left: 0;
            width: 100%;
            padding: 20px;
            box-sizing: border-box;
            background-color: #0f0f0f;
            z-index: 100;
            transition: all 0.5s ease-in-out;
        }

        .input-container.centered {
            position: absolute;
            bottom: 50%;
            transform: translateY(50%);
        }

        .input-container.bottom {
            position: fixed;
            bottom: 0;
        }

        .input-box {
            display: flex;
            align-items: center;
            max-width: 600px;
            margin: 0 auto;
            padding: 10px;
            background-color: transparent;
            border: 1px solid #aaa;
            border-radius: 25px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }

        .input-box textarea {
            flex-grow: 1;
            border: none;
            outline: none;
            color: white;
            font-size: 16px;
            padding: 0 10px;
            background-color: transparent;
            resize: none;
            overflow: hidden;
            max-height: 100px;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            border-radius: 25px;
        }
        
        .input-box button {
            background-color: #ffce00;
            color: black;
            border: none;
            border-radius: 25px;
            width: 40px;
            height: 40px;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }

        .input-box button:hover {
            background-color: #B8860B;
        }

        .suggestion-box {
            background-color: #212121;
            border: 1px solid #333;
            border-radius: 10px;
            padding: 10px;
            position: relative;
            top: -10px;
            z-index: 10;
            width: calc(100% - 20px);
            margin-top: 5px;
            box-sizing: border-box;
        }
        
        .suggestion-item {
            color: #ccc;
            padding: 8px 10px;
            cursor: pointer;
            border-bottom: 1px solid #333;
            font-size: 14px;
        }
        
        .suggestion-item:last-child {
            border-bottom: none;
        }

        .suggestion-item:hover {
            background-color: #333;
            color: white;
        }

        .loading-text {
            display: flex;
            align-items: center;
            gap: 10px;
        }
    </style>
</head>
<body>

<div class="logo-container">
    <img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhDYNXg_HBPgwViHUXJ_9F9UFT6BH3oFaAnE_JHnFI6dfoDhKziToum5i2W9YkHy4kC1xSdG6AqfYN7qdob7oLfZfg5rx1jLnfKraUBaFXK22O0DaPzk55Wu5y0EO-823b6MD__ulXGruUTZwARBZcrzXUvLfwniuzhWybM4qrstFUpzzOHcL2C83HclBVJ/s16000-rw/20250905_143027.png" alt="MirAI Logo">
</div>

<div class="chat-container">
    <div id="chatMessages"></div>
</div>

<div id="inputContainer" class="input-container centered">
    <div class="input-box">
        <textarea id="questionInput" placeholder="Tanya MirAI..." rows="1"></textarea>
        <button id="sendBtn">
            <i class="fas fa-paper-plane"></i>
        </button>
    </div>
    <div id="suggestionBox" class="suggestion-box" style="display:none;"></div>
</div>

<script>
    const API_KEY = 'AIzaSyBPN4dVYZfthVUXyQMyQGBwHFpv1UykFDQ';
    const model = 'gemini-2.5-flash';

    const faqDatabase = {
        "apa itu allwaysever?": "Allwaysever adalah ekosistem kreatif yang berfokus pada storytelling, konten sinematik low-budget, dan inovasi digital.",
        "apa itu awsoglweb?": "AwsOglWeb (Allwaysever Originals Web) adalah proyek web resmi dari Allwaysever Originals untuk menyajikan dokumentasi, FAQ, dan informasi proyek.",
        "apa itu aop?": "AOP (Allwaysever Originals Projects) adalah proyek orisinal seperti vlog, film pendek, dan dokumentasi kreatif.",
        "apa itu amp?": "AMP (Allwaysever Music Projects) adalah proyek musik dan video musik yang dibuat dengan kolaborasi AI dan sentuhan kreatif.",
        "apa itu awp?": "AWP (Allwaysever Web Projects) adalah proyek berbasis web, termasuk AwsOglWeb.",
        "apa itu aosp?": "AOSP (AllwayseverOpenSourceProjects) adalah inisiatif open-source dari Allwaysever untuk berbagi karya secara terbuka.",
        "apa itu jvt?": "Jogja Vlog Topic (JVT) adalah seri vlog perjalanan di Yogyakarta dengan pendekatan sinematik.",
        "apa itu jvt2?": "Jogja Vlog Topic Season 2 (JVT2) melanjutkan seri dengan visual dan narasi lebih matang.",
        "apa itu jvt3?": "Jogja Vlog Topic Season 3 (JVT3) adalah kelanjutan terbaru dari seri perjalanan di Jogja.",
        "apa itu jvt3tosolo?": "JVT3ToSolo adalah spin-off dari JVT3 yang berfokus pada perjalanan menuju Kota Solo.",
        "apa itu prts?": "PRTS (Parangtritis) adalah episode khusus dalam seri JVT yang berfokus pada destinasi Pantai Parangtritis.",
        "apa itu acl?": "ACL (Allwaysever Custom License) adalah sistem lisensi buatan Allwaysever untuk melindungi dan mengatur distribusi karya.",
        "kenapa pakai acl?": "ACL digunakan agar karya Allwaysever tetap memiliki kebebasan distribusi, namun tetap terlindungi dari penyalahgunaan.",
        "apa itu aephone?": "AEPhone adalah konsep smartphone fiksi dari Allwaysever, yang menggambarkan parodi dan eksplorasi dunia teknologi.",
        "apa itu aeos?": "AEos adalah sistem operasi fiksi untuk AEPhone, dengan tagline: 'System yang cukup masuk akal'.",
        "apa itu aechip?": "AEChip adalah prosesor fiksi yang digunakan AEPhone sebagai bagian dari konsep kreatif Allwaysever."
    };

    const questionInput = document.getElementById('questionInput');
    const sendBtn = document.getElementById('sendBtn');
    const chatMessages = document.getElementById('chatMessages');
    const suggestionBox = document.getElementById('suggestionBox');
    const body = document.body;
    const inputContainer = document.getElementById('inputContainer');
    const logoContainer = document.querySelector('.logo-container'); 
    
    let chatHistory = [];

    sendBtn.addEventListener('click', askAI);
    
    questionInput.addEventListener('keydown', (event) => {
        if (event.key === 'Enter' && !event.shiftKey) {
            event.preventDefault();
            askAI();
        }
    });

    questionInput.addEventListener('input', showSuggestions);

    questionInput.addEventListener('input', () => {
      questionInput.style.height = 'auto';
      questionInput.style.height = questionInput.scrollHeight + 'px';
    });

    function showSuggestions() {
        const query = questionInput.value.toLowerCase().trim();
        suggestionBox.innerHTML = '';
        suggestionBox.style.display = 'none';

        if (query.length < 2) {
            return;
        }

        const suggestions = Object.keys(faqDatabase)
            .filter(key => key.toLowerCase().includes(query))
            .slice(0, 5);

        if (suggestions.length > 0) {
            suggestionBox.style.display = 'block';
            suggestions.forEach(suggestion => {
                const div = document.createElement('div');
                div.classList.add('suggestion-item');
                div.innerText = suggestion;
                div.addEventListener('click', () => {
                    questionInput.value = suggestion;
                    suggestionBox.style.display = 'none';
                    askAI();
                });
                suggestionBox.appendChild(div);
            });
        }
    }
    
    function similarity(str1, str2) {
        str1 = str1.toLowerCase().trim();
        str2 = str2.toLowerCase().trim();
        const words1 = str1.split(" ");
        const words2 = str2.split(" ");
        const common = words1.filter(w => words2.includes(w));
        return common.length / Math.max(words1.length, words2.length);
    }

    function findSmartAnswer(question) {
        const normalized = question.toLowerCase().trim();
        
        let bestMatch = null;
        let highestScore = 0;
        const threshold = 0.6;

        const sortedKeys = Object.keys(faqDatabase).sort((a, b) => b.length - a.length);

        for (const key of sortedKeys) {
            const normalizedKey = key.toLowerCase().replace("?", "");
            if (normalized.includes(normalizedKey)) {
                return faqDatabase[key];
            }
        }

        for (const key in faqDatabase) {
            const score = similarity(normalized, key);
            if (score > highestScore) {
                highestScore = score;
                bestMatch = key;
            }
        }

        return highestScore >= threshold ? faqDatabase[bestMatch] : null;
    }
    
    function typeWriterEffect(element, text, speed = 1) {
        const parsedHtml = marked.parse(text);
        let i = 0;
        element.innerHTML = '';
        const tempElement = document.createElement('div');
        tempElement.innerHTML = parsedHtml;
        const textToType = tempElement.innerText;
        
        return new Promise(resolve => {
            function type() {
                if (i < textToType.length) {
                    element.innerHTML += textToType.charAt(i);
                    i++;
                    setTimeout(type, speed);
                } else {
                    element.innerHTML = parsedHtml;
                    resolve();
                }
            }
            type();
        });
    }

    function showLoading() {
        const loadingBubble = document.createElement('div');
        loadingBubble.classList.add('chat-bubble', 'ai-bubble', 'typing-indicator');
        loadingBubble.innerHTML = '<span></span><span></span><span></span>';
        chatMessages.appendChild(loadingBubble);
        return loadingBubble;
    }

    async function askAI() {
        if (!body.classList.contains('chat-active')) {
            body.classList.add('chat-active');
            inputContainer.classList.remove('centered');
            inputContainer.classList.add('bottom');
            logoContainer.classList.add('logo-hidden');
        }

        const question = questionInput.value;
        questionInput.value = '';
        questionInput.style.height = 'auto';
        suggestionBox.style.display = 'none';

        if (question.trim() === '') {
            return;
        }

        chatHistory.push({
            role: 'user',
            parts: [{ text: question }]
        });

        const userBubble = document.createElement('div');
        userBubble.classList.add('chat-bubble', 'user-bubble');
        userBubble.innerText = question;
        chatMessages.appendChild(userBubble);
        chatMessages.scrollTop = chatMessages.scrollHeight;

        const loadingBubble = showLoading();

        const localAnswer = findSmartAnswer(question);
        if (localAnswer) {
            await new Promise(resolve => setTimeout(resolve, 500));
            loadingBubble.remove();
            chatHistory.push({
                role: 'model',
                parts: [{ text: localAnswer }]
            });
            
            const aiBubble = document.createElement('div');
            aiBubble.classList.add('chat-bubble', 'ai-bubble');
            chatMessages.appendChild(aiBubble);
            await typeWriterEffect(aiBubble, localAnswer);
            return;
        }

        try {
            // Pastikan URL di sini adalah URL Web app yang baru Anda dapatkan
const url = `https://script.google.com/macros/s/AKfycbxOMgxciIdOLpd2vlKLHIaIpumhc3PpMnIJJhmhuYhaUoqrQxdzKto00B3rP7GcJHW1XQ/exec`;
const response = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ question })
});

            const data = await response.json();

            let answer = 'Maaf, saya tidak dapat menemukan jawaban yang tepat. Silakan coba pertanyaan lain.';
            if (data.answer) {
                answer = data.answer;
            }

            loadingBubble.remove();
            chatHistory.push({
                role: 'model',
                parts: [{ text: answer }]
            });

            const aiBubble = document.createElement('div');
            aiBubble.classList.add('chat-bubble', 'ai-bubble');
            chatMessages.appendChild(aiBubble);
            await typeWriterEffect(aiBubble, answer);

        } catch (error) {
            console.error('Terjadi kesalahan:', error);
            loadingBubble.remove();
            const aiBubble = document.createElement('div');
            aiBubble.classList.add('chat-bubble', 'ai-bubble');
            chatMessages.appendChild(aiBubble);
            const errorMessage = `Maaf, terjadi kesalahan saat mengirim pesan. Coba lakukan : \n1. Periksa Internet anda \n2. Coba muat ulang halaman ini \n3. Coba kirim ulang pesan yang kamu ingin kirim. Jangan lupa copy text nya \n4. Kalau kuota anda habis, mohon bersabar.\n\n Error: ${error.message}`;
            await typeWriterEffect(aiBubble, errorMessage);
        }
    }
</script>

</body>
</html>
