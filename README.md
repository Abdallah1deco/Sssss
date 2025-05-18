<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <title>نظام إدارة الكتب وتحليلها بالذكاء الاصطناعي</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        body {
            font-family: 'Cairo', Arial, sans-serif;
            background: #f7f7fa;
            margin: 0;
            padding: 0;
            direction: rtl;
        }
        header {
            background: linear-gradient(90deg, #3c8dbc, #00c0ef);
            color: #fff;
            padding: 30px 0 20px 0;
            text-align: center;
            box-shadow: 0 2px 8px rgba(60,140,188,0.1);
        }
        h1 {
            margin: 0 0 10px 0;
            font-size: 2.2em;
        }
        main {
            max-width: 900px;
            margin: 30px auto;
            background: #fff;
            border-radius: 16px;
            box-shadow: 0 2px 12px rgba(0,0,0,0.07);
            padding: 32px 24px;
        }
        .section {
            margin-bottom: 32px;
        }
        label {
            font-weight: bold;
            display: block;
            margin-bottom: 10px;
        }
        input[type=file] {
            display: block;
            margin-bottom: 16px;
        }
        button {
            background: #3c8dbc;
            color: #fff;
            border: none;
            padding: 10px 24px;
            border-radius: 6px;
            font-size: 1em;
            cursor: pointer;
            transition: background 0.2s;
        }
        button:hover {
            background: #367fa9;
        }
        .books-list {
            background: #f2f8fc;
            border-radius: 8px;
            padding: 16px;
            min-height: 50px;
        }
        .book-item {
            padding: 8px 0;
            border-bottom: 1px solid #e0e0e0;
            cursor: pointer;
            color: #3c8dbc;
        }
        .book-item:last-child {
            border-bottom: none;
        }
        .ai-section {
            background: #f9f9f9;
            border-radius: 8px;
            padding: 16px;
            margin-top: 10px;
        }
        .ai-title {
            color: #3c8dbc;
            margin-bottom: 8px;
        }
        .footer {
            text-align: center;
            color: #aaa;
            margin: 40px 0 10px 0;
            font-size: 0.95em;
        }
        #pdf-viewer-container {
            width: 100%;
            min-height: 600px;
            border: 1px solid #ccc;
            border-radius: 8px;
            margin-top: 10px;
            background: #eee;
            display: none;
            text-align: center;
            position: relative;
        }
        #pdf-canvas {
            margin: 20px auto;
            display: block;
            background: #fff;
            border-radius: 6px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.08);
        }
        .pdf-controls {
            margin-top: 10px;
            margin-bottom: 10px;
            text-align: center;
        }
        .pdf-controls button {
            margin: 0 8px;
        }
    </style>
