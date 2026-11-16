<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, user-scalable=no">
<title>Крипто-копилка</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<style>
:root{--bg:#0e1116;--card:#171b23;--card2:#1e2430;--tx:#eef1f6;--mut:#8b94a7;--ac:#4f8cff;--ac2:#7c5cff;--ok:#2ecc8f;--bad:#ff5d6c;--warn:#ffb347;--r:18px}
@media (prefers-color-scheme:light){:root{--bg:#f2f4f8;--card:#fff;--card2:#eef1f7;--tx:#141821;--mut:#69728a}}
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
body{margin:0;background:var(--bg);color:var(--tx);font:15px/1.45 -apple-system,BlinkMacSystemFont,"SF Pro Text","Segoe UI",Roboto,sans-serif;padding-bottom:calc(80px + env(safe-area-inset-bottom))}
header{padding:calc(18px + env(safe-area-inset-top)) 18px 6px;display:flex;justify-content:space-between;align-items:center}
h1{font-size:22px;margin:0}h2{font-size:17px;margin:22px 18px 10px}
.sub{color:var(--mut);font-size:13px}
.card{background:var(--card);border-radius:var(--r);padding:16px;margin:10px 14px}
.hero{background:linear-gradient(135deg,var(--ac),var(--ac2));color:#fff}
.hero .sub{color:#e5e9ff}.big{font-size:34px;font-weight:700;letter-spacing:-.5px}
.row{display:flex;justify-content:space-between;align-items:center;gap:10px}
.pill{display:inline-block;padding:3px 10px;border-radius:99px;font-size:12px;font-weight:600;background:var(--card2)}
.up{color:var(--ok)}.down{color:var(--bad)}
.bar{height:10px;border-radius:99px;background:var(--card2);overflow:hidden;margin-top:8px}.bar>i{display:block;height:100%;background:linear-gradient(90deg,var(--ac),var(--ac2));border-radius:99px;transition:width .5s}
.note{background:var(--card2);border-radius:12px;padding:10px 12px;margin-top:10px;color:var(--mut);font-size:13px}
.note b{color:var(--tx)}
nav{position:fixed;bottom:0;left:0;right:0;display:flex;background:var(--card);padding:8px 6px calc(8px + env(safe-area-inset-bottom));border-top:1px solid var(--card2)}
nav button{flex:1;background:none;border:0;color:var(--mut);font-size:11px;padding:4px 0;cursor:pointer}nav button span{display:block;font-size:20px}nav button.on{color:var(--ac)}
section{display:none}section.on{display:block}
input,select{width:100%;background:var(--card2);border:1px solid transparent;color:var(--tx);border-radius:12px;padding:12px;font-size:16px;margin:6px 0}
input:focus{outline:none;border-color:var(--ac)}
label{display:block;font-size:12px;color:var(--mut);margin-top:8px}
.btn{width:100%;border:0;border-radius:14px;padding:14px;font-size:16px;font-weight:600;color:#fff;background:linear-gradient(135deg,var(--ac),var(--ac2));margin-top:8px;cursor:pointer}
.btn.ghost{background:var(--card2);color:var(--tx)}
.step{display:flex;gap:12px;align-items:center;padding:10px 0;border-bottom:1px solid var(--card2)}.step:last-child{border:0}
.dot{width:28px;height:28px;border-radius:50%;background:var(--card2);display:flex;align-items:center;justify-content:center;font-size:13px;flex:none}.dot.done{background:var(--ok);color:#fff}
details{background:var(--card);border-radius:14px;margin:8px 14px;padding:12px 14px}summary{font-weight:600;cursor:pointer}details p{margin:8px 0 0;color:var(--mut)}
svg.spark{width:100%;height:60px;margin-top:8px}
.warn{border:1px solid var(--warn)}.err{color:var(--bad);font-size:13px}
</style>
</head>
<body>
<header><div><h1>🪙 Копилка</h1><div class="sub" id="sub">Загрузка…</div></div><span class="pill" id="rateTag" onclick="go('tools')">курс не задан</span></header>

<section id="home" class="on">
  <div class="card hero"><div class="sub">Стоимость портфеля</div><div class="big" id="val">—</div>
    <div class="row" style="margin-top:6px"><span id="pnl">Добавьте первую покупку</span><span class="sub" id="inv"></span></div></div>
  <div class="card"><div class="row"><b>План вложений</b><span class="pill" id="planTag">0 / 240 BYN</span></div><div class="bar"><i id="planBar" style="width:0"></i></div>
    <div class="note" id="nextBuy">Следующая покупка: —</div></div>
  <div class="card"><b>➕ Записать покупку</b>
    <label>Монета</label><select id="bCoin"><option>BTC</option><option>ETH</option></select>
    <label>Потрачено, BYN</label><input id="bByn" type="number" inputmode="decimal" placeholder="например 42">
    <label>Получено монет (из Whitebird)</label><input id="bQty" type="number" inputmode="decimal" placeholder="например 0.00031">
    <button class="btn" onclick="addBuy()">Записать</button><div class="err" id="bErr"></div></div>
  <div class="card" id="logCard" style="display:none"><b>История покупок</b><div id="log"></div></div>
  <div class="card warn"><b>⚠️ Помните</b><div class="sub">Крипта может упасть на 30–50%. Не вкладывайте то, что нельзя потерять. Прибыль не гарантирована.</div></div>
</section>

<section id="market">
  <div id="mk"></div>
</section>

<section id="plan">
  <div class="card"><b>1️⃣ Основа: 4 покупки по неделям</b><div class="sub">BTC 70% / ETH 30% от каждой покупки</div><div id="steps"></div></div>
  <div class="card"><b>2️⃣ Резерв для просадок</b>
    <div class="note">Цена −20% от месячного максимума → вложить <b>30 BYN</b><br>Цена −30% → вложить ещё <b>30 BYN</b></div>
    <div class="sub" style="margin-top:8px">Резерв нужен, чтобы докупить дешевле, а не паниковать. Это правило, а не попытка угадать дно.</div></div>
  <div class="card"><b>3️⃣ Сначала пробная покупка 10–15 BYN</b><div class="sub">Чтобы увидеть реальные комиссии и спред Whitebird до основных покупок.</div></div>
  <div class="card"><b>4️⃣ Не продавайте на эмоциях</b><div class="sub">Падения — нормальная часть крипторынка. Решение о продаже принимайте заранее, а не в панике.</div></div>
</section>

<section id="tools">
  <div class="card"><b>💱 Курс и настройки</b><label>Курс: сколько BYN за 1 USD</label><input id="rate" type="number" inputmode="decimal" placeholder="например 3.05">
    <label>Депозит, BYN</label><input id="dep" type="number" inputmode="decimal"><button class="btn" onclick="saveSet()">Сохранить</button></div>
  <div class="card"><b>🧮 Сколько съедят комиссии</b>
    <label>Комиссия + спред Whitebird на одну сторону, %</label><input id="fee" type="number" inputmode="decimal" value="0.5">
    <label>Сумма покупки, BYN</label><input id="feeAmt" type="number" inputmode="decimal" value="60">
    <div class="note" id="feeOut"></div></div>
  <div class="card"><b>🔎 Цена Whitebird против рынка</b>
    <label>Монета</label><select id="wCoin"><option>BTC</option><option>ETH</option></select>
    <label>Цена в Whitebird, BYN</label><input id="wPrice" type="number" inputmode="decimal">
    <div class="note" id="wOut">Введите цену из кабинета</div></div>
</section>

<section id="learn">
  <h2>🎓 Простыми словами</h2><div id="gl"></div>
</section>

<nav>
  <button class="on" onclick="go('home',this)"><span>🏠</span>Главная</button>
  <button onclick="go('market',this)"><span>🌡</span>Рынок</button>
  <button onclick="go('plan',this)"><span>📋</span>План</button>
  <button onclick="go('tools',this)"><span>🧮</span>Инструменты</button>
  <button onclick="go('learn',this)"><span>🎓</span>Учёба</button>
</nav>

<script>
"use strict";
const tg = window.Telegram && window.Telegram.WebApp;
if (tg) { try { tg.ready(); tg.expand(); } catch (e) {} }
const COINS = ["BTC", "ETH"], TR = 60, CORE = 240;
let S = { rate: null, dep: 300, buys: [], t0: Date.now() };
let px = {}, daily = {};

/* ---------- хранилище: Telegram CloudStorage + localStorage ---------- */
function loadState() {
  return new Promise(res => {
    let local = null; try { local = JSON.parse(localStorage.getItem("kopilka") || "null"); } catch (e) {}
    if (tg && tg.CloudStorage) {
      try { tg.CloudStorage.getItem("kopilka", (err, v) => { try { res(v ? JSON.parse(v) : local); } catch (e) { res(local); } }); return; } catch (e) {}
    }
    res(local);
  });
}
function saveState() {
  S.buys = S.buys.slice(-40);
  const j = JSON.stringify(S);
  try { localStorage.setItem("kopilka", j); } catch (e) {}
  if (tg && tg.CloudStorage && j.length < 4000) { try { tg.CloudStorage.setItem("kopilka", j, () => {}); } catch (e) {} }
}

/* ---------- данные рынка (публичные API, запасной источник Bybit) ---------- */
async function getJSON(u) { const r = await fetch(u); if (!r.ok) throw new Error(r.status); return r.json(); }
async function loadMarket() {
  let okAny = false;
  for (const c of COINS) {
    try {
      let k;
      try { k = (await getJSON(`https://data-api.binance.vision/api/v3/klines?symbol=${c}USDT&interval=1d&limit=210`)).map(x => ({ h: +x[2], c: +x[4] })); }
      catch (e) { const d = await getJSON(`https://api.bybit.com/v5/market/kline?category=spot&symbol=${c}USDT&interval=D&limit=210`); k = d.result.list.reverse().map(x => ({ h: +x[2], c: +x[4] })); }
      let p;
      try { p = +(await getJSON(`https://data-api.binance.vision/api/v3/ticker/price?symbol=${c}USDT`)).price; } catch (e) { p = k[k.length - 1].c; }
      daily[c] = k.slice(0, -1); px[c] = p; okAny = true;
    } catch (e) { px[c] = null; }
  }
  $("sub").textContent = okAny ? "Данные биржи обновлены " + new Date().toLocaleTimeString("ru-RU", { hour: "2-digit", minute: "2-digit" }) : "Нет связи с биржей (возможна блокировка у провайдера)";
}
function mstate(c) {
  const d = daily[c], p = px[c]; if (!d || !p) return null;
  const cl = d.map(x => x.c), peak = Math.max(p, ...cl.slice(-30)), dd = p / peak - 1;
  const sl = cl.slice(-200), sma = sl.reduce((a, b) => a + b, 0) / sl.length, v200 = sl.length >= 200 ? p / sma - 1 : null;
  let l = "НОРМА", e = "🟩", t = "Рынок спокойный. Действуйте по плану, ничего лишнего не делайте.";
  if (dd <= -0.30) { l = "ПАНИКА"; e = "🟥"; t = "Цена упала на 30%+. По плану можно вложить вторую часть резерва (30 BYN). Не больше."; }
  else if (dd <= -0.20) { l = "СИЛЬНАЯ ПРОСАДКА"; e = "🟧"; t = "Цена упала на 20%+. По плану можно вложить первую часть резерва (30 BYN)."; }
  else if (v200 !== null && v200 > 0.4) { l = "ПЕРЕГРЕВ"; e = "🟨"; t = "Цена сильно выше своей средней за 200 дней. Плановые покупки можно делать, но сверх плана не добавляйте."; }
  return { dd, v200, l, e, t, cl };
}

/* ---------- утилиты ---------- */
const $ = id => document.getElementById(id);
const fmt = (n, d = 0) => n == null || isNaN(n) ? "—" : n.toLocaleString("ru-RU", { maximumFractionDigits: d, minimumFractionDigits: d });
const esc = s => String(s).replace(/[&<>"']/g, m => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" }[m]));
function go(id, btn) {
  document.querySelectorAll("section").forEach(s => s.classList.toggle("on", s.id === id));
  document.querySelectorAll("nav button").forEach((b, i) => b.classList.toggle("on", b.getAttribute("onclick").includes("'" + id + "'")));
  window.scrollTo(0, 0);
}
function spark(cl) {
  const a = cl.slice(-90), mn = Math.min(...a), mx = Math.max(...a), w = 300, h = 60;
  const pts = a.map((v, i) => `${(i / (a.length - 1) * w).toFixed(1)},${(h - (v - mn) / (mx - mn || 1) * (h - 6) - 3).toFixed(1)}`).join(" ");
  const up = a[a.length - 1] >= a[0];
  return `<svg class="spark" viewBox="0 0 ${w} ${h}" preserveAspectRatio="none"><polyline fill="none" stroke="${up ? "#2ecc8f" : "#ff5d6c"}" stroke-width="2.2" stroke-linejoin="round" points="${pts}"/></svg>`;
}

/* ---------- отрисовка ---------- */
function render() {
  $("rateTag").textContent = S.rate ? `1$ = ${S.rate} BYN` : "задайте курс";
  const inv = S.buys.reduce((a, b) => a + b.byn, 0);
  let val = 0, known = S.rate && S.buys.length;
  for (const b of S.buys) { if (px[b.coin] && S.rate) val += b.qty * px[b.coin] * S.rate; else known = false; }
  $("val").textContent = known ? fmt(val) + " BYN" : (inv ? fmt(inv) + " BYN вложено" : "0 BYN");
  if (known) { const p = val / inv - 1; $("pnl").innerHTML = `<b class="${p >= 0 ? "up" : "down"}" style="color:#fff">${p >= 0 ? "▲" : "▼"} ${fmt(p * 100, 1)}%</b>`; }
  else $("pnl").textContent = S.buys.length ? (S.rate ? "Нет данных рынка" : "Задайте курс в «Инструментах»") : "Добавьте первую покупку";
  $("inv").textContent = inv ? `вложено ${fmt(inv)} BYN` : "";
  const core = Math.min(inv, CORE); $("planTag").textContent = `${fmt(inv)} / ${CORE} BYN`; $("planBar").style.width = Math.min(100, inv / CORE * 100) + "%";
  const done = Math.min(4, Math.floor(inv / (TR * 0.9)));
  if (done >= 4) $("nextBuy").innerHTML = "✅ Основа вложена. Дальше: <b>резерв ждёт просадки</b>.";
  else { const first = S.buys[0] ? S.buys[0].t : Date.now(); const due = new Date(first + done * 7 * 864e5);
    $("nextBuy").innerHTML = `Следующая покупка №${done + 1}: <b>${due.toLocaleDateString("ru-RU", { day: "numeric", month: "long" })}</b> — BTC ≈ 42 BYN, ETH ≈ 18 BYN`; }
  $("logCard").style.display = S.buys.length ? "block" : "none";
  $("log").innerHTML = S.buys.slice().reverse().slice(0, 8).map(b => `<div class="step"><div class="dot done">✓</div><div style="flex:1">${esc(b.coin)} · ${fmt(b.byn)} BYN<div class="sub">${new Date(b.t).toLocaleDateString("ru-RU")} · ${esc(b.qty)} шт</div></div></div>`).join("");
  $("steps").innerHTML = [1, 2, 3, 4].map(i => `<div class="step"><div class="dot ${i <= done ? "done" : ""}">${i <= done ? "✓" : i}</div><div style="flex:1">Покупка №${i} · ${TR} BYN<div class="sub">BTC ≈ 42, ETH ≈ 18 BYN</div></div></div>`).join("");
  $("mk").innerHTML = COINS.map(c => { const m = mstate(c), p = px[c];
    if (!m) return `<div class="card"><b>${c}</b><div class="err">Нет данных. Проверьте интернет или доступ к бирже.</div></div>`;
    return `<div class="card"><div class="row"><b>${c === "BTC" ? "🟠" : "🔷"} ${c}</b><span class="pill">${m.e} ${m.l}</span></div>
      <div class="big" style="font-size:26px">${fmt(p, 2)} $</div><div class="sub">${S.rate ? "≈ " + fmt(p * S.rate) + " BYN" : "задайте курс для пересчёта в BYN"}</div>${spark(m.cl.concat([p]))}
      <div class="row sub"><span>От максимума 30 дней: <b class="${m.dd < 0 ? "down" : "up"}">${fmt(m.dd * 100, 1)}%</b></span><span>${m.v200 !== null ? "От средней 200д: " + fmt(m.v200 * 100) + "%" : ""}</span></div>
      <div class="note">💬 <b>Простыми словами:</b> ${esc(m.t)}</div></div>`; }).join("") + `<div class="sub" style="margin:0 18px">Это не прогноз и не сигнал «купить/продать». Это «термометр» для вашего плана.</div>`;
  calcFee(); calcWb();
  $("rate").value = S.rate || ""; $("dep").value = S.dep || 300;
}
function calcFee() { const f = +$("fee").value || 0, a = +$("feeAmt").value || 0, loss = a * f / 100 * 2, be = ((1 + f / 100) / (1 - f / 100) - 1) * 100;
  $("feeOut").innerHTML = `Круг «купил → продал» стоит ≈ <b>${fmt(loss, 2)} BYN</b>. Чтобы выйти в ноль, цена должна вырасти минимум на <b>${fmt(be, 2)}%</b>.<br><span class="sub">Чем чаще торгуете, тем больше комиссий. Поэтому наш план — редкие покупки.</span>`; }
function calcWb() { const c = $("wCoin").value, w = +$("wPrice").value;
  if (!w) { $("wOut").textContent = "Введите цену из кабинета"; return; }
  if (!S.rate || !px[c]) { $("wOut").textContent = "Нужны курс BYN/USD и данные рынка."; return; }
  const ref = px[c] * S.rate, p = (w / ref - 1) * 100;
  $("wOut").innerHTML = `Рынок ≈ <b>${fmt(ref)} BYN</b>, у вас <b>${fmt(w)}</b> → разница <b class="${p > 1.5 ? "down" : "up"}">${fmt(p, 2)}%</b><br>${p > 1.5 ? "Whitebird заметно дороже рынка — для покупки невыгодно." : p > -1.5 ? "Цена близка к рыночной — нормально." : "Whitebird дешевле рынка. Проверьте актуальность цены."}`; }
function addBuy() {
  const coin = $("bCoin").value, byn = parseFloat($("bByn").value), qty = parseFloat($("bQty").value);
  if (!(byn > 0) || !(qty > 0)) { $("bErr").textContent = "Введите сумму и количество монет (больше нуля)."; return; }
  $("bErr").textContent = ""; S.buys.push({ coin, byn, qty, t: Date.now() }); saveState(); render();
  $("bByn").value = ""; $("bQty").value = "";
  if (tg && !(tg.initDataUnsafe && tg.initDataUnsafe.query_id)) { try { tg.sendData(JSON.stringify({ type: "bought", coin, byn, qty })); return; } catch (e) {} }
  alert("Записано в приложении ✅\nЧтобы бот тоже учёл покупку, отправьте ему:\n/bought " + coin + " " + byn + " " + qty);
}
function saveSet() { const r = parseFloat($("rate").value), d = parseFloat($("dep").value); if (r > 0) S.rate = r; if (d > 0) S.dep = d; saveState(); render(); }
const GL = [["Лонг", "Покупка в расчёте на рост цены."], ["Стоп-лосс", "Заранее выбранная цена, на которой вы продаёте, чтобы убыток не стал большим."], ["DCA", "Покупка равными частями через равные промежутки времени: защита от входа на самой вершине."],
  ["Просадка", "Насколько цена упала от своего недавнего максимума."], ["Тренд", "Общее направление цены: вверх, вниз или вбок."], ["Спред", "Разница между ценой покупки и продажи. Это скрытая плата за сделку."],
  ["Волатильность", "Насколько сильно цена скачет. В крипте она очень высокая."], ["Ликвидность", "Насколько легко купить или продать без сильного влияния на цену. У BTC и ETH она высокая."],
  ["FOMO", "Страх упустить: покупка на эмоциях после сильного роста, обычно у вершины."], ["Бычий / медвежий рынок", "Бычий — рынок растёт, медвежий — падает."], ["Ордер", "Заявка на покупку или продажу: рыночная — по текущей цене, лимитная — по вашей."]];
$("gl").innerHTML = GL.map(g => `<details><summary>${esc(g[0])}</summary><p>💬 ${esc(g[1])}</p></details>`).join("");
["fee", "feeAmt"].forEach(i => $(i).addEventListener("input", calcFee)); $("wCoin").addEventListener("change", calcWb); $("wPrice").addEventListener("input", calcWb);

(async function () {
  const s = await loadState(); if (s) S = Object.assign(S, s);
  render(); await loadMarket(); render();
  setInterval(async () => { await loadMarket(); render(); }, 60000);
})();
</script>
</body>
</html>
