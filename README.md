<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>حسابات الإخوة السحابية</title>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #070912; --card: #131928; --blue: #4f8eff;
            --green: #00d68f; --red: #ff4d6a; --text: #e8edf5;
        }
        body { background: var(--bg); color: var(--text); font-family: 'Cairo', sans-serif; margin: 0; padding: 20px; }
        .card { background: var(--card); padding: 20px; border-radius: 20px; text-align: center; margin-bottom: 20px; border: 1px solid rgba(255,255,255,0.1); }
        .pos { border-color: var(--green); } .neg { border-color: var(--red); }
        h1 { font-size: 1.2rem; opacity: 0.8; margin: 0; }
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
        .sync-badge { font-size: 0.7rem; color: var(--green); display: none; margin-bottom: 10px; position: fixed; top: 10px; left: 10px; background: rgba(0,0,0,0.5); padding: 5px 10px; border-radius: 20px; }
        input { width: 100%; padding: 12px; margin-bottom: 10px; border-radius: 8px; border: 1px solid #333; background: #000; color: white; box-sizing: border-box; font-family: 'Cairo'; }
        .del-btn { background: none; color: var(--red); border: none; font-size: 1.2rem; cursor: pointer; padding: 5px; opacity: 0.6; }
    </style>
</head>
<body>

    <div id="syncing" class="sync-badge">● مزامنة...</div>

    <div id="balance-card" class="card">
        <h1 id="user-welcome">الرصيد المتبقي</h1>
        <div id="bc-amount" class="amount">0.00</div>
        <div id="bc-status" class="status">جاري التحميل...</div>
    </div>

    <div class="card">
        <input type="number" id="inp-amount" placeholder="المبلغ (DZD)">
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
        const BIN_ID = '69d6b58caaba882197d9cfc3';
        const API_KEY = '$2a$10$YGnu9GSy841IMPyEr.ExE.QKmCmtDFaoMYkYy6PSBIW2vV09/PpKC';
        const URL = `https://api.jsonbin.io/v3/b/${BIN_ID}`;

        let transactions = [];
        let myIdentity = localStorage.getItem('user_id');

        if (!myIdentity) {
            myIdentity = prompt("أدخل اسمك لمرة واحدة:");
            if (!myIdentity) myIdentity = "مستخدم";
            localStorage.setItem('user_id', myIdentity);
        }
        document.getElementById('user-welcome').innerText = `حساب: ${myIdentity}`;

        async function initApp() {
            showSync(true);
            try {
                const res = await fetch(`${URL}/latest`, { headers: { 'X-Master-Key': API_KEY } });
                const data = await res.json();
                transactions = data.record.transactions || [];
                renderUI();
            } catch (e) { console.error("خطأ في التحميل"); }
            showSync(false);
        }

        // --- هذه هي الدالة التي كنت تبحث عنها ---
        async function addEntry(type) {
            const amt = document.getElementById('inp-amount').value;
            const reason = document.getElementById('inp-reason').value || "بدون عنوان";
            if(!amt) return alert("أدخل المبلغ");

            const payer = (type === 'me') ? myIdentity : "الطرف الآخر";

            const newTx = { 
                id: Date.now(),
                payer: payer,
                amount: parseFloat(amt), 
                reason: reason, 
                date: new Date().toLocaleString('ar-DZ') 
            };

            transactions.unshift(newTx);
            renderUI();
            document.getElementById('inp-amount').value = '';
            document.getElementById('inp-reason').value = '';

            await saveToCloud();

            // إرسال إشعار واتساب إذا كنت أنت الدافع
            if(type === 'me') {
                if(confirm("هل تريد إبلاغ أخيك بالعملية عبر واتساب؟")) {
                    const text = `أهلاً، لقد سجلت عملية جديدة: ${reason} بمبلغ ${amt} DZD. تحقق من الحساب هنا:\n` + window.location.href;
                    window.open(`https://wa.me/?text=${encodeURIComponent(text)}`);
                }
            }
        }

        async function deleteEntry(id) {
            if(!confirm("حذف العملية؟")) return;
            transactions = transactions.filter(t => t.id !== id);
            renderUI();
            await saveToCloud();
        }

        async function saveToCloud() {
            showSync(true);
            try {
                await fetch(URL, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json', 'X-Master-Key': API_KEY },
                    body: JSON.stringify({ transactions })
                });
            } catch (e) { alert("فشل الحفظ السحابي"); }
            showSync(false);
        }

        function renderUI() {
            let myPaid = 0, othersPaid = 0;
            const list = document.getElementById('home-list');
            list.innerHTML = '';

            transactions.forEach(tx => {
                const isMe = (tx.payer === myIdentity);
                if(isMe) myPaid += tx.amount; else othersPaid += tx.amount;

                list.innerHTML += `
                    <div class="tx-row ${isMe ? 'tx-me' : 'tx-bro'}">
                        <div style="flex-grow:1">
                            <strong>${isMe ? 'أنا' : tx.payer}: ${tx.reason}</strong><br>
                            <small style="color:#888">${tx.date}</small>
                        </div>
                        <div style="font-weight:bold; margin: 0 15px;">${tx.amount} DZD</div>
                        <button class="del-btn" onclick="deleteEntry(${tx.id})">🗑️</button>
                    </div>`;
            });

            const diff = myPaid - othersPaid;
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
