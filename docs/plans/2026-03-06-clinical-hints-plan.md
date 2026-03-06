# Clinical Hints (Podpowiedzi kliniczne) Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a toggleable clinical hints step (step 0) before SAMPLE in each scenario, with a toggle on the welcome modal persisted in localStorage.

**Architecture:** Single-file changes to `index.html`. Add CSS for toggle switch and hints step styling. Add `hints` data to each scenario in `SC`. Modify `start()` to conditionally prepend hints step. Modify `render()` to handle read-only hints rendering.

**Tech Stack:** Vanilla JS, HTML, CSS (no dependencies)

---

### Task 1: Add toggle switch CSS

**Files:**
- Modify: `index.html:100` (after `.modal-content .version` style)

**Step 1: Add CSS for toggle switch after line 100**

Insert after `.modal-content .version { ... }` (line 100):

```css
.toggle-row { display:flex; align-items:center; justify-content:center; gap:10px; margin-bottom:16px; }
.toggle-label { font-size:13px; color:var(--dm); }
.toggle { position:relative; width:44px; height:24px; cursor:pointer; }
.toggle input { opacity:0; width:0; height:0; }
.toggle .slider { position:absolute; top:0; left:0; right:0; bottom:0; background:var(--bd); border-radius:12px; transition:background 0.2s; }
.toggle .slider::before { content:''; position:absolute; width:18px; height:18px; left:3px; bottom:3px; background:var(--dm); border-radius:50%; transition:all 0.2s; }
.toggle input:checked + .slider { background:var(--gn); }
.toggle input:checked + .slider::before { transform:translateX(20px); background:var(--tx); }
```

**Step 2: Commit**
```bash
git add index.html
git commit -m "Add CSS for toggle switch component"
```

---

### Task 2: Add hints step CSS

**Files:**
- Modify: `index.html` (after toggle CSS added in Task 1)

**Step 1: Add CSS for hints step rendering**

Insert after the toggle CSS:

```css
.hints-step { background:var(--sf); border:1px solid var(--bd); border-radius:var(--r); padding:20px; margin-bottom:24px; }
.hints-cat { margin-bottom:16px; }
.hints-cat:last-child { margin-bottom:0; }
.hints-cat-title { font-size:12px; font-weight:600; color:var(--gn); text-transform:uppercase; letter-spacing:0.5px; margin-bottom:8px; }
.hints-item { font-size:13px; color:var(--dm); padding-left:14px; position:relative; margin-bottom:4px; }
.hints-item::before { content:"\2022"; position:absolute; left:0; color:var(--gn); }
```

**Step 2: Commit**
```bash
git add index.html
git commit -m "Add CSS for hints step display"
```

---

### Task 3: Add toggle switch to welcome modal HTML

**Files:**
- Modify: `index.html:139-140` (between version and author lines in welcome modal)

**Step 1: Insert toggle HTML after the version line (line 139)**

After `<p class="version">Wersja 0.2 (marzec 26)</p>` insert:

```html
<div class="toggle-row">
  <span class="toggle-label">Podpowiedzi kliniczne</span>
  <label class="toggle"><input type="checkbox" id="hints-toggle" checked><span class="slider"></span></label>
</div>
```

**Step 2: Commit**
```bash
git add index.html
git commit -m "Add hints toggle switch to welcome modal"
```

---

### Task 4: Add localStorage logic for toggle

**Files:**
- Modify: `index.html:1269-1271` (close-modal event listener area)

**Step 1: Add toggle initialization and persistence logic**

Replace the close-modal listener block (lines 1269-1271) with:

```javascript
(function(){
  var ht=document.getElementById('hints-toggle');
  var stored=localStorage.getItem('hintsEnabled');
  ht.checked=stored===null?true:stored==='1';
  ht.addEventListener('change',function(){localStorage.setItem('hintsEnabled',ht.checked?'1':'0')});
})();
document.getElementById('close-modal').addEventListener('click', function() {
  document.getElementById('welcome-modal').style.display = 'none';
});
```

**Step 2: Commit**
```bash
git add index.html
git commit -m "Add localStorage persistence for hints toggle"
```

---

### Task 5: Add hints data to all 16 scenarios

**Files:**
- Modify: `index.html` — each scenario object in `SC` (lines 209-1205)

**Step 1: Add `hints` property to each scenario**

Add a `hints` object as the first property of each scenario, right after the scenario key. Each hints object has three arrays: `badanie`, `diagnostyka`, `postepowanie`.

