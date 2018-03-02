## Sample Codes:

### Get FXCM System Parameters: C++
	void FixApplication::onMessage(const FIX44::TradingSessionStatus& tss, const SessionID& session_ID)
	{
		int param_count = FIX::IntConvertor::convert(tss.getField(9016));
	 
		cout << "TSS - FXCM System Parameters" << endl;
		for(int i = 1; i =< param_count; i++){
			FIX::FieldMap map  = tss.getGroupRef(1,9016);
	 
			string param_name  = map.getField(9017);
			string param_value = map.getField(9018);
	 
			cout << param_name << " - " << param_value << endl;
		}
	}
	
### Get Rollover Interest: C++
	void FixApplication::onMessage(const FIX44::TradingSessionStatus& tss, const SessionID& session_ID)
	{
		int symbols_count = IntConvertor::convert(tss.getField(FIELD::NoRelatedSym));
		for(int i = 1; i <= symbols_count; i++) {
			FIX44::SecurityList::NoRelatedSym symbols_group;
			tss.getGroup(i,symbols_group);
			string symbol = symbols_group.getField(FIELD::Symbol);
	 
			cout << "    Symbol -> " << symbol << endl;
			cout << "      RolloverBuy -> " << symbols_group.getField(9003) << endl;
			cout << "      RolloverSell -> " << symbols_group.getField(9004) << endl;
		}
	}
	
### Determine Hedging Status: C++
	void FixApplication::onMessage(const FIX44::CollateralReport& cr, const SessionID& session_ID)
	{
		FIX44::CollateralReport::NoPartyIDs group;
		cr.getGroup(1,group); 
		cout << "  Parties -> "<< endl;
	 
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
	}	
	
### Subscribe to a Symbol: C++
	string request_ID = "EUR_USD_Request_";
	FIX44::MarketDataRequest request;
	request.setField(MDReqID(request_ID));
	request.setField(SubscriptionRequestType(
		SubscriptionRequestType_SNAPSHOT_PLUS_UPDATES));
	request.setField(MarketDepth(0));
	request.setField(NoRelatedSym(1));
	 
	FIX44::MarketDataRequest::NoRelatedSym symbols_group;
	symbols_group.setField(Symbol("EUR/USD"));
	request.addGroup(symbols_group);
	 
	FIX44::MarketDataRequest::NoMDEntryTypes entry_types;
	entry_types.setField(MDEntryType(MDEntryType_BID));
	request.addGroup(entry_types);
	entry_types.setField(MDEntryType(MDEntryType_OFFER));
	request.addGroup(entry_types);
	entry_types.setField(MDEntryType(MDEntryType_TRADING_SESSION_HIGH_PRICE));
	request.addGroup(entry_types);
	entry_types.setField(MDEntryType(MDEntryType_TRADING_SESSION_LOW_PRICE));
	request.addGroup(entry_types);
	 
	Session::sendToTarget(request, sessionID);
	
### Subscribe to All Symbols: C++

	void FixApplication::onMessage(const FIX44::TradingSessionStatus& tss, const SessionID& session_ID)
	{
		FIX44::MarketDataRequest request;
		request.setField(MDReqID(NextRequestID()));
		request.setField(SubscriptionRequestType(
			SubscriptionRequestType_SNAPSHOT_PLUS_UPDATES));
		request.setField(MarketDepth(0));
	 
		FIX44::MarketDataRequest::NoMDEntryTypes entry_types;
		entry_types.setField(MDEntryType(MDEntryType_BID));
		request.addGroup(entry_types);
		entry_types.setField(MDEntryType(MDEntryType_OFFER));
		request.addGroup(entry_types);
		entry_types.setField(MDEntryType(MDEntryType_TRADING_SESSION_HIGH_PRICE));
		request.addGroup(entry_types);
		entry_types.setField(MDEntryType(MDEntryType_TRADING_SESSION_LOW_PRICE));
		request.addGroup(entry_types);
	 
		int symbols_count = IntConvertor::convert(tss.getField(FIELD::NoRelatedSym));
		request.setField(NoRelatedSym(symbols_count));
		for(int i = 1; i <= symbols_count; i++){
			FIX44::SecurityList::NoRelatedSym symbols_group_SL;
			tss.getGroup(i,symbols_group_SL);
			string symbol = symbols_group_SL.getField(FIELD::Symbol);
	 
			FIX44::MarketDataRequest::NoRelatedSym symbols_group_MDR;
			symbols_group_MDR.setField(Symbol(symbol));
			request.addGroup(symbols_group_MDR);
		}
	 
		Session::sendToTarget(request, sessionID);
	}

