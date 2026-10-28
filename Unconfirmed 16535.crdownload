/* Pure financial logic. No DOM, no storage. All money = integer cents. */
const Core=(()=>{
const TYPES=['income','expense','transfer','savings_in','savings_out','debt_principal','debt_interest','debt_borrow'];
const inM=(t,m)=>t.date.slice(0,7)===m;
const sum=a=>a.reduce((s,t)=>s+t.amountCents,0);
const cents=v=>{const n=Number(String(v).replace(/,/g,'').trim());return(!isFinite(n)||n<=0||n>1e9||String(v).trim()==='')?null:Math.round(n*100)};
function eff(t){const a=t.accountId,b=t.toAccountId,x=t.amountCents;
 switch(t.type){
  case'income':case'savings_out':case'debt_borrow':return[[a,x]];
  case'expense':case'savings_in':case'debt_principal':case'debt_interest':return[[a,-x]];
  case'transfer':return[[a,-x],[b,x]];}return[]}
function balances(accts,txs){const m={};accts.forEach(a=>m[a.id]=a.openingCents||0);
 txs.forEach(t=>eff(t).forEach(([id,d])=>{if(id in m)m[id]+=d}));return m}
const isSpend=t=>t.type==='expense'||t.type==='debt_interest';
const spend=(txs,m,cat)=>sum(txs.filter(t=>isSpend(t)&&inM(t,m)&&(!cat||t.categoryId===cat)));
const income=(txs,m)=>sum(txs.filter(t=>t.type==='income'&&inM(t,m)));
const status=(s,l)=>!l?'none':s>=l?'over':s*100>=l*80?'warn':'ok';
const pct=(s,l)=>l?Math.round(s*100/l):0;
const budget=(bs,m)=>{const r=bs.filter(b=>b.month<=m).sort((a,b)=>a.month<b.month?-1:1).pop();
 return r||{overallCents:200000,limits:{}}};
const goalBal=(g,txs)=>(g.openingCents||0)+sum(txs.filter(t=>t.goalId===g.id&&t.type==='savings_in'))-sum(txs.filter(t=>t.goalId===g.id&&t.type==='savings_out'));
const debtBal=(d,txs)=>{const f=ty=>sum(txs.filter(t=>t.debtId===d.id&&t.type===ty));
 return(d.openingCents||0)+f('debt_borrow')-f('debt_principal') /* interest/fees are paid from an account when recorded, so they are an expense but do not change principal owed */};
const q=s=>{s=String(s??'');return/[",\n\r]/.test(s)?'"'+s.replace(/"/g,'""')+'"':s};
function csv(txs,name){const h=['id','date','type','amount_sgd','category','source','account','to_account','payment_method','note'];
 const rows=[...txs].sort((a,b)=>a.date<b.date?-1:1).map(t=>[t.id,t.date,t.type,(t.amountCents/100).toFixed(2),name('categories',t.categoryId),name('sources',t.sourceId),name('accounts',t.accountId),name('accounts',t.toAccountId),t.paymentMethod||'',t.note||''].map(q).join(','));
 return[h.join(','),...rows].join('\r\n')}
const STORES=['accounts','categories','sources','transactions','budgets','bills','goals','debts','settings'];
function validate(o){if(!o||typeof o!=='object'||o.app!=='budget-tracker'||o.version!==1)return'Not a valid backup file (wrong format/version).';
 for(const s of STORES)if(!Array.isArray(o.data?.[s]))return'Backup is missing "'+s+'".';
 for(const r of STORES.flatMap(s=>o.data[s]))if(!r||typeof r.id!=='string')return'A record has no valid id.';
 const ids=new Set();for(const t of o.data.transactions){
  if(ids.has(t.id))return'Duplicate transaction id.';ids.add(t.id);
  if(!TYPES.includes(t.type)||!Number.isInteger(t.amountCents)||t.amountCents<=0||!/^\d{4}-\d{2}-\d{2}$/.test(t.date))return'Invalid transaction '+t.id+'.'}
 return null}
return{TYPES,STORES,cents,eff,balances,spend,income,status,pct,budget,goalBal,debtBal,csv,validate,inM,sum}})();
if(typeof module!=='undefined')module.exports=Core;