Here is the full hints data for all 16 scenarios:

```javascript
// chest_pain (line ~211)
hints:{
  badanie:["Osłuchiwanie klatki piersiowej","Ocena tętna na obwodzie i symetrii","Ocena wypełnienia żył szyjnych","Pomiar RR na obu kończynach","Ocena obrzęków kk. dolnych"],
  diagnostyka:["EKG 12-odprowadzeniowe (powtórzyć po 10 min)","Pomiar SpO2","Pomiar glikemii","Ocena bólu w skali NRS"],
  postepowanie:["Protokół MONA (Morfina, Tlen, Nitrogliceryna, ASA)","ASA 300 mg rozgryźć (bez alergii!)","Nitrogliceryna 0,4 mg s.l. (SBP>90)","Dostęp dożylny","Pozycja półsiedząca","Przygotować do transportu na hemodynamikę przy STEMI"]
},

// dyspnea (line ~261 approx)
hints:{
  badanie:["Osłuchiwanie płuc (świsty, rzężenia, szmer oddechowy)","Częstość oddechów i tor oddychania","Praca dodatkowych mięśni oddechowych","Ocena symetrii ruchów klatki","Ocena żył szyjnych","Obrzęki obwodowe"],
  diagnostyka:["Pomiar SpO2","EKG 12-odprowadzeniowe","Pomiar glikemii","Ocena w skali mMRC/stopień duszności","Kapnografia jeśli dostępna"],
  postepowanie:["Pozycja siedząca/wysoka","Tlenoterapia wg SpO2","Nebulizacja (salbutamol/ipratropium przy skurczu)","Dostęp dożylny","Rozważyć CPAP przy obrzęku płuc","Rozważyć adrenalinę przy podejrzeniu anafilaksji"]
},

// stroke
hints:{
  badanie:["Skala FAST (Face, Arms, Speech, Time)","Skala Cincinnati lub NIHSS","Ocena źrenic (wielkość, reakcja na światło)","Ocena siły mięśniowej kk.","Ocena czucia i koordynacji","Ocena mowy (afazja, dyzartria)","Objaw Babińskiego"],
  diagnostyka:["Pomiar glikemii (wykluczyć hipoglikemię!)","Pomiar RR","EKG 12-odprowadzeniowe","Pomiar SpO2","Temperatura ciała","Czas wystąpienia objawów (okno terapeutyczne!)"],
  postepowanie:["Zabezpieczyć drożność dróg oddechowych","Pozycja z głową 30°","Nie obniżać RR (chyba że >220/120)","Dostęp dożylny (strona zdrowa!)","Nie podawać glukozy iv (chyba że hipoglikemia)","Transport na oddział udarowy — powiadomić szpital"]
},

// cardiac_arrest
hints:{
  badanie:["Potwierdzić NZK: brak reakcji + brak oddechu/gasping","Kontrola tętna na t. szyjnej (max 10 s)","Ocena rytmu w defibrylatorze"],
  diagnostyka:["Podłączyć defibrylator / AED","Kapnografia (ETCO2) po intubacji","Monitorowanie jakości uciskania (głębokość, częstość)"],
  postepowanie:["Uciskanie 30:2 (100-120/min, 5-6 cm)","Defibrylacja przy VF/pVT (dwufazowo 150-200 J)","Adrenalina 1 mg iv co 3-5 min","Amiodaron 300 mg iv przy opornym VF/pVT","Zabezpieczyć drogi oddechowe (intubacja/SAD)","Rozważyć odwracalne przyczyny — 4H i 4T","Dostęp dożylny / doszpikowy"]
},

// hypertension
hints:{
  badanie:["Pomiar RR na obu kończynach","Ocena neurologiczna (ból głowy, zaburzenia widzenia, niedowład)","Osłuchiwanie płuc (obrzęk?)","Osłuchiwanie serca (szmer, galop S3)","Ocena dna oka jeśli możliwe","Ocena obrzęków"],
  diagnostyka:["EKG 12-odprowadzeniowe","Pomiar glikemii","Pomiar SpO2","Badanie neurologiczne (skala Glasgow)"],
  postepowanie:["Pozycja półsiedząca","Obniżyć RR o max 20-25% w ciągu 1h","Captopril 12,5-25 mg s.l. lub urapidyl iv","Furosemid iv przy obrzęku płuc","Dostęp dożylny","Monitorować RR co 5-10 min","Unikać gwałtownych spadków RR"]
},

// trauma
hints:{
  badanie:["Ocena wg ABCDE","Ocena mechanizmu urazu","Badanie głowy: rany, deformacje, płyn z uszu/nosa","Badanie kręgosłupa szyjnego (palpacja)","Badanie klatki piersiowej (symetria, trzeszczenie)","Badanie brzucha (tkliwość, obrona mięśniowa)","Badanie miednicy (stabilność)","Badanie kończyn (deformacje, tętno obwodowe)","Ocena neurologiczna (GCS, źrenice, czucie)"],
  diagnostyka:["Pomiar SpO2","Pomiar RR","Pomiar glikemii","EKG przy urazie klatki","Skala GCS","Ocena bólu NRS"],
  postepowanie:["Stabilizacja kręgosłupa szyjnego","Tamowanie krwawień zewnętrznych","2x dostęp dożylny (duży kaliber)","Płynoterapia — bolus 250 ml, ocena odpowiedzi","Unieruchomienie złamań","Zapobieganie hipotermii","Analgezja (fentanyl 1 mcg/kg iv)","Transport do centrum urazowego"]
},

// traffic
hints:{
  badanie:["Bezpieczeństwo miejsca zdarzenia!","Ocena liczby poszkodowanych — segregacja START","Ocena wg ABCDE każdego poszkodowanego","Mechanizm urazu (prędkość, rodzaj pojazdu, pasy, airbag)","Badanie kręgosłupa szyjnego","Pełne badanie urazowe jak trauma"],
  diagnostyka:["Pomiar SpO2","Pomiar RR","Skala GCS","Ocena bólu NRS","EKG przy urazie klatki"],
  postepowanie:["Zabezpieczenie kręgosłupa szyjnego (kołnierz)","Deska ortopedyczna / KED / materac próżniowy","2x dostęp dożylny","Tamowanie krwawień, opatrunki","Płynoterapia ostrożna (SBP>90)","Analgezja (fentanyl / ketamina)","Zapobieganie hipotermii","Wezwać LPR / HEMS przy wskazaniach"]
},

// abdominal
hints:{
  badanie:["Oglądanie brzucha (wzdęcie, asymetria, blizny)","Osłuchiwanie perystaltyki","Palpacja (tkliwość, obrona, Blumberg)","Ocena bólu: lokalizacja, charakter, promieniowanie","Badanie per rectum — krew (rozważyć)","Ocena odwodnienia (turgor skóry, śluzówki)"],
  diagnostyka:["Pomiar glikemii","Pomiar SpO2","Pomiar RR","EKG (wykluczyć OZW — ból w nadbrzuszu!)","Pomiar temperatury","Ocena bólu NRS"],
  postepowanie:["Pozycja komfortowa (na boku z podkurczonymi nogami)","Dostęp dożylny","Płynoterapia (0,9% NaCl)","Analgezja (metamizol / paracetamol iv / opioid)","Leki przeciwwymiotne (ondansetron 4 mg iv)","NIE podawać leków p.o.","Transport na SOR / oddział chirurgii"]
},

// hypoglycemia
hints:{
  badanie:["Ocena świadomości (GCS)","Ocena potliwości, bladości, drżenia","Ocena objawów neurologicznych","Ocena zdolności połykania"],
  diagnostyka:["Pomiar glikemii (pilny!)","Pomiar SpO2","Pomiar RR","EKG u diabetyków"],
  postepowanie:["Hipoglikemia przytomny: glukoza p.o. (sok, cukier, żel)","Hipoglikemia nieprzytomny: Glukoza 20% 100 ml iv","Glukagon 1 mg im jeśli brak dostępu iv","Kontrolna glikemia po 15 min","Hiperglikemia: 0,9% NaCl iv 500 ml/h","Insulina wg protokołu przy DKA","Wyrównanie elektrolitów (K+)"]
},

// allergy
hints:{
  badanie:["Ocena dróg oddechowych (obrzęk, stridor)","Osłuchiwanie płuc (świsty, skurcz)","Ocena skóry (pokrzywka, obrzęk naczynioruchowy)","Ocena RR (wstrząs anafilaktyczny?)","Czas od ekspozycji na alergen","Identyfikacja alergenu"],
  diagnostyka:["Pomiar SpO2","Pomiar RR","EKG","Pomiar glikemii"],
  postepowanie:["Adrenalina 0,3-0,5 mg im (przednio-boczna pow. uda!)","Powtórzyć adrenalinę po 5-15 min jeśli brak poprawy","Pozycja Trendelenburga (przy hipotonii)","2x dostęp dożylny","NaCl 0,9% bolus 500-1000 ml przy hipotonii","Clemastyna 2 mg iv","Metyloprednizolon 125 mg iv","Salbutamol nebulizacja przy skurczu oskrzeli","Obserwacja min. 4-6h (reakcja dwufazowa!)"]
},

// pregnancy
hints:{
  badanie:["Tydzień ciąży (data OM)","Ocena krwawienia z dróg rodnych","Ocena czynności skurczowej","Ocena ruchów płodu","Ocena objawów rzucawki (obrzęki, bóle głowy, zaburzenia widzenia)","Badanie RR (stan przedrzucawkowy?)"],
  diagnostyka:["Pomiar RR","Pomiar SpO2","Pomiar glikemii","Pomiar temperatury","EKG przy bólu w klatce piersiowej","HR płodu jeśli >24 Hbd"],
  postepowanie:["Pozycja na lewym boku (od 20 Hbd)","Dostęp dożylny","MgSO4 przy rzucawce/stanie przedrzucawkowym","Tokoliza przy porodzie przedwczesnym","Przygotować zestaw porodowy od 36 Hbd","Transport na oddział położniczy","Powiadomić szpital telefonicznie"]
},

// fever
hints:{
  badanie:["Pomiar temperatury (douszna/czołowa)","Ocena stanu nawodnienia","Ocena wysypki (petocje — meningokocemia!)","Objawy oponowe (sztywność karku, Kernig, Brudziński)","Ocena świadomości","Ocena fontanelli u niemowląt","Czas trwania gorączki"],
  diagnostyka:["Pomiar SpO2","Pomiar RR","Pomiar glikemii","EKG u starszych pacjentów","Kryteria qSOFA (FR≥22, zaburzenia świadomości, SBP≤100)"],
  postepowanie:["Paracetamol 15 mg/kg (dzieci) lub 1g iv (dorośli)","Ibuprofen u dzieci (jeśli >6 m.ż.)","Chłodzenie fizyczne","Płynoterapia iv przy odwodnieniu","Antybiotyk empiryczny przy podejrzeniu sepsy","Nie czekać z transportem przy objawach sepsy","Noworodek z gorączką — ZAWSZE pilny transport"]
},

// alcohol
hints:{
  badanie:["Ocena świadomości (GCS)","Ocena źrenic","Wykluczyć uraz głowy (mechanizm upadku!)","Ocena temperatury ciała (hipotermia!)","Ocena odwodnienia","Objawy odstawienia (drżenia, majaczenie, drgawki)","Wykluczyć hipoglikemię"],
  diagnostyka:["Pomiar glikemii (obowiązkowy!)","Pomiar SpO2","Pomiar RR","Pomiar temperatury","EKG przy kołataniu serca / arytmii"],
  postepowanie:["Zabezpieczyć drożność dróg oddechowych","Pozycja boczna ustalona (przy obniżonej świadomości)","Dostęp dożylny","Tiamina 100 mg iv PRZED glukozą","Glukoza iv przy hipoglikemii","Diazepam iv przy drgawkach / delirium tremens","Ogrzewanie przy hipotermii","Obserwacja — nie zostawiać bez opieki"]
},

// psych
hints:{
  badanie:["Ocena bezpieczeństwa (własnego i pacjenta!)","Ocena świadomości i orientacji","Ocena nastroju i afektu","Ocena myśli samobójczych (pytać wprost!)","Ocena pobudzenia psychomotorycznego","Wykluczyć przyczyny somatyczne (glikemia, uraz, leki)"],
  diagnostyka:["Pomiar glikemii (wykluczyć hipoglikemię!)","Pomiar SpO2","Pomiar RR","Pomiar temperatury","EKG przy podejrzeniu intoksykacji"],
  postepowanie:["Deeskalacja słowna (spokojny ton, krótkie zdania)","Zapewnić bezpieczeństwo (usunąć niebezpieczne przedmioty)","Midazolam 5 mg im lub haloperidol 5 mg im przy silnym pobudzeniu","Dostęp dożylny gdy możliwy","Ustawa o ochronie zdrowia psychicznego — art. 21/23","Transport z personelem (min. 2 osoby)","Dokumentować zachowanie i wypowiedzi pacjenta"]
},

// syncope
hints:{
  badanie:["Okoliczności zasłabnięcia (pozycja, aktywność, prodrom)","Czas trwania utraty przytomności","Ocena neurologiczna po odzyskaniu przytomności","Czy były drgawki? (świadkowie)","Ocena tętna (miarowy, niemiarowy, wolny)","Ocena urazu wtórnego (głowa!)"],
  diagnostyka:["EKG 12-odprowadzeniowe (blok AV, QT, Brugada)","Pomiar glikemii","Pomiar RR (leżąc i siedząc — próba ortostatyczna)","Pomiar SpO2","Pomiar temperatury"],
  postepowanie:["Pozycja leżąca z uniesionymi nogami","Dostęp dożylny","Płynoterapia przy hipotonii ortostatycznej","Atropina 0,5 mg iv przy bradykardii objawowej","Obserwacja i monitorowanie EKG","Transport na SOR (pierwsza utrata przytomności)","Kardiologiczne — pacjent musi mieć EKG na SOR"]
},

// general
hints:{
  badanie:["Ocena wg ABCDE","Ocena świadomości (GCS)","Pełne badanie fizykalne","Zebranie wywiadu SAMPLE","Ocena bólu (NRS)"],
  diagnostyka:["Pomiar SpO2","Pomiar RR","Pomiar glikemii","EKG 12-odprowadzeniowe","Pomiar temperatury"],
  postepowanie:["Dostęp dożylny wg potrzeb","Leczenie objawowe","Transport na SOR","Monitorowanie parametrów życiowych","Dokumentacja stanu pacjenta"]
},

// burns
hints:{
  badanie:["Ocena drożności dróg oddechowych (oparzenie inhalacyjne!)","Reguła dziewiątek Wallace'a — %TBSA","Głębokość oparzenia (I°, II°, III°)","Ocena oparzeń okrężnych (kończyny, klatka, szyja)","Mechanizm (termiczne, chemiczne, elektryczne)","Oparzenia twarzy, rąk, stóp, krocza — zawsze ciężkie"],
  diagnostyka:["Pomiar SpO2 (uwaga: CO zawyża!)","Pomiar RR","EKG przy oparzeniu elektrycznym","Pomiar glikemii","Ocena bólu NRS"],
  postepowanie:["Chłodzenie letnia wodą (15-20°C) max 20 min","NIE lód, NIE masło/mąka","Formuła Parklanda: 4ml × kg × %TBSA / 24h","Połowa w pierwszych 8h od oparzenia","Analgezja (morfina / fentanyl iv)","Opatrunki hydrożelowe / jałowe","Zabezpieczyć drogi oddechowe przy inhalacji","Escharotomia przy oparzeniu okrężnym","Transport do centrum oparzeniowego (>15% TBSA dorośli, >10% dzieci)"]
}
```

