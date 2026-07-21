# Benchmark: Project ATLAS (Regulatory Advisory Domain)

**System:** Multi-jurisdiction regulatory advisory platform
**Codec:** 112-entry domain dictionary, ASCII-safe prefix notation
**LLM:** Ollama gemma4 (self-hosted)
**Date:** 2026-05-10

---

## Summary

| Metric | Value |
|---|---|
| **Average compression** | **52.1% (2.1x)** |
| **Best** | 77.3% (4.4x) |
| **Worst** | 40.3% (1.7x) |
| **Entries** | 54 |
| **Domains** | 9 |
| **Jurisdictions** | 17 |
| **Tokens saved** | ~44,516 per full knowledge load |

---

## Full Results

```
Domain                Jurisdiction        Raw      LLN   Saving  Ratio
──────────────────────────────────────────────────────────────────────
income-tax            UK                7,860    4,077    48.1%   1.9x
income-tax            UK_Scottish       2,222      936    57.9%   2.4x
income-tax            UK_LossRules      4,066    2,205    45.8%   1.8x
income-tax            US                5,897    2,910    50.7%   2.0x
income-tax            US_StateIncomeTax 2,220    1,001    54.9%   2.2x
income-tax            US_AMT            2,299      990    56.9%   2.3x
income-tax            UAE               2,665    1,198    55.0%   2.2x
income-tax            Singapore         4,022    2,298    42.9%   1.8x
income-tax            Malta             4,572    2,276    50.2%   2.0x
income-tax            France            6,234    3,059    50.9%   2.0x
income-tax            Germany           4,545    2,101    53.8%   2.2x
income-tax            Italy             2,855    1,433    49.8%   2.0x
income-tax            Spain             3,147    1,538    51.1%   2.0x
capital-gains         UK                7,544    3,921    48.0%   1.9x
capital-gains         US                3,940    1,946    50.6%   2.0x
capital-gains         US_Depreciation   3,728    1,690    54.7%   2.2x
capital-gains         France            5,888    2,444    58.5%   2.4x
capital-gains         Germany           3,851    1,569    59.3%   2.5x
capital-gains         Italy             2,893    1,157    60.0%   2.5x
capital-gains         Spain             2,686    1,194    55.5%   2.2x
crypto-tax            UK                4,714    2,200    53.3%   2.1x
crypto-tax            US                5,644    2,716    51.9%   2.1x
crypto-tax            UAE               2,526    1,377    45.5%   1.8x
vat-sales-tax         UK                4,862    2,707    44.3%   1.8x
vat-sales-tax         US                3,665    1,694    53.8%   2.2x
corporate-tax         UK               10,058    5,086    49.4%   2.0x
corporate-tax         US                7,808    4,172    46.6%   1.9x
corporate-tax         Ireland           3,456    2,062    40.3%   1.7x
corporate-tax         Malta             7,762    3,715    52.1%   2.1x
corporate-tax         France            4,916    2,597    47.2%   1.9x
corporate-tax         Germany           4,678    2,257    51.8%   2.1x
corporate-tax         Italy             3,080    1,420    53.9%   2.2x
corporate-tax         Spain             3,093    1,515    51.0%   2.0x
expat-tax             UK               10,235    4,715    53.9%   2.2x
expat-tax             UK_DTATieBreakers 4,553    1,884    58.6%   2.4x
expat-tax             US                6,619    3,542    46.5%   1.9x
expat-tax             UAE              12,480    5,931    52.5%   2.1x
expat-tax             Singapore        16,052    7,279    54.7%   2.2x
expat-tax             Malta            11,614    5,388    53.6%   2.2x
expat-tax             France            7,868    3,194    59.4%   2.5x
expat-tax             Germany           4,678    1,832    60.8%   2.6x
expat-tax             Italy             5,224    2,054    60.7%   2.5x
expat-tax             Spain             4,563    1,744    61.8%   2.6x
estate-iht            UK                6,175    2,953    52.2%   2.1x
estate-iht            UK_Trusts         4,088    1,978    51.6%   2.1x
estate-iht            US                6,791    3,243    52.2%   2.1x
pension-tax           UK               19,675   10,347    47.4%   1.9x
pension-tax           US               19,600    9,730    50.4%   2.0x
property-tax          UK               19,136    9,692    49.4%   2.0x
property-tax          US               14,602    7,429    49.1%   2.0x
property-tax          France            6,578    1,492    77.3%   4.4x
property-tax          Germany           4,312    2,136    50.5%   2.0x
property-tax          Italy             3,666    1,776    51.6%   2.1x
property-tax          Spain             3,921    1,963    49.9%   2.0x
──────────────────────────────────────────────────────────────────────
TOTAL                                 341,826  163,763    52.1%   2.1x
```

---

## Dictionary Structure (categories only)

| Category | Entries | Examples |
|---|---|---|
| Operators | 10 | `->`, `\|`, `;`, `:`, `@`, `#`, `[]`, `GT`, `LT`, `NULL` |
| Temporal | 9 | `EFF:`, `EXP:`, `TY:`, `NOW`, `UPCOMING`, `SUNSET` |
| Structural | 11 | Rule, band, threshold, allowance, rate, deadline, warning |
| Domain abbreviations | 7 | Income-tax, capital-gains, corporate-tax, VAT, etc. |
| Jurisdiction codes | 11 | UK, US, UAE, SG, IE, DE, FR, MT, IT, ES, NL |
| Client profile | 7 | Residency, domicile, income, assets, holdings, filing |
| Domain-specific terms | 40+ | Standard regulatory abbreviations |
| **Total** | **112** | |

---

## Encoding Example

```
RAW (JSON, 610 chars):
{"id":"uk-it-bands","jurisdiction":"UK","domain":"income-tax","taxYear":"2025-26",
 "bands":[{"name":"Personal Allowance","rate":0,"from":0,"to":12570},
  {"name":"Basic Rate","rate":0.20,"from":12571,"to":50270},
  {"name":"Higher Rate","rate":0.40,"from":50271,"to":125140},
  {"name":"Additional Rate","rate":0.45,"from":125141}]}

LLN (291 chars, 52.3% saving):
R:uk-it-bands J:UK D:IT TY:2025-26 NOW
  B:[Personal Allowance 0% 0-12.6K | Basic Rate 20% 12.6K-50.3K |
     Higher Rate 40% 50.3K-125.1K | Additional Rate 45% 125.1K-∞]
  AL:Personal Allowance 12.6K (tapers GT 100K @0.5)
  SRC:[Income Tax Act 2007 | gov.uk/income-tax-rates]
```

---

## Observations

1. **Structured data compresses best** - bands, rates, thresholds achieve 55-65%
2. **Prose resists compression** - notes, descriptions, conditions achieve 40-50% (improved by stop-word stripping)
3. **Temporal metadata adds value** - `NOW`/`UPCOMING`/`HIST` status enables runtime filtering without parsing dates
4. **Multi-jurisdiction is consistent** - 17 jurisdictions all achieve 40-62%, no outlier failures
5. **One anomaly:** Property-tax France (77.3%) - highly structured rate tables with minimal prose

---

*Run: 2026-05-10 | Node.js v25.9.0 | ASCII-safe notation mode*
