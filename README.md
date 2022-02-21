//@version=4
//© Densz
strategy("3EMA with TP & SL (ATR)", overlay=true, initial_capital=10000.00, default_qty_type=strategy.percent_of_equity, default_qty_value=10, commission_type=strategy.commission.percent, commission_value=0.01)

// INPUTS
startTime           =       input(title="Start Time", type = input.time, defval = timestamp("01 Jan 2017 00:00 +0000"))
endTime             =       input(title="End Time", type = input.time, defval = timestamp("01 Jan 2022 00:00 +0000"))

slowEMALength       =       input(title="Slow EMA Length", type = input.integer, defval = 55)
middleEMALength     =       input(title="Middle EMA Length", type = input.integer, defval = 21)
fastEMALength       =       input(title="Fast EMA Length", type = input.integer, defval = 9)

trendMALength       =       input(title="Trend indicator MA Length", type = input.integer, defval = 200)

atrLength           =       input(title="ATR Length", type = input.integer, defval = 14)
tpATRMult           =       input(title="Take profit ATR multiplier", type = input.integer, defval = 3)
slATRMult           =       input(title="Stop loss ATR multiplier", type = input.integer, defval = 2)

rsiLength           =       input(title="RSI Length", type = input.integer, defval = 14)

// Indicators
slowEMA             =       ema(close, slowEMALength)
middEMA             =       ema(close, middleEMALength)
fastEMA             =       ema(close, fastEMALength)
atr                 =       atr(atrLength)

rsiValue            =       rsi(close, rsiLength)
isRsiOB             =       rsiValue >= 80
isRsiOS             =       rsiValue <= 20

sma200              =       sma(close, trendMALength)

inDateRange         =       (time >= startTime) and (time < endTime)

// Plotting
plot(slowEMA, title="Slow EMA", color=color.red, linewidth=2, transp=50)
plot(middEMA, title="Middle EMA", color=color.orange, linewidth=2, transp=50)
plot(fastEMA, title="Fast EMA", color=color.green, linewidth=2, transp=50)

plot(sma200, title="SMA Trend indicator", color=color.purple, linewidth=3, transp=10)
plotshape(isRsiOB, title="Overbought", location=location.abovebar, color=color.red, transp=0, style=shape.triangledown, text="OB")
plotshape(isRsiOS, title="Oversold", location=location.belowbar, color=color.green, transp=0, style=shape.triangledown, text="OS")

float takeprofit    =       na
float stoploss      =       na

var line tpline     =       na
var line slline     =       na

if strategy.position_size != 0
    takeprofit := takeprofit[1]
    stoploss := stoploss[1]
    line.set_x2(tpline, bar_index)
    line.set_x2(slline, bar_index)
    line.set_extend(tpline, extend.none)
    line.set_extend(slline, extend.none)
    
// STRATEGY
goLong  = crossover(middEMA, slowEMA) and inDateRange
closeLong = crossunder(fastEMA, middEMA) and inDateRange


if goLong
    takeprofit := close + atr * tpATRMult
    stoploss := close - atr * slATRMult
    tpline := line.new(bar_index, takeprofit, bar_index, takeprofit, color=color.green, width=2, extend=extend.right, style=line.style_dotted)
    slline := line.new(bar_index, stoploss, bar_index, stoploss, color=color.red, width=2, extend=extend.right, style=line.style_dotted)
    label.new(bar_index, takeprofit, "TP", style=label.style_labeldown)
    label.new(bar_index, stoploss, "SL", style=label.style_labelup)
    strategy.entry("Long", strategy.long, when = goLong)
    strategy.exit("TP/SL", "Long", stop=stoploss, limit=takeprofit)
if closeLong
    takeprofit := na
    stoploss := na
    strategy.close(id = "Long", when = closeLong)

if (not inDateRange)
    strategy.close_all()
