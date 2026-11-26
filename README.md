# LabZero Baby v0.4 (pilot build)

Newborn and infant care simulation for health trainees, and the next chapter of LabZero Mama. It runs on the LabZero Mama engine and is set in the same fictional clinic (Ikhwezi Community Health Centre). It follows the same family: Lwazi, the 31-week baby born to Thembeka Ngcobo in Mama, grows up across the linked cases.

- Live build (Claude artifact): https://claude.ai/artifact/XDeAqdoqwG4kyKoMhD4vbH
- Single-file build in this folder: `labzero-baby-v0.4.html`. Open it in any browser; it works offline.
- Licences: code MIT, case content CC BY 4.0. Fictional cases for training only. Not for clinical use.
- Status: prototype. Clinical content is a draft pending SMU clinical review and sign-off (see "Review" below).

## Folder layout

```
labzero-baby-v0.4/
  labzero-baby-v0.4.html   built, self-contained app (what is published)
  assemble.py              build script: patches the Mama engine and inlines clinic + cases
  src/
    head.html              Mama engine part 1 (HTML, CSS, settings panel)
    engine.html            Mama engine part 2 (dialogue, resus, drills, scoring, UI)
    clinic.js              Ikhwezi baby clinic: lobby, Lwazi's thread, first queue patients
    clinic_more.js         rest of the day: queue, collapse, surge, taxi, maternity unit, wards, emergency room
    case_*.js              one file per case (29)
    review.js              review label and open reviewer questions per case
  tests/
    test_perfect.py        plays every case with the best choices: expect 100 and no critical errors
    test_run.py            plays every case with weak choices: expect fail, no stuck states
    test_claude_mock.py    Claude matching, Claude voice and mic with mocked capabilities
  tools/
    option_length_audit.js checks that correct answers are not systematically the longest option
    rebalance*.py          one-off wording fixes used to balance option lengths
```

## Build and test

```
python3 assemble.py                 # writes index.html next to assemble.py
pip install playwright && playwright install chromium
python3 tests/test_perfect.py       # 29 cases, each "100 | no critical"
python3 tests/test_run.py           # weak play fails every case
node tools/option_length_audit.js   # v0.4: correct answer longest in 29 of 154 sets
```

`assemble.py` reads `src/head.html` and `src/engine.html`, which are split from LabZero Mama. It applies small, labelled text patches with `rep()`, each of which fails loudly if its anchor is missing, then appends clinic and case scripts in a fixed order. To change the engine, edit the patch in `assemble.py`, not the built HTML. Every patch is commented with the date and reason.

## Case schema (same as Mama)

Each case is `window.MAMA_CASES.push({...})` with these fields:

- **Identity:** `id`, `title`, `family`, `why`, `setting`, `patient`, `opening`
- **Steps:** `steps` is an ordered list. Standard steps are `vitals`, `history`, `exam`, `triage`, `tests`, `results`, `diagnosis`, `meds`, `manage` and `followup`. A key from `scenes`, `drills` or `resus` can also be a step.
- **Wording:** `stepLabels`, `stepSettings`, `vitalsTitle`/`vitalsLead`, `examTitle`/`examLead`
- **Scoring:** `domains` and `sceneDomains` (scoring areas), `tuto` (tutor prompts per step), `monitor(s)` (top bar)
- **Content:** `vitals[]`, `history{speaker, speakerRole, intents[], social, fallback}`, `exam[]`, `triage{options}`, `tests[]`, `results{reports, significant, criticalNote}`, `diagnosis{options}`, `medicationChart{lines}`, `management{options}`, `scenes{turns}`, `drills{actions, rules, demo}`, `followup{options}`
- **Debrief:** `extraFeedback`, `exampleErrors`, `learningPoints`

Scoring: an option's `score` (0–100), an item's `critical` / `flag` text for critical errors, `essential` items, `harmful` management choices, and `error: true` medication lines. Pass rule: 70 or more, no critical errors, and no domain below zero. Options are shuffled at runtime. Option wording is kept near-identical in length, so length never gives the answer away.

## The 29 cases