**Step 2: Commit**
```bash
git add index.html
git commit -m "Add clinical hints data to all 16 scenarios"
```

---

### Task 6: Modify start() to prepend hints step

**Files:**
- Modify: `index.html:1222` (start function)

**Step 1: Modify the start() function**

Replace line 1222:
```javascript
function start(k){cur=SC[k];allSteps=(k==='pocus'||k==='infusion')?cur.steps:[SAMPLE_STEP].concat(cur.steps);step=0;D={};show('s1');render()}
```

With:
```javascript
function start(k){
  cur=SC[k];curK=k;
  var base=(k==='pocus'||k==='infusion')?cur.steps:[SAMPLE_STEP].concat(cur.steps);
  var hintsOn=localStorage.getItem('hintsEnabled');hintsOn=hintsOn===null?true:hintsOn==='1';
  if(hintsOn&&cur.hints){
    var cat=CATS.find(function(c){return c.k===k});
    var title=cat?cat.l:k;
    allSteps=[{id:'_hints',title:'Wskazówki: '+title,hints:cur.hints}].concat(base);
  }else{allSteps=base;}
  step=0;D={};show('s1');render();
}
```

Also add `curK` to the variable declarations on line 1219:
```javascript
var cur=null,curK=null,step=0,D={},allSteps=[];
```

