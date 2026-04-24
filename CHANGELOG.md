# Changelog

## 2026-04-11 — v2.4.0 Email értesítések, demo adatok, mobil tesztelés
- Email értesítés küldés jóváhagyási/elutasítási döntéseknél (transactional email infra + leave-decision sablon)
- Genisys workspace demo adatok: 8 tag (owner, resourceAssistant, 6 member), 12 szabadságkérelem (pending/approved/rejected), szabadság típusok, ünnepnapok, tiltott napok, napi szabályok, jóváhagyási lánc, escalation, értesítések, audit log események
- Leiratkozás oldal (/unsubscribe) az email értesítésekhez
- Mobil reszponzivitás tesztelve: összes Enterprise tab (Tagok, Kérelmek, Jóváhagyások, Értesítések, Riportok, Audit, Export) megfelelően jelenik meg 390px szélességen

## 2026-04-11 — v2.3.0 Enterprise Phases 4-9: Approval Chain, Audit, Notifications, Export, Reporting, Templates
- Jira: SYN-146 + remaining enterprise stories
- Új DB táblák: `enterprise_approval_chains`, `enterprise_escalation_rules`, `enterprise_audit_events`, `enterprise_notifications`, `enterprise_rule_templates`, `enterprise_export_jobs`
- Phase 4: Jóváhagyási lánc konfiguráció (lépések, szerepkörök), eszkaláció szabályok (óra küszöb, célszerepkör, owner értesítés)
- Phase 5: Audit log (immutable, szűrhető, kereshető, 100 utolsó esemény)
- Phase 6: Enterprise értesítések (olvasott/olvasatlan, törlés, emoji ikonok)
- Phase 7: CSV export (dátumtartomány, státusz szűrő, ünnepnapok, audit log bejegyzés)
- Phase 8: Reporting dashboard (KPI kártyák, státusz pie chart, típus bar chart, napi távollévők chart)
- Phase 9: Szabálysablon könyvtár (létrehozás, duplikálás, archiválás, JSON adat)
- WorkspaceDashboard: 6 új tab (Értesítések, Riportok, Audit, Export + Approval chain és Templates a Szabályok tabba)

## 2026-04-11 — v2.2.0 Enterprise Phase 3: Leave Types, Holidays, Rules & Conflict Engine
- Jira: SYN-148, SYN-147
- Funkció: Egyedi szabadság típusok, ünnepnapok, tiltott napok, napi szabályok kezelése
- Új DB táblák: `enterprise_leave_types`, `enterprise_holidays`, `enterprise_blocked_dates`, `enterprise_daily_rules`
- RLS: workspace tagok olvashatják, owner/resourceAssistant kezelhetik a konfigurációt
- Új UI komponensek: `LeaveTypeManager`, `HolidayManager`, `BlockedDateManager`, `DailyRuleManager`
- Új "Szabályok" tab a WorkspaceDashboard-ban (admin only)
- Conflict engine (`src/lib/conflictEngine.ts`): tiltott nap, ünnepnap, max-off, saját átfedés ellenőrzés
- LeaveRequestDialog: kétlépcsős submit (Ellenőrzés → Beküldés), blocking/warning ütközések megjelenítése
- Módosított fájlok: `WorkspaceDashboard.tsx`, `LeaveRequestDialog.tsx`

## 2026-04-11 — v2.1.0 Enterprise Phase 2: Leave Request & Approval Workflow
- Jira: SYN-165, SYN-150, SYN-149
- Funkció: Távolléti kérelem beküldés, jóváhagyási workflow, approval inbox
- Új DB táblák: `leave_requests`, `approval_decisions`
- Új enumok: `leave_request_status` (draft/pending/approved/rejected/cancelled/expired), `leave_type` (vacation/sick_leave/unpaid_leave/other)
- RLS policies: tagok saját kérelmet adhatnak be, owner/resourceAssistant jóváhagyhat/elutasíthat
- Új UI komponensek: `LeaveRequestDialog`, `LeaveRequestList`, `ApprovalInbox`
- WorkspaceDashboard-ban új tabok: Kérelmek, Jóváhagyások
- Bulk approve/reject funkció az approval inbox-ban szűrőkkel
- Módosított fájlok: `src/components/enterprise/WorkspaceDashboard.tsx`
- Új fájlok: `src/components/enterprise/LeaveRequestDialog.tsx`, `LeaveRequestList.tsx`, `ApprovalInbox.tsx`

## 2026-04-11 — v2.0.0 Enterprise Phase 1
- Jira: SYN-168, SYN-170, SYN-169
- Funkció: Enterprise B2B modul alapjai — Workspace, Membership, Roles
- Új DB táblák: `enterprise_workspaces`, `enterprise_memberships`, `enterprise_invitations`
- Új enumok: `enterprise_role` (owner/resourceAssistant/member), `enterprise_membership_status` (active/invited/suspended/removed)
- Új SECURITY DEFINER funkciók: `has_enterprise_role()`, `is_enterprise_member()`
- RLS policies: role-alapú hozzáférés-szabályozás minden enterprise táblán
- Új UI: `/enterprise` route, workspace lista, dashboard, tag kezelés, meghívók, beállítások
- ProfileMenu-ben Enterprise menüpont hozzáadva
- Módosított fájlok: `src/App.tsx`, `src/components/ProfileMenu.tsx`
- Új fájlok: `src/pages/Enterprise.tsx`, `src/components/enterprise/*`

