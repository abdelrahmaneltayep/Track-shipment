# PDV2-839 · Shipping routes: filter destinations by carrier coverage

**Owner:** Abdelrahman Eltayep · **Reporter / PM:** Idris Yahya Bakur Goni · **Status on board:** New Request (left unchanged)
**Sources read:** PDV2-839 · SI-381 (the idea) · SHPD-12080 epic → stories SHPD-12081 / 12083 / 12085, tech task SHPD-12087, design task SHPD-12089 · SHPD-11893 (checkout coverage) · SI-358 (Rules → Routes migration) · SI-374 · SI-315 · PDV2-231 (routes redesign) · the PM's Loom of `s.salla.sa/shipping/setting/routes/new` (20 Oct 2026, 1:47, read frame by frame)

**Source key:** `S` spec/Jira · `H` the PM's Loom (live product) · `M` migration data in SI-358 · `F` Fanous · `I` Intercom · `Y` YouTrack · `O` Omni

> ⚠️ **Gaps in this run.** I could not reach Fanous, Intercom or Omni from this session. The Loom's audio could not be transcribed (the network blocked the speech-model download), so everything tagged `H` comes from the screen alone. Every voice and scale claim below is marked as a gap. None of them is filled with an estimate.

---

## Beat 1 · Analysis

### 1. Summary

Merchants build shipping routes. They choose carriers and destinations independently, and nothing stops them saving a route to a city that none of its carriers prices. The ticket asks to make routes valid by construction, keep them valid when coverage changes, and clean up existing routes without silently changing any data. "Covered" means the carrier is active in the store and has a price for that city in the route's market. This is the same definition SHPD-11893 uses at checkout.

**The PM's proposed solution, set aside as candidate #0 (wording from SI-381 / SHPD-12081..12085):**
> Destination list shows only cities covered by ≥1 selected carrier (union). Uncovered cities are **hidden** from the picker. The list is disabled until a carrier is picked. Removing a carrier **auto-removes** the cities it alone covered, with a notice (first 5 names + «و X أخرى»), and re-adding the carrier restores them. Save is blocked server-side (dashboard + API). Existing routes get a page banner, a «مدن غير مغطاة» tag, and an edit mode with «إزالة المدن غير المغطاة» / «إضافة شركة شحن». Proposed form order: carriers → destinations.

The prototype builds this as concept 0, and it gets scored the same way as every other concept.

### 2. What the evidence says

| # | Claim | Sources | Count |
|---|---|---|---|
| E1 | In the live form, **carriers already come before destinations.** The order is name → market → route type → quick delivery → carriers, and then the optional «شروط تطبيق المسار». | `H` (0:05–0:55) | 1, but direct observation |
| E2 | **The live form has no top-level "destination" field.** A destination is an *optional condition* («الطلب من مناطق محددة») with **allowed regions** and **excluded regions**, each set at country → city → **district** level. | `H` (1:04–1:57) | 1, but direct observation |
| E3 | The spec describes the form as "market → origin → destination → carriers → conditions". That does not match the live form (see E1 and E2). | `S` SI-381, SHPD-12081 vs `H` | 2 that disagree |
| E4 | The country picker inside the region condition is **not tied to the route's market**. The PM picked تونس with no market selected, and the list offers السعودية / الإمارات / الكويت / قطر / البحرين. | `H` (1:12–1:20) | 1 |
| E5 | The city list labels each city with its region (e.g. «الأحساء - المنطقة الشرقية»), so region-level grouping data already exists. | `H` (1:51) | 1 |
| E6 | Route type has three modes: تحديد الشركات · تعيين تلقائي · تسمية خاصة. Only تسمية خاصة is excluded from filtering in the spec. The spec doesn't say how coverage works for تعيين تلقائي (automatic assignment). | `H`, `S` SHPD-12087 | 2 |
| E7 | SHPD-11893, which uses the same coverage definition, decided to **show uncovered results and mark them rather than hide them**: «إخفاؤها يجعل العميل يظن أن البحث لا يعمل». Candidate #0 hides them. | `S` SHPD-11893 vs SHPD-12081 | 2 that disagree |
| E8 | Migrated routes are a known source of broken configurations. 19 active legacy rules have zero carriers attached and 42 have no condition. | `M` SI-358 | 1, quantified |
| E9 | A carrier that prices **all cities** makes every city covered (SHPD-11893 edge case). For any route that includes such a carrier, the filter changes nothing. | `S` SHPD-11893 | 1 |
| E10 | The spec already plans a read-only baseline script that counts routes with incomplete coverage. | `S` SHPD-12087 | 1 |
| E11 | "No shipping company available" in cities that are part of a route: **[TBD in the ticket]**. Support tickets about routes not working for a city: **[TBD]**. | `S` | **0. This is a hypothesis.** |

### 3. Missing information (gaps stay gaps)

