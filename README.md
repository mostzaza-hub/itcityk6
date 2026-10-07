[index.html](https://github.com/user-attachments/files/33169410/index.html)
<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Sales Management System | ITCITY</title>

<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;500;600;700&display=swap" rel="stylesheet">

<script src="https://cdn.tailwindcss.com"></script>
<script>
tailwind.config = {
  theme: {
    extend: {
      fontFamily: { sans: ['Prompt', 'ui-sans-serif', 'system-ui'] },
      colors: {
        primary: {
          50:'#eff8ff',100:'#dbeefe',200:'#bfe2fe',300:'#93d0fd',
          400:'#60b4fa',500:'#3b96f6',600:'#2577eb',700:'#1d60d8',
          800:'#1e4faf',900:'#1e448a',950:'#172b54'
        }
      },
      boxShadow: { soft: '0 2px 16px -4px rgba(37,119,235,.14)' }
    }
  }
}
</script>

<!-- React / Recharts -->
<script src="https://unpkg.com/react@18/umd/react.production.min.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js" crossorigin></script>
<script src="https://unpkg.com/prop-types@15/prop-types.min.js" crossorigin></script>
<script src="https://unpkg.com/recharts@2.12.7/umd/Recharts.js" crossorigin></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

<!-- Firebase (compat) -->
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-firestore-compat.js"></script>

<style>
  body { font-family:'Prompt',sans-serif; }
  ::-webkit-scrollbar{width:9px;height:9px}
  ::-webkit-scrollbar-thumb{background:#bfe2fe;border-radius:9px}
  ::-webkit-scrollbar-thumb:hover{background:#93d0fd}
  .fade-in{animation:fi .25s ease}
  @keyframes fi{from{opacity:0;transform:translateY(6px)}to{opacity:1;transform:none}}
  input[type=date]::-webkit-calendar-picker-indicator{cursor:pointer}
</style>
</head>
<body class="bg-slate-50 text-slate-800">
<div id="root"></div>

<script type="text/babel" data-presets="react">
/* ============================================================
   0) CONFIG
   ============================================================ */
const firebaseConfig = {
  apiKey: "AIzaSyBWeGo8AQRkB1PCA-UtkukezEtLr9E9jMo",
  authDomain: "itcity-sales.firebaseapp.com",
  projectId: "itcity-sales",
  storageBucket: "itcity-sales.firebasestorage.app",
  messagingSenderId: "178919104426",
  appId: "1:178919104426:web:92f42250ae837f2cb2d5d3"
};
firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const db   = firebase.firestore();
let _sec = null;
const secondaryAuth = () => {
  if (!_sec) _sec = firebase.initializeApp(firebaseConfig, 'Secondary');
  return _sec.auth();
};

const { useState, useEffect, useMemo, useCallback, useRef, createContext, useContext } = React;
const { ResponsiveContainer, LineChart, Line, BarChart, Bar, PieChart, Pie, Cell,
        XAxis, YAxis, CartesianGrid, Tooltip, Legend } = Recharts;

const DEFAULT_BRANDS = [
  { name:'OPPO',    color:'#1b5e20', order:1  },
  { name:'VIVO',    color:'#2196f3', order:2  },
  { name:'SAMSUNG', color:'#1565c0', order:3  },
  { name:'XIAOMI',  color:'#ff9800', order:4  },
  { name:'APPLE',   color:'#000000', order:5  },
  { name:'INFINIX', color:'#66bb6a', order:6  },
  { name:'HONOR',   color:'#757575', order:7  },
  { name:'REALME',  color:'#fdd835', order:8  },
  { name:'iQOO',    color:'#e65100', order:9  },
  { name:'TECNO',   color:'#9e9e9e', order:10 },
  { name:'Nubia',   color:'#b71c1c', order:11 }
];
const POSITIONS = ['SM','PL','PC'];
const STORES = [
  { key:'ITCITY',  label:'ITCITY',  color:'#2577eb', auto:true  },
  { key:'BANANA',  label:'Banana',  color:'#f5c518', auto:false },
  { key:'JAYMART', label:'Jaymart', color:'#e53935', auto:false },
  { key:'TELEWIZ', label:'Telewiz', color:'#2e7d32', auto:false }
];
const DEFAULT_SETTINGS = {
  kpiSamsung: 32,
  kpiTecnoInfinix: 12,
  filmTiers: [ {min:70,rate:25},{min:85,rate:32},{min:100,rate:40},{min:120,rate:50} ],
  simTiers : [ {min:14,rate:10},{min:17,rate:15},{min:20,rate:20} ],
  filmTargetPct: 100,
  simRatioTarget: 4.5
};
const MONTHS_TH = ['มกราคม','กุมภาพันธ์','มีนาคม','เมษายน','พฤษภาคม','มิถุนายน',
                   'กรกฎาคม','สิงหาคม','กันยายน','ตุลาคม','พฤศจิกายน','ธันวาคม'];

/* ============================================================
   1) UTILITIES
   ============================================================ */
const pad = n => String(n).padStart(2,'0');
const todayStr = () => { const d=new Date(); return `${d.getFullYear()}-${pad(d.getMonth()+1)}-${pad(d.getDate())}`; };
const nf  = n => (Number(n)||0).toLocaleString('th-TH');
const nf2 = n => (Number(n)||0).toLocaleString('th-TH',{minimumFractionDigits:2,maximumFractionDigits:2});
const pctOf = (a,b) => (Number(b)>0 ? (Number(a)/Number(b))*100 : 0);
const monthRange = (y,m) => ({ from:`${y}-${pad(m)}-01`, to:`${y}-${pad(m)}-31` });
const daysInMonth = (y,m) => new Date(y,m,0).getDate();
const num = v => { const n = parseFloat(v); return isNaN(n)?0:n; };
const cls = (...a) => a.filter(Boolean).join(' ');

function tierRate(tiers, value){
  let r = 0;
  [...(tiers||[])].sort((a,b)=>a.min-b.min).forEach(t => { if (value >= t.min - 1e-9) r = t.rate; });
  return r;
}
function download(blob, filename){
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = filename; document.body.appendChild(a); a.click();
  document.body.removeChild(a); setTimeout(()=>URL.revokeObjectURL(url), 1500);
}
function esc(s){ return String(s==null?'':s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;'); }
function exportExcel(filename, headers, rows, title){
  let h = '<meta charset="utf-8"><table border="1" cellspacing="0">';
  if (title) h += `<tr><td colspan="${headers.length}"><b>${esc(title)}</b></td></tr>`;
  h += '<tr>' + headers.map(x=>`<th style="background:#dbeefe">${esc(x)}</th>`).join('') + '</tr>';
  rows.forEach(r => { h += '<tr>' + r.map(c=>`<td>${esc(c)}</td>`).join('') + '</tr>'; });
  h += '</table>';
  download(new Blob(['\ufeff'+h], {type:'application/vnd.ms-excel;charset=utf-8'}), filename+'.xls');
}
function exportCSV(filename, headers, rows){
  const line = a => a.map(c => `"${String(c==null?'':c).replace(/"/g,'""')}"`).join(',');
  const csv = [line(headers), ...rows.map(line)].join('\r\n');
  download(new Blob(['\ufeff'+csv], {type:'text/csv;charset=utf-8'}), filename+'.csv');
}
function resizeImage(file, maxSize=2*1024*1024){
  return new Promise((resolve,reject)=>{
    if (!file.type.startsWith('image/')) return reject(new Error('กรุณาเลือกไฟล์รูปภาพเท่านั้น'));
    if (file.size > maxSize) return reject(new Error(`ไฟล์ใหญ่เกินไป (${(file.size/1048576).toFixed(2)} MB) จำกัดไม่เกิน 2 MB`));
    const fr = new FileReader();
    fr.onload = () => {
      const img = new Image();
      img.onload = () => {
        const MAX = 320, s = Math.min(1, MAX/Math.max(img.width,img.height));
        const c = document.createElement('canvas');
        c.width = Math.round(img.width*s); c.height = Math.round(img.height*s);
        c.getContext('2d').drawImage(img,0,0,c.width,c.height);
        resolve(c.toDataURL('image/jpeg',.85));
      };
      img.onerror = () => reject(new Error('ไฟล์รูปเสียหาย'));
      img.src = fr.result;
    };
    fr.onerror = () => reject(new Error('อ่านไฟล์ไม่สำเร็จ'));
    fr.readAsDataURL(file);
  });
}

/* ============================================================
   2) ICONS
   ============================================================ */
const P = {
  home:['M3 10.6 12 3l9 7.6','M5 9.6V21h14V9.6'],
  plus:['M12 3a9 9 0 1 0 0 18 9 9 0 0 0 0-18','M12 8v8','M8 12h8'],
  calendar:['M5 5h14a1 1 0 0 1 1 1v14a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1V6a1 1 0 0 1 1-1z','M16 3v4','M8 3v4','M4 10h16'],
  file:['M14 3H7a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V8z','M14 3v5h5','M9 13h6','M9 17h5'],
  award:['M12 4a5 5 0 1 0 0 10 5 5 0 0 0 0-10','M8.5 13.5 7 22l5-3 5 3-1.5-8.5'],
  pie:['M12 3v9h9','M21 12a9 9 0 1 1-9-9'],
  gear:['M12 9a3 3 0 1 0 0 6 3 3 0 0 0 0-6','M19.5 13.6a1.6 1.6 0 0 0 .3 1.8l.1.1a2 2 0 1 1-2.8 2.8l-.1-.1a1.6 1.6 0 0 0-2.7 1.1V21a2 2 0 1 1-4 0v-.1a1.6 1.6 0 0 0-2.7-1.1l-.1.1a2 2 0 1 1-2.8-2.8l.1-.1a1.6 1.6 0 0 0-1.1-2.7H3a2 2 0 1 1 0-4h.1a1.6 1.6 0 0 0 1.1-2.7l-.1-.1a2 2 0 1 1 2.8-2.8l.1.1a1.6 1.6 0 0 0 2.7-1.1V3a2 2 0 1 1 4 0v.1a1.6 1.6 0 0 0 2.7 1.1l.1-.1a2 2 0 1 1 2.8 2.8l-.1.1a1.6 1.6 0 0 0 1.1 2.7H21a2 2 0 1 1 0 4h-.1a1.6 1.6 0 0 0-1.4 1z'],
  logout:['M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4','M16 17l5-5-5-5','M21 12H9'],
  menu:['M3 6h18','M3 12h18','M3 18h18'],
  x:['M18 6 6 18','M6 6l12 12'],
  edit:['M12 20h9','M16.5 3.5a2.1 2.1 0 0 1 3 3L7 19l-4 1 1-4z'],
  trash:['M3 6h18','M8 6V4h8v2','M19 6l-1 14H6L5 6','M10 11v6','M14 11v6'],
  download:['M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4','M7 10l5 5 5-5','M12 15V3'],
  users:['M17 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2','M9.5 3a4 4 0 1 0 0 8 4 4 0 0 0 0-8','M22 21v-2a4 4 0 0 0-3-3.9'],
  tag:['M20.6 13.4 12 22l-9-9V3h10l7.6 7.6a2 2 0 0 1 0 2.8z','M7.5 7.5h.01'],
  target:['M12 3a9 9 0 1 0 0 18 9 9 0 0 0 0-18','M12 7a5 5 0 1 0 0 10 5 5 0 0 0 0-10','M12 11a1 1 0 1 0 0 2 1 1 0 0 0 0-2'],
  chev:['M6 9l6 6 6-6'],
  check:['M20 6 9 17l-5-5'],
  alert:['M10.3 3.9 1.8 18a2 2 0 0 0 1.7 3h17a2 2 0 0 0 1.7-3L13.7 3.9a2 2 0 0 0-3.4 0z','M12 9v4','M12 17h.01'],
  search:['M11 4a7 7 0 1 0 0 14 7 7 0 0 0 0-14','M21 21l-4.3-4.3'],
  phone:['M7 2h10a1 1 0 0 1 1 1v18a1 1 0 0 1-1 1H7a1 1 0 0 1-1-1V3a1 1 0 0 1 1-1z','M11 18.5h2'],
  sim:['M5 4h9l5 5v11a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1V5a1 1 0 0 1 1-1z','M8 12h8v6H8z'],
  film:['M4 4h16v16H4z','M8 4v16','M16 4v16','M4 9h4','M4 15h4','M16 9h4','M16 15h4'],
  cash:['M2 6h20v12H2z','M12 9a3 3 0 1 0 0 6 3 3 0 0 0 0-6','M5 9h.01','M19 15h.01'],
  trend:['M22 7 13.5 15.5l-5-5L2 17','M16 7h6v6'],
  save:['M5 3h11l3 3v15H5z','M8 3v6h8V3','M8 21v-7h8v7'],
  lock:['M5 11h14v10H5z','M8 11V7a4 4 0 1 1 8 0v4'],
  mail:['M3 5h18v14H3z','m3 6 9 7 9-7'],
  store:['M4 9h16v12H4z','M3 4h18l-1 5H4z','M9 21v-6h6v6'],
  eye:['M2 12s3.5-7 10-7 10 7 10 7-3.5 7-10 7S2 12 2 12z','M12 9a3 3 0 1 0 0 6 3 3 0 0 0 0-6']
};
function Icon({n, className='w-5 h-5', sw=1.8}){
  const d = P[n] || P.check;
  return (
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={sw}
         strokeLinecap="round" strokeLinejoin="round" className={className}>
      {d.map((p,i)=><path key={i} d={p} />)}
    </svg>
  );
}

/* ============================================================
   3) UI PRIMITIVES
   ============================================================ */
const Card = ({children, className='', ...r}) =>
  <div className={cls('bg-white rounded-2xl border border-slate-200/80 shadow-soft', className)} {...r}>{children}</div>;

const CardHead = ({title, sub, icon, right}) => (
  <div className="flex items-start justify-between gap-3 px-5 pt-5 pb-3">
    <div className="flex items-center gap-3 min-w-0">
      {icon && <span className="grid place-items-center w-10 h-10 rounded-xl bg-primary-50 text-primary-600 shrink-0"><Icon n={icon}/></span>}
      <div className="min-w-0">
        <h3 className="font-semibold text-slate-800 truncate">{title}</h3>
        {sub && <p className="text-xs text-slate-500 truncate">{sub}</p>}
      </div>
    </div>
    {right}
  </div>
);

const Btn = ({variant='primary', size='md', icon, children, className='', ...r}) => {
  const v = {
    primary:'bg-primary-600 hover:bg-primary-700 text-white shadow-sm',
    soft:'bg-primary-50 hover:bg-primary-100 text-primary-700',
    ghost:'hover:bg-slate-100 text-slate-600',
    outline:'border border-slate-300 hover:bg-slate-50 text-slate-700 bg-white',
    danger:'bg-rose-600 hover:bg-rose-700 text-white',
    dangerSoft:'bg-rose-50 hover:bg-rose-100 text-rose-600',
    success:'bg-emerald-600 hover:bg-emerald-700 text-white'
  }[variant];
  const s = { sm:'px-2.5 py-1.5 text-xs gap-1', md:'px-4 py-2.5 text-sm gap-2', lg:'px-5 py-3 text-sm gap-2' }[size];
  return <button className={cls('inline-flex items-center justify-center rounded-xl font-medium transition active:scale-[.98] disabled:opacity-50 disabled:pointer-events-none', v, s, className)} {...r}>
    {icon && <Icon n={icon} className={size==='sm'?'w-3.5 h-3.5':'w-4 h-4'} />}{children}
  </button>;
};

const Field = ({label, required, hint, children}) => (
  <label className="block">
    <span className="block text-xs font-medium text-slate-600 mb-1.5">
      {label}{required && <span className="text-rose-500"> *</span>}
    </span>
    {children}
    {hint && <span className="block text-[11px] text-slate-400 mt-1">{hint}</span>}
  </label>
);
const inputCls = 'w-full rounded-xl border border-slate-300 bg-white px-3.5 py-2.5 text-sm outline-none transition focus:border-primary-500 focus:ring-4 focus:ring-primary-100 placeholder:text-slate-400';
const Input  = p => <input  {...p} className={cls(inputCls, p.className)} />;
const Select = p => <select {...p} className={cls(inputCls,'appearance-none bg-[url("data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' fill=\'none\' stroke=\'%2394a3b8\' stroke-width=\'2\' viewBox=\'0 0 24 24\'><path d=\'M6 9l6 6 6-6\'/></svg>")] bg-[length:18px] bg-[right_.7rem_center] bg-no-repeat pr-10', p.className)} />;

const Badge = ({children, tone='blue', className=''}) => {
  const t = { blue:'bg-primary-50 text-primary-700', green:'bg-emerald-50 text-emerald-700',
    red:'bg-rose-50 text-rose-700', amber:'bg-amber-50 text-amber-700', gray:'bg-slate-100 text-slate-600' }[tone];
  return <span className={cls('inline-flex items-center gap-1 px-2 py-0.5 rounded-lg text-[11px] font-medium', t, className)}>{children}</span>;
};

const Stat = ({label, value, unit, icon, tone='blue', foot}) => {
  const t = { blue:'from-primary-500 to-primary-600', green:'from-emerald-500 to-emerald-600',
    amber:'from-amber-500 to-orange-500', violet:'from-indigo-500 to-violet-600', rose:'from-rose-500 to-pink-600' }[tone];
  return (
    <Card className="p-4 fade-in">
      <div className="flex items-center gap-3">
        <span className={cls('grid place-items-center w-11 h-11 rounded-xl text-white bg-gradient-to-br shrink-0', t)}><Icon n={icon}/></span>
        <div className="min-w-0 flex-1">
          <p className="text-[11px] text-slate-500 truncate">{label}</p>
          <p className="text-xl font-bold text-slate-800 leading-tight truncate">
            {value}<span className="text-xs font-medium text-slate-400 ml-1">{unit}</span>
          </p>
        </div>
      </div>
      {foot && <div className="mt-2 pt-2 border-t border-slate-100 text-[11px] text-slate-500">{foot}</div>}
    </Card>
  );
};

const Modal = ({open, onClose, title, children, size='md'}) => {
  useEffect(()=>{ document.body.style.overflow = open ? 'hidden' : ''; return ()=>{document.body.style.overflow='';}; },[open]);
  if (!open) return null;
  const w = { sm:'max-w-sm', md:'max-w-lg', lg:'max-w-3xl', xl:'max-w-5xl' }[size];
  return (
    <div className="fixed inset-0 z-50 flex items-end sm:items-center justify-center p-0 sm:p-4">
      <div className="absolute inset-0 bg-slate-900/50 backdrop-blur-[2px]" onClick={onClose}/>
      <div className={cls('relative bg-white w-full rounded-t-3xl sm:rounded-2xl shadow-2xl max-h-[92vh] overflow-y-auto fade-in', w)}>
        <div className="sticky top-0 bg-white/95 backdrop-blur border-b border-slate-100 px-5 py-4 flex items-center justify-between z-10">
          <h3 className="font-semibold text-slate-800">{title}</h3>
          <button onClick={onClose} className="p-1.5 rounded-lg hover:bg-slate-100 text-slate-500"><Icon n="x"/></button>
        </div>
        <div className="p-5">{children}</div>
      </div>
    </div>
  );
};

const Table = ({head, children, className=''}) => (
  <div className={cls('overflow-x-auto', className)}>
    <table className="w-full text-sm min-w-[640px]">
      <thead>
        <tr className="bg-primary-50/70 text-primary-900">
          {head.map((h,i)=>(
            <th key={i} className={cls('px-3 py-2.5 font-semibold text-xs whitespace-nowrap',
              typeof h==='object'&&h.align==='right'?'text-right':typeof h==='object'&&h.align==='center'?'text-center':'text-left')}>
              {typeof h==='object'?h.label:h}
            </th>
          ))}
        </tr>
      </thead>
      <tbody className="divide-y divide-slate-100">{children}</tbody>
    </table>
  </div>
);
const Empty = ({cols, text='ไม่พบข้อมูล'}) => (
  <tr><td colSpan={cols} className="text-center py-10 text-slate-400 text-sm">{text}</td></tr>
);
const Spinner = ({className='w-6 h-6'}) =>
  <svg className={cls('animate-spin text-primary-600', className)} viewBox="0 0 24 24" fill="none">
    <circle cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="3" opacity=".2"/>
    <path d="M22 12a10 10 0 0 0-10-10" stroke="currentColor" strokeWidth="3" strokeLinecap="round"/>
  </svg>;

/* ---- Toast ---- */
const ToastCtx = createContext(null);
const useToast = () => useContext(ToastCtx);
function ToastHost({children}){
  const [list,setList] = useState([]);
  const push = useCallback((type,msg)=>{
    const id = Math.random().toString(36).slice(2);
    setList(l=>[...l,{id,type,msg}]);
    setTimeout(()=>setList(l=>l.filter(x=>x.id!==id)), 3800);
  },[]);
  const api = useMemo(()=>({
    success: m=>push('success',m), error: m=>push('error',m), info: m=>push('info',m)
  }),[push]);
  return (
    <ToastCtx.Provider value={api}>
      {children}
      <div className="fixed top-4 right-4 z-[100] space-y-2 w-[calc(100vw-2rem)] max-w-sm">
        {list.map(t=>(
          <div key={t.id} className={cls('fade-in flex items-start gap-3 rounded-xl px-4 py-3 shadow-lg border text-sm',
            t.type==='success'?'bg-emerald-50 border-emerald-200 text-emerald-800'
            : t.type==='error'?'bg-rose-50 border-rose-200 text-rose-800'
            : 'bg-primary-50 border-primary-200 text-primary-800')}>
            <Icon n={t.type==='success'?'check':t.type==='error'?'alert':'eye'} className="w-5 h-5 shrink-0 mt-0.5"/>
            <span className="flex-1">{t.msg}</span>
          </div>
        ))}
      </div>
    </ToastCtx.Provider>
  );
}
/* ---- Confirm ---- */
const ConfirmCtx = createContext(null);
const useConfirm = () => useContext(ConfirmCtx);
function ConfirmHost({children}){
  const [st,setSt] = useState(null);
  const confirm = useCallback(opt => new Promise(res => setSt({...opt, res})), []);
  const close = ok => { st?.res(ok); setSt(null); };
  return (
    <ConfirmCtx.Provider value={confirm}>
      {children}
      <Modal open={!!st} onClose={()=>close(false)} title={st?.title||'ยืนยันการทำงาน'} size="sm">
        <div className="flex gap-3">
          <span className={cls('grid place-items-center w-11 h-11 rounded-full shrink-0',
            st?.danger?'bg-rose-100 text-rose-600':'bg-primary-100 text-primary-600')}><Icon n="alert"/></span>
          <p className="text-sm text-slate-600 pt-2">{st?.message}</p>
        </div>
        <div className="flex gap-2 justify-end mt-6">
          <Btn variant="outline" onClick={()=>close(false)}>ยกเลิก</Btn>
          <Btn variant={st?.danger?'danger':'primary'} onClick={()=>close(true)}>{st?.okText||'ยืนยัน'}</Btn>
        </div>
      </Modal>
    </ConfirmCtx.Provider>
  );
}

/* ============================================================
   4) DATA LAYER
   ============================================================ */
const colUsers=()=>db.collection('users'), colBrands=()=>db.collection('brands'),
      colSales=()=>db.collection('sales'), colTargets=()=>db.collection('targets'),
      colMS=()=>db.collection('marketshare'), docSettings=()=>db.collection('config').doc('settings');

async function seedIfEmpty(){
  const b = await colBrands().limit(1).get();
  if (b.empty){
    const batch = db.batch();
    DEFAULT_BRANDS.forEach(x => batch.set(colBrands().doc(x.name), x));
    await batch.commit();
  }
  const s = await docSettings().get();
  if (!s.exists) await docSettings().set(DEFAULT_SETTINGS);
}
async function fetchSalesRange(from,to){
  const s = await colSales().where('date','>=',from).where('date','<=',to).get();
  return s.docs.map(d=>({id:d.id,...d.data()}));
}

/* ---- aggregation helpers ---- */
const blankAgg = () => ({units:0, amount:0, sim:0, film:0, packFee:0});
function sumRows(rows){
  return rows.reduce((a,r)=>({
    units:a.units+num(r.units), amount:a.amount+num(r.amount), sim:a.sim+num(r.sim),
    film:a.film+num(r.film), packFee:a.packFee+num(r.packFee)
  }), blankAgg());
}
function groupBy(rows, key){
  const m = {};
  rows.forEach(r=>{ const k = r[key] || '-'; if(!m[k]) m[k]=[]; m[k].push(r); });
  return m;
}

/* ---- ratio helpers (Dashboard) ---- */
function ratioInfo(units, qty, redAbove, greenBelow){
  if (!qty || qty<=0) return { text:'N/A', tone:'gray', value:null };
  const v = units/qty;
  const tone = v > redAbove ? 'red' : (v <= greenBelow ? 'green' : 'amber');
  return { text:`1:${v.toFixed(v%1===0?0:1)}`, tone, value:v };
}
const toneText = t => ({ red:'text-rose-600', green:'text-emerald-600', amber:'text-amber-600', gray:'text-slate-400' }[t]);

function filmSummaryText(film, units, targetPct=100){
  if (!units) return '-';
  const p = pctOf(film, units);
  const need = Math.max(0, Math.ceil(units*targetPct/100) - film);
  return `${nf(film)} Film / ${nf(units)} เครื่อง (${p.toFixed(1)}%) · ${need===0?'ผ่าน':'ไม่ผ่าน'}${need>0?` · ต้องทำอีก ${need} แผ่น`:''}`;
}
function simSummaryText(sim, units, ratioTarget=4.5){
  if (!units) return '-';
  if (!sim) return `0 SIM / ${nf(units)} เครื่อง (N/A) · ไม่ผ่าน · ต้องทำอีก ${Math.ceil(units/ratioTarget)} ซิม`;
  const p = pctOf(sim, units);
  const need = Math.max(0, Math.ceil(units/ratioTarget) - sim);
  return `${nf(sim)} SIM / ${nf(units)} เครื่อง (${p.toFixed(1)}%) · ${need===0?'ผ่าน':'ไม่ผ่าน'}${need>0?` · ต้องทำอีก ${need} ซิม`:''}`;
}

/* ============================================================
   5) APP CONTEXT
   ============================================================ */
const AppCtx = createContext(null);
const useApp = () => useContext(AppCtx);

/* ============================================================
   6) LOGIN PAGE
   ============================================================ */
function LoginPage(){
  const [email,setEmail] = useState('');
  const [pass,setPass]   = useState('');
  const [busy,setBusy]   = useState(false);
  const [err,setErr]     = useState('');

  const submit = async e => {
    e.preventDefault(); setErr('');
    if (!email || !pass) return setErr('กรุณากรอกอีเมลและรหัสผ่าน');
    setBusy(true);
    try { await auth.signInWithEmailAndPassword(email.trim(), pass); }
    catch(ex){
      const map = {
        'auth/invalid-email':'รูปแบบอีเมลไม่ถูกต้อง',
        'auth/user-not-found':'ไม่พบบัญชีผู้ใช้นี้',
        'auth/wrong-password':'รหัสผ่านไม่ถูกต้อง',
        'auth/invalid-credential':'อีเมลหรือรหัสผ่านไม่ถูกต้อง',
        'auth/too-many-requests':'พยายามเข้าสู่ระบบหลายครั้งเกินไป กรุณารอสักครู่'
      };
      setErr(map[ex.code] || 'เข้าสู่ระบบไม่สำเร็จ: '+ex.message);
      setBusy(false);
    }
  };

  return (
    <div className="min-h-screen grid lg:grid-cols-2 bg-white">
      {/* Brand side */}
      <div className="hidden lg:flex flex-col justify-between p-12 bg-gradient-to-br from-primary-600 via-primary-700 to-primary-900 text-white relative overflow-hidden">
        <div className="absolute -right-24 -top-24 w-96 h-96 rounded-full bg-white/10"/>
        <div className="absolute -left-16 bottom-10 w-72 h-72 rounded-full bg-white/5"/>
        <div className="relative">
          <div className="flex items-center gap-3">
            <span className="grid place-items-center w-12 h-12 rounded-2xl bg-white/15 backdrop-blur"><Icon n="trend" className="w-7 h-7"/></span>
            <div>
              <p className="font-bold text-lg leading-tight">Sales Management</p>
              <p className="text-primary-200 text-xs">ระบบรายงานและจัดการยอดขาย</p>
            </div>
          </div>
        </div>
        <div className="relative space-y-6">
          <h1 className="text-4xl font-bold leading-snug">บริหารยอดขาย<br/>ครบจบในที่เดียว</h1>
          <p className="text-primary-100/90 max-w-md">ติดตามยอดขายรายวัน เป้าหมายรายเดือน KPI ค่าคอมมิชชั่น และ Market Share ในตึก แบบเรียลไทม์</p>
          <div className="grid grid-cols-2 gap-3 max-w-md">
            {[['home','Dashboard'],['file','รายงานรายเดือน'],['award','KPI & คอมมิชชั่น'],['pie','Market Share']].map(([i,t])=>(
              <div key={t} className="flex items-center gap-2 rounded-xl bg-white/10 backdrop-blur px-3 py-2.5 text-sm">
                <Icon n={i} className="w-4 h-4"/>{t}
              </div>
            ))}
          </div>
        </div>
        <p className="relative text-xs text-primary-200/70">© {new Date().getFullYear()} ITCITY Sales System</p>
      </div>

      {/* Form side */}
      <div className="flex items-center justify-center p-6 sm:p-12">
        <div className="w-full max-w-sm">
          <div className="lg:hidden flex items-center gap-3 mb-8">
            <span className="grid place-items-center w-11 h-11 rounded-2xl bg-primary-600 text-white"><Icon n="trend"/></span>
            <div><p className="font-bold text-slate-800">Sales Management</p><p className="text-xs text-slate-500">ระบบรายงานและจัดการยอดขาย</p></div>
          </div>
          <h2 className="text-2xl font-bold text-slate-800">เข้าสู่ระบบ</h2>
          <p className="text-sm text-slate-500 mt-1 mb-7">กรอกอีเมลและรหัสผ่านที่ได้รับจากผู้ดูแลระบบ</p>

          <form onSubmit={submit} className="space-y-4">
            <Field label="อีเมล" required>
              <div className="relative">
                <Icon n="mail" className="w-4 h-4 absolute left-3.5 top-1/2 -translate-y-1/2 text-slate-400"/>
                <Input type="email" value={email} onChange={e=>setEmail(e.target.value)}
                       placeholder="you@company.com" className="pl-10" autoComplete="username"/>
              </div>
            </Field>
            <Field label="รหัสผ่าน" required>
              <div className="relative">
                <Icon n="lock" className="w-4 h-4 absolute left-3.5 top-1/2 -translate-y-1/2 text-slate-400"/>
                <Input type="password" value={pass} onChange={e=>setPass(e.target.value)}
                       placeholder="••••••••" className="pl-10" autoComplete="current-password"/>
              </div>
            </Field>
            {err && (
              <div className="flex items-start gap-2 rounded-xl bg-rose-50 border border-rose-200 px-3.5 py-2.5 text-sm text-rose-700">
                <Icon n="alert" className="w-4 h-4 mt-0.5 shrink-0"/><span>{err}</span>
              </div>
            )}
            <Btn type="submit" size="lg" className="w-full" disabled={busy}>
              {busy ? <Spinner className="w-5 h-5 text-white"/> : <><Icon n="logout" className="w-4 h-4"/> เข้าสู่ระบบ</>}
            </Btn>
          </form>
          <p className="text-[11px] text-slate-400 mt-6 text-center">
            ผู้ใช้คนแรกที่เข้าสู่ระบบจะได้รับสิทธิ์ Admin โดยอัตโนมัติ
          </p>
        </div>
      </div>
    </div>
  );
}

/* ============================================================
   7) PAGE: SALES ENTRY (บันทึกข้อมูลการขาย)
   ============================================================ */
function PageSalesEntry(){
  const { brands, users, profile } = useApp();
  const toast = useToast(); const confirm = useConfirm();
  const empty = { date:todayStr(), sellerId:'', brand:'', units:'', amount:'', sim:'', film:'', packFee:'' };
  const [form,setForm] = useState(empty);
  const [editId,setEditId] = useState(null);
  const [errs,setErrs] = useState({});
  const [busy,setBusy] = useState(false);
  const [history,setHistory] = useState([]);
  const [loading,setLoading] = useState(true);

  const sellers = useMemo(()=>users.filter(u=>u.active!==false), [users]);

  const load = useCallback(async ()=>{
    setLoading(true);
    try{
      const s = await colSales().orderBy('createdAt','desc').limit(20).get();
      setHistory(s.docs.map(d=>({id:d.id,...d.data()})));
    }catch(e){ toast.error('โหลดประวัติไม่สำเร็จ: '+e.message); }
    setLoading(false);
  },[]);
  useEffect(()=>{ load(); },[load]);

  const set = (k,v) => { setForm(f=>({...f,[k]:v})); setErrs(e=>({...e,[k]:null})); };

  const validate = () => {
    const e = {};
    if (!form.date) e.date='กรุณาเลือกวันที่';
    if (!form.sellerId) e.sellerId='กรุณาเลือกผู้ขาย';
    if (!form.brand) e.brand='กรุณาเลือกแบรนด์';
    if (form.units==='' || num(form.units)<0) e.units='กรุณากรอกจำนวนเครื่อง';
    if (form.amount==='' || num(form.amount)<0) e.amount='กรุณากรอกยอดขาย';
    setErrs(e); return Object.keys(e).length===0;
  };

  const save = async ev => {
    ev.preventDefault();
    if (!validate()) { toast.error('กรุณากรอกข้อมูลให้ครบถ้วน'); return; }
    setBusy(true);
    const seller = users.find(u=>u.id===form.sellerId);
    const payload = {
      date: form.date, sellerId: form.sellerId,
      sellerName: seller ? (seller.nickname || seller.fullName) : '-',
      sellerPosition: seller?.position || '',
      brand: form.brand,
      units: num(form.units), amount: num(form.amount),
      sim: num(form.sim), film: num(form.film), packFee: num(form.packFee),
      updatedAt: firebase.firestore.FieldValue.serverTimestamp(),
      updatedBy: profile.fullName
    };
    try{
      if (editId) {
        await colSales().doc(editId).update(payload);
        toast.success('แก้ไขรายการขายเรียบร้อยแล้ว');
      } else {
        await colSales().add({...payload,
          createdAt: firebase.firestore.FieldValue.serverTimestamp(),
          createdBy: profile.fullName });
        toast.success('บันทึกการขายเรียบร้อยแล้ว');
      }
      setForm({...empty, date:form.date, sellerId:form.sellerId});
      setEditId(null); load();
    }catch(e){ toast.error('บันทึกไม่สำเร็จ: '+e.message); }
    setBusy(false);
  };

  const startEdit = r => {
    setEditId(r.id);
    setForm({ date:r.date, sellerId:r.sellerId, brand:r.brand, units:r.units,
      amount:r.amount, sim:r.sim||'', film:r.film||'', packFee:r.packFee||'' });
    window.scrollTo({top:0, behavior:'smooth'});
  };
  const remove = async r => {
    const ok = await confirm({ title:'ยืนยันการลบ', danger:true, okText:'ลบรายการ',
      message:`ต้องการลบรายการขายของ "${r.sellerName}" แบรนด์ ${r.brand} วันที่ ${r.date} ใช่หรือไม่? การลบไม่สามารถย้อนกลับได้` });
    if (!ok) return;
    try{ await colSales().doc(r.id).delete(); toast.success('ลบรายการเรียบร้อยแล้ว'); load(); }
    catch(e){ toast.error('ลบไม่สำเร็จ: '+e.message); }
  };

  const Err = ({k}) => errs[k] ? <span className="block text-[11px] text-rose-500 mt-1">{errs[k]}</span> : null;

  return (
    <div className="space-y-5">
      <Card className={cls(editId && 'ring-2 ring-amber-300')}>
        <CardHead icon="plus"
          title={editId ? 'แก้ไขรายการขาย' : 'บันทึกข้อมูลการขาย'}
          sub={editId ? 'กำลังแก้ไขรายการที่เลือก' : 'กรอกข้อมูลการขายประจำวัน'}
          right={editId && <Btn size="sm" variant="ghost" icon="x" onClick={()=>{setEditId(null);setForm(empty);setErrs({});}}>ยกเลิกแก้ไข</Btn>} />
        <form onSubmit={save} className="px-5 pb-5">
          <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
            <div><Field label="วันที่" required><Input type="date" value={form.date} onChange={e=>set('date',e.target.value)}/></Field><Err k="date"/></div>
            <div><Field label="ผู้ขาย" required>
              <Select value={form.sellerId} onChange={e=>set('sellerId',e.target.value)}>
                <option value="">-- เลือกผู้ขาย --</option>
                {sellers.map(u=><option key={u.id} value={u.id}>{u.nickname||u.fullName} ({u.position})</option>)}
              </Select></Field><Err k="sellerId"/></div>
            <div><Field label="BRAND" required>
              <Select value={form.brand} onChange={e=>set('brand',e.target.value)}>
                <option value="">-- เลือกแบรนด์ --</option>
                {brands.map(b=><option key={b.id} value={b.name}>{b.name}</option>)}
              </Select></Field><Err k="brand"/></div>
            <div><Field label="จำนวนเครื่อง" required><Input type="number" min="0" step="1" placeholder="0" value={form.units} onChange={e=>set('units',e.target.value)}/></Field><Err k="units"/></div>
            <div><Field label="ยอดขาย (บาท)" required><Input type="number" min="0" step="0.01" placeholder="0.00" value={form.amount} onChange={e=>set('amount',e.target.value)}/></Field><Err k="amount"/></div>
            <Field label="จำนวน SIM"><Input type="number" min="0" step="1" placeholder="0" value={form.sim} onChange={e=>set('sim',e.target.value)}/></Field>
            <Field label="จำนวน Film"><Input type="number" min="0" step="1" placeholder="0" value={form.film} onChange={e=>set('film',e.target.value)}/></Field>
            <Field label="PackFee (บาท)"><Input type="number" min="0" step="0.01" placeholder="0.00" value={form.packFee} onChange={e=>set('packFee',e.target.value)}/></Field>
          </div>
          <div className="flex flex-wrap gap-2 mt-5">
            <Btn type="submit" size="lg" disabled={busy} icon={busy?null:'save'}>
              {busy ? <Spinner className="w-5 h-5 text-white"/> : (editId ? 'บันทึกการแก้ไข' : 'บันทึกข้อมูล')}
            </Btn>
            <Btn type="button" variant="outline" size="lg" onClick={()=>{setForm(empty);setErrs({});setEditId(null);}}>ล้างฟอร์ม</Btn>
          </div>
        </form>
      </Card>

      <Card>
        <CardHead icon="file" title="ประวัติการขายล่าสุด" sub="แสดง 20 รายการล่าสุด"
          right={<Btn size="sm" variant="soft" icon="search" onClick={load}>รีเฟรช</Btn>}/>
        <div className="px-2 pb-4">
          {loading ? <div className="grid place-items-center py-14"><Spinner className="w-7 h-7"/></div> :
          <Table head={['วันที่','ผู้ขาย','BRAND',{label:'เครื่อง',align:'right'},{label:'ยอดขาย',align:'right'},
            {label:'SIM',align:'right'},{label:'Film',align:'right'},{label:'PackFee',align:'right'},{label:'จัดการ',align:'center'}]}>
            {history.length===0 ? <Empty cols={9}/> : history.map(r=>{
              const b = brands.find(x=>x.name===r.brand);
              return (
                <tr key={r.id} className={cls('hover:bg-primary-50/40 transition', editId===r.id && 'bg-amber-50')}>
                  <td className="px-3 py-2.5 whitespace-nowrap text-slate-600">{r.date}</td>
                  <td className="px-3 py-2.5 whitespace-nowrap font-medium">{r.sellerName}</td>
                  <td className="px-3 py-2.5 whitespace-nowrap">
                    <span className="inline-flex items-center gap-1.5">
                      <span className="w-2.5 h-2.5 rounded-full" style={{background:b?.color||'#94a3b8'}}/>{r.brand}
                    </span>
                  </td>
                  <td className="px-3 py-2.5 text-right">{nf(r.units)}</td>
                  <td className="px-3 py-2.5 text-right font-semibold text-primary-700">{nf(r.amount)}</td>
                  <td className="px-3 py-2.5 text-right">{nf(r.sim)}</td>
                  <td className="px-3 py-2.5 text-right">{nf(r.film)}</td>
                  <td className="px-3 py-2.5 text-right">{nf(r.packFee)}</td>
                  <td className="px-3 py-2.5">
                    <div className="flex gap-1.5 justify-center">
                      <Btn size="sm" variant="soft" icon="edit" onClick={()=>startEdit(r)}>แก้ไข</Btn>
                      <Btn size="sm" variant="dangerSoft" icon="trash" onClick={()=>remove(r)}>ลบ</Btn>
                    </div>
                  </td>
                </tr>
              );
            })}
          </Table>}
        </div>
      </Card>
    </div>
  );
}

/* ============================================================
   8) PAGE: DAILY DASHBOARD
   ============================================================ */
function PageDaily(){
  const { brands } = useApp(); const toast = useToast();
  const [date,setDate] = useState(todayStr());
  const [rows,setRows] = useState([]); const [loading,setLoading] = useState(true);

  useEffect(()=>{ (async()=>{
    setLoading(true);
    try{
      const s = await colSales().where('date','==',date).get();
      setRows(s.docs.map(d=>({id:d.id,...d.data()})));
    }catch(e){ toast.error('โหลดข้อมูลไม่สำเร็จ: '+e.message); }
    setLoading(false);
  })(); },[date]);

  const t = sumRows(rows);
  const byBrand = useMemo(()=>{
    const g = groupBy(rows,'brand');
    return Object.entries(g).map(([k,v])=>({brand:k, ...sumRows(v)}))
      .sort((a,b)=>b.units-a.units);
  },[rows]);

  return (
    <div className="space-y-5">
      <Card className="p-4">
        <div className="flex flex-col sm:flex-row sm:items-end gap-3">
          <div className="sm:w-60"><Field label="เลือกวันที่ต้องการดู">
            <Input type="date" value={date} onChange={e=>setDate(e.target.value)}/></Field></div>
          <div className="flex gap-2">
            <Btn variant="outline" onClick={()=>setDate(todayStr())}>วันนี้</Btn>
            <Btn variant="outline" onClick={()=>{const d=new Date(date);d.setDate(d.getDate()-1);
              setDate(`${d.getFullYear()}-${pad(d.getMonth()+1)}-${pad(d.getDate())}`);}}>ย้อนหลัง 1 วัน</Btn>
          </div>
          <div className="sm:ml-auto">
            <Btn variant="soft" icon="download" onClick={()=>exportExcel(`ยอดขายรายวัน_${date}`,
              ['วันที่','ผู้ขาย','BRAND','เครื่อง','ยอดขาย','SIM','Film','PackFee'],
              rows.map(r=>[r.date,r.sellerName,r.brand,r.units,r.amount,r.sim,r.film,r.packFee]),
              `ยอดขายประจำวันที่ ${date}`)}>Export Excel</Btn>
          </div>
        </div>
      </Card>

      {loading ? <div className="grid place-items-center py-20"><Spinner className="w-8 h-8"/></div> : <>
        <div className="grid grid-cols-2 lg:grid-cols-5 gap-3">
          <Stat icon="phone" label="จำนวนเครื่องรวม" value={nf(t.units)} unit="เครื่อง" tone="blue"/>
          <Stat icon="cash"  label="ยอดขายรวม" value={nf(t.amount)} unit="บาท" tone="green"/>
          <Stat icon="sim"   label="SIM รวม" value={nf(t.sim)} unit="ซิม" tone="violet"/>
          <Stat icon="film"  label="FILM รวม" value={nf(t.film)} unit="แผ่น" tone="amber"/>
          <Stat icon="tag"   label="PackFee รวม" value={nf(t.packFee)} unit="บาท" tone="rose"/>
        </div>

        {byBrand.length>0 && (
          <Card>
            <CardHead icon="tag" title="สรุปตามแบรนด์" sub={`วันที่ ${date}`}/>
            <div className="px-2 pb-4">
              <Table head={['BRAND',{label:'เครื่อง',align:'right'},{label:'Share %',align:'right'},
                {label:'ยอดขาย',align:'right'},{label:'SIM',align:'right'},{label:'Film',align:'right'},{label:'PackFee',align:'right'}]}>
                {byBrand.map(b=>{
                  const c = brands.find(x=>x.name===b.brand)?.color || '#94a3b8';
                  return <tr key={b.brand} className="hover:bg-primary-50/40">
                    <td className="px-3 py-2.5"><span className="inline-flex items-center gap-2">
                      <span className="w-2.5 h-2.5 rounded-full" style={{background:c}}/>{b.brand}</span></td>
                    <td className="px-3 py-2.5 text-right font-semibold">{nf(b.units)}</td>
                    <td className="px-3 py-2.5 text-right">{pctOf(b.units,t.units).toFixed(1)}%</td>
                    <td className="px-3 py-2.5 text-right text-primary-700 font-medium">{nf(b.amount)}</td>
                    <td className="px-3 py-2.5 text-right">{nf(b.sim)}</td>
                    <td className="px-3 py-2.5 text-right">{nf(b.film)}</td>
                    <td className="px-3 py-2.5 text-right">{nf(b.packFee)}</td>
                  </tr>;
                })}
              </Table>
            </div>
          </Card>
        )}

        <Card>
          <CardHead icon="file" title="รายการขายทั้งหมดของวันนี้" sub={`${rows.length} รายการ`}/>
          <div className="px-2 pb-4">
            <Table head={['ผู้ขาย','ตำแหน่ง','BRAND',{label:'เครื่อง',align:'right'},{label:'ยอดขาย',align:'right'},
              {label:'SIM',align:'right'},{label:'Film',align:'right'},{label:'PackFee',align:'right'}]}>
              {rows.length===0 ? <Empty cols={8} text="ยังไม่มีรายการขายในวันที่เลือก"/> :
                rows.map(r=>(
                  <tr key={r.id} className="hover:bg-primary-50/40">
                    <td className="px-3 py-2.5 font-medium whitespace-nowrap">{r.sellerName}</td>
                    <td className="px-3 py-2.5"><Badge tone="gray">{r.sellerPosition||'-'}</Badge></td>
                    <td className="px-3 py-2.5 whitespace-nowrap">{r.brand}</td>
                    <td className="px-3 py-2.5 text-right">{nf(r.units)}</td>
                    <td className="px-3 py-2.5 text-right text-primary-700 font-medium">{nf(r.amount)}</td>
                    <td className="px-3 py-2.5 text-right">{nf(r.sim)}</td>
                    <td className="px-3 py-2.5 text-right">{nf(r.film)}</td>
                    <td className="px-3 py-2.5 text-right">{nf(r.packFee)}</td>
                  </tr>
                ))}
            </Table>
          </div>
        </Card>
      </>}
    </div>
  );
}

/* ============================================================
   9) MONTH PICKER (shared)
   ============================================================ */
function MonthPicker({year, month, setYear, setMonth, children}){
  const years = []; const cy = new Date().getFullYear();
  for (let y=cy-3; y<=cy+1; y++) years.push(y);
  return (
    <Card className="p-4">
      <div className="flex flex-col sm:flex-row sm:items-end gap-3">
        <div className="sm:w-44"><Field label="เดือน">
          <Select value={month} onChange={e=>setMonth(+e.target.value)}>
            {MONTHS_TH.map((m,i)=><option key={m} value={i+1}>{m}</option>)}
          </Select></Field></div>
        <div className="sm:w-32"><Field label="ปี">
          <Select value={year} onChange={e=>setYear(+e.target.value)}>
            {years.map(y=><option key={y} value={y}>{y}</option>)}
          </Select></Field></div>
        {children}
      </div>
    </Card>
  );
}

/* ============================================================
   10) PAGE: MONTHLY REPORT
   ============================================================ */
function PageReport(){
  const { brands, users } = useApp(); const toast = useToast();
  const now = new Date();
  const [year,setYear]   = useState(now.getFullYear());
  const [month,setMonth] = useState(now.getMonth()+1);
  const [fBrand,setFBrand]   = useState('');
  const [fSeller,setFSeller] = useState('');
  const [view,setView] = useState('brand');
  const [rows,setRows] = useState([]); const [loading,setLoading] = useState(true);

  useEffect(()=>{ (async()=>{
    setLoading(true);
    try{ const {from,to}=monthRange(year,month); setRows(await fetchSalesRange(from,to)); }
    catch(e){ toast.error('โหลดข้อมูลไม่สำเร็จ: '+e.message); }
    setLoading(false);
  })(); },[year,month]);

  const filtered = useMemo(()=>rows.filter(r =>
    (!fBrand || r.brand===fBrand) && (!fSeller || r.sellerId===fSeller)), [rows,fBrand,fSeller]);
  const total = sumRows(filtered);

  const summary = useMemo(()=>{
    const key = view==='brand' ? 'brand' : view==='seller' ? 'sellerName' : 'date';
    const g = groupBy(filtered, key);
    return Object.entries(g).map(([k,v])=>({key:k, ...sumRows(v)}))
      .sort((a,b)=> view==='date' ? a.key.localeCompare(b.key) : b.amount-a.amount);
  },[filtered,view]);

  const headerLabel = view==='brand'?'BRAND':view==='seller'?'ผู้ขาย':'วันที่';
  const title = `รายงานยอดขาย ${MONTHS_TH[month-1]} ${year}`;
  const expHeaders = [headerLabel,'จำนวนเครื่อง','Share %','ยอดขาย (บาท)','SIM','Film','PackFee (บาท)'];
  const expRows = summary.map(s=>[s.key,s.units,pctOf(s.units,total.units).toFixed(2),s.amount,s.sim,s.film,s.packFee]);

  const chartData = useMemo(()=> summary.slice(0,12).map(s=>({name:s.key, ยอดขาย:s.amount, เครื่อง:s.units})),[summary]);

  return (
    <div className="space-y-5">
      <MonthPicker {...{year,month,setYear,setMonth}}>
        <div className="sm:w-44"><Field label="กรองตาม BRAND">
          <Select value={fBrand} onChange={e=>setFBrand(e.target.value)}>
            <option value="">ทุกแบรนด์</option>
            {brands.map(b=><option key={b.id} value={b.name}>{b.name}</option>)}
          </Select></Field></div>
        <div className="sm:w-52"><Field label="กรองตามผู้ขาย">
          <Select value={fSeller} onChange={e=>setFSeller(e.target.value)}>
            <option value="">ทุกคน</option>
            {users.map(u=><option key={u.id} value={u.id}>{u.nickname||u.fullName}</option>)}
          </Select></Field></div>
        <div className="flex gap-2 sm:ml-auto">
          <Btn variant="soft" icon="download" onClick={()=>exportExcel(title, expHeaders, expRows, title)}>Excel</Btn>
          <Btn variant="outline" icon="download" onClick={()=>exportCSV(title, expHeaders, expRows)}>CSV</Btn>
        </div>
      </MonthPicker>

      {loading ? <div className="grid place-items-center py-20"><Spinner className="w-8 h-8"/></div> : <>
        <div className="grid grid-cols-2 lg:grid-cols-5 gap-3">
          <Stat icon="phone" label="เครื่องรวมทั้งเดือน" value={nf(total.units)} unit="เครื่อง"/>
          <Stat icon="cash" label="ยอดขายรวม" value={nf(total.amount)} unit="บาท" tone="green"/>
          <Stat icon="sim" label="SIM รวม" value={nf(total.sim)} unit="ซิม" tone="violet"/>
          <Stat icon="film" label="Film รวม" value={nf(total.film)} unit="แผ่น" tone="amber"/>
          <Stat icon="tag" label="PackFee รวม" value={nf(total.packFee)} unit="บาท" tone="rose"/>
        </div>

        <Card>
          <CardHead icon="file" title="ตารางสรุป" sub={`${filtered.length} รายการ · ${title}`}
            right={
              <div className="flex bg-slate-100 rounded-xl p-1 gap-1">
                {[['brand','แบรนด์'],['seller','ผู้ขาย'],['date','รายวัน']].map(([k,l])=>(
                  <button key={k} onClick={()=>setView(k)}
                    className={cls('px-3 py-1.5 rounded-lg text-xs font-medium transition',
                      view===k?'bg-white text-primary-700 shadow-sm':'text-slate-500 hover:text-slate-700')}>{l}</button>
                ))}
              </div>}/>
          <div className="px-2 pb-4">
            <Table head={[headerLabel,{label:'เครื่อง',align:'right'},{label:'Share %',align:'right'},
              {label:'ยอดขาย (บาท)',align:'right'},{label:'SIM',align:'right'},{label:'Film',align:'right'},{label:'PackFee',align:'right'}]}>
              {summary.length===0 ? <Empty cols={7}/> : <>
                {summary.map(s=>{
                  const c = view==='brand' ? (brands.find(x=>x.name===s.key)?.color||'#94a3b8') : null;
                  return <tr key={s.key} className="hover:bg-primary-50/40">
                    <td className="px-3 py-2.5 whitespace-nowrap">
                      <span className="inline-flex items-center gap-2">
                        {c && <span className="w-2.5 h-2.5 rounded-full" style={{background:c}}/>}{s.key}</span></td>
                    <td className="px-3 py-2.5 text-right font-semibold">{nf(s.units)}</td>
                    <td className="px-3 py-2.5 text-right">
                      <div className="flex items-center justify-end gap-2">
                        <div className="w-16 h-1.5 rounded-full bg-slate-100 overflow-hidden hidden sm:block">
                          <div className="h-full bg-primary-500" style={{width:`${Math.min(100,pctOf(s.units,total.units))}%`}}/>
                        </div>{pctOf(s.units,total.units).toFixed(1)}%
                      </div></td>
                    <td className="px-3 py-2.5 text-right text-primary-700 font-medium">{nf(s.amount)}</td>
                    <td className="px-3 py-2.5 text-right">{nf(s.sim)}</td>
                    <td className="px-3 py-2.5 text-right">{nf(s.film)}</td>
                    <td className="px-3 py-2.5 text-right">{nf(s.packFee)}</td>
                  </tr>;
                })}
                <tr className="bg-primary-50 font-semibold text-primary-900">
                  <td className="px-3 py-3">รวมทั้งสิ้น</td>
                  <td className="px-3 py-3 text-right">{nf(total.units)}</td>
                  <td className="px-3 py-3 text-right">100%</td>
                  <td className="px-3 py-3 text-right">{nf(total.amount)}</td>
                  <td className="px-3 py-3 text-right">{nf(total.sim)}</td>
                  <td className="px-3 py-3 text-right">{nf(total.film)}</td>
                  <td className="px-3 py-3 text-right">{nf(total.packFee)}</td>
                </tr>
              </>}
            </Table>
          </div>
        </Card>

        <Card>
          <CardHead icon="trend" title="กราฟเปรียบเทียบ" sub={`ยอดขายตาม${headerLabel}`}/>
          <div className="px-3 pb-5 h-80">
            <ResponsiveContainer width="100%" height="100%">
              <BarChart data={chartData} margin={{top:5,right:10,left:0,bottom:5}}>
                <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" vertical={false}/>
                <XAxis dataKey="name" tick={{fontSize:11}} interval={0} angle={-20} textAnchor="end" height={60}/>
                <YAxis tick={{fontSize:11}} tickFormatter={v=>v>=1e6?(v/1e6).toFixed(1)+'M':v>=1000?(v/1000).toFixed(0)+'K':v}/>
                <Tooltip formatter={(v,n)=>[nf(v), n]} contentStyle={{borderRadius:12,border:'1px solid #dbeefe',fontSize:12}}/>
                <Bar dataKey="ยอดขาย" radius={[8,8,0,0]} fill="#2577eb"/>
              </BarChart>
            </ResponsiveContainer>
          </div>
        </Card>
      </>}
    </div>
  );
}

/* ============================================================
   11) PAGE: KPI & COMMISSION
   ============================================================ */
function PageKPI(){
  const { users, settings } = useApp(); const toast = useToast();
  const now = new Date();
  const [year,setYear]   = useState(now.getFullYear());
  const [month,setMonth] = useState(now.getMonth()+1);
  const [rows,setRows] = useState([]); const [loading,setLoading] = useState(true);

  useEffect(()=>{ (async()=>{
    setLoading(true);
    try{ const {from,to}=monthRange(year,month); setRows(await fetchSalesRange(from,to)); }
    catch(e){ toast.error('โหลดข้อมูลไม่สำเร็จ: '+e.message); }
    setLoading(false);
  })(); },[year,month]);

  const data = useMemo(()=>{
    const g = groupBy(rows,'sellerId');
    return Object.entries(g).map(([id,list])=>{
      const t = sumRows(list);
      const u = users.find(x=>x.id===id);
      const brandUnits = n => list.filter(r=>r.brand && r.brand.toUpperCase()===n).reduce((a,r)=>a+num(r.units),0);
      const samsung = brandUnits('SAMSUNG');
      const ti = brandUnits('TECNO') + brandUnits('INFINIX');
      const samsungPct = pctOf(samsung, t.units);
      const tiPct = pctOf(ti, t.units);
      const kpi1 = samsungPct >= settings.kpiSamsung;
      const kpi2 = tiPct >= settings.kpiTecnoInfinix;

      const filmPct  = pctOf(t.film, t.units);
      const filmRate = tierRate(settings.filmTiers, filmPct);
      const filmComm = filmRate * t.film;

      const simPct  = pctOf(t.sim, t.units);
      const simRate = tierRate(settings.simTiers, simPct);
      const simComm = (simRate/100) * t.packFee;

      return { id, name: u ? (u.nickname||u.fullName) : (list[0]?.sellerName||'-'),
        fullName: u?.fullName || list[0]?.sellerName || '-',
        position: u?.position || list[0]?.sellerPosition || '-', ...t,
        samsung, samsungPct, ti, tiPct, kpi1, kpi2,
        filmPct, filmRate, filmComm, simPct, simRate, simComm, total: filmComm + simComm };
    }).sort((a,b)=>b.total-a.total);
  },[rows,users,settings]);

  const grand = data.reduce((a,d)=>({ units:a.units+d.units, amount:a.amount+d.amount,
    film:a.film+d.film, sim:a.sim+d.sim, packFee:a.packFee+d.packFee,
    filmComm:a.filmComm+d.filmComm, simComm:a.simComm+d.simComm, total:a.total+d.total }),
    {units:0,amount:0,film:0,sim:0,packFee:0,filmComm:0,simComm:0,total:0});

  const title = `KPI และค่าคอมมิชชั่น ${MONTHS_TH[month-1]} ${year}`;
  const exp = () => exportExcel(title,
    ['ชื่อ-นามสกุล','ชื่อเล่น','ตำแหน่ง','เครื่องรวม','ยอดขาย','SAMSUNG','SAMSUNG %',`KPI SS (${settings.kpiSamsung}%)`,
     'TECNO+INFINIX','T+I %',`KPI T+I (${settings.kpiTecnoInfinix}%)`,'Film','Film %','เรท/แผ่น','คอม Film',
     'SIM','SIM %','PackFee','เรท SIM %','คอม SIM','คอมรวม (บาท)'],
    data.map(d=>[d.fullName,d.name,d.position,d.units,d.amount,d.samsung,d.samsungPct.toFixed(2),d.kpi1?'ผ่าน':'ไม่ผ่าน',
      d.ti,d.tiPct.toFixed(2),d.kpi2?'ผ่าน':'ไม่ผ่าน',d.film,d.filmPct.toFixed(2),d.filmRate,d.filmComm.toFixed(2),
      d.sim,d.simPct.toFixed(2),d.packFee,d.simRate,d.simComm.toFixed(2),d.total.toFixed(2)]), title);

  return (
    <div className="space-y-5">
      <MonthPicker {...{year,month,setYear,setMonth}}>
        <div className="sm:ml-auto"><Btn variant="success" icon="download" onClick={exp}>Export Excel (จ่ายเงิน)</Btn></div>
      </MonthPicker>

      <div className="grid sm:grid-cols-2 lg:grid-cols-4 gap-3">
        <Card className="p-4">
          <p className="text-xs text-slate-500">เงื่อนไข KPI ปัจจุบัน</p>
          <p className="text-sm mt-2">SAMSUNG ≥ <b className="text-primary-700">{settings.kpiSamsung}%</b></p>
          <p className="text-sm">TECNO + INFINIX ≥ <b className="text-primary-700">{settings.kpiTecnoInfinix}%</b></p>
          <p className="text-[11px] text-slate-400 mt-2">เทียบกับจำนวนเครื่องรวมของแต่ละคน</p>
        </Card>
        <Stat icon="film" label="ค่าคอม Film รวม" value={nf2(grand.filmComm)} unit="บาท" tone="amber"/>
        <Stat icon="sim" label="ค่าคอม SIM รวม" value={nf2(grand.simComm)} unit="บาท" tone="violet"/>
        <Stat icon="cash" label="ค่าคอมทั้งหมด" value={nf2(grand.total)} unit="บาท" tone="green"
          foot={`พนักงาน ${data.length} คน · ${nf(grand.units)} เครื่อง`}/>
      </div>

      <Card>
        <CardHead icon="award" title="ตารางคำนวณรายบุคคล" sub={title}/>
        <div className="px-2 pb-4">
          {loading ? <div className="grid place-items-center py-14"><Spinner className="w-7 h-7"/></div> :
          <div className="overflow-x-auto">
            <table className="w-full text-sm min-w-[1250px]">
              <thead>
                <tr className="bg-primary-50/70 text-primary-900 text-xs">
                  <th className="px-3 py-2.5 text-left font-semibold sticky left-0 bg-primary-50">พนักงาน</th>
                  <th className="px-3 py-2.5 text-right font-semibold">เครื่อง</th>
                  <th className="px-3 py-2.5 text-center font-semibold border-l border-primary-100">SAMSUNG</th>
                  <th className="px-3 py-2.5 text-center font-semibold">TECNO+INFINIX</th>
                  <th className="px-3 py-2.5 text-center font-semibold border-l border-primary-100">Film</th>
                  <th className="px-3 py-2.5 text-right font-semibold">เรท/แผ่น</th>
                  <th className="px-3 py-2.5 text-right font-semibold">คอม Film</th>
                  <th className="px-3 py-2.5 text-center font-semibold border-l border-primary-100">SIM</th>
                  <th className="px-3 py-2.5 text-right font-semibold">PackFee</th>
                  <th className="px-3 py-2.5 text-right font-semibold">คอม SIM</th>
                  <th className="px-3 py-2.5 text-right font-semibold border-l border-primary-100">รวมจ่าย</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-100">
                {data.length===0 ? <Empty cols={11}/> : data.map(d=>(
                  <tr key={d.id} className="hover:bg-primary-50/40">
                    <td className="px-3 py-3 sticky left-0 bg-white">
                      <p className="font-medium whitespace-nowrap">{d.name}</p>
                      <p className="text-[11px] text-slate-400">{d.fullName} · {d.position}</p>
                    </td>
                    <td className="px-3 py-3 text-right font-semibold">{nf(d.units)}</td>
                    <td className="px-3 py-3 text-center border-l border-slate-100">
                      <p className="font-medium">{nf(d.samsung)} <span className="text-slate-400">({d.samsungPct.toFixed(1)}%)</span></p>
                      <Badge tone={d.kpi1?'green':'red'}>{d.kpi1?'ผ่าน':'ไม่ผ่าน'}</Badge>
                    </td>
                    <td className="px-3 py-3 text-center">
                      <p className="font-medium">{nf(d.ti)} <span className="text-slate-400">({d.tiPct.toFixed(1)}%)</span></p>
                      <Badge tone={d.kpi2?'green':'red'}>{d.kpi2?'ผ่าน':'ไม่ผ่าน'}</Badge>
                    </td>
                    <td className="px-3 py-3 text-center border-l border-slate-100">
                      <p className="font-medium">{nf(d.film)}</p>
                      <span className={cls('text-xs font-medium', d.filmPct>=100?'text-emerald-600':d.filmPct>=70?'text-amber-600':'text-rose-600')}>
                        {d.filmPct.toFixed(1)}%</span>
                    </td>
                    <td className="px-3 py-3 text-right">{d.filmRate ? d.filmRate+' ฿' : <span className="text-rose-500">ไม่จ่าย</span>}</td>
                    <td className="px-3 py-3 text-right font-medium text-amber-700">{nf2(d.filmComm)}</td>
                    <td className="px-3 py-3 text-center border-l border-slate-100">
                      <p className="font-medium">{nf(d.sim)}</p>
                      <span className={cls('text-xs font-medium', d.simPct>=20?'text-emerald-600':d.simPct>=14?'text-amber-600':'text-rose-600')}>
                        {d.simPct.toFixed(1)}%</span>
                    </td>
                    <td className="px-3 py-3 text-right">{nf(d.packFee)}</td>
                    <td className="px-3 py-3 text-right font-medium text-indigo-700">
                      {d.simRate ? <>{nf2(d.simComm)} <span className="text-[11px] text-slate-400">({d.simRate}%)</span></> : <span className="text-rose-500">ไม่จ่าย</span>}
                    </td>
                    <td className="px-3 py-3 text-right font-bold text-emerald-700 border-l border-slate-100">{nf2(d.total)}</td>
                  </tr>
                ))}
                {data.length>0 && (
                  <tr className="bg-primary-50 font-semibold text-primary-900">
                    <td className="px-3 py-3 sticky left-0 bg-primary-50">รวม</td>
                    <td className="px-3 py-3 text-right">{nf(grand.units)}</td>
                    <td/><td/>
                    <td className="px-3 py-3 text-center">{nf(grand.film)}</td><td/>
                    <td className="px-3 py-3 text-right">{nf2(grand.filmComm)}</td>
                    <td className="px-3 py-3 text-center">{nf(grand.sim)}</td>
                    <td className="px-3 py-3 text-right">{nf(grand.packFee)}</td>
                    <td className="px-3 py-3 text-right">{nf2(grand.simComm)}</td>
                    <td className="px-3 py-3 text-right text-emerald-700">{nf2(grand.total)}</td>
                  </tr>
                )}
              </tbody>
            </table>
          </div>}
        </div>
      </Card>

      <div className="grid md:grid-cols-2 gap-4">
        <Card>
          <CardHead icon="film" title="เงื่อนไขค่าคอม Film" sub="ยอด Film เทียบยอดเครื่อง"/>
          <div className="px-5 pb-5 space-y-2">
            <div className="flex justify-between text-sm py-2 border-b border-slate-100">
              <span className="text-slate-500">≤ 69.99%</span><span className="text-rose-600 font-medium">ไม่จ่าย</span></div>
            {[...settings.filmTiers].sort((a,b)=>a.min-b.min).map(t=>(
              <div key={t.min} className="flex justify-between text-sm py-2 border-b border-slate-100 last:border-0">
                <span className="text-slate-600">≥ {t.min.toFixed(2)}%</span>
                <span className="font-semibold text-primary-700">{t.rate} บาท/แผ่น</span></div>
            ))}
          </div>
        </Card>
        <Card>
          <CardHead icon="sim" title="เงื่อนไขค่าคอม SIM" sub="จำนวน SIM เทียบยอดเครื่อง (จ่ายเป็น % ของ PackFee)"/>
          <div className="px-5 pb-5 space-y-2">
            <div className="flex justify-between text-sm py-2 border-b border-slate-100">
              <span className="text-slate-500">0 - 13.99%</span><span className="text-rose-600 font-medium">ไม่จ่าย</span></div>
            {[...settings.simTiers].sort((a,b)=>a.min-b.min).map(t=>(
              <div key={t.min} className="flex justify-between text-sm py-2 border-b border-slate-100 last:border-0">
                <span className="text-slate-600">≥ {t.min.toFixed(2)}%</span>
                <span className="font-semibold text-primary-700">{t.rate}% ของ PackFee</span></div>
            ))}
          </div>
        </Card>
      </div>
    </div>
  );
}

/* ============================================================
   12) PAGE: MARKET SHARE
   ============================================================ */
function Collapsible({title, color, subtitle, defaultOpen=false, children}){
  const [open,setOpen] = useState(defaultOpen);
  return (
    <Card className="overflow-hidden">
      <button onClick={()=>setOpen(o=>!o)} className="w-full flex items-center justify-between gap-3 px-5 py-4 hover:bg-slate-50 transition text-left">
        <div className="flex items-center gap-3 min-w-0">
          <span className="w-9 h-9 rounded-xl grid place-items-center text-white shrink-0" style={{background:color}}>
            <Icon n="store" className="w-5 h-5"/></span>
          <div className="min-w-0">
            <p className="font-semibold text-slate-800 truncate">{title}</p>
            {subtitle && <p className="text-xs text-slate-500 truncate">{subtitle}</p>}
          </div>
        </div>
        <Icon n="chev" className={cls('w-5 h-5 text-slate-400 transition-transform shrink-0', open && 'rotate-180')}/>
      </button>
      {open && <div className="px-5 pb-5 border-t border-slate-100 pt-4 fade-in">{children}</div>}
    </Card>
  );
}

function PageMarketShare(){
  const { brands, isAdmin } = useApp(); const toast = useToast();
  const now = new Date();
  const [year,setYear]   = useState(now.getFullYear());
  const [month,setMonth] = useState(now.getMonth()+1);
  const [manual,setManual] = useState({});           // {STORE:{BRAND:qty}}
  const [itcity,setItcity] = useState({});           // {BRAND:qty}
  const [loading,setLoading] = useState(true);
  const [saving,setSaving]   = useState(false);
  const [brandPick,setBrandPick] = useState('');

  const mkey = `${year}-${pad(month)}`;

  useEffect(()=>{ (async()=>{
    setLoading(true);
    try{
      const {from,to} = monthRange(year,month);
      const sales = await fetchSalesRange(from,to);
      const it = {}; sales.forEach(r=>{ it[r.brand] = (it[r.brand]||0) + num(r.units); });
      setItcity(it);
      const doc = await colMS().doc(mkey).get();
      setManual(doc.exists ? (doc.data().stores||{}) : {});
    }catch(e){ toast.error('โหลดข้อมูลไม่สำเร็จ: '+e.message); }
    setLoading(false);
  })(); },[mkey]);

  useEffect(()=>{ if (brands.length && !brandPick) setBrandPick(brands[0].name); },[brands]);

  const setCell = (store,brand,v) =>
    setManual(m=>({...m, [store]:{...(m[store]||{}), [brand]: num(v)}}));

  const save = async () => {
    setSaving(true);
    try{
      await colMS().doc(mkey).set({ year, month, stores:manual,
        updatedAt: firebase.firestore.FieldValue.serverTimestamp() }, {merge:true});
      toast.success(`บันทึกข้อมูล Market Share เดือน ${MONTHS_TH[month-1]} ${year} เรียบร้อย`);
    }catch(e){ toast.error('บันทึกไม่สำเร็จ: '+e.message); }
    setSaving(false);
  };

  const storeData = useMemo(()=>STORES.map(s=>{
    const data = s.auto ? itcity : (manual[s.key]||{});
    const total = brands.reduce((a,b)=>a+num(data[b.name]),0);
    return {...s, data, total};
  }),[itcity,manual,brands]);

  const grandTotal = storeData.reduce((a,s)=>a+s.total,0);
  const itcityStore = storeData[0];

  const donut = storeData.filter(s=>s.total>0).map(s=>({name:s.label, value:s.total, color:s.color}));

  const brandAcrossStores = useMemo(()=>{
    if (!brandPick) return [];
    const tot = storeData.reduce((a,s)=>a+num(s.data[brandPick]),0);
    return storeData.map(s=>({ name:s.label, color:s.color,
      value:num(s.data[brandPick]), pct: pctOf(num(s.data[brandPick]), tot) }));
  },[storeData, brandPick]);

  const brandTotalPick = brandAcrossStores.reduce((a,b)=>a+b.value,0);

  return (
    <div className="space-y-5">
      <MonthPicker {...{year,month,setYear,setMonth}}>
        <div className="sm:ml-auto flex gap-2">
          <Btn icon="save" onClick={save} disabled={saving || !isAdmin}>
            {saving ? <Spinner className="w-4 h-4 text-white"/> : 'บันทึกข้อมูล'}</Btn>
          <Btn variant="soft" icon="download" onClick={()=>exportExcel(`MarketShare_${mkey}`,
            ['BRAND',...STORES.map(s=>s.label),'รวม','ITCITY Share %'],
            brands.map(b=>{
              const vals = storeData.map(s=>num(s.data[b.name]));
              const tot = vals.reduce((a,v)=>a+v,0);
              return [b.name,...vals,tot,pctOf(vals[0],tot).toFixed(2)];
            }), `Market Share ${MONTHS_TH[month-1]} ${year}`)}>Export</Btn>
        </div>
      </MonthPicker>

      {loading ? <div className="grid place-items-center py-20"><Spinner className="w-8 h-8"/></div> : <>
        {/* Overall cards */}
        <div className="grid grid-cols-2 lg:grid-cols-4 gap-3">
          {storeData.map(s=>(
            <Card key={s.key} className="p-4">
              <div className="flex items-center gap-3">
                <span className="w-10 h-10 rounded-xl grid place-items-center text-white shrink-0" style={{background:s.color}}>
                  <Icon n="store"/></span>
                <div className="min-w-0">
                  <p className="text-[11px] text-slate-500 truncate">{s.label}{s.auto && ' (อัตโนมัติ)'}</p>
                  <p className="text-xl font-bold leading-tight">{nf(s.total)} <span className="text-xs text-slate-400">เครื่อง</span></p>
                </div>
              </div>
              <div className="mt-3">
                <div className="h-1.5 rounded-full bg-slate-100 overflow-hidden">
                  <div className="h-full rounded-full" style={{width:`${Math.min(100,pctOf(s.total,grandTotal))}%`,background:s.color}}/>
                </div>
                <p className="text-[11px] text-slate-500 mt-1">{pctOf(s.total,grandTotal).toFixed(1)}% ของทั้งตึก</p>
              </div>
            </Card>
          ))}
        </div>

        {/* Input cards */}
        <div className="grid lg:grid-cols-3 gap-4">
          {STORES.filter(s=>!s.auto).map(s=>(
            <Collapsible key={s.key} title={s.label} color={s.color}
              subtitle={`รวม ${nf(storeData.find(x=>x.key===s.key).total)} เครื่อง · กดเพื่อกรอกข้อมูล`}>
              <div className="space-y-2 max-h-80 overflow-y-auto pr-1">
                {brands.map(b=>(
                  <div key={b.id} className="flex items-center gap-2">
                    <span className="w-2.5 h-2.5 rounded-full shrink-0" style={{background:b.color}}/>
                    <span className="text-sm flex-1 truncate">{b.name}</span>
                    <Input type="number" min="0" className="w-24 py-1.5 text-right" placeholder="0"
                      value={(manual[s.key]||{})[b.name] ?? ''} disabled={!isAdmin}
                      onChange={e=>setCell(s.key,b.name,e.target.value)}/>
                  </div>
                ))}
              </div>
            </Collapsible>
          ))}
        </div>
        {!isAdmin && <p className="text-xs text-amber-600 flex items-center gap-1.5"><Icon n="alert" className="w-4 h-4"/> เฉพาะ Admin เท่านั้นที่แก้ไข/บันทึกข้อมูลร้านคู่แข่งได้</p>}

        {/* Charts */}
        <div className="grid lg:grid-cols-2 gap-4">
          <Card>
            <CardHead icon="pie" title="Market Share ทั้งตึก" sub={`รวม ${nf(grandTotal)} เครื่อง`}/>
            <div className="px-3 pb-5 h-80">
              {grandTotal===0 ? <div className="h-full grid place-items-center text-slate-400 text-sm">ยังไม่มีข้อมูล</div> :
              <ResponsiveContainer width="100%" height="100%">
                <PieChart>
                  <Pie data={donut} dataKey="value" nameKey="name" innerRadius="55%" outerRadius="82%"
                       paddingAngle={3} stroke="#fff" strokeWidth={2}
                       label={({name,percent})=>`${name} ${(percent*100).toFixed(1)}%`}
                       labelLine={{stroke:'#cbd5e1'}}>
                    {donut.map((d,i)=><Cell key={i} fill={d.color}/>)}
                  </Pie>
                  <Tooltip formatter={(v,n)=>[`${nf(v)} เครื่อง (${pctOf(v,grandTotal).toFixed(1)}%)`, n]}
                    contentStyle={{borderRadius:12,border:'1px solid #dbeefe',fontSize:12}}/>
                  <Legend verticalAlign="bottom" iconType="circle" wrapperStyle={{fontSize:12}}/>
                </PieChart>
              </ResponsiveContainer>}
            </div>
          </Card>

          <Card>
            <CardHead icon="tag" title="สัดส่วนรายแบรนด์ใน 4 ร้าน"
              sub={brandPick ? `${brandPick} · รวม ${nf(brandTotalPick)} เครื่อง` : ''}
              right={<Select value={brandPick} onChange={e=>setBrandPick(e.target.value)} className="w-36 py-1.5 text-xs">
                {brands.map(b=><option key={b.id} value={b.name}>{b.name}</option>)}
              </Select>}/>
            <div className="px-3 pb-5 h-80">
              {brandTotalPick===0 ? <div className="h-full grid place-items-center text-slate-400 text-sm">ยังไม่มีข้อมูลแบรนด์นี้</div> :
              <ResponsiveContainer width="100%" height="100%">
                <BarChart data={brandAcrossStores} margin={{top:10,right:10,left:0,bottom:5}}>
                  <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" vertical={false}/>
                  <XAxis dataKey="name" tick={{fontSize:12}}/>
                  <YAxis tick={{fontSize:11}} unit="%"/>
                  <Tooltip formatter={(v,n,p)=>[`${v.toFixed(1)}% (${nf(p.payload.value)} เครื่อง)`,'สัดส่วน']}
                    contentStyle={{borderRadius:12,border:'1px solid #dbeefe',fontSize:12}}/>
                  <Bar dataKey="pct" radius={[8,8,0,0]}>
                    {brandAcrossStores.map((d,i)=><Cell key={i} fill={d.color}/>)}
                  </Bar>
                </BarChart>
              </ResponsiveContainer>}
            </div>
          </Card>
        </div>

        {/* Table 1: ITCITY share */}
        <Card>
          <CardHead icon="target" title="สัดส่วน Market Share ของ ITCITY" sub="เทียบกับยอดรวมทั้งตึกในแต่ละแบรนด์"/>
          <div className="px-2 pb-4">
            <Table head={['BRAND',{label:'ITCITY',align:'right'},{label:'รวมทั้งตึก',align:'right'},{label:'Share ของเรา',align:'right'}]}>
              {brands.map(b=>{
                const mine = num(itcityStore.data[b.name]);
                const tot  = storeData.reduce((a,s)=>a+num(s.data[b.name]),0);
                const p = pctOf(mine,tot);
                return (
                  <tr key={b.id} className="hover:bg-primary-50/40">
                    <td className="px-3 py-2.5"><span className="inline-flex items-center gap-2">
                      <span className="w-2.5 h-2.5 rounded-full" style={{background:b.color}}/>{b.name}</span></td>
                    <td className="px-3 py-2.5 text-right font-semibold">{nf(mine)}</td>
                    <td className="px-3 py-2.5 text-right text-slate-500">{nf(tot)}</td>
                    <td className="px-3 py-2.5 text-right">
                      <div className="flex items-center justify-end gap-2">
                        <div className="w-20 h-1.5 rounded-full bg-slate-100 overflow-hidden hidden sm:block">
                          <div className="h-full bg-primary-500" style={{width:`${Math.min(100,p)}%`}}/></div>
                        <span className={cls('font-medium', p>=40?'text-emerald-600':p>=25?'text-amber-600':'text-rose-600')}>
                          {tot===0?'-':p.toFixed(1)+'%'}</span>
                      </div></td>
                  </tr>
                );
              })}
              <tr className="bg-primary-50 font-semibold text-primary-900">
                <td className="px-3 py-3">รวมทั้งหมด</td>
                <td className="px-3 py-3 text-right">{nf(itcityStore.total)}</td>
                <td className="px-3 py-3 text-right">{nf(grandTotal)}</td>
                <td className="px-3 py-3 text-right">{pctOf(itcityStore.total,grandTotal).toFixed(1)}%</td>
              </tr>
            </Table>
          </div>
        </Card>

        {/* Table 2: brand x store matrix */}
        <Card>
          <CardHead icon="store" title="เปอร์เซ็นต์ Market Share แต่ละแบรนด์ใน 4 ร้าน" sub="แสดงจำนวนเครื่องและ % ภายในแบรนด์นั้น"/>
          <div className="px-2 pb-4">
            <Table head={['BRAND', ...STORES.map(s=>({label:s.label, align:'center'})), {label:'รวม',align:'right'}]}>
              {brands.map(b=>{
                const vals = storeData.map(s=>num(s.data[b.name]));
                const tot = vals.reduce((a,v)=>a+v,0);
                return (
                  <tr key={b.id} className="hover:bg-primary-50/40">
                    <td className="px-3 py-2.5 whitespace-nowrap"><span className="inline-flex items-center gap-2">
                      <span className="w-2.5 h-2.5 rounded-full" style={{background:b.color}}/>{b.name}</span></td>
                    {vals.map((v,i)=>(
                      <td key={i} className="px-3 py-2.5 text-center">
                        <p className="font-medium">{nf(v)}</p>
                        <p className="text-[11px]" style={{color:STORES[i].color}}>{tot?pctOf(v,tot).toFixed(1)+'%':'-'}</p>
                      </td>
                    ))}
                    <td className="px-3 py-2.5 text-right font-semibold">{nf(tot)}</td>
                  </tr>
                );
              })}
              <tr className="bg-primary-50 font-semibold text-primary-900">
                <td className="px-3 py-3">รวม</td>
                {storeData.map(s=>(
                  <td key={s.key} className="px-3 py-3 text-center">
                    <p>{nf(s.total)}</p><p className="text-[11px]">{pctOf(s.total,grandTotal).toFixed(1)}%</p></td>
                ))}
                <td className="px-3 py-3 text-right">{nf(grandTotal)}</td>
              </tr>
            </Table>
          </div>
        </Card>
      </>}
    </div>
  );
}

/* ============================================================
   13) PAGE: OVERVIEW DASHBOARD
   ============================================================ */
function PageDashboard(){
  const { brands, settings } = useApp(); const toast = useToast();
  const now = new Date();
  const year = now.getFullYear(), month = now.getMonth()+1;
  const today = todayStr();
  const [monthRows,setMonthRows] = useState([]);
  const [target,setTarget] = useState(null);
  const [loading,setLoading] = useState(true);

  useEffect(()=>{ (async()=>{
    setLoading(true);
    try{
      const {from,to} = monthRange(year,month);
      const [rows, tdoc] = await Promise.all([
        fetchSalesRange(from,to),
        colTargets().doc(`${year}-${pad(month)}`).get()
      ]);
      setMonthRows(rows);
      setTarget(tdoc.exists ? tdoc.data() : null);
    }catch(e){ toast.error('โหลดข้อมูลไม่สำเร็จ: '+e.message); }
    setLoading(false);
  })(); },[]);

  const todayRows = useMemo(()=>monthRows.filter(r=>r.date===today),[monthRows,today]);
  const tToday = sumRows(todayRows);
  const tMonth = sumRows(monthRows);

  const brandToday = useMemo(()=>{
    const g = groupBy(todayRows,'brand');
    return Object.entries(g).map(([k,v])=>({brand:k, ...sumRows(v),
      color: brands.find(b=>b.name===k)?.color || '#94a3b8'}))
      .sort((a,b)=>b.units-a.units);
  },[todayRows,brands]);

  const topSales = useMemo(()=>{
    const g = groupBy(monthRows,'sellerId');
    return Object.entries(g).map(([id,v])=>({id, name:v[0].sellerName, position:v[0].sellerPosition, ...sumRows(v)}))
      .sort((a,b)=>b.units-a.units).slice(0,10);
  },[monthRows]);

  /* target progress */
  const targetAmount = num(target?.amount);
  const targetUnits  = num(target?.units);
  const dim = daysInMonth(year,month);
  const daysLeft = Math.max(1, dim - now.getDate() + 1);
  const amtPct = pctOf(tMonth.amount, targetAmount);
  const unitPct = pctOf(tMonth.units, targetUnits);
  const needAmtPerDay  = Math.max(0, targetAmount - tMonth.amount)/daysLeft;
  const needUnitPerDay = Math.ceil(Math.max(0, targetUnits - tMonth.units)/daysLeft);

  /* ratios (today) */
  const simR  = ratioInfo(tToday.units, tToday.sim, settings.simRatioTarget, settings.simRatioTarget);
  const filmR = ratioInfo(tToday.units, tToday.film, 1.2, 1.0);

  /* trends */
  const dailyTrend = useMemo(()=>{
    const g = groupBy(monthRows,'date');
    const out = [];
    for (let d=1; d<=dim; d++){
      const key = `${year}-${pad(month)}-${pad(d)}`;
      const s = g[key] ? sumRows(g[key]) : blankAgg();
      if (d <= now.getDate()) out.push({ วัน:String(d), ยอดขาย:s.amount, เครื่อง:s.units });
    }
    return out;
  },[monthRows,dim]);

  const stackedTrend = useMemo(()=>{
    const g = groupBy(monthRows,'date');
    const out = [];
    for (let d=1; d<=now.getDate(); d++){
      const key = `${year}-${pad(month)}-${pad(d)}`;
      const row = { วัน:String(d) };
      brands.forEach(b=>{ row[b.name] = (g[key]||[]).filter(r=>r.brand===b.name).reduce((a,r)=>a+num(r.units),0); });
      out.push(row);
    }
    return out;
  },[monthRows,brands]);

  const activeBrands = brands.filter(b => stackedTrend.some(r => r[b.name] > 0));

  if (loading) return <div className="grid place-items-center py-24"><Spinner className="w-9 h-9"/></div>;

  return (
    <div className="space-y-5">
      {/* Today */}
      <div>
        <div className="flex items-center gap-2 mb-3">
          <Icon n="calendar" className="w-4 h-4 text-primary-600"/>
          <h2 className="font-semibold text-slate-700">ยอดขายวันนี้ ({today})</h2>
        </div>
        <div className="grid grid-cols-2 lg:grid-cols-5 gap-3">
          <Stat icon="phone" label="จำนวนเครื่อง" value={nf(tToday.units)} unit="เครื่อง"/>
          <Stat icon="cash" label="ยอดขาย" value={nf(tToday.amount)} unit="บาท" tone="green"/>
          <Stat icon="sim" label="SIM" value={nf(tToday.sim)} unit="ซิม" tone="violet"
            foot={<span>สัดส่วน <b className={toneText(simR.tone)}>{simR.text}</b> (เป้า 1:{settings.simRatioTarget})</span>}/>
          <Stat icon="film" label="Film" value={nf(tToday.film)} unit="แผ่น" tone="amber"
            foot={<span>สัดส่วน <b className={toneText(filmR.tone)}>{filmR.text}</b> (เป้า 1:1)</span>}/>
          <Stat icon="tag" label="PackFee" value={nf(tToday.packFee)} unit="บาท" tone="rose"/>
        </div>
      </div>

      {/* Target + brand share today */}
      <div className="grid lg:grid-cols-3 gap-4">
        <Card className="lg:col-span-2">
          <CardHead icon="target" title={`เป้าหมายเดือน ${MONTHS_TH[month-1]} ${year}`}
            sub={`เหลืออีก ${daysLeft} วันทำการ จาก ${dim} วัน`}/>
          <div className="px-5 pb-5 space-y-5">
            {!target && <div className="flex items-center gap-2 text-sm text-amber-700 bg-amber-50 border border-amber-200 rounded-xl px-4 py-3">
              <Icon n="alert" className="w-4 h-4"/> ยังไม่ได้ตั้งเป้าหมายของเดือนนี้ (ตั้งค่าได้ที่หน้า "ตั้งค่าและจัดการข้อมูล")</div>}

            <div>
              <div className="flex justify-between text-sm mb-2">
                <span className="text-slate-600">ยอดขาย (บาท)</span>
                <span><b className="text-primary-700">{nf(tMonth.amount)}</b>
                  <span className="text-slate-400"> / {nf(targetAmount)}</span></span>
              </div>
              <div className="h-3 rounded-full bg-slate-100 overflow-hidden">
                <div className={cls('h-full rounded-full transition-all', amtPct>=100?'bg-emerald-500':'bg-gradient-to-r from-primary-400 to-primary-600')}
                     style={{width:`${Math.min(100,amtPct)}%`}}/>
              </div>
              <div className="flex justify-between text-xs mt-1.5">
                <span className={amtPct>=100?'text-emerald-600 font-medium':'text-slate-500'}>{amtPct.toFixed(1)}%</span>
                <span className="text-slate-500">{amtPct>=100 ? 'บรรลุเป้าหมายแล้ว'
                  : `ต้องทำอีก ${nf(Math.round(targetAmount-tMonth.amount))} บาท · เฉลี่ย ${nf(Math.round(needAmtPerDay))} บาท/วัน`}</span>
              </div>
            </div>

            <div>
              <div className="flex justify-between text-sm mb-2">
                <span className="text-slate-600">จำนวนเครื่อง</span>
                <span><b className="text-primary-700">{nf(tMonth.units)}</b>
                  <span className="text-slate-400"> / {nf(targetUnits)}</span></span>
              </div>
              <div className="h-3 rounded-full bg-slate-100 overflow-hidden">
                <div className={cls('h-full rounded-full transition-all', unitPct>=100?'bg-emerald-500':'bg-gradient-to-r from-indigo-400 to-violet-600')}
                     style={{width:`${Math.min(100,unitPct)}%`}}/>
              </div>
              <div className="flex justify-between text-xs mt-1.5">
                <span className={unitPct>=100?'text-emerald-600 font-medium':'text-slate-500'}>{unitPct.toFixed(1)}%</span>
                <span className="text-slate-500">{unitPct>=100 ? 'บรรลุเป้าหมายแล้ว'
                  : `ต้องทำอีก ${nf(Math.max(0,targetUnits-tMonth.units))} เครื่อง · เฉลี่ย ${nf(needUnitPerDay)} เครื่อง/วัน`}</span>
              </div>
            </div>

            <div className="grid grid-cols-2 sm:grid-cols-4 gap-3 pt-2">
              {[['ยอดสะสมเดือนนี้', nf(tMonth.amount)+' ฿'],['เครื่องสะสม', nf(tMonth.units)],
                ['SIM สะสม', nf(tMonth.sim)],['Film สะสม', nf(tMonth.film)]].map(([l,v])=>(
                <div key={l} className="rounded-xl bg-slate-50 px-3 py-2.5">
                  <p className="text-[11px] text-slate-500">{l}</p>
                  <p className="font-semibold text-slate-800 text-sm">{v}</p>
                </div>
              ))}
            </div>
          </div>
        </Card>

        <Card>
          <CardHead icon="pie" title="% แชร์แบรนด์วันนี้" sub={`${nf(tToday.units)} เครื่อง`}/>
          <div className="px-3 pb-5 h-64">
            {brandToday.length===0 ? <div className="h-full grid place-items-center text-slate-400 text-sm">ยังไม่มีการขายวันนี้</div> :
            <ResponsiveContainer width="100%" height="100%">
              <PieChart>
                <Pie data={brandToday} dataKey="units" nameKey="brand" innerRadius="52%" outerRadius="80%"
                     paddingAngle={2} stroke="#fff" strokeWidth={2}>
                  {brandToday.map((b,i)=><Cell key={i} fill={b.color}/>)}
                </Pie>
                <Tooltip formatter={(v,n)=>[`${nf(v)} เครื่อง (${pctOf(v,tToday.units).toFixed(1)}%)`,n]}
                  contentStyle={{borderRadius:12,border:'1px solid #dbeefe',fontSize:12}}/>
                <Legend verticalAlign="bottom" iconType="circle" wrapperStyle={{fontSize:11}}/>
              </PieChart>
            </ResponsiveContainer>}
          </div>
        </Card>
      </div>

      {/* Brand table today */}
      <Card>
        <CardHead icon="tag" title="ยอดขายแยกตาม BRAND (วันนี้)" sub={today}/>
        <div className="px-2 pb-4">
          <Table head={['BRAND',{label:'Share %',align:'right'},{label:'จำนวน',align:'right'},
            {label:'ยอดรวม',align:'right'},{label:'Film',align:'right'},{label:'SIM',align:'right'},{label:'PackFee',align:'right'}]}>
            {brandToday.length===0 ? <Empty cols={7} text="ยังไม่มีการขายวันนี้"/> : <>
              {brandToday.map(b=>(
                <tr key={b.brand} className="hover:bg-primary-50/40">
                  <td className="px-3 py-2.5"><span className="inline-flex items-center gap-2 whitespace-nowrap">
                    <span className="w-2.5 h-2.5 rounded-full" style={{background:b.color}}/>{b.brand}</span></td>
                  <td className="px-3 py-2.5 text-right font-medium">{pctOf(b.units,tToday.units).toFixed(1)}%</td>
                  <td className="px-3 py-2.5 text-right">{nf(b.units)}</td>
                  <td className="px-3 py-2.5 text-right text-primary-700 font-medium">{nf(b.amount)}</td>
                  <td className="px-3 py-2.5 text-right">{nf(b.film)}</td>
                  <td className="px-3 py-2.5 text-right">{nf(b.sim)}</td>
                  <td className="px-3 py-2.5 text-right">{nf(b.packFee)}</td>
                </tr>
              ))}
              <tr className="bg-primary-50 font-semibold text-primary-900">
                <td className="px-3 py-3">รวม</td><td className="px-3 py-3 text-right">100%</td>
                <td className="px-3 py-3 text-right">{nf(tToday.units)}</td>
                <td className="px-3 py-3 text-right">{nf(tToday.amount)}</td>
                <td className="px-3 py-3 text-right">{nf(tToday.film)}</td>
                <td className="px-3 py-3 text-right">{nf(tToday.sim)}</td>
                <td className="px-3 py-3 text-right">{nf(tToday.packFee)}</td>
              </tr>
            </>}
          </Table>
        </div>
      </Card>

      {/* Top sales */}
      <Card>
        <CardHead icon="award" title="TOP SALES" sub={`อันดับผู้ขายประจำเดือน ${MONTHS_TH[month-1]} ${year}`}/>
        <div className="px-2 pb-4">
          <Table head={[{label:'#',align:'center'},'ผู้ขาย',{label:'จำนวน',align:'right'},{label:'ยอดขาย',align:'right'},
            {label:'PackFee',align:'right'},'สถานะ Film','สถานะ SIM']}>
            {topSales.length===0 ? <Empty cols={7}/> : topSales.map((s,i)=>(
              <tr key={s.id} className="hover:bg-primary-50/40">
                <td className="px-3 py-3 text-center">
                  <span className={cls('inline-grid place-items-center w-7 h-7 rounded-lg text-xs font-bold',
                    i===0?'bg-amber-100 text-amber-700':i===1?'bg-slate-200 text-slate-700':i===2?'bg-orange-100 text-orange-700':'bg-slate-100 text-slate-500')}>
                    {i+1}</span></td>
                <td className="px-3 py-3 whitespace-nowrap">
                  <p className="font-medium">{s.name}</p>
                  <p className="text-[11px] text-slate-400">{s.position||'-'}</p></td>
                <td className="px-3 py-3 text-right font-semibold">{nf(s.units)}</td>
                <td className="px-3 py-3 text-right text-primary-700 font-medium">{nf(s.amount)}</td>
                <td className="px-3 py-3 text-right">{nf(s.packFee)}</td>
                <td className="px-3 py-3 text-xs whitespace-nowrap">
                  <span className={pctOf(s.film,s.units)>=settings.filmTargetPct?'text-emerald-600':'text-rose-600'}>
                    {filmSummaryText(s.film,s.units,settings.filmTargetPct)}</span></td>
                <td className="px-3 py-3 text-xs whitespace-nowrap">
                  <span className={s.sim && (s.units/s.sim)<=settings.simRatioTarget?'text-emerald-600':'text-rose-600'}>
                    {simSummaryText(s.sim,s.units,settings.simRatioTarget)}</span></td>
              </tr>
            ))}
          </Table>
        </div>
      </Card>

      {/* Trends */}
      <div className="grid xl:grid-cols-2 gap-4">
        <Card>
          <CardHead icon="trend" title="แนวโน้มยอดขายรายวัน" sub={`${MONTHS_TH[month-1]} ${year}`}/>
          <div className="px-3 pb-5 h-72">
            <ResponsiveContainer width="100%" height="100%">
              <LineChart data={dailyTrend} margin={{top:5,right:10,left:0,bottom:5}}>
                <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" vertical={false}/>
                <XAxis dataKey="วัน" tick={{fontSize:11}}/>
                <YAxis tick={{fontSize:11}} tickFormatter={v=>v>=1e6?(v/1e6).toFixed(1)+'M':v>=1000?(v/1000).toFixed(0)+'K':v}/>
                <Tooltip formatter={v=>nf(v)+' บาท'} labelFormatter={l=>`วันที่ ${l}`}
                  contentStyle={{borderRadius:12,border:'1px solid #dbeefe',fontSize:12}}/>
                <Line type="monotone" dataKey="ยอดขาย" stroke="#2577eb" strokeWidth={2.5}
                      dot={{r:3,fill:'#2577eb'}} activeDot={{r:5}}/>
              </LineChart>
            </ResponsiveContainer>
          </div>
        </Card>

        <Card>
          <CardHead icon="tag" title="จำนวนเครื่องรายวันแยกตามแบรนด์" sub="Stacked Bar Chart"/>
          <div className="px-3 pb-5 h-72">
            <ResponsiveContainer width="100%" height="100%">
              <BarChart data={stackedTrend} margin={{top:5,right:10,left:0,bottom:5}}>
                <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" vertical={false}/>
                <XAxis dataKey="วัน" tick={{fontSize:11}}/>
                <YAxis tick={{fontSize:11}}/>
                <Tooltip contentStyle={{borderRadius:12,border:'1px solid #dbeefe',fontSize:12}}
                  labelFormatter={l=>`วันที่ ${l}`}/>
                <Legend wrapperStyle={{fontSize:11}} iconType="circle"/>
                {activeBrands.map((b,i)=>(
                  <Bar key={b.id} dataKey={b.name} stackId="a" fill={b.color}
                       radius={i===activeBrands.length-1?[6,6,0,0]:0}/>
                ))}
              </BarChart>
            </ResponsiveContainer>
          </div>
        </Card>
      </div>
    </div>
  );
}

/* ============================================================
   14) PAGE: SETTINGS (ADMIN)
   ============================================================ */
function UsersTab(){
  const { users, brands, reloadUsers, profile } = useApp();
  const toast = useToast(); const confirm = useConfirm();
  const blank = { fullName:'', nickname:'', position:'PC', brand:'', role:'user', active:true, photo:'', email:'', password:'' };
  const [open,setOpen] = useState(false);
  const [form,setForm] = useState(blank);
  const [editId,setEditId] = useState(null);
  const [busy,setBusy] = useState(false);
  const [q,setQ] = useState('');
  const fileRef = useRef();

  const set = (k,v)=>setForm(f=>({...f,[k]:v}));

  const pick = async e => {
    const f = e.target.files[0]; if (!f) return;
    try { set('photo', await resizeImage(f)); toast.success('อัปโหลดรูปโปรไฟล์เรียบร้อย'); }
    catch(ex){ toast.error(ex.message); }
    e.target.value = '';
  };

  const openNew  = ()=>{ setForm(blank); setEditId(null); setOpen(true); };
  const openEdit = u =>{ setForm({...blank, ...u, password:''}); setEditId(u.id); setOpen(true); };

  const save = async e => {
    e.preventDefault();
    if (!form.fullName.trim()) return toast.error('กรุณากรอกชื่อ-นามสกุล');
    if (!form.nickname.trim()) return toast.error('กรุณากรอกชื่อเล่น');
    if (!editId && (!form.email.trim() || form.password.length < 6))
      return toast.error('กรุณากรอกอีเมล และรหัสผ่านอย่างน้อย 6 ตัวอักษร');
    setBusy(true);
    const data = { fullName:form.fullName.trim(), nickname:form.nickname.trim(),
      position:form.position, brand:form.brand, role:form.role, active:!!form.active, photo:form.photo||'' };
    try{
      if (editId){
        await colUsers().doc(editId).update(data);
        toast.success('แก้ไขข้อมูลผู้ใช้เรียบร้อย');
      } else {
        const cred = await secondaryAuth().createUserWithEmailAndPassword(form.email.trim(), form.password);
        await colUsers().doc(cred.user.uid).set({...data, email:form.email.trim(),
          createdAt: firebase.firestore.FieldValue.serverTimestamp()});
        await secondaryAuth().signOut();
        toast.success('เพิ่มผู้ใช้งานใหม่เรียบร้อย');
      }
      setOpen(false); reloadUsers();
    }catch(ex){
      const map = { 'auth/email-already-in-use':'อีเมลนี้ถูกใช้งานแล้ว','auth/weak-password':'รหัสผ่านสั้นเกินไป' };
      toast.error(map[ex.code] || 'บันทึกไม่สำเร็จ: '+ex.message);
    }
    setBusy(false);
  };

  const toggleActive = async u => {
    try{ await colUsers().doc(u.id).update({active: u.active===false});
      toast.success(u.active===false?'เปิดใช้งานผู้ใช้แล้ว':'ปิดการใช้งานผู้ใช้แล้ว'); reloadUsers(); }
    catch(e){ toast.error(e.message); }
  };
  const remove = async u => {
    if (u.id === profile.id) return toast.error('ไม่สามารถลบบัญชีของตัวเองได้');
    const ok = await confirm({ title:'ยืนยันการลบผู้ใช้', danger:true, okText:'ลบผู้ใช้',
      message:`ต้องการลบ "${u.fullName}" ออกจากระบบหรือไม่? (บัญชี Authentication ต้องลบแยกใน Firebase Console)` });
    if (!ok) return;
    try{ await colUsers().doc(u.id).delete(); toast.success('ลบผู้ใช้เรียบร้อย'); reloadUsers(); }
    catch(e){ toast.error(e.message); }
  };

  const list = users.filter(u => !q ||
    (u.fullName+u.nickname+(u.brand||'')+(u.position||'')).toLowerCase().includes(q.toLowerCase()));

  return (
    <Card>
      <CardHead icon="users" title="จัดการผู้ใช้งาน" sub={`ทั้งหมด ${users.length} คน`}
        right={<Btn size="sm" icon="plus" onClick={openNew}>เพิ่มผู้ใช้</Btn>}/>
      <div className="px-5 pb-3">
        <div className="relative max-w-xs">
          <Icon n="search" className="w-4 h-4 absolute left-3 top-1/2 -translate-y-1/2 text-slate-400"/>
          <Input value={q} onChange={e=>setQ(e.target.value)} placeholder="ค้นหาชื่อ / แบรนด์ / ตำแหน่ง" className="pl-9 py-2"/>
        </div>
      </div>
      <div className="px-2 pb-4">
        <Table head={['ผู้ใช้งาน','ตำแหน่ง','สังกัดแบรนด์','สิทธิ์',{label:'สถานะ',align:'center'},{label:'จัดการ',align:'center'}]}>
          {list.length===0 ? <Empty cols={6}/> : list.map(u=>(
            <tr key={u.id} className={cls('hover:bg-primary-50/40', u.active===false && 'opacity-60')}>
              <td className="px-3 py-2.5">
                <div className="flex items-center gap-3">
                  {u.photo ? <img src={u.photo} className="w-9 h-9 rounded-full object-cover border border-slate-200"/> :
                    <span className="grid place-items-center w-9 h-9 rounded-full bg-primary-100 text-primary-700 text-xs font-bold">
                      {(u.nickname||u.fullName||'?').slice(0,2).toUpperCase()}</span>}
                  <div className="min-w-0">
                    <p className="font-medium whitespace-nowrap">{u.nickname}</p>
                    <p className="text-[11px] text-slate-400 whitespace-nowrap">{u.fullName} · {u.email||'-'}</p>
                  </div>
                </div></td>
              <td className="px-3 py-2.5"><Badge tone="blue">{u.position||'-'}</Badge></td>
              <td className="px-3 py-2.5 whitespace-nowrap">
                {u.brand ? <span className="inline-flex items-center gap-1.5">
                  <span className="w-2.5 h-2.5 rounded-full" style={{background:brands.find(b=>b.name===u.brand)?.color||'#94a3b8'}}/>
                  {u.brand}</span> : <span className="text-slate-400">-</span>}</td>
              <td className="px-3 py-2.5"><Badge tone={u.role==='admin'?'amber':'gray'}>{u.role==='admin'?'Admin':'User'}</Badge></td>
              <td className="px-3 py-2.5 text-center">
                <button onClick={()=>toggleActive(u)}
                  className={cls('relative w-11 h-6 rounded-full transition', u.active===false?'bg-slate-300':'bg-emerald-500')}>
                  <span className={cls('absolute top-0.5 w-5 h-5 bg-white rounded-full shadow transition-all',
                    u.active===false?'left-0.5':'left-[22px]')}/>
                </button></td>
              <td className="px-3 py-2.5">
                <div className="flex gap-1.5 justify-center">
                  <Btn size="sm" variant="soft" icon="edit" onClick={()=>openEdit(u)}>แก้ไข</Btn>
                  <Btn size="sm" variant="dangerSoft" icon="trash" onClick={()=>remove(u)}>ลบ</Btn>
                </div></td>
            </tr>
          ))}
        </Table>
      </div>

      <Modal open={open} onClose={()=>setOpen(false)} title={editId?'แก้ไขผู้ใช้งาน':'เพิ่มผู้ใช้งานใหม่'}>
        <form onSubmit={save} className="space-y-4">
          <div className="flex items-center gap-4">
            {form.photo ? <img src={form.photo} className="w-20 h-20 rounded-2xl object-cover border border-slate-200"/> :
              <div className="w-20 h-20 rounded-2xl bg-primary-50 text-primary-400 grid place-items-center"><Icon n="users" className="w-8 h-8"/></div>}
            <div>
              <input ref={fileRef} type="file" accept="image/*" className="hidden" onChange={pick}/>
              <Btn type="button" size="sm" variant="outline" icon="download" onClick={()=>fileRef.current.click()}>อัปโหลดรูปโปรไฟล์</Btn>
              {form.photo && <Btn type="button" size="sm" variant="ghost" className="ml-1" onClick={()=>set('photo','')}>ลบรูป</Btn>}
              <p className="text-[11px] text-slate-400 mt-1.5">รองรับ JPG/PNG ขนาดไม่เกิน 2 MB</p>
            </div>
          </div>
          <div className="grid sm:grid-cols-2 gap-4">
            <Field label="ชื่อ-นามสกุล" required><Input value={form.fullName} onChange={e=>set('fullName',e.target.value)} placeholder="สมชาย ใจดี"/></Field>
            <Field label="ชื่อเล่น" required><Input value={form.nickname} onChange={e=>set('nickname',e.target.value)} placeholder="ชาย"/></Field>
            <Field label="ตำแหน่ง" required>
              <Select value={form.position} onChange={e=>set('position',e.target.value)}>
                {POSITIONS.map(p=><option key={p} value={p}>{p}</option>)}
              </Select></Field>
            <Field label="สังกัดแบรนด์ (BRAND)">
              <Select value={form.brand} onChange={e=>set('brand',e.target.value)}>
                <option value="">-- ไม่ระบุ --</option>
                {brands.map(b=><option key={b.id} value={b.name}>{b.name}</option>)}
              </Select></Field>
            <Field label="สิทธิ์การใช้งาน" required>
              <Select value={form.role} onChange={e=>set('role',e.target.value)}>
                <option value="user">User (ทั่วไป)</option>
                <option value="admin">Admin (ผู้ดูแลระบบ)</option>
              </Select></Field>
            <Field label="สถานะการใช้งาน">
              <Select value={form.active?'1':'0'} onChange={e=>set('active',e.target.value==='1')}>
                <option value="1">เปิดใช้งาน</option><option value="0">ปิดการใช้งาน</option>
              </Select></Field>
            {!editId && <>
              <Field label="อีเมล (สำหรับ Login)" required><Input type="email" value={form.email} onChange={e=>set('email',e.target.value)} placeholder="user@company.com"/></Field>
              <Field label="รหัสผ่าน" required hint="อย่างน้อย 6 ตัวอักษร">
                <Input type="password" value={form.password} onChange={e=>set('password',e.target.value)} placeholder="••••••"/></Field>
            </>}
          </div>
          <div className="flex gap-2 justify-end pt-2">
            <Btn type="button" variant="outline" onClick={()=>setOpen(false)}>ยกเลิก</Btn>
            <Btn type="submit" disabled={busy} icon={busy?null:'save'}>
              {busy ? <Spinner className="w-4 h-4 text-white"/> : 'บันทึก'}</Btn>
          </div>
        </form>
      </Modal>
    </Card>
  );
}

function BrandsTab(){
  const { brands, reloadBrands } = useApp();
  const toast = useToast(); const confirm = useConfirm();
  const [open,setOpen] = useState(false);
  const [form,setForm] = useState({name:'', color:'#2577eb', order:99});
  const [editId,setEditId] = useState(null);

  const save = async e => {
    e.preventDefault();
    if (!form.name.trim()) return toast.error('กรุณากรอกชื่อแบรนด์');
    try{
      const id = editId || form.name.trim();
      await colBrands().doc(id).set({ name:form.name.trim(), color:form.color, order:num(form.order) }, {merge:true});
      toast.success(editId?'แก้ไขแบรนด์เรียบร้อย':'เพิ่มแบรนด์ใหม่เรียบร้อย');
      setOpen(false); reloadBrands();
    }catch(ex){ toast.error('บันทึกไม่สำเร็จ: '+ex.message); }
  };
  const remove = async b => {
    const ok = await confirm({ title:'ยืนยันการลบแบรนด์', danger:true, okText:'ลบแบรนด์',
      message:`ต้องการลบแบรนด์ "${b.name}" หรือไม่? ข้อมูลการขายเดิมจะยังคงอยู่` });
    if (!ok) return;
    try{ await colBrands().doc(b.id).delete(); toast.success('ลบแบรนด์เรียบร้อย'); reloadBrands(); }
    catch(e){ toast.error(e.message); }
  };

  return (
    <Card>
      <CardHead icon="tag" title="จัดการแบรนด์ (BRAND)" sub={`ทั้งหมด ${brands.length} แบรนด์ · กำหนดสีได้เอง`}
        right={<Btn size="sm" icon="plus" onClick={()=>{setForm({name:'',color:'#2577eb',order:(brands.length+1)});setEditId(null);setOpen(true);}}>เพิ่มแบรนด์</Btn>}/>
      <div className="px-5 pb-5 grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3">
        {brands.map(b=>(
          <div key={b.id} className="flex items-center gap-3 rounded-xl border border-slate-200 p-3 hover:shadow-soft transition">
            <span className="w-10 h-10 rounded-xl shrink-0 border border-slate-200" style={{background:b.color}}/>
            <div className="flex-1 min-w-0">
              <p className="font-medium truncate">{b.name}</p>
              <p className="text-[11px] text-slate-400 font-mono">{b.color}</p>
            </div>
            <button onClick={()=>{setForm({name:b.name,color:b.color,order:b.order||99});setEditId(b.id);setOpen(true);}}
              className="p-2 rounded-lg hover:bg-primary-50 text-primary-600"><Icon n="edit" className="w-4 h-4"/></button>
            <button onClick={()=>remove(b)} className="p-2 rounded-lg hover:bg-rose-50 text-rose-500"><Icon n="trash" className="w-4 h-4"/></button>
          </div>
        ))}
      </div>

      <Modal open={open} onClose={()=>setOpen(false)} title={editId?'แก้ไขแบรนด์':'เพิ่มแบรนด์ใหม่'} size="sm">
        <form onSubmit={save} className="space-y-4">
          <Field label="ชื่อแบรนด์" required>
            <Input value={form.name} onChange={e=>setForm(f=>({...f,name:e.target.value}))} placeholder="เช่น OPPO"/></Field>
          <Field label="สีประจำแบรนด์ (Hex)" required>
            <div className="flex gap-2">
              <input type="color" value={form.color} onChange={e=>setForm(f=>({...f,color:e.target.value}))}
                className="w-14 h-11 rounded-xl border border-slate-300 cursor-pointer bg-white p-1"/>
              <Input value={form.color} onChange={e=>setForm(f=>({...f,color:e.target.value}))} className="font-mono"/>
            </div></Field>
          <Field label="ลำดับการแสดงผล">
            <Input type="number" value={form.order} onChange={e=>setForm(f=>({...f,order:e.target.value}))}/></Field>
          <div className="flex gap-2 justify-end pt-2">
            <Btn type="button" variant="outline" onClick={()=>setOpen(false)}>ยกเลิก</Btn>
            <Btn type="submit" icon="save">บันทึก</Btn>
          </div>
        </form>
      </Modal>
    </Card>
  );
}

function TargetsTab(){
  const toast = useToast(); const confirm = useConfirm();
  const now = new Date();
  const [list,setList] = useState([]); const [loading,setLoading] = useState(true);
  const [form,setForm] = useState({ year:now.getFullYear(), month:now.getMonth()+1, amount:'', units:'' });

  const load = useCallback(async()=>{
    setLoading(true);
    try{
      const s = await colTargets().get();
      setList(s.docs.map(d=>({id:d.id,...d.data()})).sort((a,b)=>b.id.localeCompare(a.id)));
    }catch(e){ toast.error(e.message); }
    setLoading(false);
  },[]);
  useEffect(()=>{load();},[load]);

  const save = async e => {
    e.preventDefault();
    if (form.amount==='') return toast.error('กรุณากรอกเป้ายอดขาย');
    const id = `${form.year}-${pad(form.month)}`;
    try{
      await colTargets().doc(id).set({ year:num(form.year), month:num(form.month),
        amount:num(form.amount), units:num(form.units) }, {merge:true});
      toast.success(`ตั้งเป้าหมายเดือน ${MONTHS_TH[form.month-1]} ${form.year} เรียบร้อย`);
      setForm(f=>({...f, amount:'', units:''})); load();
    }catch(ex){ toast.error('บันทึกไม่สำเร็จ: '+ex.message); }
  };
  const remove = async t => {
    const ok = await confirm({title:'ยืนยันการลบเป้าหมาย', danger:true, okText:'ลบ',
      message:`ลบเป้าหมายเดือน ${MONTHS_TH[t.month-1]} ${t.year} หรือไม่?`});
    if (!ok) return;
    await colTargets().doc(t.id).delete(); toast.success('ลบเรียบร้อย'); load();
  };

  const years = []; for (let y=now.getFullYear()-2; y<=now.getFullYear()+2; y++) years.push(y);

  return (
    <div className="space-y-4">
      <Card>
        <CardHead icon="target" title="ตั้งเป้าหมายยอดขายรายเดือน" sub="กำหนดเป้ายอดขาย (บาท) และจำนวนเครื่องของแต่ละเดือน"/>
        <form onSubmit={save} className="px-5 pb-5 grid sm:grid-cols-2 lg:grid-cols-5 gap-4 items-end">
          <Field label="เดือน" required>
            <Select value={form.month} onChange={e=>setForm(f=>({...f,month:+e.target.value}))}>
              {MONTHS_TH.map((m,i)=><option key={m} value={i+1}>{m}</option>)}</Select></Field>
          <Field label="ปี" required>
            <Select value={form.year} onChange={e=>setForm(f=>({...f,year:+e.target.value}))}>
              {years.map(y=><option key={y} value={y}>{y}</option>)}</Select></Field>
          <Field label="เป้ายอดขาย (บาท)" required>
            <Input type="number" min="0" value={form.amount} onChange={e=>setForm(f=>({...f,amount:e.target.value}))} placeholder="3000000"/></Field>
          <Field label="เป้าจำนวนเครื่อง" hint="ใช้คำนวณ 'ต้องทำอีกกี่เครื่อง/วัน'">
            <Input type="number" min="0" value={form.units} onChange={e=>setForm(f=>({...f,units:e.target.value}))} placeholder="450"/></Field>
          <Btn type="submit" size="lg" icon="save">บันทึกเป้าหมาย</Btn>
        </form>
      </Card>

      <Card>
        <CardHead icon="file" title="เป้าหมายที่บันทึกไว้"/>
        <div className="px-2 pb-4">
          {loading ? <div className="grid place-items-center py-10"><Spinner/></div> :
          <Table head={['เดือน/ปี',{label:'เป้ายอดขาย (บาท)',align:'right'},{label:'เป้าเครื่อง',align:'right'},{label:'จัดการ',align:'center'}]}>
            {list.length===0 ? <Empty cols={4} text="ยังไม่ได้ตั้งเป้าหมาย"/> : list.map(t=>(
              <tr key={t.id} className="hover:bg-primary-50/40">
                <td className="px-3 py-2.5 font-medium">{MONTHS_TH[t.month-1]} {t.year}</td>
                <td className="px-3 py-2.5 text-right text-primary-700 font-semibold">{nf(t.amount)}</td>
                <td className="px-3 py-2.5 text-right">{t.units?nf(t.units):'-'}</td>
                <td className="px-3 py-2.5 text-center">
                  <Btn size="sm" variant="dangerSoft" icon="trash" onClick={()=>remove(t)}>ลบ</Btn></td>
              </tr>
            ))}
          </Table>}
        </div>
      </Card>
    </div>
  );
}

function KpiTab(){
  const { settings, reloadSettings } = useApp(); const toast = useToast();
  const [f,setF] = useState(settings);
  useEffect(()=>setF(settings),[settings]);
  const setTier = (key,i,field,v) =>
    setF(s=>({...s, [key]: s[key].map((t,idx)=> idx===i ? {...t,[field]:num(v)} : t)}));

  const save = async () => {
    try{
      await docSettings().set({ ...f, kpiSamsung:num(f.kpiSamsung), kpiTecnoInfinix:num(f.kpiTecnoInfinix),
        filmTargetPct:num(f.filmTargetPct), simRatioTarget:num(f.simRatioTarget) }, {merge:true});
      toast.success('บันทึกการตั้งค่า KPI และค่าคอมมิชชั่นเรียบร้อย'); reloadSettings();
    }catch(e){ toast.error('บันทึกไม่สำเร็จ: '+e.message); }
  };

  return (
    <div className="space-y-4">
      <Card>
        <CardHead icon="award" title="เงื่อนไข KPI BRAND" sub="เทียบกับจำนวนเครื่องทั้งหมดของพนักงานแต่ละคน"/>
        <div className="px-5 pb-5 grid sm:grid-cols-2 gap-4">
          <Field label="SAMSUNG เป้าหมาย (%)" required>
            <Input type="number" step="0.01" value={f.kpiSamsung} onChange={e=>setF(s=>({...s,kpiSamsung:e.target.value}))}/></Field>
          <Field label="TECNO + INFINIX เป้าหมาย (%)" required>
            <Input type="number" step="0.01" value={f.kpiTecnoInfinix} onChange={e=>setF(s=>({...s,kpiTecnoInfinix:e.target.value}))}/></Field>
        </div>
      </Card>

      <div className="grid md:grid-cols-2 gap-4">
        <Card>
          <CardHead icon="film" title="เรทค่าคอม Film" sub="ยอด Film เทียบยอดเครื่อง (บาท/แผ่น)"/>
          <div className="px-5 pb-5 space-y-3">
            {f.filmTiers.map((t,i)=>(
              <div key={i} className="flex items-center gap-2">
                <span className="text-xs text-slate-500 w-10">≥</span>
                <Input type="number" step="0.01" value={t.min} className="py-2"
                  onChange={e=>setTier('filmTiers',i,'min',e.target.value)}/>
                <span className="text-xs text-slate-500">% จ่าย</span>
                <Input type="number" step="0.01" value={t.rate} className="py-2"
                  onChange={e=>setTier('filmTiers',i,'rate',e.target.value)}/>
                <span className="text-xs text-slate-500 whitespace-nowrap">฿/แผ่น</span>
              </div>
            ))}
            <Field label="เป้า Film ผ่าน/ไม่ผ่าน (%)" hint="ใช้คำนวณข้อความ 'ต้องทำอีกกี่แผ่น' บนแดชบอร์ด">
              <Input type="number" step="0.01" value={f.filmTargetPct}
                onChange={e=>setF(s=>({...s,filmTargetPct:e.target.value}))}/></Field>
          </div>
        </Card>
        <Card>
          <CardHead icon="sim" title="เรทค่าคอม SIM" sub="จำนวน SIM เทียบยอดเครื่อง (% ของ PackFee)"/>
          <div className="px-5 pb-5 space-y-3">
            {f.simTiers.map((t,i)=>(
              <div key={i} className="flex items-center gap-2">
                <span className="text-xs text-slate-500 w-10">≥</span>
                <Input type="number" step="0.01" value={t.min} className="py-2"
                  onChange={e=>setTier('simTiers',i,'min',e.target.value)}/>
                <span className="text-xs text-slate-500">% จ่าย</span>
                <Input type="number" step="0.01" value={t.rate} className="py-2"
                  onChange={e=>setTier('simTiers',i,'rate',e.target.value)}/>
                <span className="text-xs text-slate-500 whitespace-nowrap">% PackFee</span>
              </div>
            ))}
            <Field label="อัตราส่วน SIM เป้าหมาย (1 : X)" hint="ค่าเริ่มต้น 4.5 — เกินกว่านี้แสดงสีแดงบนแดชบอร์ด">
              <Input type="number" step="0.1" value={f.simRatioTarget}
                onChange={e=>setF(s=>({...s,simRatioTarget:e.target.value}))}/></Field>
          </div>
        </Card>
      </div>
      <div className="flex justify-end"><Btn size="lg" icon="save" onClick={save}>บันทึกการตั้งค่าทั้งหมด</Btn></div>
    </div>
  );
}

function PageSettings(){
  const [tab,setTab] = useState('users');
  const tabs = [['users','จัดการผู้ใช้งาน','users'],['brands','จัดการแบรนด์','tag'],
                ['targets','เป้าหมายยอดขาย','target'],['kpi','KPI & คอมมิชชั่น','award']];
  return (
    <div className="space-y-5">
      <div className="flex gap-2 overflow-x-auto pb-1">
        {tabs.map(([k,l,ic])=>(
          <button key={k} onClick={()=>setTab(k)}
            className={cls('inline-flex items-center gap-2 px-4 py-2.5 rounded-xl text-sm font-medium whitespace-nowrap transition border',
              tab===k ? 'bg-primary-600 text-white border-primary-600 shadow-sm'
                      : 'bg-white text-slate-600 border-slate-200 hover:bg-slate-50')}>
            <Icon n={ic} className="w-4 h-4"/>{l}
          </button>
        ))}
      </div>
      {tab==='users'   && <UsersTab/>}
      {tab==='brands'  && <BrandsTab/>}
      {tab==='targets' && <TargetsTab/>}
      {tab==='kpi'     && <KpiTab/>}
    </div>
  );
}

/* ============================================================
   15) SHELL (SIDEBAR + TOPBAR)
   ============================================================ */
const NAV = [
  { key:'dashboard', label:'แดชบอร์ดภาพรวม', icon:'home',     admin:false },
  { key:'entry',     label:'บันทึกข้อมูลการขาย', icon:'plus',  admin:false },
  { key:'daily',     label:'ยอดขายรายวัน',   icon:'calendar',  admin:false },
  { key:'report',    label:'รายงานรายเดือน', icon:'file',      admin:false },
  { key:'kpi',       label:'KPI & คอมมิชชั่น', icon:'award',   admin:false },
  { key:'market',    label:'Market Share',   icon:'pie',       admin:false },
  { key:'settings',  label:'ตั้งค่าและจัดการข้อมูล', icon:'gear', admin:true }
];

function Shell(){
  const { profile, isAdmin } = useApp(); const confirm = useConfirm();
  const [page,setPage] = useState('dashboard');
  const [open,setOpen] = useState(false);

  const items = NAV.filter(n => !n.admin || isAdmin);
  const current = items.find(i=>i.key===page) || items[0];

  const logout = async () => {
    const ok = await confirm({title:'ออกจากระบบ', message:'ต้องการออกจากระบบใช่หรือไม่?', okText:'ออกจากระบบ'});
    if (ok) auth.signOut();
  };

  const Nav = () => (
    <nav className="flex-1 px-3 py-4 space-y-1 overflow-y-auto">
      {items.map(n=>(
        <button key={n.key} onClick={()=>{setPage(n.key); setOpen(false);}}
          className={cls('w-full flex items-center gap-3 px-3.5 py-3 rounded-xl text-sm font-medium transition',
            page===n.key ? 'bg-primary-600 text-white shadow-sm shadow-primary-200'
                         : 'text-slate-600 hover:bg-primary-50 hover:text-primary-700')}>
          <Icon n={n.icon} className="w-5 h-5 shrink-0"/>
          <span className="truncate">{n.label}</span>
          {n.admin && <Badge tone={page===n.key?'blue':'amber'} className={page===n.key?'bg-white/20 text-white':''}>Admin</Badge>}
        </button>
      ))}
    </nav>
  );

  const SideContent = () => (
    <>
      <div className="flex items-center gap-3 px-5 h-16 border-b border-slate-100 shrink-0">
        <span className="grid place-items-center w-10 h-10 rounded-xl bg-gradient-to-br from-primary-500 to-primary-700 text-white shrink-0">
          <Icon n="trend"/></span>
        <div className="min-w-0">
          <p className="font-bold text-slate-800 text-sm leading-tight truncate">Sales System</p>
          <p className="text-[11px] text-slate-400 truncate">ระบบจัดการยอดขาย</p>
        </div>
        <button onClick={()=>setOpen(false)} className="lg:hidden ml-auto p-1.5 rounded-lg hover:bg-slate-100 text-slate-500">
          <Icon n="x"/></button>
      </div>
      <Nav/>
      <div className="p-3 border-t border-slate-100 shrink-0">
        <div className="flex items-center gap-3 px-2 py-2 rounded-xl bg-slate-50">
          {profile.photo ? <img src={profile.photo} className="w-9 h-9 rounded-full object-cover shrink-0"/> :
            <span className="grid place-items-center w-9 h-9 rounded-full bg-primary-100 text-primary-700 text-xs font-bold shrink-0">
              {(profile.nickname||profile.fullName||'?').slice(0,2).toUpperCase()}</span>}
          <div className="min-w-0 flex-1">
            <p className="text-sm font-medium truncate">{profile.nickname||profile.fullName}</p>
            <p className="text-[11px] text-slate-400 truncate">{isAdmin?'Admin':'User'} · {profile.position||'-'}</p>
          </div>
          <button onClick={logout} className="p-2 rounded-lg hover:bg-rose-50 text-rose-500 shrink-0" title="ออกจากระบบ">
            <Icon n="logout" className="w-4 h-4"/></button>
        </div>
      </div>
    </>
  );

  return (
    <div className="min-h-screen flex bg-slate-50">
      {/* Desktop sidebar */}
      <aside className="hidden lg:flex flex-col w-64 bg-white border-r border-slate-200 fixed inset-y-0 left-0 z-30">
        <SideContent/>
      </aside>

      {/* Mobile drawer */}
      {open && (
        <div className="lg:hidden fixed inset-0 z-50">
          <div className="absolute inset-0 bg-slate-900/50 backdrop-blur-[2px]" onClick={()=>setOpen(false)}/>
          <aside className="absolute inset-y-0 left-0 w-72 bg-white flex flex-col shadow-2xl fade-in"><SideContent/></aside>
        </div>
      )}

      <div className="flex-1 lg:ml-64 min-w-0">
        <header className="sticky top-0 z-20 bg-white/85 backdrop-blur border-b border-slate-200">
          <div className="h-16 px-4 sm:px-6 flex items-center gap-3">
            <button onClick={()=>setOpen(true)} className="lg:hidden p-2 rounded-xl hover:bg-slate-100 text-slate-600">
              <Icon n="menu"/></button>
            <div className="min-w-0">
              <h1 className="font-semibold text-slate-800 truncate">{current.label}</h1>
              <p className="text-[11px] text-slate-400 hidden sm:block">
                {new Date().toLocaleDateString('th-TH',{weekday:'long',day:'numeric',month:'long',year:'numeric'})}</p>
            </div>
            <div className="ml-auto flex items-center gap-2">
              <Badge tone={isAdmin?'amber':'blue'} className="hidden sm:inline-flex">{isAdmin?'Admin':'User'}</Badge>
              {profile.photo ? <img src={profile.photo} className="w-9 h-9 rounded-full object-cover border border-slate-200"/> :
                <span className="grid place-items-center w-9 h-9 rounded-full bg-primary-100 text-primary-700 text-xs font-bold">
                  {(profile.nickname||profile.fullName||'?').slice(0,2).toUpperCase()}</span>}
            </div>
          </div>
        </header>

        <main className="p-4 sm:p-6 max-w-[1600px] mx-auto">
          {page==='dashboard' && <PageDashboard/>}
          {page==='entry'     && <PageSalesEntry/>}
          {page==='daily'     && <PageDaily/>}
          {page==='report'    && <PageReport/>}
          {page==='kpi'       && <PageKPI/>}
          {page==='market'    && <PageMarketShare/>}
          {page==='settings'  && isAdmin && <PageSettings/>}
        </main>
      </div>
    </div>
  );
}

/* ============================================================
   16) ROOT APP
   ============================================================ */
function App(){
  const [user,setUser]       = useState(undefined);
  const [profile,setProfile] = useState(null);
  const [brands,setBrands]   = useState([]);
  const [users,setUsers]     = useState([]);
  const [settings,setSettings] = useState(DEFAULT_SETTINGS);
  const [booting,setBooting] = useState(true);
  const [denied,setDenied]   = useState('');

  const reloadBrands = useCallback(async()=>{
    const s = await colBrands().get();
    setBrands(s.docs.map(d=>({id:d.id,...d.data()})).sort((a,b)=>(a.order||99)-(b.order||99)));
  },[]);
  const reloadUsers = useCallback(async()=>{
    const s = await colUsers().get();
    setUsers(s.docs.map(d=>({id:d.id,...d.data()})).sort((a,b)=>(a.nickname||'').localeCompare(b.nickname||'')));
  },[]);
  const reloadSettings = useCallback(async()=>{
    const s = await docSettings().get();
    setSettings(s.exists ? {...DEFAULT_SETTINGS, ...s.data()} : DEFAULT_SETTINGS);
  },[]);

  useEffect(()=>auth.onAuthStateChanged(async u=>{
    setDenied('');
    if (!u){ setUser(null); setProfile(null); setBooting(false); return; }
    try{
      await seedIfEmpty();
      let snap = await colUsers().doc(u.uid).get();
      if (!snap.exists){
        const any = await colUsers().limit(1).get();
        if (any.empty){
          await colUsers().doc(u.uid).set({ fullName: u.email.split('@')[0], nickname:'Admin',
            position:'SM', brand:'', role:'admin', active:true, photo:'', email:u.email,
            createdAt: firebase.firestore.FieldValue.serverTimestamp() });
          snap = await colUsers().doc(u.uid).get();
        } else {
          setDenied('บัญชีนี้ยังไม่ได้รับสิทธิ์เข้าใช้งาน กรุณาติดต่อผู้ดูแลระบบ');
          await auth.signOut(); setBooting(false); return;
        }
      }
      const p = { id:u.uid, ...snap.data() };
      if (p.active === false){
        setDenied('บัญชีนี้ถูกปิดการใช้งาน กรุณาติดต่อผู้ดูแลระบบ');
        await auth.signOut(); setBooting(false); return;
      }
      setProfile(p); setUser(u);
      await Promise.all([reloadBrands(), reloadUsers(), reloadSettings()]);
    }catch(e){
      setDenied('เกิดข้อผิดพลาด: '+e.message);
      await auth.signOut();
    }
    setBooting(false);
  }),[]);

  if (user===undefined || booting)
    return <div className="min-h-screen grid place-items-center bg-white">
      <div className="text-center">
        <Spinner className="w-10 h-10 mx-auto"/>
        <p className="text-sm text-slate-400 mt-4">กำลังโหลดระบบ...</p>
      </div></div>;

  if (!user || !profile)
    return <>
      {denied && <div className="fixed top-4 left-1/2 -translate-x-1/2 z-[100] bg-rose-50 border border-rose-200 text-rose-700 text-sm px-4 py-3 rounded-xl shadow-lg fade-in">
        {denied}</div>}
      <LoginPage/>
    </>;

  const ctx = { profile, isAdmin: profile.role==='admin', brands, users, settings,
                reloadBrands, reloadUsers, reloadSettings };
  return <AppCtx.Provider value={ctx}><Shell/></AppCtx.Provider>;
}

ReactDOM.createRoot(document.getElementById('root'))
  .render(<ToastHost><ConfirmHost><App/></ConfirmHost></ToastHost>);
</script>
</body>
</html>
