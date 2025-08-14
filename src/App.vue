<template>
  <header class="wrap">
    <h1>🧪⚔️ <span class="spark">Batalla</span> de <span class="spark">Monstruos</span> — Vue 3</h1>
    <div class="stack">
      <button class="btn ghost" @click="seed">Cargar ejemplos</button>
      <button class="btn ghost" @click="clearAll">Limpiar</button>
      <a class="btn secondary" href="https://github.com/new" target="_blank">Crear repo en GitHub</a>
    </div>
  </header>

  <main class="wrap grid">
    <section class="card">
      <h2>👾 Monstruo</h2>
      <p class="muted">Crea, edita y elimina monstruos con sus atributos.</p>
      <MonsterForm
        :model="form"
        :is-edit="!!form.id"
        @save="saveMonster"
        @cancel="resetForm"
      />
      <div class="footer">Tip: Todo se guarda en tu navegador (localStorage).</div>
    </section>

    <section class="card">
      <h2>📜 Lista de Monstruos</h2>
      <MonsterList
        :items="monsters"
        @edit="onEdit"
        @remove="onDelete"
      />
      <div v-if="monsters.length === 0" class="empty">Sin monstruos aún. ¡Crea el primero! 🌟</div>

      <hr class="sep" />

      <h2>⚔️ Nueva Batalla</h2>
      <BattlePanel
        :monsters="monsters"
        :battle="currentBattle"
        @fight="onFight"
      />

      <hr class="sep" />

      <h2>🏆 Historial de Batallas</h2>
      <HistoryList
        :battles="battles"
        @view="(b) => currentBattle = b"
        @remove="removeBattle"
      />
      <div v-if="battles.length === 0" class="empty">Sin batallas registradas.</div>
    </section>
  </main>
</template>

<script setup>
/*
  App.vue
  - Mantiene el estado global (monstruos y batallas).
  - Implementa reglas de batalla por turnos.
  - Persistencia simple en localStorage.
  - Estilos y confeti 🎉.
*/
import { reactive, ref } from 'vue'
import MonsterForm from './components/MonsterForm.vue'
import MonsterList from './components/MonsterList.vue'
import BattlePanel from './components/BattlePanel.vue'
import HistoryList from './components/HistoryList.vue'

// ---------- Persistencia ----------
function getLS(key, def) {
  try { return JSON.parse(localStorage.getItem(key) || JSON.stringify(def)) }
  catch { return def }
}
function setLS(key, val) { localStorage.setItem(key, JSON.stringify(val)) }

// Estado
const monsters = ref(getLS('monsters', []))
const battles = ref(getLS('battles', []))
const currentBattle = ref(null)

// Formulario
const form = reactive({
  id: '', name: '', hp: 10, atk: 5, def: 2, spd: 3, img: ''
})

function uid(){ return Math.random().toString(36).slice(2,9) }
function resetForm(){
  Object.assign(form, { id:'', name:'', hp:10, atk:5, def:2, spd:3, img:'' })
}

function saveMonster(payload){
  const monster = {
    id: payload.id || uid(),
    name: (payload.name || '').trim() || 'Sin nombre',
    hp: Math.max(1, Number(payload.hp || 1)),
    atk: Math.max(1, Number(payload.atk || 1)),
    def: Math.max(0, Number(payload.def || 0)),
    spd: Math.max(1, Number(payload.spd || 1)),
    img: (payload.img || '').trim()
  }
  const i = monsters.value.findIndex(m => m.id === monster.id)
  if(i >= 0) monsters.value[i] = monster
  else monsters.value.push(monster)
  setLS('monsters', monsters.value)
  resetForm()
}

function onEdit(id){
  const m = monsters.value.find(x => x.id === id)
  if(!m) return
  Object.assign(form, m)
}
function onDelete(id){
  monsters.value = monsters.value.filter(x => x.id !== id)
  setLS('monsters', monsters.value)
}

