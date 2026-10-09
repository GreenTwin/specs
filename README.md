# GreenTwin: Specificații platformă de cartografiere și monitorizare a copacilor

Oct 9, 2026 · @Catalin BORA

## Context și scop

Scopul platformei nu este o bază de date cu copaci, ci activarea responsabilității civice față de spațiul verde din jurul fiecărui cetățean. Datele sunt consecința, nu obiectivul: cine ajunge să ude un copac începe să îl observe, iar cine îl observă începe să raporteze.

**De ce reconstruim de la zero.** Aplicația existentă a fost gândită pentru copaci dintr-o pădure sau de pe un teren propriu, unde poți bate în trunchi etichete fizice cu cip NFC și unde numărul de arbori e mic. Pentru oraș modelul nu ține: copacii nu sunt ai utilizatorului, densitatea e mare, iar accesul e gândit intern (buton ascuns de login, cont creat manual, rol de manager atribuit de administrator). Structura bazei de date actuale e legată de alt set de caracteristici decât cel de care avem nevoie acum.

**Ce vrem în loc.** O aplicație publică, deschisă oricui, în care un cetățean fără nicio legătură cu GreenTwin poate spune „copacul din fața blocului meu e al meu" și poate contribui în 30 de secunde, de pe stradă.

**Obiectivele măsurabile ale primei versiuni:**

1. Un cartier sau un parc cartografiat complet (ordin de mărime 200 până la 300 de copaci), cu poziție, specie și stare.
2. Un set de date suficient de credibil încât să poată fi pus pe masă într-o discuție instituțională.
3. Cel puțin 10 voluntari care folosesc aplicația repetat, nu o singură dată.
4. Un model de date care se actualizează singur în timp, prin revizitări, nu printr-o campanie unică.

**Ce nu este platforma.** Nu este un instrument de inventariere oficială și nu înlocuiește evidențele administrației. Este un strat complementar, construit de cetățeni, care adaugă informații pe care evidențele publice nu le conțin (diametru de trunchi și de coronament, vârstă estimată, stare sanitară actuală, istoric de îngrijire) și care rămâne proaspăt pentru că oamenii trec pe lângă acei copaci zilnic.

## Utilizatori țintă

Publicul declarat este „orice cetățean interesat", dar în practică aplicația trebuie proiectată pentru cinci profiluri cu nevoi diferite. Prioritatea de design în prima versiune este profilul 2 și profilul 3, pentru că ei produc datele.

| # | Profil | Ce îl aduce în aplicație | Ce îi trebuie | Risc |
| --- | --- | --- | --- | --- |
| 1 | Cetățean ocazional | Copacul din fața blocului; a văzut o campanie | Zero fricțiune, fără cont la prima interacțiune, o singură acțiune clară | Intră o dată și nu revine |
| 2 | Voluntar activ GreenTwin | Udă deja copaci, vrea să își urmărească zona | Check-in rapid, istoric, listă de copaci proprii, mod offline | Se plictisește de formulare lungi |
| 3 | Tânăr 12 până la 25 de ani, student | Competiția, echipa, scorul | Joc, clasament, teritoriu, feedback instant | Trișează pentru puncte |
| 4 | Expert (profesor, biolog, arborist) | Calitatea datelor, nu jocul | Mod fără gamificare, câmpuri precise, arbitraj, export | Respinge aplicația dacă pare jucărie |
| 5 | Administrator GreenTwin | Campanii, validare, raportare | Dashboard, moderare, export, statistici pe zonă | Muncă manuală care nu scalează |

**Cine rămâne în afara scopului pentru moment.** Pensionarul din spatele blocului, care are interesul dar nu confortul cu telefonul, și copilul sub 12 ani. Pentru ei soluția nu este aplicația, ci evenimentul fizic organizat de un voluntar care are aplicația.

**Consecința de design cea mai importantă:** profilul 3 și profilul 4 se exclud reciproc dacă facem o singură interfață. De aici decizia celor două moduri din secțiunea următoare.

## Principii de produs și decizii luate

Acestea sunt deciziile structurante din întâlnirea din 8 octombrie. Ele fixează cadrul; restul documentului le detaliază.

1. **Web întâi, mobil după.** Website-ul se poate itera în ore, fără proces de aprobare în store. Prima versiune e o aplicație web pe mobil, instalabilă (PWA). Aplicația nativă vine când funcționalitățile sunt stabile și când precizia locației devine limitantă. Motivul istoric: prima încercare a fost tot un web app, iar problema lui a fost că locația venea din rețea și Wi-Fi, nu din GPS. Deci PWA, dar cu geolocalizare de înaltă precizie și plan clar de trecere la nativ.
2. **Două moduri, aceeași bază de date.** Mod joc (puncte, echipe, concurs) și mod serios (introducere rapidă, fără scor vizibil). Utilizatorul alege la onboarding și poate comuta oricând. Datele produse sunt identice.
3. **Open source de la prima zi.** Nu avem niciun motiv să ținem codul închis. Dacă altcineva îl ia și îl pune în alt oraș, obiectivul nostru tot se îndeplinește. Contribuitorii externi intră prin pull request.
4. **Ușurința de folosire și acuratețea se bat cap în cap.** Soluția nu e un formular lung, ci un minim obligatoriu foarte mic plus straturi opționale de detaliu, plus un mecanism de corecție colectivă. Nicio informație introdusă de un singur om nu e tratată ca adevăr.
5. **Nu cerem utilizatorului să fie expert.** Dacă nu știe specia, face o poză la frunză și trece mai departe. Identificarea se poate întâmpla mai târziu, de către model sau de către altcineva.
6. **Date vii, nu o fotografie în timp.** Un copac introdus acum trei ani nu mai e adevărat: a crescut, poate s-a uscat, poate a fost tăiat. Fiecare înregistrare are o vârstă a datei și devine „de reconfirmat".
7. **Începem cu spațiul public.** Străzi, parcuri, spații dintre blocuri. Curțile private rămân posibile, dar nu sunt prioritate și nu intră în statisticile publice.
8. **Începem cu o zonă mică.** Un cartier sau un parc, nu orașul. Densitatea mare pe o suprafață mică face jocul să funcționeze și face datele verificabile.

