# Trading
//+------------------------------------------------------------------+
//|                                       Heatmap_Gradient_Scale.mq5 |
//|                         Rodrigo Malacarne ■ DeltaTrader © 2014 ■ |
//|                                           www.deltatrader.com.br |
//|                                   Copyright 2014, Alain Verleyen |
//|                     https://login.mql5.com/en/users/angevoyageur |
//+------------------------------------------------------------------+
#property copyright "Copyright 2014, Rodrigo Malacarne / Alain Verleyen"
#property link      "https://login.mql5.com/en/users/angevoyageur"
#property version   "1.00"
#property strict    //--- For compatibility with MT4
#property indicator_chart_window
//---
#property indicator_plots   0

//--- ENUMS
enum ENUM_SCALE_TYPE       {DYNAMIC,FIXED};
enum ENUM_INDICATOR_TYPE   {SCALE,GRADIENT,HEATMAP};

//--- INPUTS
input string               indicatorSettings    =  "----- Indicator Settings";      // ■ Indicator Management
input ENUM_INDICATOR_TYPE  indicatorType        =  HEATMAP;                         // Indicator Type

//--- GLOBALS
string         marketWatchSymbolsList[];
double         percentChange[];
color          colorArray[];
//---
int            symbolsTotal=0;
//---
MqlRates       DailyBar[];
//+------------------------------------------------------------------+
//| Indicator initialization function                                |
//+------------------------------------------------------------------+
int OnInit()
  {
//---
   if(SymbolsTotal(true)<5)
     {
      Alert("The minimum number of symbols must be 5 (five).");
      return(INIT_PARAMETERS_INCORRECT);
     }
//---
   EventSetTimer(1);
//--- 
   ArraySetAsSeries(DailyBar,true);
//---
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Indicator deinitialization function                              |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
//--- Delete only what is drawn by your code
   deleteScale(symbolsTotal);
   ChartRedraw();
  }
//+------------------------------------------------------------------+
//| Indicator iteration function                                     |
//+------------------------------------------------------------------+
int OnCalculate(const int rates_total,
                const int prev_calculated,
                const datetime &time[],
                const double &open[],
                const double &high[],
                const double &low[],
                const double &close[],
                const long &tickVolume[],
                const long &volume[],
                const int &spread[])
  {

//---
   return(rates_total);
  }
//+------------------------------------------------------------------+
//| Timer function                                                   |
//+------------------------------------------------------------------+
void OnTimer()
  {
   int currentSymbolsTotal=SymbolsTotal(true);

//--- If we add or remove a symbol to the market watch
   if(symbolsTotal!=currentSymbolsTotal)
     {
      //--- resize arrays 
      ArrayResize(marketWatchSymbolsList,currentSymbolsTotal);
      ArrayResize(percentChange,currentSymbolsTotal);
      ArrayResize(colorArray,currentSymbolsTotal);
      //--- update arrays of symbol's name
      for(int i=0;i<currentSymbolsTotal;i++) marketWatchSymbolsList[i]=SymbolName(i,true);
      //--- remove panel in excess
      deleteScale(symbolsTotal,currentSymbolsTotal);
      //---
      symbolsTotal=currentSymbolsTotal;
     }

//--- call right drawing function
   switch(indicatorType)
     {
      case SCALE     : Scale(); break;
      case GRADIENT  : Gradient(); break;
      case HEATMAP   : Heatmap(); break;
     }
   ChartRedraw();
  }
