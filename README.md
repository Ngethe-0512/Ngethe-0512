//+------------------------------------------------------------------+
//| Smart Money Concept Forex Robot                                  |
//| Author: Your Name                                                |
//| Description: Uses SMC, support & resistance, price action, and   |
//| candle range theory for trading.                                 |
//+------------------------------------------------------------------+
#property strict

// Input Parameters
input double LotSize = 0.1;          // Default lot size
input int StopLoss = 30;             // Stop Loss in pips
input int TakeProfit = 50;           // Take Profit in pips
input double RiskRewardRatio = 1.5; // Risk-to-Reward Ratio
input int LookbackPeriod = 50;       // Candles to analyze for levels
input int MinCandleRange = 10;       // Minimum candle range (in points)

// Variables
double SupportLevel, ResistanceLevel;

//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit() {
   Print("Smart Money Concept Forex Robot initialized.");
   return(INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Main trading function                                            |
//+------------------------------------------------------------------+
void OnTick() {
   double Bid = SymbolInfoDouble(_Symbol, SYMBOL_BID);
   double Ask = SymbolInfoDouble(_Symbol, SYMBOL_ASK);
   
   // Update support and resistance levels
   SupportLevel = FindSupportLevel();
   ResistanceLevel = FindResistanceLevel();
   
   // Check for trade opportunities
   if (Bid < SupportLevel && IsBullishEngulfing()) {
      OpenTrade(ORDER_TYPE_BUY, Bid, SupportLevel, ResistanceLevel);
   }
   if (Bid > ResistanceLevel && IsBearishEngulfing()) {
      OpenTrade(ORDER_TYPE_SELL, Bid, ResistanceLevel, SupportLevel);
   }
}

//+------------------------------------------------------------------+
//| Find Support Level                                               |
//+------------------------------------------------------------------+
double FindSupportLevel() {
   double minPrice = High[0];
   for (int i = 1; i <= LookbackPeriod; i++) {
      if (Low[i] < minPrice) minPrice = Low[i];
   }
   return minPrice;
}

//+------------------------------------------------------------------+
//| Find Resistance Level                                            |
//+------------------------------------------------------------------+
double FindResistanceLevel() {
   double maxPrice = Low[0];
   for (int i = 1; i <= LookbackPeriod; i++) {
      if (High[i] > maxPrice) maxPrice = High[i];
   }
   return maxPrice;
}

//+------------------------------------------------------------------+
//| Check for Bullish Engulfing Pattern                              |
//+------------------------------------------------------------------+
bool IsBullishEngulfing() {
   if (Close[1] < Open[1] && Close[0] > Open[0] && Close[0] > High[1]) {
      return true;
   }
   return false;
}

//+------------------------------------------------------------------+
//| Check for Bearish Engulfing Pattern                              |
//+------------------------------------------------------------------+
bool IsBearishEngulfing() {
   if (Close[1] > Open[1] && Close[0] < Open[0] && Close[0] < Low[1]) {
      return true;
   }
   return false;
}

//+------------------------------------------------------------------+
//| Open Trade Function                                              |
//+------------------------------------------------------------------+
void OpenTrade(int type, double price, double sl, double tp) {
   double lotSize = LotSize;
   double stopLossPrice = (type == ORDER_TYPE_BUY) ? price - (StopLoss * Point) : price + (StopLoss * Point);
   double takeProfitPrice = (type == ORDER_TYPE_BUY) ? price + (TakeProfit * Point) : price - (TakeProfit * Point);
   
   int ticket = OrderSend(_Symbol, type, lotSize, price, 3, stopLossPrice, takeProfitPrice, "SMC Robot", 0, 0, clrGreen);
   if (ticket < 0) {
      Print("Error opening trade: ", GetLastError());
   }
}

//+------------------------------------------------------------------+