## Modelul de date

Principiul de bază: **copacul este o entitate stabilă, tot restul sunt observații datate atașate lui.** Nimeni nu „editează" un copac. Oamenii adaugă observații, iar valoarea afișată a copacului este rezultatul agregării lor. Așa rezolvăm simultan acuratețea, istoricul și arbitrajul.

### Copac

| Câmp | Tip | Obligatoriu | Observații |
| --- | --- | --- | --- |
| id | uuid | da | generat local, ca să meargă offline |
| latitudine, longitudine | zecimal (7 zecimale) | da | valoarea consolidată, mediată din observații |
| precizie\_estimata | metri | da | cât de sigură e poziția |
| specie\_id | referință | nu | valoarea consolidată prin arbitraj |
| incredere\_specie | 0 până la 1 | da | derivat, nu introdus |
| stare | enum | nu | sănătos, uscat parțial, uscat, tăiat, lipsă |
| zona\_id | referință | da | cartier sau campanie, derivat din coordonate |
| tip\_amplasament | enum | nu | aliniament stradal, parc, curte bloc, curte privată |
| adoptat\_de | referință utilizator | nu | vezi secțiunea de adopție |
| creat\_de, creat\_la | referință, timestamp | da |  |
| ultima\_observatie\_la | timestamp | da | folosit pentru prospețime |
| sursa | enum | da | voluntar, import, administrație |
| status\_moderare | enum | da | activ, duplicat, suspect, șters |

### Observație

Fiecare contribuție a unui utilizator despre un copac. Nu se șterge niciodată, se invalidează.

Câmpuri: `id`, `copac_id`, `utilizator_id`, `tip` (poziție, specie, măsurătoare, stare, fotografie, corecție), `valoare` (JSON, structurat pe tip), `coordonate_la_momentul_observatiei`, `precizie_gps`, `creat_la`, `pondere` (calculată din reputația utilizatorului), `status` (activă, suprascrisă, respinsă).

### Check-in

Vizită la un copac, cu sau fără date noi. Câmpuri: `id`, `copac_id`, `utilizator_id`, `creat_la`, `coordonate`, `distanta_fata_de_copac`, `raspunsuri` (două sau trei întrebări scurte), `fotografie_id`, `puncte_acordate`.

### Specie

Taxonomie proprie, simplificată. Câmpuri: `id`, `denumire_populara`, `denumire_stiintifica`, `gen`, `familie`, `este_invaziva` (boolean), `este_autohtona` (boolean), `caracteristici_de_recunoastere` (text scurt + poze de referință), `cod_registru` (mapare către nomenclatorul public, acolo unde există).

Lista pornește cu cele 30 până la 50 de specii frecvente în Timișoara, nu cu sute. Speciile invazive (salcâm, oțetar fals, paulownia) sunt marcate explicit, pentru că semnalarea lor e o funcționalitate cerută în întâlnire.

### Fotografie

`id`, `copac_id`, `observatie_id`, `tip` (frunză, trunchi, coroană, ansamblu, scoarță), `url`, `hash_perceptual`, `exif_data_captura`, `coordonate_exif`, `incarcata_la`, `verificata` (boolean antifraudă).

### Utilizator

`id`, `pseudonim`, `email` (opțional), `echipa_id`, `scor`, `nivel`, `reputatie` (0 până la 1, influențează ponderea observațiilor), `rol` (cetățean, voluntar, expert, moderator, administrator), `creat_la`.

### Echipă și Zonă

**Echipă:** `id`, `nume`, `culoare`, `scor_total`, `membri`. **Zonă:** `id`, `nume`, `poligon`, `tip` (cartier, parc, campanie), `tinta_copaci`, `copaci_mapati`, `echipa_dominanta`, `activa_intre` (două date).

### Ce NU stocăm

Nume reale, adrese, date de contact ale cetățenilor care nu sunt voluntari înregistrați, și fețe în fotografii (vezi secțiunea despre date personale).

## Fluxul principal: adăugarea unui copac

Ținta de timp: **sub 30 de secunde pentru minimul obligatoriu**, de pe stradă, cu o mână.

### Pașii

