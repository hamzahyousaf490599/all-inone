// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © Bjorgum

//   ________________________________________________________________ 
//  |\______________________________________________________________/| 
//  ||   ________________                                           || 
//  ||   ___  __ )_____(_)___________________ ____  ________ ___    || 
//  ||   __  __  |____  /_  __ \_  ___/_  __ `/  / / /_  __ `__ \   || 
//  ||   _  /_/ /____  / / /_/ /  /   _  /_/ // /_/ /_  / / / / /   ||
//  ||   /_____/ ___  /  \____//_/    _\__, / \__,_/ /_/ /_/ /_/    ||
//  ||           /___/                /____/                        ||
//  ||______________________________________________________________||
//  |/______________________________________________________________\|

//@version=5
// @strategy_alert_message {{strategy.order.alert_message}}

strategy(
 title                  =   "Bjorgum Double Tap",
 shorttitle             =   "Bj Double Tap",
 overlay                =   true, 
 max_lines_count        =   500, 
 max_labels_count       =   500, 
 precision              =   3, 
 default_qty_type       =   strategy.cash, 
 commission_value       =   0.04, 
 commission_type        =   strategy.commission.percent, 
 slippage               =   1, 
 currency               =   currency.USD, 
 default_qty_value      =   1000, 
 initial_capital        =   1000)

// ══════════════════════════════════ //
// —————> Immutable Constants <—————— //
// ══════════════════════════════════ //

string      useStratTip =   "Enables or disables the strategy tester allowing a change between either an indicator or a strategy."
string      dLongTip    =   "Detect long setups."
string      dShortTip   =   "Detect short setups."
string      FLIPTip     =   "Allow entry in the opposite bias while already in a position."

string      startTip    =   "Start date & time to begin backtest period. Useful for beginning new bot. eg. Set time to now to make broker emulator in a flat position with the proper starting captial before setting alerts"
string      endTip      =   "End date & time to stop searching for setups for back testing."

string      tolTip      =   "The difference in height allowable for the signifcant points of the pattern expressed as a percent of the pattern height. Example: The points can vary in height by 15% of the pattern height from one another."
string      lenTip      =   "The length used to calcuate significant points to form pivots for pattern detection. Example: The highest or lowest point in 50 bars."
string      fibTip      =   "The fib target extension projected from the neckline of the pattern in the direction of the pattern bias expressed as a percent. Example: 100% is a 1:1 measurment of the height from the pattern neckline."
string      stopPerTip  =   "The fib extension of the pattern height measured from the point of invalidation. Example: 0% would be the high point of a double top. 50% would be halfway between the top and the neckline."
string      offsetTip   =   "The number of bars lines are extended into the future during an ongoing pattern."

string      atrStopTip  =   "Enables an ATR trailing stop once the target extension is breached. NOTE: This disables a take profit order in the strategy format."
string      atrLenTip   =   "The number of bars used in the ATR calculation."
string      atrMultTip  =   "The multiplier of the ATR value to subtract from the swing low or swing high point. Example: 1 is 100% of the ATR, 2 is 2x the ATR value."
string      lookbackTip =   "The number of bars to look back to find a swing high or swing low to be used in the trailing stop calculation. Example: 1 ATR subtracted from the lowest point in 5 bars. 1 would use the lowest point within the last bar."

string      tableTip    =   "Show the data table for trade limit levels during an active trade. (table will only show when a pattern is present on the chart, or in bar replay)."
string      labelTip    =   "Updates the double top/bottom label with 'Success' or 'Failure' and updates color based on the outcome of the pattern."

string      thirdTip    =   "Where would you like to send your alerts?"
string      pPhraseTip  =   "A custom passphrase to authenticate your json message with a touch more security."
string      aTronKeyTip =   "The name of your Alertatron API keys. (Add the ticker in brackets). Example: MyKeys(XBTUSD)."
string      tickerTip   =   "The ticker to be traded when using Trading Connector"
string      LongTip     =   "3Commas start long deal bot keys."
string      LongEndTip  =   "3Commas end long deal bot keys."
string      ShortTip    =   "3Commas start short deal bot keys."
string      ShortEndTip =   "3Commas end short deal bot keys."

string      dt          =   "Double Top"   
string      db          =   "Double Bottom"
string      suc         =   " - Success"  
string      fail        =   " - Failure"
string      winStr      =   "Target reached! 💰" 
string      loseStr     =   "Stopped out... 🤷‍♂"
string      tabPerc     =   "{0, number, #.##}\n({1, number, #.##}%)"     
string      tcStop      =   "slmod slprice={0} tradeid={1}"
string      dExit       =   "'{'\"content\": \"```Bjorgum {0}\\n\\n\\t'{{ticker}}' '{{interval}}'\\n\\n\\t{1}```\"'}'" 

string      S1          =   "tiny"   ,  string P1 = "top" 
string      S2          =   "small"  ,  string P2 = "middle" 
string      S3          =   "normal" ,  string P3 = "bottom" 
string      S4          =   "large"  ,  string P4 = "left" 
string      S5          =   "huge"   ,  string P5 = "center"
string      S6          =   "auto"   ,  string P6 = "right"
var string  tnB         =   ""       ,  string A1 = "Custom Json"
string      altStr      =   ""       ,  string A2 = "Trading Connector"
string      tUp         =   ""       ,  string A3 = "Alertatron"
string      dCordWin    =   ""       ,  string A4 = "3Commas"
string      dCordLose   =   ""       ,  string A5 = "Discord"
 
float       pos         =   strategy.position_size
int         sync        =   bar_index
bool        confirm     =   barstate.isconfirmed
var int     dir         =   na
var float   lmt         =   na 
var float   stp         =   na 
string      altExit     =   na

bool        FLAT        =   pos == 0
bool        LONG        =   pos >  0
bool        SHORT       =   pos <  0
var int     tradeId     =   0

