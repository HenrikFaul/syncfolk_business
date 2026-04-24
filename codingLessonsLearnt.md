# codingLessonsLearnt.md — Kapakka PubApp

## ⚠️ UTASÍTÁSOK (MINDIG OLVASD EL ELŐSZÖR!)

**KÖTELEZŐ MUNKAFOLYAMAT — Minden fejlesztés előtt:**
1. Nyisd meg és olvasd végig ezt a fájlt MIELŐTT bármit kódolnál
2. Ellenőrizd, hogy az új kódod nem tartalmaz-e az itt felsorolt hibamintákat
3. Ha új hibát találsz/javítasz, AZONNAL appendeld a megfelelő kategóriába
4. SOHA ne töröld a meglévő tartalmat — csak hozzáadni szabad
5. SOHA ne hozz létre új fájlt ezzel a céllal — mindig ebbe a fájlba írd

**Struktúra minden hiba bejegyzésnél:**
```
### [HIBA-XXX] Rövid cím
- **Dátum**: Mikor fordult elő
- **Fájl**: Melyik fájlban volt
- **Hibaüzenet**: Pontos TypeScript/build error
- **Gyökérok**: Miért történt
- **Javítás**: Hogyan lett megoldva
- **Megelőzés**: Hogyan kerüld el a jövőben
```

---

## 🔴 KATEGÓRIA 1: TypeScript típus hibák

### [HIBA-001] Hiányzó property az interface-ből
- **Dátum**: 2026-03-30 (v1.1.0)
- **Fájl**: `src/app/admin/menu/templates/page.tsx:157`
- **Hibaüzenet**: `Type error: Property 'item_sort' does not exist on type 'TemplateItem'.`
- **Gyökérok**: A `TemplateItem` interface-ben nem volt definiálva az `item_sort` property, miközben a kód hivatkozott rá (`sort_order: item.item_sort`). Az interface-t kézzel írtam, és kifelejtettem egy mezőt amit az SQL tábla tartalmaz.
- **Javítás**: Hozzáadtam `item_sort: number` a `TemplateItem` interface-hez.
- **Megelőzés**: **MINDIG** hasonlítsd össze az interface mezőket az SQL tábla oszlopaival. Ha az SQL-ben van `item_sort`, az interface-ben is KELL lennie. Checklist: minden SQL oszlop = egy interface property.

### [HIBA-002] Supabase FK reláció típusozás — `.table.number` hiba
- **Dátum**: 2026-03-30 (v1.2.0)
- **Fájl**: `src/app/admin/reports/page.tsx:61`
- **Hibaüzenet**: `Type error: Property 'number' does not exist on type '{ number: any; }[]'.`
- **Gyökérok**: Supabase `.select('table:tables(number)')` esetén a TypeScript a relációt **tömbként** (`{ number: any }[]`) típusozza, nem objektumként. Ezért `o.table.number` helyett `o.table[0].number` kellene, de valójában futásidőben objektumot ad vissza (nem tömböt).
- **Javítás**: A `.map()` callback-ben `(o: any)` típust használtam: `.map((o: any) => [...])` — ez megkerüli a Supabase TS típus problémát.
- **Megelőzés**: **MINDIG** használj `(item: any)` cast-ot amikor Supabase `.select()` eredményt iterálsz és FK relációkat (`table:tables(...)`, `venue:venues(...)`, `menu_item:menu_items(...)`) használsz. VAGY használj `useState<any[]>([])` a state-hez. A kettő közül az egyik KÖTELEZŐ.

### [HIBA-003] Supabase FK — új oszlopok nem ismertek a TS típusokban
- **Dátum**: 2026-03-30 (v1.2.0)
- **Fájl**: `src/app/admin/reports/page.tsx:137`
- **Hibaüzenet**: Potenciális — `total_orders`, `total_spent` nem létezik a `profiles` Supabase típusban
- **Gyökérok**: Ha ALTER TABLE-lel új oszlopot adsz hozzá (`total_orders`, `total_spent`), a Supabase TS generált típusok nem frissülnek automatikusan. A `.select()` eredmény típusa nem tartalmazza az új mezőket.
- **Javítás**: `(c: any)` cast a `.map()` callback-ben.
- **Megelőzés**: Ha SQL migrációval új oszlopokat adsz egy meglévő táblához, az adott tábla select eredményeit MINDIG `(row: any)` casttal kezeld, amíg a típusok nem lesznek újragenerálva (`supabase gen types`).

