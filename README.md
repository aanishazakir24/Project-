import { useState, useCallback, useReducer, useRef } from "react";

// ═══════════════════════════════════════════════════════════════════
//  DATABASE  –  rows + hooks + phones
// ═══════════════════════════════════════════════════════════════════
const initialPhones = [
  { id:101, brand:"Apple",   model:"iPhone 15 Pro Max", price:1199, ram:"8GB",  storage:"256GB", img:"📱", stock:12, category:"Flagship"  },
  { id:102, brand:"Apple",   model:"iPhone 15",         price:799,  ram:"6GB",  storage:"128GB", img:"📱", stock:25, category:"Standard"  },
  { id:103, brand:"Samsung", model:"Galaxy S24 Ultra",  price:1299, ram:"12GB", storage:"512GB", img:"📲", stock:8,  category:"Flagship"  },
  { id:104, brand:"Samsung", model:"Galaxy S24",        price:799,  ram:"8GB",  storage:"256GB", img:"📲", stock:18, category:"Standard"  },
  { id:105, brand:"Google",  model:"Pixel 8 Pro",       price:999,  ram:"12GB", storage:"256GB", img:"📳", stock:10, category:"Flagship"  },
  { id:106, brand:"OnePlus", model:"12R",               price:499,  ram:"16GB", storage:"256GB", img:"📱", stock:22, category:"Mid-range" },
  { id:107, brand:"Xiaomi",  model:"14 Ultra",          price:899,  ram:"16GB", storage:"512GB", img:"📲", stock:6,  category:"Flagship"  },
  { id:108, brand:"Sony",    model:"Xperia 1 VI",       price:1299, ram:"12GB", storage:"256GB", img:"📱", stock:4,  category:"Flagship"  },
  { id:109, brand:"Nokia",   model:"G60 5G",            price:299,  ram:"4GB",  storage:"128GB", img:"📳", stock:30, category:"Budget"    },
  { id:110, brand:"Motorola",model:"Edge 40 Pro",       price:599,  ram:"12GB", storage:"256GB", img:"📱", stock:14, category:"Mid-range" },
];

// rows: each row has an id, label, and an ordered array of hooks
// hook: { hookId, hookNo (display), phoneId | null }
const initialRows = [
  {
    id: "row-A", label: "Row A – Flagship",
    hooks: [
      { hookId:"h-A1", hookNo:"A1", phoneId:101 },
      { hookId:"h-A2", hookNo:"A2", phoneId:103 },
      { hookId:"h-A3", hookNo:"A3", phoneId:105 },
      { hookId:"h-A4", hookNo:"A4", phoneId:107 },
      { hookId:"h-A5", hookNo:"A5", phoneId:108 },
      { hookId:"h-A6", hookNo:"A6", phoneId:null },
    ]
  },
  {
    id: "row-B", label: "Row B – Standard",
    hooks: [
      { hookId:"h-B1", hookNo:"B1", phoneId:102 },
      { hookId:"h-B2", hookNo:"B2", phoneId:104 },
      { hookId:"h-B3", hookNo:"B3", phoneId:null },
      { hookId:"h-B4", hookNo:"B4", phoneId:null },
    ]
  },
  {
    id: "row-C", label: "Row C – Budget & Mid",
    hooks: [
      { hookId:"h-C1", hookNo:"C1", phoneId:106 },
      { hookId:"h-C2", hookNo:"C2", phoneId:109 },
      { hookId:"h-C3", hookNo:"C3", phoneId:110 },
      { hookId:"h-C4", hookNo:"C4", phoneId:null },
      { hookId:"h-C5", hookNo:"C5", phoneId:null },
    ]
  },
];

// ─── REDUCERS ────────────────────────────────────────────────────────
function phonesReducer(state, action) {
  switch (action.type) {
    case "ADD":    return [...state, { ...action.payload, id: Date.now() }];
    case "UPDATE": return state.map(p => p.id === action.payload.id ? { ...p, ...action.payload } : p);
    case "DELETE": return state.filter(p => p.id !== action.id);
    default: return state;
  }
}

function rowsReducer(state, action) {
  switch (action.type) {
    case "ADD_ROW": {
      const rowLetter = String.fromCharCode(65 + state.length); // A, B, C …
      return [...state, { id:`row-${Date.now()}`, label: action.label || `Row ${rowLetter}`, hooks: [] }];
    }
    case "DELETE_ROW":
      return state.filter(r => r.id !== action.rowId);
    case "RENAME_ROW":
      return state.map(r => r.id === action.rowId ? { ...r, label: action.label } : r);
    case "ADD_HOOK": {
      return state.map(r => {
        if (r.id !== action.rowId) return r;
        // count hooks in this row to build hookNo like A1, A2 …
        const prefix = action.prefix || r.label.match(/Row ([A-Z])/)?.[1] || "X";
        const nextNum = r.hooks.length + 1;
        const hookNo  = `${prefix}${nextNum}`;
        return { ...r, hooks: [...r.hooks, { hookId:`h-${Date.now()}`, hookNo, phoneId: null }] };
      });
    }
    case "DELETE_HOOK":
      return state.map(r => r.id !== action.rowId ? r : { ...r, hooks: r.hooks.filter(h => h.hookId !== action.hookId) });
    case "ASSIGN_PHONE":
      return state.map(r => r.id !== action.rowId ? r : {
        ...r,
        hooks: r.hooks.map(h => h.hookId === action.hookId ? { ...h, phoneId: action.phoneId } : h)
      });
    case "UNASSIGN_PHONE":
      return state.map(r => ({ ...r, hooks: r.hooks.map(h => h.hookId === action.hookId ? { ...h, phoneId: null } : h) }));
    default: return state;
  }
}