**Step 2: Commit**
```bash
git add index.html
git commit -m "Modify start() to prepend hints step when enabled"
```

---

### Task 7: Modify render() to display hints step

**Files:**
- Modify: `index.html:1229-1243` (render function)

**Step 1: Modify render() to handle hints step**

Replace the render function (lines 1229-1243) with:

```javascript
function render(){
  var s=allSteps[step];
  document.getElementById('si').innerHTML=allSteps.map(function(x,i){return'<div class="sd'+(i<step?' d':i===step?' cur':'')+'"></div>'}).join('');
  var htm='<div class="st">'+s.title+'</div>';
  if(s.hints){
    htm+='<div class="hints-step">';
    if(s.hints.badanie&&s.hints.badanie.length){htm+='<div class="hints-cat"><div class="hints-cat-title">Badanie</div>';s.hints.badanie.forEach(function(h){htm+='<div class="hints-item">'+h+'</div>'});htm+='</div>';}
    if(s.hints.diagnostyka&&s.hints.diagnostyka.length){htm+='<div class="hints-cat"><div class="hints-cat-title">Diagnostyka</div>';s.hints.diagnostyka.forEach(function(h){htm+='<div class="hints-item">'+h+'</div>'});htm+='</div>';}
    if(s.hints.postepowanie&&s.hints.postepowanie.length){htm+='<div class="hints-cat"><div class="hints-cat-title">Postępowanie</div>';s.hints.postepowanie.forEach(function(h){htm+='<div class="hints-item">'+h+'</div>'});htm+='</div>';}
    htm+='</div>';
  } else {
    s.fields.forEach(function(f){
      if(f.condition&&D[f.condition.field]!==f.condition.value)return;
      htm+='<div class="fl">'+f.label+'</div>';
      if(f.type==='single'){htm+='<div class="fg">';f.options.forEach(function(o){htm+='<div class="c'+(D[f.id]===o?' s':'')+'" onclick="s1(\''+e(f.id)+'\',\''+e(o)+'\')">'+o+'</div>'});htm+='</div>';}
      else if(f.type==='multi'){htm+='<div class="fg">';f.options.forEach(function(o){htm+='<div class="c m'+((D[f.id]||[]).indexOf(o)>=0?' s':'')+'" onclick="sM(\''+e(f.id)+'\',\''+e(o)+'\')">'+o+'</div>'});htm+='</div>';}
      else if(f.type==='text'){htm+='<input class="ti" type="text" data-f="'+f.id+'" placeholder="'+e(f.placeholder||'')+'" value="'+e(D[f.id]||'')+'">';}
    });
  }
  document.getElementById('fc').innerHTML=htm;
  document.getElementById('bP').style.display=step===0?'none':'';
  document.getElementById('bN').textContent=step===allSteps.length-1?'Generuj tekst':'Dalej';
}
```

