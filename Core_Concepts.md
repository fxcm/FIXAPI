## Core Concepts:

#### Logon (A)
This is the first message that you must send. Any messages you send before this will be ignored. There are two ways to Logon, one is within the Logon message should include your Username(553) and Password(554). The other one is send Username(553) and Password(554) on User Request (35=BE). It is also important to note here that FXCM requires the TargetSubID on all messages, including your Logon. If you are not receiving any responses to your Logon message, it is likely because you have not included TargetSubID

	Example Logon Messages
	
	//Send Username/Password on Logon (35=A)
	8=FIX.4.4|9=114| 35=A |34=1| 49=fx1294946_client1| 52=20120927-13:15:34.754| 56=FXCM |57=U100D1| 553=fx1294946| 554=123| 98=0 |108=30 |141=Y| 10=146|

	//Send Username/Password on User Request(35=BE)
	8=FIX.4.4|9=114|35=BE|34=2|49=fx1294946_client1|52=20140515-00:29:11.372|56=FXCM|57=U100D1|553=fx1294946|554=1234|923=1|924=1|10=150|

	//FXCM Logon response 
	8=FIX.4.4|9=92| 35=A| 34=1| 49=FXCM| 50=U100D1 |52=20120927-13:15:34.810| 56=fx1294946_client1| 98=0| 108=30| 141=Y| 10=187|
	

#### TradingSessionStatusRequest (g)
After successful authentication, it is necessary to receive an update on the status of the market (open or closed), common system parameters, and a list of securities with their characteristics. The TradingSessionStatus message is returned as a response to the TradingSessionStatusRequest and it accomplishes these things.

#### TradingSessionStatus Message (h)
The TradingSessionStatus message is used to provide an update on the status of the market. Furthermore, this message contains useful system parameters as well as information about each trading security (embedded SecurityList).

TradingSessionStatus should be requested upon successful Logon and subscribed to. The contents of the TradingSessionStatus message, specifically the SecurityList and system parameters, should dictate how fields are set when sending messages to FXCM. For example, the instrument component within SecurityList indicates minimum order quantity (9095). Subsequent NewOrderSingle (D) messages should not violate this value when setting the OrderQty field (38).

	Requesting TradingSessionStatus
	FIX44::TradingSessionStatusRequest request;
	request.setField(FIX::TradSesReqID(NextId())); 
	request.setField(FIX::SubscriptionRequestType(FIX::SubscriptionRequestType_SNAPSHOT_PLUS_UPDATES));
 
	Reading System Parameters
	Here we demonstrate how to extract the FXCM system properties from the TradingSessionStatus message. The code below will print out the name and value of the each of the properties. The following custom fields are used:
	9016 - FXCMNoParam
	9017 - FXCMParamName
	9018 - FXCMParamValue
	int param_count = FIX::IntConvertor::convert(status.getField(9016));
 
	cout << "TSS - FXCM System Parameters" << endl;
	for(int i = 1; i <= param_count; i++)
	{
		 FIX::FieldMap map = status.getGroupRef(i,9016);
		 string param_name = map.getField(9017);
		 string param_value = map.getField(9018);
	 
		 cout << param_name << " - " << param_value << endl;
	}
	FIX::Session::sendToTarget(request,session_id);

#### CollateralInquiry (BB)
CollateralInquiry is used to request the CollateralReport(BA) message from FXCM. This message contains important account related information such as the account number. With the exception of FIX sessions used solely for market data, you should include this message in your login sequence.
The login you use to connect will have access to one or more trading accounts. You will receive a CollateralReport for each of these accounts. When sending or modifying orders, you must set the Account(1) tag. This tag value must be set only to an account that you have actually received a ColleteralReport for; otherwise, you will see a rejection.

#### Requesting Market Data
*	Subscribe market price with 35=V MarketDataRequest. You will get MarketDataSnapshotFullRefresh(W) 35=W.
*	The MarketDataSnapshotFullRefresh(W) message contains the updates to market data. It is obtained as a response to the MarketDataRequest(V) message. FIX connections are then subscription based for the market data; meaning, you must request it to receive it.

