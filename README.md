<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>مختبر تحليل التفكير</title>

<style>
:root{
  --bg:#f4f6f8;
  --card:#ffffff;
  --text:#222;
  --accent:#2d89ef;
  --muted:#8a94a6;
}

.dark{
  --bg:#1e1e1e;
  --card:#2a2a2a;
  --text:#eee;
}

body{
  font-family: Arial;
  background: var(--bg);
  color: var(--text);
  text-align:center;
  margin:40px;
  transition:.3s;
}

.container{
  background: var(--card);
  padding:25px;
  border-radius:14px;
  width:560px;
  max-width:95%;
  margin:auto;
  box-shadow:0 8px 20px rgba(0,0,0,0.08);
}

textarea{
  width:100%;
  height:100px;
  padding:10px;
  margin:6px 0;
  border-radius:8px;
  border:1px solid #ddd;
  background:transparent;
  color:var(--text);
}

button{
  padding:10px 18px;
  margin:6px;
  border:0;
  border-radius:8px;
  cursor:pointer;
  background:var(--accent);
  color:white;
}

button.secondary{
  background:#555;
}

canvas{
  margin-top:10px;
}

.history{
  text-align:right;
  margin-top:15px;
}

.item{
  padding:8px 10px;
  margin:6px 0;
  border-radius:8px;
  background:rgba(0,0,0,0.05);
}

.dark .item{
  background:rgba(255,255,255,0.08);
}

.small{
  font-size:14px;
  color:var(--muted);
  margin-top:10px;
}
</style>
</head>

<body>

<div class="container">

<h2>🧠 مختبر تحليل التفكير</h2>

<textarea id="text" placeholder="اكتب قرارًا أو موقفًا..."></textarea>

<button onclick="analyze()">تحليل</button>
<button class="secondary" onclick="toggleTheme()">🌙 الوضع</button>
<button class="secondary" onclick="clearData()">مسح البيانات</button>
<button class="secondary" onclick="window.print()">حفظ PDF</button>

<canvas id="chart" width="420" height="300"></canvas>

<div id="summary" class="small"></div>

<div class="history" id="history"></div>

<div class="small" id="fingerprint"></div>

</div>

<script>
const WORDS={
logic:["تحليل","منطق","سبب","نتيجة","خطة","دراسة","احتمال","تقييم"],
emotion:["أشعر","قلق","خائف","سعيد","حزين","توتر","راحة","حب"],
risk:["مغامرة","تجربة","جديد","فرصة","تحدي","جرأة"],
caution:["مخاطرة","أمان","ضمان","حذر","تردد","تفكير"]
};

function normalize(t){
return t
.replace(/[إأآا]/g,"ا")
.replace(/ى/g,"ي")
.replace(/ة/g,"ه")
.toLowerCase();
}

function count(text, list){
let n=0;
list.forEach(w=>{
const re=new RegExp(normalize(w),"g");
const m=text.match(re);
if(m) n+=m.length;
});
return n;
}

function analyzeText(text){
text=normalize(text);
let s={
logic:count(text,WORDS.logic),
emotion:count(text,WORDS.emotion),
risk:count(text,WORDS.risk),
caution:count(text,WORDS.caution)
};
let total=Object.values(s).reduce((a,b)=>a+b,0)||1;
Object.keys(s).forEach(k=>s[k]=Math.round(s[k]/total*100));
return s;
}

