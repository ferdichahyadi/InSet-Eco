# InSet-Eco

**An Indonesian sentiment lexicon adapted from InSet for coastal tourism reviews in the Riau Islands.**

InSet-Eco extends [InSet](https://github.com/fajri91/InSet) (Koto & Rahmaningtyas, 2017) with vocabulary,
phrases, and non-standard spellings learned from online reviews of beaches, mangroves, and fishing areas in
the Riau Islands, Indonesia. It contains **12,717 entries** — single words and two- or three-word phrases —
each scored from **−5** (strongly negative) to **+5** (strongly positive).

It is described and evaluated in:

> F. Chahyadi, R.A.I. Pratiwi, D. Akbar, N. Hamid, A. Baihaqi, "InSet-Eco: Adapting an Indonesian Sentiment
> Lexicon for Coastal Tourism Reviews in the Riau Islands," *BIO Web of Conferences* **262**, 03011 (2026).
> https://doi.org/10.1051/bioconf/202626203011

*Ringkasan dalam bahasa Indonesia ada di [bagian akhir](#bahasa-indonesia).*

---

## What is different from InSet

| | Entries | What it does |
|---|---:|---|
| New domain entries | 2,926 | Words and phrases absent from InSet, scored from 6,124 star-rated Google Maps reviews — including 2,169 phrases such as *pasir putih* (+5.0), *banyak sampah* (−5.0), *tidak ada* (−5.0) |
| Revised InSet entries | 128 | InSet scores that contradict their use in this domain, rescored from corpus evidence — e.g. *berkunjung* "to visit" −4 → +5, *mahal* "expensive" +1 → −5 |
| Neutralized function words | 46 | Function words such as *yang* (−5) and *dari* (−3) set to 0 |
| Non-standard forms | 1,386 | Informal spellings, abbreviations, and letter elongations (*bgt*, *mantab*, *takde*, *enakkkkk*) that borrow the score of their standard form |
| Unchanged InSet entries | 8,231 | Kept with their InSet score |

In three-class, lexicon-only sentiment classification of coastal tourism reviews, InSet-Eco raises macro-F1
from 0.338 (InSet) to 0.436 and negative-class F1 from 0.085 to 0.320 (see the paper for the evaluation
design and its limits).

## Files

| File | Contents |
|---|---|
| `lexicon/inset_eco_full.tsv` | **Master file.** 12,717 entries with provenance columns: `entry`, `score` (decimal), `score_rounded`, `source`, `type`, `n_docs`, `score_inset`, `score_domain`, `standard_form` |
| `lexicon/positive.tsv`, `lexicon/negative.tsv` | **InSet format** (`word`, `weight`, integer): 4,731 positive and 6,620 negative entries. Drop-in replacement for InSet's two files |
| `lexicon/inset_eco_compact.tsv` | The same 11,351 non-zero entries in one file |
| `lexicon/oov_corrections_validated.tsv` | 1,519 non-standard → standard spelling pairs validated by annotators (usable on its own for text normalization) |
| `lexicon/oov_correction_map_all_layers.tsv` | All 1,956 corrections applied in the study (annotator, elongation, and listed non-standard layers) |
| `lexicon/gazetteer_proper_names.txt` | Place names kept out of the domain vocabulary |
| `lexicon/elongation_exceptions.txt` | Words exempt from automatic elongation reduction |
| `DOCUMENTATION.md` | Full documentation: construction, score scale, composition, limitations |

All files are UTF-8, tab-separated, with a header row. Zero-scored entries (neutralized words) appear only in
the master file.

## How to use

Because the lexicon contains phrases, match **longest first**: at each position try the trigram, then the
bigram, then the unigram, and do not score tokens already covered by a phrase. This keeps *tidak ada* (−5.0)
from being overridden by *ada*.

```python
import pandas as pd

lex = pd.read_csv("lexicon/inset_eco_full.tsv", sep="\t", keep_default_na=False)
score = dict(zip(lex.entry, lex.score.astype(float)))

def document_score(text, max_n=3):
    tokens = text.lower().split()
    total, i = 0.0, 0
    while i < len(tokens):
        for n in range(min(max_n, len(tokens) - i), 0, -1):
            gram = " ".join(tokens[i:i + n])
            if gram in score:
                total += score[gram]
                i += n
                break
        else:
            i += 1
    return total

print(document_score("pantainya bersih tapi tiket masuk mahal bgt"))
```

`keep_default_na=False` is required: the lexicon contains the word *nan*, which pandas would otherwise read
as a missing value. Lowercase the text and remove punctuation and URLs before matching. Spelling correction is
optional, since common non-standard forms are already entries.

## Scope and limitations

- **Domain-specific.** Scores were learned from coastal tourism reviews in the Riau Islands. *Berkunjung* +5
  and *mahal* −5 fit this domain, not necessarily others. Use InSet for general text.
- **Inherited entries.** 7,905 InSet entries are unchanged, including some that may not fit the domain but
  were too rare in the corpus to be revised. Filter with the `source` and `score_inset` columns if needed.
- **Neutral reviews.** Like InSet, the lexicon does little to recognize mixed or neutral (3-star) reviews.
- **Negation.** Negation is covered only where the negated phrase is itself an entry (54 such entries).
  Other negated forms, such as *ga mahal*, are scored word by word.
- **Small corpus.** Entries supported by few reviews are smoothed but remain fragile; `n_docs` lets you set
  your own threshold.

The raw reviews, social-media posts, and survey responses used to build the lexicon are not distributed.

## Citation

If you use InSet-Eco, please cite the paper above **and** InSet, on which it is built:

> F. Koto, G.Y. Rahmaningtyas, "InSet Lexicon: Evaluation of a Word List for Indonesian Sentiment Analysis in
> Microblogs," in *Proc. IALP* (IEEE, 2017), pp. 391–394.

GitHub's **Cite this repository** button (from `CITATION.cff`) gives both references in BibTeX and APA.

## License

InSet-Eco is released under the MIT License (`LICENSE`). It includes the entries of InSet, which is also
MIT-licensed (confirmed by its first author, Fajri Koto, on 29 September 2026); InSet's license text is in
`LICENSE-InSet.txt` and must accompany any copy.

## Acknowledgments

This work was funded by the Universitas Maritim Raja Ali Haji (UMRAH) Fundamental Research Grant 2026.

---

## Bahasa Indonesia

**InSet-Eco** adalah kamus sentimen bahasa Indonesia hasil adaptasi InSet untuk ulasan wisata pesisir di
Kepulauan Riau. Kamus ini berisi **12.717 entri** (kata tunggal dan frasa dua–tiga kata) dengan skor dari
−5 (sangat negatif) sampai +5 (sangat positif). Dibandingkan InSet, InSet-Eco menambahkan 2.926 entri domain
baru (termasuk 2.169 frasa), merevisi 128 skor InSet yang tidak sesuai domain, menetralkan 46 kata fungsi,
dan memuat 1.386 bentuk tidak baku (*bgt*, *mantab*, *takde*) yang meminjam skor bentuk bakunya.

- **Berkas utama:** `lexicon/inset_eco_full.tsv`. Untuk kode yang sudah memakai InSet, gunakan
  `lexicon/positive.tsv` dan `lexicon/negative.tsv` (format sama dengan InSet).
- **Cara pakai:** cocokkan frasa terpanjang lebih dulu (trigram → bigram → unigram), lalu jumlahkan skornya.
  Contoh kode Python ada di bagian *How to use*.
- **Batasan:** skor dipelajari dari ulasan wisata pesisir Kepulauan Riau; untuk teks umum gunakan InSet.
- **Rujukan:** sitasi artikel di atas (*BIO Web of Conferences* 262, 03011, 2026) **dan** InSet
  (Koto & Rahmaningtyas, 2017).
- **Lisensi:** MIT. Berkas `LICENSE-InSet.txt` wajib disertakan pada setiap salinan.
