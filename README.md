<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Календарь смен с часами</title>
<style>
body { margin:0; font-family:-apple-system, BlinkMacSystemFont, sans-serif; background:#f4f6fb; }
header { background: linear-gradient(135deg,#667eea,#764ba2); color:white; padding:20px; text-align:center; font-size:20px; font-weight:bold; }
.month-picker { text-align:center; padding:10px; }
input[type="month"] { padding:8px; font-size:16px; }
.calendar { display:grid; grid-template-columns:repeat(7,1fr); gap:8px; padding:10px; }
.day { background:white; border-radius:12px; padding:15px 0; text-align:center; font-weight:bold; box-shadow:0 2px 5px rgba(0,0,0,0.08); cursor:pointer; transition:.2s; }
.day:active { transform:scale(0.95); }
.shift-day { background:#4facfe; color:white; }
.shift-night { background:#2c3e50; color:white; }
.shift-off { background:#27ae60; color:white; }
.weekday { text-align:center; font-size:12px; color:#777; }

.modal { position: fixed; bottom: -300px; left:0; right:0; background:white; border-radius:20px 20px 0 0; padding:20px; box-shadow:0 -5px 20px rgba(0,0,0,0.2); transition:.3s; }
.modal.active { bottom:0; }
.modal button { width:100%; padding:15px; margin:8px 0; font-size:16px; border:none; border-radius:12px; color:white; }

.btn-day { background:#4facfe; }
.btn-night { background:#2c3e50; }
.btn-off { background:#27ae60; }
.btn-clear { background:#e74c3c; }

.summary { max-width:600px; margin:10px auto; background:white; border-radius:12px; padding:15px; box-shadow:0 2px 5px rgba(0,0,0,0.1); font-size:16px; }
</style>
</head>
<body>

<header>📅 Календарь смен с часами</header>

<div class="month-picker">
    <input type="month" id="monthPicker">
</div>

<div class="calendar" id="calendar"></div>



<div style="text-align:center; margin-bottom:10px;">
    <input 
        type="number" 
        id="hourRate" 
        placeholder="Введите ставку (в час)"
        step="0.1"
        style="padding:8px; font-size:16px; width:200px; border-radius:8px; border:1px solid #ccc;"
    >
</div>


<div class="modal" id="modal">
    <button class="btn-day">🌞 Первая смена (8 ч)</button>
    <button class="btn-night">🌙 Вторая смена (18 ч)</button>
    <button class="btn-off">🏖 Выходной (0 ч)</button>
    <button class="btn-clear">❌ Очистить</button>
</div>

<div class="summary" id="summary"></div>

<script>
const calendar = document.getElementById("calendar");
const monthPicker = document.getElementById("monthPicker");
const modal = document.getElementById("modal");
const summaryDiv = document.getElementById("summary");

let selectedDateKey = null;

// Настройка часов смен
const shiftHours = {
    "shift-day": 8,
    "shift-night": 8,
    "shift-off": 0
};

monthPicker.value = new Date().toISOString().slice(0,7);
monthPicker.addEventListener("change", generateCalendar);

function generateCalendar() {
    calendar.innerHTML = "";

    const [year, month] = monthPicker.value.split("-");
    const firstDay = new Date(year, month-1, 1);
    const lastDay = new Date(year, month, 0);

    const daysOfWeek = ["Пн","Вт","Ср","Чт","Пт","Сб","Вс"];
    daysOfWeek.forEach(day => {
        const el = document.createElement("div");
        el.textContent = day;
        el.classList.add("weekday");
        calendar.appendChild(el);
    });

    let startDay = firstDay.getDay();
    if (startDay === 0) startDay = 7;

    for (let i=1;i<startDay;i++) calendar.appendChild(document.createElement("div"));

    for (let day=1; day<=lastDay.getDate(); day++) {
        const dateKey = `${year}-${month}-${day}`;
        const dayDiv = document.createElement("div");
        dayDiv.textContent = day;
        dayDiv.classList.add("day");

        const savedShift = localStorage.getItem(dateKey);
        if (savedShift) dayDiv.classList.add(savedShift);

        dayDiv.addEventListener("click", () => {
            selectedDateKey = dateKey;
            modal.classList.add("active");
        });

        calendar.appendChild(dayDiv);
    }

    updateSummary(year, month);
}

const hourRateInput = document.getElementById("hourRate");

// Загружаем сохранённую ставку
hourRateInput.value = localStorage.getItem("hourRate") || "";

hourRateInput.addEventListener("input", () => {
    localStorage.setItem("hourRate", hourRateInput.value);
    generateCalendar();
});


document.querySelector(".btn-day").onclick = () => saveShift("shift-day");
document.querySelector(".btn-night").onclick = () => saveShift("shift-night");
document.querySelector(".btn-off").onclick = () => saveShift("shift-off");
document.querySelector(".btn-clear").onclick = () => {
    localStorage.removeItem(selectedDateKey);
    modal.classList.remove("active");
    generateCalendar();
};

function saveShift(type) {
    localStorage.setItem(selectedDateKey, type);
    modal.classList.remove("active");
    generateCalendar();
}

function updateSummary(year, month) {
    let totalHours = 0;
    let dayCount = 0, nightCount = 0, offCount = 0;

    const lastDay = new Date(year, month, 0).getDate();

    for (let day = 1; day <= lastDay; day++) {
        const key = `${year}-${month}-${day}`;
        const shift = localStorage.getItem(key);

        if (shift) {
            totalHours += shiftHours[shift];

            if (shift === "shift-day") dayCount++;
            else if (shift === "shift-night") nightCount++;
            else if (shift === "shift-off") offCount++;
        }
    }

    // 👉 Получаем ставку
    const rate = parseFloat(localStorage.getItem("hourRate")) || 0;

    // 👉 Считаем зарплату
    const salary = totalHours * rate;

    summaryDiv.innerHTML = `
        <h3>📊 Статистика за месяц</h3>
        <div style="display:flex; justify-content:space-around; text-align:center; flex-wrap:wrap;">
            <div style="background:#4facfe; color:white; padding:10px 15px; border-radius:10px; width:30%; margin:5px 0;">
                🌞<br>
                Первая смена : ${dayCount}<br>
                ${dayCount * shiftHours["shift-day"]} ч
            </div>
            <div style="background:#2c3e50; color:white; padding:10px 15px; border-radius:10px; width:30%; margin:5px 0;">
                🌙<br>
                Вторая смена: ${nightCount}<br>
                ${nightCount * shiftHours["shift-night"]} ч
            </div>
            <div style="background:#27ae60; color:white; padding:10px 15px; border-radius:10px; width:30%; margin:5px 0;">
                🏖<br>
                Выходные: ${offCount}<br>
                ${offCount * shiftHours["shift-off"]} ч
            </div>
        </div>

        <h4 style="margin-top:10px;">
            Общее количество часов: ${totalHours} ч
        </h4>

        <h3 style="margin-top:10px; color:#27ae60;">
            💰 Зарплата: ${salary.toFixed(2)}
        </h3>
    `;
}


</script>

</body>
</html>