1. **Deschiderea hărții.** Harta se centrează pe poziția utilizatorului, cu un cerc care arată precizia reală a locației. Fundalul implicit este imagine satelitară, pentru că așa poți vedea coroanele existente și aliniamentele. Comutare rapidă pe hartă stradală, pentru că sub coroane satelitul nu ajută.
2. **Marcarea poziției.** Utilizatorul nu lasă punctul pe poziția proprie. Mută harta sub un reticul fix până când acesta stă peste copac. Este ușor neobișnuit ca interacțiune, dar e singura metodă care funcționează când stai lângă copac, nu în el.
3. **Limitarea deplasării.** Reticulul poate fi mutat doar într-o rază configurabilă față de poziția GPS (propunere inițială: 25 m). Peste această rază, aplicația cere confirmare explicită sau refuză. Motivul: altfel oamenii mapează copaci de acasă, de pe canapea.
4. **Verificarea duplicatelor.** Înainte de salvare, aplicația afișează copacii deja existenți pe o rază de 5 m și întreabă „e unul dintre aceștia?". Dacă utilizatorul confirmă, acțiunea devine check-in, nu adăugare. Aceasta este principala apărare împotriva dublării bazei de date.
5. **Minimul obligatoriu.** Doar poziția. Atât. Un copac cu poziție și nimic altceva este o înregistrare validă și valoroasă.
6. **Straturi opționale, în ordinea efortului:** fotografie la ansamblu, fotografie la frunză, specie (sugerată sau aleasă), stare, circumferința trunchiului, diametrul coroanei, observații libere.
7. **Salvare locală apoi sincronizare.** Înregistrarea se salvează pe dispozitiv imediat și se sincronizează când există rețea. Pe teren, în parc, semnalul e neuniform și nu putem pierde munca omului.

### Reguli de interfață

- Un singur buton principal pe ecran: „Adaugă copac". Restul se descoperă.
- Fotografia se face din aplicație, nu se alege din galerie, în modul joc. În modul serios se acceptă și din galerie, pentru cazuri legitime.
- Toate câmpurile opționale pot fi completate ulterior, de altcineva.
- După salvare, feedback imediat: ce s-a adăugat, cât valorează, ce lipsește.

## Precizia poziției

Aceasta este problema tehnică centrală a proiectului și a fost discutată cel mai mult. Un GPS de telefon dă în condiții bune 3 până la 4 m eroare. Într-un aliniament stradal, în 3 până la 4 m încap doi sau trei copaci, deci eroarea depășește distanța dintre entitățile pe care încercăm să le distingem.

**Cazuri observate concret:**

- Eroare tipică în oraș: 3 până la 7 m.
- Pe strada Nera, lângă două sau trei turnuri de antene, tot ce se marchează apare deplasat cu aproximativ 20 m.
- Între blocuri și sub coroane dese, eroarea crește și devine sistematică, nu aleatoare.

### Strategia în straturi

1. **Poziționare manuală asistată.** Omul decide unde e copacul, nu senzorul. GPS-ul dă doar punctul de plecare și raza de plauzibilitate.
2. **Imagine satelitară ca referință vizuală.** Pe aliniamente, coroanele sunt vizibile și omul poate potrivi punctul pe coroană.
3. **Medierea observațiilor.** Fiecare check-in ulterior aduce o nouă pereche de coordonate. Poziția consolidată a copacului este media ponderată a observațiilor, cu eliminarea valorilor extreme. Cu cinci vizite, eroarea scade semnificativ fără niciun echipament suplimentar.
4. **Respingerea punctelor aberante.** Dacă o observație e la peste 25 m de poziția consolidată, nu intră în medie și se marchează pentru verificare.
5. **Detectarea zonelor problematice.** Dacă într-o zonă mai multe observații au o deplasare sistematică în aceeași direcție, zona se marchează ca nesigură și se recomandă ajustare manuală mai atentă.

### Problema acumulării de erori

Dacă un copac e împins 2 m într-o direcție și vecinul lui 1 m în direcția opusă, ordinea lor pe stradă se poate inversa și nu mai știi la care faci check-in. Contramăsuri:

- La check-in, aplicația nu cere „găsește copacul pe hartă", ci propune cel mai apropiat copac și cere confirmare.
- Copacii dintr-un aliniament pot fi legați într-o secvență ordonată pe stradă; ordinea devine o constrângere care previne inversările.
- Dacă doi copaci ajung la sub 1,5 m unul de altul, se declanșează o verificare de duplicat.

### Opțiuni pentru mai târziu

Etichetele fizice (cuie de plastic cu cip NFC, bătute în trunchi) au fost testate conceptual pe platforma veche și rezolvă complet identificarea, dar presupun contact fizic cu fiecare copac, cost pe bucată și acordul proprietarului spațiului. Varianta mai ușoară este codul QR generat pentru fiecare copac, scanabil cu orice telefon, fără aplicație instalată, care deschide pagina publică a copacului. Pentru oraș, QR-ul rămâne o opțiune pentru copacii adoptați, nu pentru toți.

## Identificarea speciilor

Regula de bază: **nimeni nu e obligat să știe specia.** Dacă utilizatorul nu știe, face o poză la frunză și trece mai departe. Identificarea e o problemă a sistemului, nu a omului de pe stradă.

### Cele trei căi de identificare

1. **Utilizatorul știe.** Alege din listă. Lista nu e alfabetică și nu are sute de intrări: are speciile frecvente din zona respectivă, cu poză de referință lângă fiecare nume.
2. **Modelul propune.** Din poza la frunză (și, unde ajută, la scoarță sau la ansamblu) un model de recunoaștere propune două sau trei variante cu nivel de încredere. Utilizatorul confirmă sau respinge.
3. **Altcineva identifică mai târziu.** Copacul rămâne cu specie necunoscută și apare într-o coadă de „de identificat", accesibilă celor care știu. Un expert poate curăța zeci de intrări de acasă, într-o seară.

### Modelul de recunoaștere

În prima fază folosim un model extern, pentru că funcționează deja bine pe majoritatea cazurilor cu frunză vizibilă și nu are sens să construim noi unul. Ne interesează costul pe cerere și posibilitatea de credite pentru ONG-uri.

