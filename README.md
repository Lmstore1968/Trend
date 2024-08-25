 using cAlgo.API;
using cAlgo.API.Indicators;
using cAlgo.API.Internals;

namespace cAlgo
{
    [Robot(TimeZone = TimeZones.UTC, AccessRights = AccessRights.None)]
    public class MyTradingBot : Robot
    {
        // Input parameters
        [Parameter("EMA 50 Period", DefaultValue = 50)]
        public int Ema50Period { get; set; }

   [Parameter("SMA 100 Period", DefaultValue = 100)]
        public int Sma100Period { get; set; }

   [Parameter("MACD Fast EMA", DefaultValue = 12)]
        public int MacdFastEma { get; set; }

  [Parameter("MACD Slow EMA", DefaultValue = 26)]
        public int MacdSlowEma { get; set; }

   [Parameter("MACD Signal SMA", DefaultValue = 9)]
        public int MacdSignalSma { get; set; }

   [Parameter("Momentum Period", DefaultValue = 14)]
        public int MomentumPeriod { get; set; }

   [Parameter("RSI Period", DefaultValue = 14)]
        public int RsiPeriod { get; set; }

  [Parameter("CCI Period", DefaultValue = 14)]
        public int CciPeriod { get; set; }

  [Parameter("Stochastic K Period", DefaultValue = 14)]
        public int StochasticKPeriod { get; set; }

  [Parameter("Stochastic D Period", DefaultValue = 3)]
        public int StochasticDPeriod { get; set; }

  [Parameter("ADX Period", DefaultValue = 14)]
        public int AdxPeriod { get; set; }

  [Parameter("Lot Size", DefaultValue = 0.1)]
        public double LotSize { get; set; }

   [Parameter("Symbol to Trade", DefaultValue = "EURUSD")]
        public string SymbolName { get; set; }

   // Indicators
        private ExponentialMovingAverage _ema50;
        private SimpleMovingAverage _sma100;
        private MacdHistogram _macd;
        private Momentum _momentum;
        private RelativeStrengthIndex _rsi;
        private CommodityChannelIndex _cci;
        private StochasticOscillator _stochastic;
        private AverageDirectionalMovementIndex _adx;
        private Symbol _symbol;

 protected override void OnStart()
        {
            // Initialize the symbol
            _symbol = Symbols.GetSymbol(SymbolName);
            
   // Initialize indicators
            _ema50 = Indicators.ExponentialMovingAverage(MarketSeries.Close, Ema50Period);
            _sma100 = Indicators.SimpleMovingAverage(MarketSeries.Close, Sma100Period);
            _macd = Indicators.MacdHistogram(MacdFastEma, MacdSlowEma, MacdSignalSma);
            _momentum = Indicators.Momentum(MarketSeries.Close, MomentumPeriod);
            _rsi = Indicators.RelativeStrengthIndex(MarketSeries.Close, RsiPeriod);
            _cci = Indicators.CommodityChannelIndex(MarketSeries.Close, CciPeriod);
            _stochastic = Indicators.StochasticOscillator(StochasticKPeriod, StochasticDPeriod, 3);
            _adx = Indicators.AverageDirectionalMovementIndex(AdxPeriod);
        }

  protected override void OnTick()
        {
            // Retrieve indicator values
            double ema50 = _ema50.Result.LastValue;
            double sma100 = _sma100.Result.LastValue;
            double macdMain = _macd.Macd.LastValue;
            double macdSignal = _macd.Signal.LastValue;
            double momentum = _momentum.Result.LastValue;
            double rsi = _rsi.Result.LastValue;
            double cci = _cci.Result.LastValue;
            double stochK = _stochastic.PercentK.LastValue;
            double stochD = _stochastic.PercentD.LastValue;
            double adx = _adx.Adx.LastValue;
            double plusDI = _adx.PlusDI.LastValue;
            double minusDI = _adx.MinusDI.LastValue;

   // Example trading logic for buying
            if (ema50 > sma100 && macdMain > macdSignal && momentum > 0 &&
                rsi < 30 && cci < -100 && stochK < 20 && adx > 25 && plusDI > minusDI)
            {
                // Place buy order
                if (Positions.Find("BuyOrder") == null)
                {
                    ExecuteMarketOrder(TradeType.Buy, _symbol.Name, LotSize, "BuyOrder");
                }
            }
            // Example trading logic for selling
            else if (ema50 < sma100 && macdMain < macdSignal && momentum < 0 &&
                     rsi > 70 && cci > 100 && stochK > 80 && adx > 25 && plusDI < minusDI)
            {
                // Place sell order
                if (Positions.Find("SellOrder") == null)
                {
                    ExecuteMarketOrder(TradeType.Sell, _symbol.Name, LotSize, "SellOrder");
                }
            }
        }
    }
}