**Step 2: Commit**
```bash
git add index.html
git commit -m "Modify render() to display read-only hints step"
```

---

### Task 8: Exclude hints step from completion percentage

**Files:**
- Modify: `index.html:1254-1256` (genOut function, completion calculation)

**Step 1: Filter out hints step from total field count**

Replace lines 1254-1256 in genOut:
```javascript
var total=allSteps.reduce(function(s,st){return s+st.fields.filter(function(f){return!f.condition}).length},0);
```

With:
```javascript
var total=allSteps.filter(function(st){return!st.hints}).reduce(function(s,st){return s+st.fields.filter(function(f){return!f.condition}).length},0);
```

**Step 2: Commit**
```bash
git add index.html
git commit -m "Exclude hints step from completion percentage calculation"
```

---

### Task 9: Apply same changes to kmcr-asystent.html

**Files:**
- Modify: `kmcr-asystent.html` — mirror all changes from Tasks 1-8

**Step 1: Apply all changes to the standalone downloadable version**

The file `kmcr-asystent.html` is the offline/downloadable copy. Apply the same CSS, HTML, and JS changes. Line numbers may differ slightly — use the same landmarks (modal-content, SC object, start function, render function, genOut function).

**Step 2: Commit**
```bash
git add kmcr-asystent.html
git commit -m "Mirror clinical hints feature in standalone version"
```

---

### Task 10: Manual testing checklist

**Step 1: Test the following in browser**

1. Open app — welcome modal shows toggle, default ON
2. Close modal, select "Ból w kl. piersiowej" — first step shows hints with 3 categories
3. Click "Dalej" — proceeds to SAMPLE step
4. Complete scenario — hints step excluded from % calculation
5. Reopen app — toggle still ON (localStorage)
6. Turn toggle OFF, close modal — select scenario, no hints step (starts at SAMPLE)
7. Reopen app — toggle still OFF
8. Test with pocus / infusion — no hints step regardless of toggle
9. Check mobile layout (responsive)

**Step 2: Final commit**
```bash
git add -A
git commit -m "feat: Add clinical hints feature (Podpowiedzi kliniczne)"
```