### [HIBA-018] Implicit `any` a chained `.map().filter()` callbackben
- **Dátum**: 2026-03-31 (v1.3.1)
- **Fájl**: `src/lib/place-search.ts:98`
- **Hibaüzenet**: `Type error: Parameter 'row' implicitly has an 'any' type.`
- **Gyökérok**: A `rows` tömb `any[]` típusú volt, és a `rows.map(...).filter((row) => row.external_id)` láncban a `filter` callback paramétere nem kapott explicit típust. `noImplicitAny` mellett ez build hibát okozott.
- **Javítás**: A nyers API választ `const rows: any[]` formában explicitáltam, külön `normalizedRows: ExternalPlace[]` tömbbe mapeltem, majd a filter callbacket `ExternalPlace` típussal adtam meg.
- **Megelőzés**: Ha `any[]` tömbből több lépéses `.map().filter().reduce()` lánc készül, a köztes eredményt MINDIG nevezd el és adj neki explicit típust. A végső callback paramétereknél ne hagyatkozz implicit inference-re `strict` TypeScript beállítás mellett.

---

## 🟡 KATEGÓRIA 2: SQL / RLS / Adatbázis hibák

### [HIBA-004] SQL szintaxis hiba — RLS policy zárójelezés
- **Dátum**: 2026-03-29 (v1.0.0)
- **Fájl**: `supabase/migrations/001_initial_schema.sql:47`
- **Hibaüzenet**: `syntax error at or near "or" LINE 47: ) or is_active = true;`
- **Gyökérok**: Az RLS policy USING() zárójelén kívül volt egy `or is_active = true` feltétel. A helyes szintaxis: `USING ((feltétel1) OR (feltétel2))` — minden feltétel a USING() BELSEJÉBE kerül.
- **Javítás**: Az egész policy-t újraírtam helyes zárójelezéssel.
- **Megelőzés**: RLS policy írásakor MINDIG ellenőrizd, hogy MINDEN feltétel a `USING(...)` zárójelen BELÜL van. Soha ne legyen logikai operátor a zárójelen kívül.

### [HIBA-005] RLS policy circular dependency — profil olvasás blokkolva
- **Dátum**: 2026-03-29 (v1.0.1)
- **Fájl**: Profiles RLS policies
- **Hibaüzenet**: Profil lekérdezés sikertelen admin felhasználóknál
- **Gyökérok**: A profiles SELECT policy JOIN-t tartalmazott a `venues` táblára, ami maga is RLS-sel volt védve. Ha a venues policy is hivatkozott a profiles-ra → circular dependency. Az admin felhasználó nem tudta olvasni a saját profilját.
- **Javítás**: Egyszerű policy: `CREATE POLICY "profiles_select_authenticated" ON public.profiles FOR SELECT USING (auth.uid() IS NOT NULL);` — minden bejelentkezett felhasználó olvashat minden profilt.
- **Megelőzés**: **SOHA** ne legyen RLS SELECT policy-ban JOIN más RLS-védett táblára. Ha kell cross-table check, használj egyszerű `auth.uid()` alapú feltételt, vagy SECURITY DEFINER funkciót.

### [HIBA-006] Profil email NULL — role update 0 rows
- **Dátum**: 2026-03-29 (v1.0.1)
- **Hibaüzenet**: `UPDATE public.profiles SET role = 'admin' WHERE email = 'x@y.com'` → 0 rows affected
- **Gyökérok**: A `handle_new_user()` trigger nem másolta át az email-t az `auth.users` táblából a `profiles` táblába. A `profiles.email` mező NULL volt, ezért a WHERE feltétel nem talált sort.
- **Javítás**: JOIN-os UPDATE: `UPDATE profiles p SET role = 'admin' FROM auth.users u WHERE p.id = u.id AND u.email = 'x@y.com';`
- **Megelőzés**: A `handle_new_user()` trigger MINDIG másolja át az email-t: `NEW.raw_user_meta_data->>'email'` VAGY `(SELECT email FROM auth.users WHERE id = NEW.id)`. Soha ne feltételezd, hogy a profiles.email ki van töltve.