//+------------------------------------------------------------------+
//| SCALE                                                            |
//+------------------------------------------------------------------+
void Scale()
  {
//--- Variables for the color gradient
   int length=SymbolsTotal(true); // Number os symbols on market watch

//--- Set values through for-loop
   for(int i=0;i<length;i++)
     {
      //---
      if(CopyRates(marketWatchSymbolsList[i],PERIOD_D1,0,2,DailyBar)!=2)
        {
         printf("Not all data available yet (%s) !",marketWatchSymbolsList[i]);
         return;
        }

      //--- Color 1 parameters
      int r_1 = 0;
      int g_1 = 64;
      int b_1 = 255;

      //--- Color 2 parameters
      int r_2 = 255;
      int g_2 = 255;
      int b_2 = 255;

      //--- Interpolation functions
      int r_value = r_1-i*((r_1-r_2)/(length-1));
      int g_value = g_1-i*((g_1-g_2)/(length-1));
      int b_value = b_1-i*((b_1-b_2)/(length-1));

      //---
      string rgbColor=IntegerToString(r_value)+","+IntegerToString(g_value)+","+IntegerToString(b_value);

      //---
      colorArray[i]=StringToColor(rgbColor);
      marketWatchSymbolsList[i]=SymbolName(i,true);
      percentChange[i]=((DailyBar[0].close/DailyBar[1].close)-1)*100;

      //--- Texts and boxes
      SetPanel("Panel "+IntegerToString(i),0,10,32+i*32,50,30,StringToColor(rgbColor),clrWhite,1);
      SetText("Text "+IntegerToString(i),marketWatchSymbolsList[i],13,33+i*32,clrBlack,8);
      //---
      if(percentChange[i]>=0)
        {
         SetText("Variation "+IntegerToString(i),"+"+DoubleToString(percentChange[i],2)+"%",17,46+i*32,clrBlack,8);
        }
      else
        {
         SetText("Variation "+IntegerToString(i),DoubleToString(percentChange[i],2)+"%",17,46+i*32,clrBlack,8);
        }
     }

  }
//+------------------------------------------------------------------+
//| GRADIENT                                                         |
//+------------------------------------------------------------------+
void Gradient()
  {
   int arrayMax, arrayMin;
   double scaleEdge, scaleMax, scaleMin;

//--- Color 1 parameters
   int r_1 = 255;
   int g_1 = 0;
   int b_1 = 0;
   
//--- Color 2 parameters
   int r_2 = 0;
   int g_2 = 255;
   int b_2 = 0;

//--- Sets values for variables
   for(int i=0;i<symbolsTotal;i++)
     {
      //--- Calculates the daily percent change
      marketWatchSymbolsList[i]=SymbolName(i,true);
      
      if(CopyRates(marketWatchSymbolsList[i],PERIOD_D1,0,2,DailyBar)!=2)
        {
         printf("Not all data available yet (%s) !",marketWatchSymbolsList[i]);
         return;
        }

      //--- Calculates daily percent change
      percentChange[i]=((DailyBar[0].close/DailyBar[1].close)-1)*100;

      //--- Interpolation functions
      int r_value = r_1-i*((r_1-r_2)/(symbolsTotal-1));
      int g_value = g_1-i*((g_1-g_2)/(symbolsTotal-1));
      int b_value = b_1-i*((b_1-b_2)/(symbolsTotal-1));
            
      //--- Sets colors to colorArray[]
      string rgbColor = IntegerToString(r_value)+","+IntegerToString(g_value)+","+IntegerToString(b_value);
      colorArray[i] = StringToColor(rgbColor);
     }

//--- Define gradient colors
   for(int i=0;i<symbolsTotal;i++)
     {
      //---
      arrayMax=ArrayMaximum(percentChange);
      arrayMin=ArrayMinimum(percentChange);
      
      //---
      if(MathAbs(percentChange[arrayMax])>MathAbs(percentChange[arrayMin]))
        {
         scaleEdge=percentChange[arrayMax];
        }
      else
        {
         scaleEdge=percentChange[arrayMin];
        }
      //---
      scaleMax = scaleEdge;
      scaleMin = -scaleEdge;

      //--- Local variable gradientColor
      int gradientColor;
      gradientColor = (int)MathFloor(((scaleMax-percentChange[i])/(scaleMax-scaleMin))*(symbolsTotal-1));

      //--- Texts and boxes
      SetPanel("Panel "+IntegerToString(i), 0, 10, 32+i*32, 50, 30, colorArray[gradientColor], clrWhite, 1);
      SetText("Text "+IntegerToString(i),marketWatchSymbolsList[i],13,33+i*32,clrBlack,8);
      if(percentChange[i]>=0)
        {
         SetText("Variation "+IntegerToString(i),"+"+DoubleToString(percentChange[i],2)+"%",17,46+i*32,clrBlack,8);
        }
      else
        {
         SetText("Variation "+IntegerToString(i),DoubleToString(percentChange[i],2)+"%",17,46+i*32,clrBlack,8);
        }
     }
  }