color       col1        =   color.new(#b2b5be, 15)
color       col2        =   color.new(#b2b5be, 87)
color       col3        =   color.new(#ffffff,  0)
color       col4        =   color.new(#17ff00, 15)
color       col5        =   color.new(#ff0000, 15)
color       col6        =   color.new(#ff5252,  0)

var matrix<float> logs  =   matrix.new<float>  (5, 3)
var line [] zLines      =   array.new_line     (5)
var line [] tLines      =   array.new_line     (5)
var line [] bLines      =   array.new_line     (5)
var label[] bullLb      =   array.new_label     ()
var label[] bearLb      =   array.new_label     ()

int timeStart           =   timestamp("01 Jan 2000")
int timeEnd             =   timestamp("01 Jan 2099")

// ══════════════════════════════════ //
// —————————> User Input <——————————— //
// ══════════════════════════════════ //

string      GRP1        =   "════  Detection and Trade Parameters  ════"
bool        useStrat    =   input.bool  (true     , "Use Strategy"    , group= GRP1, tooltip= useStratTip)
bool        dLong       =   input.bool  (true     , "Detect Bottoms"  , group= GRP1, tooltip= dLongTip   )
bool        dShort      =   input.bool  (true     , "Detect Tops"     , group= GRP1, tooltip= dShortTip  )
bool        FLIP        =   input.bool  (true     , "Flip Trades"     , group= GRP1, tooltip= FLIPTip    )       
float       tol         =   input.float (15       , "Pivot Tolerance" , group= GRP1, tooltip= tolTip     , minval= 1)
int         len         =   input.int   (50       , "Pivot Length"    , group= GRP1, tooltip= lenTip     , minval= 1)
float       fib         =   input.float (100      , "Target Fib"      , group= GRP1, tooltip= fibTip     , minval= 0)
int         stopPer     =   input.int   (0        , "Stop Loss Fib"   , group= GRP1, tooltip= stopPerTip )
int         offset      =   input.int   (30       , "Line Offset"     , group= GRP1, tooltip= offsetTip  , minval= 0)

string      GRP2        =   "═══════════ Time Filter ═══════════"
int         startTime   =   input.time(timeStart  , "Start Filter"    , group= GRP2, tooltip= startTip)      
int         endTime     =   input.time(timeEnd    , "End Filter"      , group= GRP2, tooltip= endTip  )        

string      GRP3        =   "══════════ Trailing Stop ══════════"
bool        atrStop     =   input.bool  (false    , "Use Trail Stop"  , group= GRP3, tooltip= atrStopTip )
int         atrLength   =   input.int   (14       , "ATR Length"      , group= GRP3, tooltip= atrLenTip  , minval= 1)
float       atrMult     =   input.float (1        , "ATR Multiplier"  , group= GRP3, tooltip= atrMultTip , minval= 0)
int         lookback    =   input.int   (5        , "Swing Lookback"  , group= GRP3, tooltip= lookbackTip, minval= 1)

string      GRP5        =   "════════════ Colors ════════════"
color       col         =   input.color (col1     , "Lines        "   , group= GRP5, inline= "41")
color       zCol        =   input.color (col3     , "Patterns       " , group= GRP5, inline= "42")
int         hWidth      =   input.int   (1        , ""                , group= GRP5, inline= "41", minval= 1)
int         zWidth      =   input.int   (1        , ""                , group= GRP5, inline= "42", minval= 1)
color       colf        =   input.color (col2     , "Stop Fill"       , group= GRP5)
color       tCol        =   input.color (col4     , "Target Color"    , group= GRP5)
color       sCol        =   input.color (col5     , "Stop Color"      , group= GRP5)
color       trailCol    =   input.color (col6     , "Trail Color"     , group= GRP5)

string      GRP6        =   "═════════  Table and Label  ═════════"
bool        showTable   =   input.bool  (true     , "Show Table"      , group= GRP6, tooltip= tableTip) 
bool        setLab      =   input.bool  (true     , "Update Label"    , group= GRP6, tooltip= labelTip)
string      labSize     =   input.string("small"  , "Label Text Size" , group= GRP6, options= [S1, S2, S3, S4, S5, S6])
string      textSize    =   input.string("normal" , "Table Text Size" , group= GRP6, options= [S1, S2, S3, S4, S5, S6])
string      tableYpos   =   input.string("bottom" , "Table Position"  , group= GRP6, options= [P1, P2, P3])
string      tableXpos   =   input.string("right"  , ""                , group= GRP6, options= [P4, P5, P6])

string      GRP7        =   "══════════ Alert Strings ══════════"
string      thirdParty  =   input.string(A1       , "3rd Party"       , group= GRP7, tooltip= thirdTip, options= [A1, A2, A3, A4, A5])
string      pPhrase     =   input.string("1234"   , "Json Passphrase" , group= GRP7, tooltip= pPhraseTip ) 
string      aTronKey    =   input.string("myKeys" , "Alertatron Key"  , group= GRP7, tooltip= aTronKeyTip)
string      tcTicker    =   input.string(""       , "TC Ticker"       , group= GRP7, tooltip= tickerTip  )
string      c3Long      =   input.string(""       , "3Comma Long"     , group= GRP7, tooltip= LongTip    )
string      c3LongEnd   =   input.string(""       , "3Comma Long End" , group= GRP7, tooltip= LongEndTip )
string      c3Short     =   input.string(""       , "3Comma Short"    , group= GRP7, tooltip= ShortTip   )
string      c3ShortEnd  =   input.string(""       , "3Comma Short End", group= GRP7, tooltip= ShortEndTip)

// ══════════════════════════════════ //
// ————> Variable Calculations <————— //
// ══════════════════════════════════ //
    
bool        dif         =   stopPer != 0 
int         set         =   sync + offset

float       atr         =   ta.atr          (14)
float       sLow        =   ta.lowest       (lookback) - (atr * atrMult)
float       sHigh       =   ta.highest      (lookback) + (atr * atrMult)

float       pivHigh     =   ta.highest      (len)
float       pivLows     =   ta.lowest       (len)

float       hbar        =   ta.highestbars  (len)
float       lbar        =   ta.lowestbars   (len)

// ══════════════════════════════════ //
// ———> Functional Declarations <———— //
// ══════════════════════════════════ //

High(m)  => 
    float result = (m == 1 ? high : low)
    
Low(m)   => 
    float result = (m == 1 ? low  : high)
    
perD(_p) => 
    float result = (_p - close) / close * 100

_coords(_x, _i)                      =>
    x = matrix.get                   (_x, _i, 0)
    y = matrix.get                   (_x, _i, 1)
    [int(x), y]

_arrayLoad(_x, _max, _val)           =>  
    array.unshift                    (_x,   _val)   
    if  array.size                   (_x) > _max
        array.pop                    (_x)

_matrixPush(_mx, _max, _row)         =>
    matrix.add_row(_mx, matrix.rows  (_mx),  _row)
    if matrix.rows                   (_mx) > _max
        matrix.remove_row            (_mx, 0)

_mxLog(_cond, _x, _y)                =>
    float[] _row = array.from        (sync, _y, 0)
    if _cond 
        _matrixPush                  (_x, 5, _row)

_mxUpdate(_cond, _dir, _x, y)        =>
    int m    = _dir ? 1 : -1
    int _end = matrix.rows           (_x) -1
    if  _cond and y * m > matrix.get (_x, _end, 1) * m 
        matrix.set                   (_x, _end, 0, sync)
        matrix.set                   (_x, _end, 1, y)

_extend(_x, _len) =>
    for l in _x
        line.set_x2(l, _len)

_hLine(_l, x2, y2, y3, y4, y5, _t)   =>
    line l1 =             line.new   (x2    , y2, set, y2, color= col,  width= hWidth)
    line l2 =             line.new   (x2    , y4, set, y4, color= col,  width= hWidth)
    array.set (_l    , 3, l1)
    array.set (_l    , 2, l2)
    array.set (_l    , 1, line.new   (x2    , y3, set, y3, color= col,  width= hWidth))
    array.set (_l    , 0, line.new   (sync-1, _t, set, _t, color= col,  width= hWidth))
    linefill.new                     (l1    , l2, colf)
    if stopPer != 0
        array.set (_l, 4, line.new   (sync-1, y5, set, y5, color= col,  width= hWidth))

_zLine(x1, y1, x2, y2, x3, y3, x4, y4) =>
    array.set (zLines, 3, line.new   (x1, y1, x2  , y2   , color= zCol, width= zWidth))
    array.set (zLines, 2, line.new   (x2, y2, x3  , y3   , color= zCol, width= zWidth))
    array.set (zLines, 1, line.new   (x3, y3, x4  , y4   , color= zCol, width= zWidth))
    array.set (zLines, 0, line.new   (x4, y4, sync, close, color= zCol, width= zWidth))

_label(x, y, m) =>
    m > 0 ?  
   _arrayLoad (bearLb, 1, label.new  (x, y, dt, color= color(na), style= label.style_label_down, textcolor= col, size= labSize)) : 
   _arrayLoad (bullLb, 1, label.new  (x, y, db, color= color(na), style= label.style_label_up,   textcolor= col, size= labSize))

_labelUpdate(_x, _y, m)              =>
    if (_x or _y) and setLab
        label  lab  = array.get      (m > 0 ? bearLb : bullLb, 0)
        string oStr =                (m > 0 ? dt     : db)
        string nStr = oStr +         (_x    ? suc    : fail)
        label.set_text               (lab, nStr)
        label.set_textcolor          (lab, _x ? tCol : sCol)

_atrTrail(_cond, _lt, _s, m)         =>
    var float _stop  = na
    var bool  _flag  = na
    var bool  _trail = na
    _flag           := _cond
    _stop           := _s
    _trail          := _flag ? false : _trail
    if atrStop and useStrat
        if  _lt
            _flag   := false 
            _trail  := true  
        _stop       := m == -1 ? _lt ? sLow  : math.max(_stop, sLow)  : _stop 
        _stop       := m ==  1 ? _lt ? sHigh : math.min(_stop, sHigh) : _stop 
    if High(m) * m > _stop * m 
        _flag       := true
        _trail      := false
    [_flag, _stop, _trail]

_inTrade(_cond, _x, _e, m)           =>
    var bool  _flag  =               na
    var float _stop  =               na 
    var float _limit =               na
    line      l1     = array.get     (_x, 0)
    float     lp     = line.get_price(l1, sync)
    line      l2     = array.get     (_x, dif ? 4 : 2)
    float     ls     = line.get_price(l2, sync)
    bool      win    = Low (m) *     m <= lp * m and not _e
    bool      lose   = High(m) *     m >= ls * m and not _e
    _flag           := _cond
    _stop           := _e ? ls :     _stop
    _limit          := _e ? lp :     _limit
    if win or lose  
        _flag       :=               true
        _extend                      (_x    , sync)
        _labelUpdate                 (win   , lose, m)
        line.set_color               (win   ? l1 : l2, win ? tCol : sCol)
        array.fill                   (_x    , na)
        array.fill                   (zLines, na)
    [_f, _s, _t]     = _atrTrail     (_flag , win  , _stop, m)
    _flag           :=               atrStop  ? _f : _flag
    _stop           :=               _t and _s * m < _stop * m and confirm ? _s : _stop
    [_flag, _stop, _t, _limit]

_double(_cond, _l, _x, m)            =>
    var bool _flag   = na
    int _rows        = matrix.rows   (_x)
    _flag           := _cond
    if _flag
        [x1, y1]     = _coords       (_x , _rows - 5)
        [x2, y2]     = _coords       (_x , _rows - 4)
        [x3, y3]     = _coords       (_x , _rows - 3)
        [x4, y4]     = _coords       (_x , _rows - 2)
        bool  traded = matrix.get    (_x , _rows - 2, 2)
        float height = math.avg      (y2 , y4)   - y3
        float _high  = y2 + height * (tol/ 100)
        float _low   = y2 - height * (tol/ 100)
        float _t     = y3 - height * (fib/ 100)
        float y5     = y2 * m < y4 * m ? y2 : y4
        float y6     = y2 * m > y4 * m ? y2 : y4
        float y7     = y6 - height *(stopPer/100)    
        bool result  = y1*m < y3*m and y4*m <= _high*m and y4*m >= _low*m and close*m < y3*m and not (close[1]*m < y3*m) and not traded
        if result and _flag and (m > 0 ? dShort : dLong)
            _hLine       (_l, x2, y5, y3, y6, y7, _t)
            _zLine       (x1, y1, x2, y2, x3, y3, x4, y4)
            _label       (x4, y6, m)
            matrix.set   (_x, _rows - 2, 2, 1)
            _flag :=     false
    _flag 

_scan(_l, _x, m)     =>
    var bool _cond   = true
    _cond           := _double       (_cond, _l, _x, m)
    bool enter       = _cond[1]      and not _cond
    [f,s,t,l]        = _inTrade      (_cond, _l, enter, m)
    _cond           := f
    _extend(_l, set)
    [f,s,t,l]

_populate(_n, _x, _i, _col) => 
    for [i, _a] in _x 
        if not na(_a)
            table.cell(table_id  = _n,  column     = _i  , 
                       row       =  i,  bgcolor    = na  , 
                       text      = _a,  text_color = _col, 
                       text_size =      textSize)
    
// ══════════════════════════════════ //
// ————————> Logical Order <————————— //
// ══════════════════════════════════ //

dir            :=  not hbar ? 1 : not lbar ? 0 : dir

bool dirUp      =  dir != dir[1] and     dir 
bool dirDn      =  dir != dir[1] and not dir 

bool setUp      =  not hbar and     dir 
bool setDn      =  not lbar and not dir 

_mxLog             (dirUp or dirDn,      logs, dirUp ? pivHigh : pivLows)
_mxUpdate          (setUp or setDn, dir, logs, setUp ? pivHigh : pivLows)

[bear,ss,ts,sl] =  _scan(tLines, logs,  1)
[bull,ls,tl,ll] =  _scan(bLines, logs, -1)

bool st         =  SHORT ? ts : false
bool lt         =  LONG  ? tl : false

bool sell       =  bear[1] and not bear 
bool buy        =  bull[1] and not bull

color ssCol     =  st or st[1] ? trailCol : na
color lsCol     =  lt or lt[1] ? trailCol : na

bool longEntry  =  buy  and (FLAT or (SHORT and FLIP)) 
bool shortEntry =  sell and (FLAT or (LONG  and FLIP)) 

bool dateFilter =  time >= startTime and time <= endTime

tradeId        +=  longEntry or shortEntry ? 1 : 0

lmt            :=  atrStop    ? na :
                   shortEntry ? sl : longEntry ? ll : lmt 

stp            :=  shortEntry or SHORT and atrStop ? ss : 
                   longEntry  or LONG  and atrStop ? ls : stp
                   
plot               (atrStop ? ss : na, "Short Stop", ssCol, style= plot.style_linebr)
plot               (atrStop ? ls : na, "Long Stop",  lsCol, style= plot.style_linebr)

bgcolor            (not dateFilter ? color.new(color.red,80) : na, title= "Filter Color")

// ══════════════════════════════════ //
// —————————> Data Display <————————— //
// ══════════════════════════════════ //

if showTable
    
    string   tls    = bull ? na : str.format(tabPerc, ls, perD(ls))
    string   tss    = bear ? na : str.format(tabPerc, ss, perD(ss))
    string   tll    = bull ? na : str.format(tabPerc, ll, perD(ll))
    string   tsl    = bear ? na : str.format(tabPerc, sl, perD(sl))
    
    string[] titles = array.from(na, bull ? na : "Bullish", bear ? na : "Bearish")
    
    string[] stops  = array.from("Stop"  , tls, tss)
    string[] limtis = array.from("Target", tll, tsl)
    
    table    bjTab  = table.new(tableYpos + "_" + tableXpos, 3, 3, border_color= color.new(color.gray, 60), border_width= 1)
    
    if not bear or not bull 
        _populate(bjTab, titles, 0, color.white)
        _populate(bjTab, stops,  1, color.red)
        if not (na(ll) or lt) or not (na(sl) or st)
            _populate(bjTab, limtis, 2, color.green)

// ══════════════════════════════════ //
// ——————> String Variables <———————— //
// ══════════════════════════════════ //

bool cSon  = thirdParty == A1
bool tCon  = thirdParty == A2
bool aTron = thirdParty == A3
bool c3    = thirdParty == A4
bool dCord = thirdParty == A5

if cSon and useStrat
    
    string json = 
    
     "'{'
     \n    \"passphrase\": \"{0}\",
     \n    \"time\": '\"{{timenow}}\"',
     \n    \"ticker\": '\"{{ticker}}\"',
     \n    \"plot\": '{'
     \n        \"stop_price\": {1, number, #.########},
     \n        \"limit_price\": {2, number, #.########}
     \n    '}',
     \n    \"strategy\": '{'
     \n        \"position_size\": '{{strategy.position_size}}',
     \n        \"order_action\": '\"{{strategy.order.action}}\"',
     \n        \"market_position\": '\"{{strategy.market_position}}\"',
     \n        \"market_position_size\": '{{strategy.market_position_size}}',
     \n        \"prev_market_position\": '\"{{strategy.prev_market_position}}\"',
     \n        \"prev_market_position_size\": '{{strategy.prev_market_position_size}}'
     \n    '}'
     \n'}'"
    
    altStr := str.format(json, pPhrase, stp, lmt)

if tCon and useStrat

    string tcTrade = 

     "'{{strategy.order.action}}' tradesymbol={0} tradeid={1}" 
     + (atrStop ? "" : ' tpprice={2, number, #.########}') 
     + ' slprice={3, number, #.########}' 
    
    altStr  := str.format(tcTrade, tcTicker, tradeId, lmt, stp)
    tUp     := str.format(tcStop, stp, tradeId)

if aTron and useStrat

    string altEnt  = aTronKey + " '{'\ncancel(which=all);\nmarket(position='{{strategy.position_size}}');\n"
    string altEnd  = "'}'\n#bot"
    string altBrkt = 

       (atrStop ? "stopOrder(" : "stopOrTakeProfit(")
     + (atrStop ? "" : "tp=@{0, number, #.########}, ")
     + (atrStop ? "offset=@" : "sl=@") + "{1, number, #.########}, position=0, reduceOnly=true"
     + (atrStop ? ", tag=trail" : "")
     + ");\n"
    
    string enter = altEnt + altBrkt + altEnd 
    string exit  = altEnt + altEnd
    
    altStr  := str.format(enter, lmt, stp)
    altExit := na(altExit) ? exit : altExit

    string stopUpdate = aTronKey + " '{'\ncancel(which=tagged, tag=trail);\n" + altBrkt + altEnd
    
    tUp := atrStop ? str.format(stopUpdate, lmt, stp) : tUp

if dCord and useStrat

    tnB := longEntry ? db : shortEntry ? dt : tnB

    string postTrade = 
    
     "'{'\"content\": \"```🚨 Bjorgum {0} detected 🚨\\n\\n\\t'{{ticker}}' '{{interval}}'\\n\\n\\t"
     + (atrStop ? "" : "🎯 Target: {1, number, #.########}\\n\\t") 
     + "🛑 Stop:   {2, number, #.########}```\"'}'"

    altStr    := str.format(postTrade, tnB, lmt, stp)
    dCordWin  := str.format(dExit, tnB, winStr)
    dCordLose := str.format(dExit, tnB, loseStr)

if c3 and useStrat

    c3Long  := SHORT and buy  ? str.format("[{0}, {1}]", c3ShortEnd, c3Long)  : c3Long 
    c3Short := LONG  and sell ? str.format("[{0}, {1}]", c3LongEnd,  c3Short) : c3Short

// ══════════════════════════════════ //
// ——————> Strategy Execution <—————— //
// ══════════════════════════════════ //

strategy.entry("Long" , strategy.long , comment= "Long",  when= useStrat and longEntry  and dateFilter, alert_message= c3 ? c3Long  : altStr)
strategy.entry("Short", strategy.short, comment= "Short", when= useStrat and shortEntry and dateFilter, alert_message= c3 ? c3Short : altStr)

strategy.exit("Long Exit", 
              "Long",  
              stop          = stp, 
              limit         = lmt, 
              comment       = "L Exit", 
              alert_message = c3    ? c3LongEnd : aTron ? altExit : tCon ? "closelong"  : dCord ? na : altStr, 
              alert_profit  = dCord ? dCordWin  : na, 
              alert_loss    = dCord ? dCordLose : na)

strategy.exit("Short Exit", 
              "Short", 
              stop          = stp, 
              limit         = lmt, 
              comment       = "S Exit", 
              alert_message = c3    ? c3ShortEnd : aTron ? altExit : tCon ? "closeshort" : dCord ? na : altStr, 
              alert_profit  = dCord ? dCordWin   : na, 
              alert_loss    = dCord ? dCordLose  : na)
    
// ══════════════════════════════════ //
// —————> Alert Functionality <—————— //
// ══════════════════════════════════ //

if (lt and ls != ls[1] or st and ss != ss[1]) and (tCon or aTron)
    alert(tUp, alert.freq_once_per_bar_close)

if not useStrat and (buy or sell)
    alert((buy ? db : dt) + ' Detected', alert.freq_once_per_bar)

//  ____  __ _  ____ 
// (  __)(  ( \(    \
//  ) _) /    / ) D (
// (____)\_)__)(____/


// █▀▀▄ ──▀ █▀▀█ █▀▀█ █▀▀▀ █──█ █▀▄▀█ 
// █▀▀▄ ──█ █──█ █▄▄▀ █─▀█ █──█ █─▀─█ 
// ▀▀▀─ █▄█ ▀▀▀▀ ▀─▀▀ ▀▀▀▀ ─▀▀▀ ▀───▀

// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © Bjorgum

//@version=5
indicator           ('Bjorgum Key Levels', 'Bj Key Levels', overlay= true, max_boxes_count= 500, max_labels_count= 500, max_lines_count=500)

import              Bjorgum/BjCandlePatterns/2 as bj

// ================================== //
// ------------> Tips <-------------- //
// ================================== //

leftTip         =   "Look left for swing high/low in x number of bars to form pivot. The higher the number, the higher the script looks to the left for the highest/lowest point before drawing pivot"        
rightTip        =   "Look right for swing high/low in x number of bars to form pivot. The higher the number, the higher the script looks to the right for the highest/lowest point before drawing pivot"       
nPivTip         =   "This sets the array size, or the number of pivots to track at a time (x highs, and x number of lows)" 
atrLenTip       =   "Number of bars to average. ATR is used to standardize zone width between assets and timeframes"     
multTip         =   "ATR multiplier to set zone width. Default is half of one ATR from box bottom to box top"     
perTip          =   "Max zone size as a percent of price. Some assets can be too volatile at low prices creating an unreasonably sized zone"
maxTip          =   "Number of boxes for candlestick patterns to track historically. Note: the higher the number the less pivot zones will be tracked when looking back in time due to the limitation on the number of box elements allowed at once"
futTip          =   "Number of bars to offset labels for price levels"
srcTip          =   "Source input for pivots. Default tracks the highest and lowest bodies of HA candles to average price action, which can result in a level that sits in the overlap of support and resistance"     
alignZonesTip   =   "Aligns recurring zones who's edges overlap an existing zone creating a zone that ages in time and intensifies visually"     
extendTip       =   "Extends current zones right"
lLabTip         =   "Show labels for price levels extended off Key Levels"

dhighsTip       =   "Disabling will prevent highs from being tracked"          
dlowsTip        =   "Disabling will prevent lows from being tracked"         
detectBOTip     =   "Show points that price action breaks above all pivots. An arrow from below is displayed"        
detectBDTip     =   "Show points that price action breaks below all pivots. An arrow from above is displayed"         
breakUpTip      =   "Show points that price action breaks above resistance. An arrow from below is displayed"         
breakDnTip      =   "Show points that price action breaks below support. An arrow from above is displayed"          
falseBullTip    =   "Show points that price action initially breaks below support before reversing. False moves can lead to fast moves in the opposite direction (bear trap). A large arrow from below is displayed"          
falseBearTip    =   "Show points that price action initially breaks above resistance before reversing. False moves can lead to fast moves in the opposite direction (bull trap). A large arrow from above is displayed"           
supPushTip      =   "Show up candles that are detected within a support zone. Can show points support is being respected. A triangle from below is displayed"          
resPushTip      =   "Show down candles that are detected within a resistance zone. Can show points resistance is being respected. A triangle from above is displayed"           
curlTip         =   "Show Bjorgum TSI 'curl' when candles are detected in the range of a key zone. Can show momentum shift at Key Levels. (Correlates to Bjorgum TSI indicator)" 

repaintTip      =   "Wait for candles end before detecting patterns. False will show potential patterns forming before they are confirmed."
labelsTip       =   "Show a label for detected candle patterns"
sBoxTip         =   "Show a box around detected candle patterns"
dTip            =   "Detect Doji candle patterns"      
beTip           =   "Detect Engulfing patterns"     
hsTip           =   "Detect Hammers and Shooting Star patterns"     
dgTip           =   "Detect Dragonfly Doji and Gravestone Doji patterns"     
twTip           =   "Detect Tweezer Top and Tweezer Bottom patterns"     
stTip           =   "Detect Spinning Top patterns"     
pcTip           =   "Detect Piercing and Dark Cloud Cover patterns"     
bhTip           =   "Detect Harami candle patterns"     
lsTip           =   "Detect Long Upper Shadow and Long Lower Shadow patterns"     

ecWickTip       =   "Determines if engulfing candles must engulf the wick or just the body of the preceding candle"     
colorMatchTip   =   "Determines if hammers must be up candles and shooting stars must be down candles"     
closeHalfTip    =   "Determines if Tweezer patterns must close beyond the half way point of the preceding candle"     
atrMaxTip       =   "Maximum size of setup candles (as a multiplier of the current ATR)"     
rejectWickTip   =   "The maximum wick size as a percentage of body size allowable for a rejection wick on the resolution candle of the pattern. 0 disables the filter"
hammerFibTip    =   "The relationship of body to candle size for hammers and stars. (ie. body is 33% of total candle size)."     
hsShadowPercTip =   "The maximum allowable opposing wick size as a percent of body size (ex. top wick for a hammer pattern etc.)"     
hammerSizeTip   =   "The minimum size of hammers, stars, or long shadows as a multiplier of ATR. (To filter out tiny setups)"     
dojiSizeTip     =   "The relationship of body to candle size (ie. body is 5% of total candle size)."     
dojiWickSizeTip =   "Maximum wick size comparative to the opposite wick. (eg. 2 = bottom wick must be less than or equal to 2x the top wick)."     
luRatioTip      =   "A relationship of the upper wick to the overall candle size expressed as a percent."     

lookbackTip     =   "Number of candles that can be included in a false break signal"        
swingTip        =   "Swing detection is used to filter signals on breakout type signals. A higher number will mean more significant points, but less of them"        
reflectTip      =   "Filter to ensure a setup is a significant swing point. Look back this far"
offsetTip       =   "Candle pattern high/low distance from absolute swing high/low. Example: 0 would filter patterns that are only the highest/lowest, 1 filters second highest over the significant length, etc."

bullPivotTip    =   "Color of bullish Key Levels\n(border, background)"            
bearPivotTip    =   "Color of bearish Key Levels\n(border, background)"            
breakoutTip     =   "Color of breakout arrows\n(bull, bear,)"           
SnRTip          =   "Color of triangles for broken support or resistance\n(bull, bear)"   
falseBreakTip   =   "Color of arrows for false breaks\n(bull, bear, arrow max height in pixels)"            
moveTip         =   "Color of triangles for candles that are detected within zones\n(bull, bear)"    
patTip          =   "Color of boxes that wrap candestick patterns\nBackgrounds: (bull, neutral, bear)\nBorders: (bull, neutral, bear)"    
labTip          =   "Color of labels that mark candestick patterns\nText: (bull, neutral, bear)\nLabels: (bull, neutral, bear)"    
stratTip        =   "TSI speed control presets. Both speeds correlate to the Bjorgum TSI indicator"

// ================================== //
// ---------> User Input <----------- //
// ================================== //

left            =   input.int       (20     ,   "Look Left"                     ,   group= "Zones"                , tooltip= leftTip            )    
right           =   input.int       (15     ,   "Look Right"                    ,   group= "Zones"                , tooltip= rightTip           )    
nPiv            =   input.int       (4      ,   "Number of Pivots"              ,   group= "Zones"                , tooltip= nPivTip            )
atrLen          =   input.int       (30     ,   "ATR Length"                    ,   group= "Zones"                , tooltip= atrLenTip          )
mult            =   input.float     (0.5    ,   "Zone Width (ATR)"              ,   group= "Zones"                , tooltip= multTip            ,   step   = 0.1)
per             =   input.float     (5      ,   "Max Zone Percent"              ,   group= "Zones"                , tooltip= perTip             )
max             =   input.float     (10     ,   "Max Boxes for Patterns"        ,   group= "Zones"                , tooltip= maxTip             )
fut             =   input.int       (30     ,   "Offset For Labels"             ,   group= "Zones"                , tooltip= futTip             )
src             =   input.string    ("HA"   ,   "Source For Pivots"             ,   group= "Zones"                , tooltip= srcTip             ,   options= ["HA", "High/Low Body", "High/Low"])
alignZones      =   input.bool      (true   ,   "Align Zones"                   ,   group= "Zones"                , tooltip= alignZonesTip      )
extend          =   input.bool      (false  ,   "Extend Right"                  ,   group= "Zones"                , tooltip= extendTip          )
lLab            =   input.bool      (false  ,   "Show Level Labels"             ,   group= "Zones"                , tooltip= lLabTip            )

dhighs          =   input.bool      (true   ,   "Detect Pivot Highs"            ,   group= "Detection"            , tooltip= dhighsTip          )
dlows           =   input.bool      (true   ,   "Detect Pivot Lows"             ,   group= "Detection"            , tooltip= dlowsTip           )
detectBO        =   input.bool      (false  ,   "Detect Breakout"               ,   group= "Detection"            , tooltip= detectBOTip        )
detectBD        =   input.bool      (false  ,   "Detect Breakdown"              ,   group= "Detection"            , tooltip= detectBDTip        )
breakUp         =   input.bool      (false  ,   "Detect Resistance Break"       ,   group= "Detection"            , tooltip= breakUpTip         )
breakDn         =   input.bool      (false  ,   "Detect Support Break"          ,   group= "Detection"            , tooltip= breakDnTip         ) 
falseBull       =   input.bool      (false  ,   "Detect False Breakdown"        ,   group= "Detection"            , tooltip= falseBullTip       )
falseBear       =   input.bool      (false  ,   "Detect False Breakup"          ,   group= "Detection"            , tooltip= falseBearTip       ) 
supPush         =   input.bool      (false  ,   "Detect Moves Off Support"      ,   group= "Detection"            , tooltip= supPushTip         )
resPush         =   input.bool      (false  ,   "Detect Moves Off Resistance"   ,   group= "Detection"            , tooltip= resPushTip         ) 
curl            =   input.bool      (false  ,   "Detect TSI Curl"               ,   group= "Detection"            , tooltip= curlTip            ) 

repaint         =   input.bool      (true   ,   "Wait For Confirmed Bar"        ,   group= "Candle Patterns"      , tooltip= repaintTip         )
labels          =   input.bool      (false  ,   "Show Label"                    ,   group= "Candle Patterns"      , tooltip= labelsTip          )
sBox            =   input.bool      (false  ,   "Show Boxes Around Patterns"    ,   group= "Candle Patterns"      , tooltip= sBoxTip            )
d_              =   input.bool      (false  ,   "Detect Doji"                   ,   group= "Candle Patterns"      , tooltip= dTip               )
be_             =   input.bool      (false  ,   "Detect Engulfing"              ,   group= "Candle Patterns"      , tooltip= beTip              )
hs_             =   input.bool      (false  ,   "Detect Hammers and Stars"      ,   group= "Candle Patterns"      , tooltip= hsTip              )
dg_             =   input.bool      (false  ,   "Detect Dragons and Graves"     ,   group= "Candle Patterns"      , tooltip= dgTip              )
tw_             =   input.bool      (false  ,   "Detect Tweezers"               ,   group= "Candle Patterns"      , tooltip= twTip              )
st_             =   input.bool      (false  ,   "Detect Spinning Top"           ,   group= "Candle Patterns"      , tooltip= stTip              )
pc_             =   input.bool      (false  ,   "Detect Piercing and Clouds"    ,   group= "Candle Patterns"      , tooltip= pcTip              )
bh_             =   input.bool      (false  ,   "Detect Harami"                 ,   group= "Candle Patterns"      , tooltip= bhTip              )
ls_             =   input.bool      (false  ,   "Detect Long Shadows"           ,   group= "Candle Patterns"      , tooltip= lsTip              )

alertMode       =   input.string    (alert.freq_once_per_bar_close              ,   "Alerts Mode"                 , group  = "Alert Frequency"  ,   options= [alert.freq_once_per_bar, alert.freq_once_per_bar_close]) 

ecWick          =   input.bool      (false  ,   "Engulfing Must Engulf Wick"    ,   group= "Candle Filters"       , tooltip= ecWickTip          )
colorMatch      =   input.bool      (false  ,   "H&S Must Match Color"          ,   group= "Candle Filters"       , tooltip= colorMatchTip      )
closeHalf       =   input.bool      (false  ,   "Tweezer Close Over Half"       ,   group= "Candle Filters"       , tooltip= closeHalfTip       )
atrMax          =   input.float     (0.0    ,   "Max Candle Size (× ATR)"       ,   group= "Candle Filters"       , tooltip= atrMaxTip          ,   step= 0.1 )
rejectWickMax   =   input.float     (0.0    ,   "[EC] Max Reject Wick Size"     ,   group= "Candle Filters"       , tooltip= rejectWickTip      ,   step= 1   )  
hammerFib       =   input.float     (33     ,   "[HS] H&S Ratio (%)"            ,   group= "Candle Filters"       , tooltip= hammerFibTip       ,   step= 1   ) 
hsShadowPerc    =   input.float     (5      ,   "[HS] H&S Opposing Shadow (%)"  ,   group= "Candle Filters"       , tooltip= hsShadowPercTip    ,   step= 1   ) 
hammerSize      =   input.float     (0.1    ,   "[HS] H&S Min Size (× ATR)"     ,   group= "Candle Filters"       , tooltip= hammerSizeTip      ,   step= 0.1 ) 
dojiSize        =   input.float     (5      ,   "[DJ] Doji Size (%)"            ,   group= "Candle Filters"       , tooltip= dojiSizeTip        ,   step= 1   )
dojiWickSize    =   input.float     (2      ,   "[DJ] Max Doji Wick Size"       ,   group= "Candle Filters"       , tooltip= dojiWickSizeTip    ,   step= 1   )
luRatio         =   input.float     (75     ,   "[LS] Long Shadow (%)"          ,   group= "Candle Filters"       , tooltip= luRatioTip         ,   step= 1   ) 

lookback        =   input.int       (2      ,   "Lookback For Breaks"           ,   group= "Lookback"             , tooltip= lookbackTip        )
swing           =   input.int       (5      ,   "swing High/Low"                ,   group= "Lookback"             , tooltip= swingTip           )
reflect         =   input.int       (10     ,   "Significant High/Low"          ,   group= "Lookback"             , tooltip= reflectTip         )
offset          =   input.int       (1      ,   "Consider Bar From High/Low"    ,   group= "Lookback"             , tooltip= offsetTip          )

bullBorder      =   input.color     (color.new  (#64b5f6, 60), "", inline= "0"  ,   group= "Pivot Color"                                        )
bullBgCol       =   input.color     (color.new  (#64b5f6, 95), "", inline= "0"  ,   group= "Pivot Color"          , tooltip= bullPivotTip       )
bearBorder      =   input.color     (color.new  (#ffeb3b, 60), "", inline= "1"  ,   group= "Pivot Color"                                        )   
bearBgCol       =   input.color     (color.new  (#ffeb3b, 95), "", inline= "1"  ,   group= "Pivot Color"          , tooltip= bearPivotTip       )

upCol           =   input.color     (color.new  (#ff6d00, 25), "", inline= "2"  ,   group= "Breakout Color"                                     )
dnCol           =   input.color     (color.new  (#ff00ff, 25), "", inline= "2"  ,   group= "Breakout Color"       , tooltip= breakoutTip        ) 

supCol          =   input.color     (color.new  (#17ff00, 25), "", inline= "3"  ,   group= "S&R Break Color"                                    )
resCol          =   input.color     (color.new  (#ff0000, 25), "", inline= "3"  ,   group= "S&R Break Color"      , tooltip= SnRTip             ) 

fBull           =   input.color     (color.new  (#17ff00, 25), "", inline= "4"  ,   group= "False Break Color"                                  )
fBear           =   input.color     (color.new  (#ff0000, 25), "", inline= "4"  ,   group= "False Break Color"                                  )
arrowMax        =   input.int       (75                      , "", inline= "4"  ,   group= "False Break Color"    , tooltip= falseBreakTip      )

moveBullCol     =   input.color     (color.new  (#64b5f6, 25), "", inline= "5"  ,   group= "Moves From S&R Color"                               )
moveBearCol     =   input.color     (color.new  (#ffeb3b, 25), "", inline= "5"  ,   group= "Moves From S&R Color" , tooltip= moveTip            ) 

curlBullCol     =   input.color     (color.new  (#17ff00, 40), "", inline= "6"  ,   group= "Momentum Curl Color"                                )
curlBearCol     =   input.color     (color.new  (#f3ff00, 40), "", inline= "6"  ,   group= "Momentum Curl Color"  , tooltip= curlTip            ) 

patBullBg       =   input.color     (color.new  (#17ff00, 90), "", inline= "7"  ,   group= "Pattern Box Color"                                  )
patNeutBg       =   input.color     (color.new  (#b2b5be, 90), "", inline= "7"  ,   group= "Pattern Box Color"                                  )
patBearBg       =   input.color     (color.new  (#ff0000, 90), "", inline= "7"  ,   group= "Pattern Box Color"                                  )
patBullBo       =   input.color     (color.new  (#17ff00, 80), "", inline= "8"  ,   group= "Pattern Box Color"                                  )
patNeutBo       =   input.color     (color.new  (#b2b5be, 80), "", inline= "8"  ,   group= "Pattern Box Color"                                  )
patBearBo       =   input.color     (color.new  (#ff0000, 80), "", inline= "8"  ,   group= "Pattern Box Color"    , tooltip= patTip             ) 

textBullCol     =   input.color     (color.new  (#17ff00,  0), "", inline= "9"  ,   group= "Label Color (Text/Bg)"                              )
textNeutCol     =   input.color     (color.new  (#b2b5be,  0), "", inline= "9"  ,   group= "Label Color (Text/Bg)"                              )
textBearCol     =   input.color     (color.new  (#ff0000,  0), "", inline= "9"  ,   group= "Label Color (Text/Bg)"                              )
labBullCol      =   input.color     (color.new  (#17ff00, 80), "", inline= "10" ,   group= "Label Color (Text/Bg)"                              )
labNeutCol      =   input.color     (color.new  (#b2b5be, 80), "", inline= "10" ,   group= "Label Color (Text/Bg)"                              )
labBearCol      =   input.color     (color.new  (#ff0000, 80), "", inline= "10" ,   group= "Label Color (Text/Bg)", tooltip= labTip             ) 

strat           =   input.string    ("Fast" ,   "Select a Speed"                ,   group= "TSI Speed Control"    , tooltip= stratTip           ,   options= ["Fast", "Slow"])
    
longf           =   input.int       (25     ,   "Long Length"                   ,   group= "TSI Fast Settings"                                  )
shortf          =   input.int       (5      ,   "Short Length"                  ,   group= "TSI Fast Settings"                                  )
signalf         =   input.int       (14     ,   "Signal Length"                 ,   group= "TSI Fast Settings"                                  )

longs           =   input.int       (25     ,   "Long Length"                   ,   group= "TSI Slow Settings"                                  )
shorts          =   input.int       (13     ,   "Short Length"                  ,   group= "TSI Slow Settings"                                  )
signals         =   input.int       (13     ,   "Signal Length"                 ,   group= "TSI Slow Settings"                                  )

// ================================== //
// -----> Immutable Constants <------ //
// ================================== //    
 
sync            =   bar_index
labUp           =   label.style_label_up
labDn           =   label.style_label_down
confirmed       =   barstate.isconfirmed
extrap          =   extend ?        extend.right  : extend.none

var pivotHigh   =   array.new_box   (nPiv)
var pivotLows   =   array.new_box   (nPiv)  
var highBull    =   array.new_bool  (nPiv)
var lowsBull    =   array.new_bool  (nPiv)
var boxes       =   array.new_box   ()

haSrc           =   src    ==       "HA"    
hiLoSrc         =   src    ==       "High/Low"
tsifast         =   strat  ==       "Fast"
tsislow         =   strat  ==       "Slow"

// ================================== //
// ---> Functional Declarations <---- //
// ================================== //

atr             =   ta.atr          (atrLen)
perMax          =   close*          0.02
min             =   math.min        (perMax, atr*0.3)

_haBody()       =>
    haClose     =   (open + high  +  low  + close)    / 4
    haOpen      =   float(na)
    haOpen      :=  na(haOpen[1]) ? (open + close)    / 2 : 
                   (nz(haOpen[1]) + nz(haClose[1]))   / 2
    
    [haOpen, haClose]
    
_extend(_x) =>
    for i = 0 to               array.size       (_x)-1
        box.set_right          (array.get       (_x, i), sync)
        
_arrayLoad(_x, _max, _val) =>  
    array.unshift                               (_x,   _val)   
    if  array.size                              (_x) > _max
        array.pop                               (_x)

_arrayBox(_x, _max, _val) =>  
    array.unshift                               (_x,   _val)   
    if       array.size                         (_x) > _max
        _b = array.pop                          (_x)
        if extend
            box.set_extend                      (_b, extend.none)

_arrayWrap(_x, _max, _val) =>  
    array.unshift                               (_x,   _val)   
    if  array.size                              (_x) > _max
        box.delete(array.pop                    (_x))

_delLab(_x)     =>
    if array.size(_x) > 0 
        label.delete           (array.pop       (_x))

_delLine(_x)    =>
    if array.size(_x) > 0 
        line.delete            (array.pop       (_x))

_delLevels(_x, _y)  =>
    for i = 0 to array.size                     (_x)-1
        _delLab                                 (_x)
        _delLine                                (_y)

_box(_x1, _t, _r, _b, _boCol, _bgCol, _e) =>
    box.new(                   _x1, _t, _r, _b  , 
     xloc        =             xloc.bar_index   ,
     extend      =             _e               ,
     border_color=             _boCol           ,   
     bgcolor     =             _bgCol           ) 

_wrap(_cond, _x, _bb, _bc, _bgc) =>
    _t           =             ta.highest       (high, _bb) + min
    _b           =             ta.lowest        (low , _bb) - min
    _l           =             bar_index -      _bb
    _r           =             bar_index +      1
    if  _cond
        _arrayWrap            (_x, max, _box    (_l, _t, _r, _b, _bc, _bgc, extend.none)) 

_getBox(_x,_i)   =>
    _box         =             array.get        (_x,_i)
    _t           =             box.get_top      (_box)
    _b           =             box.get_bottom   (_box)
    [_t, _b]
    
_align(_x,_y)    =>
    for i = 0 to               array.size       (_x) -1
        [_T, _B] =             _getBox          (_y, 0)
        [_t, _b] =             _getBox          (_x, i)
        if _T > _b and         _T < _t or 
           _B < _t and         _B > _b or 
           _T > _t and         _B < _b or 
           _B > _b and         _T < _t
            box.set_top        (array.get       (_y, 0), _t)
            box.set_bottom     (array.get       (_y, 0), _b)
 
_color(_x, _y)     =>
    var int _track = nPiv
    for i = 0 to               array.size       (_x) -1
        [t_, b_] =             _getBox          (_x, i)
        _isBull  =             array.get        (_y, i)
        if close > t_ and not  _isBull
            box.set_extend(    array.get        (_x, i), extend.none)
            array.set(_x, i,   _box             (sync  , t_, sync, b_, bullBorder, bullBgCol, extrap))
            array.set(_y, i,   true)
            _track += 1
        if close < b_ and _isBull
            box.set_extend(    array.get        (_x, i), extend.none)
            array.set(_x, i,   _box             (sync  , t_, sync, b_, bearBorder, bearBgCol, extrap))
            array.set(_y, i,   false)
            _track -= 1
    _track

_detect(_x,_y)      =>
    int  _i         = 0
    bool _found     = false
    bool _isBull    = na
    while (not _found and _i < array.size       (_x)  )
        [t_, b_] =             _getBox          (_x,_i)
        if low < t_ and high > b_
            _isBull :=         array.get        (_y,_i)
            _found  :=         true
        _i          +=         1
    [_found, _isBull]

_falseBreak(_l)     =>       
    bool _d         = false
    bool _u         = false
    for i = 1 to lookback
        if _l[i] < _l and _l[i+1] >= _l and _l[1] < _l 
            _d      := true
        if _l[i] > _l and _l[i+1] <= _l and _l[1] > _l 
            _u      := true
    [_d, _u]

_numLevel(_x,_y)    =>
    int _above      = 0
    int _fill       = 0
    for i = 0 to               array.size       (_x)-1
        _isBull     =          array.get        (_x,i)
        if  _isBull
            _above += 1
        if  not na(_isBull)
            _fill  += 1
    for i = 0 to               array.size       (_y)-1
        _isBull     =          array.get        (_y,i)
        if  _isBull
            _above += 1
        if  not na(_isBull)
            _fill  += 1
    [_above, _fill]  

_check(_src,_l)     =>
    bool _check     = false
    for i = 0 to _l
        if _src[i]
            _check := true
    _check

_count(_src, _l)    =>
    int _result     = 0
    for i = 0 to _l
        if _src > _src[i]
            _result += 1
    _result

_label(_x, _y, y, _s, _col1, _col2) =>
    transp = math.min   (color.t(_col1),  color.t(_col2))
    array.unshift       (_x,   label.new (sync+fut,   y                                        , 
                                          text      = str.tostring(math.round_to_mintick(y)   ), 
                                          color     = color.new(_col1, transp)                 , 
                                          style     = _s                                       , 
                                          textcolor = color.white                             ))
    if not extend and fut > 0
        array.unshift   (_y,   line.new  (sync, y, sync+fut, y, color= color.new(_col1, transp)))

_level(_x, _y)          =>
    var label [] lab    =      array.new_label  (nPiv)
    var line  [] lines  =      array.new_line   (nPiv)
    if barstate.islast and lLab
        _delLevels             (lab, lines)
        for i = 0 to           array.size       (_x)-1
            [_t, _b]    =      _getBox          (_x,i)
            _isBull     =      array.get        (_y,i)
            _col1        =     _isBull ?        bullBgCol  : bearBgCol
            _col2        =     _isBull ?        bullBorder : bearBorder
            if close >  _t 
                _label  (lab, lines, _t, labUp, _col1, _col2)
            if close <  _b 
                _label  (lab, lines, _b, labDn, _col1, _col2)
            if close <  _t and close >   _b
                _label  (lab, lines, _t, labDn, _col1, _col2)
                _label  (lab, lines, _b, labUp, _col1, _col2)

_alert(_x, _y) =>
    if _x
        alert   (_y + timeframe.period + ' chart. Price is ' + str.tostring(close), alertMode)
        
// ================================== //
// ----> Variable Calculations <----- //
// ================================== //

shortvar        =   tsifast ?           shortf  :       shorts   
longvar         =   tsifast ?           longf   :       longs    
signalvar       =   tsifast ?           signalf :       signals 

tsi             =   ta.tsi              (close,         shortvar,   longvar)
tsl             =   ta.ema              (tsi,           signalvar)

highest         =   close ==            ta.highest      (close,     right)
lowest          =   close ==            ta.lowest       (close,     right)

closeLows       =   ta.lowest           (close,         swing)
closeHigh       =   ta.highest          (close,         swing)

numLows         =   _count              (low,           reflect)
numHigh         =   _count              (high,          reflect)

[open_, close_] =   _haBody             ()

hiHaBod         =   math.max            (close_,        open_)
loHaBod         =   math.min            (close_,        open_)

hiBod           =   math.max            (close,         open)
loBod           =   math.min            (close,         open)

srcHigh         =   haSrc ?             hiHaBod :       hiLoSrc ?   high :      hiBod
srcLow          =   haSrc ?             loHaBod :       hiLoSrc ?   low  :      loBod

pivot_high      =   ta.pivothigh        (srcHigh,       left,       right)
pivot_low       =   ta.pivotlow         (srcLow,        left,       right)

perc            =   close*              (per/100)

band            =   math.min            (atr*mult,      perc)       [right]     /2

HH              =   pivot_high+         band
HL              =   pivot_high-         band

LH              =   pivot_low+          band
LL              =   pivot_low-          band

coDiff          =   close -             open

// ================================== //
// --------> Logical Order <--------- //
// ================================== //

if pivot_high and   dhighs and  confirmed
    _arrayLoad      (highBull , nPiv,   false)      
    _arrayBox       (pivotHigh, nPiv,   _box(sync[right], HH, sync, HL, bearBorder, bearBgCol, extrap))

if pivot_low  and   dlows and   confirmed
    _arrayLoad      (lowsBull , nPiv,   true)      
    _arrayBox       (pivotLows, nPiv,   _box(sync[right], LH, sync, LL, bullBorder, bullBgCol, extrap))

if alignZones
    _align          (pivotHigh,         pivotHigh)
    _align          (pivotHigh,         pivotLows)    
    _align          (pivotLows,         pivotLows)
    _align          (pivotLows,         pivotHigh)

_extend             (pivotHigh)
_extend             (pivotLows)

trackHigh       =   _color              (pivotHigh,     highBull)
trackLows       =   _color              (pivotLows,     lowsBull)

// ================================== //
// ----> Conditional Parameters <---- //
// ================================== //

isLows          =   closeLows      ==   close
isHigh          =   closeHigh      ==   close

wasLows         =   _check              (isLows,        lookback)
wasHigh         =   _check              (isHigh,        lookback)

[above, total]  =   _numLevel           (highBull,      lowsBull)

moveAbove       =   trackHigh       >   trackHigh[1]
moveBelow       =   trackLows       <   trackLows[1]

resBreak        =   (trackLows      >   trackLows[1]    or  moveAbove) 
supBreak        =   (trackHigh      <   trackHigh[1]    or  moveBelow) 

breakOut        =   moveAbove     and   highest and     above == total             
breakDwn        =   moveBelow     and   lowest  and     above == 0         

[dh, uh]        =   _falseBreak         (trackHigh) 
[dl, ul]        =   _falseBreak         (trackLows) 

falseBreakBull  =   wasLows       and   (dh or dl)
falseBreakBear  =   wasHigh       and   (uh or ul)

[fh,hb]         =   _detect             (pivotHigh,     highBull)
[fl,lb]         =   _detect             (pivotLows,     lowsBull)

bull            =   (fh or fl) and      (hb or lb)
bear            =   (fh or fl) and not  (hb or lb)

bullCheck       =   not resBreak  and   not resBreak[1] and (fh or fl) and  close > open and     (hb or lb)
bearCheck       =   not supBreak  and   not supBreak[1] and (fh or fl) and  close < open and not (hb or lb)

highrange       =   reflect-offset
lowsrange       =   offset

sigLows         =   numLows        <=   lowsrange  
sigHigh         =   numHigh        >=   highrange 

isBull1         =   sigLows       and   bull
isBear1         =   sigHigh       and   bear 

isBull2         =   (sigLows       or   sigLows[1]) and         (bull or bull[1])
isBear2         =   (sigHigh       or   sigHigh[1]) and         (bear or bear[1])

data            =   tsi > tsi[1]  and   tsi < tsl 
dtat            =   tsi < tsi[1]  and   tsi > tsl 

hMatch          =   not colorMatch or   close > open
sMatch          =   not colorMatch or   close < open

hsFilter        =   bj.barRange()  >=   hammerSize * atr
atrMaxSize      =   bj.barRange()  <=   atrMax     * atr or     atrMax == 0.0

rp              =   confirmed  or not   repaint

// ================================== //
// -----> Pattern Recognition <------ //
// ================================== //

dw              =   isBull1 and rp and d_  and atrMaxSize and bj.doji              (dojiSize           = dojiSize,         dojiWickSize    = dojiWickSize)
db              =   isBear1 and rp and d_  and atrMaxSize and bj.doji              (dojiSize           = dojiSize,         dojiWickSize    = dojiWickSize)
bew             =   isBull2 and rp and be_ and atrMaxSize and bj.bullEngulf        (maxRejectWick      = rejectWickMax,    mustEngulfWick  = ecWick) 
beb             =   isBear2 and rp and be_ and atrMaxSize and bj.bearEngulf        (maxRejectWick      = rejectWickMax,    mustEngulfWick  = ecWick)
h               =   isBull1 and rp and hs_ and atrMaxSize and bj.hammer            (ratio              = hammerFib,        shadowPercent   = hsShadowPerc) and hsFilter and hMatch
ss              =   isBear1 and rp and hs_ and atrMaxSize and bj.star              (ratio              = hammerFib,        shadowPercent   = hsShadowPerc) and hsFilter and sMatch
dd              =   isBull1 and rp and dg_ and atrMaxSize and bj.dragonflyDoji     ()
gd              =   isBear1 and rp and dg_ and atrMaxSize and bj.gravestoneDoji    ()
tb              =   isBull2 and rp and tw_ and atrMaxSize and bj.tweezerBottom     (closeUpperHalf     = closeHalf)
tt              =   isBear2 and rp and tw_ and atrMaxSize and bj.tweezerTop        (closeLowerHalf     = closeHalf)
stw             =   isBull1 and rp and st_ and atrMaxSize and bj.spinningTop       ()
stb             =   isBear1 and rp and st_ and atrMaxSize and bj.spinningTop       ()
p               =   isBull1 and rp and pc_ and atrMaxSize and bj.piercing          ()
dcc             =   isBear1 and rp and pc_ and atrMaxSize and bj.darkCloudCover    ()
bhw             =   isBull1 and rp and bh_ and atrMaxSize and bj.haramiBull        ()  
bhb             =   isBear1 and rp and bh_ and atrMaxSize and bj.haramiBear        ()
ll              =   isBull1 and rp and ls_ and atrMaxSize and bj.lls               (ratio              = luRatio)          and hsFilter
lu              =   isBear1 and rp and ls_ and atrMaxSize and bj.lus               (ratio              = luRatio)          and hsFilter

// ================================== //
// ------> Graphical Display <------- //
// ================================== //

plotFalseDn     =   falseBull     and   falseBreakBull
plotFalseUp     =   falseBear     and   falseBreakBear

falseUpCol      =   plotFalseUp     ?   upCol       :   na
falseDnCol      =   plotFalseDn     ?   dnCol       :   na

plotBreakOut    =   breakOut      and   detectBO    and not     plotFalseDn
plotBreakDn     =   breakDwn      and   detectBD    and not     plotFalseUp

plotResBreak    =   resBreak      and   breakUp     and not     (plotBreakOut or plotFalseDn)
plotSupBreak    =   supBreak      and   breakDn     and not     (plotBreakDn  or plotFalseUp)

plotBullCheck   =   bullCheck     and   supPush
plotBearCheck   =   bearCheck     and   resPush

plotCurlBull    =   curl and data and   bull
plotCurlBear    =   curl and dtat and   bear

plotarrow           (plotFalseUp    ?   coDiff      :   na      ,   colorup  = fBull          ,     colordown=      fBear ,         maxheight=      arrowMax)
plotarrow           (plotFalseDn    ?   coDiff      :   na      ,   colorup  = fBull          ,     colordown=      fBear ,         maxheight=      arrowMax)

plotshape           (plotBreakOut   ,   style=shape.arrowup     ,   location=location.belowbar,     color=          upCol ,         size=           size.small)
plotshape           (plotBreakDn    ,   style=shape.arrowdown   ,   location=location.abovebar,     color=          dnCol ,         size=           size.small)

plotshape           (plotResBreak   ,   style=shape.arrowup     ,   location=location.belowbar,     color=          supCol,         size=           size.small)
plotshape           (plotSupBreak   ,   style=shape.arrowdown   ,   location=location.abovebar,     color=          resCol,         size=           size.small)

plotshape           (plotBullCheck  ,   style=shape.triangleup  ,   location=location.belowbar,     color=          moveBullCol)
plotshape           (plotBearCheck  ,   style=shape.triangledown,   location=location.abovebar,     color=          moveBearCol)

plotshape           (plotCurlBull   ,   style=shape.triangleup  ,   location=location.belowbar,     color=          curlBullCol)
plotshape           (plotCurlBear   ,   style=shape.triangledown,   location=location.abovebar,     color=          curlBearCol)

bj.dLab             (dw  and labels, labNeutCol, textNeutCol), _wrap (dw  and sBox, boxes, 1, patNeutBo, patNeutBg)
bj.bewLab           (bew and labels, labBullCol, textBullCol), _wrap (bew and sBox, boxes, 2, patBullBo, patBullBg)
bj.hLab             (h   and labels, labBullCol, textBullCol), _wrap (h   and sBox, boxes, 1, patBullBo, patBullBg)
bj.ddLab            (dd  and labels, labBullCol, textBullCol), _wrap (dd  and sBox, boxes, 1, patBullBo, patBullBg)
bj.tbLab            (tb  and labels, labBullCol, textBullCol), _wrap (tb  and sBox, boxes, 2, patBullBo, patBullBg)
bj.stwLab           (stw and labels, labNeutCol, textNeutCol), _wrap (stw and sBox, boxes, 1, patBullBo, patNeutBg)
bj.pLab             (p   and labels, labBullCol, textBullCol), _wrap (p   and sBox, boxes, 2, patBullBo, patBullBg)
bj.hwLab            (bhw and labels, labBullCol, textBullCol), _wrap (bhw and sBox, boxes, 2, patBullBo, patBullBg)
bj.llsLab           (ll  and labels, labBullCol, textBullCol), _wrap (ll  and sBox, boxes, 1, patBullBo, patBullBg)

bj.dLab             (db  and labels, labNeutCol, textNeutCol), _wrap (db  and sBox, boxes, 1, patNeutBo, patNeutBg)
bj.bebLab           (beb and labels, labBearCol, textBearCol), _wrap (beb and sBox, boxes, 2, patBearBo, patBearBg)
bj.ssLab            (ss  and labels, labBearCol, textBearCol), _wrap (ss  and sBox, boxes, 1, patBearBo, patBearBg)
bj.gdLab            (gd  and labels, labBearCol, textBearCol), _wrap (gd  and sBox, boxes, 1, patBearBo, patBearBg)
bj.ttLab            (tt  and labels, labBearCol, textBearCol), _wrap (tt  and sBox, boxes, 2, patBearBo, patBearBg)
bj.stbLab           (stb and labels, labNeutCol, textNeutCol), _wrap (stb and sBox, boxes, 1, patBearBo, patBearBg)
bj.dccLab           (dcc and labels, labBearCol, textBearCol), _wrap (dcc and sBox, boxes, 2, patBearBo, patBearBg)
bj.hbLab            (bhb and labels, labBearCol, textBearCol), _wrap (bhb and sBox, boxes, 2, patBearBo, patBearBg)
bj.lusLab           (lu  and labels, labBearCol, textBearCol), _wrap (lu  and sBox, boxes, 1, patBearBo, patBearBg)

_level              (pivotHigh, highBull)
_level              (pivotLows, lowsBull)

// ================================== //
// -----> Alert Functionality <------ //
// ================================== //

alertcondition      (resBreak       ,   'Resistance break'                      ,   'Resistance broke on {{interval}} chart. Price is {{close}}'                    )
alertcondition      (supBreak       ,   'Support break'                         ,   'Support broke on {{interval}} chart. Price is {{close}}'                       )
alertcondition      (bullCheck      ,   'Found support'                         ,   'Pushing Off Key Level Support on {{interval}} chart. Price is {{close}}'       )
alertcondition      (bearCheck      ,   'Found resistance'                      ,   'Pushing Off Key Level Resistance on {{interval}} chart. Price is {{close}}'    )
alertcondition      (falseBreakBull ,   'False break down'                      ,   'False Break Down on {{interval}} chart. Price is {{close}}'                    )
alertcondition      (falseBreakBear ,   'False break up'                        ,   'False Break Up on {{interval}} chart. Price is {{close}}'                      )
alertcondition      (breakOut       ,   'Breakout'                              ,   'Breakout on {{interval}} chart. Price is {{close}}'                            )
alertcondition      (breakDwn       ,   'Breakdown'                             ,   'Breakdown on {{interval}} chart. Price is {{close}}'                           )

_alert              (plotResBreak   ,   'Resistance broke on '                  )
_alert              (plotSupBreak   ,   'Support break '                        )
_alert              (plotBullCheck  ,   'Pushing off key level support on '     )
_alert              (plotBearCheck  ,   'Pushing off key level resistance on '  )
_alert              (plotFalseDn    ,   'False break down on '                  )
_alert              (plotFalseUp    ,   'False break up on '                    )
_alert              (plotBreakOut   ,   'Breakout on '                          )
_alert              (plotBreakDn    ,   'Breakdown on '                         )

_alert              (dw             ,   'Doji at support on '                   )
_alert              (db             ,   'Doji at resistance on '                )
_alert              (bew            ,   'Bullish Engulfing on '                 )
_alert              (beb            ,   'Bearish Engulfing on '                 )
_alert              (h              ,   'Hammer candle on '                     )
_alert              (ss             ,   'Shooting star on '                     )
_alert              (dd             ,   'Dragonfly Doji on '                    )
_alert              (gd             ,   'Gravestone Doji on '                   )
_alert              (tb             ,   'Tweezer Bottom on '                    )
_alert              (tt             ,   'Tweezer Top on '                       )
_alert              (stw            ,   'White Spinning Top on '                )
_alert              (stb            ,   'Black Spinning Top on '                )
_alert              (p              ,   'Piercing on '                          )
_alert              (dcc            ,   'Dark Cloud Cover on '                  )
_alert              (bhw            ,   'Bullish Harami on '                    )
_alert              (bhb            ,   'Bearish Harami on '                    )
_alert              (ll             ,   'Long Lower Shadow on '                 )
_alert              (lu             ,   'Long Upper Shadow on '                 )

//  ____  __ _  ____ 
// (  __)(  ( \(    \
//  ) _) /    / ) D (
// (____)\_)__)(____/

// ██████╗░░░░░░██╗░█████╗░██████╗░░██████╗░██╗░░░██╗███╗░░░███╗
// ██╔══██╗░░░░░██║██╔══██╗██╔══██╗██╔════╝░██║░░░██║████╗░████║
// ██████╦╝░░░░░██║██║░░██║██████╔╝██║░░██╗░██║░░░██║██╔████╔██║
// ██╔══██╗██╗░░██║██║░░██║██╔══██╗██║░░╚██╗██║░░░██║██║╚██╔╝██║
// ██████╦╝╚█████╔╝╚█████╔╝██║░░██║╚██████╔╝╚██████╔╝██║░╚═╝░██║
// ╚═════╝░░╚════╝░░╚════╝░╚═╝░░╚═╝░╚═════╝░░╚═════╝░╚═╝░░░░░╚═╝

//@version=5
indicator       ("Bjorgum SuperScript", "Bj SuperScript", true, precision = 3)

// ================================== //
// ---------> User Input <----------- //
// ================================== //

strat           =  input.string ("Bj Reversal", "Select a Strategy"             ,   group= "Strategy Selector"  ,   options= ["Bj Reversal", "MA", "TRM", "RSI Color"])
source          =  input.source (close        , "MA source"                     ,   group= "Strategy Selector")
alphaInput      =  input.float  (0.7          , "T3 Alpha"                      ,   group= "Strategy Selector")

t3vis           =  input.bool   (false        , "Show Tilson MA's"              ,   group= "Strategy Overides"  )
hemavis         =  input.bool   (false        , "Show HEMA MA's"                ,   group= "Strategy Overides"  )  
psarvis         =  input.bool   (false        , "Show PSAR"                     ,   group= "Strategy Overides"  )  
arrowvis        =  input.bool   (false        , "Show Arrows"                   ,   group= "Strategy Overides"  ) 
showTable       =  input.bool   (false        , "Show Bjorgum Buy Rating"       ,   group= "Strategy Overides"  ) 
disable         =  input.bool   (false        , "Disable Current MA"            ,   group= "Strategy Overides"  )

tsistrat        =  input.string ("Fast"       , "TRM Speed Control"             ,   group= "Select a Speed"     ,   options= ["Fast", "Slow"])
rsistrat        =  input.string ("Slow"       , "RSI Bar Color Speed Control"   ,   group= "Select a Speed"     ,   options= ["Slow", "Fast"])

type            =  input.string ("Hollow"     , "Candle Type"                   ,   group= "Heikin-Ashi Overlay",   options= ["Hollow", "Bars", "Candles"])
haover          =  input.bool   (false        , "Overlays HA Plotcandle"        ,   group= "Heikin-Ashi Overlay",   tooltip= "(Must 'mute' main candle series while in use. Toggle '👁' symbol next to ticker name to hide candles)")
rPrice          =  input.bool   (false        , "Actual Close Value"            ,   group= "Heikin-Ashi Overlay",   tooltip= "Displays real close value - useful when using HA overlay")

i_alert_mode    =  input.string (alert.freq_once_per_bar_close, "Alerts Mode"   ,   group= "Variable Alerts"    ,   options= [alert.freq_once_per_bar, alert.freq_once_per_bar_close], tooltip="Using the alert() function call will allow use of only one alert for any of the following selected. Any current settings in the script will be what is saved at the time the alert is set")

BUY4            =  input.bool   (false        , "TRM Buy"                       ,   group= "Variable Alerts"    )
SELL4           =  input.bool   (false        , "TRM Sell"                      ,   group= "Variable Alerts"    )
REVUP           =  input.bool   (false        , "Reversal Up"                   ,   group= "Variable Alerts"    )
REVDWN          =  input.bool   (false        , "Reversal Down"                 ,   group= "Variable Alerts"    )
HCROSSUP        =  input.bool   (false        , "MA Cross Up"                   ,   group= "Variable Alerts"    )
HCROSSDWN       =  input.bool   (false        , "MA Cross Down"                 ,   group= "Variable Alerts"    ) 
RSIUP           =  input.bool   (false        , "RSI Cross Up 50"               ,   group= "Variable Alerts"    )
RSIDWN          =  input.bool   (false        , "RSI Cross Down 50"             ,   group= "Variable Alerts"    )
RSIOB           =  input.bool   (false        , "RSI Cross Up 70"               ,   group= "Variable Alerts"    )
RSIOS           =  input.bool   (false        , "RSI Cross Down 30"             ,   group= "Variable Alerts"    )
RSICD           =  input.bool   (false        , "RSI Cross Down 70"             ,   group= "Variable Alerts"    )
RSICU           =  input.bool   (false        , "RSI Cross Up 30"               ,   group= "Variable Alerts"    )
DATA            =  input.bool   (false        , "TSI Curl Up"                   ,   group= "Variable Alerts"    )
DTAT            =  input.bool   (false        , "TSI Curl Down"                 ,   group= "Variable Alerts"    )
SARUP           =  input.bool   (false        , "Sar Up"                        ,   group= "Variable Alerts"    )
SARDWN          =  input.bool   (false        , "Sar Down"                      ,   group= "Variable Alerts"    )

textSize        =  input.string ("small"      , "Text Size"                     ,   group= "Table"              ,   options= ["tiny", "small", "normal", "large", "huge", "auto"])
tableYpos       =  input.string ("bottom"     , "Position"                      ,   group= "Table"              ,   options= ["top", "middle", "bottom"])
tableXpos       =  input.string ("right"      , ""                              ,   group= "Table"              ,   options= ["left", "center", "right"])

bullRev5        =  input.color  (color.new    (#64b5f6,  0), "", inline= "1"    ,   group= "T3 MA (Bull, Bear, Fill, Length)"    )
bearRev5        =  input.color  (color.new    (#ef5350,  0), "", inline= "1"    ,   group= "T3 MA (Bull, Bear, Fill, Length)"    )

bullRev8        =  input.color  (color.new    (#64b5f6,  0), "", inline= "2"    ,   group= "T3 MA (Bull, Bear, Fill, Length)"    )
bearRev8        =  input.color  (color.new    (#ef5350,  0), "", inline= "2"    ,   group= "T3 MA (Bull, Bear, Fill, Length)"    )

bullRevF        =  input.color  (color.new    (#64b5f6, 80), "", inline= "1"    ,   group= "T3 MA (Bull, Bear, Fill, Length)"    )
bearRevF        =  input.color  (color.new    (#ef5350, 80), "", inline= "2"    ,   group= "T3 MA (Bull, Bear, Fill, Length)"    ) 

bullHema1       =  input.color  (color.new    (#64b5f6, 55), "", inline= "4"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )
bearHema1       =  input.color  (color.new    (#d32f2f, 55), "", inline= "4"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )

bullHema2       =  input.color  (color.new    (#64b5f6, 45), "", inline= "5"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )
bearHema2       =  input.color  (color.new    (#d32f2f, 45), "", inline= "5"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )

bullHema3       =  input.color  (color.new    (#64b5f6, 35), "", inline= "6"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )
bearHema3       =  input.color  (color.new    (#d32f2f, 35), "", inline= "6"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )

bullHema1F      =  input.color  (color.new    (#64b5f6, 85), "", inline= "4"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )
bearHema1F      =  input.color  (color.new    (#d32f2f, 85), "", inline= "4"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )

bullHema2F      =  input.color  (color.new    (#64b5f6, 80), "", inline= "5"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )
bearHema2F      =  input.color  (color.new    (#d32f2f, 80), "", inline= "5"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )

bullHema3F      =  input.color  (color.new    (#64b5f6, 75), "", inline= "6"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )
bearHema3F      =  input.color  (color.new    (#d32f2f, 75), "", inline= "6"    ,   group= "MA (Bull, Bear, Fill, Length, Type)" )

bullSar         =  input.color  (color.new    (#64b5f6,  0), "", inline= "7"    ,   group= "SAR (Bull, Bear)"                    )
bearSar         =  input.color  (color.new    (#ef5350,  0), "", inline= "7"    ,   group= "SAR (Bull, Bear)"                    )

aLength         =  input.int    (5            , ""  ,   inline="1"              ,   group= "T3 MA (Bull, Bear, Fill, Length)"    )
Length          =  input.int    (8            , ""  ,   inline="2"              ,   group= "T3 MA (Bull, Bear, Fill, Length)"    )

hemaslow        =  input.int    (5            , ""  ,   inline="4"              ,   group= "MA (Bull, Bear, Fill, Length, Type)" )
hemaslow2       =  input.int    (9            , ""  ,   inline="5"              ,   group= "MA (Bull, Bear, Fill, Length, Type)" )
hemaslow3       =  input.int    (21           , ""  ,   inline="6"              ,   group= "MA (Bull, Bear, Fill, Length, Type)" )

maType1         =  input.string ("HEMA"       , ""  ,   inline= "4"             ,   group= "MA (Bull, Bear, Fill, Length, Type)",     options= ["SMA", "EMA", "HEMA", "T3", "DEMA", "TEMA", "HMA", "WMA", "VWMA", "SWMA", "VWAP"])
maType2         =  input.string ("HEMA"       , ""  ,   inline= "5"             ,   group= "MA (Bull, Bear, Fill, Length, Type)",     options= ["SMA", "EMA", "HEMA", "T3", "DEMA", "TEMA", "HMA", "WMA", "VWMA", "SWMA", "VWAP"])
maType3         =  input.string ("HEMA"       , ""  ,   inline= "6"             ,   group= "MA (Bull, Bear, Fill, Length, Type)",     options= ["SMA", "EMA", "HEMA", "T3", "DEMA", "TEMA", "HMA", "WMA", "VWMA", "SWMA", "VWAP"])

bullcol         =  input.color  (#64b5f6      , "Bull Trend Color"              ,   group= "Bar Color"              )
bearcol         =  input.color  (#ef5350      , "Bear Trend Color"              ,   group= "Bar Color"              )
bullrev         =  input.color  (#fff176      , "Bullish Reversal Color"        ,   group= "Bar Color"              )
bearrev         =  input.color  (#fff176      , "Bearish Reversal Color"        ,   group= "Bar Color"              )

hemabull        =  input.color  (#64b5f6      , "MA Bull Color"                 ,   group= "Bar Color"              )
hemabear        =  input.color  (#e91e63      , "MA Bear Color"                 ,   group= "Bar Color"              )
hemahold        =  input.color  (#b2b5be      , "MA Hold Color"                 ,   group= "Bar Color"              )

trmbull         =  input.color  (#3BB3E4      , "TRM Buy"                       ,   group= "Bar Color"              )
trmbear         =  input.color  (#ffeb3b      , "TRM Sell"                      ,   group= "Bar Color"              )
trmhold         =  input.color  (#b2b5be      , "TRM Hold"                      ,   group= "Bar Color"              )

rsi5            =  input.color  (#17ff00      , "RSI 0-5"                       ,   group= "Bar Color"              )
rsi10           =  input.color  (#17ff00      , "RSI 5-10"                      ,   group= "Bar Color"              )
rsi15           =  input.color  (#ffffff      , "RSI 10-15"                     ,   group= "Bar Color"              )
rsi20           =  input.color  (#ffffff      , "RSI 15-20"                     ,   group= "Bar Color"              )
rsi25           =  input.color  (#ffffff      , "RSI 20-25"                     ,   group= "Bar Color"              )
rsi30           =  input.color  (#ffffff      , "RSI 25-30"                     ,   group= "Bar Color"              )
rsi35           =  input.color  (#ffffff      , "RSI 30-35"                     ,   group= "Bar Color"              )
rsi40           =  input.color  (#ffffff      , "RSI 35-40"                     ,   group= "Bar Color"              )
rsi45           =  input.color  (#ffffff      , "RSI 40-45"                     ,   group= "Bar Color"              )
rsi50           =  input.color  (#ffffff      , "RSI 45-50"                     ,   group= "Bar Color"              )
rsi55           =  input.color  (#2196f3      , "RSI 50-55"                     ,   group= "Bar Color"              )
rsi60           =  input.color  (#2196f3      , "RSI 55-60"                     ,   group= "Bar Color"              )
rsi65           =  input.color  (#2196f3      , "RSI 60-65"                     ,   group= "Bar Color"              )
rsi70           =  input.color  (#2196f3      , "RSI 65-70"                     ,   group= "Bar Color"              )
rsi75           =  input.color  (#2196f3      , "RSI 70-75"                     ,   group= "Bar Color"              )
rsi80           =  input.color  (#2196f3      , "RSI 75-80"                     ,   group= "Bar Color"              )
rsi85           =  input.color  (#9c27b0      , "RSI 80-85"                     ,   group= "Bar Color"              )
rsi90           =  input.color  (#9c27b0      , "RSI 85-90"                     ,   group= "Bar Color"              )
rsi95           =  input.color  (#ff0000      , "RSI 90-95"                     ,   group= "Bar Color"              )
rsi100          =  input.color  (#ff0000      , "RSI 95-100"                    ,   group= "Bar Color"              )

longf           =  input.int    (25           , "Long Length"                   ,   group= "TSI Fast Settings"      )
shortf          =  input.int    (5            , "Short Length"                  ,   group= "TSI Fast Settings"      )
signalf         =  input.int    (14           , "Signal Length"                 ,   group= "TSI Fast Settings"      )
lenf            =  input.int    (5            , "RSI Length"                    ,   group= "TSI Fast Settings"      )

longs           =  input.int    (25           , "Long Length"                   ,   group= "TSI Slow Settings"      )
shorts          =  input.int    (13           , "Short Length"                  ,   group= "TSI Slow Settings"      )
signals         =  input.int    (13           , "Signal Length"                 ,   group= "TSI Slow Settings"      )
lens            =  input.int    (14           , "RSI Length"                    ,   group= "TSI Slow Settings"      )

start           =  input.float  (.043         , "Start"                         ,   group="PSAR"                    )   
inc             =  input.float  (.043         , "Increment"                     ,   group="PSAR"                    )
max             =  input.float  (.34          , "Max"                           ,   group="PSAR"                    )

// ================================== //
// -----> Immutable Constants <------ //
// ================================== //

cell1           =  color.new    (color.silver , 80)

bjrev           =  strat        == "Bj Reversal"
hema            =  strat        == "MA"
trm             =  strat        == "TRM" 
rsicol          =  strat        == "RSI Color"

tsifast         =  tsistrat     == "Fast"
tsislow         =  tsistrat     == "Slow" 

rsifast         =  rsistrat     == "Fast"
rsislow         =  rsistrat     == "Slow"

hollow          =  type         == "Hollow" 
bars            =  type         == "Bars"
candle          =  type         == "Candles"

shortvar        =  tsifast ?    shortf  : shorts   
longvar         =  tsifast ?    longf   : longs    
signalvar       =  tsifast ?    signalf : signals  

len             =  tsifast ?    lenf    :
                   tsislow ?    lens    :
                   rsifast ?    lenf    :
                   rsislow ?    lens    : lens

// ================================== //
// ---> Functional Declarations <---- //
// ================================== //

_gd(src, length, alpha) =>
    float result = ta.ema(src, length) * (1 + alpha) - ta.ema(ta.ema(src, length), length) * alpha

_t3(src, length, alpha = 0.7) =>
    float result = _gd(_gd(_gd(src, length, alpha), length, alpha), length, alpha)

_dema(src, length) =>
    float ema1   = ta.ema(src,  length)
    float ema2   = ta.ema(ema1, length)
    float result = 2 * ema1 - ema2

_tema(src, length) =>
    float ema1   = ta.ema(src,  length)
    float ema2   = ta.ema(ema1, length)
    float ema3   = ta.ema(ema2, length)
    float result = 3 * (ema1 - ema2) + ema3

_haVal()         =>
    float _c     =  (open + high + low + close) / 4
    float _o     =  float(na)
    _o          :=  na(_o[1])  ? (open + close) / 2 : (nz(_o[1]) + nz(_c[1])) / 2
    float _h     =  math.max     (high , math.max     (_o,  _c))
    float _l     =  math.min     (low  , math.min     (_o,  _c))
    [_o, _h, _l, _c]

[o_,h_,l_,c_]   =  _haVal       ()

_ma(src, type, length) =>
    float result = switch type
        "EMA"  => ta.ema (src, length)
        "HEMA" => ta.ema (o_,  length)
        "SMA"  => ta.sma (src, length)
        "HMA"  => ta.hma (src, length)
        "WMA"  => ta.wma (src, length)
        "VWMA" => ta.vwma(src, length)
        "DEMA" => _dema  (src, length)
        "TEMA" => _tema  (src, length)
        "T3"   => _t3    (src, length, alphaInput)
        "SWMA" => ta.swma(src)
        "VWAP" => ta.vwap(src)

_alert(cond, txt)  =>
    if cond
        alert      (txt + timeframe.period + ' chart. Price is ' + str.tostring(close), i_alert_mode)

_triGrad(src, min, max, midCol, bearCol, bullCol) =>
    float center = min + (max - min) / 2
    color result = src >= center ? 
      color.from_gradient(src, center, max, midCol, bullCol) : 
      color.from_gradient(src, min, center, bearCol, midCol)

_populate(id, names, syms, col1, col2) =>
    if showTable 
        for [i, a] in names
            s = array.get(syms, i)
            table.cell(table_id= id, column= 0, row= i, bgcolor= cell1,     text= a, text_color= col1, text_size= textSize)
            table.cell(table_id= id, column= 1, row= i, bgcolor= color(na), text= s, text_color= col2, text_size= textSize)

// ================================== //
// ----> Variable Calculations <----- //
// ================================== //

tsi             =  ta.tsi           (close ,    shortvar,   longvar   )
tsl             =  ta.ema           (tsi   ,    signalvar             )
rsi             =  ta.rsi           (close ,    len                   )

anT3Average     =  _t3              (source,    aLength,    alphaInput)
nT3Average      =  _t3              (source,    Length,     alphaInput)

bjhemaslow      =  _ma              (source,    maType1,    hemaslow  )
bjhemaslow2     =  _ma              (source,    maType2,    hemaslow2 )
bjhemaslow3     =  _ma              (source,    maType3,    hemaslow3 )
bjhemafast      =  ta.ema           (hl2   ,    1                     )

sar             =  ta.sar           (start ,    inc,        max       )

// ================================== //
// ----> Conditional Parameters <---- //
// ================================== //

hadu            =  c_ >= o_

buy1            =  ta.crossover     (tsi,tsl) and   rsi > 50
buy2            =  ta.crossover     (rsi,50)  and   tsi > tsl
buy3            =  ta.crossover     (tsi,tsl) and   ta.crossover    (rsi,50)

sell1           =  ta.crossunder    (tsi,tsl) and   rsi < 50
sell2           =  ta.crossunder    (rsi,50)  and   tsi < tsl
sell3           =  ta.crossunder    (tsi,tsl) and   ta.crossunder   (rsi,50)

rsicross        =  ta.cross         (rsi,           50) 
rsiup           =  ta.crossover     (rsi,           50) 
rsidwn          =  ta.crossunder    (rsi,           50) 
rsiob           =  ta.crossover     (rsi,           70) 
rsios           =  ta.crossunder    (rsi,           30) 
rsicd           =  ta.crossunder    (rsi,           70) 
rsicu           =  ta.crossover     (rsi,           30) 

revbar          =  ta.cross         (close,         nT3Average)
trendbar        =  ta.cross         (nT3Average,    anT3Average)
revup           =  ta.crossover     (close,         nT3Average)
revdwn          =  ta.crossunder    (close,         nT3Average)

emasig          =  ta.cross         (close,         bjhemaslow)     and     ta.cross     (close,    bjhemaslow2)

hemauc          =                   (close>         bjhemaslow)     and     (close >                bjhemaslow2)
hemadc          =                   (close<         bjhemaslow)     and     (close <                bjhemaslow2)

crossup1        =  ta.crossover     (close,         bjhemaslow)     and     (close >                bjhemaslow2)
crossup2        =  ta.crossover     (close,         bjhemaslow2)    and     (close >                bjhemaslow)
crossup3        =  ta.crossover     (close,         bjhemaslow2)    and     ta.crossover (close,    bjhemaslow)

crossdown1      =  ta.crossunder    (close,         bjhemaslow)     and     (close <                bjhemaslow2)
crossdown2      =  ta.crossunder    (close,         bjhemaslow2)    and     (close <                bjhemaslow)
crossdown3      =  ta.crossunder    (close,         bjhemaslow2)    and     ta.crossunder(close,    bjhemaslow)

sarup           =  ta.crossover     (close,         sar)
sardwn          =  ta.crossunder    (close,         sar)

Hcrossup        =  crossup1   or    crossup2   or   crossup3
Hcrossdwn       =  crossdown1 or    crossdown2 or   crossdown3

buy4            =  buy1  or buy2    or buy3
sell4           =  sell1 or sell2   or sell3

buy             =  tsi > tsl  and   rsi > 50
sell            =  tsi < tsl  and   rsi < 50 

data            =  tsi < tsl  and   tsi > tsi   [1]  
dtat            =  tsi > tsl  and   tsi < tsi   [1]  

colOne          =  anT3Average  >   anT3Average [1] 
upCol           =  nT3Average   >   nT3Average  [1] 
fillData        =  anT3Average  >   nT3Average   

uc              =  (anT3Average >=  nT3Average) and (close > nT3Average) 
dc              =  (anT3Average <=  nT3Average) and (close < nT3Average)   
dr              =  (anT3Average >=  nT3Average) and (close < nT3Average)  
ur              =  (anT3Average <=  nT3Average) and (close > nT3Average)  
  
hauc            =  (anT3Average >=  nT3Average) and (c_    > nT3Average) 
hadc            =  (anT3Average <=  nT3Average) and (c_    < nT3Average) 
hadr            =  (anT3Average >=  nT3Average) and (c_    < nT3Average) 
haur            =  (anT3Average <=  nT3Average) and (c_    > nT3Average) 

up              =  bjhemaslow   >   bjhemaslow  [1]
up2             =  bjhemaslow2  >   bjhemaslow2 [1]
up3             =  bjhemaslow3  >   bjhemaslow3 [1]

hfillData       =  bjhemaslow   <   bjhemafast
hfillDtat       =  bjhemaslow2  <   bjhemaslow
hfillDat        =  bjhemaslow3  <   bjhemaslow2

ema1            =  bjhemaslow   <   close
ema2            =  bjhemaslow2  <   close
ema3            =  bjhemaslow3  <   close

sarUp           =  close        >=  sar
rsiUp           =  rsi          >   50
tsiUp           =  tsi          >   tsl
rsiH            =  rsi          >=  90 or      rsi <= 10

// ================================== //
// ------> Statistical Rating <------ //
// ================================== //

names           =  array.from(
                   "TRM"                                    ,
                   "Rev"                                    ,
                   "Curl"                                   ,
                   "RSI"                                    ,
                   "TSI"                                    ,
                   str.format("{0} {1}", maType1, hemaslow) ,
                   str.format("{0} {1}", maType2, hemaslow2),
                   str.format("{0} {1}", maType3, hemaslow3),
                   "SAR"                                    )

syms            =  array.from(
                   buy   ? "✅" : sell ? "❌" : "➖" ,
                   uc    ? "✅" : dc   ? "❌" : "🔸" , 
                   data  ? "✅" : dtat ? "❌" : "➖" ,
                   rsiH  ? "🔥": rsiUp ? "✅" : "❌" ,                           
                   tsiUp ? "✅" :        "❌"        ,
                   ema1  ? "✅" :        "❌"        ,
                   ema2  ? "✅" :        "❌"        ,
                   ema3  ? "✅" :        "❌"        ,
                   sarUp ? "✅" :        "❌"        ) 

sigs            =  array.from(
                   buy   ?  1   : sell ?  -1   : 0    ,
                   uc    ?  1   : dc   ?  -1   : 0    ,
                   data  ?  1   : dtat ?  -1   : 0    ,
                   rsiUp ?  1   :         -1          ,
                   tsiUp ?  1   :         -1          ,
                   ema1  ?  1   :         -1          ,
                   ema2  ?  1   :         -1          ,
                   ema3  ?  1   :         -1          ,
                   sarUp ?  1   :         -1          ) 

sigSum          = array.sum (sigs)

tabCol          = _triGrad  (sigSum, -6, 6, color.gray, #ff0000, color.green)

var stats       = table.new (position=tableYpos + "_" + tableXpos, columns=2, rows=9, border_color= color.new(color.gray, 60), border_width=1)

_populate         (stats, names, syms, tabCol, tabCol)

// ================================== //
// ------> Graphical Display <------- //
// ================================== //

color2          =  colOne    ?  bullRev5      : bearRev5 
myColor         =  upCol     ?  bullRev8      : bearRev8 

revFillCol      =  fillData  ?  bullRevF      : bearRevF 
hfillCol1       =  hfillData ?  bullHema1F    : bearHema1F 
hfillCol2       =  hfillDtat ?  bullHema2F    : bearHema2F
hfillCol3       =  hfillDat  ?  bullHema3F    : bearHema3F 

mycolor         =  up        ?  bullHema1     : bearHema1
mycolor2        =  up2       ?  bullHema2     : bearHema2
mycolor3        =  up3       ?  bullHema3     : bearHema3

colUp           =  sarUp     ?  bullSar       : bearSar
hadefval        =  hadu      ?  bullcol       : bearcol

trmcolor        =  buy       ?  trmbull       : sell        ?       trmbear     :   trmhold
hemabar         =  hemauc    ?  hemabull      : hemadc      ?       hemabear    :   hemahold 

rsicon          =  rsi > 0  and rsi <= 5  ? rsi5  :
                   rsi > 5  and rsi <= 10 ? rsi10 :
                   rsi > 10 and rsi <= 15 ? rsi15 :
                   rsi > 15 and rsi <= 20 ? rsi20 :
                   rsi > 20 and rsi <= 25 ? rsi25 :
                   rsi > 25 and rsi <= 30 ? rsi30 :
                   rsi > 30 and rsi <= 35 ? rsi35 :
                   rsi > 35 and rsi <= 40 ? rsi40 :
                   rsi > 40 and rsi <= 45 ? rsi45 :
                   rsi > 45 and rsi <= 50 ? rsi50 :
                   rsi > 50 and rsi <= 55 ? rsi55 :
                   rsi > 55 and rsi <= 60 ? rsi60 :
                   rsi > 60 and rsi <= 65 ? rsi65 :
                   rsi > 65 and rsi <= 70 ? rsi70 :
                   rsi > 70 and rsi <= 75 ? rsi75 :
                   rsi > 75 and rsi <= 80 ? rsi80 :
                   rsi > 80 and rsi <= 85 ? rsi85 :
                   rsi > 85 and rsi <= 90 ? rsi90 :
                   rsi > 90 and rsi <= 95 ? rsi95 :
                   rsi > 95 and rsi <= 100? rsi100: hadefval

bjrevcol        =  uc        ?  bullcol       : 
                   dc        ?  bearcol       :
                   dr        ?  bearrev       :
                   ur        ?  bullrev       : na
 
habjrevcol      =  hauc      ?  bullcol       :
                   hadc      ?  bearcol       :
                   hadr      ?  bearrev       :
                   haur      ?  bullrev       : hadefval

barColor        =  trm       ?  trmcolor      : 
                   bjrev     ?  bjrevcol      : 
                   rsicol    ?  rsicon        : 
                   hema      ?  hemabar       : na

habarColor      =  trm       ?  trmcolor      : 
                   bjrev     ?  habjrevcol    : 
                   rsicol    ?  rsicon        : 
                   hema      ?  hemabar       : na

p2              =  plot     ((t3vis   or (not disable and bjrev)) ? anT3Average : na, "T3 Fast" , color2  )
p1              =  plot     ((t3vis   or (not disable and bjrev)) ? nT3Average  : na, "T3 Slow" , myColor )

hemaslowplot    =  plot     ((hemavis or (not disable and hema))  ? bjhemaslow  : na, 'EMA 5'   , mycolor )
hemaslowplot2   =  plot     ((hemavis or (not disable and hema))  ? bjhemaslow2 : na, 'EMA 9'   , mycolor2)
hemaslowplot3   =  plot     ((hemavis or (not disable and hema))  ? bjhemaslow3 : na, 'EMA 21'  , mycolor3)

hemafastplot    =  plot     (bjhemafast , 'EMA fast', #FF000000 , display=display.none, editable=false)

fill               (hemaslowplot , hemafastplot , color= hemavis or hema  ? hfillCol1  : na, title= "5 - Plot Fill")
fill               (hemaslowplot , hemaslowplot2, color= hemavis or hema  ? hfillCol2  : na, title= "5 - 9 Fill"   )
fill               (hemaslowplot2, hemaslowplot3, color= hemavis or hema  ? hfillCol3  : na, title= "9 - 21 Fill"  )
fill               (p1           , p2           , color= t3vis   or bjrev ? revFillCol : na, title= "T3 Fill"      )

plotshape          (arrowvis ? data   : na, style=shape.triangleup,   location=location.belowbar, color= #17ff00, title="Curl Up"  )
plotshape          (arrowvis ? dtat   : na, style=shape.triangledown, location=location.abovebar, color= #FFEB3B, title="Curl Down")

plot               (psarvis  ? sar    : na, "SAR"       , colUp     , style=   plot.style_circles  , linewidth= 1)
plot               (rPrice   ? close  : na, "Real Close", habarColor, trackprice=true, offset=-9999, editable=  false)

candleCol       =  haover and hollow  ? c_ < o_  ? habarColor : na :
                   haover and candle  ?            habarColor : na

wickCol         =  (hollow or candle) and haover ? #b2b5be    : na
borderCol       =  (hollow or candle) and haover ? habarColor : na
barCol          =  haover and bars               ? habarColor : na

plotcandle         (o_, h_, l_, c_, "Hollow Candles", candleCol, wickCol, bordercolor= borderCol)
plotbar            (o_, h_, l_, c_, 'Heikin-Ashi'   , barCol) 

barcolor           (color=barColor)

// ================================== //
// -----> Alert Functionality <------ //
// ================================== //

alertcondition     (buy4        ,   "TRM Buy"           ,   "TRM Buy on {{interval}} chart. Price is {{close}}"                     )
alertcondition     (sell4       ,   "TRM Sell"          ,   "TRM Sell on {{interval}} chart. Price is {{close}}"                    )
alertcondition     (revbar      ,   "Bj Reversal"       ,   "Reversal candle is forming on {{interval}} chart. Price is {{close}}"  )
alertcondition     (trendbar    ,   "T3 Crossing T3"    ,   "A new trend may be forming on {{interval}} chart. Price is {{close}}"  )
alertcondition     (revup       ,   "Reversal Up"       ,   "Reversal up on {{interval}} chart. Price is {{close}}"                 )
alertcondition     (revdwn      ,   "Reversal Down"     ,   "Reversal down on {{interval}} chart. Price is {{close}}"               )
alertcondition     (emasig      ,   "MA Alert"          ,   "Price is crossing MAs on {{interval}} chart. Price is {{close}}"       )
alertcondition     (Hcrossup    ,   "MA Cross Up"       ,   "MA crossed up on {{interval}} chart. Price is {{close}}"               )    
alertcondition     (Hcrossdwn   ,   "MA Cross Down"     ,   "MA crossed down on {{interval}} chart. Price is {{close}}"             )
alertcondition     (rsicross    ,   "RSI Signal"        ,   "RSI crossed 50 on {{interval}} chart. Price is {{close}}"              )
alertcondition     (rsiup       ,   "RSI Cross Up"      ,   "RSI crossed up 50 on {{interval}} chart. Price is {{close}}"           )
alertcondition     (rsidwn      ,   "RSI Cross Down"    ,   "RSI crossed down 50 on {{interval}} chart. Price is {{close}}"         )
alertcondition     (rsiob       ,   "RSI Overbought"    ,   "RSI crossed overbought on {{interval}} chart. Price is {{close}}"      )
alertcondition     (rsios       ,   "RSI Oversold"      ,   "RSI crossed oversold on {{interval}} chart. Price is {{close}}"        )
alertcondition     (rsicd       ,   "RSI momo down"     ,   "RSI crossed down 70 on {{interval}} chart. Price is {{close}}"         )
alertcondition     (rsicu       ,   "RSI momo up"       ,   "RSI crossed up 30 on {{interval}} chart. Price is {{close}}"           )
alertcondition     (data        ,   "TSI Curl Up"       ,   "TSI curled up on {{interval}} chart. Price is {{close}}"               )
alertcondition     (dtat        ,   "TSI Curl Down"     ,   "TSI curled down on {{interval}} chart. Price is {{close}}"             )
alertcondition     (sarup       ,   "Sar Up"            ,   "SAR triggered up {{interval}} chart. Price is {{close}}"               )
alertcondition     (sardwn      ,   "Sar Down"          ,   "SAR triggered down {{interval}} chart. Price is {{close}}"             )

_alert             (buy4        and BUY4                ,   'TRM Buy on '               )
_alert             (sell4       and SELL4               ,   'TRM Sell on '              )
_alert             (revup       and REVUP               ,   'Reversal up on '           )
_alert             (revdwn      and REVDWN              ,   'Reversal down on '         )
_alert             (Hcrossup    and HCROSSUP            ,   'MA crossed up on '         )
_alert             (Hcrossdwn   and HCROSSDWN           ,   'MA crossed down on '       )
_alert             (rsiup       and RSIUP               ,   'RSI crossed up 50 on '     )
_alert             (rsidwn      and RSIDWN              ,   'RSI crossed down 50 on '   )
_alert             (rsiob       and RSIOB               ,   'RSI crossed up 70 on '     )
_alert             (rsios       and RSIOS               ,   'RSI crossed down 30 on '   )
_alert             (rsicd       and RSICD               ,   'RSI crossed down 70 on '   )
_alert             (rsicu       and RSICU               ,   'RSI crossed up 30 on '     )
_alert             (data        and DATA                ,   'TSI curled up on '         )
_alert             (dtat        and DTAT                ,   'TSI curled down on '       )
_alert             (sarup       and SARUP               ,   'SAR triggered up '         )
_alert             (sardwn      and SARDWN              ,   'SAR triggered down '       )


//  ____  __ _  ____ 
// (  __)(  ( \(    \
//  ) _) /    / ) D (
// (____)\_)__)(____/