### [HIBA-007] Supabase FK constraint név — törékeny hivatkozás
- **Dátum**: 2026-03-30 (v1.1.0)
- **Fájl**: `src/app/siteadmin/venues/page.tsx`
- **Hibaüzenet**: Potenciális — `profiles!venues_owner_id_fkey` nem létezik
- **Gyökérok**: `.select('*, owner:profiles!venues_owner_id_fkey(full_name, email)')` — a constraint név adatbázisonként eltérhet. A Supabase automatikusan generálja a FK constraint nevet, és nem garantált, hogy mindig `venues_owner_id_fkey`.
- **Javítás**: Lecseréltem `.select('*, owner:profiles(full_name, email)')` — constraint név nélkül, a Supabase automatikusan feloldja.
- **Megelőzés**: **SOHA** ne használj explicit FK constraint nevet a `.select()` relációkban. Használd a szimpla `table_name(columns)` szintaxist. Ha ambiguous, használd a `table_name!column_name(columns)` formátumot (oszlop nevet, NEM constraint nevet).

---

## 🟠 KATEGÓRIA 3: Auth / Redirect / Session hibák

### [HIBA-008] Auth redirect loop — 4 helyen konkurens redirect
- **Dátum**: 2026-03-29 (v1.0.0 → v1.0.1)
- **Fájl**: middleware.ts + page.tsx + customer/page.tsx + admin/layout.tsx
- **Hibaüzenet**: Végtelen loading screen — az alkalmazás sosem jutott túl az „Átirányítás..." képernyőn
- **Gyökérok**: 4 különböző helyen volt routing logika, és egymásba irányítottak: middleware → /admin → admin/layout ellenőrzi → /customer → customer/page ellenőrzi → /admin → ∞ loop
- **Javítás**:
  1. `middleware.ts` — CSAK cookie frissítés, NULLA redirect
  2. `page.tsx` — Egyetlen auth check 4s timeout-tal, `hasRedirected` ref a dupla redirect ellen
  3. `customer/page.tsx` — Admin felhasználóknak "Admin panel megnyitása" GOMB, nem redirect
  4. `admin/layout.tsx` — Nem-admin felhasználóknak error screen, nem redirect
- **Megelőzés**: **EGY SZABÁLY**: Routing döntés KIZÁRÓLAG client-side, egyetlen helyen. Middleware SOHA ne redirecteljen. Ha jogosultsági hiba van, mutass error screen-t, ne redirectelj másik oldalra.

### [HIBA-009] getSession() vs getUser() — elavult session
- **Dátum**: 2026-03-29
- **Gyökérok**: `getSession()` a helyi cache-ből olvas, ami elavult lehet. `getUser()` mindig a Supabase szerverhez fordul.
- **Megelőzés**: Auth ellenőrzésnél MINDIG `getUser()` a megbízható módszer, NEM `getSession()`.

### [HIBA-014] Venue JOIN a profil lekérdezésben blokkolja az auth-ot
- **Dátum**: 2026-03-30 (v1.2.0)
- **Fájl**: `src/app/admin/layout.tsx`
- **Hibaüzenet**: "Nincs hozzáférésed" — admin felhasználó nem tud belépni az admin panelre
- **Gyökérok**: A profil lekérdezés `select('*, venue:venues(*)')` formában volt, ami FK JOIN-t csinál a venues táblára. Ha a `profiles.venue_id` NULL (nincs venue hozzárendelve), VAGY ha nincs explicit FK constraint a DB-ben, VAGY ha az RLS policy blokkolja a venues lekérést, az EGÉSZ lekérdezés hibával tér vissza (`profileError` != null). Emiatt a kód a `no-permission` ágra futott, pedig a felhasználó valójában admin role-lal rendelkezett.
- **Javítás**: A profil és venue lekérdezést SZÉTVÁLASZTOTTAM:
  1. Először: `select('*')` a profiles-ból (FK JOIN nélkül) — ez az auth check
  2. Utána: külön `select('*')` a venues-ból venue_id alapján — ez már NEM blokkolja az auth-ot