În paralel, fiecare poză confirmată de doi sau mai mulți oameni intră într-un set propriu etichetat. Când setul ajunge la câteva mii de imagini pe speciile frecvente, se poate antrena un clasificator propriu, mult mai mic și mai ieftin, specializat pe speciile din Timișoara. Aceasta e o etapă pentru anul doi, nu pentru lansare.

**Limită de reținut:** identificarea funcționează sezonier. Cu frunză, e ușor. Iarna, pe scoarță și ramuri, e mult mai greu și pentru model și pentru om. De aici decizia ca sezonul principal de cartografiere să fie din primăvară până toamna.

### Modul de învățare

Ideea propusă în întâlnire: utilizatorul spune întâi ce crede, apoi vede ce spune modelul. După trei corecții pe aceeași specie, începe să o rețină. Acest mod este opțional și se poate dezactiva, pentru că cineva care știe deja nu vrea să fie testat de fiecare dată.

Scopul nu este ca toată lumea să devină expertă în specii. Dacă se întâmplă, e un câștig secundar. Scopul rămâne baza de date vie.

### Speciile invazive

Funcționalitate cerută explicit: marcarea și vizualizarea pe hartă a speciilor invazive, cu explicația de ce sunt o problemă. Oțetarul fals (cenușar), paulownia și salcâmul în context urban cresc foarte repede, se înmulțesc din rădăcină după tăiere și înlocuiesc vegetația locală. Un strat de hartă cu aceste specii este un rezultat concret și comunicabil al cartografierii, independent de orice altă utilizare a datelor.

## Măsurători pe teren

Acestea sunt informațiile care lipsesc din evidențele existente și care dau valoare stratului nostru. Toate sunt opționale, dar fiecare aduce puncte în modul joc, pentru că sunt scumpe ca efort.

| Măsurătoare | Cum se obține fără instrumente | Precizie realistă | Notă |
| --- | --- | --- | --- |
| Circumferința trunchiului la 1,3 m | Bandă de croitorie sau ață măsurată ulterior; estimare prin comparație cu mâna sau cu o foaie A4 | ±10% estimat, ±2% măsurat | Standardul dendrologic e la 1,3 m, nu la bază |
| Diametrul trunchiului | Derivat din circumferință, nu introdus separat | Derivat | Evită erorile de estimare vizuală |
| Diametrul coroanei | Pași măsurați pe sol, de la margine la margine | ±15% | Se poate valida din imagine satelitară |
| Înălțimea | Estimare vizuală cu referință (bloc, stâlp, om) | ±30% | Cea mai nesigură; marcată ca estimare |
| Vârsta | Nu se măsoară pe teren | Estimare | Vezi mai jos |

**De ce circumferința și nu diametrul.** Omul poate înconjura trunchiul cu o ață; nu poate măsura un diametru. Aplicația cere circumferința și calculează restul. Câmpul se introduce în centimetri, cu tastatură numerică.

**Despre vârstă.** Vârsta exactă se poate obține doar prin numărarea inelelor de creștere, ceea ce presupune o probă extrasă cu burghiu din trunchi, la aproximativ 1,3 m. Nu facem asta pe copacii din oraș și nici nu avem dreptul. Ce putem face: o estimare statistică, pornind de la specie și circumferință, calibrată pe câteva zeci de cazuri unde vârsta e cunoscută (plantări documentate). Rezultatul se afișează clar ca interval estimat, niciodată ca cifră exactă.

**Starea sanitară.** Câmp simplu, cu patru valori și o poză: sănătos, stres vizibil (frunze uscate, coroană rară), uscat parțial, uscat. Plus semnalări punctuale: ramuri rupte, scorbură, răni pe trunchi, dăunători vizibili, lipsă spațiu la bază (asfaltat până în trunchi). Diagnosticul real de dăunători depășește ce poate face un voluntar și rămâne pentru un arborist.

## Check-in și prospețimea datelor

Check-in-ul este mecanismul care transformă o cartografiere unică într-o bază de date vie. Fără el, peste trei ani datele sunt la fel de învechite ca orice altă inventariere.

### Cum funcționează

Utilizatorul ajunge lângă un copac, deschide aplicația, aplicația propune copacul cel mai apropiat, el confirmă. Apoi răspunde la două sau trei întrebări scurte, de tip buton, nu text liber:

- Cum arată? (sănătos / cam uscat / uscat / nu mai există)
- Are nevoie de apă? (da / nu / nu știu)
- Ceva de semnalat? (ramuri rupte / gunoi la bază / vandalism / nimic)

Opțional: o fotografie, care se adaugă la seria temporală a copacului.

### Frecvența

Un check-in pe copac pe săptămână, per utilizator, pentru punctaj. Mai des nu aduce informație nouă (un copac nu se schimbă în două ore), mai rar lasă intervale prea mari vara, în secetă. Alți utilizatori pot face check-in la același copac în aceeași săptămână; limita e pe pereche utilizator plus copac.

### Prospețimea ca proprietate vizibilă

Fiecare copac are o vârstă a ultimei observații, afișată pe hartă prin transparență sau culoare:

| Vechime ultima observație | Stare | Efect |
| --- | --- | --- |
| sub 3 luni | proaspăt | afișat normal |
| 3 până la 12 luni | de reconfirmat | marcat discret |
| peste 12 luni | învechit | marcat vizibil, punctaj dublu la revizitare |
| peste 36 de luni | nesigur | exclus din statisticile publice până la reconfirmare |