*	The types of data you can receive, such as the Bid price or Offer price, are referred to as MDEntryTypes in FIX. FXCM supports the following MDEntryTypes in each message: Bid(0), Offer(1), High Price(7), and Low Price(8). Additional MDEntryTypes such as MDEntryDate, MDEntryTime, QuoteCondition, etc., are found only once within the first repeating group of the message.

*	Also we don't have liquidity size information and depth information, only display BBO (best bid offer).

```
8=FIX.4.4|9=497|35=W|34=4|49=FXCM|50=U100D1|52=20180817-13:44:06.495|56=MD_d101968187_client1|55=EUR/JPY|228=1|231=1|460=4|9001=3|9002=0.01|9005=10|9011=0|9020=0|9080=1|9090=0|9091=0|9092=0|9093=0|9094=50000000|9095=1|9096=O|268=4|269=0|270=126.085|271=0|272=20180817|273=13:44:06.000|336=FXCM|625=PSFX|276=A|282=PSFX_DESK|299=FXCM-EURJPY-19288641|537=1|269=1|270=126.093|271=0|272=20180817|273=13:44:06.000|336=FXCM|625=PSFX|276=A|282=PSFX_DESK|299=FXCM-EURJPY-19288641|537=1|269=7|270=126.448|269=8|270=125.567|10=117|
```
*	If you want subscribe once and get update been pushed whenever there is update, you need set Subscription request type 263=1 which is snapshot + update.  If 0, you will only get one snapshot. 
*	You can also subscribe security in list, instead of just one security. 
```
8=FIX.4.4|9=134|35=V|34=5|49=d101970033_client2|52=20180822-19:20:49.137|56=FXCM|57=U100D1|262=4|263=1|264=3|265=0|146=1|55=EUR/USD|267=2|269=0|269=1|10=189|
8=FIX.4.4|9=808|35=V|34=5|49=MD_d101968187_client1|52=20180817-13:44:05.840|56=FXCM|57=U100D1|262=4|263=1|264=0|265=0|146=65|55=USOil|55=AUD/JPY|55=NZD/CAD|55=EUR/CAD|55=USD/ZAR|55=AUS200|55=UKOil|55=EUR/NOK|55=NGAS|55=EUR/AUD|55=USD/HKD|55=EUSTX50|55=GBP/CAD|55=USD/CAD|55=GER30|55=CAD/CHF|55=USD/TRY|55=EUR/TRY|55=Copper|55=HKG33|55=USOilF2|55=GBP/AUD|55=NAS100|55=EUR/CHF|55=TRY/JPY|55=AUD/NZD|55=USD/CHF|55=XAU/USD|55=FRA40|55=USOilF|55=AUD/USD|55=NZD/JPY|55=USD/MXN|55=USDOLLAR|55=CHN50|55=ESP35|55=EUR/NZD|55=UKOilF|55=ZAR/JPY|55=GBP/CHF|55=NZD/USD|55=USD/JPY|55=GBP/NZD|55=SPX500|55=CHF/JPY|55=UK100|55=EUR/USD|55=SOYF|55=GBP/USD|55=EUR/JPY|55=AUD/CHF|55=EUR/GBP|55=XAG/USD|55=US30|55=GBP/JPY|55=NZD/CHF|55=USD/NOK|55=CAD/JPY|55=AUD/CAD|55=Bund|55=USD/SEK|55=EUR/SEK|55=USD/CNH|55=JPN225|55=UKOilF2|267=2|269=0|269=1|10=004|
```

#### Table
*	Send Collateral Inquiry 35=BB, you will get Collateral report 35=BA, which contains account information
```
8=FIX.4.4|9=88|35=BB|34=4|49=d101970033_client2|52=20180817-20:17:27.716|56=FXCM|57=U100D1|263=1|909=3|10=203|
8=FIX.4.4|9=361|35=BA|34=5|49=FXCM|50=U100D1|52=20180817-20:17:27.737|56=d101970033_client2|1=01960313|53=1000|336=FXCM|625=U100D1|898=0|901=1000562.37|908=4647057334|909=3|910=0|911=1|912=Y|921=1000562.37|922=1000562.37|9038=0|9045=N|9046=0|9047=0|453=1|448=FXCM ID|447=D|452=3|802=5|523=1960313|803=10|523=d101970033|803=2|523=fix-test138|803=22|523=32|803=26|523=Y|803=4000|10=033|
```
*	For Open positions, send Request for positions 35=AN with 724=0 (positions). You will get position report 35=AP for each open position. If you don’t have open position, you will get “No open positions” in message 35=AO

