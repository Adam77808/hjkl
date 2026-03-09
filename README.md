import { useState, useEffect, useRef } from "react";

const TEAMS = [
  { key: "ECLAIR", name: "LES ÉCLAIRS", color: "#00f0ff", bg: "rgba(0,240,255,0.08)", icon: "⚡" },
  { key: "DRAGON", name: "LES DRAGONS", color: "#ff2d6b", bg: "rgba(255,45,107,0.08)", icon: "🔥" },
  { key: "ETOILE", name: "LES ÉTOILES", color: "#b44dff", bg: "rgba(180,77,255,0.08)", icon: "🌀" },
  { key: "ROBOT", name: "LES ROBOTS", color: "#00ff88", bg: "rgba(0,255,136,0.08)", icon: "🧬" },
];

const AVATARS = ["🤖","👾","🐱","🦊","🐸","🦄","🐉","🚀","💎","🌟","🐶","🐧","🦋","🌈","🎮","🧠"];

const MISSIONS = [
  { id: 1, title: "Le robot perdu", difficulty: "CP", xp: 50, icon: "🤖", description: "Guide le robot jusqu'à l'étoile ! Choisis les bonnes flèches.", type: "path", grid: [["🤖","⬜","⬜"],["🚫","🚫","⬜"],["⬜","⬜","⭐"]], correctSequence: ["→","→","↓","↓"], options: ["→","↓","←","↑"] },
  { id: 2, title: "Suite logique", difficulty: "CP", xp: 50, icon: "🔢", description: "Quel nombre vient après ?", type: "sequence", sequences: [{ shown: [2,4,6,8], answer: 10, hint: "On ajoute 2 !" },{ shown: [1,3,5,7], answer: 9, hint: "De 2 en 2 !" },{ shown: [5,10,15,20], answer: 25, hint: "De 5 en 5 !" }] },
  { id: 3, title: "Le code couleur", difficulty: "CP", xp: 60, icon: "🎨", description: "Quel est le prochain dans la séquence ?", type: "pattern", patterns: [{ shown: ["🔴","🔵","🔴","🔵","🔴"], answer: "🔵", options: ["🔴","🔵","🟢","🟡"] },{ shown: ["🟢","🟢","🟡","🟢","🟢"], answer: "🟡", options: ["🔴","🔵","🟢","🟡"] },{ shown: ["🔵","🔴","🔴","🔵","🔴"], answer: "🔴", options: ["🔴","🔵","🟢","🟡"] }] },
  { id: 4, title: "Robot artiste", difficulty: "CE1", xp: 80, icon: "🎨", description: "Place les instructions dans le bon ordre !", type: "draw", instructions: [{ label: "Avance 3 cases →", icon: "➡️➡️➡️" },{ label: "Tourne en bas ↓", icon: "⤵️" },{ label: "Avance 3 cases ↓", icon: "⬇️⬇️⬇️" },{ label: "Tourne à gauche ←", icon: "⤴️" }], correctOrder: [0,1,2,3], shape: "Un carré !" },
  { id: 5, title: "Le tri magique", difficulty: "CE1", xp: 80, icon: "📦", description: "Range du plus petit au plus grand !", type: "sort", sets: [{ numbers: [7,2,9,1,5], answer: [1,2,5,7,9] },{ numbers: [15,8,23,3,11], answer: [3,8,11,15,23] },{ numbers: [40,10,30,20,50], answer: [10,20,30,40,50] }] },
  { id: 6, title: "Si... Alors...", difficulty: "CE1", xp: 100, icon: "🧠", description: "Si c'est vrai, que se passe-t-il ?", type: "condition", questions: [{ q: "SI il pleut ☔ ALORS je prends...", options: ["🏖️ Maillot","☂️ Parapluie","🕶️ Lunettes","🎸 Guitare"], answer: 1 },{ q: "SI il fait nuit 🌙 ALORS j'allume...", options: ["🚿 Douche","💡 Lumière","📻 Radio","🎨 Peinture"], answer: 1 },{ q: "SI j'ai faim 😋 ALORS je...", options: ["💤 Dors","🍽️ Mange","🏃 Cours","📖 Lis"], answer: 1 }] },
  { id: 7, title: "Le labyrinthe", difficulty: "CE2", xp: 120, icon: "🏰", description: "Guide le héros ! Attention aux murs !", type: "path", grid: [["🦊","⬜","🚫","⬜","⬜"],["🚫","⬜","🚫","⬜","🚫"],["⬜","⬜","⬜","⬜","⬜"],["⬜","🚫","🚫","🚫","⬜"],["⬜","⬜","⬜","⬜","⭐"]], correctSequence: ["↓","↓","→","→","→","→","↓","↓"], options: ["→","↓","←","↑"] },
  { id: 8, title: "Boucle magique", difficulty: "CE2", xp: 120, icon: "🔄", description: "Compte ce que la boucle répète !", type: "loop", questions: [{ action: "⭐", times: 4, q: "RÉPÈTE 4 FOIS : ajoute ⭐\nCombien d'étoiles ?", answer: 4, options: [2,3,4,5] },{ action: "🎵", times: 3, q: "RÉPÈTE 3 FOIS : joue 2 notes 🎵🎵\nCombien de notes au total ?", answer: 6, options: [3,5,6,8] },{ action: "🧱", times: 5, q: "RÉPÈTE 5 FOIS : pose 1 brique 🧱\nCombien de briques ?", answer: 5, options: [4,5,6,10] }] },
  { id: 9, title: "Décryptage", difficulty: "CE2", xp: 150, icon: "🔐", description: "Trouve la valeur de chaque symbole !", type: "decrypt", puzzles: [{ equation: "⭐ + ⭐ = 6", q: "Que vaut ⭐ ?", answer: 3, options: [2,3,4,5] },{ equation: "🌙 + 2 = 7", q: "Que vaut 🌙 ?", answer: 5, options: [3,4,5,6] },{ equation: "🔥 × 3 = 12", q: "Que vaut 🔥 ?", answer: 4, options: [2,3,4,6] }] },
];

// ============ MINI-GAMES ============

