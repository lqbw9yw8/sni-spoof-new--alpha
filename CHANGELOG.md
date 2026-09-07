# CHANGELOG

قالب: [Keep a Changelog](https://keepachangelog.com/) · نسخه‌بندی: SemVer

> وضعیت هر ورودی طبق `AI_RULES.md` بخش ۳ برچسب می‌خورد.

## [Unreleased] — گیت‌های کامل CI سبز شد (TESTS_EXECUTED)

این دور تمام پنج گیت `ci/github-actions.yml` واقعاً اجرا و سبز شد
(Rust 1.98.1 stable، لینوکس): `cargo fmt --all -- --check` ✓،
`cargo check --all-targets` → **۰ خطا / ۰ هشدار**، `cargo test --all-targets`
→ **۳۸۳ پاس / ۰ شکست**، `cargo clippy --all-targets -- -D warnings` ✓،
`npm test` (uitest) → **۳۶۹ پاس / ۰ شکست**.

### Fixed — `VERIFIED` (اجرا شده روی Rust 1.98.1)
- **ادعای قبلیِ «صفر هشدار» ناقص بود** — `cargo check --all-targets` هنوز ۳ هشدار داشت که در چک‌های قبلی از قلم افتاده بود:
  - **`dns_cache.rs`**: فیلد هرز `DnsCache.ttl` (هرگز خوانده نمی‌شد؛ تازگی per-entry جایگزینش شده بود) و سازندهٔ بلااستفادهٔ `with_ttl` حذف شدند؛ دلیل طراحی per-entry در کامنت مستند شد.
  - **`main.rs`**: `ProxyRestoreGuard` فقط داخل `backend_main` ویندوزی arm می‌شود و به `proxy_cleanup` فقط-ویندوزی وصل است — با `#[cfg(windows)]` گیت شد تا چک غیر-ویندوزی هم بدون هشدار باشد (رفتار ویندوزی بدون تغییر).
  - **`pipeline.rs`**: نام تست `hold_watchdog_preserves_original_winDivert_address_semantics` به snake_case اصلاح شد.
- **۱۰ ماسک `#[allow(unused_imports)]`** در geedge/scanner/client_detect/mobile_gateway/proxy_cleanup/engine برداشته شد: ۶ ایمپورت واقعاً مرده حذف (`Rng`، `DpiGuardError` ×۳، `ToSocketAddrs` — که بعداً با تست کامپایل برگشت، `CloseAction`، `Path`) و بقیه بدون ماسک نگه داشته شدند.
- **۴۸ هشدار clippy صفر شد** (گیت CI `-D warnings` می‌بست و قرمز می‌ماند):
  - ۲۰ × `field_reassign_with_default` → سینتکس struct-update (`..Default::default()`) در تست‌های lib/config/main/pipeline/webui/native_gui.
  - ۱۲ × `doc_list_item_without_indentation` → تورفتگی ادامه‌بندهای لیست در `strategy.rs`/`geedge.rs`.
  - ۴ × `&mut Vec` → `&mut [_]` در `packet.rs`/`sequence.rs`/`engine_stub.rs` (بدون تغییر رفتار؛ صدا زننده‌ها با coerce خودکار).
  - ۲ × `type_complexity` → الیاس `ResolvedSlot` در main.rs و `MutationFn` در sni_mutations.rs.
  - `manual_clamp` در autottl.rs و `chunks_exact_to_as_chunks` در packet.rs::checksum_rfc1071 (کدجنریشن هم‌ارز).
  - ۴ × `too_many_arguments` با `#[allow]` مستند (امضاهای ۸-پارامتری آینهٔ فلگ‌های سیم TCP هستند؛ refactor ساختار ریسک بی‌مورد دارد).
- **`cargo fmt`** دو انحراف فرمت (fragmentation.rs از clippy-fix و derive دوخطی proxy_cleanup.rs) نرمال شد.

### Fixed — تست‌های داشبورد (uitest، `VERIFIED`)
- **`test-v2rayn.mjs`**: regex پارسِ `default_relay_listen_port`/`default_web_ui_port` فقط بدنهٔ تک‌خطی می‌پذیرفت در حالی که `cargo fmt` (گیت CI) همان بدنه را چندخطی می‌کند — regex به `\s*` تساهل‌پذیر شد و تست «parsed … — got NaN» پاس شد.
- **`test-resilience.mjs`**: دو regex کهنه به API فعلی سورس به‌روز شد — `DnsCache::load` چندخطی در doh.rs و `insert_with_ttl` (TTL authoritative) به‌جای `insert` ساده.
- **`.gitignore`** (شکاف واقعی حریم خصوصی): فایل‌های runtime که کنار exe بیلد-محلی نوشته می‌شوند و IP/آدرس سرور دارند (`dpi_guard.dns_cache`، `dpi_guard.proxy_state`، `dpi_guard.instance.lock`) commit‌نشدنی شدند — تست «the cache file is gitignored» همین را گِرد می‌کرد.
- **`test-settings.mjs`**: فهرست انتظاریِ فیلدهای restart-required ۴ فیلد اسکنر (`edge_candidates`، `enable_sni_scanner`، `sni_candidates`، `sni_rotation_mode`) را کم داشت — راستی‌آزمایی شد که `backend_main` این‌ها را فقط هنگام بوت می‌خواند، پس تگ‌گذاری schema درست است و فهرست تست به‌روز شد (۱۹ فیلد).

## [Unreleased] — ممیزی معماری ۲۰۲۶-۰۹ (TESTS_EXECUTED)

این دور اولین باری است که سوئیت روی کامپایلر واقعی اجرا شده است
(Rust 1.98.1 stable، لینوکس): `cargo test` → **۳۸۳ پاس / ۰ شکست**،
`cargo clippy --all-targets` → **۰ خطا**، `gen_status.py` →
**۴۰ ماژول، ۳۸۳ تست، ۰ تابع مرده**.

### Fixed — `VERIFIED` (اجرا شده روی Rust 1.98.1)
- **`geedge.rs::prepend_grease_extensions`** (بحرانی): طول handshake سه‌بایتی به‌اشتباه به‌صورت u16 بروزرسانی می‌شد (`delta<<8`) و ClientHello خروجی مسیر پیش‌فرض `enable_geedge_evasion` برای هر سرور واقعی خراب بود. وصلهٔ u24 با `checked_add` و خطای صریح. تست رگرسیون: `grease_prepend_keeps_record_parseable`، `padding_after_grease_keeps_record_parseable`.
- **`pipeline.rs::on_inbound`** (بحرانی): ServerHello ورودی entry جدول `recent` را مصرف نمی‌کرد — هر segment تکراری یک `+1` دیگر به استراتژی می‌داد. اکنون با `remove()` یک تلاش = حداکثر یک امتیاز. تست `inbound_serverhello_scores_once` پاس شد.
- **`pipeline.rs` MD5SIG**: آپشن TCP 19 در آفست ثابت `l4+20` نوشته می‌شد و با هدرهای دارای options واقعی (MSS/timestamps) داخل آپشن‌ها/پیلود می‌نوشت. اکنون آفست padding از data-offset پکت wrap‌شده محاسبه می‌شود.
- **`scanner.rs::probe_tls_handshake`**: علاوه بر `0x16`، handshake-type `0x02` (ServerHello واقعی) در بایت ۶ اعتبارسنجی می‌شود؛ رکورد ساختگی middlebox دیگر «سالم» شمرده نمی‌شود.
- **`native_gui.rs`**: نقض قرض‌گیری در دکمهٔ «⚡ Test & Select Lowest Ping Target» (E0500) که کامپایل GUI را می‌شکست.
- **`engine_stub.rs`**: تست `capture_loop` با امضای قدیمی ۳-آرگومتری (E0061).
- **`main.rs`**: تست `redact_lan_and_rewrites_guarantees` برای فرمت فعلی `ep-`+۱۶hex.
- **`quic.rs`**: importهای بلااستفاده (E0433 در بیلد تست).
- **`webui.rs`**: تجمیع کامل هدرهای fragment‌شده قبل از parse (قبلاً Content-Length داخل segment دوم گم می‌شد) + رد صریح `Transfer-Encoding: chunked` با 501.
- **`singleton.rs`**: `truncate(false)` صریح روی هر سه باز فایل قفل + حذف ثابت مردهٔ `LOCK_SH`.
- **`mobile_gateway.rs`**: مقایسهٔ همیشه‌درست `count >= 0` (clippy deny) با نامساوی معنادار.

### Added — `VERIFIED`
- **تقسیم ۱-۲ بایتی TCP روی SNI در مسیر زنده**: `packet::tcp_segment_payload_at_offsets` + سیم‌کشی به `enable_frag_by_sni` (بدون reframing) در `pipeline.rs::apply_client_hello` با seq پیوسته و fallback امن. تست: `frag_by_sni_emits_tcp_one_byte_split`.
- **TTL واقعی DNS**: `doh::parse_a_records` کمینهٔ TTL رکوردهای A را برمی‌گرداند (کف ۳۰s، سقف `MAX_STALE`)؛ `dns_cache::insert_with_ttl` و تازگی per-entry؛ ستون چهارم در فایل کش با سازگاری کامل با فایل‌های ۳-ستونی قدیمی. تست‌ها: `ttl_floor_and_cap_are_applied`، `per_entry_ttl_overrides_default`.
- **توقف نرم بک‌اند از GUI**: فایل `<config>.stop` توسط watcher (چرخهٔ ≤۲۰۰ms) خوانده می‌شود و مسیر کامل graceful shutdown (WinDivert close + restore پروکسی) را اجرا می‌کند؛ دکمهٔ Stop ابتدا graceful، سپس پس از ۳s فال‌بک به kill.
- **گارد Drop بازیابی پروکسی** (`ProxyRestoreGuard` در main.rs): `restore_state` روی هر مسیر unwind/خروج زودهنگام اجرا می‌شود، نه فقط Ctrl+C.
- تست‌های جدید در این دور: ۷ (geedge ×۲، packet ×۲، pipeline ×۱، doh ×۱، dns_cache ×۱).

### Changed — `VERIFIED`
- `scanner.rs`: حذف ثابت‌های مردهٔ `SCAN_PROBES`/`SCAN_TIMEOUT` (بودجهٔ probe از caller می‌آید).
- `config.rs`: کامنت کهنهٔ «بدون TLS probe زنده» حذف و با رفتار واقعی هماهنگ شد.
- `engine.rs`: doc توالی backoff با سلوک واقعی (شروع از 40ms) هماهنگ شد.
- `main.rs`: importهای فقط-ویندوز (`engine`, `webui`, `AtomicU64`) به‌صورت `cfg(windows)` شرطی شدند.
- `native_gui.rs`: hint توکن از «empty = no token» به «empty = auto-generated token» اصلاح شد (رفتار واقعی از همیشه همین بود).

## [Unreleased] — شاخهٔ `arena/01a06e41-sni-spoof-new-5-6`

### Added
- `AI_RULES.md` — قوانین اجباری برای توسعهٔ AI-assisted
- `ARCHITECTURE.md` — گراف وابستگی تولیدشده از سورس
- `TEST_MATRIX.md` + `tools/gen_status.py` — جدول وضعیت خودکار
- `KNOWN_ISSUES.md` — مشکلات تأییدشده
- `.gitignore`
- ۱۹ تست جدید Rust (**اجرا نشده**)

### Fixed — همه `UNTESTED` (کامپایل نشده)
- `lib.rs::build_filter`: `||` → `&&`
- `webui.rs::token_ok`: رد توکن خالی
- `webui.rs::handle_conn`: حلقهٔ خواندن کراندار
- `doh.rs::parse_a_records`: بررسی مرز
- `main.rs`: `recover_mutex` به‌جای `unwrap`
- `self_update.rs`: `validate_repo_slug` + اصلاح repo + حذف ادعای SHA-256
- `singleton.rs`: حذف unlink مسابقه‌ای
- `pipeline.rs`: سقف `last_activity`/`inbound_ttl`
- `autottl.rs`: اصلاح مقیاس وارونه
- `relay.rs`: `FlowHooks` + `FlowSlot` (RAII)

### Changed
- ۱۶ سند قدیمی به `docs/archive/` منتقل شد
- `relay::run()` امضایش عوض شد: `Arc<FlowHooks>` به‌جای closure

### Verified
- `uitest`: ۳۶۹ پاس / ۰ شکست

### Not verified
- `cargo test`, `cargo clippy`, `cargo build` — `cargo` در محیط نبود
- هر رفتار مربوط به ویندوز/WinDivert

---

## تاریخچهٔ git

- `9d77955` Update audit report: all 11 findings now fixed
- `a2e8020` Fix the 5 remaining audit findings
- `8658116` Add full-project audit findings (11 issues: 6 fixed, 5 open)
- `1013c0d` Add line-by-line review report for 2026-09
- `40cdfc0` Fix filter/auth/DoH/mutex bugs found in line-by-line review
- `7a076ac` Add .gitignore so the full uitest suite runs (test-resilience no longer crashes)
- `a2ce27e` Add files via upload
