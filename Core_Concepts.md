## Core Concepts:

### Getting Connected:

After your application has created a FIX session, you can begin sending and receiving FIX messages. However, there is a sequence of messages that should be sent prior to conducting any other messaging activity. These messages are described below.

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
The MarketDataSnapshotFullRefresh(W) message contains the updates to market data. It is obtained as a response to the MarketDataRequest(V) message. FIX connections are then subscription based for the market data; meaning, you must request it to receive it.

The types of data you can receive, such as the Bid price or Offer price, are referred to as MDEntryTypes in FIX. FXCM supports the following MDEntryTypes in each message: Bid(0), Offer(1), High Price(7), and Low Price(8). Additional MDEntryTypes such as MDEntryDate, MDEntryTime, QuoteCondition, etc., are found only once within the first repeating group of the message.

	Sending MarketDataRequest(V) Message
	FIX44::MarketDataRequest mdr;
 
	mdr.set(FIX::MDReqID(NextId()));
	mdr.set(FIX::SubscriptionRequestType(FIX::SubscriptionRequestType_SNAPSHOT_PLUS_UPDATES));
	mdr.set(FIX::MarketDepth(0));
	mdr.set(FIX::NoMDEntryTypes(2));
 
	FIX44::MarketDataRequest::NoMDEntryTypes types_group;
	types_group.set(FIX::MDEntryType(FIX::MDEntryType_BID));
	mdr.addGroup(types_group);
	types_group.set(FIX::MDEntryType(FIX::MDEntryType_OFFER));
	mdr.addGroup(types_group);
 
	int no_sym = FIX::IntConvertor::convert(security_list.getField(FIX::FIELD::NoRelatedSym));
	for(int i = 1; i <= no_sym; i++)
	{
	   FIX44::SecurityList::NoRelatedSym sym_group;
	   mdr.addGroup(security_list.getGroup(i,sym_group));
	}
	 
	FIX::Session::sendToTarget(mdr,session_id);
	
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

	OrderQty(38) = 1,000,000; OrdStatus(39) = Rejected; CumQty(14) = 600,000 
	6=85.488 14=600000 17=59342049 31=85.488 32=0 37=31654622 38=1000000 39=8 40=4 44=85.488 54=258=Rejected 59=399=0 150=8 151=0 211=0 835=0 836=0 1094=0 9000=17 9041=13151888 9050=OR 9051=R 9061=0

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

#### Want to retrieve short version of market price:
FXCM give client an opportunity to retrieve market price for just Bid/Ask, please follow instructions at [here](https://docs.fxcorporate.com/api-message-info.pdf)

#### EMF

Important detail to note about order execution with FXCM is the difference between order fill notification and order finished notification.
As an order is filled by a liquidity provider, client will be sent a fill confirmation in the form of an execution report that includes 35=8|39=7|150=F or, in case of a partial fill, 35=8|39=1|150=F. This confirmation is sent as soon as the LP confirms the trade.
After the order is completed and every database operation associated with it is committed, the client will be sent an execution report of order being done. This execution report includes 35=8|39=2|150=F.
Alternatively, if the order was filled only partially before being canceled, the final confirmation will include 35=8|39=4|150=4. You can find the remaining quantity that was not filled in tag 151.
It is important to note, that the final execution report can be sent much later. When looking for fill confirmations, clients can take advantage of faster notifications than before implementing  EMF.
Even if clients are not taking advantage of the EMF execution, they will always be notified of the orders being filled. The only difference would be the delivery delay.

Execution Disclaimer: FXCM aggregates bid and ask prices from a pool of liquidity providers and is the final counterparty when trading forex on FXCM's dealing desk and No Dealing Desk (NDD) execution models. With NDD, FXCM's platforms display the best-available direct bid and ask prices from the liquidity providers. In addition to the spread, the trading cost with NDD is a fixed lot-based commission at the open and close of the trade. While generally NDD accounts offer spreads with no markups, in some circumstances, FXCM may add a markup to NDD spreads. This may occur due to, but not limited to, account type, such as accounts opened through a referring agent. With dealing desk execution, FXCM can act as the dealer on any or all currency pairs. Backup liquidity providers fill in when FXCM does not act as the dealer. FXCM’s dealing desk has fewer liquidity providers than NDD. There are many other factors to consider when choosing an execution model (such as conflict of interest, trading style or strategy). See Execution Risks. Note: Contractual relationships with liquidity providers are consolidated through the FXCM Group, which, in turn, provides technology and pricing to the group affiliate entities.

