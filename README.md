# Fibo_Bar
The Fibonacci retracement indicator is a technical analysis tool used to identify potential support and resistance levels based on Fibonacci ratios. 
This script calculates and displays these levels on price charts, making it easier for traders to identify potential entry and exit points.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Inputs](#inputs)
- [Customization](#customization)
- [Script Code](#script-code)
- [Contributing](#contributing)
- [Acknowledgments](#acknowledgments)
- [License](#license)
- [Credits](#credits)

## Features

- Automatic calculation of Fibonacci retracement levels.
- Customizable input parameters such as period type, timeframe, colors, and line styles.
- Visual representation of Fibonacci levels on price charts.
- Text labels for highest and lowest Fibonacci levels.

## Installation

1. Download the Script: Download the Fibo_Bar.mq5 file from this repository.
2. Open MetaTrader 5
   - Launch MetaTrader 5.
   - Go to `File` -> `Open Data Folder`.
3. Place the Script
   - Navigate to `MQL5` -> `Indicators`.
   - Copy the `Fibo_Bar.mq5` file into the Experts folder.
4. Refresh MetaTrader 5
   - Restart MetaTrader 5 or right-click in the Navigator window and select Refresh.

## Usage

1. Install the indicator script in your trading platform.
2. Configure the input parameters according to your preferences.
3. Apply the indicator to your price charts.
4. Interpret the Fibonacci retracement levels to inform your trading decisions.

## Inputs

- **Period Type:** Represents the type of period.
- **Timeframe:** Range period for calculating Fibonacci levels.
- **Line Position:** Position of the Fibonacci lines on the chart.
- **Colors:** Colors for different Fibonacci levels.
- **Line Styles:** Styles for the Fibonacci lines.
- **Line Widths:** Widths for the Fibonacci lines.

## Customization

You can customize the Fibonacci levels by modifying the following arrays in the script

```mql5
double fibo_main[]  = {0,23.6,38.2,50,55.9,61.8,66.7,78.6,100};
double fibo_upper[] = {123.6,138.2,150,161.8,178.6,200,261.8,361.8,423.6};
double fibo_lower[] = {-23.6,-38.2,-50,-61.8,-78.6,-100,-161.8,-261.8,-323.6};
```
- `fibo_main[]` Represents the main Fibonacci levels.
- `fibo_upper[]` Represents the upper Fibonacci levels.
- `fibo_lower[]` Represents the lower Fibonacci levels.

The values in these arrays determine the specific Fibonacci levels that will be displayed on the chart. Users can adjust these values according to their preference to customize the appearance of the Fibonacci levels on the chart.

## Script Code
Below is the MQL5 code used to create the "Fibo_Bar" levels
```mql5
//+------------------------------------------------------------------+
//|                                                     Fibo_Bar.mq5 |
//|                          Copyright 2024, Tecciztecatl and FaceND |
//|                               https://github.com/FaceND/Fibo_Bar |
//+------------------------------------------------------------------+
#property copyright     "Copyright 2024, Tecciztecatl and FaceND"
#property link          "https://github.com/FaceND/Fibo_Bar"
#property version       "2.0"
#property description   "This indicator automatically calculates and plots Fibonacci retracement levels"
#property description   "on the most recent bar of the chart. traders can quickly assess the current market conditions"
#property description   "and make more informed trading decisions."
#property strict
#property indicator_chart_window
#property indicator_plots 0

enum ENUM_TYPE_PERIOD
{
 CURRENT  =  0,       // Current
 PREVIOUS =  1        // Previous
};

enum ENUM_POSITION
{
 RIGHT         =  1,  // Right
 CURRENT_PRICE =  0   // Current Price
};

input group "PERIOD MODE"
input ENUM_TYPE_PERIOD  Period_Type  =  PREVIOUS;           // Represents the type period
input ENUM_TIMEFRAMES   Fibo_Bar     =  PERIOD_D1;          // Range period

input group "LEVELS"
input ENUM_POSITION     Fibo_Pos     =  RIGHT;              // Line position
input color             fibo_color0  =  clrLimeGreen;       // Main color
input color             fibo_color1  =  clrSkyBlue;         // Upper color
input color             fibo_color2  =  clrOrange;          // Lower color
input ENUM_LINE_STYLE   fibo_style   =  STYLE_DOT;          // Style lines
input int               fibo_width   =  1;                  // Line width

double fibo_main[]  = {0,23.6,38.2,50,55.9,61.8,66.7,78.6,100};
double fibo_upper[] = {123.6,138.2,150,161.8,178.6,200,261.8,361.8,423.6};
double fibo_lower[] = {-23.6,-38.2,-50,-61.8,-78.6,-100,-161.8,-261.8,-323.6};

string Label_prefix="Fibo_";
double FIBO_levels[];
int    allBars;
long   chart_ID;
//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
int OnInit()
  {
   chart_ID=ChartID();
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Custom indicator deinitialization function                       |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
   ObjectsDeleteAll(chart_ID,Label_prefix,-1,-1);
   ChartRedraw(chart_ID);
  }
//+------------------------------------------------------------------+
//| Chart event handler                                              |
//+------------------------------------------------------------------+
void OnChartEvent(const int                 id,
                  const long           &lparam,
                  const double         &dparam,
                  const string         &sparam)
  {
   if(id==CHARTEVENT_CHART_CHANGE)
     {
      BuildLevels();
      ChartRedraw(chart_ID);
     }
  }
//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(const int           rates_total,
                const int       prev_calculated,
                const datetime          &time[],
                const double            &open[],
                const double            &high[],
                const double             &low[],
                const double           &close[],
                const long       &tick_volume[],
                const long            &volume[],
                const int             &spread[])
  {
   ResetLastError();
   if(allBars!=Bars(_Symbol,Fibo_Bar) || ObjectFind(chart_ID,Label_prefix+"f0")<0)
     {
      allBars=Bars(_Symbol,Fibo_Bar);
      BuildLevels();
     }
   return(rates_total);
  }
//+------------------------------------------------------------------+
//| Build The Fibonacci Levels object                                |
//+------------------------------------------------------------------+
void BuildLevels()
  {
   double   Maximum=iHigh(NULL,Fibo_Bar,Period_Type);
   double   Minimum=iLow(NULL,Fibo_Bar,Period_Type);
   datetime time1=iTime(NULL,Fibo_Bar,Period_Type);
   datetime time2=TimeCurrent();
   int      Start_Bar=0,End_Bar,High_Index,Low_Index;
   //+---------------------------------------------------------------+
   //--- Current Period
   if(Period_Type==0)
     {
      Maximum=fmax(Maximum,iHigh(NULL,Fibo_Bar,Period_Type));
      Minimum=fmin(Minimum,iLow(NULL,Fibo_Bar,Period_Type));
     }
   //--- Previous Period
   else if(Period_Type==1)
     {
      Start_Bar=iBarShift(NULL,_Period,iTime(NULL,Fibo_Bar,0));
     }
   //+---------------------------------------------------------------+
   End_Bar=iBarShift(NULL,_Period,iTime(NULL,Fibo_Bar,Period_Type))-Start_Bar;
   High_Index=iHighest(NULL,0,MODE_HIGH,End_Bar,Start_Bar);  
   Low_Index=iLowest( NULL,0,MODE_LOW,End_Bar,Start_Bar);
   //+---------------------------------------------------------------+
   if(Low_Index>High_Index)
     {
      Swap(Maximum,Minimum);
     }
   MakeFibo(fibo_main);
   SetFibo(Label_prefix+"Main",time1,Maximum,time2,Minimum,fibo_color0,fibo_width);
   if(Period_Type!=0)
     {
      MakeFibo(fibo_upper);
      SetFibo(Label_prefix+"Upper",time1,Maximum,time2,Minimum,fibo_color1,fibo_width);

      MakeFibo(fibo_lower);
      SetFibo(Label_prefix+"Lower",time1,Maximum,time2,Minimum,fibo_color2,fibo_width);
     }
   SetHighLow(Label_prefix+"Main",Maximum,Minimum);
  }
//+------------------------------------------------------------------+
//| Sets up and creates a Fibonacci object                           |
//+------------------------------------------------------------------+
void SetFibo(const string     name,
             datetime        start,
             double           high,
             datetime          end,
             double            low,
             color            cvet,
             int             width)
  {
   int    levels=ArraySize(FIBO_levels);
   double fiboPrice;
   //+---------------------------------------------------------------+
   ResetLastError();
   if(ObjectFind(chart_ID,name)<0)
     {
      if(!ObjectCreate(chart_ID,name,OBJ_FIBO,0,start,high,end,low))
        {
         Print(__FUNCTION__,
            ": Failed to create Fibonacci Object [",name,"] = ",GetLastError());
         return;
        }
      ObjectSetInteger(chart_ID,name,OBJPROP_COLOR,clrNONE);
      ObjectSetInteger(chart_ID,name,OBJPROP_SELECTABLE,false);
      ObjectSetInteger(chart_ID,name,OBJPROP_SELECTED,false);
      ObjectSetInteger(chart_ID,name,OBJPROP_BACK,false);
      ObjectSetInteger(chart_ID,name,OBJPROP_RAY_RIGHT,Fibo_Pos);
      ObjectSetInteger(chart_ID,name,OBJPROP_HIDDEN,true);
     }
   else
     {
      ObjectMove(chart_ID,name,0,start,high);
      ObjectMove(chart_ID,name,1,end,low);
     }
   ObjectSetInteger(chart_ID,name,OBJPROP_LEVELS,levels);
   //+---------------------------------------------------------------+
   for(int i=0;i<levels;i++)
      {
       fiboPrice=NormalizeDouble(((high-low)*FIBO_levels[i]) + low,_Digits);
       ObjectSetDouble(chart_ID,name,OBJPROP_LEVELVALUE,i,FIBO_levels[i]);
       ObjectSetString(chart_ID,name,OBJPROP_LEVELTEXT,i,DoubleToString(100*FIBO_levels[i],1)+"%  ("+DoubleToString(fiboPrice,_Digits)+")");
       ObjectSetInteger(chart_ID,name,OBJPROP_LEVELCOLOR,i,cvet);
       ObjectSetInteger(chart_ID,name,OBJPROP_LEVELWIDTH,i,width);
       ObjectSetInteger(chart_ID,name,OBJPROP_LEVELSTYLE,i,fibo_style);
      }
  }
//+------------------------------------------------------------------+
//| Sets the high and low text for the Fibonacci levels              |
//+------------------------------------------------------------------+
void SetHighLow(const string     name,
                double           high,
                double            low)
  {
   int maxArr=ArraySize(FIBO_levels)-1;
   string high_text,low_text;
   //+---------------------------------------------------------------+
   //--- Current Period
   if(Period_Type==0)
     {
      high_text = "High";
      low_text  = "Low";
     }
   //--- Previous Period
   else if(Period_Type==1)
     {
      high_text = "Last High";
      low_text  = "Last Low";
     }
   //+---------------------------------------------------------------+
   if(high>low)
     {
      Swap(high_text,low_text);
     }
   ObjectSetString(chart_ID,name,OBJPROP_LEVELTEXT,0,high_text+" "+ObjectGetString(chart_ID,name,OBJPROP_LEVELTEXT,0));
   ObjectSetString(chart_ID,name,OBJPROP_LEVELTEXT,maxArr,low_text+" "+ObjectGetString(chart_ID,name,OBJPROP_LEVELTEXT,maxArr));
  }
//+------------------------------------------------------------------+
//| Match the number of Fibonacci elements                           |
//+------------------------------------------------------------------+
void MakeFibo(double   &value[])
  {
   int num_levels=ArraySize(value);
   ArrayResize(FIBO_levels,num_levels);
   for(int i=0;i<num_levels;i++)
      {
       FIBO_levels[i]=value[i]/100;
      }
   ArraySort(FIBO_levels);
  }
//+------------------------------------------------------------------+
//| Function swaps the values of two variables                       |
//+------------------------------------------------------------------+
template <typename T>
void Swap(T   &value1,
          T   &value2)
  {
   T tmp=value1;
   value1=value2;
   value2=tmp;
  }
//+------------------------------------------------------------------+
```

## Contributing

1. Fork the repository.
2. Create a new branch for your feature or fix.
3. Commit your changes with descriptive commit messages.
4. Push your branch to your forked repository.
4. Open a pull request to the main repository's main branch.
5. Please ensure your code follows the existing code style and includes comments where necessary.

Please ensure your code follows the existing code style and includes comments where necessary.

## Acknowledgments

- [MetaTrader](https://www.metatrader.com/) The trading platform where this indicator can be used.
- [Investopedia](https://www.investopedia.com/terms/f/fibonacciretracement.asp) Information on Fibonacci retracement levels.

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Credits

This project includes code based on [Tecciztecatl](https://www.mql5.com/en/users/tecciztecatl)'s work. The original code can be found [here](https://www.mql5.com/en/code/16327).