Punctajul mai mare pe copacii învechiți împinge utilizatorii exact acolo unde baza de date are nevoie de ei. Aceasta este cea mai elegantă suprapunere între joc și calitatea datelor din tot sistemul.

### Istoricul copacului

Pagina publică a unui copac arată linia lui de timp: data introducerii, fiecare observație, fotografiile în ordine cronologică, schimbările de stare. Pentru un copac adoptat și fotografiat lunar, după doi ani ai o secvență vizuală a creșterii lui, care este în sine un conținut de comunicare.

## Acuratețe, arbitraj și antifraudă

Problema formulată în întâlnire: dacă lăsăm liber, fiecare cetățean va spune orice despre fiecare copac. Dacă restrângem, nimeni nu mai contribuie. Soluția nu e controlul la intrare, ci **consensul la ieșire**.

### Regula de consens

O valoare (specia, în primul rând) devine „confirmată" când există cel puțin trei observații independente și cel puțin 50% + 1 dintre ele sunt de acord. Ținta ideală, pentru datele care contează, sunt cinci surse cu trei în acord.

Stări posibile ale unei valori:

| Stare | Condiție | Afișare |
| --- | --- | --- |
| Nedeterminată | zero observații | câmp gol, apare în coada de completat |
| Provizorie | o observație | valoarea, cu semn de întrebare |
| Confirmată | 3 sau mai multe observații, majoritate clară | valoarea, fără semn |
| În conflict | observații împărțite fără majoritate | semn de avertizare, prioritate maximă la vizitare |
| Validată expert | confirmată de un utilizator cu rol de expert | valoarea, cu marcaj de validare |

### Ponderea observațiilor

Nu toate vocile cântăresc la fel. Ponderea unui utilizator crește când observațiile lui sunt confirmate ulterior de alții și scade când sunt infirmate sistematic. Un expert validat intră cu pondere mare din start. Ponderea nu este publică și nu se afișează ca scor, ca să nu devină un clasament paralel.

### Arbitrajul

Când o valoare intră în conflict, copacul respectiv apare marcat pe harta modului joc ca „are nevoie de o părere" și aduce punctaj dublu. Astfel, mecanismul de corecție devine el însuși o sursă de joc, nu o corvoadă administrativă.

Dacă după cinci observații conflictul persistă, intrarea merge într-o coadă de moderare umană, rezolvată de un expert sau de un administrator.

### Puncte pentru corecții

Se acordă puncte și celui care corectează o greșeală, nu doar celui care adaugă primul. Altfel nimeni nu are motiv să fie al doilea sau al treilea pe același copac, iar consensul nu se formează niciodată.

### Antifraudă

Oamenii vor încerca să câștige concursul. Contramăsuri, în ordinea importanței:

1. **Verificarea momentului capturii.** Poza făcută din aplicație are marcaj temporal propriu. Pozele vechi din galerie sunt acceptate doar în modul serios și nu aduc puncte.
2. **Verificarea poziției.** Punctele se acordă doar dacă utilizatorul este efectiv în apropierea copacului în momentul contribuției.
3. **Hash perceptual pe fotografii.** Aceeași poză trimisă la copaci diferiți este detectată automat.
4. **Limite de ritm.** Un număr maxim de copaci adăugați pe oră, calibrat pe ce e fizic posibil mergând pe jos.
5. **Consensul ca filtru final.** Un utilizator care introduce constant date infirmate de alții își pierde ponderea, apoi punctele, apoi accesul la clasament.

Fraudele nu se previn complet și nu merită efort disproporționat. Ținta e ca trișatul să fie mai obositor decât mapatul corect.

## Gamificare

Referința discutată în întâlnire este Pokémon Go: un joc care a scos efectiv oamenii din casă ca să colecteze lucruri care nu există. Noi avem avantajul că obiectele de colectat sunt reale și că efectul secundar e util.

### Punctaj

Punctele reflectă efortul și valoarea pentru baza de date, nu doar volumul.

| Acțiune | Puncte orientative | Motiv |
| --- | --- | --- |
| Copac nou cu poziție | 10 | acțiunea de bază |
| Fotografie la frunză | 5 | permite identificarea |
| Specie confirmată de alții | 15 | valoare validată, acordate retroactiv |
| Circumferință măsurată | 10 | efort fizic real |
| Check-in pe copac proaspăt | 2 | menține obiceiul |
| Check-in pe copac învechit (peste 12 luni) | 8 | direcționează efortul |
| Arbitraj pe un copac în conflict | 12 | rezolvă problema cea mai scumpă |
| Semnalare specie invazivă | 10 | produce un strat util |

Valorile se calibrează după primele două săptămâni de teren, pe date reale, nu acum.

### Echipe și teritoriu

Patru sau cinci echipe pe oraș, cu culoare proprie. O zonă (cartier, parc) este „a echipei" care are cele mai multe contribuții recente acolo. Influența se pierde dacă datele se învechesc, deci echipele sunt obligate să revină, nu doar să cucerească.

În faza pilot, echipele se formează pe cartiere sau pe facultăți, iar competiția durează o campanie (două până la patru săptămâni), nu la nesfârșit.

### Niveluri

Praguri simple, pe număr de copaci contribuiți și pe acuratețe: la 100 de arbori mapați se trece la nivelul următor, care deblochează câmpuri mai avansate (măsurători, arbitraj) și dreptul de a valida contribuțiile altora. Nivelul funcționează deci ca instruire progresivă, nu doar ca decorație.

