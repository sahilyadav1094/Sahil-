import { useState } from "react";

// ── 2026 DIFFICULTY CALIBRATION ──────────────────────────────
// 2024 = Easy  → high marks, very competitive top ranks
// 2025 = Hard  → low marks, relaxed cutoffs
// 2026 = Medium → blended estimate; rank shown as RANGE ±12%
// ─────────────────────────────────────────────────────────────

const getRankRange = (marks) => {
  let base;
  if (marks >= 715) base = 10;
  else if (marks >= 710) base = 35;
  else if (marks >= 705) base = 80;
  else if (marks >= 700) base = 160;
  else if (marks >= 695) base = 280;
  else if (marks >= 690) base = 430;
  else if (marks >= 685) base = 620;
  else if (marks >= 680) base = 870;
  else if (marks >= 675) base = 1180;
  else if (marks >= 670) base = 1550;
  else if (marks >= 665) base = 2000;
  else if (marks >= 660) base = 2600;
  else if (marks >= 655) base = 3400;
  else if (marks >= 650) base = 4400;
  else if (marks >= 645) base = 5600;
  else if (marks >= 640) base = 7100;
  else if (marks >= 635) base = 8800;
  else if (marks >= 630) base = 10800;
  else if (marks >= 625) base = 13200;
  else if (marks >= 620) base = 15900;
  else if (marks >= 615) base = 18900;
  else if (marks >= 610) base = 22300;
  else if (marks >= 605) base = 26000;
  else if (marks >= 600) base = 30200;
  else if (marks >= 595) base = 34500;
  else if (marks >= 590) base = 39300;
  else if (marks >= 585) base = 44500;
  else if (marks >= 580) base = 50200;
  else if (marks >= 575) base = 56300;
  else if (marks >= 570) base = 62900;
  else if (marks >= 565) base = 70000;
  else if (marks >= 560) base = 77700;
  else if (marks >= 555) base = 86000;
  else if (marks >= 550) base = 95000;
  else if (marks >= 540) base = 114000;
  else if (marks >= 530) base = 134000;
  else if (marks >= 520) base = 156000;
  else if (marks >= 510) base = 179000;
  else if (marks >= 500) base = 204000;
  else if (marks >= 490) base = 231000;
  else if (marks >= 480) base = 259000;
  else if (marks >= 470) base = 289000;
  else if (marks >= 460) base = 321000;
  else if (marks >= 450) base = 354000;
  else if (marks >= 440) base = 390000;
  else if (marks >= 430) base = 428000;
  else if (marks >= 420) base = 468000;
  else if (marks >= 410) base = 510000;
  else base = 555000;

  const spread = Math.max(50, Math.round(base * 0.12));
  const round = (n) => {
    if (n < 500) return Math.round(n / 10) * 10;
    if (n < 5000) return Math.round(n / 50) * 50;
    if (n < 50000) return Math.round(n / 500) * 500;
    return Math.round(n / 1000) * 1000;
  };
  return { low: round(Math.max(1, base - spread)), high: round(base + spread) };
};

const getPercentile = (marks) => {
  if (marks >= 715) return 99.99;
  if (marks >= 700) return 99.95;
  if (marks >= 690) return 99.88;
  if (marks >= 680) return 99.75;
  if (marks >= 670) return 99.55;
  if (marks >= 660) return 99.25;
  if (marks >= 650) return 98.85;
  if (marks >= 640) return 98.25;
  if (marks >= 630) return 97.45;
  if (marks >= 620) return 96.40;
  if (marks >= 610) return 95.00;
  if (marks >= 600) return 93.20;
  if (marks >= 590) return 91.00;
  if (marks >= 580) return 88.20;
  if (marks >= 570) return 85.00;
  if (marks >= 560) return 81.40;
  if (marks >= 550) return 77.20;
  if (marks >= 540) return 72.40;
  if (marks >= 530) return 67.00;
  if (marks >= 520) return 61.00;
  if (marks >= 510) return 54.50;
  if (marks >= 500) return 47.50;
  if (marks >= 480) return 37.00;
  if (marks >= 460) return 27.50;
  if (marks >= 440) return 19.00;
  if (marks >= 420) return 12.50;
  return 7.00;
};