- **Megelőzés**: **SOHA** ne legyen FK JOIN egy auth-kritikus lekérdezésben! Az auth ellenőrzés (profil + role check) MINDIG egyszerű, single-table query legyen. Ha kiegészítő adatok kellenek (venue, orders stb.), azokat KÜLÖN, NEM-BLOKKOLÓ lekérdezésben szerzd be MIUTÁN az auth check sikeres.

---

## 🔵 KATEGÓRIA 4: Build / Import / Kompatibilitás hibák

### [HIBA-010] Next.js fájlnév konvenció — page.tsx kötelező
- **Dátum**: 2026-03-29
- **Gyökérok**: A felhasználó a letöltött fájlokat `admin-layout.tsx` és `customer-page.tsx` néven mentette el `layout.tsx` és `page.tsx` helyett. Next.js App Router CSAK a `page.tsx`, `layout.tsx`, `loading.tsx` stb. pontos neveket ismeri fel.
- **Megelőzés**: Fájlok MINDIG a pontos Next.js konvenció szerinti nevekkel készüljenek. A letöltési/mentési utasításokban MINDIG jelöld meg a cél fájlnevet.

### [HIBA-011] Lucide React ikon import — nem létező ikon név
- **Dátum**: Általános (megelőző figyelmeztetés)
- **Megelőzés**: Lucide React ikonokat MINDIG a hivatalos listáról importáld. Ha nem biztos, hogy létezik, használj olyan ikont ami biztosan megvan (pl. `Settings`, `User`, `Search`, `Plus`, `Check`, `X`). A `lucide-react@0.363.0` verzióban ezek biztosan elérhetők: Zap, ClipboardList, UtensilsCrossed, Package, BarChart3, Settings, HelpCircle, Menu, Bell, LogOut, Shield, ChevronRight, X, Monitor, CalendarClock, FileDown, Plus, Pencil, Trash2, Search, CheckCircle, XCircle, Volume2, VolumeX, Maximize, Minimize, RefreshCw, Check, Clock, AlertTriangle, Download, FileSpreadsheet, Calendar, TrendingUp, ShoppingBag, Users, Phone, Mail, User, ChevronLeft, ChevronRight, ChevronDown, ChevronUp, ArrowLeft, Sparkles, Star, MapPin, Store, ScrollText, LayoutDashboard, Activity, Info, AlertCircle, ToggleLeft, ToggleRight, Save, Filter, Bug, Send.

### [HIBA-015] Lucide React redesign patch — `House` ikon build hibát okozott
- **Dátum**: 2026-03-30 (v1.2.1)
- **Fájl**: `src/app/customer/page.tsx:15`
- **Hibaüzenet**: `Type error: "lucide-react" has no exported member named 'House'. Did you mean 'Mouse'?`
- **Gyökérok**: A redesign patch-ben olyan Lucide ikont importáltam (`House`), ami a projektben használt verzióban nem exportált. Ráadásul több más ikon is a "biztosan elérhető" listán kívül volt, ezért a patch nem követte a kötelező ikon-import szabályt.
- **Javítás**: A `House` importot `LayoutDashboard`-ra cseréltem, és a redesign patch összes új Lucide importját átnéztem. Az összes bizonytalan ikont lecseréltem a codingLessonsLearnt-ben felsorolt, biztosan elérhető ikonokra.
- **Megelőzés**: **MINDIG** ellenőrizd a redesign patch összes Lucide importját a `codingLessonsLearnt.md` [HIBA-011] pontja alapján. Új UI csomag kiadása előtt kötelező grep-pel végignézni az összes `from 'lucide-react'` importot, és csak a whitelistelt ikonok maradhatnak.