### Limitele gamificării

Au fost ridicate explicit în întâlnire și sunt reale:

- O parte din publicul potrivit nu vrea să se joace. Profesorul, biologul, voluntarul serios. Pentru ei există modul fără scor, cu aceleași date.
- Jocul atrage trișori. Vezi secțiunea de antifraudă.
- Jocul atrage volum, nu calitate. De aceea punctajul e legat de confirmarea ulterioară, nu de simpla introducere.
- Jocul funcționează doar cu densitate. Într-un cartier cu 300 de copaci mapați de 10 oameni, merge. Pe oraș, cu 20 de oameni, nu se simte nimic. De aici regula de a începe mic.

## Adopția copacilor

Adopția este mecanismul prin care un copac capătă un om responsabil. Este și originea întregului proiect: platforma veche s-a născut din ideea de a adopta un copac, a-i da nume și a-l putea găsi pe hartă.

### Ce înseamnă adopția în noua aplicație

Un utilizator își asumă unul până la trei copaci. Asumarea nu costă bani și nu dă niciun drept asupra copacului, care rămâne în proprietate publică. Ce primește:

- Copacii lui, separat, în aplicație.
- Memento săptămânal în sezonul cald: „ai trecut pe la copacul tău?"
- Istoricul lui vizual, poză după poză.
- O plăcuță virtuală, cu numele sau pseudonimul lui, vizibilă pe pagina publică a copacului.

Ce se așteaptă de la el: un check-in pe săptămână în sezonul de vegetație și o fotografie ocazională. Dacă nu mai face nimic câteva luni, copacul se eliberează discret, fără penalizare și fără mesaje de rușinare.

### Ce nu promitem

Nu punem plăcuță fizică cu numele cuiva pe un copac din domeniul public. Nu încasăm bani pentru adopție în această fază. Nu promitem că administrația va ține cont de informațiile raportate.

### Codul QR

Pentru copacii adoptați, se poate genera un cod QR care deschide pagina publică a copacului în orice browser, fără aplicație instalată. Este util la evenimente și pe copacii din parcuri, unde trecătorii pot afla ce specie e și cine are grijă de el. Montarea fizică presupune acordul administratorului spațiului, deci rămâne o opțiune pentru mai târziu.

### Legătura cu platforma veche

Platforma de adopție existentă conține un import al arborilor din evidența publică, plus structura de adopție. Decizia: **nu migrăm aplicația, migrăm datele utile.** Concret:

1. Se exportă adopțiile existente și se păstrează legătura om plus copac, dacă oamenii confirmă.
2. Arborii importați din evidența publică se trec în noua bază ca înregistrări cu sursa „import", fără a fi considerați confirmați. Ei devin schelet de pornire, nu adevăr.
3. Codul vechi rămâne arhivat, nu se dezvoltă mai departe.

## Arhitectură și backend

&#91;embedded content: arhitectura platformei · 3 clienți, 1 API, 3 servicii externe\]

Toți clienții trec prin același API. Calculele scumpe (consensul, poziția consolidată, recunoașterea speciilor, curățarea fotografiilor) nu se fac în momentul în care utilizatorul apasă butonul, ci în fundal, pentru că pe teren omul trebuie să primească confirmare instantanee.

### Alegeri tehnice propuse

| Componentă | Propunere | Motiv |
| --- | --- | --- |
| Client | aplicație web instalabilă, cu stocare locală | iterație rapidă, fără aprobare în store |
| Hartă | bibliotecă open source de hărți, cu tile-uri de la un furnizor extern | costul pe mia de afișări contează la scară |
| API | REST, documentat, versionat | simplu de consumat de oricine, inclusiv din exterior |
| Bază de date | PostgreSQL cu PostGIS | interogări spațiale (cei mai apropiați copaci, poligoane de zonă) fără efort suplimentar |
| Fotografii | stocare de obiecte, redimensionare la încărcare | fotografiile sunt cel mai mare cost de stocare |
| Procesare | cozi de job-uri | consens, scor, identificare specii, curățare EXIF |

### Lucrul offline

Obligatoriu, nu opțional. În parc și între blocuri semnalul cade. Clientul scrie local, cu identificatori generați pe dispozitiv, și sincronizează când revine rețeaua. Conflictele se rezolvă simplu, pentru că modelul de date e aditiv: două observații despre același copac nu se contrazic tehnic, ci intră amândouă și se arbitrează ulterior.

### Scară estimată

Pentru un oraș de mărimea Timișoarei, ordinul de mărime este de aproximativ 150.000 până la 200.000 de copaci în spațiul public, câteva observații pe copac și câteva fotografii pe copac. Volumul de date este mic pentru infrastructura modernă; costul real este stocarea fotografiilor și apelurile către modelul de specii. Nimic din arhitectura de mai sus nu are nevoie de scalare specială în primii doi ani.

## Interoperabilitate cu datele publice

Poziția noastră, stabilită în întâlnire: construim stratul nostru și mergem înainte. Dacă datele vor fi preluate, bine. Dacă nu, tot au valoare. Dar proiectăm de la început astfel încât preluarea să fie posibilă fără refacere.

### Compatibilitate de format

Evidențele publice folosesc nomenclatoare și indici proprii, cu formule de calcul pe care nu le putem reproduce și nici nu are rost să le reproducem. Ce putem face:

1. **Mapare de specii.** Fiecare specie din taxonomia noastră primește, unde există, codul corespondent din nomenclatorul public. Maparea se face o dată, manual, pentru cele 30 până la 50 de specii frecvente.
2. **Export structurat.** Un export standard (GeoJSON și CSV) cu coordonate, specie, măsurători, dată, număr de confirmări și nivel de încredere pentru fiecare valoare. Nivelul de încredere e esențial: arată cine poate folosi datele și pentru ce.
3. **Fără pretenția de indici proprii.** Nu calculăm și nu publicăm indicii din metodologia administrativă. Dăm materia primă, nu interpretarea ei.

### Import

Datele publice disponibile se importă ca strat de pornire, marcat distinct ca sursă externă și neconfirmat. Pe hartă, un copac din import arată altfel decât unul verificat pe teren. Prima vizită a unui voluntar îl confirmă sau îl infirmă (tăiat, inexistent, altă specie), ceea ce e în sine o contribuție valoroasă.

### API-uri deschise

Există infrastructură publică de API-uri la nivel local, care se dezvoltă. Când un API oficial pentru spațiile verzi devine disponibil, ne conectăm la el. Până atunci lucrăm în paralel, fără să blocăm nimic în așteptarea lui.

În sens invers, publicăm și noi un API public read only, documentat, cu aceleași date pe care le vede oricine pe hartă. Oricine, inclusiv administrația, le poate consuma fără să ne ceară voie și fără să semneze nimic. Aceasta e cea mai simplă formă de colaborare și singura care nu depinde de proceduri.

### Riscul de registru paralel

Este real și a fost discutat. Un strat alternativ poate fi perceput ca o critică implicită. Reducem riscul prin trei lucruri: limbaj care nu compară și nu acuză, marcarea clară a datelor noastre ca estimări cetățenești, nu ca evidență oficială, și oferirea datelor fără condiții.

## Date personale, moderare și aspecte legale

### Minimizarea datelor

Aplicația funcționează cu pseudonim. Email-ul e opțional și cerut doar pentru recuperarea contului sau pentru notificări. Nu cerem nume real, adresă, vârstă sau telefon. Un utilizator care vrea doar să mapeze poate face asta fără cont, cu o identitate locală pe dispozitiv, convertibilă în cont mai târziu.

### Fotografii

Riscul principal este surprinderea accidentală a persoanelor, a numerelor de înmatriculare și a interioarelor de proprietăți private. Măsuri:

- Instrucțiune clară în interfață: fotografiem copacul, nu strada.
- Fețele și plăcuțele de înmatriculare se estompează automat la încărcare, dacă putem integra o soluție suficient de ieftină; altfel, raportare și ștergere la cerere.
- Metadatele EXIF se curăță la publicare, cu excepția coordonatelor și a momentului capturii, care rămân în sistem dar nu se expun public.
- Buton de raportare pe fiecare fotografie publică.

### Curți private

Copacii din curți private nu intră în harta publică și nu contează la statistici. Un proprietar poate să își mapeze propriii copaci, vizibil doar pentru el. Motivul e dublu: protecția vieții private și faptul că oricum nu putem verifica acele date.

### Moderare

Trei niveluri: automat (duplicate, ritm, hash de imagine), comunitar (raportare și arbitraj) și uman (un moderator GreenTwin care tratează coada de cazuri). Volumul estimat în faza pilot este mic și gestionabil de o singură persoană, câteva minute pe zi.

### Licență și drepturi asupra datelor

Propunere de discutat, nu decizie luată:

- **Codul:** licență permisivă (MIT sau Apache 2.0), ca oricine să poată prelua proiectul în alt oraș.
- **Datele:** licență deschisă cu atribuire (ODbL sau CC BY), astfel încât datele să rămână deschise și la reutilizare.
- **Fotografiile:** rămân ale autorului, cu drept de utilizare acordat platformei; contribuitorul acceptă acest lucru la încărcare.

### GDPR

Asociația este operator de date. Necesare înainte de lansarea publică: politică de confidențialitate, temei legal pentru prelucrare (consimțământ pentru cont, interes legitim pentru datele de mediu), procedură de ștergere la cerere și un registru al prelucrărilor. Ștergerea unui cont păstrează observațiile, anonimizate, pentru că altfel baza de date s-ar destrăma retroactiv. Acest lucru se spune explicit la înregistrare.

## Organizarea echipei și procesul de lucru

### Cine lucrează

| Rol | Cine | Observație |
| --- | --- | --- |
| Coordonare produs și specificații | Bob | extrage specificațiile din întâlnire, le trimite grupului |
| Development | Bob plus doi voluntari interesați | capacitatea reală se clarifică după primul sprint |
| Expertiză dendrologică | profesorul de la geografie | specii, inele de creștere, validare metodologică |
| Design | designer colaborator | revine din noiembrie, până atunci mergem pe o interfață simplă |
| Marketing și comunicare | echipă de contactat | se activează când există ceva de arătat |

### Repository și contribuții

Proiect public pe GitHub, de la prima linie de cod. Reguli minime, ca să nu descurajăm voluntarii:

- Branch principal protejat, orice modificare prin pull request.
- Un review înainte de merge, oricât de scurt.
- Issues etichetate cu „bun pentru început", ca cineva nou să aibă unde intra.
- README cu instrucțiuni de rulare locală care funcționează din prima.
- Fără obligativitate de convenții stricte de commit în faza inițială. Important e că lumea contribuie.

### Comunicare