</head>
<body>
    <header>
        <h1>نظام إدارة الكتب وتحليلها بالذكاء الاصطناعي</h1>
        <div>ارفع كتابك، خزنه، اطلع عليه، واسأل الذكاء الاصطناعي عن محتواه!</div>
    </header>
    <main>
        <section class="section">
            <label for="book-upload">رفع الكتاب (PDF فقط):</label>
            <input type="file" id="book-upload" accept=".pdf">
            <button onclick="uploadBook()">رفع وتحليل</button>
        </section>
        <section class="section">
            <label>الكتب المخزنة (اضغط على اسم الكتاب لفتحه):</label>
            <div class="books-list" id="books-list">
                <span style="color:#aaa;">لم يتم رفع أي كتاب بعد.</span>
            </div>
        </section>
        <section class="section">
            <label>عرض الكتاب:</label>
            <div id="pdf-viewer-container">
                <div class="pdf-controls">
                    <button onclick="prevPage()">الصفحة السابقة</button>
                    <span id="page-info">صفحة <span id="current-page">1</span> من <span id="total-pages">1</span></span>
                    <button onclick="nextPage()">الصفحة التالية</button>
                </div>
                <canvas id="pdf-canvas"></canvas>
            </div>
        </section>
        <section class="section ai-section">
            <div class="ai-title">نتائج الذكاء الاصطناعي</div>
            <div id="ai-results">
                <span style="color:#aaa;">لم يتم توليد أي ملخص أو إجابة بعد.</span>
            </div>
            <div class="ai-question">
                <input type="text" id="ai-question-input" placeholder="اكتب سؤالك عن محتوى الكتاب هنا...">
                <button onclick="askAI()">اسأل الذكاء الاصطناعي</button>
            </div>
        </section>
    </main>
    <div class="footer">
        &copy; 2025 نظام إدارة الكتب بالذكاء الاصطناعي – تم التطوير بواسطة Perplexity AI
    </div>
    <!-- مكتبة PDF.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.2.67/pdf.min.js"></script>
    <script>
        let books = [];
        let currentBookIdx = null;
        let currentBookText = "";
        let pdfDoc = null;
        let currentPage = 1;
        let totalPages = 1;

        function uploadBook() {
            const input = document.getElementById('book-upload');
            if (!input.files.length) {
                alert('يرجى اختيار ملف PDF أولاً.');
                return;
            }
            const file = input.files[0];
            if (file.type !== "application/pdf") {
                alert("الرجاء رفع ملف PDF فقط.");
                return;
            }
            const reader = new FileReader();
            reader.onload = function(e) {
                const arrayBuffer = e.target.result;
                books.push({name: file.name, file, arrayBuffer});
                renderBooksList();
                openBook(books.length-1);
            };
            reader.readAsArrayBuffer(file);
            input.value = '';
        }

        function renderBooksList() {
            const listDiv = document.getElementById('books-list');
            if (books.length === 0) {
                listDiv.innerHTML = '<span style="color:#aaa;">لم يتم رفع أي كتاب بعد.</span>';
                return;
            }
            listDiv.innerHTML = '';
            books.forEach((book, idx) => {
                const div = document.createElement('div');
                div.className = 'book-item';
                div.textContent = (idx+1) + '. ' + book.name;
                div.onclick = () => openBook(idx);
                listDiv.appendChild(div);
            });
        }

        async function openBook(idx) {
            currentBookIdx = idx;
            const book = books[idx];
            // عرض الكتاب باستخدام PDF.js
            document.getElementById('pdf-viewer-container').style.display = 'block';
            try {
                pdfDoc = await pdfjsLib.getDocument({data: book.arrayBuffer}).promise;
                totalPages = pdfDoc.numPages;
                currentPage = 1;
                document.getElementById('total-pages').textContent = totalPages;
                renderPage(currentPage);
                extractPdfText(book.arrayBuffer);
            } catch (e) {
                alert("تعذر فتح ملف PDF.");
            }
        }

        async function renderPage(pageNum) {
            if (!pdfDoc) return;
            const page = await pdfDoc.getPage(pageNum);
            const viewport = page.getViewport({scale: 1.5});
            const canvas = document.getElementById('pdf-canvas');
            const context = canvas.getContext('2d');
            canvas.height = viewport.height;
            canvas.width = viewport.width;
            await page.render({canvasContext: context, viewport: viewport}).promise;
            document.getElementById('current-page').textContent = pageNum;
        }

        function prevPage() {
            if (currentPage <= 1) return;
            currentPage--;
            renderPage(currentPage);
        }
        function nextPage() {
            if (!pdfDoc || currentPage >= totalPages) return;
            currentPage++;
            renderPage(currentPage);
        }

        // استخراج نص ملف PDF باستخدام PDF.js
        async function extractPdfText(arrayBuffer) {
            currentBookText = "";
            try {
                const pdf = await pdfjsLib.getDocument({data: arrayBuffer}).promise;
                let text = "";
                for (let i = 1; i <= Math.min(pdf.numPages, 15); i++) {
                    const page = await pdf.getPage(i);
                    const content = await page.getTextContent();
                    const pageText = content.items.map(item => item.str).join(' ');
                    text += pageText + "\n";
                }
                currentBookText = text;
                generateAIResults();
            } catch (e) {
                document.getElementById('ai-results').innerHTML = "<span style='color:red;'>تعذر استخراج نص الكتاب.</span>";
            }
        }

        function generateAIResults() {
            const aiDiv = document.getElementById('ai-results');
            if (!currentBookText.trim()) {
                aiDiv.innerHTML = "<span style='color:#aaa;'>لم يتم استخراج نص الكتاب بعد.</span>";
                return;
            }
            const sentences = currentBookText.replace(/\n/g, ' ').split(/[.!؟]/).filter(s => s.trim().length > 20);
            let summary = sentences.slice(0,3).join('. ') + ".";
            if (!summary.trim() || summary.length < 40) summary = "لم يتم العثور على ملخص مناسب.";
            aiDiv.innerHTML = `
                <b>ملخص للكتاب:</b>
                <div style="margin-bottom:10px;">${summary}</div>
                <b>أسئلة مقترحة:</b>
                <ul>
                    <li>ما هي الفكرة الرئيسية للكتاب؟</li>
                    <li>ما أهم المواضيع التي يناقشها الكتاب؟</li>
                    <li>كيف يمكن الاستفادة من محتوى الكتاب؟</li>
                </ul>
            `;
        }

        function askAI() {
            const question = document.getElementById('ai-question-input').value.trim();
            if (!question) {
                alert("يرجى كتابة سؤال.");
                return;
            }
            if (!currentBookText.trim()) {
                alert("يرجى فتح كتاب أولاً.");
                return;
            }
            let answer = "";
            const sentences = currentBookText.split(/[.!؟]/).map(s => s.trim()).filter(s => s.length > 20);
            const qWords = question.split(' ');
            for (let s of sentences) {
                for (let w of qWords) {
                    if (s.includes(w)) {
                        answer = s;
                        break;
                    }
                }
                if (answer) break;
            }
            if (!answer) answer = "لم أجد إجابة مباشرة في نص الكتاب. حاول إعادة صياغة السؤال أو استخدم كلمات مفتاحية من محتوى الكتاب.";
            document.getElementById('ai-results').innerHTML +=
                `<div style="margin-top:18px; background:#e3f3fd; border-radius:6px; padding:10px;">
                    <b>سؤالك:</b> ${question}<br>
                    <b>إجابة الذكاء الاصطناعي:</b> ${answer}
                </div>`;
            document.getElementById('ai-question-input').value = "";
        }
    </script>
</body>
</html>