### Create Market Order: C++
	FIX44::NewOrderSingle order;
	order.setField(FIX::ClOrdID(NextClOrdID())); 
	order.setField(FIX::Account(account));
	order.setField(FIX::Symbol("EUR/USD")); 
	order.setField(FIX::Side(FIX::Side_BUY)); 
	order.setField(FIX::TransactTime(FIX::TransactTime())); 
	order.setField(FIX::OrderQty(10000));
	order.setField(FIX::OrdType(FIX::OrdType_MARKET));
	 
	FIX::Session::sendToTarget(order,session_id);
	
### Create Market Range Order: C++
In the case of a market range order, we set the OrdType to StopLimit and we must set the StopPx tag. The StopPx tag indicates the worst price we are willing to get filled at; i.e., the stop.

	FIX44::NewOrderSingle order;
	order.setField(FIX::ClOrdID(NextClOrdID())); 
	order.setField(FIX::Account(account));
	order.setField(FIX::Symbol("EUR/USD")); 
	order.setField(FIX::Side(FIX::Side_BUY)); 
	order.setField(FIX::TransactTime(FIX::TransactTime())); 
	order.setField(FIX::OrderQty(10000));
	order.setField(FIX::OrdType(FIX::OrdType_STOPLIMIT));
	order.setField(FIX::StopPx(stop));
	 
	FIX::Session::sendToTarget(order,session_id);	

### Create Entry (Pending) Order: C++
	FIX44::NewOrderSingle order;
	order.setField(FIX::ClOrdID(NextClOrdID())); 
	order.setField(FIX::Account(account));
	order.setField(FIX::Symbol("EUR/USD")); 
	order.setField(FIX::Side(FIX::Side_BUY)); 
	order.setField(FIX::TransactTime(FIX::TransactTime())); 
	order.setField(FIX::OrderQty(10000));
	order.setField(FIX::OrdType(FIX::OrdType_LIMIT)); 
	order.setField(FIX::Price(price));
	 
	FIX::Session::sendToTarget(order,session_id);

### Create One-Cancels-Other (OCO) Order: C++
	FIX44::NewOrderList olist;
	 
	olist.setField(FIX::ListID(NextClOrdID())); 
	olist.setField(FIX::TotNoOrders(2));
	olist.setField(FIX::ContingencyType(FIX::ContingencyType_ONE_CANCELS_THE_OTHER)); 
	 
	FIX44::NewOrderList::NoOrders stop;
	stop.setField(FIX::ClOrdID(next_ClOrdID())); 
	stop.setField(FIX::ListSeqNo(0)); 
	stop.setField(FIX::ClOrdLinkID("1")); 
	stop.setField(FIX::Account(account));
	stop.setField(FIX::Symbol(symbol)); 
	stop.setField(FIX::Side(FIX::Side_SELL)); 
	stop.setField(FIX::OrderQty(20000));
	stop.setField(FIX::OrdType(FIX::OrdType_STOP)); 
	stop.setField(FIX::StopPx(stop_price));
	olist.addGroup(stop);
	 
	FIX44::NewOrderList::NoOrders limit;
	limit.setField(FIX::ClOrdID(next_ClOrdID())); 
	limit.setField(FIX::ListSeqNo(1)); 
	limit.setField(FIX::ClOrdLinkID("1")); 
	limit.setField(FIX::Account(account));
	limit.setField(FIX::Symbol(symbol)); 
	limit.setField(FIX::Side(FIX::Side_SELL));
	limit.setField(FIX::OrderQty(20000));
	limit.setField(FIX::OrdType(FIX::OrdType_LIMIT)); 
	limit.setField(FIX::Price(limit_price));
	olist.addGroup(limit);
	 
	FIX::Session::sendToTarget(olist,session_id);

