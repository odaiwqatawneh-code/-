<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>المتابعة اليومية</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <style>
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f8fafc; }
    .custom-check {
      appearance: none;
      width: 1.35rem;
      height: 1.35rem;
      border: 2px solid #94a3b8;
      border-radius: 0.35rem;
      cursor: pointer;
      display: inline-flex;
      align-items: center;
      justify-content: center;
      transition: all 0.15s ease;
      background-color: white;
    }
    .custom-check:checked {
      background-color: #10b981;
      border-color: #10b981;
    }
    .custom-check:checked::after {
      content: '✓';
      color: white;
      font-size: 0.9rem;
      font-weight: bold;
    }
    .today-row { background-color: #ecfdf5 !important; }
  </style>
</head>
<body class="text-slate-800 pb-12">

  <!-- الشريط العلوي -->
  <header class="bg-slate-900 text-white shadow-md sticky top-0 z-30">
    <div class="max-w-7xl mx-auto px-4 py-3 flex flex-wrap justify-between items-center gap-3">
      <div class="flex items-center gap-3">
        <h1 class="text-lg font-bold">قائمة المتابعة اليومية</h1>
        <span id="syncStatus" class="text-xs px-2.5 py-1 rounded-full bg-amber-500/20 text-amber-300 border border-amber-400/30">
          حفظ محلي (غير مربوط بالسحابة)
        </span>
      </div>

      <div class="flex items-center gap-2">
        <button onclick="switchView('today')" id="btnToday" class="px-3 py-1.5 rounded-lg text-sm font-medium bg-emerald-600 text-white">
          مهام اليوم (للهاتف)
        </button>
        <button onclick="switchView('table')" id="btnTable" class="px-3 py-1.5 rounded-lg text-sm font-medium bg-slate-800 text-slate-300 hover:bg-slate-700">
          الجدول الكامل
        </button>
        <button onclick="toggleModal('manageModal')" class="px-3 py-1.5 rounded-lg text-sm font-medium bg-slate-700 hover:bg-slate-600 text-white">
          ⚙️ إدارة البنود
        </button>
      </div>
    </div>
  </header>

  <main class="max-w-7xl mx-auto px-4 mt-6">

    <!-- 1. واجهة الهاتف: مهام اليوم المحدد -->
    <section id="todayView" class="max-w-lg mx-auto">
      <div class="bg-white rounded-2xl shadow-sm border border-slate-200 p-5">
        <div class="flex items-center justify-between border-b border-slate-100 pb-4 mb-4">
          <button onclick="changeSelectedDate(-1)" class="p-2 rounded-lg bg-slate-100 hover:bg-slate-200 text-sm font-bold">◀ اليوم السابق</button>
          <div class="text-center">
            <div id="selectedDateLabel" class="text-lg font-bold text-slate-900 dir-ltr">07/11/2026</div>
            <div id="selectedDayProgress" class="text-xs text-emerald-600 font-semibold mt-0.5">الإنجاز: 0%</div>
          </div>
          <button onclick="changeSelectedDate(1)" class="p-2 rounded-lg bg-slate-100 hover:bg-slate-200 text-sm font-bold">اليوم التالي ▶</button>
        </div>

        <div id="todayHabitsList" class="space-y-3"></div>
      </div>
    </section>

    <!-- 2. واجهة الكمبيوتر: الجدول الكامل -->
    <section id="tableView" class="hidden">
      <div class="bg-white rounded-xl shadow-sm border border-slate-200 overflow-x-auto">
        <table class="w-full text-sm text-center border-collapse">
          <thead id="tableHead" class="bg-slate-100 text-slate-800 font-bold border-b border-slate-300 sticky top-0">
          </thead>
          <tbody id="tableBody" class="divide-y divide-slate-200">
          </tbody>
        </table>
      </div>
    </section>

  </main>

  <!-- نافذة إدارة البنود (إضافة / حذف / تعديل) -->
  <div id="manageModal" class="fixed inset-0 bg-black/50 hidden items-center justify-center z-50 p-4">
    <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-xl">
      <div class="flex justify-between items-center border-b pb-3 mb-4">
        <h2 class="text-lg font-bold">إدارة بنود القائمة</h2>
        <button onclick="toggleModal('manageModal')" class="text-slate-400 hover:text-slate-700 text-xl font-bold">&times;</button>
      </div>

      <!-- إضافة بند جديد -->
      <div class="bg-slate-50 p-3 rounded-xl border border-slate-200 mb-4">
        <h3 class="text-xs font-bold text-slate-600 mb-2">إضافة بند جديد</h3>
        <div class="flex gap-2">
          <input id="newHabitName" type="text" placeholder="اسم البند (مثال: مشي)" class="flex-1 border rounded-lg px-3 py-1.5 text-sm" />
          <input id="newHabitCount" type="number" min="1" max="10" value="1" title="عدد التيكات" class="w-20 border rounded-lg px-2 py-1.5 text-sm text-center" />
          <button onclick="addHabit()" class="bg-emerald-600 hover:bg-emerald-700 text-white px-4 py-1.5 rounded-lg text-sm font-bold">إضافة</button>
        </div>
        <p class="text-[11px] text-slate-500 mt-1">الرقم يحدد عدد مربعات الاختيار داخل الخانة (مثلاً 1 للعادة العادية، 5 للسنن).</p>
      </div>

      <!-- قائمة البنود الحالية -->
      <div id="manageHabitsList" class="space-y-2 max-h-64 overflow-y-auto mb-4"></div>

      <!-- إضافة أيام إضافية -->
      <div class="border-t pt-3 flex justify-between items-center">
        <span class="text-xs text-slate-600">تمديد الجدول:</span>
        <button onclick="extendDates(15)" class="text-xs bg-slate-800 text-white px-3 py-1.5 rounded-lg hover:bg-slate-700">
          + إضافة 15 يوماً إضافياً
        </button>
      </div>
    </div>
  </div>

  <script>
    // =========================================================================
    // إعدادات SUPABASE (ضع بيانات مشروعك هنا لتفعيل المزامنة بين الهاتف والكمبيوتر)
    // =========================================================================
    const SUPABASE_URL = "https://fyadgrklguuusgntpvxy.supabase.co"; // مثال: "https://xxxx.supabase.co"
    const SUPABASE_ANON_KEY = "sb_publishable_DIfOKxzSZAhU5TFATLDsBA_e4YSy1kh"; // مفتاح anon public الخاص بك
    const RECORD_ID = "odai_main_tracker";

    let supabaseClient = null;
    if (SUPABASE_URL && SUPABASE_ANON_KEY) {
      supabaseClient = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
    }

    // البنود الافتراضية مطابقة لصورتك تماماً
    const defaultHabits = [
      { id: "morning_vit", name: "فيتامينات الصبح", count: 1 },
      { id: "quran", name: "القرآن", count: 1 },
      { id: "workout", name: "التمرين", count: 1 },
      { id: "creatine", name: "كرياتين", count: 1 },
      { id: "supplements", name: "مكملين", count: 1 },
      { id: "work", name: "الشغل", count: 1 },
      { id: "study", name: "الدراسة", count: 1 },
      { id: "night_vit", name: "فيتامينات المساء", count: 1 },
      { id: "kinship", name: "أرحام", count: 1 },
      { id: "tasbeeh", name: "تسبيح", count: 1 },
      { id: "adhkar", name: "أذكار", count: 1 },
      { id: "sunan", name: "سنن", count: 5 }
    ];

    // توليد التواريخ من 07/11/2026 إلى 24/12/2026
    function generateInitialDates() {
      const dates = [];
      const start = new Date(2026, 10, 7); // 7 Nov 2026
      const end = new Date(2026, 11, 24);  // 24 Dec 2026
      for (let d = new Date(start); d <= end; d.setDate(d.getDate() + 1)) {
        dates.push(formatDate(d));
      }
      return dates;
    }

    function formatDate(dateObj) {
      const dd = String(dateObj.getDate()).padStart(2, '0');
      const mm = String(dateObj.getMonth() + 1).padStart(2, '0');
      const yyyy = dateObj.getFullYear();
      return `${dd}/${mm}/${yyyy}`;
    }

    let state = {
      habits: defaultHabits,
      dates: generateInitialDates(),
      logs: {} // هيكل الحفظ: { "07/11/2026": { "sunan": [true, false, ...], "quran": [true] } }
    };

    let currentSelectedDateIndex = 0;

    // تحميل البيانات
    async function loadState() {
      const localData = localStorage.getItem("habit_tracker_v1");
      if (localData) {
        state = JSON.parse(localData);
      }

      if (supabaseClient) {
        updateSyncBadge("جاري المزامنة...", "blue");
        const { data, error } = await supabaseClient
          .from("habit_tracker_state")
          .select("*")
          .eq("id", RECORD_ID)
          .maybeSingle();

        if (!error && data) {
          state.habits = data.config.habits || state.habits;
          state.dates = data.config.dates || state.dates;
          state.logs = data.logs || state.logs;
          localStorage.setItem("habit_tracker_v1", JSON.stringify(state));
          updateSyncBadge("متصل ومحفوظ سحابياً ✓", "emerald");
        } else if (!error && !data) {
          await saveState();
        } else {
          updateSyncBadge("خطأ في الاتصال بالسحابة", "red");
        }
      }

      const todayStr = formatDate(new Date());
      const idx = state.dates.indexOf(todayStr);
      currentSelectedDateIndex = idx !== -1 ? idx : 0;

      renderAll();
    }

    // حفظ البيانات محلياً وسحابياً
    async function saveState() {
      localStorage.setItem("habit_tracker_v1", JSON.stringify(state));
      if (supabaseClient) {
        updateSyncBadge("جاري الحفظ...", "blue");
        const { error } = await supabaseClient
          .from("habit_tracker_state")
          .upsert({
            id: RECORD_ID,
            config: { habits: state.habits, dates: state.dates },
            logs: state.logs,
            updated_at: new Date().toISOString()
          });
        if (!error) {
          updateSyncBadge("محفوظ سحابياً ✓", "emerald");
        } else {
          updateSyncBadge("تعذر الحفظ السحابي!", "red");
        }
      }
    }

    function updateSyncBadge(text, color) {
      const badge = document.getElementById("syncStatus");
      badge.textContent = text;
      badge.className = `text-xs px-2.5 py-1 rounded-full border border-${color}-400/30 bg-${color}-500/20 text-${color}-300`;
    }

    function getCheckValue(dateStr, habitId, index) {
      return state.logs?.[dateStr]?.[habitId]?.[index] || false;
    }

    function toggleCheck(dateStr, habitId, index) {
      if (!state.logs[dateStr]) state.logs[dateStr] = {};
      if (!state.logs[dateStr][habitId]) state.logs[dateStr][habitId] = [];
      state.logs[dateStr][habitId][index] = !state.logs[dateStr][habitId][index];
      renderAll();
      saveState();
    }

    // عرض واجهة اليوم (للهاتف)
    function renderTodayView() {
      const dateStr = state.dates[currentSelectedDateIndex];
      document.getElementById("selectedDateLabel").textContent = dateStr;

      let totalChecks = 0;
      let completedChecks = 0;

      const listEl = document.getElementById("todayHabitsList");
      listEl.innerHTML = "";

      state.habits.forEach(habit => {
        totalChecks += habit.count;
        let checksHtml = "";
        for (let i = 0; i < habit.count; i++) {
          const isChecked = getCheckValue(dateStr, habit.id, i);
          if (isChecked) completedChecks++;
          checksHtml += `
            <input type="checkbox" class="custom-check"
              ${isChecked ? "checked" : ""}
              onchange="toggleCheck('${dateStr}', '${habit.id}', ${i})" />
          `;
        }

        const item = document.createElement("div");
        item.className = "flex items-center justify-between p-3 rounded-xl bg-slate-50 border border-slate-200";
        item.innerHTML = `
          <span class="font-semibold text-slate-800">${habit.name}</span>
          <div class="flex items-center gap-1.5 flex-wrap justify-end">${checksHtml}</div>
        `;
        listEl.appendChild(item);
      });

      const pct = totalChecks > 0 ? Math.round((completedChecks / totalChecks) * 100) : 0;
      document.getElementById("selectedDayProgress").textContent = `الإنجاز: ${completedChecks} من ${totalChecks} (${pct}%)`;
    }

    // عرض الجدول الكامل (للكمبيوتر)
    function renderTableView() {
      const thead = document.getElementById("tableHead");
      const tbody = document.getElementById("tableBody");

      let headHtml = `<tr><th class="border border-slate-300 px-3 py-2.5 bg-slate-200 sticky right-0 z-10">اليوم</th>`;
      state.habits.forEach(h => {
        headHtml += `<th class="border border-slate-300 px-3 py-2.5 whitespace-nowrap">${h.name}</th>`;
      });
      headHtml += `</tr>`;
      thead.innerHTML = headHtml;

      const todayStr = formatDate(new Date());
      let bodyHtml = "";

      state.dates.forEach(dateStr => {
        const isToday = dateStr === todayStr;
        bodyHtml += `<tr class="${isToday ? 'today-row' : 'hover:bg-slate-50'}">`;
        bodyHtml += `<td class="border border-slate-300 px-3 py-2 font-semibold text-slate-900 bg-slate-50 sticky right-0">${dateStr}</td>`;

        state.habits.forEach(h => {
          let checks = "";
          for (let i = 0; i < h.count; i++) {
            const isChecked = getCheckValue(dateStr, h.id, i);
            checks += `
              <input type="checkbox" class="custom-check"
                ${isChecked ? "checked" : ""}
                onchange="toggleCheck('${dateStr}', '${h.id}', ${i})" />
            `;
          }
          bodyHtml += `<td class="border border-slate-300 px-2 py-1.5"><div class="flex items-center justify-center gap-1">${checks}</div></td>`;
        });

        bodyHtml += `</tr>`;
      });

      tbody.innerHTML = bodyHtml;
    }

    // إدارة البنود (إضافة / حذف)
    function renderManageList() {
      const container = document.getElementById("manageHabitsList");
      container.innerHTML = "";
      state.habits.forEach((h, idx) => {
        const div = document.createElement("div");
        div.className = "flex items-center justify-between bg-slate-50 px-3 py-2 rounded-lg border text-sm";
        div.innerHTML = `
          <div>
            <span class="font-bold">${h.name}</span>
            <span class="text-xs text-slate-500 mr-2">(${h.count} تيك)</span>
          </div>
          <button onclick="removeHabit(${idx})" class="text-red-600 hover:text-red-800 text-xs font-bold">حذف</button>
        `;
        container.appendChild(div);
      });
    }

    function addHabit() {
      const nameInput = document.getElementById("newHabitName");
      const countInput = document.getElementById("newHabitCount");
      const name = nameInput.value.trim();
      const count = parseInt(countInput.value) || 1;
      if (!name) return;

      state.habits.push({
        id: "habit_" + Date.now(),
        name: name,
        count: Math.min(Math.max(count, 1), 10)
      });
      nameInput.value = "";
      countInput.value = 1;
      renderAll();
      saveState();
    }

    function removeHabit(index) {
      state.habits.splice(index, 1);
      renderAll();
      saveState();
    }

    function extendDates(daysCount) {
      const lastDateStr = state.dates[state.dates.length - 1];
      const [dd, mm, yyyy] = lastDateStr.split("/").map(Number);
      const d = new Date(yyyy, mm - 1, dd);
      for (let i = 0; i < daysCount; i++) {
        d.setDate(d.getDate() + 1);
        state.dates.push(formatDate(d));
      }
      renderAll();
      saveState();
    }

    function changeSelectedDate(step) {
      const next = currentSelectedDateIndex + step;
      if (next >= 0 && next < state.dates.length) {
        currentSelectedDateIndex = next;
        renderTodayView();
      }
    }

    function switchView(view) {
      const isToday = view === "today";
      document.getElementById("todayView").classList.toggle("hidden", !isToday);
      document.getElementById("tableView").classList.toggle("hidden", isToday);
      document.getElementById("btnToday").className = isToday
        ? "px-3 py-1.5 rounded-lg text-sm font-medium bg-emerald-600 text-white"
        : "px-3 py-1.5 rounded-lg text-sm font-medium bg-slate-800 text-slate-300";
      document.getElementById("btnTable").className = !isToday
        ? "px-3 py-1.5 rounded-lg text-sm font-medium bg-emerald-600 text-white"
        : "px-3 py-1.5 rounded-lg text-sm font-medium bg-slate-800 text-slate-300";
    }

    function toggleModal(id) {
      const el = document.getElementById(id);
      el.classList.toggle("hidden");
      el.classList.toggle("flex");
    }

    function renderAll() {
      renderTodayView();
      renderTableView();
      renderManageList();
    }

    loadState();
  </script>
</body>
</html>
