# Implementation Plan — In-dashboard "Setup guide" card (Approach A)

Goal: add a numbered setup guide card at the TOP of the dashboard scrolling content so a new
user can configure everything FCM push needs. Each step has a button that triggers an EXISTING
action/screen. No new permissions, no manifest change, no new deps, plain Android views + XML,
plain Java, targetSdk 22 / compileSdk 35, no Material Components.

## Design decisions (made here, grounded in the code I read)

- Placement: the guide is a new `LinearLayout` card inserted as the FIRST child of
  `dashboard_scroll_content.xml` (before the Protection card). This matches the confirmed
  architecture: `activity_main.xml` is a custom `StickyDashboardLayout` whose `ScrollView`
  (`@+id/scroll`) `<include>`s `dashboard_scroll_content.xml`; the sticky header + CollapsingStatusCard
  stay untouched and the guide is purely additive. First-child placement needs NO `card_gap` top
  margin (the existing first card — Protection — also has none); the Protection card keeps its own
  layout because it currently relies on being first with no top margin, so inserting before it is safe.
- Styling: reuse the existing card/button conventions verified in the layout and resources:
  card container `@drawable/card_bg`, title 16sp bold `@color/text_primary`, step text ~13sp
  `@color/text_primary`, secondary/intro text 12sp `@color/text_secondary`, buttons styled like the
  existing secondary buttons (`@drawable/secondary_button_bg`, 48dp height, `textAllCaps=false`,
  `elevation=0dp`, `stateListAnimator=@null`, `textColor=@color/blue` bold for the emphasis button and
  `@color/text_primary` for the rest — mirroring `permissionBtn` vs `openAutostartBtn`). All of
  `@drawable/card_bg`, `@drawable/secondary_button_bg`, `@color/blue`, `@color/text_primary`,
  `@color/text_secondary`, `@dimen/card_gap` were confirmed to exist.
- Static vs live hint: v1 is a STATIC numbered guide (no live "satisfied" state). The task says a
  static guide is fully acceptable and not to over-engineer; refreshStatus() already surfaces live
  state (granted / enabled / present) in the sticky status card, so a second live indicator is redundant.
- Repair reuse: `repairBtn`'s listener is currently an inline lambda in `bindActions()`. Extract it
  into a private `performRepair()` so both `repairBtn` and `guideRepairBtn` call the identical path
  (saveConfig -> SettingsGuard.repair -> FcmReconnect.kick if changed -> refreshStatus(result.message)).
  No logic duplicated.
- Protection step: call `protectionSwitch.setChecked(true)` so the EXISTING
  `OnCheckedChangeListener` runs (permission check, service start, repair, toast). Do not re-implement.
  If already on, setChecked(true) is a no-op and fires nothing — acceptable.
- Button count: use 6 distinct buttons (`guideGrantBtn`, `guideRepairBtn`, `guideProtectionBtn`,
  `guideAutostartBtn`, `guideNotificationBtn`, `guideScanBtn`) — one per step, clearest for users and
  matches the task's preferred id set.