1. **Scale:** how many active routes contain at least one uncovered city today, and how many stores have them. The script is specced (SHPD-12087) but hasn't been run. Without it, the primary KPI "→ 0 in 30 days" has no baseline.
2. **The bigger, unspecced population:** how many routes have **no region condition** (so they apply to the whole market) and include a carrier with city-specific pricing. Those routes are uncovered by definition, and the brief never mentions them (see section 5).
3. **Pricing shape:** what share of active carrier pricing is city-specific and what share is "all cities". If most carriers price nationally, the filter rarely activates and the impact is small (E9).
4. **Voice:** no Intercom quotes, Fanous results or YouTrack IDs. I need to run Fanous in Arabic («مسار الشحن لا يعمل لمدينة», «لا يوجد شركة شحن متوفرة مسار») and in English.
5. **Location in the funnel:** no PostHog view showing how often merchants add a region condition to a route, or how many cities they add.
6. **District-level conditions:** the form allows district picks, but coverage is per city. How many saved routes go down to district level?

### 4. Technical risks, edge cases, contradictions

**Contradictions (the ones that matter):**
- **C1. Spec vs. live form (E1–E3).** The open question "should the form order change to carriers → destinations?" is already settled in production, because carriers come first. The real question is different: the destination is an *optional* condition, so most routes may have no destination list to filter.
- **C2. Hide vs. mark (E7).** Two stories in the same quarter, built on the same coverage definition, made opposite UX decisions. Pick one pattern for both the merchant and the customer surfaces.
- **C3. Ownership.** PDV2-839 is assigned to Abdelrahman. SHPD-12089, the design task with a 12 Nov deadline, names **Mahmoud Fekry** and is unassigned. **PDV2-231** («إعادة ترتيب وتبسيط تجربة ميزة المسارات», Abdelrahman, Full Flow Design) is redesigning the same form, including condition order. Two design streams are touching one form.
- **C4. "No silent change" vs. auto-remove.** SHPD-12083 auto-removes cities from the form when a carrier is removed. It isn't saved until «حفظ», but it still changes the merchant's input without their say. The stated principle is no silent change.

**Edge cases the spec does not cover:**
- **Excluded regions** should *not* be filtered. Excluding an uncovered city is harmless and may be intentional.
- **District picks** under a covered city: allow them, but city coverage doesn't prove district coverage (open question 4 in SI-381).
- **Condition country ≠ route market** (E4): the city list has to be scoped to the market before filtering means anything.
- **«عند تحقق شرط واحد على الأقل» (OR logic):** if region is OR'ed with a product condition, the route can fire for an uncovered city through the product branch. Validation has to apply to the route's *effective* area, not only to the region list.
- **تعيين تلقائي:** which carrier pool does coverage come from?
- **Pricing that grows:** if a carrier adds a city's price later, nothing should be added to the route automatically (SHPD-12083 already says this).
- **Scale:** a store with 6 carriers and 200+ cities. The removal notice and any per-city chips must stay readable.
- **API save rejection** for existing integrations that post routes. This is a breaking change for API clients, so it needs a versioning or deprecation note.

### 5. The problem in one sentence

> **A shipping route can promise delivery to places its carriers don't price, either explicitly through a region condition or implicitly by having no region condition at all, and the merchant gets no signal until a customer hits "no shipping company available".**

**What changed from the brief:** the brief frames this as a *destination-list* problem, so the fix it implies is to filter the picker. The live form (E2) shows destinations are optional, so the gap covers the **whole area the route applies to**, not just the list. A route with ريدبوكس and no region condition applies to every city in the market, and ريدبوكس covers a handful. Filtering a list that the merchant never opens doesn't fix that route. This is the most important finding to bring to the kickoff.

**GMV lever (one):** OPS – Conversion Rate (Storewide) → Orders per Seller. This matches the brief and I see no reason to change it.

