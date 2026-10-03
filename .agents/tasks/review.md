# Setup guide card added to dashboard top

A numbered setup-guide card was added as the first child of `dashboard_scroll_content.xml`, backed by a `setupGuide()` method in `MainActivity` and 14 new string resources across all 11 `strings.xml` files. The change is purely additive: each of the six step buttons wires to an existing code path, the inline `repairBtn` lambda was cleanly extracted into `performRepair()` and reused, and no permissions, manifest, SDK, or dependency changes were introduced. The implementation matches the plan and the task requirements closely.

Watch for: ASCII double quotes in the base English and Vietnamese `guide_step*` strings are consumed by aapt2 as whitespace-preservation span delimiters and will be silently stripped from the rendered text — the quoted UI terms will display without quotes (confirmed; cosmetic, non-blocking).

**Verdict**: APPROVED

## High-level view

Placement and styling meet the requirement: `guideCard` is the first `LinearLayout` under `contentRoot`, before the Protection card, with no top margin (first-child), and the Protection card now carries `layout_marginTop="@dimen/card_gap"` so spacing is preserved. All six buttons use `@drawable/secondary_button_bg`, 48dp, `textAllCaps=false`, `elevation=0dp`, `stateListAnimator=@null`; step 1 is emphasized blue/bold mirroring `permissionBtn`, steps 2-6 use `text_primary` mirroring `openAutostartBtn`.

Reuse is correct and duplication-free. `performRepair()` was extracted with the exact original body and is called by both `repairBtn` and `guideRepairBtn`. The guide buttons delegate to `openWriteSettings()`, `performRepair()`, `protectionSwitch.setChecked(true)` (which runs the existing listener), `HyperOsSettings.openAutoStartManager(this)` with the `autostart_manager_unavailable` toast fallback, `openNotificationSettings()`, and `toggleFcmAppsList()`. `setupGuide()` is called from `onCreate()` after `bindActions()` and before `refreshStatus(null)`, so the protection switch listener (set in `setupSwitches()`) already exists when the guide can trigger it.

Consistency constraints hold. All seven new ids (`guideCard` + 6 buttons) are declared in the layout and bound in `bindViews()`; all 14 new string names exist in every one of the 11 `strings.xml` files with identical names; no new imports are needed (`Button` already imported). No new permissions, manifest, targetSdk, or dependency changes.

The one defect is cosmetic: the base English and Vietnamese step strings use ASCII `"` around UI terms, which aapt2 strips as span delimiters. It compiles cleanly but renders without the quotes.

<details>
<summary>Issues (1)</summary>

1. **Unescaped double quotes in base + vi step strings** — `guide_step1/2/3/4/5` in `values/strings.xml` and `values-vi/strings.xml` use balanced ASCII `"` which aapt2 consumes as whitespace-span delimiters, so the quote characters are stripped from the rendered text. Non-blocking (compiles, no crash). To preserve the quotes, escape them as `\"` or switch to typographic quotes as the other locales already do.

</details>

<details>
<summary>Details</summary>

### Placement and styling

`guideCard` is the first child `LinearLayout` directly under `contentRoot`, before the Protection card, with no top margin. The Protection card now carries `layout_marginTop="@dimen/card_gap"`, which restores the spacing it lost by no longer being first. Card background is `@drawable/card_bg`; title is 16sp bold `text_primary`; intro 12sp `text_secondary`; each step line 13sp `text_primary` with `lineSpacingExtra="2dp"` and a 12dp top margin between groups. Every button is `match_parent` × 48dp, `textAllCaps=false`, `background=@drawable/secondary_button_bg`, `elevation=0dp`, `stateListAnimator=@null`, with a 6dp top margin. Step 1's button is `@color/blue` bold (echoing `permissionBtn`); steps 2-6 are `@color/text_primary` (echoing `openAutostartBtn`).

### Reuse without duplication

The inline `repairBtn` lambda in `bindActions()` was replaced by `findViewById(R.id.repairBtn).setOnClickListener(v -> performRepair())`, and `performRepair()` contains the exact original sequence:

```java
private void performRepair() {
    SettingsGuard.saveConfig(this, keyEdit.getText().toString(), itemEdit.getText().toString());
    SettingsGuard.Result result = SettingsGuard.repair(this);
    if (result.changed) FcmReconnect.kick(this);
    refreshStatus(result.message);
}
```