const COLLEGES = [
  { name: "AIIMS New Delhi", type: "Government", state: "Delhi", minRank: 1, maxRank: 80, fees: "₹1,628/year", seats: 107 },
  { name: "AIIMS Mumbai", type: "Government", state: "Maharashtra", minRank: 80, maxRank: 280, fees: "₹1,628/year", seats: 100 },
  { name: "Maulana Azad Medical College", type: "Government", state: "Delhi", minRank: 100, maxRank: 450, fees: "₹10,000/year", seats: 250 },
  { name: "AIIMS Jodhpur", type: "Government", state: "Rajasthan", minRank: 280, maxRank: 700, fees: "₹1,628/year", seats: 100 },
  { name: "AIIMS Bhopal", type: "Government", state: "MP", minRank: 370, maxRank: 900, fees: "₹1,628/year", seats: 100 },
  { name: "AIIMS Rishikesh", type: "Government", state: "Uttarakhand", minRank: 430, maxRank: 1100, fees: "₹1,628/year", seats: 100 },
  { name: "VMMC & Safdarjung Hospital", type: "Government", state: "Delhi", minRank: 430, maxRank: 1400, fees: "₹5,000/year", seats: 100 },
  { name: "AIIMS Nagpur", type: "Government", state: "Maharashtra", minRank: 500, maxRank: 1400, fees: "₹1,628/year", seats: 100 },
  { name: "AIIMS Patna", type: "Government", state: "Bihar", minRank: 600, maxRank: 1600, fees: "₹1,628/year", seats: 100 },
  { name: "AIIMS Raipur", type: "Government", state: "Chhattisgarh", minRank: 700, maxRank: 1900, fees: "₹1,628/year", seats: 100 },
  { name: "Lady Hardinge Medical College", type: "Government", state: "Delhi", minRank: 900, maxRank: 2800, fees: "₹10,000/year", seats: 150 },
  { name: "GMCH Chandigarh", type: "Government", state: "Chandigarh", minRank: 1800, maxRank: 6500, fees: "₹20,000/year", seats: 100 },
  { name: "Grant Medical College Mumbai", type: "Government", state: "Maharashtra", minRank: 700, maxRank: 2400, fees: "₹16,000/year", seats: 250 },
  { name: "King George Medical University", type: "Government", state: "UP", minRank: 900, maxRank: 2800, fees: "₹25,000/year", seats: 250 },
  { name: "Madras Medical College", type: "Government", state: "Tamil Nadu", minRank: 1400, maxRank: 5500, fees: "₹12,000/year", seats: 250 },
  { name: "SMS Medical College Jaipur", type: "Government", state: "Rajasthan", minRank: 1100, maxRank: 3800, fees: "₹15,000/year", seats: 250 },
  { name: "BJ Medical College Ahmedabad", type: "Government", state: "Gujarat", minRank: 1400, maxRank: 4800, fees: "₹30,000/year", seats: 180 },
  { name: "Bangalore Medical College", type: "Government", state: "Karnataka", minRank: 1800, maxRank: 5600, fees: "₹30,000/year", seats: 250 },
  { name: "Pt. BD Sharma PGIMS Rohtak", type: "Government", state: "Haryana", minRank: 2800, maxRank: 9500, fees: "₹20,000/year", seats: 150 },
  { name: "Osmania Medical College", type: "Government", state: "Telangana", minRank: 2300, maxRank: 6800, fees: "₹25,000/year", seats: 200 },
  { name: "Govt Medical College Kozhikode", type: "Government", state: "Kerala", minRank: 2800, maxRank: 8500, fees: "₹10,000/year", seats: 200 },
  { name: "RG Kar Medical College Kolkata", type: "Government", state: "WB", minRank: 3800, maxRank: 14000, fees: "₹8,000/year", seats: 200 },
  { name: "Stanley Medical College Chennai", type: "Government", state: "Tamil Nadu", minRank: 3800, maxRank: 11500, fees: "₹12,000/year", seats: 200 },
  { name: "ESIC Medical College Hyderabad", type: "Government", state: "Telangana", minRank: 4500, maxRank: 17000, fees: "₹1,000/year", seats: 100 },
  { name: "NRS Medical College Kolkata", type: "Government", state: "WB", minRank: 5500, maxRank: 19000, fees: "₹8,000/year", seats: 150 },
  { name: "Govt Medical College Nagpur", type: "Government", state: "Maharashtra", minRank: 4800, maxRank: 17000, fees: "₹20,000/year", seats: 200 },
  { name: "Christian Medical College Vellore", type: "Deemed", state: "Tamil Nadu", minRank: 1800, maxRank: 14000, fees: "₹5 L/year", seats: 100 },
  { name: "Sri Venkateswara Medical College", type: "Government", state: "AP", minRank: 6500, maxRank: 24000, fees: "₹15,000/year", seats: 150 },
  { name: "Gandhi Medical College Bhopal", type: "Government", state: "MP", minRank: 5500, maxRank: 21000, fees: "₹18,000/year", seats: 150 },
  { name: "MGM Medical College Indore", type: "Government", state: "MP", minRank: 7500, maxRank: 27000, fees: "₹18,000/year", seats: 150 },
  { name: "Govt Medical College Aurangabad", type: "Government", state: "Maharashtra", minRank: 7500, maxRank: 24000, fees: "₹20,000/year", seats: 150 },
  { name: "Amrita Institute of Medical Sciences", type: "Deemed", state: "Kerala", minRank: 7500, maxRank: 33000, fees: "₹18 L/year", seats: 150 },
  { name: "Kasturba Medical College Manipal", type: "Private", state: "Karnataka", minRank: 4800, maxRank: 19000, fees: "₹24 L/year", seats: 250 },
  { name: "Sri Ramachandra Medical College", type: "Deemed", state: "Tamil Nadu", minRank: 9500, maxRank: 38000, fees: "₹14 L/year", seats: 150 },
  { name: "Rangaraya Medical College", type: "Government", state: "AP", minRank: 9500, maxRank: 33000, fees: "₹15,000/year", seats: 150 },
  { name: "Govt Medical College Srinagar", type: "Government", state: "J&K", minRank: 9500, maxRank: 38000, fees: "₹10,000/year", seats: 100 },
  { name: "Assam Medical College Dibrugarh", type: "Government", state: "Assam", minRank: 11500, maxRank: 48000, fees: "₹8,000/year", seats: 120 },
  { name: "JSS Medical College Mysore", type: "Deemed", state: "Karnataka", minRank: 14000, maxRank: 58000, fees: "₹16 L/year", seats: 150 },
  { name: "RIMS Imphal", type: "Government", state: "Manipur", minRank: 14000, maxRank: 58000, fees: "₹5,000/year", seats: 100 },
  { name: "Hamdard Instt of Medical Sciences", type: "Deemed", state: "Delhi", minRank: 19000, maxRank: 75000, fees: "₹12 L/year", seats: 100 },
  { name: "Central Institute of Psychiatry", type: "Government", state: "Jharkhand", minRank: 19000, maxRank: 75000, fees: "₹5,000/year", seats: 50 },
  { name: "Shri Ram Murti Smarak Inst.", type: "Private", state: "UP", minRank: 28000, maxRank: 95000, fees: "₹12 L/year", seats: 150 },
  { name: "People's College of Medical Sciences", type: "Private", state: "MP", minRank: 38000, maxRank: 115000, fees: "₹10 L/year", seats: 150 },
  { name: "Index Medical College Indore", type: "Private", state: "MP", minRank: 50000, maxRank: 150000, fees: "₹9 L/year", seats: 150 },
  { name: "Pacific Medical College Udaipur", type: "Private", state: "Rajasthan", minRank: 60000, maxRank: 180000, fees: "₹11 L/year", seats: 150 },
];