**7-Point Building Dashboard lens (pre-ideation):** Flows simplified: route creation, region condition · Steps: the merchant currently checks carrier pricing outside the form, and that step can go · Differentiated: show coverage at the moment of configuration (I haven't benchmarked whether Shopify zones do this) · Happy moment: «مسارك يوصل لكل المدن التي اخترتها» · Re-purpose: reuse the existing «الملخص» side panel and the SHPD-11893 coverage endpoint · AI: suggest the carrier that closes the most coverage gaps · Cross-feature: carrier pricing → routes → checkout gate (SHPD-11893) → AWB.

---

## Beat 2a · Questions for Idris (Arabic, numbered, most blocking first)

> Each question says what I already found and what the answer decides.

1. **نطاق المشكلة — المسارات بدون شرط مناطق.** في الفيديو، الوجهة شرط اختياري («الطلب من مناطق محددة») داخل «شروط تطبيق المسار». المسار بدون هذا الشرط يُطبّق على كل مدن السوق، ولو فيه شركة تسعّر لمدن محددة فهو غير مكتمل التغطية تلقائياً. **هل هذه المسارات داخل النطاق؟**
   → إذا نعم: الحل يجب أن يُظهر «نطاق المسار الفعلي» وليس فقط تصفية القائمة. إذا لا: نكتب افتراضاً صريحاً في الـ Epic أنها خارج النطاق.
2. **خط الأساس.** سكربت SHPD-12087 مكتوب لكنه لم يُشغّل. هل يمكن تشغيله قبل الاجتماع، مع تقسيم: (أ) مسارات بشرط مناطق فيه مدن غير مغطاة، (ب) مسارات بدون شرط مناطق مع شركة تسعيرها محدود؟
   → يحدد أي الحالتين نصمم لها أولاً، ويعطي مؤشر «→ 0 خلال 30 يوم» رقماً يبدأ منه.
3. **إخفاء أم تمييز؟** SHPD-11893 قرر «نتائج البحث خارج النطاق تظهر ولا تُحذف… إخفاؤها يجعل العميل يظن أن البحث لا يعمل». SHPD-12081 يقرر الإخفاء. **أيهما الالتزام؟**
   → إذا التمييز: المدن غير المغطاة تظهر معطّلة مع السبب. إذا الإخفاء: نعتمد نص البحث فقط.
4. **من يملك التصميم؟** SHPD-12089 يذكر Mahmoud Fekry بموعد 12 نوفمبر وهو غير مُسند، وPDV2-839 مُسند لي. وكذلك PDV2-231 (إعادة ترتيب تجربة المسارات) عندي في Full Flow Design ويمس نفس النموذج. **هل أملك الاثنين، وهل نوحدهما في ملف واحد؟**
   → يحدد هل أبني على ترتيب PDV2-231 الجديد أم على النموذج الحالي.
5. **الإزالة التلقائية.** SHPD-12083 يزيل المدن من النموذج تلقائياً عند إزالة شركة. هل يُقبل بديل «معاينة قبل الإزالة» (تُخبر التاجر بما سيخرج قبل أن يؤكد)، اتساقاً مع مبدأ «لا تغيير بدون علم التاجر»؟
   → إذا نعم: تأكيد قبل التنفيذ. إذا لا: إزالة + تنبيه كما في القصة.
6. **المناطق المستثناة.** هل تُستثنى من التصفية؟ اقتراحي: نعم، استثناء مدينة غير مغطاة لا يضر.
   → يحدد هل قائمة «مناطق مستثناة» تبقى كاملة.
7. **دولة الشرط مقابل سوق المسار.** في الفيديو تم اختيار «تونس» داخل الشرط بدون تحديد سوق. هل دولة الشرط يجب أن تُقفل على سوق المسار؟
   → شرط مسبق للتصفية؛ بدونه التغطية تُحسب لسوق ومدن الشرط من دولة أخرى.
8. **«تعيين تلقائي».** من أين تأتي التغطية في هذا النوع، من كل الشركات المفعّلة أم من قائمة محددة؟
   → يحدد هل نعرض التصفية في هذا النوع أصلاً.
9. **منطق «شرط واحد على الأقل».** إذا كان شرط المناطق مع شرط منتجات بمنطق OR، فالمسار قد يُطبّق على مدينة غير مغطاة عبر شرط المنتجات. هل التحقق عند الحفظ يشمل هذه الحالة؟
   → يحدد صياغة قاعدة التحقق في SHPD-12087.
10. **الـ API.** رفض الحفظ عبر API تغيير غير متوافق مع التكاملات الحالية. هل هناك فترة سماح أو تحذير قبل الرفض؟
    → يحدد هل نصمم حالة «تحذير» بجانب حالة «رفض».

---

## Beat 2b · 45-minute kickoff agenda (no concepts on screen)

| # | Block | Min | My opening position | Written live |
|---|---|---|---|---|
| 1 | Problem alignment | 5 | Read section 5 aloud and show frame 1:04 (destination is an optional condition) | One shared sentence |
| 2 | Evidence | 8 | E1–E11. E11 has zero sources, so the size of the problem is unproven | Findings vs. hypotheses; who closes the gaps |
| 3 | Scope | 8 | **In:** new and edited routes with region conditions; whole-market routes *flagged*; existing routes flag-and-fix. **Out:** checkout changes, QD system routes (a reason is already in the ticket). **Not now:** auto-cleanup of migrated routes (trigger: baseline shows more than X routes) | In / Out / Not now |
| 4 | Metric | 7 | "Active routes with uncovered cities move from **[BASELINE?]** to 0 within 30 days, measured in **[Omni?]**." Guardrail: active routes per store doesn't fall | One KPI sentence |
| 5 | Data & access | 5 | Run the SHPD-12087 script, PostHog event for region-condition usage, a Fanous sweep | Owner + date per pull |
| 6 | Open questions | 8 | Q1 → Q4 → Q3 → Q5 in that order | Answered / deferred / assumption |
| 7 | Decisions | 4 | — | decision · owner · date · what would reverse it |

**Likely pushback, and my answers:**
- *"Whole-market routes are out of scope, keep it small."* → Then write it on the epic as an explicit assumption and keep the flag (story 3) for them. A flag costs a tag; leaving them out entirely leaves the most common broken shape unfixed.
- *"Hiding is cleaner."* → Hiding is cleaner until a merchant searches for جدة, finds nothing, and opens a support ticket. SHPD-11893 already rejected hiding for this exact reason.
- *"The design is Mahmoud's."* → Fine. Then this analysis goes to him and I limit PDV2-839 to the PDV2-231 alignment.

**Decision log (empty):** decision · owner · date · what would reverse it