### Create Entry with Limit and Stop (ELS) Order: C++
The entry with limit and stop is a FXCM specific contingency type that allows you to associate a stop and limit with a specific position (or market order). In this case, the ContingencyType field must be set to “101” for the ELS contingency. When the stop or limit is executed, or when you close the position, these contingent orders will be deleted automatically. 

	FIX44::NewOrderList olist;
	 
	olist.setField(FIX::ListID(next_ClOrdID())); 
	olist.setField(FIX::TotNoOrders(3));
	olist.setField(FIX::FIELD::ContingencyType,"101");
	 
	FIX44::NewOrderList::NoOrders order;
	order.setField(FIX::ClOrdID(next_ClOrdID()));
	order.setField(FIX::ListSeqNo(0)); 
	order.setField(FIX::ClOrdLinkID("1"));
	order.setField(FIX::Account(account));
	order.setField(FIX::Symbol(symbol));
	order.setField(FIX::Side(FIX::Side_BUY));
	order.setField(FIX::Symbol(symbol));
	order.setField(FIX::OrderQty(10000));
	order.setField(FIX::OrdType(FIX::OrdType_MARKET)); 
	olist.addGroup(order);
	 
	FIX44::NewOrderList::NoOrders stop;
	stop.setField(FIX::ClOrdID(next_ClOrdID())); 
	stop.setField(FIX::ListSeqNo(1)); 
	stop.setField(FIX::ClOrdLinkID("2")); 
	stop.setField(FIX::Account(account));
	stop.setField(FIX::Side(FIX::Side_SELL));
	stop.setField(FIX::Symbol(symbol)); 
	stop.setField(FIX::OrderQty(10000)); 
	stop.setField(FIX::OrdType(FIX::OrdType_STOP)); 
	stop.setField(FIX::StopPx(stop_price));
	olist.addGroup(stop);
	 
	FIX44::NewOrderList::NoOrders limit;
	limit.setField(FIX::ClOrdID(next_ClOrdID())); 
	limit.setField(FIX::ListSeqNo(2)); 
	limit.setField(FIX::ClOrdLinkID("2")); 
	limit.setField(FIX::Account(account));
	limit.setField(FIX::Side(FIX::Side_SELL));
	limit.setField(FIX::Symbol(symbol)); 
	limit.setField(FIX::OrderQty(10000)); 
	limit.setField(FIX::OrdType(FIX::OrdType_LIMIT)); 
	limit.setField(FIX::Price(limit_price));
	olist.addGroup(limit);
	 
	FIX::Session::sendToTarget(olist,session_id);
	
### Create Market Order with Trailing Stop: C++
In our example below, we use two orders with ELS contingency type (see above for details on ELS). Specifically, we send both a market order and a stop order. What makes this stop order a trailing stop is the existence of the FXCMPegFluctuatePts(9061) tag, which we have enumerated as FXCM_PEG_FLUCTUATE_PTS. This field is set to “10” which means our stop will trail the market at a rate of 10 pips. 

	FIX44::NewOrderList olist;
	olist.setField(FIX::ListID(next_ClOrdID())); 
	olist.setField(FIX::TotNoOrders(2)); 
	olist.setField(FIX::FIELD::ContingencyType,"101");
	 
	FIX44::NewOrderList::NoOrders order;
	order.setField(FIX::ClOrdID(next_ClOrdID()));
	order.setField(FIX::ListSeqNo(0)); 
	order.setField(FIX::ClOrdLinkID("1"));
	order.setField(FIX::Account(account)); 
	order.setField(FIX::Symbol(symbol)); 
	order.setField(FIX::Side(FIX::Side_BUY)); 
	order.setField(FIX::OrderQty(10000)); 
	order.setField(FIX::OrdType(FIX::OrdType_MARKET)); 
	olist.addGroup(order);
	 
	FIX44::NewOrderList::NoOrders stop;
	stop.setField(FIX::ClOrdID(next_ClOrdID())); 
	stop.setField(FIX::ListSeqNo(1)); 
	stop.setField(FIX::ClOrdLinkID("2"));
	stop.setField(FIX::Account(account)); 
	stop.setField(FIX::Side(FIX::Side_SELL));
	stop.setField(FIX::Symbol(symbol)); 
	stop.setField(FIX::OrderQty(10000)); 
	stop.setField(FIX::OrdType(FIX::OrdType_STOP)); 
	stop.setField(FIX::StopPx(stop_price));
	stop.setField(FXCM_PEG_FLUCTUATE_PTS, "10");
	olist.addGroup(stop);
	 
	FIX::Session::sendToTarget(olist,session_id);	
	