//+------------------------------------------------------------------+
//| HEATMAP                                                          |
//+------------------------------------------------------------------+
void Heatmap()
  {
//--- locals
   double scaleMax,scaleMin,scaleEdge;
//---
   int    arrayMax,arrayMin;

//--- Color 1 parameters
   int r_1 = 0;
   int g_1 = 255;
   int b_1 = 0;

//--- Color 2 parameters
   int r_2 = 255;
   int g_2 = 255;
   int b_2 = 255;

//--- Color 3 parameters
   int r_3 = 255;
   int g_3 = 0;
   int b_3 = 0;

   int midValue=(symbolsTotal%2==0 ? symbolsTotal/2 :(symbolsTotal-1)/2)-1;
//--- Build an array of colors and calculate percentage price's change
   for(int i=0;i<symbolsTotal;i++)
     {
      //--- Local variables
      int r_value = r_2;
      int g_value = g_2;
      int b_value = b_2;

      //--- Calculates the percent change of each symbol
      if(CopyRates(marketWatchSymbolsList[i],PERIOD_D1,0,2,DailyBar)==2)
        {
         percentChange[i]=((DailyBar[0].close/DailyBar[1].close)-1)*100;

         if(i<=midValue) // Positive values
           {
            //--- Positive interpolation function
            r_value = r_1-i*(r_1-r_2)/midValue;
            g_value = g_1-i*(g_1-g_2)/midValue;
            b_value = b_1-i*(b_1-b_2)/midValue;
           }
         else
           {
            //--- Negative interpolation function
            r_value = r_2-(i-midValue-1)*(r_2-r_3)/midValue;
            g_value = g_2-(i-midValue-1)*(g_2-g_3)/midValue;
            b_value = b_2-(i-midValue-1)*(b_2-b_3)/midValue;
           }
        }
      //--- Sets all possible colors to the array colorArray[]
      string rgbColor=IntegerToString(r_value)+","+IntegerToString(g_value)+","+IntegerToString(b_value);
      colorArray[i]=StringToColor(rgbColor);
     }

//--- Determine maximum/minimum and the scale value
   arrayMax=ArrayMaximum(percentChange); arrayMin=ArrayMinimum(percentChange);
   if(arrayMax==-1 || arrayMin==-1) return;

//--- Determine the scale
   scaleEdge=MathMax(MathAbs(percentChange[arrayMax]),MathAbs(percentChange[arrayMin]));
//---
   scaleMax=scaleEdge; scaleMin=-scaleEdge;
   
   midValue=(symbolsTotal%2==0 ? symbolsTotal/2 :(symbolsTotal-1)/2);
//--- Sets colors to the heatmap
   for(int i=0;i<symbolsTotal;i++)
     {
      //--- Local variable
      int heatmapColor=0;

      //--- Sets the color position (gradientColor) inside the colorArray
      if(percentChange[i]>0)
        {
         //--- color index between 0 and (symbolsTotal/2)-1
         heatmapColor=(int)MathFloor((1-percentChange[i]/scaleMax)*midValue);
        }
      else if(percentChange[i]<0)
        {
         //--- color index between symbolsTotal/2 and symbolsTotal-1
         heatmapColor=(int)MathCeil((percentChange[i]/scaleMin)*midValue)+midValue-1;
        }
      else
        {
         heatmapColor=midValue; // Mid position
        }

      if(heatmapColor<0) Print("Array out of range, heatmapcolor=",heatmapColor);
      if(heatmapColor>=ArraySize(colorArray)) Print("Array out of range, heatmapcolor=",heatmapColor," colorArray size=",ArraySize(colorArray));
      //--- Texts and boxes
      SetPanel("Panel "+IntegerToString(i),0,10,32+i*32,50,30,colorArray[heatmapColor],clrWhite,1);
      SetText("Text "+IntegerToString(i),marketWatchSymbolsList[i],13,33+i*32,clrBlack,8);
      if(percentChange[i]>=0)
        {
         SetText("Variation "+IntegerToString(i),"+"+DoubleToString(percentChange[i],2)+"%",17,46+i*32,clrBlack,8);
        }
      else
        {
         SetText("Variation "+IntegerToString(i),DoubleToString(percentChange[i],2)+"%",17,46+i*32,clrBlack,8);
        }
     }
//---
  }