---

## 🟢 KATEGÓRIA 5: CSS / UI hibák

### [HIBA-012] Admin `.input` class hiányzik
- **Dátum**: 2026-03-30 (v1.1.0)
- **Gyökérok**: Az admin oldalak `.input` CSS class-t használnak az input mezőkhöz, de ez nem volt definiálva a globals.css-ben. A Tailwind nem generálja automatikusan.
- **Javítás**: `.input` class hozzáadása a globals.css-hez explicit CSS-ként.
- **Megelőzés**: Ha egyedi CSS class-t használsz (`.input`, `.status-badge`, `.animate-slide-up`), MINDIG ellenőrizd, hogy definiálva van-e a globals.css-ben.

### [HIBA-013] Admin sidebar mobil nézet — nem jelenik meg
- **Dátum**: 2026-03-30
- **Gyökérok**: A `display: none` `@media(max-width:768px)` felülírta a JavaScript-ből adott `translate-x-0` class-t.
- **Javítás**: CSS override: `.admin-sidebar.translate-x-0 { display: flex !important; }`
- **Megelőzés**: Ha egy elem CSS-ből `display:none`, a JS class hozzáadás NEM elég — `!important` kell a CSS-ben is.

---

### [HIBA-015] Patch-only csomagból kimaradt új supporting fájlak
- **Dátum**: 2026-03-30 (v1.2.1)
- **Fájl**: patch csomag / `src/app/layout.tsx`, `src/app/admin/config/page.tsx`
- **Hibaüzenet**: Build/import hiba, mert az újonnan hivatkozott `@/components/AppShellProviders` és `@/lib/themes` fájlok nem voltak benne a patch-only zipben.
- **Gyökérok**: A patch-only csomagolásnál nem csak a módosított meglévő fájlakat, hanem az újonnan BEVEZETETT supporting fájlokat is csomagolni kell. Ezek kimaradtak.
- **Javítás**: A patch-only csomag listáját úgy kell összeállítani, hogy minden új import célfájlja bekerüljön.
- **Megelőzés**: Patch készítés előtt **MINDIG** futtasd le ezt a checklistet: minden `import '@/...'` útvonalhoz létezik fájl ÉS a zipben is benne van, ha újonnan lett bevezetve.

### [HIBA-016] Design patch buildbiztonság — csak syntax-ellenőrzött fájl csomagolható
- **Dátum**: 2026-03-30 (v1.3.0)
- **Fájl**: összes új / módosított `.tsx` fájl
- **Hibaüzenet**: Potenciális — reszponzív redesign közben könnyű szintaktikai hibát vagy félbehagyott importot hagyni.
- **Gyökérok**: Nagy redesignnál sok fájl változik egyszerre, ezért megnő a hibázás esélye.
- **Javítás**: A patch csomagolás előtt a módosított TS/TSX fájlakat legalább TypeScript parser szinten ellenőrizni kell.
- **Megelőzés**: **MINDIG** legyen build-safety lépés: ha teljes `npm build` nem futtatható, akkor minimum parser/syntax ellenőrzést kell végezni minden módosított TS/TSX fájlra.

### [HIBA-017] Új adatbázis tábla / migráció még nincs fent — UI ne omoljon össze
- **Dátum**: 2026-03-30 (v1.3.0)
- **Fájl**: `src/app/customer/page.tsx`, `src/app/admin/config/page.tsx`, új social/place feature lekérdezések
- **Hibaüzenet**: Potenciális — ha a `place_favorites`, `friendships`, `place_lists`, `app_settings` vagy `reservations` migráció még nincs lefuttatva, a featurelekérdezések hibát dobhatnak.
- **Gyökérok**: A frontend hamarabb kerülhet fel, mint az új migráció.
- **Javítás**: A lekérdezések `maybeSingle()` / `|| []` fallback mintával készültek, és a feature nem auth-kritikus ágon fut.
- **Megelőzés**: **SOHA** ne legyen új opcionális feature táblára épített lekérdezés auth-kritikus vagy page-blocking. Új feature tábla = null-safe, fallbackes, nem-blokkoló betöltés.