```
Sample for open positions request:
8=FIX.4.4|9=149|35=AN|34=5|49=d101968168_client1|52=20151111-21:01:12.396|56=FXCM|57=U100D1|1=01958448|60=20151111-21:01:12.395|263=1|581=6|710=4|715=20151111|724=0|10=085|
 
Sample for open position.
8=FIX.4.4|9=565|35=AP|34=8|49=FXCM|50=U100D1|52=20151111-20:19:59.929|56=d101968168_client1|1=01958448|11=FIX.4.4:d101968168_client1->FXCM/U100D1-1437981786837-10|15=EUR|37=207486895|55=EUR/USD|60=20150727-07:23:08|325=N|336=FXCM|526=fix_example_test|581=6|625=U100D1|710=4|715=20151111|721=3684204026|724=0|727=2|728=0|730=1.10728|731=1|734=0|912=N|9000=1|9038=260|9040=-21.16|9041=80775478|9042=20150727-07:23:08|9053=0.8|453=1|448=FXCM ID|447=D|452=3|802=4|523=32|803=26|523=d101968168|803=2|523=fix-test112|803=22|523=1958448|803=10|702=1|703=TQ|704=10000|753=1|707=CASH|708=0|10=137|

```
*	For closed positions, please send positions request 35=AN with 724=1 (Trades). You will receive position report in 35=AP

```
Sample for closed positions request:
8=FIX.4.4|9=177|35=AN|34=6|49=d101968168_client1|52=20151111-21:01:12.400|56=FXCM|57=U100D1|1=01958448|60=20151111-21:01:12.400|263=1|581=6|710=5|715=20151111|724=1|9012=20150311|9014=20151112|10=110|
 
Sample for closed position.
8=FIX.4.4|9=702|35=AP|34=20|49=FXCM|50=U100D1|52=20151111-21:01:11.936|56=d101968168_client1|1=01958448|11=FIX.4.4:d101968168_client1->FXCM/U100D1-1428599035518-4|15=EUR|37=202027586|55=EUR/USD|60=20150519-03:30:43|325=N|336=FXCM|526=fix_example_test|581=6|625=U100D1|710=5|715=20151111|721=3533878441|724=1|727=13|728=0|730=1.06572|731=1|734=0|912=Y|9000=1|9040=-6.08|9041=78911063|9042=20150409-17:03:56|9043=1.12979|9044=20150519-03:30:43|9048=U100D1_16679142D2EE08ABE053142B3C0A452A_05192015032653174913_QCV-127|9049=FXTS|9052=640.7|9053=0.8|9054=204437509|453=1|448=FXCM ID|447=D|452=3|802=4|523=32|803=26|523=d101968168|803=2|523=fix-test112|803=22|523=1958448|803=10|702=1|703=TQ|704=10000|753=1|707=CASH|708=0|10=042|
```

#### Order

*	Please set account number on tag 1, 1=00648329 when you place orders. Otherwise you will get error "No Account specified".
*	Place market order via 35=D. you will get execution report in 35=8

```
Open market position:
20160411-06:16:50.909 : 8=FIX.4.4 9=163 35=D 34=7 49=D101546502001_client1 52=20160411-06:16:50.909 56=FXCM 57=U100D1 1=01537581 11=635959630109097564 38=10 40=1 54=1 55=SPX500 59=1 60=20160411-06:16:50 10=054

Sample execution report 35=8
20160411-06:16:51.399 : 8=FIX.4.4 9=478 35=8 34=15 49=FXCM 50=U100D1 52=20160411-06:16:51.177 56=D101546502001_client1 1=01537581 6=2047.53 11=635959630109097564 14=10 15=USD 17=821172034 31=2047.53 32=10 37=225909074 38=10 39=2 40=1 44=2047.53 54=1 55=SPX500 58=Executed 59=1 60=20160411-06:16:51 99=0 150=F 151=0 211=0 336=FXCM 625=U100D1 835=0 836=0 1094=0 9000=1010 9041=89603919 9050=OM 9051=F 9061=0 453=1 448=FXCM ID 447=D 452=3 802=4 523=1537581 803=10 523=d101546502001 803=2 523=Halpert 803=22 523=32 803=26 10=088

```

