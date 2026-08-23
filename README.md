# Reverse Engineering Write-ups

This repository contains my reverse engineering notes, crackme walkthroughs, and malware-analysis practice. It is organized by challenge set so the content is easier to browse and review.

## Repository structure

```text
re-writeups/
├── README.md
├── crackmes/
│   ├── crackme1_by_broken.md
│   ├── easyaf_476f64.md
│   ├── easy_reverse.md
│   ├── muhemed_crackme.md
│   ├── really_easy_elzooms.md
│   ├── secret_key.md
│   ├── terratist_pin_exe.md
│   └── yuris_simple_keygen.md
├── Flare-2014_writeups/
│   ├── challenge_1.md
│   └── challenge_2.md
└── Mandiant_flare/
    ├── Analyzing_PE.md
    ├── Flare-flash_quiz.md
    ├── Mal_analys_basic_tech.md
    ├── Mandant_images/
    └── Labs/
        └── Static_analysis_L1.md
```

## Write-up index

### Crackmes

- [crackme1_by_broken.md](crackmes/crackme1_by_broken.md)
- [easyaf_476f64.md](crackmes/easyaf_476f64.md)
- [easy_reverse.md](crackmes/easy_reverse.md)
- [muhemed_crackme.md](crackmes/muhemed_crackme.md)
- [really_easy_elzooms.md](crackmes/really_easy_elzooms.md)
- [secret_key.md](crackmes/secret_key.md)
- [terratist_pin_exe.md](crackmes/terratist_pin_exe.md)
- [yuris_simple_keygen.md](crackmes/yuris_simple_keygen.md)

### Flare 2014

- [challenge_1.md](Flare-2014_writeups/challenge_1.md)
- [challenge_2.md](Flare-2014_writeups/challenge_2.md)

### Mandiant / Flare training

- [Analyzing_PE.md](Mandiant_flare/Analyzing_PE.md)
- [Flare-flash_quiz.md](Mandiant_flare/Flare-flash_quiz.md)
- [Mal_analys_basic_tech.md](Mandiant_flare/Mal_analys_basic_tech.md)
- [Labs/Static_analysis_L1.md](Mandiant_flare/Labs/Static_analysis_L1.md)

## Progress status

| Category | Item | Status | Notes |
|----------|------|--------|-------|
| Crackme | muhemed_crackme | Complete | Basic static analysis and decode practice |
| Crackme | easyaf_476f64 | Complete | String comparison / simple reversing |
| Crackme | easy_reverse | Complete | String Comparision |
| Crackme | really_easy_elzooms | Complete | TBD |
| Crackme | secret_key | Complete | TBD |
| Crackme | terratist_pin_exe | Complete | TBD |
| Crackme | yuris_simple_keygen | Complete | TBD |
| Crackme | crackme1_by_broken | Complete | TBD |
| Flare 2014 | challenge_1 | In progress | Review and complete notes |
| Flare 2014 | challenge_2 | Planned | TBD |
| Mandiant / Flare | Analyzing_PE | Complete | PE fundamentals review |
| Mandiant / Flare | Flare-flash_quiz | Complete | Notes captured |
| Mandiant / Flare | Mal_analys_basic_tech | Complete | Basic malware analysis concepts |
| Mandiant / Flare | Static_analysis_L1 | In progress | Continue lab notes |

## Notes

- This repo is mainly for learning and documenting malware analysis / reverse engineering workflows.
- Some write-ups focus on crackmes, while others cover PE analysis and reverse engineering fundamentals.
- The structure is kept simple so files can be found quickly without digging through nested content.

## Tools used

- IDA Pro
- x64dbg
- Ghidra
- checksec
- strings
- Python
- REMnux / FlareVM (when applicable)
 