## 📋 ELLENŐRZŐ LISTA (Minden commit előtt)

- [ ] Auth-kritikus lekérdezésben NINCS FK JOIN? (profiles select = egyszerű `select('*')`)
- [ ] Minden interface/type property megegyezik az SQL tábla oszlopaival?
- [ ] Supabase `.select()` FK relációk használatánál van `(row: any)` cast?
- [ ] Nincs explicit FK constraint név a Supabase select-ben?
- [ ] Nincs middleware-ben redirect?
- [ ] Auth check `getUser()`-t használ, nem `getSession()`-t?
- [ ] Fájlnevek Next.js konvenciónak megfelelnek (`page.tsx`, `layout.tsx`)?
- [ ] Egyedi CSS class-ok definiálva vannak a globals.css-ben?
- [ ] Lucide ikonok a hivatalos listáról importálva?
- [ ] Minden új import célfájlja benne van a patch-only csomagban?
- [ ] Parser/syntax ellenőrzés lefutott a módosított TS/TSX fájlakon?
- [ ] RLS policy-kban nincs cross-table JOIN más RLS-védett táblára?
- [ ] Új SQL oszlopok esetén a kód `(: any)` castot használ?

---

*Utoljára frissítve: 2026-04-11 — v2.0.0*
*Ez egy FOLYAMATOSAN BŐVÜLŐ fájl. Új hibákat MINDIG appendelj, SOHA ne törölj!*

## ➕ APPEND — 2026-03-31 build hiba kiegészítés

### [HIBA-023] Supabase Edge Function `Deno` globál — Next.js build alatti típushiba
- **Dátum**: 2026-03-31
- **Fájl**: `supabase/functions/place-search/index.ts:110`
- **Hibaüzenet**: `Type error: Cannot find name 'Deno'.`
- **Gyökérok**: A Next.js root build / TypeScript ellenőrzés belefutott a `supabase/functions/...` alatti Supabase Edge Function fájlba, ami **Deno runtime-ra** íródott (`Deno.serve(...)`). A Next/Node oldali TypeScript környezet nem ismeri automatikusan a `Deno` globált, ezért a build megállt. A probléma nem maga a business logika, hanem a runtime-keveredés: a Deno-s Edge Function ugyanabban a TypeScript ellenőrzési körben maradt, mint a Next app.
- **Javítás**: A stabil megoldás két részből áll:
  1. A Next app `tsconfig.json` fájljából ki kell zárni a `supabase/functions/**/*` útvonalat, hogy a Next build ne próbálja Node/Next környezetben típusellenőrizni a Deno Edge Functionöket.
  2. Az Edge Function saját Deno/Supabase runtime típusreferenciát kapjon, és külön Supabase / Deno folyamatban legyen ellenőrizve (például a projektben használt Supabase edge runtime type importtal).
- **Megelőzés**: **SOHA** ne hagyd a Deno runtime-ra írt Supabase Edge Function fájlokat a Next.js root typecheck hatókörében. Node/Next build és Supabase Edge Function typecheck legyen külön kezelve. Ha új Edge Function készül, azonnal ellenőrizd, hogy:
  - a `supabase/functions/**` mappa ki van-e zárva a root `tsconfig.json`-ból;
  - az adott function rendelkezik-e a szükséges Deno / Supabase edge runtime típusreferenciával;
  - az ellenőrzése külön Supabase / Deno parancsból történik-e, nem `npm run build` alatt.

## 📋 ELLENŐRZŐ LISTA — új buildbiztonsági pontok

- [ ] A `supabase/functions/**` mappa ki van zárva a Next.js root `tsconfig.json` typecheckjéből?
- [ ] A Deno runtime-os Edge Function saját runtime típusreferenciával rendelkezik?
- [ ] A Supabase Edge Function ellenőrzése külön történik a Next app buildtől?

*Appendelve: 2026-03-31 — v1.3.3*

