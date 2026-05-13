# Artifact Template Reference

Use this as the blueprint when generating the animated dependency graph artifact.
Replace NODES_DATA and EDGES_DATA with actual scanned project data.

## Full React JSX Template

```jsx
import { useState, useEffect, useRef, useCallback } from "react";

// ── DATA (replace with actual scanned project data) ──────────────────────────
const NODES_DATA = [
  { id: "app",        label: "App",          type: "page",    lines: 142, severity: "high",     rerenderRisk: "high",   cluster: "pages",    issues: ["C2: inline style prop","C9: depth 9"] },
  { id: "layout",     label: "Layout",       type: "layout",  lines: 88,  severity: "clean",    rerenderRisk: "low",    cluster: "pages",    issues: [] },
  { id: "dashboard",  label: "Dashboard",    type: "page",    lines: 312, severity: "critical", rerenderRisk: "high",   cluster: "pages",    issues: ["C1: god component 312 lines","C3: prop drilling x4"] },
  { id: "sidebar",    label: "Sidebar",      type: "feature", lines: 95,  severity: "medium",   rerenderRisk: "medium", cluster: "features", issues: ["C2: inline fn prop"] },
  { id: "usercard",   label: "UserCard",     type: "ui",      lines: 67,  severity: "low",      rerenderRisk: "low",    cluster: "ui",       issues: ["C13: unused prop"] },
  { id: "table",      label: "DataTable",    type: "ui",      lines: 198, severity: "high",     rerenderRisk: "high",   cluster: "ui",       issues: ["C4: no virtualization","C2: key=index"] },
  { id: "useauth",    label: "useAuth",      type: "hook",    lines: 54,  severity: "medium",   rerenderRisk: "medium", cluster: "hooks",    issues: ["C6: no AbortController"] },
  { id: "authctx",    label: "AuthContext",  type: "context", lines: 41,  severity: "high",     rerenderRisk: "high",   cluster: "contexts", issues: ["C2: context value not memoized"] },
  { id: "modal",      label: "Modal",        type: "ui",      lines: 73,  severity: "medium",   rerenderRisk: "low",    cluster: "ui",       issues: ["C12: no focus trap"] },
  { id: "usedata",    label: "useData",      type: "hook",    lines: 88,  severity: "low",      rerenderRisk: "low",    cluster: "hooks",    issues: [] },
];

const EDGES_DATA = [
  { source: "app",       target: "layout",    type: "renders" },
  { source: "app",       target: "authctx",   type: "uses-context" },
  { source: "layout",    target: "dashboard", type: "renders" },
  { source: "layout",    target: "sidebar",   type: "renders" },
  { source: "dashboard", target: "usercard",  type: "renders" },
  { source: "dashboard", target: "table",     type: "renders" },
  { source: "dashboard", target: "modal",     type: "renders" },
  { source: "dashboard", target: "usedata",   type: "uses-hook" },
  { source: "sidebar",   target: "useauth",   type: "uses-hook" },
  { source: "useauth",   target: "authctx",   type: "uses-context" },
  { source: "usercard",  target: "useauth",   type: "uses-hook" },
];

// ── CONSTANTS ─────────────────────────────────────────────────────────────────
const SEV_COLOR  = { critical:"#ff3b3b", high:"#ff8c00", medium:"#ffd60a", low:"#4cc9f0", clean:"#2ecc71" };
const SEV_LABEL  = { critical:"🔴 Critical", high:"🟠 High", medium:"🟡 Medium", low:"🔵 Low", clean:"✅ Clean" };
const TYPE_ICON  = { page:"⬡", layout:"⬢", feature:"◈", ui:"◇", hook:"↺", context:"⊛", util:"⊡" };
const CLUSTER_POS = { pages:[340,110], features:[180,280], ui:[480,300], hooks:[200,460], contexts:[520,460] };
const CLUSTER_COLOR = { pages:"rgba(99,102,241,0.06)", features:"rgba(16,185,129,0.06)", ui:"rgba(245,158,11,0.06)", hooks:"rgba(236,72,153,0.06)", contexts:"rgba(139,92,246,0.06)" };

// ── LAYOUT — radial spring positions ─────────────────────────────────────────
function buildPositions(nodes, edges) {
  const W = 680, H = 520;
  const pos = {};
  const clusters = {};
  nodes.forEach(n => { (clusters[n.cluster] = clusters[n.cluster] || []).push(n.id); });

  // seed from cluster centers
  nodes.forEach(n => {
    const [cx, cy] = CLUSTER_POS[n.cluster] || [W/2, H/2];
    const idx = clusters[n.cluster].indexOf(n.id);
    const total = clusters[n.cluster].length;
    const angle = (idx / total) * Math.PI * 2;
    const r = total === 1 ? 0 : 55;
    pos[n.id] = { x: cx + Math.cos(angle) * r, y: cy + Math.sin(angle) * r, vx: 0, vy: 0 };
  });

  // force-directed iterations
  for (let iter = 0; iter < 220; iter++) {
    nodes.forEach(a => {
      nodes.forEach(b => {
        if (a.id === b.id) return;
        const dx = pos[a.id].x - pos[b.id].x;
        const dy = pos[a.id].y - pos[b.id].y;
        const d = Math.sqrt(dx*dx + dy*dy) || 1;
        const rep = Math.min(3200 / (d*d), 6);
        pos[a.id].vx += (dx/d) * rep;
        pos[a.id].vy += (dy/d) * rep;
      });
    });
    edges.forEach(e => {
      const a = pos[e.source], b = pos[e.target];
      if (!a || !b) return;
      const dx = b.x - a.x, dy = b.y - a.y;
      const d = Math.sqrt(dx*dx + dy*dy) || 1;
      const ideal = 120, f = (d - ideal) * 0.04;
      const fx = (dx/d)*f, fy = (dy/d)*f;
      a.vx += fx; a.vy += fy;
      b.vx -= fx; b.vy -= fy;
    });
    // cluster attraction
    nodes.forEach(n => {
      const [cx, cy] = CLUSTER_POS[n.cluster] || [W/2, H/2];
      pos[n.id].vx += (cx - pos[n.id].x) * 0.012;
      pos[n.id].vy += (cy - pos[n.id].y) * 0.012;
    });
    nodes.forEach(n => {
      pos[n.id].vx *= 0.7; pos[n.id].vy *= 0.7;
      pos[n.id].x = Math.max(48, Math.min(W-48, pos[n.id].x + pos[n.id].vx));
      pos[n.id].y = Math.max(40, Math.min(H-40, pos[n.id].y + pos[n.id].vy));
    });
  }
  return pos;
}

// ── HEALTH SCORE ─────────────────────────────────────────────────────────────
function calcHealth(nodes) {
  const weights = { critical: 20, high: 10, medium: 4, low: 1 };
  const total = nodes.reduce((s, n) => s + n.issues.length * (weights[n.severity] || 0), 0);
  return Math.max(0, Math.min(100, 100 - Math.round(total / nodes.length)));
}

// ── MAIN COMPONENT ────────────────────────────────────────────────────────────
export default function ComponentGraph() {
  const [positions]     = useState(() => buildPositions(NODES_DATA, EDGES_DATA));
  const [selected, setSelected]   = useState(null);
  const [hovered, setHovered]     = useState(null);
  const [filter, setFilter]       = useState("all");
  const [mounted, setMounted]     = useState(false);
  const [gaugeVal, setGaugeVal]   = useState(0);
  const [flowOffset, setFlowOffset] = useState(0);
  const animRef = useRef(null);
  const health  = calcHealth(NODES_DATA);

  // mount animation
  useEffect(() => {
    requestAnimationFrame(() => setMounted(true));
    let start = null;
    const animate = (t) => {
      if (!start) start = t;
      const p = Math.min((t - start) / 1200, 1);
      setGaugeVal(Math.round(health * p));
      setFlowOffset(prev => (prev - 0.6) % 24);
      animRef.current = requestAnimationFrame(animate);
    };
    animRef.current = requestAnimationFrame(animate);
    return () => cancelAnimationFrame(animRef.current);
  }, [health]);

  // continuous edge flow
  useEffect(() => {
    const id = setInterval(() => setFlowOffset(p => (p - 0.6) % 24), 30);
    return () => clearInterval(id);
  }, []);

  const filteredNodes = NODES_DATA.filter(n =>
    filter === "all" || n.severity === filter
  );
  const visibleIds = new Set(filteredNodes.map(n => n.id));

  const connectedTo = selected
    ? new Set(EDGES_DATA.flatMap(e =>
        e.source === selected ? [e.target] :
        e.target === selected ? [e.source] : []
      ))
    : null;

  const nodeOpacity = (id) => {
    if (!selected) return 1;
    if (id === selected) return 1;
    return connectedTo?.has(id) ? 0.85 : 0.18;
  };

  const tip = hovered ? NODES_DATA.find(n => n.id === hovered) : null;
  const tipPos = hovered ? positions[hovered] : null;

  const W = 680, H = 520;

  return (
    <div style={{ background:"#0a0a0f", borderRadius:12, overflow:"hidden", fontFamily:"monospace", userSelect:"none" }}>

      {/* HEADER */}
      <div style={{ padding:"14px 18px 10px", borderBottom:"1px solid #1e1e2e", display:"flex", alignItems:"center", justifyContent:"space-between" }}>
        <div>
          <div style={{ color:"#e2e8f0", fontSize:13, fontWeight:600, letterSpacing:"0.05em" }}>COMPONENT DEPENDENCY GRAPH</div>
          <div style={{ color:"#64748b", fontSize:11, marginTop:2 }}>{NODES_DATA.length} nodes · {EDGES_DATA.length} edges</div>
        </div>
        {/* Health gauge */}
        <div style={{ display:"flex", alignItems:"center", gap:10 }}>
          <svg width="52" height="52" viewBox="0 0 52 52">
            <circle cx="26" cy="26" r="20" fill="none" stroke="#1e1e2e" strokeWidth="5"/>
            <circle cx="26" cy="26" r="20" fill="none"
              stroke={gaugeVal >= 80 ? "#2ecc71" : gaugeVal >= 50 ? "#ffd60a" : "#ff3b3b"}
              strokeWidth="5" strokeLinecap="round"
              strokeDasharray={`${(gaugeVal/100)*125.7} 125.7`}
              transform="rotate(-90 26 26)"
              style={{ transition:"stroke-dasharray 0.05s" }}
            />
            <text x="26" y="30" textAnchor="middle" fill="#e2e8f0" fontSize="12" fontWeight="600">{gaugeVal}</text>
          </svg>
          <div style={{ color:"#64748b", fontSize:10, lineHeight:1.4 }}>Health<br/>Score</div>
        </div>
      </div>

      {/* FILTER BAR */}
      <div style={{ display:"flex", gap:6, padding:"8px 18px", borderBottom:"1px solid #1e1e2e", flexWrap:"wrap" }}>
        {["all","critical","high","medium","low","clean"].map(f => (
          <button key={f} onClick={() => setFilter(f)}
            style={{ padding:"3px 10px", borderRadius:20, border:"1px solid",
              borderColor: filter===f ? SEV_COLOR[f] || "#6366f1" : "#2a2a3a",
              background: filter===f ? (SEV_COLOR[f] || "#6366f1")+"22" : "transparent",
              color: filter===f ? SEV_COLOR[f] || "#6366f1" : "#64748b",
              fontSize:11, cursor:"pointer", transition:"all 0.15s" }}>
            {f === "all" ? "All" : SEV_LABEL[f]}
          </button>
        ))}
        {selected && (
          <button onClick={() => setSelected(null)}
            style={{ marginLeft:"auto", padding:"3px 10px", borderRadius:20, border:"1px solid #2a2a3a",
              background:"transparent", color:"#94a3b8", fontSize:11, cursor:"pointer" }}>
            ✕ Clear
          </button>
        )}
      </div>

      {/* GRAPH CANVAS */}
      <div style={{ position:"relative" }}>
        <svg width="100%" viewBox={`0 0 ${W} ${H}`} style={{ display:"block" }}>
          <defs>
            {Object.entries(SEV_COLOR).map(([k,c]) => (
              <marker key={k} id={`arr-${k}`} viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
                <path d="M2 2L8 5L2 8" fill="none" stroke={c} strokeWidth="1.5" strokeLinecap="round"/>
              </marker>
            ))}
            <marker id="arr-default" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse">
              <path d="M2 2L8 5L2 8" fill="none" stroke="#334155" strokeWidth="1.5" strokeLinecap="round"/>
            </marker>
          </defs>

          {/* Cluster zones */}
          {Object.entries(CLUSTER_POS).map(([cluster, [cx,cy]]) => (
            <ellipse key={cluster} cx={cx} cy={cy} rx={90} ry={72}
              fill={CLUSTER_COLOR[cluster]} stroke="none"
              style={{ opacity: mounted ? 1 : 0, transition:"opacity 0.8s" }}/>
          ))}
          {Object.entries(CLUSTER_POS).map(([cluster, [cx,cy]]) => (
            <text key={cluster+"-lbl"} x={cx} y={cy-78} textAnchor="middle"
              fill="#334155" fontSize="9" letterSpacing="0.1em"
              style={{ opacity: mounted ? 1 : 0, transition:"opacity 1s 0.3s" }}>
              {cluster.toUpperCase()}
            </text>
          ))}

          {/* Edges */}
          {EDGES_DATA.map((e, i) => {
            const a = positions[e.source], b = positions[e.target];
            if (!a || !b || !visibleIds.has(e.source) || !visibleIds.has(e.target)) return null;
            const isRerender = e.type === "uses-context" || e.type === "renders";
            const srcNode = NODES_DATA.find(n => n.id === e.source);
            const isHot = isRerender && (srcNode?.rerenderRisk === "high");
            const isFocused = selected === e.source || selected === e.target;
            const mx = (a.x + b.x)/2, my = (a.y + b.y)/2 - 30;
            const opacity = selected ? (isFocused ? 0.9 : 0.04) : 0.35;
            const color = isHot ? "#ff3b3b" : e.type === "uses-context" ? "#a855f7" : e.type === "uses-hook" ? "#22d3ee" : "#334155";
            return (
              <g key={i} style={{ opacity, transition:"opacity 0.2s" }}>
                <path d={`M${a.x},${a.y} Q${mx},${my} ${b.x},${b.y}`}
                  fill="none" stroke={color} strokeWidth={isFocused ? 1.5 : 0.8}
                  strokeDasharray={isHot ? "5 4" : undefined}
                  strokeDashoffset={isHot ? flowOffset : undefined}
                  markerEnd={`url(#arr-default)`}
                />
              </g>
            );
          })}

          {/* Nodes */}
          {filteredNodes.map((node, i) => {
            const p = positions[node.id];
            if (!p) return null;
            const col = SEV_COLOR[node.severity];
            const size = Math.max(22, Math.min(38, 20 + node.lines/14));
            const isSelected = selected === node.id;
            const isHov = hovered === node.id;
            const op = nodeOpacity(node.id);
            return (
              <g key={node.id}
                style={{
                  opacity: mounted ? op : 0,
                  transition: `opacity 0.2s, transform 0.3s`,
                  transform: mounted ? "none" : "scale(0)",
                  transformOrigin: `${p.x}px ${p.y}px`,
                  cursor: "pointer",
                  transitionDelay: mounted ? "0s" : `${i*0.04}s`
                }}
                onClick={() => setSelected(selected === node.id ? null : node.id)}
                onMouseEnter={() => setHovered(node.id)}
                onMouseLeave={() => setHovered(null)}
              >
                {/* Glow ring */}
                {(isSelected || isHov) && (
                  <circle cx={p.x} cy={p.y} r={size + 7} fill={col + "22"} stroke={col} strokeWidth="0.5"/>
                )}
                {/* Re-render pulse ring */}
                {node.rerenderRisk === "high" && !isSelected && (
                  <circle cx={p.x} cy={p.y} r={size + 3} fill="none" stroke="#ff3b3b" strokeWidth="0.5" opacity="0.4"
                    style={{ animation:"pulse 2s ease-in-out infinite" }}/>
                )}
                {/* Node body */}
                <circle cx={p.x} cy={p.y} r={size} fill={col+"18"} stroke={col} strokeWidth={isSelected ? 2 : 0.8}/>
                {/* Type icon */}
                <text x={p.x} y={p.y - 4} textAnchor="middle" fontSize="13" fill={col}>{TYPE_ICON[node.type]}</text>
                {/* Label */}
                <text x={p.x} y={p.y + size + 11} textAnchor="middle" fontSize="9" fill="#94a3b8" letterSpacing="0.03em">
                  {node.label}
                </text>
                {/* Issue count badge */}
                {node.issues.length > 0 && (
                  <g>
                    <circle cx={p.x + size - 3} cy={p.y - size + 3} r={8} fill="#0a0a0f" stroke={col} strokeWidth="0.8"/>
                    <text x={p.x + size - 3} y={p.y - size + 7} textAnchor="middle" fontSize="8" fontWeight="700" fill={col}>
                      {node.issues.length}
                    </text>
                  </g>
                )}
              </g>
            );
          })}
        </svg>

        {/* HOVER TOOLTIP */}
        {tip && tipPos && (
          <div style={{
            position:"absolute",
            left: tipPos.x > 460 ? tipPos.x - 200 : tipPos.x + 30,
            top: Math.min(tipPos.y - 10, 380),
            background:"#0f0f1a", border:`1px solid ${SEV_COLOR[tip.severity]}44`,
            borderRadius:8, padding:"10px 12px", minWidth:180, maxWidth:220,
            pointerEvents:"none", zIndex:10,
            boxShadow:`0 0 20px ${SEV_COLOR[tip.severity]}22`
          }}>
            <div style={{ color:"#e2e8f0", fontSize:12, fontWeight:600, marginBottom:4 }}>
              {TYPE_ICON[tip.type]} {tip.label}
            </div>
            <div style={{ color:"#64748b", fontSize:10, marginBottom:6 }}>{tip.filePath || tip.type} · {tip.lines} lines</div>
            <div style={{ display:"flex", gap:8, marginBottom:tip.issues.length ? 8 : 0, flexWrap:"wrap" }}>
              <span style={{ fontSize:10, color: SEV_COLOR[tip.severity] }}>{SEV_LABEL[tip.severity]}</span>
              <span style={{ fontSize:10, color: tip.rerenderRisk==="high" ? "#ff3b3b" : tip.rerenderRisk==="medium" ? "#ffd60a" : "#2ecc71" }}>
                ↺ {tip.rerenderRisk} re-render risk
              </span>
            </div>
            {tip.issues.map((iss, i) => (
              <div key={i} style={{ fontSize:10, color:"#94a3b8", borderLeft:"2px solid #1e1e2e", paddingLeft:6, marginTop:3 }}>
                {iss}
              </div>
            ))}
          </div>
        )}
      </div>

      {/* LEGEND */}
      <div style={{ padding:"8px 18px 12px", borderTop:"1px solid #1e1e2e", display:"flex", gap:16, flexWrap:"wrap", alignItems:"center" }}>
        {Object.entries(SEV_COLOR).map(([k,c]) => (
          <div key={k} style={{ display:"flex", alignItems:"center", gap:5 }}>
            <div style={{ width:8, height:8, borderRadius:"50%", background:c }}/>
            <span style={{ color:"#64748b", fontSize:10 }}>{k}</span>
          </div>
        ))}
        <div style={{ marginLeft:"auto", display:"flex", gap:12 }}>
          {[["renders","#334155"],["uses-hook","#22d3ee"],["uses-context","#a855f7"],["hot re-render","#ff3b3b"]].map(([l,c]) => (
            <div key={l} style={{ display:"flex", alignItems:"center", gap:4 }}>
              <div style={{ width:16, height:1.5, background:c }}/>
              <span style={{ color:"#64748b", fontSize:10 }}>{l}</span>
            </div>
          ))}
        </div>
      </div>

      <style>{`
        @keyframes pulse {
          0%,100% { opacity:0.3; r:25; }
          50% { opacity:0.7; r:28; }
        }
      `}</style>
    </div>
  );
}
```

## Visual Spec Summary

| Property | Value |
|---|---|
| Background | `#0a0a0f` |
| Node color | Severity: critical=`#ff3b3b` high=`#ff8c00` medium=`#ffd60a` low=`#4cc9f0` clean=`#2ecc71` |
| Node size | `20 + lineCount/14`, clamped 22–38px radius |
| Edge: renders | `#334155` thin |
| Edge: uses-hook | `#22d3ee` |
| Edge: uses-context | `#a855f7` |
| Edge: hot re-render | `#ff3b3b` animated dashes |
| Cluster zones | Subtle filled ellipses, 6% opacity |
| Type icons | page=⬡ layout=⬢ feature=◈ ui=◇ hook=↺ context=⊛ |
| Health gauge | SVG circle, color by score: <50=red, <80=yellow, ≥80=green |

## Customization When Generating

When generating for a real project:
1. Replace `NODES_DATA` with actual components found
2. Replace `EDGES_DATA` with actual import relationships
3. Add real `filePath` values to nodes
4. Populate real `issues` arrays with actual check findings
5. Adjust `CLUSTER_POS` if the project has different folder structure
6. Keep all animation and interaction code exactly as shown