function drawRadar(s){
const c=document.getElementById("chart");
const ctx=c.getContext("2d");
ctx.clearRect(0,0,c.width,c.height);

const cx=210, cy=150, R=110;
const keys=["logic","emotion","risk","caution"];
const labels=["منطقي","عاطفي","مغامر","حذر"];

ctx.strokeStyle="#ccc";

for(let r=1;r<=5;r++){
ctx.beginPath();
keys.forEach((_,i)=>{
let a=i*2*Math.PI/4-Math.PI/2;
let x=cx+Math.cos(a)*R*r/5;
let y=cy+Math.sin(a)*R*r/5;
i?ctx.lineTo(x,y):ctx.moveTo(x,y);
});
ctx.closePath(); ctx.stroke();
}

keys.forEach((_,i)=>{
let a=i*2*Math.PI/4-Math.PI/2;
ctx.beginPath();
ctx.moveTo(cx,cy);
ctx.lineTo(cx+Math.cos(a)*R,cy+Math.sin(a)*R);
ctx.stroke();
ctx.fillText(labels[i],cx+Math.cos(a)*(R+15)-18,cy+Math.sin(a)*(R+15));
});

ctx.beginPath();
keys.forEach((k,i)=>{
let v=s[k]/100;
let a=i*2*Math.PI/4-Math.PI/2;
let x=cx+Math.cos(a)*R*v;
let y=cy+Math.sin(a)*R*v;
i?ctx.lineTo(x,y):ctx.moveTo(x,y);
});
ctx.closePath();
ctx.fillStyle="rgba(45,137,239,.3)";
ctx.fill();
ctx.strokeStyle="rgba(45,137,239,.9)";
ctx.stroke();
}

function recommendation(s){
let max=Object.keys(s).sort((a,b)=>s[b]-s[a])[0];
return{
logic:"تميل للتحليل — استمر في جمع المعلومات قبل القرار.",
emotion:"عاطفي — راجع الحقائق مع شعورك.",
risk:"مغامر — قسّم المخاطرة لخطوات.",
caution:"حذر — حدّد موعدًا للحسم."
}[max];
}

function analyze(){
let text=document.getElementById("text").value.trim();
if(!text) return alert("اكتب نصًا أولًا");

let s=analyzeText(text);
drawRadar(s);

document.getElementById("summary").textContent=recommendation(s);

saveSession(text,s);
updateFingerprint(s);
renderHistory();
showFingerprint();
}

function saveSession(text,s){
let data=JSON.parse(localStorage.getItem("sessions")||"[]");
data.unshift({text,s,time:Date.now()});
localStorage.setItem("sessions",JSON.stringify(data.slice(0,10)));
}

function renderHistory(){
let data=JSON.parse(localStorage.getItem("sessions")||"[]");
let html="<h3>📜 الجلسات</h3>";
data.forEach(i=>{
html+=`<div class="item">
${new Date(i.time).toLocaleString()}<br>
منطقي ${i.s.logic}% | عاطفي ${i.s.emotion}% | مغامر ${i.s.risk}% | حذر ${i.s.caution}%<br>
<span class="small">${i.text}</span>
</div>`;
});
document.getElementById("history").innerHTML=html;
}

function updateFingerprint(s){
let f=JSON.parse(localStorage.getItem("fingerprint")||'{"logic":0,"emotion":0,"risk":0,"caution":0,"count":0}');
f.logic+=s.logic;
f.emotion+=s.emotion;
f.risk+=s.risk;
f.caution+=s.caution;
f.count++;
localStorage.setItem("fingerprint",JSON.stringify(f));
}

function showFingerprint(){
let f=JSON.parse(localStorage.getItem("fingerprint")||"null");
if(!f||!f.count) return;
document.getElementById("fingerprint").textContent=
"🧬 بصمتك التراكمية — منطقي "+Math.round(f.logic/f.count)+"% | عاطفي "+Math.round(f.emotion/f.count)+"% | مغامر "+Math.round(f.risk/f.count)+"% | حذر "+Math.round(f.caution/f.count)+"%";
}

function clearData(){
if(!confirm("مسح كل البيانات؟")) return;
localStorage.clear();
location.reload();
}

function toggleTheme(){
document.body.classList.toggle("dark");
localStorage.setItem("theme",document.body.classList.contains("dark"));
}

if(localStorage.getItem("theme")==="true"){
document.body.classList.add("dark");
}

renderHistory();
showFingerprint();
</script>

</body>
</html>