| Where | Case id | Topic |
|---|---|---|
| Lwazi's thread (from Mama) | too-quiet-day27 | Late-onset GBS sepsis, day 27 |
| | six-week-visit | Routine 6-week visit, corrected age, vaccines |
| | missed-visit-12wk | Catch-up vaccines; mother's HIV retest positive; infant PCR and prophylaxis |
| | pneumonia-4m | Severe pneumonia in an HIV-exposed infant, PCP cover |
| | diarrhoea-7m | Some dehydration: Plan B, zinc, vitamin A catch-up |
| | first-birthday-12m | 12-month visit: MR 2, vitamin A, deworming, HIV plan, burns |
| Baby clinic queue | zero-dose-7m | Zero-dose catch-up in a measles outbreak |
| | bronchiolitis-6wk | RSV bronchiolitis with apnoea |
| | pertussis-7wk | Whooping cough |
| | jaundice-day4 | ABO haemolysis, exchange transfusion |
| | meningitis-12d | E. coli meningitis |
| | vkdb-5wk | Late vitamin K deficiency bleeding |
| | cchd-day6 | Transposition of the great arteries, prostaglandin |
| | alcohol-14wk | Microcephaly, possible prenatal alcohol exposure |
| | sam-9m | Complicated severe acute malnutrition |
| Clinic emergencies | choking-10m | Infant choking (timed drill) |
| | abusive-head-trauma-4m | Abuse: social worker, SAPS FCS, Children's Act s110, Form 22 |
| | paraffin-11m | Paraffin ingestion; social worker, no police |
| | tetanus-day8 | Neonatal tetanus (taxi arrival) |
| Maternity unit (MOU) | birth-asphyxia-cord | Cord prolapse, resuscitation, cooling |
| | shoulder-dystocia-idm | Shoulder dystocia, diabetic mother (Shireen from Mama) |
| | spina-bifida-birth | Myelomeningocele |
| | duodenal-atresia-ds | Duodenal atresia with Down syndrome |
| | hiv-exposed-birth | HIV-exposed newborn at high risk |
| | congenital-syphilis-day1 | Symptomatic congenital syphilis |
| Kangaroo ward | nec-twin-a | Necrotising enterocolitis (Precious Dube's twins, next to Thembeka) |
| | preterm-feeds-twin-b | Preterm feeds, apnoea, caffeine |
| Emergency room | drowning-10m | Drowning (infant CPR drill), social worker |
| | road-injury-8m | Taxi crash, haemorrhagic shock, SAPS, J88 |

**Unlocking:** 16 cases are in the waiting room at the start. The rest unlock during the day:

1. The maternity unit, the wards and the emergency room open after the first completed visit.
2. The collapse (choking) opens at the same time.
3. The surge opens after the collapse case is followed.
4. The taxi arrival opens after a surge case is followed.

Reviewers can press "Reviewer shortcut: open all 29 now" in the lobby, or use "Educator: open a case directly".

## Patient dialogue

1. **Offline matcher, always on.** Typed questions are matched to the case's approved answers (`history.intents`) using keywords, synonyms and typo tolerance. No data leaves the device.
2. **Claude matching (artifact capability `sample`).** Claude is used when the offline match fails or is unsure. Claude either returns the id of an approved answer or, for an off-script question, gives a short in-character reply that adds no clinical fact. Scoring uses only approved answers. The chat tags each reply "understood by Claude" or "off script".
3. **Voice.** The default is the Claude voice (capability `tts`), with separate voices for women and men set in Settings and a preview button. The fallback is device speech, which matches the speaker's gender and prefers South African, Nigerian or Kenyan English voices.
4. **Dictation (capability `dictation`).** The 🎤 Speak button by the question box.

The three Claude capabilities only work when the page is opened on claude.ai by viewers in the owner's organisation. Dictation also needs a Chromium browser or the Claude desktop app, and the Claude voice doesn't work in Safari or the mobile apps. Everywhere else the app falls back to offline matching and device speech. The optional OpenAI-compatible endpoint from Mama is still in Settings.

## Learner record

Results are stored locally (key `labzero-baby`, schema `labzero-baby/learner-record@0.1`) and can be exported from the dashboard. Dialogue entries record the method used: suggested, local, claude, claude-off, social or unmatched.

## Review

Every case carries the label "Prototype · draft case content · SMU clinical review and sign-off pending". Open questions per case are in `src/review.js` and appear in the app's review panel. The SMU review sheets document lists them for reviewers.

## Changes since v0.1

- **v0.2:** expanded from 4 to 20 cases. Added the maternity unit, the kangaroo ward, and the collapse, surge and taxi events. Rebalanced option lengths.
- **v0.3:** added 9 cases (Lwazi at 4, 7 and 12 months; congenital syphilis; HIV-exposed newborn; prenatal alcohol; SAM; drowning; road injury) and the emergency room. Added the case-count banner and the open-all shortcut.
- **v0.3.2:** device speech matches the speaker's gender, with African English accents.
- **v0.4:** smarter offline matching, Claude matching with off-script replies, Claude voice and dictation.