// ---------- Reglas de batalla ----------
// 1) Orden: mayor velocidad; si empatan, mayor ataque.
// 2) Daño: atk - def; mínimo 1.
// 3) Fin: cuando la vida de un monstruo llega a 0.
// 4) Registrar ganador y turnos.
function simulateBattle(a, b){
  const A = JSON.parse(JSON.stringify(a))
  const B = JSON.parse(JSON.stringify(b))
  A.hpCurrent = A.hp; B.hpCurrent = B.hp
  const turns = []

  let attacker = A, defender = B
  if(B.spd > A.spd || (B.spd === A.spd && B.atk > A.atk)){
    attacker = B; defender = A
  }

  while(A.hpCurrent > 0 && B.hpCurrent > 0){
    const dmg = Math.max(1, attacker.atk - defender.def)
    defender.hpCurrent = Math.max(0, defender.hpCurrent - dmg)
    turns.push({ from: attacker.name, to: defender.name, dmg, toHp: defender.hpCurrent })
    if(defender.hpCurrent === 0) break
    const tmp = attacker; attacker = defender; defender = tmp
  }

  const winner = A.hpCurrent > 0 ? A : B
  const loser  = winner === A ? B : A
  return {winner, loser, turns, at: new Date().toISOString()}
}

function onFight({ idA, idB }){
  if(!idA || !idB || idA === idB){ alert('Selecciona dos monstruos distintos.'); return }
  const a = monsters.value.find(x => x.id === idA)
  const b = monsters.value.find(x => x.id === idB)
  if(!a || !b){ alert('Monstruos inválidos'); return }
  const result = simulateBattle(a,b)
  currentBattle.value = result
  battles.value.push(result)
  setLS('battles', battles.value)
  confetti()
}

function removeBattle(at){
  battles.value = battles.value.filter(b => b.at !== at)
  setLS('battles', battles.value)
}

// ---------- Semillas ----------
function seed(){
  const seeds = [
    {name:'Glorp', hp:18, atk:7, def:3, spd:4, img:'https://images.unsplash.com/photo-1558981403-c5f9899a28bc?q=80&w=300&auto=format&fit=crop'},
    {name:'Zazz',  hp:14, atk:10,def:2, spd:5, img:'https://images.unsplash.com/photo-1606112219348-204d7d8b94ee?q=80&w=300&auto=format&fit=crop'},
    {name:'Muki',  hp:22, atk:6, def:5, spd:2, img:'https://images.unsplash.com/photo-1603349206293-4a49c9bb0965?q=80&w=300&auto=format&fit=crop'},
    {name:'Roxo',  hp:16, atk:8, def:1, spd:6, img:'https://images.unsplash.com/photo-1616626783878-3c11472edbba?q=80&w=300&auto=format&fit=crop'}
  ].map(s => ({ id: uid(), ...s }))
  monsters.value = seeds
  setLS('monsters', monsters.value)
}
function clearAll(){
  if(!confirm('Esto borrará monstruos y batallas.')) return
  monsters.value = []
  battles.value = []
  currentBattle.value = null
  setLS('monsters', monsters.value)
  setLS('battles', battles.value)
}

// ---------- Confeti 🎉 ----------
function confetti(){
  const count = 60
  for(let i=0;i<count;i++){
    const p = document.createElement('div')
    p.className = 'confetti'
    const sx = Math.random() * 100 + 'vw'
    const tx = (Math.random()*200 - 100) + 'px'
    const dur = (5 + Math.random()*3) + 's'
    p.style.left = sx
    p.style.setProperty('--x', tx)
    p.style.background = ['#ff69b4','#7cf5ff','#ffd166','#58f79c','#a5b4fc'][i%5]
    p.style.animationDuration = dur
    document.body.appendChild(p)
    setTimeout(() => p.remove(), 9000)
  }
}
</script>