function PathGame({ mission, onWin }) {
  const [seq, setSeq] = useState([]);
  const [msg, setMsg] = useState("");
  const [won, setWon] = useState(false);
  const add = (a) => {
    if (won) return;
    const n = [...seq, a];
    for (let i = 0; i < n.length; i++) { if (n[i] !== mission.correctSequence[i]) { setMsg("❌ Mauvais chemin !"); setSeq([]); return; } }
    setSeq(n);
    if (n.length === mission.correctSequence.length) { setMsg("🎉 Bravo !"); setWon(true); setTimeout(onWin, 1200); }
    else setMsg(`${n.length}/${mission.correctSequence.length}`);
  };
  return (
    <div style={{ textAlign: "center" }}>
      <div style={{ display: "inline-grid", gridTemplateColumns: `repeat(${mission.grid[0].length}, 50px)`, gap: 4, marginBottom: 14 }}>
        {mission.grid.flat().map((c, i) => <div key={i} style={{ width: 50, height: 50, borderRadius: 8, display: "flex", alignItems: "center", justifyContent: "center", fontSize: "1.4rem", background: c === "🚫" ? "rgba(255,0,0,0.1)" : c === "⭐" ? "rgba(255,215,0,0.15)" : "rgba(255,255,255,0.03)", border: c === "⭐" ? "2px solid #ffd70066" : "1px solid #1a1a2e" }}>{c === "⬜" ? "" : c}</div>)}
      </div>
      <div style={{ minHeight: 36, marginBottom: 10, display: "flex", justifyContent: "center", gap: 5, flexWrap: "wrap" }}>
        {seq.map((s, i) => <span key={i} style={{ background: "rgba(0,240,255,0.1)", border: "1px solid #00f0ff33", borderRadius: 8, padding: "5px 10px", fontSize: "1.1rem" }}>{s}</span>)}
      </div>
      <div style={{ display: "flex", justifyContent: "center", gap: 8, marginBottom: 10 }}>
        {mission.options.map(a => <button key={a} onClick={() => add(a)} disabled={won} style={{ width: 56, height: 56, borderRadius: 12, fontSize: "1.6rem", background: "rgba(0,240,255,0.08)", border: "2px solid #00f0ff44", cursor: won ? "default" : "pointer" }}>{a}</button>)}
      </div>
      <button onClick={() => { setSeq([]); setMsg(""); }} style={{ background: "none", border: "1px solid #333", borderRadius: 8, padding: "5px 14px", color: "#666", fontSize: "0.8rem", cursor: "pointer" }}>🔄 Recommencer</button>
      {msg && <div style={{ marginTop: 10, fontSize: "1rem", color: won ? "#00ff88" : msg.includes("❌") ? "#ff2d6b" : "#ffaa00", fontWeight: 700 }}>{msg}</div>}
    </div>
  );
}

function MultiStepGame({ items, checkAnswer, onWin, label }) {
  const [idx, setIdx] = useState(0);
  const [msg, setMsg] = useState("");
  const [won, setWon] = useState(false);
  const [input, setInput] = useState("");
  const item = items[idx];
  const tryAnswer = (ans) => {
    if (checkAnswer(item, ans)) {
      if (idx + 1 >= items.length) { setMsg("🎉 Tout est réussi !"); setWon(true); setTimeout(onWin, 1200); }
      else { setMsg("✅ Correct !"); setTimeout(() => { setIdx(i => i + 1); setMsg(""); setInput(""); }, 800); }
    } else { setMsg(item.hint ? `❌ ${item.hint}` : "❌ Réessaie !"); }
  };
  return { idx, msg, won, input, setInput, item, tryAnswer, total: items.length };
}

function SequenceGame({ mission, onWin }) {
  const g = MultiStepGame({ items: mission.sequences, checkAnswer: (it, a) => parseInt(a) === it.answer, onWin, label: "Suite" });
  return (
    <div style={{ textAlign: "center" }}>
      <div style={{ fontSize: "0.75rem", color: "#555", marginBottom: 8 }}>Suite {g.idx + 1}/{g.total}</div>
      <div style={{ display: "flex", justifyContent: "center", gap: 8, marginBottom: 18, flexWrap: "wrap", alignItems: "center" }}>
        {g.item.shown.map((n, i) => <span key={i} style={{ background: "rgba(0,240,255,0.1)", border: "1px solid #00f0ff33", borderRadius: 10, padding: "10px 16px", fontSize: "1.4rem", fontWeight: 700, color: "#00f0ff" }}>{n}</span>)}
        <span style={{ fontSize: "1.4rem", color: "#ffaa00" }}>→</span>
        <span style={{ background: "rgba(255,170,0,0.1)", border: "2px dashed #ffaa0066", borderRadius: 10, padding: "10px 16px", fontSize: "1.4rem", fontWeight: 700, color: "#ffaa00" }}>?</span>
      </div>
      <div style={{ display: "flex", justifyContent: "center", gap: 8 }}>
        <input value={g.input} onChange={e => g.setInput(e.target.value.replace(/[^0-9]/g, ""))} onKeyDown={e => e.key === "Enter" && g.tryAnswer(g.input)} placeholder="?" style={{ width: 70, textAlign: "center", background: "rgba(0,0,0,0.4)", border: "1px solid #333", borderRadius: 10, padding: 10, color: "#fff", fontSize: "1.2rem", fontFamily: "monospace", outline: "none" }} />
        <button onClick={() => g.tryAnswer(g.input)} disabled={g.won || !g.input} style={{ background: "#00f0ff22", border: "2px solid #00f0ff66", borderRadius: 10, padding: "10px 18px", color: "#00f0ff", fontSize: "1rem", cursor: "pointer", fontWeight: 700 }}>Valider ✓</button>
      </div>
      {g.msg && <div style={{ marginTop: 12, fontSize: "1rem", color: g.msg.includes("🎉") || g.msg.includes("✅") ? "#00ff88" : "#ff8844", fontWeight: 700 }}>{g.msg}</div>}
    </div>
  );
}

function PatternGame({ mission, onWin }) {
  const [idx, setIdx] = useState(0);
  const [msg, setMsg] = useState("");
  const [won, setWon] = useState(false);
  const p = mission.patterns[idx];
  const check = (c) => {
    if (c === p.answer) {
      if (idx + 1 >= mission.patterns.length) { setMsg("🎉 Tous trouvés !"); setWon(true); setTimeout(onWin, 1200); }
      else { setMsg("✅ Bien !"); setTimeout(() => { setIdx(i => i + 1); setMsg(""); }, 800); }
    } else setMsg("❌ Regarde le motif !");
  };
  return (
    <div style={{ textAlign: "center" }}>
      <div style={{ fontSize: "0.75rem", color: "#555", marginBottom: 8 }}>Motif {idx + 1}/{mission.patterns.length}</div>
      <div style={{ display: "flex", justifyContent: "center", gap: 6, marginBottom: 20, flexWrap: "wrap", alignItems: "center" }}>
        {p.shown.map((s, i) => <span key={i} style={{ fontSize: "1.8rem", background: "rgba(255,255,255,0.03)", borderRadius: 10, padding: "8px 12px", border: "1px solid #1a1a2e" }}>{s}</span>)}
        <span style={{ fontSize: "1.8rem", background: "rgba(255,170,0,0.08)", borderRadius: 10, padding: "8px 12px", border: "2px dashed #ffaa0055" }}>❓</span>
      </div>
      <div style={{ display: "flex", justifyContent: "center", gap: 10 }}>
        {p.options.map((o, i) => <button key={i} onClick={() => check(o)} disabled={won} style={{ fontSize: "1.8rem", borderRadius: 12, padding: "12px 18px", background: "rgba(0,0,0,0.3)", border: "2px solid #222", cursor: won ? "default" : "pointer" }}>{o}</button>)}
      </div>
      {msg && <div style={{ marginTop: 12, fontSize: "1rem", color: msg.includes("🎉") || msg.includes("✅") ? "#00ff88" : "#ff8844", fontWeight: 700 }}>{msg}</div>}
    </div>
  );
}