//+------------------------------------------------------------------+
//| Remove unneeded objects from main chart                          |
//+------------------------------------------------------------------+
void deleteScale(int from,int to=1)
  {
//---   
   from--;to--;
   for(int i=from;i>=to;i--)
     {
      ObjectDelete(0,"Panel "+IntegerToString(i));
      ObjectDelete(0,"Text "+IntegerToString(i));
      ObjectDelete(0,"Variation "+IntegerToString(i));
     }
  }
//+------------------------------------------------------------------+
//| Draw data about a symbol in a panel                              |
//+------------------------------------------------------------------+
void SetText(string name,string text,int x,int y,color colour,int fontsize=12)
  {
   if(ObjectCreate(0,name,OBJ_LABEL,0,0,0))
     {
      ObjectSetInteger(0,name,OBJPROP_XDISTANCE,x);
      ObjectSetInteger(0,name,OBJPROP_YDISTANCE,y);
      ObjectSetInteger(0,name,OBJPROP_COLOR,colour);
      ObjectSetInteger(0,name,OBJPROP_FONTSIZE,fontsize);
      ObjectSetInteger(0,name,OBJPROP_CORNER,CORNER_LEFT_UPPER);
     }
   ObjectSetString(0,name,OBJPROP_TEXT,text);
  }
//+------------------------------------------------------------------+
//| Draw a panel with given color for a symbol                       |
//+------------------------------------------------------------------+
void SetPanel(string name,int sub_window,int x,int y,int width,int height,color bg_color,color border_clr,int border_width)
  {
   if(ObjectCreate(0,name,OBJ_RECTANGLE_LABEL,sub_window,0,0))
     {
      ObjectSetInteger(0,name,OBJPROP_XDISTANCE,x);
      ObjectSetInteger(0,name,OBJPROP_YDISTANCE,y);
      ObjectSetInteger(0,name,OBJPROP_XSIZE,width);
      ObjectSetInteger(0,name,OBJPROP_YSIZE,height);
      ObjectSetInteger(0,name,OBJPROP_COLOR,border_clr);
      ObjectSetInteger(0,name,OBJPROP_BORDER_TYPE,BORDER_FLAT);
      ObjectSetInteger(0,name,OBJPROP_WIDTH,border_width);
      ObjectSetInteger(0,name,OBJPROP_CORNER,CORNER_LEFT_UPPER);
      ObjectSetInteger(0,name,OBJPROP_STYLE,STYLE_SOLID);
      ObjectSetInteger(0,name,OBJPROP_BACK,false);
      ObjectSetInteger(0,name,OBJPROP_SELECTABLE,0);
      ObjectSetInteger(0,name,OBJPROP_SELECTED,0);
      ObjectSetInteger(0,name,OBJPROP_HIDDEN,true);
      ObjectSetInteger(0,name,OBJPROP_ZORDER,0);
     }
   ObjectSetInteger(0,name,OBJPROP_BGCOLOR,bg_color);
  }
//+------------------------------------------------------------------+
