# Verification — In-dashboard "Setup guide" card (Approach A)

Iteration: FIRST (no `.agents/tasks/review.json` present when this ran).

## What was implemented
- New guide card inserted as the FIRST child `LinearLayout` of
  `app/src/main/res/layout/dashboard_scroll_content.xml` (id `guideCard`), before the Protection
  card. The Protection card was given `layout_marginTop="@dimen/card_gap"` since it is no longer
  first. Card styled like existing cards: `@drawable/card_bg`, title 16sp bold `@color/text_primary`,
  intro 12sp `@color/text_secondary`, step text 13sp `@color/text_primary`, buttons
  `@drawable/secondary_button_bg`, 48dp, `textAllCaps=false`, `elevation=0dp`,
  `stateListAnimator=@null`. Step 1 button emphasized `@color/blue` bold (mirrors `permissionBtn`);
  steps 2-6 `@color/text_primary` (mirror `openAutostartBtn`).
- Button ids: `guideGrantBtn, guideRepairBtn, guideProtectionBtn, guideAutostartBtn,
  guideNotificationBtn, guideScanBtn`.
- `MainActivity.java`: added 6 `Button` fields, 6 `bindViews()` assignments, extracted the inline
  `repairBtn` lambda into `private void performRepair()` (repairBtn now calls `performRepair()`), added
  `private void setupGuide()` wiring each guide button to the EXISTING path
  (`openWriteSettings()`, `performRepair()`, `protectionSwitch.setChecked(true)`,
  `HyperOsSettings.openAutoStartManager(this)` with the `autostart_manager_unavailable` toast fallback,
  `openNotificationSettings()`, `toggleFcmAppsList()`), and called `setupGuide()` from `onCreate()`
  after `bindActions()` and before `refreshStatus(null)`.
- 14 new strings (`guide_title`, `guide_intro`, `guide_step1..6`, `guide_btn_grant`,
  `guide_btn_repair`, `guide_btn_protection`, `guide_btn_autostart`, `guide_btn_notification`,
  `guide_btn_scan`) added to the base `values/strings.xml` and translated in all 10 locale
  `strings.xml` files (de, es, fr, ja, ko, pt, ru, vi, zh-rCN, zh-rTW).

## Evidence
- `python tools\check_layout_profiles.py` -> exit 0, "Responsive layout checks passed" for all width
  profiles (320/360/393/411/430/480dp). Baseline was green before edits; still green after.
- All 11 `strings.xml` files parse as well-formed XML and each contains all 14 new string names
  (verified by ElementTree script). `values-night` / `values-w420dp` / `feature_strings.xml` were NOT
  modified (confirmed: `guide_title` appears only in the 11 `strings.xml` and the layout).
- `dashboard_scroll_content.xml` parses well-formed; the 7 guide ids (`guideCard` + 6 buttons) are
  present exactly as referenced in `MainActivity.bindViews()`.
- Static check (a): every `R.id.guide*` referenced in MainActivity is declared in the layout, and
  every new `@+id/guide*` in the layout is bound in MainActivity. Match.
- Static check (c): guide listeners call existing methods with correct signatures; `performRepair()`
  exists and is used by BOTH `repairBtn` and `guideRepairBtn` (no duplicated repair logic).
- Static check (d): no new imports needed (`android.widget.Button` already imported); no unresolved
  symbols.
- No new permissions, no `AndroidManifest.xml` change, no `app/build.gradle` change, no targetSdk/
  signing/dependency change. Guide is purely additive; sticky header + CollapsingStatusCard +
  existing cards untouched.

## Not run
- `gradle assembleDebug` was NOT run: the Android SDK and Gradle are not installed in this
  environment. Verification is static inspection + the responsive layout checker, as instructed.
- No git operations: there is no git repo here, so merge/rebase/commit is a no-op.