function ConditionGame({ mission, onWin }) {
  const [idx, setIdx] = useState(0);
  const [msg, setMsg] = useState("");
  const [won, setWon] = useState(false);
  const q = mission.questions[idx];
  const check = (i) => {
    if (i === q.answer) {
      if (idx + 1 >= mission.questions.length) { setMsg("🎉 Conditions maîtrisées !"); setWon(true); setTimeout(onWin, 1200); }
      else { setMsg("✅ Logique !"); setTimeout(() => { setIdx(j => j + 1); setMsg(""); }, 800); }
    } else setMsg("❌ Réfléchis bien !");
  };
  return (
    <div style={{ textAlign: "center" }}>
      <div style={{ fontSize: "0.75rem", color: "#555", marginBottom: 8 }}>{idx + 1}/{mission.questions.length}</div>
      <div style={{ fontSize: "1.15rem", color: "#ffaa00", marginBottom: 20, padding: "14px 18px", background: "rgba(255,170,0,0.06)", borderRadius: 12, border: "1px solid #ffaa0022", fontWeight: 600, lineHeight: 1.5 }}>{q.q}</div>
      <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10, maxWidth: 380, margin: "0 auto" }}>
        {q.options.map((o, i) => <button key={i} onClick={() => check(i)} disabled={won} style={{ padding: "12px", borderRadius: 12, fontSize: "0.95rem", background: "rgba(0,0,0,0.3)", border: "2px solid #222", color: "#ddd", cursor: won ? "default" : "pointer" }}>{o}</button>)}
      </div>
      {msg && <div style={{ marginTop: 12, fontSize: "1rem", color: msg.includes("🎉") || msg.includes("✅") ? "#00ff88" : "#ff8844", fontWeight: 700 }}>{msg}</div>}
    </div>
  );
}

function SortGame({ mission, onWin }) {
  const [idx, setIdx] = useState(0);
  const [sel, setSel] = useState([]);
  const [rem, setRem] = useState([...mission.sets[0].numbers]);
  const [msg, setMsg] = useState("");
  const [won, setWon] = useState(false);
  const set = mission.sets[idx];
  const pick = (n) => {
    if (n === set.answer[sel.length]) {
      const ns = [...sel, n];
      setSel(ns);
      setRem(r => { const i = r.indexOf(n); return [...r.slice(0,i),...r.slice(i+1)]; });
      if (ns.length === set.answer.length) {
        if (idx + 1 >= mission.sets.length) { setMsg("🎉 Tout trié !"); setWon(true); setTimeout(onWin, 1200); }
        else { setMsg("✅ Bien !"); setTimeout(() => { const next = mission.sets[idx+1]; setIdx(i=>i+1); setSel([]); setRem([...next.numbers]); setMsg(""); }, 800); }
      }
    } else setMsg("❌ Cherche le plus petit !");
  };
  return (
    <div style={{ textAlign: "center" }}>
      <div style={{ fontSize: "0.75rem", color: "#555", marginBottom: 8 }}>Série {idx+1}/{mission.sets.length}</div>
      <div style={{ fontSize: "0.85rem", color: "#888", marginBottom: 14 }}>Clique du plus petit au plus grand !</div>
      <div style={{ display: "flex", justifyContent: "center", gap: 6, marginBottom: 14, minHeight: 50, flexWrap: "wrap" }}>
        {sel.map((n,i) => <span key={i} style={{ background: "rgba(0,255,136,0.1)", border: "1px solid #00ff8844", borderRadius: 10, padding: "8px 14px", fontSize: "1.2rem", fontWeight: 700, color: "#00ff88" }}>{n}</span>)}
      </div>
      <div style={{ display: "flex", justifyContent: "center", gap: 8, flexWrap: "wrap" }}>
        {rem.map((n,i) => <button key={`${n}-${i}`} onClick={() => pick(n)} disabled={won} style={{ width: 56, height: 56, borderRadius: 12, fontSize: "1.2rem", fontWeight: 700, background: "rgba(0,0,0,0.3)", border: "2px solid #333", color: "#fff", cursor: won ? "default" : "pointer" }}>{n}</button>)}
      </div>
      {msg && <div style={{ marginTop: 12, fontSize: "1rem", color: msg.includes("🎉") || msg.includes("✅") ? "#00ff88" : "#ff8844", fontWeight: 700 }}>{msg}</div>}
    </div>
  );
}