<style>
  :root{
    --bg: #0f1020;
    --card: #1b1e34;
    --ink: #e7e8ff;
    --muted: #9aa0ff;
    --accent: #ff69b4;
    --accent-2: #7cf5ff;
    --danger: #ff4d6d;
    --success: #58f79c;
    --warning: #ffd166;
    --shadow: 0 10px 30px rgba(0,0,0,.35);
    --radius: 18px;
  }
  *{box-sizing:border-box}
  html,body,#app{height:100%}
  body{
    margin:0;
    font-family: 'Fredoka', system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
    color:var(--ink);
    background: radial-gradient(1200px 800px at 20% 10%, #1d2040 0, #0f1020 50%) fixed;
    background-color: var(--bg);
  }
  header{
    display:flex;align-items:center;justify-content:space-between;
    gap:16px;padding:24px 20px 10px 20px;position:sticky;top:0;z-index:5;
    backdrop-filter:saturate(1.1) blur(8px);
    background: linear-gradient(180deg, rgba(15,16,32,.8), rgba(15,16,32,0));
  }
  header h1{ margin:0;font-size: clamp(22px, 4vw, 36px); letter-spacing:.5px; }
  header h1 .spark{color:var(--accent)}
  .wrap{max-width:1200px;margin:0 auto;padding:0 16px 40px}
  .grid{display:grid;gap:16px;grid-template-columns:1fr}
  @media(min-width:960px){ .grid{grid-template-columns: 360px 1fr} }
  .card{
    background: linear-gradient(180deg, rgba(27,30,52,.9), rgba(27,30,52,.8));
    border:1px solid rgba(255,255,255,.06);
    border-radius: var(--radius);
    box-shadow: var(--shadow);
    padding:16px;
  }
  .card h2{margin:0 0 8px;font-size:22px}
  .muted{color:var(--muted);font-size:14px}
  label{display:block;font-size:14px;margin:10px 0 6px}
  .row{display:flex; gap:10px}
  .row > *{flex:1}
  .btn{
    display:inline-flex; align-items:center; justify-content:center;
    gap:8px; border:none; padding:10px 14px; border-radius:14px;
    color:#12131f; background:var(--accent-2); font-weight:700; cursor:pointer;
    box-shadow: var(--shadow);
    transition: transform .05s ease, filter .15s ease;
    text-decoration:none;
  }
  .btn:hover{filter:brightness(1.05)}
  .btn:active{transform:translateY(1px)}
  .btn.secondary{background:#a5b4fc;color:#0b0c16}
  .btn.danger{background:var(--danger);color:white}
  .btn.ghost{background:transparent;border:1px solid rgba(255,255,255,.15);color:var(--ink)}
  .stack{display:flex;flex-wrap:wrap;gap:10px}
  input[type="text"], input[type="number"], select, textarea{
    width:100%; padding:10px 12px; border-radius:12px;
    border:1px solid rgba(255,255,255,.12); background:#14162b; color:var(--ink);
    outline:none;
  }
  input[type="number"]::-webkit-inner-spin-button{opacity:1}
  .sep{margin:18px 0;border:0;border-top:1px solid rgba(255,255,255,.1)}
  .empty{padding:12px;border:1px dashed rgba(255,255,255,.15);border-radius:12px;color:var(--muted);text-align:center}
  .footer{margin-top:12px;text-align:center;color:var(--muted);font-size:12px}

  /* Lista de monstruos */
  .monster-list{display:grid;grid-template-columns:repeat(auto-fill, minmax(240px,1fr));gap:12px}
  .monster{
    background: #12142a; border:1px solid rgba(255,255,255,.06); border-radius:16px; padding:10px;
    display:flex; gap:10px; align-items:flex-start; position:relative; overflow:hidden;
  }
  .monster img{
    width:72px; height:72px; border-radius:12px; object-fit:cover; flex: none;
    border:2px solid rgba(255,255,255,.1);
  }
  .monster .name{font-weight:700}
  .stat{font-size:12px;opacity:.9}
  .badge{font-size:11px;padding:4px 8px;border-radius:999px;border:1px solid rgba(255,255,255,.18);}

  /* Timeline */
  .timeline{ margin-top:10px; max-height:260px; overflow:auto; padding-right:6px; border-top:1px dashed rgba(255,255,255,.2) }
  .turn{ display:flex; gap:8px; padding:8px 0; border-bottom:1px dashed rgba(255,255,255,.08); align-items:center }
  .turn .hit{font-weight:700}
  .winner{color:var(--success)} .loser{color:var(--danger)}

  /* Confeti */
  .confetti{
    position:fixed; top:-10px; width:8px; height:12px;
    background:white; opacity:.95; transform:rotate(0deg);
    border-radius:2px; pointer-events:none;
    animation: fall linear forwards;
  }
  @keyframes fall{ to{ transform: translate(var(--x), 100vh) rotate(720deg); opacity:1;} }
</style>
