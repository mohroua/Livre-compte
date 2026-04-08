<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>حسابات الإخوة السحابية</title>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&display=swap" rel="stylesheet">
    <style>
        /* تصميم الواجهة السريع */
        :root {
            --bg: #070912; --card: #131928; --blue: #4f8eff;
            --green: #00d68f; --red: #ff4d6a; --text: #e8edf5;
        }
        body { background: var(--bg); color: var(--text); font-family: 'Cairo', sans-serif; margin: 0; padding: 20px; }
        .card { background: var(--card); padding: 20px; border-radius: 20px; text-align: center; margin-bottom: 20px; border: 1px solid rgba(255,255,255,0.1); }
        .pos { border-color: var(--green); } .neg { border-color: var(--red); }
        h1 { font-size: 1.2rem; opacity: 0.8; }
        .amount { font-size: 2.5rem; font-weight: bold; margin: 10px 0; }
        .status { font-size: 0.9rem; color: #888; }
        .btn-group { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        button { padding: 15px; border-radius: 12px; border: none; cursor: pointer; font-weight: bold; font-family: 'Cairo'; }
        .btn-me { background: rgba(0, 214, 143, 0.2); color: var(--green); border: 1px solid var(--green); }
        .btn-bro { background: rgba(255, 77, 106, 0.2); color: var(--red); border: 1px solid var(--red); }
        .history { margin-top: 30px; }
        .tx-row { background: var(--card); padding: 15px; border-radius: 12px; margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; border-right: 4px solid #ccc; }
        .tx-me { border-right-color: var(--green); }
        .tx-bro { border-right-color: var(--red); }
        .sync-badge { font-size: 0.7rem; color: var(--green); display: none; margin-bottom: 10px; position: fixed; top: 10px; left: 10px; z-index: 100; }
        input { width: 100%; padding: 12px; margin-bottom: 10px; border-radius: 8px; border: 1px solid #333; background: #000; color: white; box-sizing: border-box; }
        
        /* تصميم زر الحذف */
        .del-btn { background: none; color: var(--red); border: none; font-size: 1.2rem; cursor: pointer; padding: 5px 10px; transition: 0.2s; }
        .del-btn:hover { transform: scale(1.2); opacity: 0.8; }
        .tx-details { flex-grow: 1; margin-right: 10px; }
    </style>
</head>
<body>

    <div id="syncing" class="sync-badge">● جاري المزامنة...</div>

    <div id="balance-card" class="card">
        <h1>الرصيد المتبقي</h1>
        <div id="bc-amount" class="amount">0.00</div>
        <div id="bc-status" class="status">جاري تحميل البيانات...</div>
    </div>

    <div class="card">
        <input type="number" id="inp-amount" placeholder="أدخل المبلغ (مثلاً: 500)">
        <input type="text" id="inp-reason" placeholder="ماذا اشتريت؟">
        <div class="btn-group">
            <button class="btn-me" onclick="addEntry('me')">أنا دفعت 👍</button>
            <button class="btn-bro" onclick="addEntry('bro')">أخي دفع 🤝</button>
        </div>
    </div>

    <div class="history">
        <h3>آخر العمليات</h3>
        <div id="home-list"></div>
    </div>

    <script>
        // مفاتيحك السحابية
        const BIN_ID = '69d6b58caaba882197d9cfc3';
        const API_KEY = '$2a$10$YGnu9GSy841IMPyEr.ExE.QKmCmtDFaoMYkYy6PSBIW2vV09/PpKC';
        const URL = `https://api.jsonbin.io/v3/b/${BIN_ID}`;

        let transactions = [];

        // 1. جلب البيانات عند التشغيل
        async function initApp() {
            showSync(true);
            try {
                const res = await fetch(`${URL}/latest`, { headers: { 'X-Master-Key': API_KEY } });
                const data = await res.json();
                transactions = data.record.transactions || [];
                renderUI();
            } catch (e) { alert("فشل الاتصال بالسحاب"); }
            showSync(false);
        }

        // 2. إضافة معاملة جديدة
        async function addEntry(type) {
            const amt = document.getElementById('inp-amount').value;
            const reason = document.getElementById('inp-reason').value || "بدون عنوان";
            if(!amt) return alert("أدخل المبلغ");

            const newTx = { 
                id: Date.now(), // إضافة معرف فريد للحذف
                type, 
                amount: parseFloat(amt), 
                reason, 
                date: new Date().toLocaleString('ar-DZ') 
            };

            transactions.unshift(newTx);
            renderUI();
            
            document.getElementById('inp-amount').value = '';
            document.getElementById('inp-reason').value = '';

            await saveToCloud();
        }

        // 3. حذف معاملة
        async function deleteEntry(id) {
            if(!confirm("هل أنت متأكد من حذف هذه العملية؟")) return;

            // تصفية المصفوفة لإزالة العنصر
            transactions = transactions.filter(t => t.id !== id);
            
            renderUI();
            await saveToCloud();
        }

        // 4. وظيفة الحفظ السحابي الموحدة
        async function saveToCloud() {
            showSync(true);
            try {
                await fetch(URL, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json', 'X-Master-Key': API_KEY },
                    body: JSON.stringify({ transactions })
                });
            } catch (e) { 
                alert("فشل التحديث السحابي!"); 
            }
            showSync(false);
        }

        // 5. تحديث واجهة المستخدم
        function renderUI() {
            let meTotal = 0, broTotal = 0;
            const list = document.getElementById('home-list');
            list.innerHTML = '';

            transactions.forEach(tx => {
                if(tx.type === 'me') meTotal += tx.amount; else broTotal += tx.amount;
                
                list.innerHTML += `
                    <div class="tx-row ${tx.type === 'me' ? 'tx-me' : 'tx-bro'}">
                        <div class="tx-details">
                            <strong>${tx.reason}</strong><br>
                            <small style="color:#888">${tx.date}</small>
                        </div>
                        <div style="font-weight:bold; margin-left:10px;">${tx.amount} DZD</div>
                        <button class="del-btn" onclick="deleteEntry(${tx.id})">🗑️</button>
                    </div>`;
            });

            const diff = meTotal - broTotal;
            const amtEl = document.getElementById('bc-amount');
            const statusEl = document.getElementById('bc-status');
            const cardEl = document.getElementById('balance-card');

            amtEl.innerText = Math.abs(diff).toFixed(2) + " DZD";
            if(diff > 0) {
                statusEl.innerText = "أخوك يدين لك"; cardEl.className = "card pos";
            } else if(diff < 0) {
                statusEl.innerText = "أنت مدين لأخيك"; cardEl.className = "card neg";
            } else {
                statusEl.innerText = "الحساب صافي"; cardEl.className = "card";
            }
        }

        function showSync(show) { document.getElementById('syncing').style.display = show ? 'block' : 'none'; }

        window.onload = initApp;
    </script>
</body>
</html>
