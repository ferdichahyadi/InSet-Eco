# InSet-Eco — An Adaptive Indonesian Sentiment Lexicon for the Maritime Ecosystem Domain of the Riau Islands

**Version 1.0 · 20 September 2026** (documentation revised 29 September 2026)

Authors: Ferdi Chahyadi (principal investigator), Risdy Absari Indah Pratiwi,
Dhani Akbar — Universitas Maritim Raja Ali Haji, Tanjungpinang, Indonesia.
Funded by the UMRAH Fundamental Research Grant 2026.
License: **MIT** (`LICENSE`). InSet-Eco incorporates the entries of InSet (Koto &
Rahmaningtyas 2017), which is also distributed under the MIT License, as
confirmed by its first author, Fajri Koto, in correspondence on 29 September
2026; InSet's license text is reproduced in `LICENSE-InSet.txt` and must
accompany any copy. Users are asked to cite both InSet-Eco and InSet — see §11.

Paper: F. Chahyadi, R.A.I. Pratiwi, D. Akbar, N. Hamid, A. Baihaqi, *BIO Web of
Conferences* **262**, 03011 (2026), https://doi.org/10.1051/bioconf/202626203011.

---

## 1. Summary

InSet-Eco is an Indonesian sentiment lexicon built on InSet (Koto &
Rahmaningtyas, 2017) and adapted to online reviews of coasts, beaches,
mangroves, and fishing areas in the Riau Islands. It holds **12,717 entries** —
single words and two- or three-word phrases — each with one sentiment score on
a scale from **−5 to +5**.

It differs from InSet in three ways:

1. **Domain vocabulary.** 2,926 new entries absent from InSet, scored from
   6,124 star-rated Google Maps reviews, including 2,169 phrases such as
   *tidak ada* ("there is none", −5.0), *pasir putih* ("white sand", +5.0), and
   *banyak sampah* ("a lot of litter", −5.0).
2. **Non-standard forms.** 1,386 informal spellings, abbreviations, and
   elongations (*bgt*, *mantab*, *takde*, *enakkkkk*) that borrow the score of
   their standard form, so the lexicon also works on raw text.
3. **Revisions to InSet.** 128 words whose InSet score contradicts their use in
   this domain were rescored (*berkunjung* "to visit" −4 → +5; *mahal*
   "expensive" +1 → −5), and 46 function words (*yang* −5, *dari* −3) were
   neutralized to 0.

In three-class, lexicon-only sentiment classification, InSet-Eco raises
macro-F1 from 0.338 (InSet) to 0.436 (+29%; p < 0.0001 with the corrected
resampled t-test; all 25 paired evaluations), and negative-class F1 from 0.085
to 0.320.

---

## 2. Files

| File | Contents | Intended use |
|---|---|---|
| `lexicon/inset_eco_full.tsv` | Full lexicon, 9 columns | **Master file.** For research, auditing, and further development |
| `lexicon/inset_eco_compact.tsv` | Two columns `word`, `weight` (integer), 11,351 entries | Derived file in exactly InSet's format, for code already written for InSet |
| `lexicon/oov_corrections_validated.tsv` | 1,519 non-standard → standard pairs from manual annotation | Spelling-correction resource; usable on its own |
| `lexicon/oov_correction_map_all_layers.tsv` | All 1,956 corrections applied to the text, in three layers | Reproduces the corrected text |

All files are tab-separated (TSV), UTF-8, with a header row.
`inset_eco_compact.tsv`, `positive.tsv`, and `negative.tsv` are derived automatically
from the master file; where they differ, the master file prevails.

### Columns of `inset_eco_full.tsv`

| Column | Type | Meaning |
|---|---|---|
| `entry` | text | Word or phrase, lowercase, space-separated for phrases |
| `score` | decimal | Sentiment score −5.00 … +5.00. This is the value to compute with |
| `score_rounded` | integer | `score` rounded to the nearest integer, for reading and InSet compatibility |
| `source` | text | Origin of the score (see §4) |
| `type` | text | `unigram`, `bigram`, or `trigram` |
| `n_docs` | integer | Number of rated reviews containing the entry (corpus-scored entries only) |
| `score_inset` | decimal | Original InSet score where the entry is in InSet — kept so that every change can be traced |
| `score_domain` | decimal | Score induced from the corpus where the entry passed the frequency threshold |
| `standard_form` | text | For non-standard forms: the standard word whose score is borrowed |