## 2026-03-26
- Jira: nincs hozzárendelt jegyszám
- Funkció: a kitűzött nap kiválasztó popup mérete limitálva lett, egyszerre legfeljebb kb. 5 dátum látható, a lista görgethető maradt.
- Funkció: kitűzött nap esetén a batch kitöltés panel továbbra is megjelenik, de minden vezérlője inaktív állapotba kerül.
- Funkció: a kitűzött nap módosítása ablakban megjelent a Feloldás művelet, amellyel újranyitható a szavazás, ha az esemény nincs határidőn túl és nincs inaktiválva.
- Funkció: a személyes elérhetőség másolása nem fut le olyan eseményre, amelyen már van kitűzött nap.
- UI/reszponzivitás: a központi naptár lett az elsődleges szélességi referencia, a bal és jobb oldali panelek ehhez igazodnak kisebb hézagokkal.
- Technikai megvalósítás: módosítva `src/pages/Index.tsx`, `src/components/BatchVotePanel.tsx`, `src/components/PersonalCalendar.tsx`.

## 2026-03-31
- Jira: nincs hozzárendelt jegyszám
- Versioning hivatkozások:
  - `versioning/01033102_syncfolk_calendar_mobile_fix_request.pdf`
  - `versioning/01033102_syncfolk_calendar_mobile_fix_prompt.md`
- Üzleti kérés: csak az esemény részletező modal dátumválasztó naptárának mobilos megjelenését kellett javítani úgy, hogy a popup mindig a képernyő közepén jelenjen meg, és az OK gomb minden mobil nézetben látható maradjon.
- Technikai megvalósítás: az `EventInfoModal` dátumszerkesztő részében a korábbi nyers `Popover + Calendar` megoldás helyett a már létező, mobilon középre pozicionált `DatePopoverField` került használatba.
- Ellenőrzési checklist:
  - [x] `CHANGELOG.md` beolvasva
  - [ ] `codingLessonsLearnt.md` / lessons learnt fájl nem található a feltöltött repóban
  - [x] csak célzott hotfix készült az érintett dátumválasztó mezőkre
  - [x] mobilon középre kerül a calendar popup
  - [x] az OK gomb explicit módon megjelenik és használható marad
  - [x] a kezdő/vég dátum alaplogikája megmaradt
  - [x] build ellenőrzés lefutott sikeresen
- Módosított kódfájl: `src/components/EventInfoModal.tsx`

## 2026-04-22 — v2.5.0 Resource Management ökoszisztéma + dinamikus jogosultság-fa
- Phase 1 (Core Architecture, Single Source of Truth):
  - `enterprise_memberships.base_working_hours` (numeric, 0–24, default 8) — alap napi munkaóra tagonként
  - `capacityEngine` átállítva órás számításra: `hours = base_working_hours * (allocation_pct / 100)`
  - `BusinessRoleManager` (Pozíciók) most már listázza az ÖSSZES allokált tagot pozíciónként, %-os allokációval és számolt napi órával — megszüntetve a korábbi "csak 1 tag" data dissonance-t
  - `LeaveCalendar` csapat-szűrő dinamikusan a `enterprise_teams` táblából töltődik (hardcoded lista törölve)
  - `MemberProfileSheet`: szerkeszthető "Napi alap munkaóra" mező
- Phase 2 (UI Relocation & Dynamic Permissions):
  - Pozíciók és Csapatok kezelése áthelyezve a Beállításokból az **Erőforrások** fülre (alapból összecsukott akkordionok)
  - Új `enterprise_feature_catalog` tábla (parent_key hierarchia) — dinamikus, fa-alapú jogosultság-katalógus
  - `useEnterprisePermissions` hook visszaadja a `featureTree`-t (rekurzív struktúra)
  - `RolePermissionManager` UI átírva rekurzív fa-renderelésre (`FeatureTreeRow` komponens) — a jogosultság-választó automatikusan tükrözi az alkalmazás navigációs fáját, fallback a régi flat csoportosításra ha a katalógus üres
- Phase 3 (Resource Module — előző iterációkban):
  - `ResourceDashboard`, `ProjectList`, `ProjectEditor`, `GanttTimeline`, `CapacityGapReport`
  - "Kitűz a Riportokra" widget integráció (`ResourceWidgetCard`, `PinnedReportsWidget`)
- Új/módosított fájlok: `supabase/migrations/20260422164431_*.sql`, `src/lib/capacityEngine.ts`, `src/components/enterprise/BusinessRoleManager.tsx`, `LeaveCalendar.tsx`, `MemberProfileSheet.tsx`, `WorkspaceDashboard.tsx`, `RolePermissionManager.tsx`, `resources/ResourcesTab.tsx`, `src/hooks/useEnterprisePermissions.ts`