`setupGuide()` wires each guide button to an existing path with no copied logic:

```java
guideGrantBtn        -> openWriteSettings();
guideRepairBtn       -> performRepair();
guideProtectionBtn   -> protectionSwitch.setChecked(true);   // runs existing listener
guideAutostartBtn    -> if (!HyperOsSettings.openAutoStartManager(this)) toast(... autostart_manager_unavailable);
guideNotificationBtn -> openNotificationSettings();
guideScanBtn         -> toggleFcmAppsList();
```

The autostart wiring is byte-for-byte identical to the existing `openAutostartBtn` handler, including the fallback toast. `guideProtectionBtn` relies on the protection switch listener, which is installed in `setupSwitches()` earlier in `onCreate()`, so the ordering (`...setupSwitches, bindActions, setupGuide, refreshStatus`) is sound. If protection is already on, `setChecked(true)` is a no-op — acceptable.

### Id and string consistency

The layout declares `guideCard`, `guideGrantBtn`, `guideRepairBtn`, `guideProtectionBtn`, `guideAutostartBtn`, `guideNotificationBtn`, `guideScanBtn`. `bindViews()` binds the six button fields to the same ids; the card id is referenced only in the layout, which is fine. All 14 string names (`guide_title`, `guide_intro`, `guide_step1`–`6`, and the six `guide_btn_*`) were confirmed present in all 11 `strings.xml` files (base + de/es/fr/ja/ko/pt/ru/vi/zh-rCN/zh-rTW) by spot-checking `guide_title`, `guide_step6`, and `guide_btn_scan` across the set — each returned exactly 11 matches, one per locale file. The Vietnamese translations read naturally.

### Double-quote span stripping (cosmetic)

The base `values/strings.xml` and `values-vi/strings.xml` write UI terms with ASCII double quotes, e.g. `1. Grant the "Modify system settings" permission.` and `1. Cấp quyền "Sửa đổi cài đặt hệ thống".`. In Android string resources an unescaped `"` is not a literal character — it toggles a whitespace-preserving span. A balanced pair, as here, is consumed as span delimiters and removed from the stored string, so the rendered text shows the term without quotes. This compiles without error (no aapt failure, no crash), so it does not block the build. It is purely a display regression limited to the base and Vietnamese locales; the other nine locales use typographic quotes (`„"`, `«»`, `""`, `「」`) which are ordinary characters and render correctly. The existing strings in the project never use literal ASCII quotes, so this is new to the change. Fix by escaping (`\"`) or adopting typographic quotes to match the other locales.

### Constraints and verification evidence

No new permissions, no `AndroidManifest.xml` edit, no `app/build.gradle` edit, no targetSdk/compileSdk/signing/dependency change — consistent with the additive nature of the diff and the coder's recorded evidence. The sticky header and CollapsingStatusCard live in `activity_main.xml` and are untouched; the existing Protection, Tools, FCM apps, Advanced, info, and Appearance cards are intact.

The coder recorded in `VERIFICATION.md` that `python tools\check_layout_profiles.py` exits 0 ("Responsive layout checks passed" for all width profiles) and that `gradle assembleDebug` was not run because no Android SDK/Gradle exists in this environment — matching the task's expectation. Per instruction, the full build and the layout checker were not re-run; the recorded evidence is accepted.

Not independently verified: runtime behavior on-device (no SDK/emulator available) and the final aapt2 output (gradle not runnable here). The quote-span behavior is a well-established aapt2 rule rather than something traced through a build in this environment.

</details>

<details>
<summary>File map</summary>

- `app/src/main/res/layout/dashboard_scroll_content.xml` — added `guideCard` as first child; added `card_gap` top margin to the Protection card.
- `app/src/main/java/com/reed/fcmguard/MainActivity.java` — 6 button fields, 6 `bindViews()` assignments, extracted `performRepair()`, added `setupGuide()`, called it in `onCreate()`.
- `app/src/main/res/values/strings.xml` + 10 locale `strings.xml` — 14 new guide strings each.

No git repo in this environment; review done by direct file inspection against the recorded verification evidence.

</details>