---

## 3. Score scale

The scale follows InSet: strongly negative −5 to strongly positive +5, with 0
neutral. Original InSet scores are integers; corpus-induced scores are decimals
with two places. Both are provided: `score` for computation (rounding first
would equate −1.43 with −0.51, and the error accumulates over long reviews),
`score_rounded` for human reading.

Words listed in both InSet's positive and negative files (1,084 words) were
merged by mean weighting, (w⁺ + w⁻)/2. InSet does not prescribe a merge rule;
the mean is a choice made in this study, following our earlier work, which found
mean weighting preferable to summing (Chahyadi et al., 2025). The performance
gain of InSet-Eco does
not depend on that choice: tested under four rules (mean, sum, dominant
polarity, doubly-listed entries dropped), the gain was 29–30% under all four.
Because of mean merging, some scores inherited from InSet are multiples of 0.5.

---

## 4. Sources of entries and how they were built

InSet-Eco = L_InSet ∪ L_OOV ∪ L_domain, combined by the rules below. The column
`source` records the path of each entry:

| `source` | Entries | Meaning |
|---|---:|---|
| `InSet` | 7,905 | Original InSet score, unchanged |
| `InSet>domain` | 326 | In InSet and in the corpus, but corpus evidence below threshold (n_docs < 30): InSet score kept |
| `domain>InSet` | 128 | In InSet and in the corpus with sufficient evidence (n_docs ≥ 30): corpus score replaces InSet |
| `InSet/neutral` | 46 | InSet function word neutralized to 0 (§4.4) |
| `domain` | 2,926 | New entry from the corpus, absent from InSet |
| `L_OOV/annotator` | 1,091 | Non-standard form from manual annotation, borrowing the standard form's score |
| `L_OOV/elongation` | 230 | Letter elongation (*bagussss*), normalized automatically by a safe pattern |
| `L_OOV/non-standard` | 65 | Non-standard form listed in KBBI (*mantab* → *mantap*), filtered by manual audit |
| **Total** | **12,717** | |

### 4.1 L_InSet — the base lexicon

InSet (Koto & Rahmaningtyas, 2017) was downloaded from its official repository
as a positive file (3,609 entries) and a negative file (6,609 entries). The
entries include phrases, so these are not word counts. The processing chain
(`bangun_kamus.py`):

| Step | Positive | Negative | Total |
|---|---:|---:|---:|
| Entries in the source files | 3,609 | 6,609 | 10,218 |
| − duplicates within the same file after normalization* | 7 | 11 | 18 |
| − multi-word entries (L_InSet holds unigrams only) | 228 | 483 | 711 |
| = unigrams | 3,374 | 6,115 | 9,489 |
| − words present in both files, merged by the mean | | | 1,084 |
| **= unique L_InSet unigrams** | | | **8,405** |

\* Normalization: lowercasing; removal of syllable-division dots and
parenthetical notes. Where a word appears twice in the same file, the entry
with the larger |weight| is kept. The 711 multi-word entries comprise 654
distinct phrases; InSet's own phrases are not included in InSet-Eco.

### 4.2 L_domain — domain vocabulary from star-rated reviews

Corpus: 6,124 Indonesian-language Google Maps reviews of 44 coastal, mangrove,
and fishing-harbor sites in seven regencies and cities of the Riau Islands,
collected in August 2026. Star ratings serve as weak supervision: 1–2 stars →
y = −1, 3 stars → 0, 4–5 stars → +1. Star ratings measure overall
satisfaction, not the polarity of every sentence, so these are weak labels
rather than error-free sentiment annotations.

For every unigram, bigram, and trigram g occurring in at least f_min reviews
(5 for unigrams and bigrams, 4 for trigrams):

    score(g) = (Σ y_i + κ·ȳ) / (n_docs(g) + κ) − ȳ,     κ = 10

ȳ is the mean y over the corpus (+0.82; 87% of reviews are positive).
Subtracting ȳ centers the scores so that a word neutral with respect to the
corpus scores 0; the smoothing constant κ pulls low-frequency entries toward 0.
Scores are then normalized asymmetrically to −5 … +5, using the 95th percentile
on the positive side and the 5th percentile on the negative side as anchors,
so that a single outlying word does not set the scale.