#### Limit Order

*	There are two cases of limit order base on TIF and they are different
*	Case 1: TIF tag59 as GTC/GTD limit order: it work like entry order, because order will be executed in the future. If long limit price <= Ask. 

```
DEBUG (2016-01-18 21:15:54,015) [QF/J Session dispatcher: FIX.4.4:FXCM/MINIREAL->1601094176_client2] (app) - <<< app message from counterparty: 8=FIX.4.4|9=182|35=D|34=522|49=1601094176_client2|52=20160119-02:15:54|56=FXCM|57=MINIREAL|1=1601094176|11=F029d160118t211554L0015|38=21000|40=2|44=0.68657|54=1|55=AUD/USD|59=1|60=20160119-02:15:54|10=022|

```

*	Case 2: TIF as IOC/FOK. it is executed immediately at market. for buy limit IOC, limit price should >= Ask

```
DEBUG (2016-01-18 23:15:54,053) [QF/J Session dispatcher: FIX.4.4:FXCM/MINIREAL->1601094176_client2] (app) - <<< app message from counterparty: 8=FIX.4.4|9=181|35=D|34=804|49=1601094176_client2|52=20160119-04:15:54|56=FXCM|57=MINIREAL|1=1601094176|11=F085d160118t231554L0044|38=7000|40=2|44=1.45063|54=1|55=USD/CAD|59=3|60=20160119-04:15:54|10=217|

```

#### Getting Positions:

Open and closed positions are retrieved through the PositionReport (AP) message. Unlike the ExecutionReport (8) message which contains information relating to orders, the PositionReport is not automatically sent to your FIX client. You can make individual requests for PositionReport, or you can subscribe to updates on this message. Sending a RequestForPositions (AN) message with SubscriptionRequestType (263) set to 1 (SnapshotAndUpdates) will subscribe to updates.	

Open Position vs. Closed Position
PosReqType (724) is used to determine if a received PositionReport represents an open position or closed position. A value of 0 indicates an open position while a value of 1 indicates a closed position.

Open Positions
When a PositionReport representing an open position is sent to you, it will contain the price at which the position was opened. This can be seen using SettlPrice (730). The close price and the P/L of the position are not present given that the position is open. Close price and P/L are real-time calculated values and are not contain in any PositionReport where PosReqType (724) = 0 (Open Position).

Closed Positions
PositionReport messages representing closed positions will include the price at which the position was closed, as well as other useful information such as P/L. The following additional tags are present:

	Tag							Description
	FXCMPosClosePNL (9052)		Gross P/L of the position; e.g., $24.17, or €43.72
	FXCMPosInterest (9040)		Rollover interest applied to the position
	FXCMPosCommission (9053)	Commission applied to the position
	FXCMCloseSettlPrice (9043)	Close price of the position
	
Position Margin
The margin applied to each individual position can be obtained from a PositionReport representing an open position. FXCMUsedMargin (9038) will contain this margin value. Note that the total margin required for an account can be obtained by this same tag from the CollateralReport (BA) message.	

#### Overview of Basic Order and Time-In-Force Types

Time-In-Force (TIF) Types
	The Time-In-Force (59) Tag is used to indicate how long an order should remain active before it is either executed by the broker or cancelled by the client. Below are the four TIF values with descriptions.

	Good Til Cancel (GTC)
	Orders with this TIF value remain open and active until fully executed or cancelled. This means that the order remains active until the entire order amount is executed.

	When to use GTC
	Use GTC when your order must remain active until it can be filled
	Use GTC when your entire order must get filled
	Day
	Orders with this TIF value will remain open and active until fully executed, cancelled by the client, or when the trading day ends. Like Good Til Cancel (GTC), this means the order will remain active until the entire order amount is executed, unless the order is cancelled or the trading day ends.

	When to use Day
	Use Day when your original intention for the order becomes obsolete with time.
	Immediate or Cancel (IOC)
	Orders with this TIF value will immediately attempt to execute as much of your order as possible and cancel any remaining amount if necessary. As a result, this TIF value will allow partial fills.

	When to use IOC
	Use IOC when you expect execution to take place immediately
	Use IOC when it is acceptable if your entire order does not get filled
	Fill or Kill (FOK)
	Orders with this TIF value will attempt to execute the entire order amount immediately. If the entire order amount cannot be executed, the order is cancelled.

	When to use FOK
	Use FOK when you expect execution to take place immediately
	Use FOK when your entire order must get filled