### [HIBA-017] Supabase enum típus nem illeszkedik string filterhez
- **Dátum**: 2026-04-11 (v2.1.0)
- **Fájl**: `src/components/enterprise/ApprovalInbox.tsx:61`
- **Hibaüzenet**: `Argument of type 'string' is not assignable to parameter of type 'NonNullable<"approved" | ...>'`
- **Gyökérok**: A Supabase SDK generált típusai szűk uniót várnak, de a React state `string`-ként tárolta a filter értéket
- **Javítás**: `as any` cast alkalmazása a `.eq('status', statusFilter as any)` hívásban
- **Megelőzés**: Supabase `.eq()` hívásokban ha a filter értéke React state-ből jön és az enum típus szűk, használj explicit castot

*Appendelve: 2026-04-11 — v2.1.0*

---

## 🟢 KATEGÓRIA — Architektúra: Single Source of Truth (2026-04-22, v2.5.0)

### [LESSON-SSOT-001] Pozíció ↔ tag allokáció listázása JOIN-on keresztül
- **Dátum**: 2026-04-22
- **Fájl**: `src/components/enterprise/BusinessRoleManager.tsx`
- **Probléma**: A pozíció kártya csak 1 tagot mutatott, miközben a Member Profile %-os allokációt is támogat → adat-dissonance.
- **Gyökérok**: A korábbi lekérdezés a `business_role` mezőre szűrt a `enterprise_memberships`-en (régi 1:1 modell), figyelmen kívül hagyva az `enterprise_member_role_allocations` junction táblát.
- **Javítás**: Always read az allokációt a junction táblából, és join-old a `enterprise_memberships` + `profiles` adatokkal. UI-ban listázd MINDEN tagot %-os értékkel és számolt napi órával (`base_working_hours * pct / 100`).
- **Megelőzés**: Ha létezik junction tábla bármilyen N:M kapcsolatra, sose dolgozz a denormalizált, régi szűrőmezővel — a junction table az igazság forrása.

### [LESSON-SSOT-002] Dropdown / szűrő hardcoded listák tilosak
- **Dátum**: 2026-04-22
- **Fájl**: `src/components/enterprise/LeaveCalendar.tsx`
- **Probléma**: A csapat-szűrő hardcoded értékeket mutatott, eltérve a Settings → Teams modultól.
- **Javítás**: Minden szűrő opciólistát ugyanabból a Supabase táblából tölts (`enterprise_teams`), amit a CRUD-ot végző modul is használ.
- **Megelőzés**: Ha egy entitásnak van CRUD UI-ja, sose duplikáld statikusan máshol — egyetlen forrás (DB tábla / view) létezzen.

### [LESSON-PERMISSIONS-001] Hierarchikus jogosultság-fa katalógus táblából
- **Dátum**: 2026-04-22
- **Fájl**: `src/hooks/useEnterprisePermissions.ts`, `RolePermissionManager.tsx`
- **Probléma**: Statikus, lapos `FEATURE_GROUPS` array nehezen tartható szinkronban a tényleges navigációs fával.
- **Javítás**: `enterprise_feature_catalog` tábla bevezetése `parent_key` self-FK-val. A hook `featureTree`-t épít, az UI rekurzív komponenssel rendereli (`FeatureTreeRow`). Fallback: ha a katalógus üres, marad a régi flat lista.
- **Megelőzés**: Bármi, ami "tükrözi az alkalmazás struktúráját", legyen DB-ben tárolt fa, ne forrásban kódolt enum.

### [LESSON-CAPACITY-001] Kapacitás óra-egységben, ne csak százalékban
- **Dátum**: 2026-04-22
- **Fájl**: `src/lib/capacityEngine.ts`
- **Probléma**: Csak %-ban számolt kapacitás félrevezető részmunkaidős (4–6 órás) tagoknál.
- **Javítás**: Új `base_working_hours` mező a membership-en; minden ouputba (PositionSummary) számolj `total_available_hours = base_working_hours * (pct / 100)`-ot is.
- **Megelőzés**: Ha valós erőforrás-tervezést támogatunk, mindig legyen abszolút mértékegység (óra/nap), ne csak relatív arány.
