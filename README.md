# SKORE: Skeleton-Key Oriented Routing for Entity resolution

Code and benchmark generator for the article
*"Disambiguating transliterated author names with a consonantal skeleton key
and calibrated three-way routing"* by Mohammad Saeid Safaei Sadegh and
Ahmad Sharif (Department of Computer Engineering, Qom Branch, Islamic Azad
University). The case study and the included language
profile are for names romanised from Persian.

Everything runs on an ordinary laptop. No GPU is needed. The extension needs internet access, a free OpenAlex API key and at
least one LLM API key (or a local Ollama model).

## Layout

| Path | What it is |
|---|---|
| `skore/psk.py` | Phonetic Skeleton Key (Persian profile): rules R1-R9, abjad blocking keys, PSK-soft score, PSK-LR features |
| `skore/strsim.py` | Levenshtein, Jaro-Winkler, Soundex, Monge-Elkan (no external dependency) |
| `bench/generator.py` | Controlled generator of romanised Persian name mentions |
| `bench/run_benchmark.py` | Controlled benchmark (all numbers in Section 5 of the paper) |
| `bench/make_figures.py` | Figures 1-7 |
| `exp2/` | Extension described in Section 6.3: OpenAlex + ORCID labels + KG features + LLM adjudication |
| `results/exp1.json` | Output of the benchmark as reported in the paper |

## Install

```bash
python -m pip install numpy scipy scikit-learn matplotlib
```

## Controlled benchmark (~3 minutes on a laptop)

```bash
python -m bench.run_benchmark --out results/exp1.json
python bench/make_figures.py results/exp1.json figures
```

Five seeds (11, 23, 37, 41, 59) are used. The output is deterministic.

## Extension on real data (Section 6.3; run these in order)

1. Get a free OpenAlex API key (required since February 2026): https://openalex.org/settings/api
2. Set keys as environment variables (never commit them):
   ```bash
   export OPENALEX_API_KEY=...        # required
   export OPENALEX_MAILTO=...         # optional contact e-mail
   export DEEPSEEK_API_KEY=...        # optional, one LLM is enough
   export OPENAI_API_KEY=...          # optional
   ```
   For a local model install Ollama and run `ollama pull qwen2.5:7b`.
3. Check `exp2/config.json` (sample size, years, model list, prices per 1M
   tokens - update the prices to the providers' current rates).
4. Run:
   ```bash
   rm -rf data                                   # removes any mock data
   python -m exp2.s1_collect                     # 1-3 hours, mostly ORCID API calls
   python -m exp2.s2_pairs
   python -m exp2.s3_features
   python -m exp2.s4_llm --model deepseek-chat --limit 50   # pilot: inspect data/llm_deepseek-chat.jsonl
   python -m exp2.s4_llm --model deepseek-chat              # full run (resumable)
   python -m exp2.s4_llm --model qwen2.5:7b                 # optional second model
   python -m exp2.s5_evaluate
   ```
5. Results are written to `results/exp2.json`, `results/exp2_tables.md`,
   `data/blocking.json` and `figures/fig8_*.png`, `figures/fig9_*.png`.

`python -m exp2.mock_data` builds a synthetic stand-in for Step 1 so the
pipeline can be tested offline. Numbers from mock data are not results.

## Ethics and data

Only public metadata (OpenAlex, public ORCID records) is used. ORCID iDs are
used solely as ground-truth labels and are never given to the matchers or the
LLM. Release only aggregate results and pair identifiers, not personal profiles.

## Citation

If you use this code, please cite the article (see `CITATION.cff`).

## Author

Mohammad Saeid Safaei Sadegh — [saeidsafaei.ir](https://saeidsafaei.ir) · s.safaei@iau.ac.ir

## License

MIT (see `LICENSE`).