Order Types
	Market
	A market order is an order to buy or sell immediately at the next available price. This means that the order is not guaranteed to fill at any specific price.

	When to Use Market
	Use market when your order being filled is more important than the price it is filled at
	Supported TIF Values
	GTC, DAY, IOC, and FOK

	Market Range (Stop-Limit)
	Market range is a market order that comes with a limitation on the price at which the order can be filled. In other words, it is a market order with a protection against slippage (price deviation).

	In FIX terms, you can convert a market order to a market range order by setting the OrdType (40) to 4 (Stop-Limit) and by setting the StopPx(99) Tag. The StopPx tag value should be set to the worst price you would accept being filled.

Example

	Assume you want to Buy EUR/USD now while it is trading at 1.4531 but you do not want to get filled at a price higher than 1.4535. In this case you would set the StopPx (99) tag value to 1.4535. If your order cannot be filled at 1.4535 or below, it will be cancelled.

	When to use Market Range
	Use market range when you are concerned about slippage
	Use market range when it is acceptable that your order may be cancelled
	Supported TIF Values
	IOC and FOK

	Limit
	A limit order is an order to buy or sell only at a specific price (or better). In other words, the order can only be filled at the limit price or for some better price.

	When to use Limit
	Use limit when you must guarantee the price at which an order is filled
	Supported TIF Values
	GTC, Day, IOC, and FOK

	Common Applications
	The limit order can be used to achieve multiple objectives when combined with different TIF values. The two common application are:

	GTC/Day Limit Order

	Recall that both GTC and DAY remain active until the entire order is filled, until cancelled, or the trading day ends (for DAY orders). When you combine the limit order with these TIF values, you have an order that will remain active until the entire amount is filled at your limit price or better. This type of order is often used to close an existing position and ensure the position is closed at a specific rate.

	IOC/FOK Limit Order

	Recall that with IOC and FOK, your order will immediately attempt execution. In the case of IOC, part of the order will be filled if possible. In the case of FOK, the entire order must be filled. When you combine IOC/FOK with the limit order, you have an order which will attempt execution immediately but will fill only at your limit price or better. This order type is commonly used to guarantee the price at which a new order is filled while also controlling how much can be filled; IOC would allow partial fills while FOK would not.

	Stop
	A stop is an order to buy or sell some amount when the current market price reaches your stop price. In other words, a stop order is a market order which is waiting to be active until the market price reaches a certain level (your stop price). Given that a stop is effectively a type of market order, it does not guarantee any specific fill price.

	When to use Stop
	Use stop when your order being filled is more important than the price it is filled at
	Supported TIF Values
	GTC and DAY