function DrawGame({ mission, onWin }) {
  const [order, setOrder] = useState([]);
  const [rem, setRem] = useState(mission.instructions.map((_,i)=>i));
  const [msg, setMsg] = useState("");
  const [won, setWon] = useState(false);
  const pick = (i) => {
    if (i === mission.correctOrder[order.length]) {
      const n = [...order, i]; setOrder(n); setRem(r => r.filter(x=>x!==i));
      if (n.length === mission.correctOrder.length) { setMsg(`🎉 ${mission.shape}`); setWon(true); setTimeout(onWin, 1500); }
    } else setMsg("❌ Pas le bon ordre !");
  };
  return (
    <div style={{ textAlign: "center" }}>
      <div style={{ display: "flex", flexDirection: "column", gap: 5, marginBottom: 14, alignItems: "center" }}>
        {order.map((i,j) => <div key={j} style={{ background: "rgba(0,255,136,0.08)", border: "1px solid #00ff8844", borderRadius: 10, padding: "8px 18px", fontSize: "0.9rem", color: "#00ff88", fontWeight: 600 }}>#{j+1} {mission.instructions[i].icon} {mission.instructions[i].label}</div>)}
      </div>
      <div style={{ display: "flex", flexDirection: "column", gap: 6, alignItems: "center" }}>
        {rem.map(i => <button key={i} onClick={() => pick(i)} disabled={won} style={{ padding: "10px 20px", borderRadius: 12, fontSize: "0.95rem", background: "rgba(0,0,0,0.3)", border: "2px solid #333", color: "#ddd", cursor: won ? "default" : "pointer" }}>{mission.instructions[i].icon} {mission.instructions[i].label}</button>)}
      </div>
      {!won && order.length > 0 && <button onClick={() => { setOrder([]); setRem(mission.instructions.map((_,i)=>i)); setMsg(""); }} style={{ marginTop: 10, background: "none", border: "1px solid #333", borderRadius: 8, padding: "5px 12px", color: "#555", fontSize: "0.8rem", cursor: "pointer" }}>🔄 Recommencer</button>}
      {msg && <div style={{ marginTop: 12, fontSize: "1rem", color: msg.includes("🎉") ? "#00ff88" : "#ff8844", fontWeight: 700 }}>{msg}</div>}
    </div>
  );
}

function ChoiceGame({ items, getQuestion, getOptions, getAnswer, onWin, visual }) {
  const [idx, setIdx] = useState(0);
  const [msg, setMsg] = useState("");
  const [won, setWon] = useState(false);
  const it = items[idx];
  const check = (v) => {
    if (v === getAnswer(it)) {
      if (idx+1 >= items.length) { setMsg("🎉 Tout réussi !"); setWon(true); setTimeout(onWin, 1200); }
      else { setMsg("✅ Exact !"); setTimeout(() => { setIdx(i=>i+1); setMsg(""); }, 800); }
    } else setMsg("❌ Réessaie !");
  };
  return (
    <div style={{ textAlign: "center" }}>
      <div style={{ fontSize: "0.75rem", color: "#555", marginBottom: 8 }}>{idx+1}/{items.length}</div>
      <div style={{ fontSize: "1.1rem", color: "#b44dff", marginBottom: 14, padding: "12px 18px", background: "rgba(180,77,255,0.06)", borderRadius: 12, border: "1px solid #b44dff22", fontWeight: 600, lineHeight: 1.5, whiteSpace: "pre-line" }}>{getQuestion(it)}</div>
      {visual && visual(it)}
      <div style={{ display: "flex", justifyContent: "center", gap: 10, flexWrap: "wrap" }}>
        {getOptions(it).map(o => <button key={o} onClick={() => check(o)} disabled={won} style={{ minWidth: 56, height: 56, borderRadius: 12, fontSize: "1.2rem", fontWeight: 700, background: "rgba(0,0,0,0.3)", border: "2px solid #333", color: "#fff", cursor: won ? "default" : "pointer", padding: "0 14px" }}>{o}</button>)}
      </div>
      {msg && <div style={{ marginTop: 12, fontSize: "1rem", color: msg.includes("🎉") || msg.includes("✅") ? "#00ff88" : "#ff8844", fontWeight: 700 }}>{msg}</div>}
    </div>
  );
}

function LoopGame({ mission, onWin }) {
  return <ChoiceGame items={mission.questions} getQuestion={it => it.q} getOptions={it => it.options} getAnswer={it => it.answer} onWin={onWin}
    visual={it => <div style={{ display: "flex", justifyContent: "center", gap: 5, marginBottom: 16, flexWrap: "wrap" }}>{Array.from({length: it.times}, (_,i) => <span key={i} style={{ fontSize: "1.6rem" }}>{it.action}</span>)}</div>} />;
}

function DecryptGame({ mission, onWin }) {
  return <ChoiceGame items={mission.puzzles}
    getQuestion={it => <><div style={{ fontSize: "1.5rem", color: "#00f0ff", marginBottom: 8, fontFamily: "monospace" }}>{it.equation}</div>{it.q}</>}
    getOptions={it => it.options} getAnswer={it => it.answer} onWin={onWin} />;
}

const GAME_MAP = { path: PathGame, sequence: SequenceGame, pattern: PatternGame, condition: ConditionGame, sort: SortGame, draw: DrawGame, loop: LoopGame, decrypt: DecryptGame };

// ============ MATRIX RAIN ============
function MatrixRain() {
  const ref = useRef(null);
  useEffect(() => {
    const c = ref.current; if (!c) return;
    const ctx = c.getContext("2d");
    c.width = c.parentElement.offsetWidth; c.height = c.parentElement.offsetHeight;
    const cols = Math.floor(c.width / 18); const drops = Array(cols).fill(1);
    const chars = "⭐🌟✨💫⚡🔥🌈🎮🚀🤖";
    const draw = () => {
      ctx.fillStyle = "rgba(10,10,18,0.08)"; ctx.fillRect(0,0,c.width,c.height);
      ctx.fillStyle = "#00ff8822"; ctx.font = "14px serif";
      for (let i=0; i<drops.length; i++) {
        ctx.fillText(chars[Math.floor(Math.random()*chars.length)], i*18, drops[i]*18);
        if (drops[i]*18 > c.height && Math.random()>0.98) drops[i]=0;
        drops[i]++;
      }
    };
    const id = setInterval(draw, 80); return () => clearInterval(id);
  }, []);
  return <canvas ref={ref} style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%", opacity: 0.18, pointerEvents: "none" }} />;
}

// ============ MAIN APP ============
export default function CodeNexus() {
  const [screen, setScreen] = useState("SETUP");
  const [students, setStudents] = useState([]);
  const [nameInput, setNameInput] = useState("");
  const [avatarInput, setAvatarInput] = useState("🚀");
  const [numTeams, setNumTeams] = useState(4);
  const [teamAssignments, setTeamAssignments] = useState({});
  const [activePlayer, setActivePlayer] = useState(null);
  const [currentMission, setCurrentMission] = useState(null);
  const [completedByTeam, setCompletedByTeam] = useState({});
  const [teamScores, setTeamScores] = useState({});
  const [chatMessages, setChatMessages] = useState([]);
  const [chatInput, setChatInput] = useState("");
  const [particles, setParticles] = useState([]);

  const activeTeams = TEAMS.slice(0, numTeams);

  const addStudent = () => {
    if (!nameInput.trim() || students.find(s => s.name.toLowerCase() === nameInput.trim().toLowerCase())) return;
    setStudents(prev => [...prev, { name: nameInput.trim(), avatar: avatarInput, xp: 0 }]);
    setNameInput("");
    setAvatarInput(AVATARS[Math.floor(Math.random() * AVATARS.length)]);
  };

  const removeStudent = (name) => setStudents(prev => prev.filter(s => s.name !== name));

  const distributeTeams = () => {
    if (students.length < 2) return;
    const shuffled = [...students].sort(() => Math.random() - 0.5);
    const assignments = {};
    const scores = {};
    const completed = {};
    activeTeams.forEach(t => { assignments[t.key] = []; scores[t.key] = 0; completed[t.key] = []; });
    shuffled.forEach((s, i) => {
      const teamKey = activeTeams[i % activeTeams.length].key;
      assignments[teamKey].push(s);
    });
    setTeamAssignments(assignments);
    setTeamScores(scores);
    setCompletedByTeam(completed);
    setChatMessages([{ from: "SYSTÈME", text: `🎮 ${students.length} élèves répartis dans ${activeTeams.length} équipes ! C'est parti !`, color: "#ffaa00" }]);
    setScreen("TEAMS_REVEAL");
  };

  const startPlaying = () => setScreen("SELECT_PLAYER");

  const selectPlayer = (player, teamKey) => {
    setActivePlayer({ ...player, teamKey });
    setScreen("HUB");
  };

  const spawnParticles = (color) => {
    setParticles(Array.from({length: 20}, (_, i) => ({ id: Date.now()+i, x: 30+Math.random()*40, y: 30+Math.random()*40, color, size: 4+Math.random()*8 })));
    setTimeout(() => setParticles([]), 1500);
  };

  const onMissionWin = () => {
    const m = currentMission;
    const tk = activePlayer.teamKey;
    const team = activeTeams.find(t => t.key === tk);
    setTeamScores(p => ({ ...p, [tk]: (p[tk]||0) + m.xp }));
    setCompletedByTeam(p => ({ ...p, [tk]: [...(p[tk]||[]), m.id] }));
    setStudents(prev => prev.map(s => s.name === activePlayer.name ? { ...s, xp: s.xp + m.xp } : s));
    spawnParticles(team.color);
    setChatMessages(p => [...p, { from: "SYSTÈME", text: `🏆 ${activePlayer.avatar} ${activePlayer.name} a réussi "${m.title}" ! +${m.xp} XP pour ${team.name} !`, color: team.color }]);
    setTimeout(() => setScreen("HUB"), 1500);
  };

  const sendChat = () => {
    if (!chatInput.trim() || !activePlayer) return;
    const team = activeTeams.find(t => t.key === activePlayer.teamKey);
    setChatMessages(p => [...p, { from: activePlayer.name, text: chatInput, color: team?.color || "#aaa" }]);
    setChatInput("");
  };

  const diffLabel = { CP: "⭐ Facile", CE1: "⭐⭐ Moyen", CE2: "⭐⭐⭐ Dur" };
  const diffColor = { CP: "#00ff88", CE1: "#ffaa00", CE2: "#ff2d6b" };

  const globalStyle = `
    @import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700;800;900&display=swap');
    @keyframes fadeIn { from { opacity: 0; transform: translateY(-20px); } to { opacity: 1; transform: translateY(0); } }
    @keyframes slideUp { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    @keyframes bounceIn { 0% { transform: scale(0); } 60% { transform: scale(1.1); } 100% { transform: scale(1); } }
    @keyframes particleBurst { 0% { transform: scale(1); opacity: 1; } 100% { transform: scale(0) translate(${Math.random()*100-50}px,${Math.random()*100-50}px); opacity: 0; } }
    * { box-sizing: border-box; } input::placeholder { color: #555; }
    ::-webkit-scrollbar { width: 5px; } ::-webkit-scrollbar-track { background: transparent; } ::-webkit-scrollbar-thumb { background: #222; border-radius: 3px; }
    button:hover:not(:disabled) { filter: brightness(1.15); }
  `;

  const Particles = () => particles.map(p => (
    <div key={p.id} style={{ position: "fixed", left: `${p.x}%`, top: `${p.y}%`, width: p.size, height: p.size, background: p.color, borderRadius: "50%", pointerEvents: "none", zIndex: 100, boxShadow: `0 0 ${p.size*2}px ${p.color}`, animation: "particleBurst 1s ease-out forwards" }} />
  ));

  const base = { minHeight: "100vh", background: "#0a0a12", color: "#e0e0e0", fontFamily: "'Nunito', sans-serif", position: "relative", overflow: "hidden" };

  // ====== SETUP : Inscription des élèves ======
  if (screen === "SETUP") {
    return (
      <div style={base}>
        <MatrixRain />
        <div style={{ position: "relative", zIndex: 2, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", minHeight: "100vh", padding: 20 }}>
          <div style={{ textAlign: "center", marginBottom: 24, animation: "fadeIn 0.8s ease" }}>
            <div style={{ fontSize: "2.4rem", fontWeight: 900, letterSpacing: "0.08em", background: "linear-gradient(135deg, #00f0ff, #b44dff, #ff2d6b, #00ff88)", WebkitBackgroundClip: "text", WebkitTextFillColor: "transparent" }}>CODE::NEXUS</div>
            <div style={{ fontSize: "1rem", color: "#888", marginTop: 6 }}>🚀 Inscris tous les élèves puis répartis-les en équipes !</div>
          </div>

          <div style={{ background: "rgba(15,15,25,0.92)", border: "1px solid #1a1a2e", borderRadius: 18, padding: 24, width: "100%", maxWidth: 550, backdropFilter: "blur(20px)" }}>
            {/* Add student */}
            <div style={{ marginBottom: 18 }}>
              <div style={{ fontSize: "0.9rem", color: "#999", marginBottom: 8, fontWeight: 700 }}>➕ Ajouter un élève</div>
              <div style={{ display: "flex", gap: 8, alignItems: "center", marginBottom: 10 }}>
                <input value={nameInput} onChange={e => setNameInput(e.target.value)} onKeyDown={e => e.key === "Enter" && addStudent()} placeholder="Prénom de l'élève..." maxLength={14} style={{ flex: 1, background: "rgba(0,0,0,0.4)", border: "2px solid #222", borderRadius: 10, padding: "12px 14px", color: "#00f0ff", fontSize: "1rem", fontFamily: "'Nunito', sans-serif", outline: "none" }} onFocus={e => e.target.style.borderColor = "#00f0ff55"} onBlur={e => e.target.style.borderColor = "#222"} />
                <button onClick={addStudent} style={{ background: "rgba(0,240,255,0.1)", border: "2px solid #00f0ff55", borderRadius: 10, padding: "12px 16px", color: "#00f0ff", fontSize: "1rem", fontWeight: 800, cursor: "pointer" }}>+ Ajouter</button>
              </div>
              <div style={{ display: "flex", flexWrap: "wrap", gap: 5 }}>
                {AVATARS.map(a => (
                  <button key={a} onClick={() => setAvatarInput(a)} style={{ width: 38, height: 38, borderRadius: 8, border: avatarInput === a ? "2px solid #00f0ff" : "1px solid #1a1a2e", background: avatarInput === a ? "rgba(0,240,255,0.12)" : "rgba(0,0,0,0.2)", fontSize: "1.2rem", cursor: "pointer", transition: "all 0.2s", transform: avatarInput === a ? "scale(1.15)" : "scale(1)" }}>{a}</button>
                ))}
              </div>
            </div>

            {/* Student list */}
            <div style={{ marginBottom: 18 }}>
              <div style={{ fontSize: "0.9rem", color: "#999", marginBottom: 8, fontWeight: 700 }}>👥 Élèves inscrits ({students.length})</div>
              {students.length === 0 ? (
                <div style={{ color: "#333", fontSize: "0.85rem", padding: 12, textAlign: "center" }}>Aucun élève pour le moment...</div>
              ) : (
                <div style={{ display: "flex", flexWrap: "wrap", gap: 8 }}>
                  {students.map((s, i) => (
                    <div key={s.name} style={{ display: "flex", alignItems: "center", gap: 6, background: "rgba(0,240,255,0.05)", border: "1px solid #1a1a2e", borderRadius: 10, padding: "8px 12px", animation: `bounceIn 0.3s ease ${i * 0.05}s both` }}>
                      <span style={{ fontSize: "1.2rem" }}>{s.avatar}</span>
                      <span style={{ fontSize: "0.85rem", color: "#ddd", fontWeight: 600 }}>{s.name}</span>
                      <button onClick={() => removeStudent(s.name)} style={{ background: "none", border: "none", color: "#ff2d6b88", cursor: "pointer", fontSize: "0.8rem", padding: "0 2px" }}>✕</button>
                    </div>
                  ))}
                </div>
              )}
            </div>

            {/* Team config */}
            <div style={{ marginBottom: 18 }}>
              <div style={{ fontSize: "0.9rem", color: "#999", marginBottom: 8, fontWeight: 700 }}>⚙️ Nombre d'équipes</div>
              <div style={{ display: "flex", gap: 8 }}>
                {[2, 3, 4].map(n => (
                  <button key={n} onClick={() => setNumTeams(n)} style={{ flex: 1, padding: "10px", borderRadius: 10, border: numTeams === n ? "2px solid #b44dff" : "2px solid #1a1a2e", background: numTeams === n ? "rgba(180,77,255,0.1)" : "rgba(0,0,0,0.2)", color: numTeams === n ? "#b44dff" : "#666", fontWeight: 800, fontSize: "1rem", cursor: "pointer" }}>
                    {n} équipes
                  </button>
                ))}
              </div>
              {students.length >= 2 && (
                <div style={{ fontSize: "0.75rem", color: "#555", marginTop: 6, textAlign: "center" }}>
                  ≈ {Math.floor(students.length / numTeams)}-{Math.ceil(students.length / numTeams)} élèves par équipe
                </div>
              )}
            </div>

            {/* Launch */}
            <button onClick={distributeTeams} disabled={students.length < 2} style={{
              width: "100%", padding: "16px", borderRadius: 14, fontSize: "1.1rem", fontWeight: 800,
              background: students.length >= 2 ? "linear-gradient(135deg, #00f0ff22, #b44dff22, #ff2d6b22)" : "rgba(0,0,0,0.2)",
              border: students.length >= 2 ? "2px solid #b44dff66" : "2px solid #1a1a2e",
              color: students.length >= 2 ? "#fff" : "#333",
              cursor: students.length >= 2 ? "pointer" : "not-allowed",
              transition: "all 0.3s", letterSpacing: "0.05em",
            }}>
              🚀 Répartir en équipes et JOUER !
            </button>
            {students.length < 2 && <div style={{ fontSize: "0.75rem", color: "#ff884488", textAlign: "center", marginTop: 6 }}>Il faut au moins 2 élèves</div>}
          </div>
        </div>
        <style>{globalStyle}</style>
      </div>
    );
  }

  // ====== TEAMS REVEAL ======
  if (screen === "TEAMS_REVEAL") {
    return (
      <div style={base}>
        <MatrixRain />
        <div style={{ position: "relative", zIndex: 2, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", minHeight: "100vh", padding: 20 }}>
          <div style={{ fontSize: "1.8rem", fontWeight: 900, marginBottom: 8, background: "linear-gradient(135deg, #00f0ff, #b44dff, #ff2d6b, #00ff88)", WebkitBackgroundClip: "text", WebkitTextFillColor: "transparent" }}>LES ÉQUIPES !</div>
          <div style={{ fontSize: "0.9rem", color: "#666", marginBottom: 28 }}>Voici la répartition de vos escouades</div>

          <div style={{ display: "grid", gridTemplateColumns: `repeat(${Math.min(activeTeams.length, 4)}, 1fr)`, gap: 16, width: "100%", maxWidth: 700, marginBottom: 28 }}>
            {activeTeams.map((team, ti) => (
              <div key={team.key} style={{ background: team.bg, border: `2px solid ${team.color}55`, borderRadius: 16, padding: 18, textAlign: "center", animation: `bounceIn 0.5s ease ${ti * 0.15}s both` }}>
                <div style={{ fontSize: "2rem", marginBottom: 4 }}>{team.icon}</div>
                <div style={{ fontSize: "0.95rem", color: team.color, fontWeight: 800, marginBottom: 10 }}>{team.name}</div>
                <div style={{ display: "flex", flexDirection: "column", gap: 6 }}>
                  {(teamAssignments[team.key] || []).map((s, i) => (
                    <div key={s.name} style={{ display: "flex", alignItems: "center", gap: 6, justifyContent: "center", animation: `slideUp 0.3s ease ${(ti * 0.15) + (i * 0.1)}s both` }}>
                      <span style={{ fontSize: "1.2rem" }}>{s.avatar}</span>
                      <span style={{ fontSize: "0.9rem", color: "#ddd", fontWeight: 600 }}>{s.name}</span>
                    </div>
                  ))}
                </div>
              </div>
            ))}
          </div>

          <div style={{ display: "flex", gap: 12 }}>
            <button onClick={() => { setScreen("SETUP"); setTeamAssignments({}); }} style={{ padding: "12px 24px", borderRadius: 12, background: "none", border: "2px solid #333", color: "#888", fontSize: "0.9rem", cursor: "pointer", fontWeight: 700 }}>← Modifier</button>
            <button onClick={distributeTeams} style={{ padding: "12px 24px", borderRadius: 12, background: "rgba(255,170,0,0.1)", border: "2px solid #ffaa0055", color: "#ffaa00", fontSize: "0.9rem", cursor: "pointer", fontWeight: 700 }}>🎲 Re-mélanger</button>
            <button onClick={startPlaying} style={{ padding: "12px 28px", borderRadius: 12, background: "linear-gradient(135deg, #00f0ff22, #00ff8822)", border: "2px solid #00ff8866", color: "#00ff88", fontSize: "1rem", cursor: "pointer", fontWeight: 800, letterSpacing: "0.03em" }}>▶ C'est parti !</button>
          </div>
        </div>
        <style>{globalStyle}</style>
      </div>
    );
  }

  // ====== SELECT PLAYER ======
  if (screen === "SELECT_PLAYER") {
    return (
      <div style={base}>
        <MatrixRain />
        <div style={{ position: "relative", zIndex: 2, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", minHeight: "100vh", padding: 20 }}>
          <div style={{ fontSize: "1.5rem", fontWeight: 800, color: "#00f0ff", marginBottom: 6 }}>🎮 Qui joue ?</div>
          <div style={{ fontSize: "0.85rem", color: "#666", marginBottom: 24 }}>Choisis l'élève qui va jouer cette manche</div>

          <div style={{ display: "grid", gridTemplateColumns: `repeat(${Math.min(activeTeams.length, 4)}, 1fr)`, gap: 14, width: "100%", maxWidth: 700 }}>
            {activeTeams.map(team => (
              <div key={team.key} style={{ background: "rgba(15,15,25,0.9)", border: `2px solid ${team.color}33`, borderRadius: 16, padding: 14, textAlign: "center" }}>
                <div style={{ fontSize: "1.4rem", marginBottom: 2 }}>{team.icon}</div>
                <div style={{ fontSize: "0.8rem", color: team.color, fontWeight: 800, marginBottom: 8 }}>{team.name}</div>
                <div style={{ fontSize: "0.7rem", color: "#555", marginBottom: 8 }}>{teamScores[team.key] || 0} XP</div>
                <div style={{ display: "flex", flexDirection: "column", gap: 6 }}>
                  {(teamAssignments[team.key] || []).map(s => {
                    const updated = students.find(st => st.name === s.name) || s;
                    return (
                      <button key={s.name} onClick={() => selectPlayer(updated, team.key)} style={{
                        display: "flex", alignItems: "center", gap: 8, padding: "10px 12px", borderRadius: 10,
                        background: "rgba(0,0,0,0.3)", border: `1px solid ${team.color}33`,
                        cursor: "pointer", transition: "all 0.2s", width: "100%",
                      }}>
                        <span style={{ fontSize: "1.3rem" }}>{updated.avatar}</span>
                        <div style={{ textAlign: "left" }}>
                          <div style={{ fontSize: "0.85rem", color: "#ddd", fontWeight: 700 }}>{updated.name}</div>
                          <div style={{ fontSize: "0.65rem", color: "#555" }}>{updated.xp} XP</div>
                        </div>
                      </button>
                    );
                  })}
                </div>
              </div>
            ))}
          </div>

          {/* Scoreboard mini */}
          <div style={{ marginTop: 24, display: "flex", gap: 10 }}>
            {activeTeams.sort((a, b) => (teamScores[b.key]||0) - (teamScores[a.key]||0)).map((t, i) => (
              <div key={t.key} style={{ padding: "6px 14px", borderRadius: 8, background: t.bg, border: `1px solid ${t.color}33`, textAlign: "center" }}>
                <span style={{ fontSize: "0.7rem", color: "#555" }}>{["🥇","🥈","🥉","#4"][i]} </span>
                <span style={{ fontSize: "0.8rem", color: t.color, fontWeight: 700 }}>{t.name}: {teamScores[t.key]||0}</span>
              </div>
            ))}
          </div>
        </div>
        <style>{globalStyle}</style>
      </div>
    );
  }

  // ====== HUB ======
  if (screen === "HUB" && activePlayer) {
    const team = activeTeams.find(t => t.key === activePlayer.teamKey);
    const myCompleted = completedByTeam[activePlayer.teamKey] || [];
    return (
      <div style={{ ...base, overflow: "hidden" }}>
        <MatrixRain />
        <Particles />
        {/* Top bar */}
        <div style={{ position: "relative", zIndex: 2, borderBottom: "1px solid #1a1a2e", padding: "10px 18px", display: "flex", justifyContent: "space-between", alignItems: "center", background: "rgba(10,10,18,0.95)" }}>
          <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
            <button onClick={() => { setActivePlayer(null); setScreen("SELECT_PLAYER"); }} style={{ background: "none", border: "1px solid #333", borderRadius: 8, padding: "5px 12px", color: "#888", cursor: "pointer", fontSize: "0.8rem" }}>← Changer</button>
            <span style={{ fontSize: "1.3rem" }}>{activePlayer.avatar}</span>
            <div>
              <div style={{ fontSize: "0.95rem", color: team.color, fontWeight: 800 }}>{activePlayer.name}</div>
              <div style={{ fontSize: "0.6rem", color: "#555" }}>{team.icon} {team.name}</div>
            </div>
          </div>
          <div style={{ fontSize: "0.8rem", color: team.color, fontWeight: 700 }}>{teamScores[team.key]||0} XP</div>
        </div>

        <div style={{ position: "relative", zIndex: 2, display: "grid", gridTemplateColumns: "1fr 240px", height: "calc(100vh - 50px)" }}>
          <div style={{ padding: 18, overflowY: "auto" }}>
            {/* Scoreboard */}
            <div style={{ marginBottom: 20 }}>
              <div style={{ fontSize: "0.8rem", color: "#666", marginBottom: 8, fontWeight: 700 }}>🏆 Classement</div>
              <div style={{ display: "flex", gap: 8 }}>
                {activeTeams.sort((a,b) => (teamScores[b.key]||0)-(teamScores[a.key]||0)).map((t,i) => (
                  <div key={t.key} style={{ flex: 1, background: t.key === team.key ? t.bg : "rgba(15,15,25,0.5)", border: `2px solid ${t.key === team.key ? t.color+"66" : "#1a1a2e"}`, borderRadius: 12, padding: 8, textAlign: "center" }}>
                    <div style={{ fontSize: "0.6rem", color: "#555" }}>{["🥇","🥈","🥉","#4"][i]}</div>
                    <div style={{ fontSize: "1rem" }}>{t.icon}</div>
                    <div style={{ fontSize: "0.6rem", color: t.color, fontWeight: 800 }}>{t.name}</div>
                    <div style={{ fontSize: "1rem", color: "#fff", fontWeight: 800 }}>{teamScores[t.key]||0}</div>
                  </div>
                ))}
              </div>
            </div>

            {/* Missions */}
            <div style={{ fontSize: "0.8rem", color: "#666", marginBottom: 10, fontWeight: 700 }}>🎯 Missions</div>
            <div style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 10 }}>
              {MISSIONS.map(m => {
                const done = myCompleted.includes(m.id);
                return (
                  <button key={m.id} onClick={() => { if (!done) { setCurrentMission(m); setScreen("MISSION"); }}} disabled={done} style={{ background: done ? "rgba(0,255,136,0.04)" : "rgba(15,15,25,0.7)", border: `2px solid ${done ? "#00ff8833" : "#1a1a2e"}`, borderRadius: 14, padding: 12, textAlign: "center", cursor: done ? "default" : "pointer", transition: "all 0.3s", opacity: done ? 0.5 : 1, position: "relative" }}>
                    {done && <div style={{ position: "absolute", top: 6, right: 8, color: "#00ff88", fontSize: "0.7rem" }}>✅</div>}
                    <div style={{ fontSize: "1.4rem", marginBottom: 3 }}>{m.icon}</div>
                    <div style={{ fontSize: "0.8rem", color: "#eee", fontWeight: 800, marginBottom: 3 }}>{m.title}</div>
                    <span style={{ fontSize: "0.6rem", color: diffColor[m.difficulty], fontWeight: 700, padding: "2px 6px", background: diffColor[m.difficulty]+"15", borderRadius: 5 }}>{diffLabel[m.difficulty]}</span>
                    <div style={{ fontSize: "0.6rem", color: "#555", marginTop: 3 }}>{m.xp} XP</div>
                  </button>
                );
              })}
            </div>
          </div>

          {/* Sidebar */}
          <div style={{ borderLeft: "1px solid #1a1a2e", display: "flex", flexDirection: "column", background: "rgba(10,10,18,0.95)" }}>
            <div style={{ padding: 10, borderBottom: "1px solid #1a1a2e" }}>
              <div style={{ fontSize: "0.75rem", color: "#666", marginBottom: 6, fontWeight: 700 }}>👥 {team.name}</div>
              {(teamAssignments[team.key]||[]).map((s,i) => {
                const up = students.find(st => st.name === s.name) || s;
                return (
                  <div key={i} style={{ display: "flex", alignItems: "center", gap: 6, padding: "4px 0", borderBottom: "1px solid #111" }}>
                    <span style={{ fontSize: "1rem" }}>{up.avatar}</span>
                    <div>
                      <div style={{ fontSize: "0.75rem", color: up.name === activePlayer.name ? team.color : "#bbb", fontWeight: up.name === activePlayer.name ? 800 : 400 }}>{up.name}{up.name === activePlayer.name ? " 🎮" : ""}</div>
                      <div style={{ fontSize: "0.55rem", color: "#444" }}>{up.xp} XP</div>
                    </div>
                  </div>
                );
              })}
            </div>
            <div style={{ flex: 1, display: "flex", flexDirection: "column", overflow: "hidden" }}>
              <div style={{ fontSize: "0.75rem", color: "#666", padding: "8px 10px 4px", fontWeight: 700 }}>💬 Chat</div>
              <div style={{ flex: 1, overflowY: "auto", padding: "0 10px 6px" }}>
                {chatMessages.map((m,i) => (
                  <div key={i} style={{ marginBottom: 3 }}>
                    <span style={{ fontSize: "0.65rem", color: m.color, fontWeight: 700 }}>{m.from}: </span>
                    <span style={{ fontSize: "0.7rem", color: m.from === "SYSTÈME" ? m.color : "#aaa" }}>{m.text}</span>
                  </div>
                ))}
              </div>
              <div style={{ padding: "6px 8px", borderTop: "1px solid #1a1a2e", display: "flex", gap: 4 }}>
                <input value={chatInput} onChange={e => setChatInput(e.target.value)} onKeyDown={e => e.key === "Enter" && sendChat()} placeholder="Message..." style={{ flex: 1, background: "rgba(0,0,0,0.4)", border: "1px solid #1a1a2e", borderRadius: 6, padding: "6px 8px", color: "#ddd", fontSize: "0.75rem", fontFamily: "'Nunito', sans-serif", outline: "none" }} />
                <button onClick={sendChat} style={{ background: team.bg, border: `1px solid ${team.color}44`, borderRadius: 6, padding: "5px 10px", color: team.color, fontSize: "0.75rem", cursor: "pointer", fontWeight: 700 }}>▶</button>
              </div>
            </div>
          </div>
        </div>
        <style>{globalStyle}</style>
      </div>
    );
  }

  // ====== MISSION ======
  if (screen === "MISSION" && currentMission && activePlayer) {
    const team = activeTeams.find(t => t.key === activePlayer.teamKey);
    const m = currentMission;
    const GameComp = GAME_MAP[m.type];
    return (
      <div style={{ ...base, display: "flex", flexDirection: "column" }}>
        <Particles />
        <div style={{ borderBottom: "1px solid #1a1a2e", padding: "10px 18px", display: "flex", justifyContent: "space-between", alignItems: "center", background: "rgba(10,10,18,0.95)", position: "relative", zIndex: 2 }}>
          <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
            <button onClick={() => setScreen("HUB")} style={{ background: "none", border: "1px solid #333", borderRadius: 8, padding: "5px 12px", color: "#888", cursor: "pointer", fontSize: "0.8rem" }}>← Retour</button>
            <span style={{ fontSize: "1.2rem" }}>{m.icon}</span>
            <span style={{ fontSize: "0.95rem", color: diffColor[m.difficulty], fontWeight: 800 }}>{m.title}</span>
          </div>
          <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
            <span style={{ fontSize: "0.8rem", color: "#888" }}>{activePlayer.avatar} {activePlayer.name}</span>
            <span style={{ fontSize: "0.7rem", color: diffColor[m.difficulty], padding: "3px 8px", border: `1px solid ${diffColor[m.difficulty]}33`, borderRadius: 6, fontWeight: 700 }}>{diffLabel[m.difficulty]} • {m.xp} XP</span>
          </div>
        </div>
        <div style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", padding: 20, position: "relative", zIndex: 2 }}>
          <div style={{ background: "rgba(15,15,25,0.92)", border: "1px solid #1a1a2e", borderRadius: 18, padding: 24, maxWidth: 600, width: "100%", backdropFilter: "blur(20px)" }}>
            <div style={{ fontSize: "0.95rem", color: "#aaa", marginBottom: 18, textAlign: "center", lineHeight: 1.5 }}>{m.description}</div>
            {GameComp && <GameComp mission={m} onWin={onMissionWin} teamColor={team.color} />}
          </div>
        </div>
        <style>{globalStyle}</style>
      </div>
    );
  }

  return null;
}
