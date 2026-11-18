# Wick CE + Engineered Liquidity: TradingView strategy

`WickCE_Strategy.pine` is a Pine Script v6 strategy for the course's two entry routes.

## How to use
1. In TradingView, open **Pine Editor**, paste the file, then click **Add to chart**.
2. Use the chart as the **execution timeframe**, for example 5m or 15m. Set **HTF location timeframe** to the wick timeframe. The default is H4 (`240`).
3. Under **Entry route**, choose the route:
   - **Setup 1 - Direct wick CE**: places a limit order at the HTF wick CE. The stop goes beyond the wick extreme.
   - **Setup 2 - LTF rejection block**: waits for price to reach the HTF CE, then for a chart-timeframe rejection wick. The entry is that wick's CE or its beginning. The stop goes beyond the smaller wick's extreme.

## How the course maps to the logic
| Course rule | Implementation |
|---|---|
| Destination / draw on liquidity | Untaken prior-day low for shorts and prior-day high for longs. You can use a fixed R multiple instead. If the destination has already traded, the setup is skipped. |
| Daily context | A prior-high sweep with a close back inside means bearish. A prior-low sweep with a close back inside means bullish. A close beyond a prior extreme means continuation. |
| Meaningful wick | The HTF wick sweeps the high or low of the last N HTF candles and closes back inside. It must also meet a minimum size in points and a minimum % of the candle range. |
| Wick measurement | Beginning = body edge (max/min of open and close). Extreme = high or low. CE = their average. |
| Engineered liquidity | At least N chart-TF swing highs below CE (bearish) or swing lows above CE (bullish) must form after the wick and before price returns to CE. |
| 2-point proximity filter | If a pool forms 2 points or less from CE, the CE counts as mitigated and the setup is discarded. |
| Unfilled imbalance origin | If the wick candle becomes the first candle of a fair value gap, the setup is discarded. The check runs after 2 HTF candles. |
| Equal-swing exclusion | A wick whose extreme is within the tolerance of the prior swing extreme is discarded. |
| Stop / size | Stop = structure extreme ± buffer. Sizing is fixed contracts or fixed cash risk. The trade is passed if R < minimum, the stop is too wide, or size < 1. |
| Management | Optional break-even at X R. Optional structural trail: a swing becomes the new stop once price breaks the swing that follows it. |
| Discipline | Session window, max trades per day, and a stop after N losses per day. |

The strategy does not code the "competing nearer draw" check or any SMT confluence. Review those by eye.
Backtest results are illustrative and do not establish a win rate.

## Signal indicators (one per setup)
These are standalone indicators, not strategies. Use them for chart signals and alerts.
- `WickCE_Setup1_Indicator.pine`: Setup 1, a direct limit at the HTF wick CE.
- `WickCE_Setup2_Indicator.pine`: Setup 2, a chart-timeframe rejection block at the HTF CE.

**Early mode** (setting: "Early mode: show HTF wick CE while the HTF candle is forming", on by default) detects and draws the HTF wick CE live, as soon as the forming HTF candle has a qualifying wick. The CE is re-anchored if the HTF candle makes a new extreme and is confirmed when the HTF candle closes. Turn it off to use completed HTF candles only.

Each indicator shows:
- A **BUY / SELL LIMIT** label when the order is armed.
- A **BUY / SELL** signal when the entry fills.
- A green take-profit box and a red stop-loss box.
- A **TP / SL / BE** label with the R result when the trade closes.
- A stats table with wins, losses, win % and net R.

Alerts: in TradingView, choose *Create alert → condition: the indicator → "Any alert() function call"*. You can also pick one of the BUY / SELL / Take profit / Stop loss conditions.

### Trade management and more entries (indicators)
- **Exit plan**: *Partial at target + runner* (default) takes a partial at the draw, moves the stop to break-even and trails the rest. *Runner* has no fixed TP. *Fixed TP* exits everything at the draw.
- **HOLD / CAUTION / EXIT** status for the open trade (table row "Open trade", plus chart labels and alerts):
  - HOLD: structure intact.
  - CAUTION: a higher high (short) or lower low (long) formed against the trade.
  - EXIT: a candle closed beyond the last swing against the trade.
- Structural trailing stop. Optional automatic exit on a structure shift after +X R.
- More entries:
  - HTF 2 (H1) wicks alongside H4.
  - "Both if unclear" bias.
  - London + NY session.
  - 3 trades per day.
  - Re-entry after a loss while the wick is still valid.
  - Setup 1 can enter at the wick beginning.