#### Handling of Partial Fills

	Order Quantity Fields
	There are three fields which can be used to determine the quantity filled or rejected by FXCM. These fields are:

	LastQty (32) – the quantity filled on the last successful attempt to fill the order
	CumQty (14) – the total quantity filled
	LeavesQty (151) – the remaining quantity to be filled
	Importance of OrdStatus (39)
	It is important to consider OrdStatus (39) when using the quantity fields above. As FXCM is attempting to execute an order, the values of OrdStatus will progress from an initial value of New (0) to some final state. There are three possible final values for OrdStatus:

	OrdStatus = Filled (2)
	OrdStatus = Rejected (8)
	OrdStatus = Cancelled (4)
	When you receive an ExecutionReport (8) with OrdStatus set to one of these final values, you can inspect the CumQty (14) field to determine the total amount executed. If OrdStatus = Filled (2), the entire order was filled and CumQty will equal the original OrdQty value. If OrdStatus = Rejected (8), the order was partially filled and CumQty will be some value less than the original OrdQty.

	Example Partial Fill
	The following ExecutionReport messages serve as an example of a partially filled order. The original OrderQty(38) was 1,000,000. In this example only 600,000 of the order was filled. The most important line here is the last, where we can see a final OrdStatus value (Rejected in this case). When this last ExecutionReport is received, we can inspect CumQty(14) to see that 600,000 was filled.

	OrderQty(38) = 1,000,000; OrdStatus(39) = New 
	6=85.558 14=0 17=59342024 31=85.558 32=0 37=31654622 38=1000000 39=0 40=4 44=85.55 854=2 59=3 99=0 150=0 151=1000000 211=0835=0 836=0 1094=0 9000=17 9041=13151786 9050=OR 9051=P 9061=0

	OrderQty(38) = 1,000,000; OrdStatus(39) = Stopped 
	6=85.558 14=0 17=59342025 31=85.558 32=0 37=31654622 38=1000000 39=7 40=4 44=85.55 854=2 59=3 99=0 150=7 151=1000000 211=0835=0 836=0 1094=0 9000=17 9041=13151786 9050=OR 9051=U 9061=0

	OrderQty(38) = 1,000,000; OrdStatus(39) = Stopped 
	6=85.488 14=0 17=59342047 31=85.488 32=0 37=31654622 38=1000000 39=7 40=4 44=85.48 854=2 9=3 99=0 150=7 151=1000000 211=0835=0 836=0 1094=0 9000=17 9041=13151786 9050=OR 9051=U 9061=0

	OrderQty(38) = 1,000,000; OrdStatus(39) = Partially Filled; LastQty(32) = 600,000; CumQty(14) = 600,000 
	6=85.488 14=600000 17=59342048 31=85.488 32=600000 37=31654622 38=1000000 39=1 40=4 44=85.488 54=259=399=0 150=F 151=400000 211=0 835=0 836=0 1094=0 9000=17 9041=13151888 9050=OR 9051=U 9061=0

	OrderQty(38) = 1,000,000; OrdStatus(39) = Cancelled; CumQty(14) = 600,000 
	6=85.488 14=600000 17=59342049 31=85.488 32=0 37=31654622 38=1000000 39=8 40=4 44=85.488 54=258=Cancelled 59=399=0 150=4 151=0 211=0 835=0 836=0 1094=0 9000=17 9041=13151888 9050=OR 9051=R 9061=0

#### Closing a Position:
How you close a position depends upon the position maintenance type of the account. Some accounts support hedging while others do not. Hedging is the ability to have two positions in the same symbol but of a different side; for example, holding both Buy EUR/USD and Sell EUR/USD positions at the same time.

	Accounts with Hedging
	Accounts that support hedging allow you to close individual positions, regardless of when they were opened relative to other positions. Clearly with these accounts, Buy and Sell orders do not offset themselves but instead form a hedge. Consequently, you must close these positions with a NewOrderSingle message that specifies the TicketID to close.

	Sending a Closing Order
	NewOrderSingle (D) can be used to close a specific position simply by setting the FXCMPosID (9041) field. This converts a basic market order into a closing order.

	Closing Order in Code
		FIX44::NewOrderSingle order;
		 
		order.setField(ClOrdID(NextClOrdID())); 
		order.setField(Account(account_ID));
		order.setField(Symbol("EUR/USD")); 
		order.setField(Side(Side_BUY)); 
		order.setField(TransactTime()); 
		order.setField(OrderQty(10000));
		order.setField(OrdType(OrdType_MARKET));
		order.setField(FXCM_POS_ID/*9041*/, ”84736256”);
	 
	Session::sendToTarget(order, session_ID);
	Accounts without Hedging
	For accounts without hedging, orders of the opposite Side cancel each other out; e.g., sending a NewOrderSingle with a Side of Buy will net against any existing Sell positions. This netting is done in First-In, First-Out (FIFO) order. As a result, a basic market order will suffice to close any open position.

	Getting Account Position Maintenance
	The position maintenance type of each account can be retrieved from the Parties component of CollateralReport (BA). The NoPartySubIDs group contains a custom PartySubIDType for position maintenance. This specific PartySubIDType tag is set to a value of 4000. PartySubID can be checked for the value of position maintenance. Y = “Hedging Enabled,” N = “No Hedging,” and 0 = “Netting.” Anything other than Y implies hedging is disabled and we will not use closing orders.

	Getting Position Maintenance in Code
		int number_subID = IntConvertor::convert(group.getField(FIELD::NoPartySubIDs));
		for(int u = 1; u <= number_subID; u++){
			FIX44::CollateralReport::NoPartyIDs::NoPartySubIDs sub_group;
			group.getGroup(u, sub_group);
		 
			string sub_type  = sub_group.getField(FIELD::PartySubIDType);
			string sub_value = sub_group.getField(FIELD::PartySubID);
			if(sub_type == "4000"){
				// Check sub_value for position maintenance 
				// Y = Hedging
				// N = No Hedging
				// 0 = Netting
			}
		}