// ─── CUSTOM HOOKS ─────────────────────────────────────────────────────
function useShopDB() {
  const [phones, dispatchPhones] = useReducer(phonesReducer, initialPhones);
  const [rows,   dispatchRows]   = useReducer(rowsReducer,   initialRows);

  const addPhone    = useCallback(d  => dispatchPhones({ type:"ADD",    payload:d }),      []);
  const updatePhone = useCallback(d  => dispatchPhones({ type:"UPDATE", payload:d }),      []);
  const deletePhone = useCallback(id => dispatchPhones({ type:"DELETE", id }),             []);

  const addRow      = useCallback(label  => dispatchRows({ type:"ADD_ROW",    label }),                          []);
  const deleteRow   = useCallback(rowId  => dispatchRows({ type:"DELETE_ROW", rowId }),                          []);
  const renameRow   = useCallback((rowId,label)=> dispatchRows({ type:"RENAME_ROW", rowId, label }),             []);
  const addHook     = useCallback((rowId,prefix)=> dispatchRows({ type:"ADD_HOOK", rowId, prefix }),             []);
  const deleteHook  = useCallback((rowId,hookId)=> dispatchRows({ type:"DELETE_HOOK",  rowId, hookId }),         []);
  const assignPhone = useCallback((rowId,hookId,phoneId)=> dispatchRows({ type:"ASSIGN_PHONE", rowId, hookId, phoneId }), []);
  const unassign    = useCallback((hookId)=> dispatchRows({ type:"UNASSIGN_PHONE", hookId }),                    []);

  return { phones, rows, addPhone, updatePhone, deletePhone, addRow, deleteRow, renameRow, addHook, deleteHook, assignPhone, unassign };
}

function useCart() {
  const [cart, setCart] = useState([]);
  const add    = useCallback(phone => setCart(prev => {
    const ex = prev.find(p => p.id === phone.id);
    if (ex) return prev.map(p => p.id === phone.id ? { ...p, qty: p.qty+1 } : p);
    return [...prev, { ...phone, qty:1 }];
  }), []);
  const remove = useCallback(id => setCart(prev => prev.filter(p => p.id !== id)), []);
  const total  = cart.reduce((s,p) => s + p.price*p.qty, 0);
  return { cart, add, remove, total };
}

function useNotif() {
  const [n, setN] = useState(null);
  const notify = useCallback((msg, type="ok") => { setN({ msg, type }); setTimeout(()=>setN(null), 2600); }, []);
  return { n, notify };
}

function useModal() {
  const [m, setM] = useState(null);
  const open  = useCallback((type,data=null) => setM({ type, data }), []);
  const close = useCallback(() => setM(null), []);
  return { m, open, close };
}

function useForm(init) {
  const [form, setForm] = useState(init);
  const set   = (k,v) => setForm(p => ({ ...p, [k]:v }));
  const reset = ()    => setForm(init);
  return { form, set, reset, setForm };
}

// ─── CONSTANTS ────────────────────────────────────────────────────────
const EMPTY_PHONE = { brand:"", model:"", price:"", ram:"", storage:"", img:"📱", stock:"", category:"Standard" };
const EMOJIS = ["📱","📲","📳","☎️","📞"];
const CATS   = ["Flagship","Standard","Mid-range","Budget"];
const catColor = c => ({ Flagship:"#6c5ce7", Standard:"#00b894", "Mid-range":"#fdcb6e", Budget:"#ff7675" }[c]||"#636e72");