Filters: unigrams that are stop words or proper names are removed, and n-grams
are removed if they contain a proper name or consist entirely of stop words.
Proper names are recognized from two sources: capitalization in mid-sentence in
at least 60% of occurrences (and no KBBI lemma), and the gazetteer
`gazetteer_proper_names.txt`, which lists regency, city, and country names and
the data-collection sites. The gazetteer is needed because names such as
*bintan* and *lagoi* are more often written in lowercase; without it, 30 place
names obtained scores up to +5.0 — leakage of site ratings, not sentiment. A
unigram immediately preceded by a negation word is not counted for itself; the
occurrence is recorded on the negation bigram (*tidak terawat*, "not
maintained"), where it belongs.

### 4.3 Conflict rule

For the 454 entries present in both InSet and L_domain, the corpus score is used
when the entry is supported by **at least 30 reviews**; otherwise the InSet
score is kept. The principle is that a general lexicon acts as a prior that
corpus evidence replaces only when strong enough (Lu et al., 2011; Kreutz &
Daelemans, 2018); the frequency threshold follows Islam, Dong & de Melo (2020).
The threshold was tested at 0/10/15/20/30/50/∞, and classification performance
is insensitive to the choice once the lexicon is re-induced per fold. The
decomposition (§8) shows that new entries, phrases, and revised InSet scores
each individually recover about 60% of the gain when added alone and substitute
for one another; non-standard forms add coverage but not accuracy.

Before the conflict rule was applied, 75 unigrams whose score actually belonged
to a phrase they almost always occur in (*salah* "wrong" scored +5.0 because 88%
of its occurrences are *salah satu* "one of") were reviewed manually: 19 were
dropped, 10 moved to their bigram, 9 kept, and 37 set to follow InSet.

### 4.4 Function-word neutralization

46 InSet entries that are function words — the Sastrawi stop-word list, with
negation words (*tidak, bukan, kurang, tanpa, jangan, belum*) excepted — were
neutralized to 0. Function words carry no evaluative content; content-based
lexicons such as SO-CAL (Taboada et al., 2011) include only content words. A
score such as *yang* ("which/that") −5.0 reflects contextual association in
InSet's construction data (microblogs), not the polarity of the word; in this
corpus the word occurs 1,456 times and alone contributes −7,280 to the sum of
document scores. The original score remains in the column `score_inset`.
Phrases containing function words (*tidak ada*, *pantai yang indah*) are
untouched — they are separate entries. The step is retained on conceptual
grounds; its effect on macro-F1 is not distinguishable from zero under the
corrected test.

### 4.5 L_OOV — non-standard forms

Of 3,851 word types in the corpus outside both InSet and KBBI, 1,519 were judged
misspellings or non-standard forms by the annotators and given their standard
form (two automatic layers add 295 further entries). Each non-standard form
enters the lexicon with the score of its standard form, so *bgt* receives the
score of *banget* ("very"). 96 of them are labeled local Riau Malay terms
(*takde* → *tidak ada* "there is none", *saye* → *saya* "I", *nengok* →
*melihat* "to see").

The annotation was carried out by three annotators — two students and one
lecturer — on equal, non-overlapping ranges under the same written guideline;
every type was then reviewed by the principal investigator. The result is
adjudicated annotation rather than independently replicated annotation, so no
agreement coefficient is available.

---

## 5. Composition

| Type | Entries | | Rounded score | Entries |
|---|---:|---|---:|---:|
| unigram | 10,548 | | −5 | 1,177 |
| bigram | 1,592 | | −4 | 1,708 |
| trigram | 577 | | −3 | 2,165 |
| | | | −2 | 737 |
| **Polarity** | | | −1 | 833 |
| positive (score > 0) | 5,082 | | 0 | 1,366 |
| negative (score < 0) | 7,263 | | +1 | 712 |
| neutral (score = 0) | 372 | | +2 | 752 |
| | | | +3 | 1,874 |
| | | | +4 | 966 |
| | | | +5 | 427 |

The skew toward negative entries (7,263 vs. 5,082) is inherited from InSet, 66%
of whose entries are negative. Users should not interpret the sign of a summed
document score in absolute terms (see §7 and §9).

---

## 6. Example entries

| entry | score | source | note |
|---|---:|---|---|
| bersih ("clean") | +4.55 | domain | Absent from InSet; the key word of beach reviews, 774 reviews |
| berkunjung ("to visit") | +5.00 | domain>InSet | InSet −4.0; clearly positive in travel reviews |
| mahal ("expensive") | −5.00 | domain>InSet | InSet +1.0 |
| sampah ("litter") | −5.00 | domain>InSet | InSet −4.0 |
| yang ("which/that") | 0.00 | InSet/neutral | InSet −5.0 |
| tidak ada ("there is none") | −5.00 | domain | bigram; arises from correcting *takde* |
| pasir putih ("white sand") | +5.00 | domain | bigram; the unigram *putih* follows InSet |
| banyak sampah berserakan ("a lot of scattered litter") | −5.00 | domain | trigram |
| bagus banget ("very good") | +5.00 | domain | bigram |
| bgt | +1.44 | L_OOV/annotator | borrows *banget* |
| mantab | +5.00 | L_OOV/non-standard | borrows *mantap* |
| takde | −5.00 | L_OOV/annotator | local term, borrows *tidak ada* |
| ac ("air conditioner") | −5.00 | domain | complaints about lodging air conditioning |

---

## 7. How to use

**Matching.** Because the lexicon contains phrases, use greedy longest-first
matching: at each token position, try the trigram, then the bigram, then the
unigram; tokens already consumed by a phrase are not scored again. This keeps
*salah satu* (+5.0) from being overridden by *salah* (InSet −4.0), and *tidak
ada* (−5.0) from being overridden by *ada*.

**Document score.** Sum the scores of all matched entries: S(d) = Σ score(g).
For classification features, add the counts of matched positive, negative, and
neutral entries.

**Minimal preprocessing.** Lowercase, remove punctuation and URLs, split on
whitespace. Spelling correction is optional — common non-standard forms are
already in the lexicon — but if `oov_corrections_validated.tsv` is used, apply it
before matching.

**Python example:**

```python
import pandas as pd

lex = pd.read_csv("lexicon/inset_eco_full.tsv", sep="\t", keep_default_na=False)
score = dict(zip(lex.entry, lex.score.astype(float)))
N = 3  # longest phrase length

def document_score(text):
    tok = text.lower().split()
    total, i = 0.0, 0
    while i < len(tok):
        for n in range(min(N, len(tok) - i), 0, -1):
            g = " ".join(tok[i:i + n])
            if g in score:
                total += score[g]
                i += n
                break
        else:
            i += 1
    return total

document_score("pantainya bersih tapi tiket masuk mahal bgt")
```

`keep_default_na=False` is required: the lexicon contains the word *nan* (a
literary variant of *yang*), which pandas would otherwise read as a missing
value.

**InSet compatibility.** Code that reads InSet's `positive.tsv` and
`negative.tsv` can read `inset_eco_compact.tsv` (columns `word`, `weight`) with
one change: there is no need to merge two files, since every word appears once
with a single sign.

---

## 8. Evaluation

Evaluated on 6,124 star-rated Google Maps reviews (positive 86.9%, neutral
8.3%, negative 4.9%), three-class classification with a linear SVM, stratified
five-fold cross-validation repeated five times (25 paired resampled
evaluations). For InSet-Eco, the domain vocabulary was re-induced from the
training fold alone in every evaluation, so the lexicon never saw test labels.
The full-corpus file `inset_eco_full.tsv` was therefore not used unchanged on
any test fold.

Paired comparisons use the corrected resampled t-test of Nadeau & Bengio
(2003), because the 25 evaluations share documents and are not independent
observations. The first row is the primary comparison; the others are
secondary.

| Features | InSet, original text | InSet-Eco, corrected text | Δ | corrected p | 95% CI |
|---|---:|---:|---:|---:|---|
| Lexicon only — macro-F1 | 0.338 | **0.436** | +0.098 (+29%) | < 0.0001 | [+0.076, +0.121] |
| Lexicon only — negative-class F1 | 0.085 | **0.320** | +0.235 | < 0.0001 | [+0.177, +0.293] |
| TF-IDF + lexicon — macro-F1 | 0.476 | 0.478 | +0.003 | 0.77 | [−0.015, +0.020] |

Token coverage (the share of review tokens that receive a score) rises from
48.5% to 64.5%. Accuracy falls slightly because the lexicon stops favoring the
majority class — not a deterioration, since minority-class F1 rises sharply.
With the combined TF-IDF + lexicon model, the lexicon adds at most 0.020 (upper
confidence limit), because TF-IDF features already carry almost all of the
information: TF-IDF without any lexicon reaches macro-F1 0.473 on the same 25
evaluations. Full results are reported in the paper.

---

## 9. Limitations

- **Domain-specific.** Scores are computed from coastal tourism reviews in the
  Riau Islands. *Berkunjung* +5 and *mahal* −5 make sense here, not necessarily
  elsewhere, because word polarity changes across domains (Asghar et al., 2015).
- **InSet legacy.** 7,905 entries are carried over as they are, including the
  negative skew and scores that may not fit the domain but occur too rarely in
  the corpus to be corrected (n_docs < 30). The columns `score_inset` and
  `source` let users filter them.
- **Proper names.** Place names are filtered by capitalization detection and a
  gazetteer of 39 names (regencies and cities, countries, data-collection
  sites). Place names outside that list — for example sites mentioned by
  reviewers but not collected — may still pass if written in lowercase. Users
  can extend `gazetteer_proper_names.txt` before rebuilding.
- **Neutral class.** Like InSet, this lexicon barely helps recognize 3-star
  reviews (neutral-class F1 0.06). Neutral is not "no sentiment" but a mixture,
  and summing scores does not capture it.
- **Corpus size.** 110 thousand tokens is a small corpus for lexicon induction.
  Entries supported by few reviews (n_docs 5–14) are smoothed but remain more
  fragile; the column `n_docs` is provided so that users can set their own
  threshold.
- **Language.** Frequent foreign words in the reviews (*view, sunset, seafood*)
  are deliberately not scored.
- **Annotation.** The spelling-correction map is adjudicated rather than
  independently replicated annotation (§4.5).

---

## 10. Rebuilding

This repository contains the lexicon only. The lexicon was built with seven Python scripts (reference word
lists from KBBI and InSet; preprocessing and out-of-vocabulary detection; correction candidates; application of
annotator verdicts and automatic layers; domain vocabulary and scoring; collocation review; assembly). The
scripts need the raw corpus, which cannot be redistributed. They are available from the corresponding author
(ferdi.chahyadi@umrah.ac.id) for research use.

---

## 11. Citation

When using InSet-Eco, cite the paper and InSet as its base:

> F. Chahyadi, R.A.I. Pratiwi, D. Akbar, N. Hamid, A. Baihaqi, "InSet-Eco: Adapting an Indonesian Sentiment
> Lexicon for Coastal Tourism Reviews in the Riau Islands," *BIO Web of Conferences* **262**, 03011 (2026).
> https://doi.org/10.1051/bioconf/202626203011

> F. Koto, G.Y. Rahmaningtyas, "InSet Lexicon: Evaluation of a Word List for Indonesian Sentiment Analysis in
> Microblogs," in *Proc. IALP* (IEEE, 2017), pp. 391–394.

### Method references

- M.Z. Asghar, A. Khan, S. Ahmad, I.A. Khan, F.M. Kundi, *PLoS ONE* **10**, e0140204 (2015).
- F. Chahyadi, A. Uperiati, R.A.I. Pratiwi, N. Hamid, *J. Tek. Inform. (JUTIF)* **6**, 5679 (2025).
- S.M. Islam, X. Dong, G. de Melo, in *Proc. COLING* (2020), pp. 6576–6587.
- T. Kreutz, W. Daelemans, in *Proc. COLING* (2018), pp. 1056–1064.
- Y. Lu, M. Castellanos, U. Dayal, C. Zhai, in *Proc. WWW* (2011), pp. 347–356.
- C. Nadeau, Y. Bengio, *Mach. Learn.* **52**, 239 (2003).
- M. Taboada, J. Brooke, M. Tofiloski, K. Voll, M. Stede, *Comput. Linguist.* **37**, 267 (2011).

---

## 12. Version history

| Version | Date | Changes |
|---|---|---|
| 1.0 | 20 Sep 2026 | First release: 12,717 entries; full OOV annotation of 3,851 types; conflict rule n_docs ≥ 30; 75 collocations reviewed; 46 function words neutralized; place names filtered with a gazetteer |
| 1.0 (doc.) | 29 Sep 2026 | Lexicon unchanged. Documentation corrected: new entries 2,926 (not 3,054); doubly-listed InSet entries 1,084 (not 1,142), mean merge stated as a study choice; source of the gain aligned with the decomposition; corrected test and confidence intervals; annotation protocol; attribution in place of the license placeholder |
| 1.0.0 | 11 Nov 2026 | First public release on GitHub. Lexicon unchanged; InSet-format `positive.tsv` and `negative.tsv` added; documentation linked to the published paper |