- Strings location: the task is authoritative and says put the new strings in `strings.xml`, so they
  go there. (Convention note: this repo also keeps a `feature_strings.xml` per locale for feature
  strings; aapt merges all resource files in a `values*` folder, so `strings.xml` is correct and
  compiles. Not blocking — following the task's explicit instruction.)
- Encoding: the base files declare `encoding="utf-8"`; new entries may be written as plain UTF-8
  (Latin/CJK). Existing ja/ko files use `&#x..;` entities, but mixing plain UTF-8 is valid XML and
  compiles — no need to entity-encode new strings.

## New string resource names (14 total)

- `guide_title` — card title, e.g. "Setup guide"  (1)
- `guide_intro` — one-line intro under the title  (1)
- `guide_step1` .. `guide_step6` — the six step lines, wording from README Quick setup  (6)
- `guide_btn_grant`, `guide_btn_repair`, `guide_btn_protection`, `guide_btn_autostart`,
  `guide_btn_notification`, `guide_btn_scan` — the six button labels  (6)

Total 2 + 6 + 6 = 14. Decision: add dedicated guide buttons rather than reusing labels such as
`repair_now`, because the guide reads better with imperative labels ("Grant permission", "Enable
automatic protection"). 14 strings is lean. Every one of these 14 names MUST exist in ALL 11
`strings.xml` files with identical names, or aapt fails.

The 11 string files (base + 10 locales) confirmed present:
`values`, `values-de`, `values-es`, `values-fr`, `values-ja`, `values-ko`, `values-pt`,
`values-ru`, `values-vi`, `values-zh-rCN`, `values-zh-rTW`.
(`values-night` and `values-w420dp` are NOT string-translation folders — do not add strings there.)

---

- [ ] 1. Add the 14 guide strings to the BASE locale `app/src/main/res/values/strings.xml` (English).
      Insert a `<!-- Setup guide -->` block before `</resources>`. Use these exact names/values:
      - `guide_title` = `Setup guide`
      - `guide_intro` = `Follow these steps once so FCM push notifications arrive reliably.`
      - `guide_step1` = `1. Grant the "Modify system settings" permission.`
      - `guide_step2` = `2. Tap "Repair now" once to add Google Play services to the whitelist.`
      - `guide_step3` = `3. Enable "Automatic protection".`
      - `guide_step4` = `4. In HyperOS, enable Autostart for FCM Guard and set battery policy to "No restrictions".`
      - `guide_step5` = `5. Keep "Persistent notification" on, and allow notifications if the system blocks them.`
      - `guide_step6` = `6. Optional: scan FCM apps and configure any that still deliver late.`
      - `guide_btn_grant` = `Grant permission`
      - `guide_btn_repair` = `Repair now`
      - `guide_btn_protection` = `Enable automatic protection`
      - `guide_btn_autostart` = `Open HyperOS Autostart`
      - `guide_btn_notification` = `Open notification settings`
      - `guide_btn_scan` = `Scan FCM apps`
      Files: app/src/main/res/values/strings.xml
      Verify: file is well-formed XML (opens/closes `<resources>`); all 14 names present once.
      Static check (b) in step 9 confirms the full 11-file set.

- [ ] 2. Add the same 14 string names with natural translations to EACH of the 10 locale
      `strings.xml` files. One item (same pattern applied across 10 files). Suggested natural
      translations below; the coder may refine wording but names MUST match base exactly.
      Files: app/src/main/res/values-de/strings.xml, app/src/main/res/values-es/strings.xml,
      app/src/main/res/values-fr/strings.xml, app/src/main/res/values-ja/strings.xml,
      app/src/main/res/values-ko/strings.xml, app/src/main/res/values-pt/strings.xml,
      app/src/main/res/values-ru/strings.xml, app/src/main/res/values-vi/strings.xml,
      app/src/main/res/values-zh-rCN/strings.xml, app/src/main/res/values-zh-rTW/strings.xml

      vi (Vietnamese):
        guide_title = Hướng dẫn thiết lập
        guide_intro = Thực hiện các bước sau một lần để thông báo đẩy FCM đến đáng tin cậy.
        guide_step1 = 1. Cấp quyền "Sửa đổi cài đặt hệ thống".
        guide_step2 = 2. Chạm "Sửa ngay" một lần để thêm Google Play services vào danh sách trắng.
        guide_step3 = 3. Bật "Bảo vệ tự động".
        guide_step4 = 4. Trong HyperOS, bật Tự khởi động cho FCM Guard và đặt chế độ pin thành "Không giới hạn".
        guide_step5 = 5. Giữ "Thông báo thường trực" bật, và cho phép thông báo nếu hệ thống chặn.
        guide_step6 = 6. Tùy chọn: quét ứng dụng FCM và cấu hình những ứng dụng còn nhận trễ.
        guide_btn_grant = Cấp quyền
        guide_btn_repair = Sửa ngay
        guide_btn_protection = Bật bảo vệ tự động
        guide_btn_autostart = Mở Tự khởi động HyperOS
        guide_btn_notification = Mở cài đặt thông báo
        guide_btn_scan = Quét ứng dụng FCM

      zh-rCN (Simplified Chinese):
        guide_title = 设置向导
        guide_intro = 按以下步骤设置一次，让 FCM 推送通知可靠送达。
        guide_step1 = 1. 授予“修改系统设置”权限。
        guide_step2 = 2. 点击一次“立即修复”，将 Google Play services 加入白名单。
        guide_step3 = 3. 开启“自动保护”。
        guide_step4 = 4. 在 HyperOS 中为 FCM Guard 开启自启动，并将省电策略设为“无限制”。
        guide_step5 = 5. 保持“常驻通知”开启；若系统拦截通知，请允许通知。
        guide_step6 = 6. 可选：扫描 FCM 应用，并配置仍然延迟送达的应用。
        guide_btn_grant = 授予权限
        guide_btn_repair = 立即修复
        guide_btn_protection = 开启自动保护
        guide_btn_autostart = 打开 HyperOS 自启动
        guide_btn_notification = 打开通知设置
        guide_btn_scan = 扫描 FCM 应用

      zh-rTW (Traditional Chinese):
        guide_title = 設定精靈
        guide_intro = 依下列步驟設定一次，讓 FCM 推播通知可靠送達。
        guide_step1 = 1. 授予「修改系統設定」權限。
        guide_step2 = 2. 點一次「立即修復」，將 Google Play 服務加入白名單。
        guide_step3 = 3. 開啟「自動保護」。
        guide_step4 = 4. 在 HyperOS 中為 FCM Guard 開啟自動啟動，並將電池政策設為「無限制」。
        guide_step5 = 5. 保持「常駐通知」開啟；若系統封鎖通知，請允許通知。
        guide_step6 = 6. 選用：掃描 FCM 應用程式，並設定仍延遲送達的應用程式。
        guide_btn_grant = 授予權限
        guide_btn_repair = 立即修復
        guide_btn_protection = 開啟自動保護
        guide_btn_autostart = 開啟 HyperOS 自動啟動
        guide_btn_notification = 開啟通知設定
        guide_btn_scan = 掃描 FCM 應用程式

      fr (French):
        guide_title = Guide de configuration
        guide_intro = Suivez ces étapes une fois pour que les notifications push FCM arrivent de façon fiable.
        guide_step1 = 1. Accordez l’autorisation « Modifier les paramètres système ».
        guide_step2 = 2. Touchez une fois « Réparer maintenant » pour ajouter Google Play services à la liste blanche.
        guide_step3 = 3. Activez « Protection automatique ».
        guide_step4 = 4. Dans HyperOS, activez le Démarrage auto pour FCM Guard et réglez la batterie sur « Aucune restriction ».
        guide_step5 = 5. Gardez « Notification persistante » activée, et autorisez les notifications si le système les bloque.
        guide_step6 = 6. Facultatif : analysez les apps FCM et configurez celles qui livrent encore en retard.
        guide_btn_grant = Accorder l’autorisation
        guide_btn_repair = Réparer maintenant
        guide_btn_protection = Activer la protection automatique
        guide_btn_autostart = Ouvrir le Démarrage auto HyperOS
        guide_btn_notification = Ouvrir les paramètres de notification
        guide_btn_scan = Analyser les apps FCM

      ja (Japanese):
        guide_title = セットアップガイド
        guide_intro = FCM プッシュ通知を確実に受け取るため、次の手順を一度だけ実行してください。
        guide_step1 = 1. 「システム設定の変更」権限を許可します。
        guide_step2 = 2. 「今すぐ修復」を一度タップし、Google Play services をホワイトリストに追加します。
        guide_step3 = 3. 「自動保護」を有効にします。
        guide_step4 = 4. HyperOS で FCM Guard の自動起動を有効にし、バッテリーを「制限なし」に設定します。
        guide_step5 = 5. 「常駐通知」を有効のままにし、システムが通知をブロックする場合は許可します。
        guide_step6 = 6. 任意: FCM アプリをスキャンし、まだ遅延するアプリを設定します。
        guide_btn_grant = 権限を許可
        guide_btn_repair = 今すぐ修復
        guide_btn_protection = 自動保護を有効化
        guide_btn_autostart = HyperOS の自動起動を開く
        guide_btn_notification = 通知設定を開く
        guide_btn_scan = FCM アプリをスキャン

      ko (Korean):
        guide_title = 설정 가이드
        guide_intro = FCM 푸시 알림이 안정적으로 도착하도록 다음 단계를 한 번 수행하세요.
        guide_step1 = 1. "시스템 설정 변경" 권한을 허용하세요.
        guide_step2 = 2. "지금 복구"를 한 번 눌러 Google Play 서비스를 허용 목록에 추가하세요.
        guide_step3 = 3. "자동 보호"를 켜세요.
        guide_step4 = 4. HyperOS에서 FCM Guard의 자동 시작을 켜고 배터리 정책을 "제한 없음"으로 설정하세요.
        guide_step5 = 5. "상시 알림"을 켠 상태로 두고, 시스템이 알림을 차단하면 허용하세요.
        guide_step6 = 6. 선택: FCM 앱을 검색하고 여전히 늦게 도착하는 앱을 설정하세요.
        guide_btn_grant = 권한 허용
        guide_btn_repair = 지금 복구
        guide_btn_protection = 자동 보호 켜기
        guide_btn_autostart = HyperOS 자동 시작 열기
        guide_btn_notification = 알림 설정 열기
        guide_btn_scan = FCM 앱 검색

      es (Spanish):
        guide_title = Guía de configuración
        guide_intro = Sigue estos pasos una vez para que las notificaciones push de FCM lleguen de forma fiable.
        guide_step1 = 1. Concede el permiso «Modificar ajustes del sistema».
        guide_step2 = 2. Toca «Reparar ahora» una vez para añadir Google Play services a la lista blanca.
        guide_step3 = 3. Activa la «Protección automática».
        guide_step4 = 4. En HyperOS, activa el Inicio automático de FCM Guard y pon la batería en «Sin restricciones».
        guide_step5 = 5. Mantén activada la «Notificación persistente» y permite las notificaciones si el sistema las bloquea.
        guide_step6 = 6. Opcional: busca apps FCM y configura las que aún llegan tarde.
        guide_btn_grant = Conceder permiso
        guide_btn_repair = Reparar ahora
        guide_btn_protection = Activar protección automática
        guide_btn_autostart = Abrir Inicio automático de HyperOS
        guide_btn_notification = Abrir ajustes de notificaciones
        guide_btn_scan = Buscar apps FCM

      pt (Portuguese):
        guide_title = Guia de configuração
        guide_intro = Siga estes passos uma vez para que as notificações push do FCM cheguem de forma fiável.
        guide_step1 = 1. Conceda a permissão «Modificar definições do sistema».
        guide_step2 = 2. Toque em «Reparar agora» uma vez para adicionar o Google Play services à lista branca.
        guide_step3 = 3. Ative a «Proteção automática».
        guide_step4 = 4. No HyperOS, ative o Arranque automático do FCM Guard e defina a bateria como «Sem restrições».
        guide_step5 = 5. Mantenha a «Notificação persistente» ativada e permita as notificações se o sistema as bloquear.
        guide_step6 = 6. Opcional: procure apps FCM e configure as que ainda entregam com atraso.
        guide_btn_grant = Conceder permissão
        guide_btn_repair = Reparar agora
        guide_btn_protection = Ativar proteção automática
        guide_btn_autostart = Abrir Arranque automático do HyperOS
        guide_btn_notification = Abrir definições de notificações
        guide_btn_scan = Procurar apps FCM

      de (German):
        guide_title = Einrichtungsassistent
        guide_intro = Führe diese Schritte einmal aus, damit FCM-Push-Benachrichtigungen zuverlässig ankommen.
        guide_step1 = 1. Erteile die Berechtigung „Systemeinstellungen ändern“.
        guide_step2 = 2. Tippe einmal auf „Jetzt reparieren“, um Google Play services zur Whitelist hinzuzufügen.
        guide_step3 = 3. Aktiviere den „Automatischen Schutz“.
        guide_step4 = 4. Aktiviere in HyperOS den Autostart für FCM Guard und setze die Akkurichtlinie auf „Keine Einschränkungen“.
        guide_step5 = 5. Lass die „Dauerhafte Benachrichtigung“ aktiviert und erlaube Benachrichtigungen, falls das System sie blockiert.
        guide_step6 = 6. Optional: Scanne FCM-Apps und konfiguriere die, die noch verspätet zustellen.
        guide_btn_grant = Berechtigung erteilen
        guide_btn_repair = Jetzt reparieren
        guide_btn_protection = Automatischen Schutz aktivieren
        guide_btn_autostart = HyperOS-Autostart öffnen
        guide_btn_notification = Benachrichtigungseinstellungen öffnen
        guide_btn_scan = FCM-Apps scannen

      ru (Russian):
        guide_title = Руководство по настройке
        guide_intro = Выполните эти шаги один раз, чтобы push-уведомления FCM приходили надёжно.
        guide_step1 = 1. Предоставьте разрешение «Изменение системных настроек».
        guide_step2 = 2. Нажмите «Восстановить» один раз, чтобы добавить Google Play services в белый список.
        guide_step3 = 3. Включите «Автоматическую защиту».
        guide_step4 = 4. В HyperOS включите автозапуск для FCM Guard и задайте для батареи режим «Без ограничений».
        guide_step5 = 5. Оставьте «Постоянное уведомление» включённым и разрешите уведомления, если система их блокирует.
        guide_step6 = 6. Необязательно: просканируйте FCM-приложения и настройте те, что доставляют с задержкой.
        guide_btn_grant = Предоставить разрешение
        guide_btn_repair = Восстановить
        guide_btn_protection = Включить автоматическую защиту
        guide_btn_autostart = Открыть автозапуск HyperOS
        guide_btn_notification = Открыть настройки уведомлений
        guide_btn_scan = Сканировать FCM-приложения

      Verify: each file stays well-formed XML and contains all 14 new names. Covered by static
      check (b) and by `python tools\check_layout_profiles.py` not regressing (step 10).

- [ ] 3. Add the guide card as the FIRST child `LinearLayout` of the content root in
      `app/src/main/res/layout/dashboard_scroll_content.xml`, immediately after the opening
      `<LinearLayout android:id="@+id/contentRoot" ...>` tag and BEFORE the existing Protection card.
      The card uses `@drawable/card_bg`, NO top margin (it is first). Structure:
      - Card container: `LinearLayout` orientation vertical, `@+id/guideCard`, background `@drawable/card_bg`.
      - Title `TextView`: `@string/guide_title`, `@color/text_primary`, 16sp bold, paddingBottom 4dp.
      - Intro `TextView`: `@string/guide_intro`, `@color/text_secondary`, 12sp, lineSpacingExtra 2dp,
        paddingBottom 8dp.
      - Repeat the pattern below for each step (step text then its button), with the button marginTop 6dp
        and a step marginTop ~12dp between groups:
          - step `TextView` `@string/guide_stepN`, `@color/text_primary`, 13sp, lineSpacingExtra 2dp.
          - `Button` `@+id/guideGrantBtn` text `@string/guide_btn_grant`, layout_width match_parent,
            layout_height 48dp, `textAllCaps=false`, `textColor=@color/blue`, `textStyle=bold`,
            `background=@drawable/secondary_button_bg`, `elevation=0dp`, `stateListAnimator=@null`.
          - Steps 2–6 buttons `@+id/guideRepairBtn`, `@+id/guideProtectionBtn`, `@+id/guideAutostartBtn`,
            `@+id/guideNotificationBtn`, `@+id/guideScanBtn` with their `@string/guide_btn_*` labels,
            same style but `textColor=@color/text_primary` (no bold) to match secondary buttons like
            `openAutostartBtn`. Keep step 1's button emphasized in blue to echo `permissionBtn`.
      Six stable button ids total: guideGrantBtn, guideRepairBtn, guideProtectionBtn, guideAutostartBtn,
      guideNotificationBtn, guideScanBtn, plus the card id guideCard.
      Files: app/src/main/res/layout/dashboard_scroll_content.xml
      Verify: `python tools\check_layout_profiles.py` passes (it parses activity_main.xml, not this file,
      but confirms no XML/geometry regression in the responsive set); XML is well-formed; the existing
      Protection card and all later cards remain unchanged.

- [ ] 4. In `app/src/main/java/com/reed/fcmguard/MainActivity.java`, add six `Button` fields for the
      guide buttons near the existing field declarations (e.g. after `permissionBtn`):
      `guideGrantBtn, guideRepairBtn, guideProtectionBtn, guideAutostartBtn, guideNotificationBtn,
      guideScanBtn`. (Optional: bind directly in setupGuide via findViewById without fields; fields are
      used here for symmetry with existing buttons like `permissionBtn`/`scanFcmAppsBtn`.)
      Files: app/src/main/java/com/reed/fcmguard/MainActivity.java
      Verify: compiles conceptually (static check d); fields reference `android.widget.Button` already imported.

- [ ] 5. In the same file, add `bindViews()` assignments for the six new buttons:
      `guideGrantBtn = findViewById(R.id.guideGrantBtn);` and the five others, matching the layout ids.
      Files: app/src/main/java/com/reed/fcmguard/MainActivity.java
      Verify: every new `R.id.guide*Btn` referenced in bindViews exists in the layout (static check a).

- [ ] 6. Extract the inline `repairBtn` lambda in `bindActions()` into a private method
      `performRepair()` containing exactly the current body:
      `SettingsGuard.saveConfig(this, keyEdit.getText().toString(), itemEdit.getText().toString());
       SettingsGuard.Result result = SettingsGuard.repair(this);
       if (result.changed) FcmReconnect.kick(this);
       refreshStatus(result.message);`
      Then change the existing `repairBtn` listener to `findViewById(R.id.repairBtn).setOnClickListener(v -> performRepair());`.
      Files: app/src/main/java/com/reed/fcmguard/MainActivity.java
      Verify: behavior of repairBtn unchanged (same statements); static check c confirms the guide repair
      button reuses `performRepair()`.

- [ ] 7. Add a private `setupGuide()` method that attaches click listeners calling the EXISTING paths,
      and call it from `onCreate()`. Listener wiring:
      - `guideGrantBtn` -> `openWriteSettings();`
      - `guideRepairBtn` -> `performRepair();`  (reuses step 6)
      - `guideProtectionBtn` -> `protectionSwitch.setChecked(true);`  (fires existing listener)
      - `guideAutostartBtn` -> `if (!HyperOsSettings.openAutoStartManager(this)) toast(getString(R.string.autostart_manager_unavailable));`
        (identical to the existing `openAutostartBtn` wiring)
      - `guideNotificationBtn` -> `openNotificationSettings();`
      - `guideScanBtn` -> `toggleFcmAppsList();`  (same as `scanFcmAppsBtn`)
      Place the `setupGuide();` call in `onCreate()` AFTER `bindActions();` and before `refreshStatus(null);`
      so all other views/switches are already wired (protectionSwitch listener must exist before the
      guide can trigger it; it is set in `setupSwitches()` which runs earlier). Final onCreate order:
      bindViews, loadConfigIntoFields, setupLanguagePicker, setupAppearance, setupFallbackInterval,
      setupSwitches, bindActions, setupGuide, refreshStatus.
      Alternatively wire these inside `bindActions()`; a dedicated `setupGuide()` keeps the guide
      cohesive — chosen for clarity.
      Files: app/src/main/java/com/reed/fcmguard/MainActivity.java
      Verify: static check c — each guide listener calls an existing method with the correct signature
      (`openWriteSettings()`, `performRepair()`, `protectionSwitch.setChecked(boolean)`,
      `HyperOsSettings.openAutoStartManager(Context)`, `openNotificationSettings()`, `toggleFcmAppsList()`).
      No new imports needed (all symbols already imported/declared).

- [ ] 8. Confirm no new permissions / manifest / dependency / signing changes were introduced and the
      sticky header + CollapsingStatusCard + existing cards are untouched (guide is additive only).
      Files: (review only) AndroidManifest.xml, app/build.gradle — must be UNCHANGED.
      Verify: `git`/build are no-ops here; confirm by inspection that these files were not edited.

- [ ] 9. Run the four static self-checks (manual inspection, since gradle cannot run here):
      a. Every new `R.id.guide*` referenced in MainActivity (guideGrantBtn, guideRepairBtn,
         guideProtectionBtn, guideAutostartBtn, guideNotificationBtn, guideScanBtn) is declared in
         dashboard_scroll_content.xml, and every new `@+id/` in the layout is referenced in MainActivity.
      b. Every new `R.string.guide_*` (14 names) exists in ALL 11 `strings.xml` files with identical
         names. Suggested command to count occurrences per name across the 11 files:
         `Select-String -Path app\src\main\res\values*\strings.xml -Pattern 'name="guide_title"'`
         (repeat per name, or loop) — each must return exactly 11 matches, and `values-night` /
         `values-w420dp` must have ZERO (they only hold non-string resources).
      c. Guide buttons call existing methods with correct signatures (see step 7 list); `performRepair()`
         exists and both repairBtn and guideRepairBtn use it.
      d. Java imports/symbols sane: no new imports required; Button already imported; no unresolved symbols.
      Files: (review only)
      Verify: all four checks pass by inspection / grep counts.

- [ ] 10. Run the responsive layout checker to confirm no regression.
      Files: (none)
      Verify: `python tools\check_layout_profiles.py` prints "Responsive layout checks passed for:" and
      exits 0 (baseline already confirmed green before these edits).

## Notes / constraints restated

- `gradle assembleDebug` is NOT runnable in this environment (Android SDK + Gradle not installed);
  verification is static inspection + `python tools\check_layout_profiles.py`.
- There is NO git repo here — do NOT run `git init` / worktree / commit; git operations are a no-op.
- No new permissions, no manifest change, no targetSdk/compileSdk change, no signing change, no new
  dependencies/libraries. Plain Android views + XML, plain Java only.
- Do NOT duplicate repair/permission logic — reuse `performRepair()`, `openWriteSettings()`,
  `openNotificationSettings()`, `HyperOsSettings.openAutoStartManager()`, `toggleFcmAppsList()`, and the
  protection switch's existing listener.
- Keep the sticky header + CollapsingStatusCard + all existing cards intact; the guide is purely
  additive at the top of the scrolling content.

## Gap / assumption log

- The task permits either 6 buttons or combined labels; I chose 6 distinct buttons for clarity and to
  match the preferred id set. If a reviewer prefers fewer strings, the step/button labels could be
  merged, but 14 strings is already lean.
- Repo convention puts feature strings in `feature_strings.xml`; the task explicitly says `strings.xml`,
  which also compiles (aapt merges all values files). Followed the task. If strict convention is
  desired later, the same 14 strings could instead live in `feature_strings.xml` per locale with no
  behavior change — but that is NOT part of this task.