const CATEGORIES = {
  General: { minMark: 145 }, OBC: { minMark: 113 },
  SC: { minMark: 113 }, ST: { minMark: 113 },
  EWS: { minMark: 145 }, "General-PH": { minMark: 129 },
};

const STATES = ["All India","Delhi","Maharashtra","Karnataka","Tamil Nadu","Kerala","UP","Rajasthan","Gujarat","MP","Telangana","AP","WB","Haryana","Chandigarh","J&K","Assam","Manipur","Jharkhand","Uttarakhand","Bihar","Chhattisgarh"];

const fmt = (n) => n.toLocaleString("en-IN");

const scoreColor = (marks) => {
  if (marks >= 650) return "#00d084";
  if (marks >= 550) return "#ffd60a";
  if (marks >= 450) return "#ff9f43";
  return "#ff6b6b";
};

export default function NEETPredictor() {
  const [totalMarks, setTotalMarks] = useState("");
  const [category, setCategory] = useState("General");
  const [state, setState] = useState("All India");
  const [result, setResult] = useState(null);
  const [filter, setFilter] = useState("All");

  const predict = () => {
    const m = parseInt(totalMarks) || 0;
    if (!m || m < 0 || m > 720) return;
    const { low, high } = getRankRange(m);
    setResult({ marks: m, low, high, percentile: getPercentile(m) });
    setFilter("All");
  };

  const reset = () => {
    setResult(null); setTotalMarks("");
  };

  const qualifies = result && result.marks >= (CATEGORIES[category]?.minMark || 145);

  const colleges = () => {
    if (!result) return [];
    const mid = Math.round((result.low + result.high) / 2);
    let list = COLLEGES.filter(c => mid >= c.minRank && mid <= c.maxRank);
    if (filter !== "All") list = list.filter(c => c.type === filter);
    if (state !== "All India") list = list.filter(c => c.state === state);
    return list.sort((a, b) => a.minRank - b.minRank);
  };

  const S = {
    wrap: { minHeight: "100vh", background: "#080c18", color: "#e8f4fd", fontFamily: "'Segoe UI',system-ui,sans-serif" },
    header: { background: "linear-gradient(90deg,#003566,#001d3d)", borderBottom: "2px solid #00b4d8", padding: "16px 22px", display: "flex", alignItems: "center", gap: 14 },
    icon: { width: 44, height: 44, background: "linear-gradient(135deg,#00b4d8,#0077b6)", borderRadius: 11, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 20 },
    diffBar: { background: "rgba(0,60,100,0.3)", borderBottom: "1px solid rgba(0,180,216,0.2)", padding: "8px 22px", display: "flex", gap: 22, justifyContent: "center", flexWrap: "wrap" },
    body: { maxWidth: 740, margin: "0 auto", padding: "22px 14px" },
    card: { background: "rgba(0,25,55,0.7)", borderRadius: 16, padding: "20px 18px", border: "1px solid rgba(0,180,216,0.18)", marginBottom: 16 },
    label: { display: "block", color: "#90e0ef", fontWeight: 700, marginBottom: 7, fontSize: 11, textTransform: "uppercase", letterSpacing: 1 },
    input: { width: "100%", padding: "12px 14px", borderRadius: 10, background: "#001730", border: "2px solid #0077b6", color: "#e8f4fd", fontSize: 20, fontWeight: 800, outline: "none", boxSizing: "border-box" },
    select: { width: "100%", padding: "9px 10px", borderRadius: 8, background: "#001730", border: "1px solid #0077b6", color: "#e8f4fd", fontSize: 13, outline: "none" },
    btn: { width: "100%", padding: "14px", borderRadius: 12, border: "none", background: "linear-gradient(135deg,#0077b6,#00b4d8)", color: "#fff", fontSize: 16, fontWeight: 900, cursor: "pointer", boxShadow: "0 6px 24px rgba(0,180,216,0.3)", letterSpacing: 0.5 },
    tag: (type) => ({
      display: "inline-block", padding: "3px 10px", borderRadius: 20, fontSize: 11, fontWeight: 700,
      background: type === "Government" ? "rgba(0,160,80,0.18)" : type === "Deemed" ? "rgba(200,150,0,0.18)" : "rgba(0,100,200,0.18)",
      color: type === "Government" ? "#00d084" : type === "Deemed" ? "#ffd60a" : "#00b4d8",
      border: `1px solid ${type === "Government" ? "#00d08433" : type === "Deemed" ? "#ffd60a33" : "#00b4d833"}`,
    }),
  };

  return (
    <div style={S.wrap}>
      {/* Header */}
      <div style={S.header}>
        <div style={S.icon}>🩺</div>
        <div>
          <div style={{ fontSize: 19, fontWeight: 900, color: "#90e0ef" }}>NEET 2026 Rank Predictor</div>
          <div style={{ fontSize: 10, color: "#48cae4", letterSpacing: 2, textTransform: "uppercase" }}>Medium Difficulty Calibrated • Rank as Range</div>
        </div>
      </div>

      {/* Difficulty bar */}
      <div style={S.diffBar}>
        {[["2024","Easy 😊","#00d084",false],["2025","Hard 😰","#ff6b6b",false],["2026","Medium 😐 (Est.)","#ffd60a",true]].map(([yr,lbl,clr,active]) => (
          <div key={yr} style={{ display: "flex", alignItems: "center", gap: 7, opacity: active ? 1 : 0.5 }}>
            <div style={{ width: 8, height: 8, borderRadius: "50%", background: clr, boxShadow: active ? `0 0 8px ${clr}` : "none" }} />
            <span style={{ color: clr, fontWeight: active ? 800 : 500, fontSize: 12 }}>{yr}: {lbl}</span>
          </div>
        ))}
      </div>

      <div style={S.body}>
        {!result ? (
          /* INPUT */
          <>
            <div style={{ textAlign: "center", marginBottom: 24 }}>
              <div style={{ fontSize: 24, fontWeight: 900, color: "#90e0ef" }}>Expected Score Dalein</div>
              <div style={{ color: "#48cae4", fontSize: 12, marginTop: 3 }}>2026 medium paper ke hisaab se rank range milegi</div>
            </div>

            <div style={S.card}>
              <label style={S.label}>Total Marks (0–720)</label>
              <input type="number" min="0" max="720" value={totalMarks}
                onChange={e => setTotalMarks(e.target.value)} placeholder="jaise ki 580" style={S.input} />
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 12, marginBottom: 18 }}>
              {[["Category", category, setCategory, Object.keys(CATEGORIES)],
                ["State", state, setState, STATES]].map(([lbl, val, setVal, opts]) => (
                <div key={lbl} style={S.card}>
                  <label style={S.label}>{lbl}</label>
                  <select value={val} onChange={e => setVal(e.target.value)} style={S.select}>
                    {opts.map(o => <option key={o} value={o}>{o}</option>)}
                  </select>
                </div>
              ))}
            </div>

            <button onClick={predict} style={S.btn}>🔮 2026 Rank Predict Karein</button>
          </>
        ) : (
          /* RESULT */
          <>
            {/* Qualify banner */}
            <div style={{
              background: qualifies ? "rgba(0,100,60,0.22)" : "rgba(120,0,0,0.22)",
              border: `1px solid ${qualifies ? "#00d084" : "#ff4444"}`,
              borderRadius: 14, padding: "12px 16px", marginBottom: 16,
              display: "flex", alignItems: "center", gap: 12,
            }}>
              <span style={{ fontSize: 24 }}>{qualifies ? "✅" : "❌"}</span>
              <div>
                <div style={{ fontWeight: 800, color: qualifies ? "#00d084" : "#ff6b6b", fontSize: 14 }}>
                  {qualifies ? "Eligible! Cutoff Clear Ho Gayi" : "Cutoff Qualify Nahi Hua"}
                </div>
                <div style={{ color: "#90e0ef", fontSize: 11 }}>
                  {category} min marks (2026 est.): {CATEGORIES[category]?.minMark}
                </div>
              </div>
            </div>

            {/* ★ RANK RANGE — Hero */}
            <div style={{
              background: "linear-gradient(135deg,rgba(0,35,75,0.95),rgba(0,18,45,0.95))",
              border: "2px solid #00b4d8", borderRadius: 18, padding: "22px 18px",
              marginBottom: 16, textAlign: "center",
              boxShadow: "0 0 40px rgba(0,180,216,0.12)",
            }}>
              <div style={{ fontSize: 11, color: "#48cae4", textTransform: "uppercase", letterSpacing: 2, marginBottom: 6 }}>
                🏆 Predicted Rank Range — NEET 2026
              </div>
              <div style={{ fontSize: 44, fontWeight: 900, color: "#ffd60a", textShadow: "0 0 28px rgba(255,214,10,0.35)", lineHeight: 1.1 }}>
                {fmt(result.low)} – {fmt(result.high)}
              </div>
              <div style={{ color: "#90e0ef", fontSize: 12, marginTop: 5, marginBottom: 16 }}>
                Medium paper ke liye calibrated range (±12% uncertainty)
              </div>
              <div style={{ display: "flex", justifyContent: "center", gap: 36 }}>
                <div>
                  <div style={{ fontSize: 10, color: "#48cae4", textTransform: "uppercase", letterSpacing: 1 }}>Marks</div>
                  <div style={{ fontSize: 26, fontWeight: 900, color: scoreColor(result.marks) }}>{result.marks}/720</div>
                </div>
                <div style={{ width: 1, background: "rgba(0,180,216,0.25)" }} />
                <div>
                  <div style={{ fontSize: 10, color: "#48cae4", textTransform: "uppercase", letterSpacing: 1 }}>Percentile</div>
                  <div style={{ fontSize: 26, fontWeight: 900, color: "#00d084" }}>{result.percentile}%ile</div>
                </div>
              </div>
            </div>

            {/* Score bar */}
            <div style={{ ...S.card, padding: "14px 16px" }}>
              <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 7, fontSize: 12 }}>
                <span style={{ color: "#90e0ef", fontWeight: 700 }}>Score Progress</span>
                <span style={{ color: scoreColor(result.marks), fontWeight: 800 }}>{result.marks} / 720</span>
              </div>
              <div style={{ height: 11, background: "#001730", borderRadius: 6, overflow: "hidden" }}>
                <div style={{ height: "100%", width: `${(result.marks/720)*100}%`, background: `linear-gradient(90deg,#0077b6,${scoreColor(result.marks)})`, borderRadius: 6 }} />
              </div>
              <div style={{ display: "flex", justifyContent: "space-between", marginTop: 5, fontSize: 10, color: "#48cae4" }}>
                {[0,180,360,540,720].map(v => <span key={v}>{v}</span>)}
              </div>
            </div>

            {/* Calibration note */}
            <div style={{ background: "rgba(255,214,10,0.05)", border: "1px solid rgba(255,214,10,0.18)", borderRadius: 12, padding: "11px 14px", marginBottom: 16, fontSize: 12, color: "#90e0ef", lineHeight: 1.7 }}>
              <span style={{ color: "#ffd60a", fontWeight: 800 }}>📊 2026 Calibration Logic: </span>
              2024 (easy) mein same marks pe rank zyada competitive thi. 2025 (hard) mein cutoff giri. 2026 medium paper mein dono ka average lagaya gaya hai.
            </div>

            {/* Colleges */}
            <div style={{ marginBottom: 12 }}>
              <div style={{ fontWeight: 800, color: "#90e0ef", fontSize: 14, marginBottom: 10 }}>🏥 Possible Colleges</div>
              <div style={{ display: "flex", gap: 7, flexWrap: "wrap", marginBottom: 12 }}>
                {["All","Government","Deemed","Private"].map(f => (
                  <button key={f} onClick={() => setFilter(f)} style={{
                    padding: "5px 13px", borderRadius: 20, cursor: "pointer", fontWeight: 700, fontSize: 12,
                    border: `1px solid ${filter === f ? "transparent" : "#0077b633"}`,
                    background: filter === f ? "linear-gradient(135deg,#0077b6,#00b4d8)" : "rgba(0,25,55,0.8)",
                    color: filter === f ? "#fff" : "#48cae4",
                  }}>{f}</button>
                ))}
              </div>
            </div>

            <div style={{ display: "flex", flexDirection: "column", gap: 9, marginBottom: 22 }}>
              {colleges().length === 0 ? (
                <div style={{ textAlign: "center", padding: 32, color: "#48cae4", background: "rgba(0,25,55,0.4)", borderRadius: 14 }}>
                  <div style={{ fontSize: 32, marginBottom: 8 }}>🔍</div>
                  <div style={{ fontWeight: 700, marginBottom: 3 }}>Koi college nahi mila</div>
                  <div style={{ fontSize: 12 }}>Filter ya state change karein</div>
                </div>
              ) : colleges().map((col, i) => (
                <div key={i} style={{
                  background: "rgba(0,20,45,0.85)", borderRadius: 12, padding: "13px 15px",
                  border: "1px solid rgba(0,180,216,0.12)",
                  display: "grid", gridTemplateColumns: "1fr auto", gap: 8, alignItems: "start",
                }}>
                  <div>
                    <div style={{ fontWeight: 700, color: "#caf0f8", marginBottom: 4, fontSize: 13 }}>{col.name}</div>
                    <div style={{ display: "flex", gap: 10, flexWrap: "wrap", marginBottom: 4 }}>
                      <span style={{ fontSize: 11, color: "#48cae4" }}>📍 {col.state}</span>
                      <span style={{ fontSize: 11, color: "#48cae4" }}>💺 {col.seats} seats</span>
                      <span style={{ fontSize: 11, color: "#90e0ef", fontWeight: 600 }}>💰 {col.fees}</span>
                    </div>
                    <div style={{ fontSize: 11, color: "#48cae4" }}>
                      Closing rank: <strong style={{ color: "#90e0ef" }}>{fmt(col.minRank)} – {fmt(col.maxRank)}</strong>
                    </div>
                  </div>
                  <span style={S.tag(col.type)}>{col.type}</span>
                </div>
              ))}
            </div>

            {/* Disclaimer */}
            <div style={{ background: "rgba(255,200,0,0.05)", border: "1px solid rgba(255,200,0,0.18)", borderRadius: 12, padding: "12px 14px", marginBottom: 18 }}>
              <div style={{ color: "#ffd60a", fontWeight: 700, fontSize: 12, marginBottom: 4 }}>⚠️ Disclaimer</div>
              <div style={{ color: "#90e0ef", fontSize: 11, lineHeight: 1.7 }}>
                Yeh prediction 2024 (easy) aur 2025 (hard) ke historical data se 2026 ke liye estimate hai. Actual rank NTA ke scoring, normalization, aur official counselling par depend karti hai. MCC / State Counselling website zaroor check karein.
              </div>
            </div>

            <button onClick={reset} style={{ ...S.btn, background: "transparent", border: "2px solid #0077b6", color: "#00b4d8", boxShadow: "none" }}>
              🔄 Dobara Try Karein
            </button>
          </>
        )}
      </div>

      <style>{`
        input[type=number]::-webkit-inner-spin-button,
        input[type=number]::-webkit-outer-spin-button { -webkit-appearance: none; }
        select option { background: #001730; }
        * { box-sizing: border-box; }
      `}</style>
    </div>
  );
}
