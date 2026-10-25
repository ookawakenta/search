# search<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>APEIRON</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@300;400&display=swap">
<style>
:root{--bg:#0e0e0e;--glow:#2b2b2b;--fg:#f2f2f2;--sub:#8c8c8c;--bar:#fff;--barfg:#222;--barsub:#555;--line:#3a3a3a;color-scheme:dark;
box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
html{scroll-padding-top:env(safe-area-inset-top,0px)}
*{box-sizing:border-box}
body{margin:0;min-height:100vh;color:var(--fg);font-family:"Noto Sans JP","Hiragino Sans",Meiryo,sans-serif;font-weight:300;
background:radial-gradient(ellipse at 50% 35%,var(--glow) 0%,var(--bg) 70%) fixed,var(--bg)}
body::before{content:"";position:fixed;inset:0;pointer-events:none;opacity:.35;
background-image:radial-gradient(rgba(255,255,255,.07) 1px,transparent 1.2px);background-size:4px 4px}
main{position:relative;max-width:1100px;margin:0 auto;padding:12px clamp(18px,5vw,52px) 48px}
h1{font-weight:300;font-size:clamp(30px,5vw,40px);margin:0 0 22px;letter-spacing:.02em}
.bar{display:flex;align-items:center;gap:18px;height:52px;padding:0 20px;background:var(--bar);border-radius:6px}
.bar svg{flex:none;width:28px;height:28px;fill:none;stroke:var(--barfg);stroke-width:1.6;stroke-linecap:round}
.bar input{flex:1;min-width:0;border:0;outline:0;background:transparent;color:var(--barfg);font:inherit;font-weight:400;font-size:16px}
.bar input::placeholder{color:var(--barsub)}
.bar:focus-within{box-shadow:0 0 0 2px rgba(255,255,255,.35)}
.modes{display:flex;flex-wrap:wrap;gap:6px 22px;margin-top:18px}
.modes button{font:inherit;font-size:14px;color:var(--sub);background:none;border:0;border-bottom:1px solid transparent;padding:4px 0;cursor:pointer}
.modes button[aria-pressed="true"]{color:var(--fg);border-bottom-color:var(--fg)}
.hist{margin-top:34px}
.hist ul{list-style:none;margin:0;padding:0}
.hist li{border-top:1px solid var(--line)}
.hist li:last-child{border-bottom:1px solid var(--line)}
.hist li button{width:100%;display:flex;justify-content:space-between;gap:12px;font:inherit;font-size:15px;color:var(--fg);background:none;border:0;padding:12px 2px;cursor:pointer;text-align:left}
.hist li button:hover{background:rgba(255,255,255,.04)}
.hist li span{color:var(--sub);font-size:12px;flex:none}
.hist .top{display:flex;justify-content:space-between;color:var(--sub);font-size:12px;margin-bottom:8px}
.hist .top button{font:inherit;color:var(--sub);background:none;border:0;text-decoration:underline;cursor:pointer}
button:focus-visible,input:focus-visible{outline:2px solid #fff;outline-offset:2px}
</style>
</head>
<body>
<main>
  <h1>APEIRON</h1>
  <form class="bar" id="f" role="search">
    <svg viewBox="0 0 28 28" aria-hidden="true"><circle cx="12" cy="12" r="8.5"/><path d="M18.5 18.5L25 25"/></svg>
    <input id="q" type="search" autocomplete="off" autofocus placeholder="APEIRONで検索" aria-label="検索ワード">
  </form>
  <div class="modes" id="modes" role="group" aria-label="検索の種類"></div>
  <section class="hist" id="hist" hidden>
    <div class="top"><span>最近の検索</span><button type="button" id="clr">全部消す</button></div>
    <ul id="hl"></ul>
  </section>
</main>
<script>
const MODES=[
 {n:"ウェブ",u:"https://duckduckgo.com/?q="},
 {n:"画像",u:"https://duckduckgo.com/?iax=images&ia=images&q="},
 {n:"地図",u:"https://www.openstreetmap.org/search?query="},
 {n:"動画",u:"https://www.youtube.com/results?search_query="},
 {n:"辞書",u:"https://ja.wikipedia.org/w/index.php?search="},
 {n:"翻訳",u:"https://translate.google.com/?sl=auto&tl=ja&op=translate&text="}
];
let m=0,hist=[];
try{const s=localStorage.getItem("mode");if(s!==null&&MODES[+s])m=+s;
const h=JSON.parse(localStorage.getItem("hist")||"[]");if(Array.isArray(h))hist=h.filter(x=>x&&MODES[x.m]&&typeof x.q==="string")}catch(_){}
const $=id=>document.getElementById(id);
const store=()=>{try{localStorage.setItem("mode",m);localStorage.setItem("hist",JSON.stringify(hist))}catch(_){}};
function drawModes(){
 const box=$("modes");box.innerHTML="";
 MODES.forEach((x,i)=>{const b=document.createElement("button");b.type="button";b.textContent=x.n;
  b.setAttribute("aria-pressed",i===m);b.onclick=()=>{m=i;store();drawModes();$("q").focus()};box.appendChild(b)});
}
function drawHist(){
 const ul=$("hl");ul.innerHTML="";$("hist").hidden=!hist.length;
 hist.forEach(x=>{const li=document.createElement("li"),b=document.createElement("button");b.type="button";
  const t=document.createElement("em");t.style.fontStyle="normal";t.textContent=x.q;
  const s=document.createElement("span");s.textContent=MODES[x.m].n;b.append(t,s);
  b.onclick=()=>go(x.q,x.m);li.appendChild(b);ul.appendChild(li)});
}
function go(q,mode){
 hist=[{q,m:mode},...hist.filter(x=>!(x.q===q&&x.m===mode))].slice(0,8);store();drawHist();
 const a=document.createElement("a");
 a.href=MODES[mode].u+encodeURIComponent(q);a.target="_blank";a.rel="noopener noreferrer";
 document.body.appendChild(a);a.click();a.remove();
}
$("f").addEventListener("submit",e=>{e.preventDefault();const v=$("q").value.trim();if(v)go(v,m)});
$("clr").onclick=()=>{hist=[];store();drawHist()};
drawModes();drawHist();
</script>
</body>
</html>