### Get Order Status and Executed Amount: C++
	void FixApplication::onMessage(const FIX44::ExecutionReport& er, const SessionID& session_ID)
	{
		string status  = er.getField(FIELD::OrdStatus);
		string execQty = er.getField(FIELD::CumQty);
	 
		cout << "ExecutionReport ->" << endl;
		cout << "  OrderStatus: " << status << endl;
		if(status == "2" /*Filled*/ || status == "8" /*Rejected */ || status == "4" /*Cancelled*/) {
			cout << "    Executed Amount: "  << execQty << endl;
		}
	}
	
### Request All Open Positions: C++
	FIX44::RequestForPositions request;
	request.setField(PosReqID(NextRequestID()));
	request.setField(PosReqType(PosReqType_POSITIONS));
	 
	request.setField(Account(account_ID)); 
	request.setField(SubscriptionRequestType(SubscriptionRequestType_SNAPSHOT_PLUS_UPDATES));
	request.setField(AccountType(
		AccountType_ACCOUNT_IS_CARRIED_ON_NON_CUSTOMER_SIDE_OF_BOOKS_AND_IS_CROSS_MARGINED));
	request.setField(TransactTime());
	request.setField(ClearingBusinessDate());
	request.setField(TradingSessionID("FXCM"));
	 
	Session::sendToTarget(request, sessionID);
	
###	Request Open Positions for a Single Account: C++
	FIX44::RequestForPositions request;
	request.setField(PosReqID(NextRequestID()));
	request.setField(PosReqType(PosReqType_POSITIONS));
	 
	request.setField(Account(account_ID)); 
	request.setField(SubscriptionRequestType(SubscriptionRequestType_SNAPSHOT_PLUS_UPDATES));
	request.setField(AccountType(
		AccountType_ACCOUNT_IS_CARRIED_ON_NON_CUSTOMER_SIDE_OF_BOOKS_AND_IS_CROSS_MARGINED));
	request.setField(TransactTime());
	request.setField(ClearingBusinessDate());
	request.setField(TradingSessionID("FXCM"));
	 
	request.setField(NoPartyIDs(1));
	FIX44::RequestForPositions::NoPartyIDs parties_group;
	parties_group.setField(PartyID("FXCM ID"));
	parties_group.setField(PartyIDSource('D'));
	parties_group.setField(PartyRole(3));
	parties_group.setField(NoPartySubIDs(1));
	FIX44::RequestForPositions::NoPartyIDs::NoPartySubIDs sub_parties;
	sub_parties.setField(PartySubIDType(PartySubIDType_SECURITIES_ACCOUNT_NUMBER));
	sub_parties.setField(PartySubID(account_ID));
	parties_group.addGroup(sub_parties);
	request.addGroup(parties_group);
	 
	Session::sendToTarget(request, sessionID);
	
### Get All Waiting Orders: C++
	FIX44::OrderMassStatusRequest request;
	request.setField(MassStatusReqID(NextRequestID()));
	request.setField(MassStatusReqType(MassStatusReqType_STATUS_FOR_ALL_ORDERS));
	request.setField(Account(account_ID));
	Session::sendToTarget(request, sessionID);	