Grup de WhatsApp pentru coordonare zilnică, pentru că acolo sunt deja voluntarii. Issues pe GitHub pentru lucrurile care trebuie urmărite. Fără alt instrument adăugat acum.

### Hosting și costuri

Costurile reale în faza pilot sunt mici: un server mic, stocare pentru fotografii, un domeniu, plus costul apelurilor către modelul de recunoaștere a speciilor. Ordinul de mărime estimat este de zeci de euro pe lună, nu sute, până la câteva mii de utilizatori.

Mai multe platforme oferă credite sau licențe gratuite pentru ONG-uri (găzduire, servicii cloud, modele AI). Merită depuse cereri, dar nu construim planul tehnic pe presupunerea că vom fi acceptați. Infrastructura se alege astfel încât mutarea de la un furnizor la altul să fie simplă.

**De reținut din experiența de până acum:** dezvoltarea de software nu se finanțează prin programele de mediu. Se finanțează campania fizică, promovarea, logistica, materialele pentru voluntari. Software-ul se face pe voluntariat sau din alte surse. Asta e și motivul pentru care open source nu e doar un principiu, ci singura formulă sustenabilă.

## Plan de livrare

Calendarul e dictat de sezonul de vegetație. Cu frunză, identificarea speciilor funcționează; iarna, nu. Deci nu are sens să forțăm o campanie de teren înainte de primăvară, dar avem toată iarna pentru construit.

&#91;embedded content: plan de livrare · 5 faze și un eveniment, octombrie 2026 până în aprilie 2027\]

### Ce înseamnă fiecare fază

**Faza 0, specificații și acord (două săptămâni).** Documentul acesta, circulat în grup, cu răspunsuri la întrebările deschise. Repository creat, mediu de dezvoltare care rulează local din prima. *Criteriu de ieșire:* trei oameni știu ce construiesc și cine ce face.

**Faza 1, MVP web (șapte săptămâni).** Hartă, adăugare de copac cu poziție ajustabilă, fotografie, specie din listă, cont minimal, pagină publică de copac. Fără joc, fără AI, fără arbitraj. *Criteriu de ieșire:* un voluntar poate adăuga un copac de pe stradă, de pe telefon, fără ajutor.

**Faza 2, test intern (cinci săptămâni).** Cei trei plus doi sau trei voluntari mapează 50 de copaci reali, iarna, exact ca să găsim problemele de folosire în condiții proaste. Se calibrează câmpurile și fluxul. *Criteriu de ieșire:* lista de probleme reale, nu presupuse.

**Faza 3, mobil, joc și identificare (opt săptămâni).** Aplicație instalabilă cu lucru offline, punctaj, echipe, identificare de specii prin fotografie, mecanismul de consens și arbitraj. *Criteriu de ieșire:* cineva din afara grupului o folosește fără instruire.

**Faza 4, cartier pilot (șase săptămâni).** Campanie reală într-un cartier sau parc, cu voluntari și cu studenți, cu eveniment fizic la mijloc. Aici aflăm ce costă o campanie și ce buget cere scalarea la tot orașul. *Criteriu de ieșire:* o zonă cartografiată complet și un set de date pe care îl putem arăta oricui.

### Ce nu intră în acest plan

Adopția completă cu notificări, codurile QR fizice, modelul propriu de recunoaștere, exportul către sisteme externe și orice discuție instituțională. Toate vin după ce există date dintr-un cartier.

## Întrebări deschise

Acestea nu au fost decise în întâlnire și blochează sau modifică părți din specificații. Fiecare are nevoie de un răspuns înainte de faza în care contează.

1. **Cross platform sau nativ, și când.** PWA acoperă prima fază, dar precizia locației și accesul la senzori sunt mai bune nativ. Decizia poate aștepta până după pilot, dacă arhitectura separă clar interfața de API.
2. **Cine validează speciile la scară.** Un profesor nu poate valida 10.000 de copaci. Avem nevoie de 10 până la 20 de oameni cu competență rezonabilă. Întrebare deschisă din întâlnire: există acești oameni, vor să facă asta, și cât durează să îi instruim?
3. **Cât de mult joc.** Riscul de a aliena publicul serios există. Propunerea celor două moduri rezolvă parțial, dar punctajul, echipele și clasamentul mai trebuie calibrate pe teren.
4. **Ce se întâmplă cu copacii din evidența importată.** Îi arătăm de la început ca schelet, sau pornim de la hartă goală și lăsăm voluntarii să construiască? Harta goală motivează mai mult; scheletul e mai util.
5. **Finanțarea campaniei fizice.** Apelul avut în vedere pentru octombrie nu mai e disponibil. Trebuie identificate alte direcții pentru promovare, logistică și materiale pentru voluntari. Scopul inițial era oricum să aflăm cât costă o campanie pe cartier, ca să știm ce buget cere scalarea la tot orașul.
6. **Pragul de date la care mergem spre instituții.** Un cartier complet? Trei? Nu are sens să mergem cu 50 de copaci, și nu are sens să așteptăm tot orașul.
7. **Cine ține proiectul viu după pilot.** Dacă aplicația depinde de o singură persoană, se oprește când acea persoană e ocupată. Deschiderea codului ajută, dar nu rezolvă.

---

*Document generat din transcrierea întâlnirii din 8 octombrie 2026 (70 de minute). Deciziile marcate ca luate provin din discuție; propunerile tehnice detaliate (praguri, punctaje, câmpuri, licențe) sunt propuneri de lucru, de confirmat de echipă.*
