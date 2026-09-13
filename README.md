from pathlib import Path
import zipfile

root = Path("/mnt/data/GenZedits")
root.mkdir(parents=True, exist_ok=True)

html = r'''<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="description" content="GenZedits — discover AI-generated images and copy the prompts that inspired them.">
<title>GenZedits — AI Images & Prompts</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
<style>
:root{--bg:#07070b;--panel:#111119;--panel2:#171722;--text:#f7f7fb;--muted:#a7a7b5;--line:#292936;--accent:#9b5cff;--accent2:#00e5ff}
*{box-sizing:border-box}html{scroll-behavior:smooth}body{margin:0;background:radial-gradient(circle at 80% -10%,#32145c 0,transparent 30%),var(--bg);color:var(--text);font-family:Inter,system-ui,sans-serif}
nav{height:72px;display:flex;align-items:center;justify-content:space-between;padding:0 6%;border-bottom:1px solid rgba(255,255,255,.08);backdrop-filter:blur(14px);position:sticky;top:0;background:rgba(7,7,11,.75);z-index:5}
.logo{font-weight:800;font-size:22px;letter-spacing:-.7px}.logo span{color:var(--accent)}
nav a{color:#ddd;text-decoration:none;margin-left:25px;font-size:14px}.btn{background:linear-gradient(135deg,var(--accent),#6e38ff);padding:11px 17px;border-radius:12px;color:#fff!important;font-weight:700}
.hero{text-align:center;padding:90px 20px 55px;max-width:900px;margin:auto}.badge{display:inline-block;padding:8px 12px;border:1px solid #39324c;border-radius:999px;color:#d6c6ff;background:#161023;font-size:12px;font-weight:600}
h1{font-size:clamp(46px,8vw,82px);line-height:.98;letter-spacing:-4px;margin:20px 0 18px}.gradient{background:linear-gradient(90deg,#fff,#b76cff,#00e5ff);-webkit-background-clip:text;background-clip:text;color:transparent}
.hero p{color:var(--muted);font-size:18px;line-height:1.7;max-width:650px;margin:0 auto 28px}
.search{max-width:650px;margin:0 auto;display:flex;background:#11111a;border:1px solid #30303c;border-radius:15px;padding:7px}.search input{flex:1;background:transparent;border:0;outline:0;color:white;padding:13px;font-size:15px}.search button{border:0;border-radius:10px;padding:0 20px;background:#fff;color:#08080c;font-weight:800;cursor:pointer}
.wrap{max-width:1180px;margin:auto;padding:30px 20px 80px}.chips{display:flex;gap:10px;overflow:auto;padding:8px 0 25px}.chip{white-space:nowrap;border:1px solid var(--line);background:#101018;color:#ccc;border-radius:999px;padding:9px 14px;cursor:pointer}.chip.active,.chip:hover{background:#21143a;border-color:#6d42a7;color:#fff}
.grid{display:grid;grid-template-columns:repeat(3,1fr);gap:20px}.card{background:linear-gradient(180deg,#111119,#0d0d13);border:1px solid var(--line);border-radius:18px;overflow:hidden;transition:.2s}.card:hover{transform:translateY(-4px);border-color:#4b3b62}.pic{aspect-ratio:4/5;background:#20202b;overflow:hidden}.pic img{width:100%;height:100%;object-fit:cover;display:block}.body{padding:16px}.tag{font-size:11px;color:#c5a8ff;text-transform:uppercase;letter-spacing:.8px}.body h3{margin:7px 0;font-size:17px}.prompt{color:#aaaaba;font-size:13px;line-height:1.55;min-height:62px}.copy{width:100%;margin-top:12px;padding:11px;border:1px solid #3a3a48;background:#171722;color:#fff;border-radius:10px;cursor:pointer;font-weight:700}.copy:hover{background:#242433}
.empty{text-align:center;color:#777;padding:50px;display:none}.about{margin-top:60px;padding:35px;border:1px solid var(--line);border-radius:22px;background:linear-gradient(135deg,#11111b,#0c0c11)}.about h2{margin-top:0}.about p{color:var(--muted);line-height:1.7}
footer{border-top:1px solid var(--line);padding:30px 6%;color:#777;text-align:center;font-size:13px}
.toast{position:fixed;bottom:25px;left:50%;transform:translateX(-50%) translateY(20px);background:#fff;color:#111;padding:11px 16px;border-radius:10px;font-weight:700;opacity:0;pointer-events:none;transition:.25s}.toast.show{opacity:1;transform:translateX(-50%) translateY(0)}
@media(max-width:850px){.grid{grid-template-columns:repeat(2,1fr)}}@media(max-width:580px){nav a:not(.btn){display:none}.grid{grid-template-columns:1fr 1fr;gap:12px}.body{padding:12px}.prompt{font-size:12px}.hero{padding-top:65px}h1{letter-spacing:-2.5px}.search button{padding:0 14px}}
</style>
</head>
<body>
<nav>
  <div class="logo">GenZ<span>edits</span> ✦</div>
  <div><a href="#explore">Explore</a><a href="#about">About</a><a class="btn" href="#explore">Explore Prompts</a></div>
</nav>

<header class="hero">
  <div class="badge">✦ AI IMAGE INSPIRATION</div>
  <h1>See it. <span class="gradient">Prompt it.</span></h1>
  <p>Discover eye-catching AI-generated images and the prompts behind them. Find inspiration, copy a prompt, and create your own.</p>
  <div class="search">
    <input id="search" placeholder="Search images, styles or prompts..." aria-label="Search">
    <button onclick="filterCards()">Search</button>
  </div>
</header>

<main class="wrap" id="explore">
  <div class="chips" id="chips">
    <button class="chip active" data-cat="All">All</button>
    <button class="chip" data-cat="Portrait">Portraits</button>
    <button class="chip" data-cat="Cinematic">Cinematic</button>
    <button class="chip" data-cat="Anime">Anime</button>
    <button class="chip" data-cat="Fantasy">Fantasy</button>
    <button class="chip" data-cat="Aesthetic">Aesthetic</button>
  </div>
  <section class="grid" id="grid"></section>
  <div class="empty" id="empty">No matching prompts found. Try another search.</div>

  <section class="about" id="about">
    <h2>About GenZedits</h2>
    <p>GenZedits is a simple showcase for AI image ideas and reusable prompts. This starter version includes sample cards and a working search, category filter, and copy-prompt button. Replace the sample images and prompts with your own collection.</p>
  </section>
</main>

<footer>© 2026 GenZedits · AI image inspiration</footer>
<div class="toast" id="toast">Prompt copied ✓</div>

<script>
const items=[
 {cat:"Portrait",title:"Neon Street Portrait",img:"https://images.unsplash.com/photo-1506794778202-cad84cf45f1d?auto=format&fit=crop&w=900&q=85",prompt:"Ultra-realistic cinematic portrait, neon city lights, rainy street reflections, shallow depth of field, detailed skin, 85mm lens, moody atmosphere."},
 {cat:"Cinematic",title:"Golden Hour Journey",img:"https://images.unsplash.com/photo-1500530855697-b586d89ba3ee?auto=format&fit=crop&w=900&q=85",prompt:"Cinematic wide shot at golden hour, lone traveler on a mountain ridge, dramatic clouds, warm sunlight, atmospheric haze, film still, high detail."},
 {cat:"Fantasy",title:"Enchanted Forest",img:"https://images.unsplash.com/photo-1511497584788-876760111969?auto=format&fit=crop&w=900&q=85",prompt:"Mystical enchanted forest, glowing particles, ancient trees, soft moonlight, magical mist, fantasy concept art, rich atmosphere, ultra detailed."},
 {cat:"Aesthetic",title:"Minimal Coffee Mood",img:"https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?auto=format&fit=crop&w=900&q=85",prompt:"Cozy minimalist coffee scene, soft morning window light, neutral tones, editorial photography, subtle shadows, clean composition, premium lifestyle aesthetic."},
 {cat:"Anime",title:"Anime City Night",img:"https://images.unsplash.com/photo-1519608487953-e999c86e7455?auto=format&fit=crop&w=900&q=85",prompt:"Anime-inspired futuristic city at night, glowing signs, reflective pavement, expressive character silhouette, vibrant atmosphere, detailed background."},
 {cat:"Cinematic",title:"Retro Film Frame",img:"https://images.unsplash.com/photo-1492691527719-9d1e07e534b4?auto=format&fit=crop&w=900&q=85",prompt:"Retro 35mm film photography, cinematic composition, soft grain, muted highlights, nostalgic mood, natural light, authentic analog texture."}
];
let active="All";
function render(){
 const q=document.getElementById("search").value.toLowerCase().trim();
 const list=items.filter(x=>(active==="All"||x.cat===active)&&(!q||(x.title+" "+x.cat+" "+x.prompt).toLowerCase().includes(q)));
 document.getElementById("grid").innerHTML=list.map((x,i)=>`
 <article class="card"><div class="pic"><img src="${x.img}" alt="${x.title}" loading="lazy"></div>
 <div class="body"><div class="tag">${x.cat}</div><h3>${x.title}</h3><div class="prompt">${x.prompt}</div>
 <button class="copy" onclick="copyPrompt(${items.indexOf(x)})">Copy Prompt</button></div></article>`).join("");
 document.getElementById("empty").style.display=list.length?"none":"block";
}
function filterCards(){render()}
function copyPrompt(i){navigator.clipboard.writeText(items[i].prompt).then(()=>{const t=document.getElementById("toast");t.classList.add("show");setTimeout(()=>t.classList.remove("show"),1500)})}
document.querySelectorAll(".chip").forEach(b=>b.onclick=()=>{document.querySelectorAll(".chip").forEach(x=>x.classList.remove("active"));b.classList.add("active");active=b.dataset.cat;render()});
document.getElementById("search").addEventListener("input",render);render();
</script>
</body>
</html>'''

(root/"index.html").write_text(html, encoding="utf-8")
zip_path=Path("/mnt/data/GenZedits-website.zip")
with zipfile.ZipFile(zip_path,"w",zipfile.ZIP_DEFLATED) as z:
    z.write(root/"index.html","index.html")
print(f"Created: {zip_path}")