// ═══════════════════════════════════════════════════════════════════
//  ROOT APP
// ═══════════════════════════════════════════════════════════════════
export default function App() {
  const db = useShopDB();
  const { cart, add:addCart, remove:removeCart, total } = useCart();
  const { n, notify } = useNotif();
  const { m, open, close } = useModal();
  const { form, set, reset, setForm } = useForm(EMPTY_PHONE);
  const [view, setView]       = useState("shop");
  const [newRowLabel, setNewRowLabel] = useState("");
  const [renamingRow, setRenamingRow] = useState(null); // { rowId, label }

  // helpers
  const phoneById = id => db.phones.find(p => p.id === id);

  const handleAddPhone = () => {
    if (!form.brand||!form.model||!form.price) return notify("Fill required fields","err");
    db.addPhone({ ...form, price:+form.price, stock:+form.stock||0 });
    notify(`✅ ${form.model} added to catalogue`); reset(); close();
  };
  const handleUpdatePhone = () => {
    db.updatePhone({ ...form, price:+form.price, stock:+form.stock });
    notify(`✏️ ${form.model} updated`); close();
  };
  const handleDeletePhone = p => { db.deletePhone(p.id); notify(`🗑️ ${p.model} removed`,"err"); close(); };

  const openEditPhone = p => { setForm({ ...p, price:String(p.price), stock:String(p.stock) }); open("editPhone",p); };

  // assign modal state
  const [assignTarget, setAssignTarget] = useState(null); // { rowId, hookId, hookNo }
  const openAssign = (rowId,hookId,hookNo) => { setAssignTarget({ rowId,hookId,hookNo }); open("assign"); };

  return (
    <div style={S.root}>
      {/* NOTIFICATION */}
      {n && <div style={{ ...S.notif, background: n.type==="err"?"#ff4757":"#2ed573" }}>{n.msg}</div>}

      {/* HEADER */}
      <header style={S.header}>
        <div>
          <div style={S.logo}>📡 MobiStore</div>
          <div style={S.tagline}>Multi-Row Hook Display System</div>
        </div>
        <nav style={S.nav}>
          {[["shop","🏪 Shop"],["admin","⚙️ Admin"],["cart",`🛒 ${cart.length}`]].map(([v,l])=>(
            <button key={v} onClick={()=>setView(v)} style={{ ...S.navBtn, ...(view===v?S.navActive:{}) }}>{l}</button>
          ))}
        </nav>
      </header>

      {/* ── SHOP VIEW ── */}
      {view==="shop" && (
        <div style={S.page}>
          <p style={S.hint}>🪝 Each row has multiple hooks. Click a hook to pick the phone.</p>

          {db.rows.map((row,ri) => (
            <ShopRow key={row.id} row={row} rowIndex={ri} phoneById={phoneById}
              onPick={(phone,hookNo) => { addCart(phone); notify(`🛒 Hook ${hookNo} – ${phone.model} added`); }}
            />
          ))}

          {db.rows.length === 0 && <div style={S.empty}>No rows yet. Go to Admin → add a row.</div>}
        </div>
      )}

      {/* ── ADMIN VIEW ── */}
      {view==="admin" && (
        <div style={S.page}>
          <div style={S.adminCols}>

            {/* LEFT: ROWS & HOOKS */}
            <div style={S.adminLeft}>
              <div style={S.sectionHead}>
                <span>🪝 Rows & Hooks</span>
                <div style={S.rowAddBar}>
                  <input value={newRowLabel} onChange={e=>setNewRowLabel(e.target.value)}
                    placeholder="Row label…" style={{ ...S.smInput, width:150 }} />
                  <button onClick={()=>{ db.addRow(newRowLabel||undefined); setNewRowLabel(""); notify("✅ Row added"); }}
                    style={S.smBtn}>＋ Row</button>
                </div>
              </div>

              {db.rows.map((row,ri) => {
                const prefix = row.label.match(/Row ([A-Za-z]+)/)?.[1] || String.fromCharCode(65+ri);
                return (
                  <div key={row.id} style={S.adminRow}>
                    {/* Row header */}
                    <div style={S.adminRowHead}>
                      {renamingRow?.rowId === row.id
                        ? <input autoFocus value={renamingRow.label}
                            onChange={e=>setRenamingRow(r=>({...r,label:e.target.value}))}
                            onBlur={()=>{ db.renameRow(row.id,renamingRow.label); setRenamingRow(null); }}
                            onKeyDown={e=>{ if(e.key==="Enter"){ db.renameRow(row.id,renamingRow.label); setRenamingRow(null); }}}
                            style={{ ...S.smInput, flex:1, fontWeight:700 }} />
                        : <span style={S.adminRowLabel} onClick={()=>setRenamingRow({rowId:row.id,label:row.label})}>
                            ✏️ {row.label}
                          </span>
                      }
                      <div style={{ display:"flex", gap:6 }}>
                        <button onClick={()=>{ db.addHook(row.id, prefix); notify(`🪝 Hook added to ${row.label}`); }}
                          style={S.smBtn}>＋ Hook</button>
                        <button onClick={()=>open("confirmRow",row)} style={S.smDelBtn}>🗑️ Row</button>
                      </div>
                    </div>

                    {/* Hooks grid */}
                    <div style={S.hooksGrid}>
                      {row.hooks.length===0 && <span style={{ color:"#4a4a6a", fontSize:12 }}>No hooks – click ＋ Hook</span>}
                      {row.hooks.map(hook => {
                        const phone = hook.phoneId ? phoneById(hook.phoneId) : null;
                        return (
                          <div key={hook.hookId} style={S.adminHookCard}>
                            <div style={S.adminHookTop}>
                              <span style={S.adminHookNo}>🪝 {hook.hookNo}</span>
                              <button onClick={()=>{ db.deleteHook(row.id,hook.hookId); notify(`Hook ${hook.hookNo} removed`,"err"); }}
                                style={S.tinyDelBtn}>✕</button>
                            </div>
                            {phone
                              ? <>
                                  <div style={{ fontSize:26, textAlign:"center" }}>{phone.img}</div>
                                  <div style={S.hookCardModel}>{phone.model}</div>
                                  <div style={{ color:"#00d2d3", fontWeight:800, fontSize:13 }}>${phone.price}</div>
                                  <div style={{ display:"flex", gap:4, marginTop:4 }}>
                                    <button onClick={()=>openAssign(row.id,hook.hookId,hook.hookNo)} style={S.tinyBtn}>🔄</button>
                                    <button onClick={()=>{ db.unassign(hook.hookId); notify(`Hook ${hook.hookNo} cleared`); }} style={S.tinyDelBtn}>⛔</button>
                                  </div>
                                </>
                              : <>
                                  <div style={{ fontSize:26, textAlign:"center", opacity:0.3 }}>📵</div>
                                  <div style={{ color:"#4a4a6a", fontSize:11, textAlign:"center" }}>Empty</div>
                                  <button onClick={()=>openAssign(row.id,hook.hookId,hook.hookNo)} style={{ ...S.tinyBtn, marginTop:4, width:"100%" }}>
                                    Assign
                                  </button>
                                </>
                            }
                          </div>
                        );
                      })}
                    </div>
                  </div>
                );
              })}
            </div>

            {/* RIGHT: PHONE CATALOGUE */}
            <div style={S.adminRight}>
              <div style={S.sectionHead}>
                <span>📋 Phone Catalogue</span>
                <button onClick={()=>{ reset(); open("addPhone"); }} style={S.smBtn}>＋ Phone</button>
              </div>
              <div style={S.catalogueList}>
                {db.phones.map(p => (
                  <div key={p.id} style={S.catalogueItem}>
                    <span style={{ fontSize:22 }}>{p.img}</span>
                    <div style={{ flex:1, minWidth:0 }}>
                      <div style={{ fontWeight:700, fontSize:13, color:"#f1f2f6", whiteSpace:"nowrap", overflow:"hidden", textOverflow:"ellipsis" }}>{p.model}</div>
                      <div style={{ fontSize:11, color:"#a4b0be" }}>{p.brand} · <span style={{ color:"#00d2d3" }}>${p.price}</span></div>
                    </div>
                    <span style={{ ...S.catChip, background:catColor(p.category), fontSize:9 }}>{p.category}</span>
                    <div style={{ display:"flex", gap:4 }}>
                      <button onClick={()=>openEditPhone(p)} style={S.tinyEditBtn}>✏️</button>
                      <button onClick={()=>open("confirmPhone",p)} style={S.tinyDelBtn}>🗑️</button>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>
        </div>
      )}

      {/* ── CART VIEW ── */}
      {view==="cart" && (
        <div style={S.page}>
          <h2 style={S.pageTitle}>🛒 Cart</h2>
          {!cart.length
            ? <div style={S.empty}>Cart is empty. Pick phones from the hooks!</div>
            : <>
                {cart.map(p=>(
                  <div key={p.id} style={S.cartItem}>
                    <span style={{ fontSize:28 }}>{p.img}</span>
                    <div style={{ flex:1 }}>
                      <div style={{ fontWeight:700, color:"#f1f2f6" }}>{p.model}</div>
                      <div style={{ color:"#a4b0be", fontSize:12 }}>{p.brand} · Qty: {p.qty}</div>
                    </div>
                    <span style={{ color:"#00d2d3", fontWeight:700 }}>${p.price*p.qty}</span>
                    <button onClick={()=>removeCart(p.id)} style={S.tinyDelBtn}>✕</button>
                  </div>
                ))}
                <div style={S.cartTotal}>
                  <span>Total</span>
                  <span style={{ color:"#00d2d3", fontSize:24, fontWeight:800 }}>${total}</span>
                </div>
                <button onClick={()=>notify("🎉 Order placed! Thank you!")} style={S.bigBtn}>Checkout →</button>
              </>
          }
        </div>
      )}

      {/* ═══ MODALS ═══ */}
      {m && (
        <div style={S.overlay} onClick={close}>
          <div style={S.modalBox} onClick={e=>e.stopPropagation()}>

            {/* ADD / EDIT PHONE */}
            {(m.type==="addPhone"||m.type==="editPhone") && (
              <div>
                <h2 style={S.modalTitle}>{m.type==="addPhone"?"➕ Add Phone to Catalogue":"✏️ Edit Phone"}</h2>
                <div style={S.formGrid}>
                  {[["Brand *","brand"],["Model *","model"],["RAM","ram"],["Storage","storage"]].map(([l,k])=>(
                    <div key={k} style={S.fg}>
                      <label style={S.lbl}>{l}</label>
                      <input value={form[k]} onChange={e=>set(k,e.target.value)} style={S.inp} placeholder={l} />
                    </div>
                  ))}
                  <div style={S.fg}><label style={S.lbl}>Price * ($)</label>
                    <input type="number" value={form.price} onChange={e=>set("price",e.target.value)} style={S.inp} /></div>
                  <div style={S.fg}><label style={S.lbl}>Stock</label>
                    <input type="number" value={form.stock} onChange={e=>set("stock",e.target.value)} style={S.inp} /></div>
                  <div style={S.fg}><label style={S.lbl}>Category</label>
                    <select value={form.category} onChange={e=>set("category",e.target.value)} style={S.inp}>
                      {CATS.map(c=><option key={c}>{c}</option>)}
                    </select></div>
                  <div style={S.fg}><label style={S.lbl}>Icon</label>
                    <div style={{ display:"flex", gap:7 }}>
                      {EMOJIS.map(e=>(
                        <button key={e} onClick={()=>set("img",e)}
                          style={{ ...S.emojiBtn, ...(form.img===e?S.emojiBtnOn:{}) }}>{e}</button>
                      ))}
                    </div></div>
                </div>
                <div style={S.modalFooter}>
                  <button onClick={close} style={S.cancelBtn}>Cancel</button>
                  <button onClick={m.type==="addPhone"?handleAddPhone:handleUpdatePhone} style={S.bigBtn}>
                    {m.type==="addPhone"?"Add to Catalogue":"Save"}
   button>
                </div>
              </div>
            )}

            {/* ASSIGN PHONE TO HOOK */}
            {m.type==="assign" && assignTarget && (
              <div>
                <h2 style={S.modalTitle}>🪝 Assign Phone to Hook {assignTarget.hookNo}</h2>
                <p style={{ color:"#636e72", textAlign:"center", fontSize:13, marginBottom:14 }}>Select a model from the catalogue</p>
                <div style={{ display:"flex", flexDirection:"column", gap:8, maxHeight:360, overflowY:"auto" }}>
                  {db.phones.map(p=>(
                    <div key={p.id} style={S.assignRow}
                      onClick={()=>{ db.assignPhone(assignTarget.rowId,assignTarget.hookId,p.id); notify(`✅ Hook ${assignTarget.hookNo} → ${p.model}`); close(); }}>
                      <span style={{ fontSize:26 }}>{p.img}</span>
                      <div style={{ flex:1 }}>
                        <div style={{ fontWeight:700, fontSize:13, color:"#f1f2f6" }}>{p.model}</div>
                        <div style={{ fontSize:11, color:"#a4b0be" }}>{p.brand} · {p.ram} · {p.storage}</div>
                      </div>
                      <span style={{ color:"#00d2d3", fontWeight:800 }}>${p.price}</span>
                      <span style={{ color:"#636e72", fontSize:18 }}>›</span>
                    </div>
                  ))}
                </div>
                <button onClick={close} style={{ ...S.cancelBtn, marginTop:14, width:"100%" }}>Cancel</button>
              </div>
            )}

            {/* CONFIRM DELETE ROW */}
            {m.type==="confirmRow" && m.data && (
              <div style={{ textAlign:"center" }}>
                <div style={{ fontSize:48 }}>⚠️</div>
                <h2 style={S.modalTitle}>Delete {m.data.label}?</h2>
                <p style={{ color:"#a4b0be" }}>All {m.data.hooks.length} hooks in this row will be removed.</p>
                <div style={S.modalFooter}>
                  <button onClick={close} style={S.cancelBtn}>Cancel</button>
                  <button onClick={()=>{ db.deleteRow(m.data.id); notify(`🗑️ ${m.data.label} deleted`,"err"); close(); }}
                    style={{ ...S.bigBtn, background:"#ff4757" }}>Delete Row</button>
                </div>
              </div>
            )}

            {/* CONFIRM DELETE PHONE */}
            {m.type==="confirmPhone" && m.data && (
              <div style={{ textAlign:"center" }}>
                <div style={{ fontSize:48 }}>⚠️</div>
                <h2 style={S.modalTitle}>Delete {m.data.model}?</h2>
                <p style={{ color:"#a4b0be" }}>It will be removed from the catalogue and all hooks.</p>
                <div style={S.modalFooter}>
                  <button onClick={close} style={S.cancelBtn}>Cancel</button>
                  <button onClick={()=>handleDeletePhone(m.data)} style={{ ...S.bigBtn, background:"#ff4757" }}>Delete</button>
                </div>
              </div>
            )}
          </div>
        </div>
      )}
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════════
//  SHOP ROW – shelf rod + hooks
// ═══════════════════════════════════════════════════════════════════
function ShopRow({ row, rowIndex, phoneById, onPick }) {
  const filled = row.hooks.filter(h=>h.phoneId).length;
  return (
    <div style={S.shopRowWrap}>
      {/* CEILING CABLE */}
      <div style={S.ceilingCables}>
        {row.hooks.map(h=>(
          <div key={h.hookId} style={S.ceilingCable} />
        ))}
        {row.hooks.length===0 && <div style={S.ceilingCable} />}
      </div>

      {/* ROD */}
      <div style={S.rodWrap}>
        <div style={S.rod} />
        <div style={S.rodLabel}>
          <span style={S.rodRowName}>{row.label}</span>
          <span style={S.rodCount}>{filled}/{row.hooks.length} hooks filled</span>
        </div>
      </div>

      {/* HOOKS */}
      <div style={S.shopHooksRow}>
        {row.hooks.length === 0 && (
          <div style={S.emptyHookMsg}>No hooks on this row yet</div>
        )}
        {row.hooks.map(hook => {
          const phone = hook.phoneId ? phoneById(hook.phoneId) : null;
          return <ShopHook key={hook.hookId} hook={hook} phone={phone} onPick={onPick} />;
        })}
      </div>
    </div>
  );
}

// ─── INDIVIDUAL HOOK ON SHOP SHELF ────────────────────────────────
function ShopHook({ hook, phone, onPick }) {
  const [hov, setHov] = useState(false);
  const empty = !phone;
  return (
    <div style={{ ...S.shopHook, ...(hov&&!empty?S.shopHookHov:{}), ...(empty?S.shopHookEmpty:{}) }}
      onMouseEnter={()=>setHov(true)} onMouseLeave={()=>setHov(false)}
      onClick={()=>!empty && onPick(phone, hook.hookNo)}>
      {/* hook hardware */}
      <div style={S.hWire} />
      <div style={S.hCurve} />
      <div style={{ ...S.hNo, ...(hov&&!empty?{ boxShadow:"0 0 10px rgba(0,210,211,0.8)" }:{}) }}>
        {hook.hookNo}
      </div>

      {empty ? (
        <div style={S.emptySlot}>
          <span style={{ fontSize:28, opacity:0.2 }}>📵</span>
          <span style={{ fontSize:10, color:"#3a3a5a" }}>Empty</span>
        </div>
      ) : (
        <div style={S.hookPhoneBody}>
          <div style={{ fontSize:36, filter: hov?"drop-shadow(0 0 10px #00d2d3)":"none", transition:"filter 0.3s" }}>{phone.img}</div>
          <div style={S.hBrand}>{phone.brand}</div>
          <div style={S.hModel}>{phone.model}</div>
          <div style={S.hSpec}>{phone.ram} · {phone.storage}</div>
          <div style={S.hPrice}>${phone.price}</div>
          {phone.stock < 5 && <span style={S.lowBadge}>Low</span>}
          <button style={{ ...S.pickBtn, ...(hov?S.pickBtnHov:{}) }}
            onClick={e=>{ e.stopPropagation(); onPick(phone,hook.hookNo); }}>
            🛒 Pick
          </button>
        </div>
      )}
    </div>
  );
}

// ═══════════════════════════════════════════════════════════════════
//  STYLES
// ═══════════════════════════════════════════════════════════════════
const S = {
  root:      { minHeight:"100vh", background:"#0a0a18", fontFamily:"'Sora','Segoe UI',sans-serif", color:"#f1f2f6" },
  header:    { display:"flex", justifyContent:"space-between", alignItems:"center", padding:"14px 28px", background:"rgba(255,255,255,0.03)", borderBottom:"1px solid rgba(255,255,255,0.07)", position:"sticky", top:0, zIndex:100, backdropFilter:"blur(12px)" },
  logo:      { fontSize:20, fontWeight:800, background:"linear-gradient(135deg,#00d2d3,#6c5ce7)", WebkitBackgroundClip:"text", WebkitTextFillColor:"transparent" },
  tagline:   { fontSize:9, color:"#4a4a6a", letterSpacing:2, textTransform:"uppercase", marginTop:2 },
  nav:       { display:"flex", gap:7 },
  navBtn:    { padding:"7px 15px", borderRadius:8, border:"1px solid rgba(255,255,255,0.1)", background:"transparent", color:"#a4b0be", cursor:"pointer", fontSize:13, fontWeight:600 },
  navActive: { background:"rgba(0,210,211,0.14)", border:"1px solid #00d2d3", color:"#00d2d3" },
  page:      { maxWidth:1400, margin:"0 auto", padding:"24px 20px" },
  hint:      { color:"#4a4a6a", fontSize:13, marginBottom:24, letterSpacing:0.5 },
  pageTitle: { fontSize:20, fontWeight:800, margin:"0 0 18px" },
  empty:     { textAlign:"center", padding:"60px 0", color:"#3a3a5a", fontSize:16 },
  notif:     { position:"fixed", top:68, right:18, zIndex:400, padding:"11px 20px", borderRadius:12, color:"#fff", fontWeight:700, fontSize:13, boxShadow:"0 8px 30px rgba(0,0,0,0.5)" },

  // shop rows
  shopRowWrap:  { marginBottom:52 },
  ceilingCables:{ display:"flex", gap:0, paddingLeft:24 },
  ceilingCable: { width:2, height:18, background:"rgba(100,100,150,0.4)", marginRight:138 },
  rodWrap:      { position:"relative", marginBottom:0 },
  rod:          { height:10, background:"linear-gradient(90deg,#0a0a18,#3a3a6a 10%,#8888bb 50%,#3a3a6a 90%,#0a0a18)", borderRadius:5, boxShadow:"0 4px 16px rgba(0,0,0,0.6), inset 0 1px 0 rgba(255,255,255,0.12)" },
  rodLabel:     { display:"flex", alignItems:"baseline", gap:10, padding:"6px 6px 10px", borderBottom:"1px solid rgba(255,255,255,0.04)" },
  rodRowName:   { fontSize:12, fontWeight:800, letterSpacing:2, textTransform:"uppercase", color:"#6868a0" },
  rodCount:     { fontSize:11, color:"#3a3a5a" },
  shopHooksRow: { display:"flex", gap:18, flexWrap:"wrap", paddingTop:6, paddingLeft:10 },
  emptyHookMsg: { color:"#2a2a4a", fontSize:13, padding:"30px 0" },

  // individual shop hook
  shopHook:      { width:134, display:"flex", flexDirection:"column", alignItems:"center", cursor:"pointer", transition:"transform 0.22s", userSelect:"none", position:"relative" },
  shopHookHov:   { transform:"translateY(-8px)" },
  shopHookEmpty: { opacity:0.45, cursor:"default" },
  hWire:         { width:2, height:16, background:"rgba(136,136,180,0.5)" },
  hCurve:        { width:16, height:8, border:"2px solid rgba(136,136,180,0.5)", borderTop:"none", borderRadius:"0 0 12px 12px", marginBottom:4 },
  hNo:           { background:"linear-gradient(135deg,#3a3a7a,#00d2d3)", color:"#fff", fontSize:11, fontWeight:900, padding:"3px 10px", borderRadius:20, marginBottom:6, letterSpacing:0.5, transition:"box-shadow 0.2s" },
  emptySlot:     { width:"100%", height:130, display:"flex", flexDirection:"column", alignItems:"center", justifyContent:"center", gap:4, background:"rgba(255,255,255,0.02)", border:"1px dashed rgba(255,255,255,0.06)", borderRadius:14 },
  hookPhoneBody: { width:"100%", background:"rgba(255,255,255,0.04)", border:"1px solid rgba(255,255,255,0.07)", borderRadius:14, padding:"10px 10px 10px", display:"flex", flexDirection:"column", alignItems:"center", gap:3, position:"relative" },
  hBrand:        { fontSize:9, color:"#636e72", textTransform:"uppercase", letterSpacing:1.5, fontWeight:700 },
  hModel:        { fontSize:11, fontWeight:700, color:"#f1f2f6", textAlign:"center", lineHeight:1.3 },
  hSpec:         { fontSize:9, color:"#636e72" },
  hPrice:        { fontSize:15, fontWeight:800, color:"#00d2d3", marginTop:1 },
  lowBadge:      { position:"absolute", top:6, right:6, background:"#ff4757", color:"#fff", fontSize:8, fontWeight:800, padding:"2px 5px", borderRadius:5 },
  pickBtn:       { marginTop:6, width:"100%", padding:"6px 0", borderRadius:8, border:"none", background:"rgba(0,210,211,0.15)", color:"#00d2d3", cursor:"pointer", fontSize:11, fontWeight:700, transition:"background 0.2s" },
  pickBtnHov:    { background:"linear-gradient(135deg,#00d2d3,#6c5ce7)", color:"#fff" },

  // admin
  adminCols:    { display:"flex", gap:24, alignItems:"flex-start" },
  adminLeft:    { flex:"1 1 0", minWidth:0 },
  adminRight:   { width:320, flexShrink:0 },
  sectionHead:  { display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:14, paddingBottom:10, borderBottom:"1px solid rgba(255,255,255,0.07)" },
  rowAddBar:    { display:"flex", gap:6, alignItems:"center" },
  adminRow:     { background:"rgba(255,255,255,0.03)", border:"1px solid rgba(255,255,255,0.07)", borderRadius:14, padding:"14px 16px", marginBottom:16 },
  adminRowHead: { display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:12 },
  adminRowLabel:{ fontWeight:700, fontSize:14, color:"#a4b0be", cursor:"pointer", flex:1 },
  hooksGrid:    { display:"flex", flexWrap:"wrap", gap:10 },
  adminHookCard:{ width:110, background:"rgba(255,255,255,0.04)", border:"1px solid rgba(255,255,255,0.08)", borderRadius:12, padding:"8px 9px", display:"flex", flexDirection:"column", alignItems:"center", gap:4 },
  adminHookTop: { display:"flex", justifyContent:"space-between", alignItems:"center", width:"100%" },
  adminHookNo:  { fontSize:11, fontWeight:800, color:"#6c5ce7" },
  hookCardModel:{ fontSize:11, fontWeight:700, color:"#f1f2f6", textAlign:"center", lineHeight:1.2 },
  catalogueList:{ display:"flex", flexDirection:"column", gap:7, maxHeight:"calc(100vh - 180px)", overflowY:"auto" },
  catalogueItem:{ display:"flex", alignItems:"center", gap:9, padding:"9px 12px", background:"rgba(255,255,255,0.04)", border:"1px solid rgba(255,255,255,0.07)", borderRadius:10 },
  catChip:      { padding:"3px 8px", borderRadius:20, color:"#fff", fontWeight:700, whiteSpace:"nowrap" },
  assignRow:    { display:"flex", alignItems:"center", gap:10, padding:"10px 13px", background:"rgba(255,255,255,0.04)", border:"1px solid rgba(255,255,255,0.07)", borderRadius:11, cursor:"pointer" },

  // cart
  cartItem:  { display:"flex", alignItems:"center", gap:13, padding:"13px", background:"rgba(255,255,255,0.04)", border:"1px solid rgba(255,255,255,0.07)", borderRadius:12, marginBottom:8 },
  cartTotal: { display:"flex", justifyContent:"space-between", alignItems:"center", padding:"16px 0", borderTop:"1px solid rgba(255,255,255,0.1)", marginTop:6, fontSize:18, fontWeight:700 },

  // shared buttons
  smBtn:       { padding:"6px 12px", borderRadius:8, border:"none", background:"linear-gradient(135deg,#00d2d3,#6c5ce7)", color:"#fff", cursor:"pointer", fontSize:12, fontWeight:700, whiteSpace:"nowrap" },
  smDelBtn:    { padding:"6px 10px", borderRadius:8, border:"1px solid rgba(255,71,87,0.3)", background:"rgba(255,71,87,0.1)", color:"#ff4757", cursor:"pointer", fontSize:12, fontWeight:700 },
  tinyBtn:     { padding:"4px 8px", borderRadius:7, border:"1px solid rgba(0,210,211,0.3)", background:"rgba(0,210,211,0.08)", color:"#00d2d3", cursor:"pointer", fontSize:11 },
  tinyEditBtn: { padding:"4px 8px", borderRadius:7, border:"1px solid rgba(255,200,0,0.3)", background:"rgba(255,200,0,0.08)", cursor:"pointer", fontSize:11 },
  tinyDelBtn:  { padding:"4px 8px", borderRadius:7, border:"1px solid rgba(255,71,87,0.3)", background:"rgba(255,71,87,0.08)", color:"#ff4757", cursor:"pointer", fontSize:11 },
  bigBtn:      { width:"100%", padding:"12px", borderRadius:12, border:"none", background:"linear-gradient(135deg,#00d2d3,#6c5ce7)", color:"#fff", cursor:"pointer", fontSize:15, fontWeight:700 },
  cancelBtn:   { flex:1, padding:"11px", borderRadius:10, border:"1px solid rgba(255,255,255,0.1)", background:"transparent", color:"#a4b0be", cursor:"pointer", fontSize:14 },

  // form
  smInput:  { padding:"7px 10px", background:"rgba(255,255,255,0.07)", border:"1px solid rgba(255,255,255,0.1)", borderRadius:8, color:"#f1f2f6", fontSize:13, outline:"none" },
  formGrid: { display:"grid", gridTemplateColumns:"1fr 1fr", gap:10, marginBottom:4 },
  fg:       { display:"flex", flexDirection:"column", gap:5 },
  lbl:      { fontSize:10, color:"#636e72", fontWeight:700, letterSpacing:1, textTransform:"uppercase" },
  inp:      { padding:"9px 11px", background:"rgba(255,255,255,0.06)", border:"1px solid rgba(255,255,255,0.1)", borderRadius:8, color:"#f1f2f6", fontSize:14, outline:"none" },
  emojiBtn:    { padding:"6px 8px", borderRadius:8, border:"1px solid rgba(255,255,255,0.1)", background:"transparent", cursor:"pointer", fontSize:20 },
  emojiBtnOn:  { border:"2px solid #00d2d3", background:"rgba(0,210,211,0.12)" },

  // modal
  overlay:     { position:"fixed", inset:0, background:"rgba(0,0,0,0.82)", zIndex:200, display:"flex", alignItems:"center", justifyContent:"center", backdropFilter:"blur(6px)" },
  modalBox:    { background:"#10102a", border:"1px solid rgba(255,255,255,0.1)", borderRadius:20, padding:28, width:"92%", maxWidth:500, maxHeight:"88vh", overflowY:"auto", boxShadow:"0 30px 80px rgba(0,0,0,0.7)" },
  modalTitle:  { textAlign:"center", fontSize:17, fontWeight:800, margin:"0 0 16px" },
  modalFooter: { display:"flex", gap:10, marginTop:18 },
};