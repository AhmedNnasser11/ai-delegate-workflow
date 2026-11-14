# ai-delegate-workflow (بالعربي)

workflow هندسي للـ AI مبني على [delegate-skills](https://github.com/amElnagdy/delegate-skills).

**Orchestrator واحد بيخطط ويقرر ويراجع ويعمل commit، وCLIs أرخص بتكتب الكود.**

اتطلّع من مشروع حقيقي (Ecommerce SaaS monorepo) بس مفيش أي حاجة هنا بتعتمد عليه. انسخ الأفكار والـ templates لأي ريبو.

---

## 1. المشكلة

لما agent واحد يعمل كل حاجة في جلسة واحدة:

- موديل غالي بيصرف tokens على شغل مكرر (ترجمات، نسخ شاشات موجودة، tests بسيطة)
- نفس الـ agent اللي كتب الكود هو اللي بيراجعه
- الـ context بيتملي قراءة ملفات وشات
- "الاختبارات نجحت" كلام مش دليل
- مفيش حاجة بتمنعه يعدّل ملفات حساسة (schema, auth, config)

## 2. الفكرة

نقسّم الأدوار:

| الدور | مين | بيعمل إيه | عمره ما بيعمل |
|---|---|---|---|
| **Orchestrator** | الجلسة الرئيسية (مثلاً Claude Code) | يخطط، يقرر، يكتب brief، يراجع الـ diff، يعيد الفحوصات، يحدّث الدوكس، **يعمل commit** | يوزّع الشغل الحساس |
| **Implementer** | CLI تاني (Codex, OpenCode, Cursor Agent, Aider...) | يعدّل الكود في الـ working tree بناءً على brief | يخطط، يقرر، يراجع، يعدّل دوكس، يعمل commit |

ناتج الـ implementer هو **git diff**، والـ orchestrator بيراجعه زي أي pull request.

## 3. المصطلحات

- **delegate-skills** — مكتبة skills مفتوحة المصدر (`*-delegate` و`delegate-setup`). كل skill بتلف CLI واحد في سكريبت Node صغير اسمه `relay.mjs`.
- **Relay** — بيشغّل الـ implementer، يستنى، ويكتب `result.json` منظم (status, exit code, التقرير النهائي, الملفات اللي اتغيرت, session id). **عمره ما بيعمل commit.**
- **Lane** — ربط نوع شغل بـ implementer واحد + إعدادات (model, effort). أمثلة: `feature`, `tests`, `ui`, `i18n`.
- **Fleet** — مجموعة الـ lanes بتاعتك. بتتعمل مرة واحدة بـ `$delegate-setup` اللي بيكتشف الـ CLIs المتسطبة ويقترح lanes ومبيكتبش إلا بعد موافقتك.
- **Brief** — وصف مهمة مستقل بذاته، لأن الـ implementer مبيشوفش شات الـ orchestrator.
- **Delta brief** — رسالة قصيرة لنفس الـ session (`--session <id>`) عشان تصلّح ملاحظات المراجعة.
- **BASE** — الـ commit SHA قبل التشغيل، بنقارن بيه بعد التشغيل.
- **Protected paths** — ملفات عمرها ما بتتوزع (schema/migrations، packages مشتركة، auth، إعدادات الـ workflow، الدوكس).

## 4. اللوب

```
خطة ─▶ موافقة ─▶ Brief ─▶ Dispatch ─▶ result.json ─▶ مراجعة الـ diff ─▶ Verify ─▶ Commit
main    إنت      main     relay        relay          main              main      main
```

1. **الخطة** — الـ orchestrator يكتب خطة فيها *جدول delegation* (مهمة ← ملفات ← lanes) وإنت توافق.
2. **Brief** — brief لكل مهمة: الهدف، المسارات بالظبط، ملف مرجعي، شروط القبول، ومقتطف الخطة اللي الـ agent محتاجه بس.
3. **Dispatch** — `git status` نضيف، `BASE=$(git rev-parse HEAD)`، شغّل الـ relay في الخلفية، تشغيل واحد في كل مرة.
4. **اقرأ `result.json`** — الـ status والملفات والرسالة النهائية. كلامه عن الـ gates مش دليل.
5. **المراجعة** — شغّل فحص الـ diff قصاد `$BASE`، واقرأ الـ diff كله قصاد الـ brief (زيادة عن المطلوب؟ حاجة ناقصة؟). التصليح يترجع كـ delta brief.
6. **Verify والـ commit** — الـ orchestrator يعيد typecheck/lint/tests مرة واحدة على الـ diff النهائي، يحدّث الدوكس، ويعمل commit.

## 5. الـ Routing: مين ياخد إيه

| الشغل | الأول | التاني | الأخير |
|---|---|---|---|
| مفاتيح i18n (الصياغة متقررة) | lane رخيصة | lane تانية | Claude agent صغير |
| نسخ sibling معين (≤5 ملفات، من غير logic جديد) | lane رخيصة | lane تانية | Claude agent صغير |
| logic وhooks وservices وtests وendpoints جوه contract معتمد | lane أساسية | lane أقوى | Claude agent متوسط |
| بحث قراءة فقط واسع | – | – | explorer agent للقراءة |
| معمارية، auth، دفع، schema، أمان، التحقق النهائي | – | – | **الـ orchestrator بس** |

قواعد:

- **قاعدة 0** — لو أسرع تعمله من إنك تكتب له brief (≤3 ملفات) الـ orchestrator يعمله بنفسه. الـ agent بيبدأ من الصفر وإنت بتدفع تمن الـ brief.
- **قاعدة 1** — أي حاجة تكلفة الغلط فيها عالية (auth, tenant, دفع, migrations, سلامة البيانات) → الـ orchestrator.
- **قاعدة 2** — القرار المفتوح بيتاخد الأول من الـ orchestrator وبعدين يتوزّع الشغل.
- **الـ fallback** — انقل للعمود اللي بعده لو الـ CLI مش موجود، الكوتا خلصت، الـ status مش ok، أو فحص ما بعد التشغيل فشل بعد delta brief واحد. فشلين على نفس المهمة → الـ lane اللي بعدها، وبعدين الـ orchestrator.

## 6. الـ Guardrails

1. **مسارات محمية** الـ orchestrator مبيديهاش لأي lane.
2. **حدود الـ brief** — كل brief بيقول: ممنوع commit/push/install، ممنوع المسارات المحمية، شغّل الـ gates دي.
3. **فحص diff بعد كل تشغيل** — سكريبت بياخد `BASE` ويكشف: الـ HEAD اتحرك (يعني الـ implementer عمل commit)؟ مسارات ممنوعة اتلمست؟ مخالفات strictness على السطور المضافة؟
4. **اللي بيراجع غير اللي كتب** — الكاتب مبيراجعش شغله. عمق المراجعة على حسب التعقيد (منخفض: inline، متوسط: الـ orchestrator يقرأ الـ diff كله، عالي: reviewer agent منفصل).
5. **Ledger (متقدم، اختياري)** — بيتسجل خريطة المهمة←الملفات←lane، وhook بيمنع الـ orchestrator من كتابة ملفات تخص lane بنفسه، وفحص نهائي بيقارن hashes الملفات بالحالة اللي كل lane سابتها.

حدود لازم تعرفها: الـ read-only مختلف من CLI للتاني، و`touchedFiles` مبني على `git status` (مش بيشوف ملفات الـ ignored ولا الكتابة برا الريبو). الـ ledger بيكشف الانحراف، مش حماية أمنية ضد عملية شغالة بنفس يوزر النظام. **راجع الـ diff، مش التقرير.**

## 7. تبدأ إزاي

```bash
npx skills add amElnagdy/delegate-skills --skill delegate-setup
npx skills add amElnagdy/delegate-skills --skill codex-delegate
```

المتطلبات: Node 18+، git، كل CLI عامل login، وagent يقدر يشغّل أوامر shell.

1. قول للـ agent: `Use $delegate-setup to discover my installed implementer CLIs and create a fleet for feature, tests, and UI work.` ووافق على الـ lanes المقترحة.
2. الصق [templates/CLAUDE.md.snippet](templates/CLAUDE.md.snippet) في `CLAUDE.md` (أو `AGENTS.md`) بتاع الريبو.
3. حدد المسارات المحمية وسكريبت فحص الـ diff.
4. وزّع: `Use $codex-delegate --lane feature to implement <task>, then review and commit it.`

## 8. إيه اللي في الريبو

| المسار | إيه هو |
|---|---|
| `docs/01-concepts.md` | الأدوار والمصطلحات |
| `docs/02-the-loop.md` | اللوب بالتفصيل |
| `docs/03-routing.md` | جدول الـ routing وقواعده |
| `docs/04-guardrails.md` | المسارات المحمية، فحص الـ diff، مستويات المراجعة، الـ ledger |
| `docs/05-setup.md` | التسطيب والربط |
| `templates/CLAUDE.md.snippet` | قواعد جاهزة للـ orchestrator |
| `templates/delegation-table.md` | قسم الخطة اللي بيوزّع الـ lanes |
| `templates/brief.md` | قالب الـ brief |
| `templates/review-checklist.md` | قايمة المراجعة قبل الـ commit |

## 9. الشكر

الـ skills والـ relays وإعداد الـ fleet من [amElnagdy/delegate-skills](https://github.com/amElnagdy/delegate-skills). الريبو دي بتوثّق workflow مبني حواليها بس.

الرخصة: MIT