#### When to Reset MsgSeqNum
Reset on Logon
MsgSeqNum should be reset upon each Logon. This means that every Logon message should include tags MsgSeqNum (34) set to “1” and ResetSeqNumFlag (141) set to “Yes.” It is necessary to reset upon each Logon due to the fact that connections to FXCM are load balanced against a cluster of servers. This promotes a stable trading environment for users, but it also means you should reset upon each Logon.

	Example Logon Message
	8(BeginString)=FIX.4.4 
	9(BodyLength)=114  
	35(MsgType)=A  
	34(MsgSeqNum)=1
	49(SenderCompID)=sender_client1  
	52(SendingTime)=20120927-13:15:34.754  
	56(TargetCompID)=FXCM  
	57(TargetSubID)=U100D1  
	553(Username)=some_user  
	554(Password)=some_password 
	98(EncryptMethod)=0  
	108(HeartBtInt)=30  
	141(ResetSeqNumFlag)=Y
	10(CheckSum)=146
	
#### Account Equity
FIX API does not have a field which represents account equity. Equity is a real-time value that is dependent upon a floating price. If your application needs immediate access to equity in real-time, you would have to calculate it using market data. However, the CollateralReport (BA) does provide an equity value that corresponds with a specific time in the trading day.

StartCash(921)
The StartCash (921) field from CollateralReport is the equity value of the account at 5:00pm EST (New York). This can be used as a snapshot of what the equity was at that time. This value will include the account balance and any profit or loss on open trades.


#### EMF

Important detail to note about order execution with FXCM is the difference between order fill notification and order finished notification.
As an order is filled by a liquidity provider, client will be sent a fill confirmation in the form of an execution report that includes 35=8|39=7|150=F or, in case of a partial fill, 35=8|39=1|150=F. This confirmation is sent as soon as the LP confirms the trade.
After the order is completed and every database operation associated with it is committed, the client will be sent an execution report of order being done. This execution report includes 35=8|39=2|150=F.
Alternatively, if the order was filled only partially before being canceled, the final confirmation will include 35=8|39=4|150=4. You can find the remaining quantity that was not filled in tag 151.
It is important to note, that the final execution report can be sent much later. When looking for fill confirmations, clients can take advantage of faster notifications than before implementing  EMF.
Even if clients are not taking advantage of the EMF execution, they will always be notified of the orders being filled. The only difference would be the delivery delay.

Execution Disclaimer: FXCM aggregates bid and ask prices from a pool of liquidity providers and is the final counterparty when trading forex on FXCM's dealing desk and No Dealing Desk (NDD) execution models. With NDD, FXCM's platforms display the best-available direct bid and ask prices from the liquidity providers. In addition to the spread, the trading cost with NDD is a fixed lot-based commission at the open and close of the trade. While generally NDD accounts offer spreads with no markups, in some circumstances, FXCM may add a markup to NDD spreads. This may occur due to, but not limited to, account type, such as accounts opened through a referring agent. With dealing desk execution, FXCM can act as the dealer on any or all currency pairs. Backup liquidity providers fill in when FXCM does not act as the dealer. FXCM’s dealing desk has fewer liquidity providers than NDD. There are many other factors to consider when choosing an execution model (such as conflict of interest, trading style or strategy). See Execution Risks. Note: Contractual relationships with liquidity providers are consolidated through the FXCM Group, which, in turn, provides technology and pricing to the group affiliate entities.

