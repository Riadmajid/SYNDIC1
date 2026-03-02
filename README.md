<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تسيير السانديك - Syndic Management</title>
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;500;700;800&display=swap" rel="stylesheet">
    
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-firestore.js"></script>

    <style>
        :root {
            --primary: #1e3a8a; --secondary: #3b82f6; --success: #10b981; 
            --danger: #ef4444; --warning: #f59e0b; --light: #f3f4f6; 
            --dark: #1f2937; --white: #ffffff;
        }

        * { box-sizing: border-box; font-family: 'Tajawal', sans-serif; }
        body { margin: 0; padding: 0; background-color: var(--light); color: var(--dark); transition: 0.3s; }
        
        /* Navbar */
        .navbar { background: linear-gradient(90deg, var(--primary), var(--secondary)); padding: 15px 20px; color: white; display: flex; justify-content: space-between; align-items: center; box-shadow: 0 4px 6px rgba(0,0,0,0.1); flex-wrap: wrap; gap: 10px; }
        .navbar-brand { display: flex; align-items: center; gap: 15px; flex-wrap: wrap; }
        .navbar h2 { margin: 0; font-weight: 800; font-size: 22px; }
        
        .nav-links button { background: rgba(255, 255, 255, 0.1); border: none; color: white; font-size: 15px; font-weight: 700; cursor: pointer; padding: 8px 12px; border-radius: 8px; margin: 0 3px; transition: 0.3s; }
        .nav-links button:hover, .nav-links button.active { background-color: white; color: var(--primary); }
        
        select.lang-select { background: rgba(255,255,255,0.2); color: white; border: 1px solid rgba(255,255,255,0.5); padding: 8px 12px; border-radius: 8px; font-weight: bold; font-family: 'Tajawal', sans-serif; outline: none; cursor: pointer; }
        select.lang-select option { color: var(--dark); background: white; }

        /* Auth Buttons */
        .auth-controls { display: flex; gap: 10px; }
        .btn-icon { background: rgba(255,255,255,0.2); border: 1px solid rgba(255,255,255,0.5); border-radius: 8px; padding: 8px 15px; color: white; cursor: pointer; font-weight: bold; font-family: 'Tajawal', sans-serif; transition: 0.3s; display: flex; align-items: center; gap: 5px; }
        .btn-icon:hover { background: white; color: var(--primary); }
        .btn-icon.unlocked { background: #10b981; border-color: #10b981; }

        /* Container & Pages */
        .container { padding: 30px 20px; max-width: 1300px; margin: 0 auto; }
        .page { display: none; animation: fadeIn 0.4s; }
        .page.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        
        /* Stats Grid */
        .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 20px; margin-bottom: 30px; }
        .stat-card { background: var(--white); padding: 30px 20px; border-radius: 15px; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.05); text-align: center; border-bottom: 5px solid transparent; transition: transform 0.3s ease; }
        .stat-card:hover { transform: translateY(-5px); }
        .stat-card.incomes { border-color: var(--success); }
        .stat-card.expenses { border-color: var(--danger); }
        .stat-card.balance { border-color: var(--secondary); }
        .stat-icon { font-size: 40px; margin-bottom: 10px; line-height: 1; }
        .stat-card h3 { margin: 0 0 10px 0; color: #6b7280; font-size: 16px; font-weight: 700; }
        .stat-card .amount { font-size: 28px; font-weight: 800; }
        .text-success { color: var(--success); }
        .text-danger { color: var(--danger); }
        .text-primary { color: var(--secondary); }

        /* Actions & Selects */
        .header-action { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; background: white; padding: 15px 25px; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.02); flex-wrap: wrap; gap: 15px; }
        .header-action h3 { margin: 0; font-size: 18px; color: var(--primary); }
        select.modern-select { padding: 10px 20px; border: 2px solid #e5e7eb; border-radius: 8px; font-size: 15px; font-weight: bold; color: var(--dark); outline: none; cursor: pointer; transition: 0.3s; }
        select.modern-select:focus { border-color: var(--secondary); }

        /* Tables */
        .table-wrapper { background: white; border-radius: 15px; padding: 0; box-shadow: 0 10px 20px rgba(0,0,0,0.05); overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; min-width: 800px; }
        th, td { padding: 12px; text-align: center; border-bottom: 1px solid #f3f4f6; }
        th { background-color: #f8fafc; color: #475569; font-weight: 700; font-size: 14px; white-space: nowrap; }
        html[dir="rtl"] th:first-child, html[dir="rtl"] td:first-child { position: sticky; right: 0; background-color: #ffffff; z-index: 2; box-shadow: -2px 0 5px rgba(0,0,0,0.03); text-align: right; padding-right: 20px; }
        html[dir="ltr"] th:first-child, html[dir="ltr"] td:first-child { position: sticky; left: 0; background-color: #ffffff; z-index: 2; box-shadow: 2px 0 5px rgba(0,0,0,0.03); text-align: left; padding-left: 20px; }
        th:first-child { background-color: #f8fafc !important; color: var(--primary); font-weight: 800; }
        td:first-child { font-weight: 800; color: var(--primary); }
        tbody tr:hover td { background-color: #f9fafb; }
        .badge { padding: 5px 10px; border-radius: 20px; font-size: 12px; font-weight: 700; display: inline-flex; align-items: center; justify-content: center; gap: 5px; white-space: nowrap; }
        .badge-paid { background-color: #dcfce7; color: #166534; border: 1px solid #bbf7d0; }
        .badge-unpaid { background-color: #fee2e2; color: #991b1b; border: 1px solid #fecaca; }

        /* Buttons & Inputs */
        .form-group { margin-bottom: 15px; display: flex; gap: 10px; flex-wrap: wrap; }
        input[type="text"], input[type="number"], input[type="date"], input[type="password"], textarea { padding: 12px 15px; border: 1px solid #d1d5db; border-radius: 8px; flex: 1; font-size: 15px; outline: none; transition: 0.3s; }
        textarea { resize: vertical; min-height: 45px; }
        input:focus, textarea:focus { border-color: var(--secondary); box-shadow: 0 0 0 3px rgba(59,130,246,0.1); }
        button.btn { padding: 10px 20px; background-color: var(--success); color: white; border: none; border-radius: 8px; cursor: pointer; font-weight: 700; font-size: 14px; transition: 0.3s; box-shadow: 0 4px 6px rgba(16,185,129,0.2); }
        button.btn:hover { transform: translateY(-2px); box-shadow: 0 6px 8px rgba(16,185,129,0.3); }
        button.btn-danger { background-color: var(--danger); box-shadow: 0 4px 6px rgba(239,68,68,0.2); }
        button.btn-danger:hover { background-color: #dc2626; }
        button.btn-primary { background-color: var(--primary); box-shadow: 0 4px 6px rgba(30,58,138,0.2); }
        button.btn-primary:hover { background-color: #152c6b; }
        button.btn-warning { background-color: var(--warning); box-shadow: 0 4px 6px rgba(245,158,11,0.2); }

        /* Grids & Cards */
        .grid-cards { display: grid; grid-template-columns: repeat(auto-fill, minmax(160px, 1fr)); gap: 15px; margin-top: 20px; }
        .item-card { background: white; padding: 25px 10px 15px 10px; text-align: center; border-radius: 12px; box-shadow: 0 4px 6px rgba(0,0,0,0.05); cursor: pointer; transition: 0.3s; border: 2px solid transparent; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 8px; position: relative; }
        .item-card:hover { transform: translateY(-5px); border-color: var(--secondary); box-shadow: 0 10px 15px rgba(0,0,0,0.1); }
        .item-card .icon { font-size: 40px; line-height: 1; }
        .item-card .name { font-size: 16px; font-weight: 800; color: var(--dark); }
        
        /* Card Action Buttons (Edit/Delete) */
        .card-actions { position: absolute; top: 5px; left: 5px; right: 5px; display: flex; justify-content: space-between; }
        .card-action-btn { background: none; border: none; font-size: 14px; cursor: pointer; opacity: 0.5; transition: 0.2s; padding: 2px 5px; }
        .card-action-btn:hover { opacity: 1; transform: scale(1.2); }

        .months-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(100px, 1fr)); gap: 15px; margin-top: 20px; }
        .month-card { padding: 15px 10px; text-align: center; border-radius: 10px; cursor: pointer; font-weight: 700; border: 2px solid #e5e7eb; background: white; transition: 0.2s; display: flex; flex-direction: column; gap: 5px; }
        .month-card:hover { border-color: var(--secondary); transform: scale(1.05); }
        .month-card.paid { background-color: #ecfdf5; border-color: var(--success); color: var(--success); }
        .month-card.paid span { color: #065f46; font-size: 13px; font-weight: 800; }
        
        .section-box { display: none; animation: fadeIn 0.3s; }
        .section-box.active { display: block; }

        /* Communication Page */
        .comm-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap: 20px; align-items: start; }
        .announcements-wrapper, .chat-wrapper { background: white; border-radius: 15px; padding: 20px; box-shadow: 0 5px 15px rgba(0,0,0,0.05); }
        .announce-list { display: flex; flex-direction: column; gap: 15px; margin-top: 20px; max-height: 500px; overflow-y: auto; padding-right: 5px; }
        .announce-item { background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 10px; padding: 15px; position: relative; }
        .announce-item h4 { margin: 0 0 5px 0; color: var(--primary); font-size: 16px; font-weight: 800; }
        .announce-item p { margin: 0; color: #475569; font-size: 14px; line-height: 1.5; }
        .announce-item .date { font-size: 11px; color: #94a3b8; margin-bottom: 8px; display: block; }
        .announce-item .del-btn { position: absolute; top: 10px; cursor: pointer; color: var(--danger); font-size: 16px; background: none; border: none; padding: 5px; transition: 0.2s; }
        html[dir="rtl"] .announce-item .del-btn { left: 10px; } html[dir="ltr"] .announce-item .del-btn { right: 10px; }
        
        .chat-box { flex: 1; min-height: 350px; max-height: 450px; background: #f1f5f9; border-radius: 10px; padding: 15px; overflow-y: auto; margin-bottom: 15px; display: flex; flex-direction: column; gap: 10px; }
        .chat-msg { background: white; padding: 10px 15px; border-radius: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); width: fit-content; max-width: 85%; position: relative; }
        .chat-msg .sender { font-weight: bold; color: var(--secondary); font-size: 13px; margin-bottom: 3px; display: block; }
        .chat-msg .text { margin: 0; font-size: 14px; color: var(--dark); line-height: 1.4; }
        .chat-msg .time { font-size: 10px; color: #9ca3af; margin-top: 5px; display: block; text-align: right; }
        html[dir="rtl"] .chat-msg .time { text-align: left; }
        .del-chat-btn { position: absolute; top: 5px; cursor: pointer; color: var(--danger); font-size: 12px; background: none; border: none; padding: 5px; opacity: 0.5; transition: 0.2s; }
        .del-chat-btn:hover { opacity: 1; transform: scale(1.2); }
        html[dir="rtl"] .del-chat-btn { left: 5px; } html[dir="ltr"] .del-chat-btn { right: 5px; }

        /* Modals & Loaders */
        .modal-overlay { display: none; position: fixed; z-index: 1000; left: 0; top: 0; width: 100%; height: 100%; background-color: rgba(17,24,39,0.7); align-items: center; justify-content: center; backdrop-filter: blur(4px); }
        .modal-content { background-color: white; padding: 30px; border-radius: 20px; width: 90%; max-width: 400px; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1); text-align: center; animation: fadeIn 0.3s; }
        .modal-content h3 { margin-top: 0; color: var(--primary); font-size: 22px; font-weight: 800; }
        .modal-content input { width: 100%; margin: 20px 0; font-size: 18px; text-align: center; padding: 15px; border-radius: 10px; background: #f9fafb; border: 2px solid #e5e7eb; }
        .modal-content input:focus { border-color: var(--secondary); background: white; }
        .modal-buttons { display: flex; justify-content: center; gap: 10px; flex-wrap: wrap; }
        .modal-buttons button { flex: 1; min-width: 100px; }
        
        #loadingScreen { position: fixed; top:0; left:0; width: 100%; height: 100%; background: var(--light); z-index: 9999; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .spinner { border: 5px solid #f3f3f3; border-top: 5px solid var(--primary); border-radius: 50%; width: 50px; height: 50px; animation: spin 1s linear infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body>

    <div id="loadingScreen">
        <div class="spinner"></div>
        <h3 style="color: var(--primary); margin-top: 20px;" id="loadingText">جاري الاتصال بقاعدة البيانات...</h3>
    </div>

    <div class="navbar">
        <div class="navbar-brand">
            <h2 data-i18n="app_title">🏢 تسيير السانديك</h2>
            <select id="langSelect" class="lang-select" onchange="changeLanguage()">
                <option value="ar">العربية</option>
                <option value="fr">Français</option>
                <option value="en">English</option>
            </select>
            <div class="auth-controls">
                <button id="btn-lock" class="btn-icon" onclick="toggleAuth()">🔒 <span data-i18n="btn_locked">مقفل</span></button>
                <button class="btn-icon" onclick="openChangePwdModal()" title="تغيير كلمة المرور">🔑</button>
            </div>
        </div>
        <div class="nav-links">
            <button onclick="changePage('dashboard')" class="active" id="btn-dashboard" data-i18n="nav_dashboard">الرئيسية</button>
            <button onclick="changePage('apartments')" id="btn-apartments" data-i18n="nav_apartments">العمارات والشقق</button>
            <button onclick="changePage('expenses')" id="btn-expenses" data-i18n="nav_expenses">المصاريف</button>
            <button onclick="changePage('communication')" id="btn-communication" data-i18n="nav_comm">الإعلانات والدردشة</button>
        </div>
    </div>

    <div class="container">
        <!-- PAGE 1: DASHBOARD -->
        <div id="dashboard" class="page active">
            <div class="stats-grid">
                <div class="stat-card incomes"><div class="stat-icon">💰</div><h3 data-i18n="stat_incomes">مجموع المداخيل</h3><div class="amount text-success" id="totalIncomes">0</div></div>
                <div class="stat-card expenses"><div class="stat-icon">📉</div><h3 data-i18n="stat_expenses">مجموع المصاريف</h3><div class="amount text-danger" id="totalExpenses">0</div></div>
                <div class="stat-card balance"><div class="stat-icon">🏦</div><h3 data-i18n="stat_balance">صندوق السانديك</h3><div class="amount text-primary" id="boxBalance">0</div></div>
            </div>
            <div class="header-action">
                <div>
                    <h3 style="margin: 0; margin-bottom: 8px;" data-i18n="dash_table_title">📋 الأداء (آخر 10 شقق فقط)</h3>
                    <button class="btn btn-primary" style="padding: 6px 15px; font-size: 13px;" onclick="changePage('full-payments')" data-i18n="btn_view_all">📄 عرض تفاصيل جميع الشقق</button>
                </div>
                <select id="dashboardYear" class="modern-select" onchange="renderDashboardTable()"></select>
            </div>
            <div class="table-wrapper">
                <table id="dashboardTable"><thead><tr id="dashboardTableHead"></tr></thead><tbody id="dashboardTableBody"></tbody></table>
            </div>
        </div>

        <!-- PAGE 4: FULL PAYMENTS -->
        <div id="full-payments" class="page">
            <div class="header-action">
                <div style="display: flex; gap: 15px; align-items: center;">
                    <button class="btn btn-danger" onclick="changePage('dashboard')" data-i18n="btn_back_dash">🔙 رجوع</button>
                    <h3 style="margin: 0;" data-i18n="full_table_title">الجدول الكامل لجميع الشقق 🏢</h3>
                </div>
                <select id="fullYear" class="modern-select" onchange="renderFullTable()"></select>
            </div>
            <div id="fullBldListSection" class="section-box active">
                <p style="color: #6b7280; font-weight: bold; margin-top: 0;">اختر العمارة لعرض جدول الأداء الخاص بها:</p>
                <div class="grid-cards" id="fullBuildingsGrid"></div>
            </div>
            <div id="fullTableSection" class="section-box">
                <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #f3f4f6; padding-bottom: 15px; margin-bottom: 20px;">
                    <h2 id="currentFullBldName" style="margin: 0; color: var(--primary);"></h2>
                    <button class="btn btn-danger" onclick="closeFullTableBuilding()" data-i18n="btn_back_blds">🔙 رجوع للعمارات</button>
                </div>
                <div class="table-wrapper">
                    <table id="fullTable"><thead><tr id="fullTableHead"></tr></thead><tbody id="fullTableBody"></tbody></table>
                </div>
            </div>
        </div>

        <!-- PAGE 2: BUILDINGS & APTS -->
        <div id="apartments" class="page">
            <!-- BUILDINGS -->
            <div id="bldListSection" class="section-box active">
                <div class="header-action" style="padding: 20px; background: white; border-radius: 15px; margin-bottom: 20px;">
                    <div class="form-group" style="margin: 0; width: 100%;">
                        <input type="text" id="newBldName" data-i18n-placeholder="ph_bld_name" placeholder="مثال: العمارة أ...">
                        <button class="btn" onclick="addBuilding()" data-i18n="btn_add_bld">+ إضافة عمارة</button>
                    </div>
                </div>
                <div class="grid-cards" id="buildingsGrid"></div>
            </div>
            <!-- APARTMENTS -->
            <div id="aptListSection" class="section-box">
                <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #f3f4f6; padding-bottom: 15px; margin-bottom: 20px;">
                    <h2 id="currentBldName" style="margin: 0; color: var(--primary);"></h2>
                    <button class="btn btn-danger" onclick="closeBuilding()" data-i18n="btn_back_blds">🔙 رجوع</button>
                </div>
                <div class="header-action" style="padding: 20px; background: white; border-radius: 15px; margin-bottom: 20px;">
                    <div class="form-group" style="margin: 0; width: 100%;">
                        <input type="text" id="newAptName" data-i18n-placeholder="ph_apt_name" placeholder="مثال: شقة 1...">
                        <button class="btn" onclick="addApartment()" data-i18n="btn_add_apt">+ إضافة شقة</button>
                    </div>
                </div>
                <div class="grid-cards" id="apartmentsGrid"></div>
            </div>
            <!-- PAYMENTS -->
            <div id="aptDetailsSection" class="section-box" style="background: white; padding: 30px; border-radius: 15px; box-shadow: 0 10px 20px rgba(0,0,0,0.05);">
                <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #f3f4f6; padding-bottom: 15px; margin-bottom: 20px;">
                    <h2 id="currentAptName" style="margin: 0; color: var(--primary);"></h2>
                    <button class="btn btn-danger" onclick="closeAptDetails()" data-i18n="btn_back_apts">🔙 رجوع</button>
                </div>
                <div class="form-group" style="max-width: 250px;">
                    <label style="font-weight: bold; color: #4b5563;" data-i18n="lbl_year">اختر السنة:</label>
                    <select id="paymentYear" class="modern-select" style="width: 100%;" onchange="renderMonths()"></select>
                </div>
                <div class="months-grid" id="monthsGrid"></div>
            </div>
        </div>

        <!-- PAGE 3: EXPENSES -->
        <div id="expenses" class="page">
            <div class="header-action" style="flex-direction: column; align-items: stretch; gap: 15px;">
                <h3 style="margin: 0;" data-i18n="title_new_exp">تسجيل مصروف جديد 💸</h3>
                <div class="form-group" style="margin: 0;">
                    <input type="text" id="expenseDesc" data-i18n-placeholder="ph_exp_desc" placeholder="فاش تخسرو لفلوس؟">
                    <input type="number" id="expenseAmount" data-i18n-placeholder="ph_amount" placeholder="المبلغ (درهم)">
                    <input type="date" id="expenseDate">
                    <button class="btn" onclick="addExpense()" data-i18n="btn_add_exp">+ إضافة مصروف</button>
                </div>
            </div>
            <div class="table-wrapper" style="margin-top: 20px;">
                <table>
                    <thead><tr><th data-i18n="th_date">التاريخ</th><th data-i18n="th_desc">البيان (السبب)</th><th data-i18n="th_amount">المبلغ</th><th data-i18n="th_action">إجراء</th></tr></thead>
                    <tbody id="expensesTableBody"></tbody>
                </table>
            </div>
        </div>

        <!-- PAGE 5: COMMUNICATION -->
        <div id="communication" class="page">
            <div class="comm-grid">
                <div class="announcements-wrapper">
                    <h3 style="margin-top: 0; color: var(--primary); font-size: 20px;" data-i18n="title_announcements">📢 إعلانات العمارة</h3>
                    <div class="form-group" style="flex-direction: column; gap: 10px;">
                        <input type="text" id="announceTitle" data-i18n-placeholder="ph_announce_title" placeholder="عنوان الإعلان...">
                        <textarea id="announceMsg" data-i18n-placeholder="ph_announce_msg" placeholder="محتوى الإعلان..."></textarea>
                        <button class="btn" onclick="addAnnouncement()" data-i18n="btn_add_announce">نشر الإعلان ✔️</button>
                    </div>
                    <div class="announce-list" id="announcementsList"></div>
                </div>
                <div class="chat-wrapper">
                    <h3 style="margin-top: 0; color: var(--primary); font-size: 20px;" data-i18n="title_chat">💬 دردشة السكان</h3>
                    <div class="chat-box" id="chatBox"></div>
                    <div class="form-group" style="margin-bottom: 0;">
                        <input type="text" id="chatName" data-i18n-placeholder="ph_chat_name" placeholder="الاسم" style="max-width: 100px;">
                        <input type="text" id="chatMsg" data-i18n-placeholder="ph_chat_msg" placeholder="اكتب رسالتك..." onkeypress="handleChatKeyPress(event)">
                        <button class="btn btn-primary" onclick="sendChatMessage()" data-i18n="btn_send_chat">إرسال 🚀</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- MODALS -->
    <!-- 1. Password Auth Modal -->
    <div id="authModal" class="modal-overlay">
        <div class="modal-content">
            <div class="stat-icon" style="font-size: 50px;">🔒</div>
            <h3 data-i18n="auth_title">أدخل كلمة المرور</h3>
            <p style="font-size: 13px; color: #6b7280; margin: 0;" data-i18n="auth_desc">المرجو إدخال كلمة المرور لتأكيد التعديلات.</p>
            <input type="password" id="authPwdInput" data-i18n-placeholder="ph_pwd" placeholder="كلمة المرور..." onkeypress="if(event.key==='Enter') verifyAuth()">
            <div class="modal-buttons">
                <button class="btn" onclick="verifyAuth()" data-i18n="btn_confirm">تأكيد ✔️</button>
                <button class="btn btn-danger" onclick="closeAuthModal()" data-i18n="btn_cancel">إلغاء ✖️</button>
            </div>
        </div>
    </div>

    <!-- 2. Change Password Modal -->
    <div id="changePwdModal" class="modal-overlay">
        <div class="modal-content">
            <div class="stat-icon" style="font-size: 50px;">🔑</div>
            <h3 data-i18n="change_pwd_title">تغيير كلمة المرور</h3>
            <input type="password" id="oldPwdInput" data-i18n-placeholder="ph_old_pwd" placeholder="كلمة المرور الحالية">
            <input type="password" id="newPwdInput" data-i18n-placeholder="ph_new_pwd" placeholder="كلمة المرور الجديدة" onkeypress="if(event.key==='Enter') saveNewPassword()">
            <div class="modal-buttons">
                <button class="btn" onclick="saveNewPassword()" data-i18n="btn_save">حفظ ✔️</button>
                <button class="btn btn-danger" onclick="closeChangePwdModal()" data-i18n="btn_cancel">إلغاء ✖️</button>
            </div>
        </div>
    </div>

    <!-- 3. Payment Modal -->
    <div id="paymentModal" class="modal-overlay">
        <div class="modal-content">
            <div class="stat-icon" style="font-size: 50px;">💵</div>
            <h3 id="modalTitle"></h3>
            <input type="number" id="modalAmountInput" placeholder="0.00">
            <div class="modal-buttons">
                <button class="btn" onclick="savePayment()" data-i18n="btn_save">✔️ حفظ</button>
                <button class="btn btn-warning" onclick="clearPayment()" data-i18n="btn_clear">✖️ مسح</button>
                <button class="btn btn-danger" onclick="closePaymentModal()" data-i18n="btn_cancel">إلغاء</button>
            </div>
        </div>
    </div>

    <!-- 4. Rename Modal (NEW) -->
    <div id="renameModal" class="modal-overlay">
        <div class="modal-content">
            <div class="stat-icon" style="font-size: 50px;">✏️</div>
            <h3 data-i18n="modal_rename_title">تغيير الاسم</h3>
            <input type="text" id="renameInput" data-i18n-placeholder="ph_rename" placeholder="الاسم الجديد..." onkeypress="if(event.key==='Enter') saveRename()">
            <div class="modal-buttons">
                <button class="btn" onclick="saveRename()" data-i18n="btn_save">✔️ حفظ</button>
                <button class="btn btn-danger" onclick="closeRenameModal()" data-i18n="btn_cancel">إلغاء</button>
            </div>
        </div>
    </div>

    <!-- 5. Global Delete Confirm Modal -->
    <div id="deleteModal" class="modal-overlay">
        <div class="modal-content">
            <div class="stat-icon" style="font-size: 50px;">🗑️</div>
            <h3 style="color: var(--danger);" data-i18n="modal_del_title">تأكيد المسح</h3>
            <p style="color: #4b5563; font-weight: 600; font-size: 14px; margin-bottom: 20px;" id="deleteMsgText"></p>
            <div class="modal-buttons">
                <button class="btn btn-danger" onclick="confirmDelete()" data-i18n="btn_yes_del">نعم، امسح</button>
                <button class="btn" style="background-color: #9ca3af;" onclick="closeDeleteModal()" data-i18n="btn_undo">تراجع</button>
            </div>
        </div>
    </div>

    <script>
        // ================= FIREBASE INITIALIZATION =================
        const firebaseConfig = {
            apiKey: "AIzaSyDdIUHvUTRze0Pj8TqF36sKnlRUQcSCtUM",
            authDomain: "syndic-54327.firebaseapp.com",
            projectId: "syndic-54327",
            storageBucket: "syndic-54327.firebasestorage.app",
            messagingSenderId: "221199418649",
            appId: "1:221199418649:web:0569352de9de01390af188",
            measurementId: "G-37C59NTMP0"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();
        db.enablePersistence().catch(function(err) { console.log("Offline mode warning: ", err.code); });

        // --- TRANSLATION DICTIONARY ---
        const dict = {
            ar: {
                app_title: "🏢 تسيير السانديك", nav_dashboard: "الرئيسية", nav_apartments: "العمارات والشقق", nav_expenses: "المصاريف", nav_comm: "الإعلانات والدردشة",
                stat_incomes: "مجموع المداخيل", stat_expenses: "مجموع المصاريف", stat_balance: "صندوق السانديك", dash_table_title: "📋 الأداء (آخر 10 شقق فقط)", btn_view_all: "📄 عرض تفاصيل جميع الشقق", full_table_title: "الجدول الكامل لجميع الشقق 🏢", btn_back_dash: "🔙 رجوع للرئيسية", 
                ph_bld_name: "مثال: العمارة A...", btn_add_bld: "+ إضافة عمارة", btn_back_blds: "🔙 رجوع للعمارات", alert_bld: "المرجو كتابة اسم العمارة", ph_apt_name: "مثال: شقة 1...", btn_add_apt: "+ إضافة شقة", btn_back_apts: "🔙 رجوع للشقق",
                lbl_year: "اختر السنة:", title_new_exp: "تسجيل مصروف جديد 💸", ph_exp_desc: "فاش تخسرو لفلوس؟", ph_amount: "المبلغ", btn_add_exp: "+ إضافة مصروف", th_date: "التاريخ", th_desc: "البيان (السبب)", th_amount: "المبلغ", th_action: "إجراء",
                title_announcements: "📢 إعلانات العمارة", ph_announce_title: "عنوان الإعلان...", ph_announce_msg: "محتوى الإعلان...", btn_add_announce: "نشر الإعلان ✔️", title_chat: "💬 دردشة السكان", ph_chat_name: "الاسم", ph_chat_msg: "اكتب رسالتك هنا...", btn_send_chat: "إرسال 🚀",
                btn_save: "✔️ حفظ", btn_clear: "✖️ مسح", btn_cancel: "إلغاء", modal_del_title: "تأكيد المسح", btn_yes_del: "نعم، امسح", btn_undo: "تراجع", curr: "د.م", no_apts: "لا توجد شقق مسجلة بعد.", paid: "✔️", unpaid: "✖️ غير مؤدى", apt_month: "الشقة / الشهر", year_word: "سنة", btn_delete: "مسح 🗑️",
                alert_apt: "المرجو كتابة اسم أو رقم الشقة", alert_amt: "مبلغ غير صحيح!", alert_fill: "المرجو ملء جميع المعلومات.", months:["يناير", "فبراير", "مارس", "أبريل", "ماي", "يونيو", "يوليوز", "غشت", "شتنبر", "أكتوبر", "نونبر", "دجنبر"], pay_title: "أداء",
                btn_locked: "مقفل", btn_unlocked: "مفتوح", auth_title: "أدخل كلمة المرور", auth_desc: "المرجو إدخال كلمة المرور لتأكيد التعديلات.", ph_pwd: "كلمة المرور...", btn_confirm: "تأكيد ✔️",
                change_pwd_title: "تغيير كلمة المرور", ph_old_pwd: "كلمة المرور الحالية", ph_new_pwd: "كلمة المرور الجديدة", alert_wrong_pwd: "كلمة المرور غير صحيحة!", alert_pwd_changed: "تم تغيير كلمة المرور بنجاح!", alert_pwd_req: "المرجو إدخال كلمة مرور صالحة.",
                modal_rename_title: "تغيير الاسم", ph_rename: "الاسم الجديد...",
                msg_del_default: "هل أنت متأكد من المسح؟ (لا يمكن التراجع)", msg_del_bld: "تحذير: مسح العمارة سيؤدي إلى مسح جميع الشقق التابعة لها نهائياً!"
            },
            fr: {
                app_title: "🏢 Gestion Syndic", nav_dashboard: "Tableau de bord", nav_apartments: "Immeubles & Appts", nav_expenses: "Dépenses", nav_comm: "Annonces & Chat", stat_incomes: "Total Revenus", stat_expenses: "Total Dépenses", stat_balance: "Caisse Syndic", dash_table_title: "📋 Paiements (10 derniers)", btn_view_all: "📄 Voir tous les appartements", full_table_title: "Tableau complet 🏢", btn_back_dash: "🔙 Retour à l'accueil", 
                ph_bld_name: "Ex: Bâtiment A...", btn_add_bld: "+ Ajouter Immeuble", btn_back_blds: "🔙 Retour aux immeubles", alert_bld: "Entrez le nom de l'immeuble", ph_apt_name: "Ex: Appt 1...", btn_add_apt: "+ Ajouter Appt", btn_back_apts: "🔙 Retour aux Appts", lbl_year: "Choisir l'année :", title_new_exp: "Nouvelle Dépense 💸", ph_exp_desc: "Description", ph_amount: "Montant", btn_add_exp: "+ Ajouter Dépense", th_date: "Date", th_desc: "Description", th_amount: "Montant", th_action: "Action",
                title_announcements: "📢 Annonces", ph_announce_title: "Titre de l'annonce...", ph_announce_msg: "Contenu...", btn_add_announce: "Publier ✔️", title_chat: "💬 Chat des résidents", ph_chat_name: "Nom", ph_chat_msg: "Écrivez un message...", btn_send_chat: "Envoyer 🚀", btn_save: "✔️ Enregistrer", btn_clear: "✖️ Effacer", btn_cancel: "Annuler", modal_del_title: "Confirmation", btn_yes_del: "Oui", btn_undo: "Annuler", curr: "MAD", no_apts: "Aucun appartement enregistré.", paid: "✔️", unpaid: "✖️ Non payé", apt_month: "Appt / Mois", year_word: "Année", btn_delete: "Supprimer",
                alert_apt: "Entrez le nom de l'appartement", alert_amt: "Montant invalide!", alert_fill: "Remplissez toutes les infos.", months:["Janvier", "Février", "Mars", "Avril", "Mai", "Juin", "Juillet", "Août", "Septembre", "Octobre", "Novembre", "Décembre"], pay_title: "Paiement",
                btn_locked: "Verrouillé", btn_unlocked: "Déverrouillé", auth_title: "Mot de passe requis", auth_desc: "Veuillez entrer le mot de passe pour modifier.", ph_pwd: "Mot de passe...", btn_confirm: "Confirmer ✔️", change_pwd_title: "Changer le mot de passe", ph_old_pwd: "Mot de passe actuel", ph_new_pwd: "Nouveau mot de passe", alert_wrong_pwd: "Mot de passe incorrect!", alert_pwd_changed: "Mot de passe changé avec succès!", alert_pwd_req: "Veuillez entrer un mot de passe valide.",
                modal_rename_title: "Changer le nom", ph_rename: "Nouveau nom...",
                msg_del_default: "Voulez-vous vraiment supprimer ?", msg_del_bld: "Attention : Supprimer l'immeuble supprimera tous ses appartements !"
            },
            en: {
                app_title: "🏢 Syndic Manager", nav_dashboard: "Dashboard", nav_apartments: "Buildings & Appts", nav_expenses: "Expenses", nav_comm: "Announcements & Chat", stat_incomes: "Total Incomes", stat_expenses: "Total Expenses", stat_balance: "Syndic Balance", dash_table_title: "📋 Payments (Last 10)", btn_view_all: "📄 View all apartments", full_table_title: "Full Apartments Table 🏢", btn_back_dash: "🔙 Back to Dashboard", 
                ph_bld_name: "E.g.: Building A...", btn_add_bld: "+ Add Building", btn_back_blds: "🔙 Back to Buildings", alert_bld: "Enter building name", ph_apt_name: "E.g.: Apt 1...", btn_add_apt: "+ Add Appt", btn_back_apts: "🔙 Back to Appts", lbl_year: "Select Year:", title_new_exp: "New Expense 💸", ph_exp_desc: "Description", ph_amount: "Amount", btn_add_exp: "+ Add Expense", th_date: "Date", th_desc: "Description", th_amount: "Amount", th_action: "Action",
                title_announcements: "📢 Announcements", ph_announce_title: "Title...", ph_announce_msg: "Content...", btn_add_announce: "Publish ✔️", title_chat: "💬 Residents Chat", ph_chat_name: "Name", ph_chat_msg: "Type message...", btn_send_chat: "Send 🚀", btn_save: "✔️ Save", btn_clear: "✖️ Clear", btn_cancel: "Cancel", modal_del_title: "Confirm Deletion", btn_yes_del: "Yes", btn_undo: "Undo", curr: "MAD", no_apts: "No apartments registered yet.", paid: "✔️", unpaid: "✖️ Unpaid", apt_month: "Apt / Month", year_word: "Year", btn_delete: "Delete",
                alert_apt: "Enter apartment name", alert_amt: "Invalid amount!", alert_fill: "Fill all information.", months:["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"], pay_title: "Payment",
                btn_locked: "Locked", btn_unlocked: "Unlocked", auth_title: "Password Required", auth_desc: "Please enter the password to modify data.", ph_pwd: "Password...", btn_confirm: "Confirm ✔️", change_pwd_title: "Change Password", ph_old_pwd: "Current Password", ph_new_pwd: "New Password", alert_wrong_pwd: "Incorrect password!", alert_pwd_changed: "Password changed successfully!", alert_pwd_req: "Please enter a valid password.",
                modal_rename_title: "Change Name", ph_rename: "New name...",
                msg_del_default: "Are you sure you want to delete?", msg_del_bld: "Warning: Deleting the building will delete all its apartments!"
            }
        };

        // --- 1. STATE & FIREBASE DATA SYNC ---
        let currentLang = localStorage.getItem('syndicLang') || 'ar';
        let data = JSON.parse(localStorage.getItem('syndicLocalData')) || { buildings: [], apartments:[], payments: {}, expenses:[], announcements:[], chat:[] };
        let isDataLoaded = false;
        let isAuthenticated = false; let pendingCallback = null; 
        let currentSelectedBuildingId = null, currentSelectedAptId = null, selectedMonthForPay = null, selectedYearForPay = null;
        let currentFullBuildingId = null; 
        let itemToDeleteId = null, itemToDeleteType = null; 
        let itemToRenameId = null, itemToRenameType = null; // للمتغيرات ديال التعديل (Rename)

        db.collection("syndic_data").doc("main").onSnapshot((doc) => {
            if (doc.exists) {
                const fetchedData = doc.data();
                data.buildings = fetchedData.buildings ||[]; data.apartments = fetchedData.apartments ||[]; data.payments = fetchedData.payments || {}; data.expenses = fetchedData.expenses ||[]; data.announcements = fetchedData.announcements ||[]; data.chat = fetchedData.chat ||[];
                localStorage.setItem('syndicLocalData', JSON.stringify(data)); 
            } else { db.collection("syndic_data").doc("main").set(data).catch(e => console.log("Set failed", e)); }
            if (!isDataLoaded) { isDataLoaded = true; document.getElementById('loadingScreen').style.display = 'none'; initApp(); } else { refreshUI(); }
        }, (error) => {
            console.error("Firestore Error: ", error);
            if (!isDataLoaded) { isDataLoaded = true; document.getElementById('loadingScreen').style.display = 'none'; alert("⚠️ لا يمكن الاتصال بقاعدة البيانات حالياً. التطبيق يعمل في وضع (Offline)."); initApp(); }
        });

        setTimeout(() => { if(!isDataLoaded) { isDataLoaded = true; document.getElementById('loadingScreen').style.display = 'none'; initApp(); } }, 3000);

        function saveData() { 
            localStorage.setItem('syndicLocalData', JSON.stringify(data));
            db.collection("syndic_data").doc("main").set(data).catch(e => console.log("Save error", e));
            updateDashboardStats(); 
        }
        function refreshUI() {
            updateDashboardStats(); renderDashboardTable(); renderFullBuildings(); renderFullTable(); 
            renderBuildings(); if(currentSelectedBuildingId) renderApartments(); if(currentSelectedAptId) renderMonths();
            renderExpenses(); renderAnnouncements(); renderChat();
        }

        // --- AUTH LOGIC ---
        function requireAuth(callback) {
            if(isAuthenticated) { callback(); } else { pendingCallback = callback; document.getElementById('authPwdInput').value = ""; document.getElementById('authModal').style.display = 'flex'; setTimeout(() => document.getElementById('authPwdInput').focus(), 100); }
        }
        function verifyAuth() { const pwd = document.getElementById('authPwdInput').value, savedPwd = localStorage.getItem('syndicPassword') || '0000'; if (pwd === savedPwd) { isAuthenticated = true; updateAuthUI(); closeAuthModal(); if(pendingCallback) { pendingCallback(); pendingCallback = null; } } else { alert(t('alert_wrong_pwd')); } }
        function closeAuthModal() { document.getElementById('authModal').style.display = 'none'; pendingCallback = null; }
        function toggleAuth() { if(isAuthenticated) { isAuthenticated = false; updateAuthUI(); } else { requireAuth(() => {}); } }
        function updateAuthUI() { const btn = document.getElementById('btn-lock'); if(isAuthenticated) { btn.classList.add('unlocked'); btn.innerHTML = `🔓 <span>${t('btn_unlocked')}</span>`; } else { btn.classList.remove('unlocked'); btn.innerHTML = `🔒 <span>${t('btn_locked')}</span>`; } }
        function openChangePwdModal() { document.getElementById('oldPwdInput').value = ""; document.getElementById('newPwdInput').value = ""; document.getElementById('changePwdModal').style.display = 'flex'; setTimeout(() => document.getElementById('oldPwdInput').focus(), 100); }
        function closeChangePwdModal() { document.getElementById('changePwdModal').style.display = 'none'; }
        function saveNewPassword() { const oldP = document.getElementById('oldPwdInput').value, newP = document.getElementById('newPwdInput').value, savedPwd = localStorage.getItem('syndicPassword') || '0000'; if(oldP !== savedPwd) return alert(t('alert_wrong_pwd')); if(!newP) return alert(t('alert_pwd_req')); localStorage.setItem('syndicPassword', newP); alert(t('alert_pwd_changed')); closeChangePwdModal(); }

        // --- LANGUAGE LOGIC ---
        function changeLanguage() { currentLang = document.getElementById('langSelect').value; localStorage.setItem('syndicLang', currentLang); applyLanguage(); }
        function t(key) { return dict[currentLang][key]; }
        function applyLanguage() {
            document.getElementById('langSelect').value = currentLang; document.documentElement.lang = currentLang; document.documentElement.dir = currentLang === 'ar' ? 'rtl' : 'ltr';
            document.querySelectorAll('[data-i18n]').forEach(el => el.innerHTML = t(el.getAttribute('data-i18n'))); document.querySelectorAll('[data-i18n-placeholder]').forEach(el => el.placeholder = t(el.getAttribute('data-i18n-placeholder')));
            populateYears(); updateAuthUI(); refreshUI();
            if (currentSelectedBuildingId && !currentSelectedAptId) { document.getElementById('currentBldName').innerHTML = `🏢 ${data.buildings.find(b=>b.id===currentSelectedBuildingId).name}`; }
            if (currentSelectedAptId) { document.getElementById('currentAptName').innerHTML = `🏠 ${data.apartments.find(a=>a.id===currentSelectedAptId).name}`; renderMonths(); }
        }

        // --- NAVIGATION ---
        function showSection(sectionId) { document.querySelectorAll('.section-box').forEach(s => s.classList.remove('active')); document.getElementById(sectionId).classList.add('active'); }
        function changePage(pageId) {
            document.querySelectorAll('.page').forEach(p => p.classList.remove('active')); document.querySelectorAll('.nav-links button').forEach(b => b.classList.remove('active')); document.getElementById(pageId).classList.add('active'); const navBtn = document.getElementById('btn-' + pageId); if(navBtn) navBtn.classList.add('active');
            if(pageId === 'dashboard') { renderDashboardTable(); updateDashboardStats(); } else if (pageId === 'apartments') { currentSelectedBuildingId = null; currentSelectedAptId = null; showSection('bldListSection'); renderBuildings(); } else if (pageId === 'expenses') { renderExpenses(); } else if (pageId === 'full-payments') { closeFullTableBuilding(); renderFullBuildings(); } else if (pageId === 'communication') { renderAnnouncements(); renderChat(); }
        }
        function populateYears() { const currentYear = new Date().getFullYear(); let yearOptions = ""; for (let y = currentYear - 2; y <= currentYear + 5; y++) { yearOptions += `<option value="${y}" ${y === currentYear ? 'selected' : ''}>${t('year_word')} ${y}</option>`; } document.getElementById('dashboardYear').innerHTML = yearOptions; document.getElementById('paymentYear').innerHTML = yearOptions; document.getElementById('fullYear').innerHTML = yearOptions; }

        // --- UNIFIED DELETE & RENAME LOGIC (تحديث جديد: مسح وتعديل العمارات والشقق) ---
        function openDeleteModal(id, type) {
            requireAuth(() => {
                itemToDeleteId = id; itemToDeleteType = type;
                const msgEl = document.getElementById('deleteMsgText');
                msgEl.innerText = (type === 'building') ? t('msg_del_bld') : t('msg_del_default');
                msgEl.style.color = (type === 'building') ? "var(--danger)" : "#4b5563";
                document.getElementById('deleteModal').style.display = 'flex';
            });
        }
        function closeDeleteModal() { document.getElementById('deleteModal').style.display = 'none'; itemToDeleteId = null; itemToDeleteType = null; }
        
        function confirmDelete() {
            if (itemToDeleteId !== null) {
                if (itemToDeleteType === 'expense') { data.expenses = data.expenses.filter(e => e.id !== itemToDeleteId); renderExpenses(); } 
                else if (itemToDeleteType === 'announcement') { data.announcements = data.announcements.filter(a => a.id !== itemToDeleteId); renderAnnouncements(); } 
                else if (itemToDeleteType === 'chat') { data.chat = data.chat.filter(c => c.id !== itemToDeleteId); renderChat(); }
                else if (itemToDeleteType === 'building') { 
                    data.buildings = data.buildings.filter(b => b.id !== itemToDeleteId); 
                    data.apartments = data.apartments.filter(a => a.buildingId !== itemToDeleteId); // كيمسح الشقق ديال العمارة
                    renderBuildings(); 
                }
                else if (itemToDeleteType === 'apartment') { 
                    data.apartments = data.apartments.filter(a => a.id !== itemToDeleteId); 
                    renderApartments(); 
                }
                saveData(); closeDeleteModal();
            }
        }

        function openRenameModal(id, type, currentName) {
            requireAuth(() => {
                itemToRenameId = id; itemToRenameType = type;
                document.getElementById('renameInput').value = currentName;
                document.getElementById('renameModal').style.display = 'flex';
                setTimeout(() => document.getElementById('renameInput').focus(), 100);
            });
        }
        function closeRenameModal() { document.getElementById('renameModal').style.display = 'none'; itemToRenameId = null; itemToRenameType = null; }
        
        function saveRename() {
            const newName = document.getElementById('renameInput').value.trim();
            if(!newName) return;
            if (itemToRenameType === 'building') {
                const b = data.buildings.find(b => b.id === itemToRenameId);
                if(b) b.name = newName; renderBuildings();
            } else if (itemToRenameType === 'apartment') {
                const a = data.apartments.find(a => a.id === itemToRenameId);
                if(a) a.name = newName; renderApartments();
            }
            saveData(); closeRenameModal();
        }

        // --- Buildings & Apts ---
        function addBuilding() { requireAuth(() => { const n = document.getElementById('newBldName').value.trim(); if(!n) return alert(t('alert_bld')); data.buildings.push({ id: Date.now(), name: n }); saveData(); document.getElementById('newBldName').value = ""; renderBuildings(); }); }
        
        function renderBuildings() { 
            document.getElementById('buildingsGrid').innerHTML = data.buildings.map(b => `
                <div class="item-card" onclick="openBuilding(${b.id}, '${b.name}')">
                    <div class="card-actions">
                        <button class="card-action-btn" style="color:var(--primary);" onclick="event.stopPropagation(); openRenameModal(${b.id}, 'building', '${b.name}')" title="تعديل">✏️</button>
                        <button class="card-action-btn" style="color:var(--danger);" onclick="event.stopPropagation(); openDeleteModal(${b.id}, 'building')" title="مسح">🗑️</button>
                    </div>
                    <div class="icon">🏢</div><div class="name">${b.name}</div>
                </div>`).join(''); 
        }
        
        function openBuilding(id, name) { currentSelectedBuildingId = id; document.getElementById('currentBldName').innerHTML = `🏢 ${name}`; showSection('aptListSection'); renderApartments(); }
        function closeBuilding() { currentSelectedBuildingId = null; showSection('bldListSection'); renderBuildings(); }

        function addApartment() { requireAuth(() => { if(!currentSelectedBuildingId) return; const n = document.getElementById('newAptName').value.trim(); if(!n) return alert(t('alert_apt')); data.apartments.push({ id: Date.now(), buildingId: currentSelectedBuildingId, name: n }); saveData(); document.getElementById('newAptName').value = ""; renderApartments(); }); }
        
        function renderApartments() { 
            document.getElementById('apartmentsGrid').innerHTML = data.apartments.filter(a => a.buildingId === currentSelectedBuildingId).map(a => `
                <div class="item-card" onclick="openAptDetails(${a.id}, '${a.name}')">
                    <div class="card-actions">
                        <button class="card-action-btn" style="color:var(--primary);" onclick="event.stopPropagation(); openRenameModal(${a.id}, 'apartment', '${a.name}')" title="تعديل">✏️</button>
                        <button class="card-action-btn" style="color:var(--danger);" onclick="event.stopPropagation(); openDeleteModal(${a.id}, 'apartment')" title="مسح">🗑️</button>
                    </div>
                    <div class="icon">🏠</div><div class="name">${a.name}</div>
                </div>`).join(''); 
        }
        
        function openAptDetails(id, name) { currentSelectedAptId = id; document.getElementById('currentAptName').innerHTML = `🏠 ${name}`; showSection('aptDetailsSection'); renderMonths(); }
        function closeAptDetails() { currentSelectedAptId = null; showSection('aptListSection'); renderApartments(); }

        // --- Payments ---
        function renderMonths() { if(!currentSelectedAptId) return; const y = document.getElementById('paymentYear').value; let html = ""; for(let m = 1; m <= 12; m++) { const amt = data.payments[`${currentSelectedAptId}_${y}_${m}`] || 0; const isP = amt > 0; html += `<div class="${isP ? 'month-card paid' : 'month-card'}" onclick="openPaymentModal(${m}, '${y}')">${t('months')[m-1]}<br><span style="${!isP?'color:#ef4444;font-size:12px;':''}">${isP ? amt + ' ' + t('curr') : t('unpaid').replace('✖️ ', '')}</span></div>`; } document.getElementById('monthsGrid').innerHTML = html; }
        function openPaymentModal(m, y) { requireAuth(() => { selectedMonthForPay = m; selectedYearForPay = y; document.getElementById('modalTitle').innerText = `${t('pay_title')} ${t('months')[m-1]} - ${y}`; document.getElementById('modalAmountInput').value = data.payments[`${currentSelectedAptId}_${y}_${m}`] || ""; document.getElementById('paymentModal').style.display = 'flex'; setTimeout(() => document.getElementById('modalAmountInput').focus(), 100); }); }
        function closePaymentModal() { document.getElementById('paymentModal').style.display = 'none'; }
        function savePayment() { const amt = parseFloat(document.getElementById('modalAmountInput').value); if (isNaN(amt) || amt <= 0) return alert(t('alert_amt')); data.payments[`${currentSelectedAptId}_${selectedYearForPay}_${selectedMonthForPay}`] = amt; saveData(); renderMonths(); closePaymentModal(); }
        function clearPayment() { delete data.payments[`${currentSelectedAptId}_${selectedYearForPay}_${selectedMonthForPay}`]; saveData(); renderMonths(); closePaymentModal(); }

        // --- Expenses ---
        function addExpense() { requireAuth(() => { const d = document.getElementById('expenseDesc').value, a = parseFloat(document.getElementById('expenseAmount').value), dt = document.getElementById('expenseDate').value; if(!d || isNaN(a) || a <= 0 || !dt) return alert(t('alert_fill')); data.expenses.push({ id: Date.now(), desc: d, amount: a, date: dt }); saveData(); document.getElementById('expenseDesc').value = ""; document.getElementById('expenseAmount').value = ""; renderExpenses(); }); }
        function renderExpenses() { document.getElementById('expensesTableBody').innerHTML =[...data.expenses].sort((a,b) => new Date(b.date) - new Date(a.date)).map(e => `<tr><td style="font-weight:bold; color:#4b5563;">${e.date}</td><td style="text-align:${currentLang==='ar'?'right':'left'}; font-weight:600;">${e.desc}</td><td class="text-danger" style="font-weight:800; font-size:16px;">${e.amount} ${t('curr')}</td><td><button class="btn btn-danger" style="padding: 8px 12px; font-size:12px;" onclick="openDeleteModal(${e.id}, 'expense')">${t('btn_delete')}</button></td></tr>`).join(''); }

        // --- Announcements & Chat ---
        function addAnnouncement() { requireAuth(() => { const title = document.getElementById('announceTitle').value.trim(), msg = document.getElementById('announceMsg').value.trim(); if (!title || !msg) return alert(t('alert_fill')); const dateStr = new Date().toLocaleString(currentLang === 'ar' ? 'ar-MA' : 'fr-FR', { dateStyle: 'medium', timeStyle: 'short' }); data.announcements.unshift({ id: Date.now(), title, msg, date: dateStr }); saveData(); document.getElementById('announceTitle').value = ""; document.getElementById('announceMsg').value = ""; renderAnnouncements(); }); }
        function renderAnnouncements() { const list = document.getElementById('announcementsList'); if (data.announcements.length === 0) { list.innerHTML = `<p style="text-align:center; color:#9ca3af;">لا توجد إعلانات حالياً.</p>`; return; } list.innerHTML = data.announcements.map(ann => `<div class="announce-item"><button class="del-btn" onclick="openDeleteModal(${ann.id}, 'announcement')">✖</button><h4>${ann.title}</h4><span class="date">🕒 ${ann.date}</span><p>${ann.msg.replace(/\n/g, '<br>')}</p></div>`).join(''); }

        function sendChatMessage() { const nameInput = document.getElementById('chatName').value.trim(), msgInput = document.getElementById('chatMsg').value.trim(); if (!msgInput) return; const name = nameInput || (currentLang === 'ar' ? "سكان العمارة" : "Résident"); const timeStr = new Date().toLocaleTimeString(currentLang === 'ar' ? 'ar-MA' : 'fr-FR', { hour: '2-digit', minute:'2-digit' }); data.chat.push({ id: Date.now(), name, msg: msgInput, time: timeStr }); if(data.chat.length > 100) data.chat.shift(); saveData(); document.getElementById('chatMsg').value = ""; renderChat(); }
        function handleChatKeyPress(e) { if(e.key === 'Enter') sendChatMessage(); }
        function renderChat() { const box = document.getElementById('chatBox'); if (data.chat.length === 0) { box.innerHTML = `<p style="text-align:center; color:#9ca3af; margin-top:auto; margin-bottom:auto;">ابدأ الدردشة الآن...</p>`; return; } box.innerHTML = data.chat.map(c => `<div class="chat-msg"><button class="del-chat-btn" onclick="openDeleteModal(${c.id}, 'chat')" title="مسح الرسالة">✖</button><span class="sender">${c.name}</span><p class="text">${c.msg}</p><span class="time">${c.time}</span></div>`).join(''); box.scrollTop = box.scrollHeight; }

        // --- FULL TABLE & DASHBOARD ---
        function updateDashboardStats() { const tin = Object.values(data.payments).reduce((a,b) => a+b, 0), tout = data.expenses.reduce((s, e) => s + e.amount, 0); document.getElementById('totalIncomes').innerText = `${tin.toLocaleString('en-US')} ${t('curr')}`; document.getElementById('totalExpenses').innerText = `${tout.toLocaleString('en-US')} ${t('curr')}`; document.getElementById('boxBalance').innerText = `${(tin - tout).toLocaleString('en-US')} ${t('curr')}`; }
        
        function buildTableHTML(yId, limit, bldId = null) { 
            const y = document.getElementById(yId).value; let headHTML = `<th>${t('apt_month')}</th>`; t('months').forEach(m => headHTML += `<th>${m}</th>`); 
            let apts = data.apartments; if (bldId) apts = apts.filter(a => a.buildingId === bldId); if (limit && apts.length > limit) apts = apts.slice(-limit); 
            if(apts.length === 0) return { headHTML, bodyHTML: `<tr><td colspan="13" style="padding:30px; color:#9ca3af;">${t('no_apts')}</td></tr>` }; 
            let bodyHTML = apts.map(a => { 
                let b = data.buildings.find(bx => bx.id === a.buildingId); 
                let r = `<tr><td><span style="color:#6b7280; font-size:12px;">${b ? `[${b.name}] - ` : ""}</span><br>${a.name}</td>`; 
                for(let m = 1; m <= 12; m++) { const amt = data.payments[`${a.id}_${y}_${m}`]; r += amt > 0 ? `<td><span class="badge badge-paid">${t('paid')} ${amt}</span></td>` : `<td><span class="badge badge-unpaid">${t('unpaid')}</span></td>`; } 
                return r + `</tr>`; 
            }).join(''); return { headHTML, bodyHTML }; 
        }

        function renderDashboardTable() { const { headHTML, bodyHTML } = buildTableHTML('dashboardYear', 10, null); document.getElementById('dashboardTableHead').innerHTML = headHTML; document.getElementById('dashboardTableBody').innerHTML = bodyHTML; }
        
        function renderFullBuildings() { document.getElementById('fullBuildingsGrid').innerHTML = data.buildings.map(b => `<div class="item-card" onclick="openFullTableBuilding(${b.id}, '${b.name}')"><div class="icon">🏢</div><div class="name">${b.name}</div></div>`).join(''); }
        function openFullTableBuilding(id, name) { currentFullBuildingId = id; document.getElementById('currentFullBldName').innerHTML = `🏢 ${name}`; document.getElementById('fullBldListSection').classList.remove('active'); document.getElementById('fullTableSection').classList.add('active'); renderFullTable(); }
        function closeFullTableBuilding() { currentFullBuildingId = null; document.getElementById('fullTableSection').classList.remove('active'); document.getElementById('fullBldListSection').classList.add('active'); }
        function renderFullTable() { if(!currentFullBuildingId) return; const { headHTML, bodyHTML } = buildTableHTML('fullYear', null, currentFullBuildingId); document.getElementById('fullTableHead').innerHTML = headHTML; document.getElementById('fullTableBody').innerHTML = bodyHTML; }

        function initApp() { document.getElementById('expenseDate').valueAsDate = new Date(); applyLanguage(); refreshUI(); }

    </script>
</body>
</html>
