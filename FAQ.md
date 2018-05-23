## FAQ Questions

* Why did I get the following error message: SEVERE: onMessageRecieved::Force reconnect from server. Update session state::19915;DAS 19915: ZDas Exception ORA-20103: Session expired.

	Unordered List ItemError code 20103 session expired, means your connection has been lost.
	This error message could be displayed due to a number of reasons, including network instability, a system issue or a client side program crash. If the problem is a system issue, please try to reboot.

* Why did I get the following error message: Done! request.RequestID: U10D1_0F742A280DE8276EE053182B3C0A1526_02192015163720178345_XIX0-2 offer.OfferID:1 AccountID: 831293 iAmount: 10000 dRate: 1.13953 dRateLimit: 1.14153 dRateStop: 1.13803 BuySell: B OrderType: LE 19915;DAS 19915: ZDas Exception ORA-20143: Order price too far from market price :1.14219 vs 1.14162

	Unordered List ItemThis error message is generated when the Buy Limit price is above the Bid price. For example if the Bid price was 1.13919, and your Buy limit 1.13953. If you want to place a Buy Limit above the Bid price you can do so using an OpenLimit order, which is available in API ver. 1.3.2. The fill price will be the limit price or better.

* Why did I get the following error message: 19915;DAS 19915: ZDas Exception ORA-20112: Limit price did not pass validation: A:308386 OF:22 SB:B

	Unordered List ItemThis error message is generated when the Limit price does not correspond to the ask price for the order type required. Ff the Time in Force is IOC or FOK then the Buy limit price should be >= Ask price. For GTC or GTD the Buy Limit should be ⇐ Ask price.

* Why did I get the following error message: 19915;DAS 19915: ZDas Exception ORA-20113: Insufficient margin in session:

	Unordered List ItemThis error message is generated when you don’t have enough margin.

* Why did I get the following error message: 19915;DAS 19915: ZDas Exception ORA-20102: Access Violation: U10D1_0C73B32A0A4299C5E053182B3C0ADFC8_01122015125318715906_UGQ1 phase: 4

	Unordered List ItemThis error message is generated when a trade account is missing from the dealer account.

* Why did I get the following error message: 19915: ZDas Exception ORA-20105: Order price did not pass validation 8=FIX.4.4|9=179|35=D|1=2620231783|11=FXSTAT_32951421147085|15=EUR|38=5|40=3|54=2|55=ESP35|59=1|60=20150113-11:04:45|99=9864|386=1|336=FXCM|625=EUREAL|516=0|526=OT_-2_1421147086_ESP35_ESP35_0__0_|10=080

	Unordered List ItemThe rejected orders error message is generated when the stop price is within 12 points of ask price. The reason for rejected orders is because the Minimum Stop Distance for ESP35 is 12 points. For example, if the Ask price was 9911 and your Stop price 99=9917, you would receive this error message. In this example, the stop price should be at least 9911 + 12 = 9923.

* Why did I get the following error message:ORA-20008: Failed to create order, primary validation

	This error message is generated when Range prices are below the Ask price. For example if orders were placed on news events, and the spreads got wider, e.g. If Buy range IOC 55=CAD/JPY 44=92.55100 99=95.55100 and Ask price was 95.612
	8=FIX.4.4 9=190 35=D 34=19 49=2620237129_client1 52=20150304-15:00:00 56=FXCM 57=EUREAL 1=2620237129 11=28020200337fa23 38=1000000 40=4 44=92.55100 54=1 55=CAD/JPY 59=3 60=20150304-14:57:19.363 99=95.55100 10=252
	DEBUG (2015-03-04 10:00:00,171) [0:Bus:2620237129_client1FXCMEUREAL] (app) - »> app message to counterparty: 8=FIX.4.4 9=558 35=8 34=18308 49=FXCM 50=EUREAL 52=20150304-15:00:00.171 56=2620237129_client1 1=2620237129 6=0 11=28020200337fa23 14=0 15=CAD 17=0 31=0 32=0 37=NONE 38=1000000 39=8 40=4 44=0 54=1 55=CAD/JPY 58=19915;DAS 19915: ZDas Exception ORA-20008: Failed to create order, primary validation

* How can I tell what account type I have?

	* Checking on Trading station: To check the type of account you have, you can login to Trading station and look in the tab “Accounts”. Scroll to the end and find column Type. Y = Hedging is allowed; N = Hedging is not allowed, O = Netting only, D = Day netting, F = FIFO
	* FIX: Using tag 453 and 803 (PartySubIDType) Y = Heding, N = No Hedging, 0 = Netting. This is located on page 34 of the documentation.

* Why did I get the following error message:“tag specified out of required order”

	This error message is generated when the tag order does not pass our system check. You can avoid this by setting ValidateFieldsOutOfOrder=N in your config file.

* Limit order Day order vs Limit IOC/FOK A Limit Order is an order to buy or sell a predetermined amount at a specified price. This order will be filled only when the market price equals the specified limit price or better. Limit orders also allow a trader to limit the length of time an order can be outstanding before being canceled with the following time in force values GTC, DAY, GTD, IOC, FOK.

	The Limit price for order with TIF GTC or DAY (for future execution) should be market or better than current market price, but for orders with TIF IOC or FOK (immediate execution) the Limit price should be market or worse. This is because IOC/FOK orders will be sent immediately for execution, without waiting for the market price to reach the Limit price.

	If the current Bid is 100, and you place a Sell Limit IOC/FOK at 102, this order will be rejected because the Limit price has to be market or worse for IOC/FOK, (it would be like asking to get order filled immediately at 102 or better).

	If your intention was to control the slippage, you can use a Limit OIC or FOK. The Limit price should be the Bid price or below. So in this example, if Limit=98, the order will be filled at 98 or better, but because it’s immediate execution, if for any reason the order can’t be filled in few attempts, the order will be canceled. Can you fix the paragraph spacing below?

* What is the Base Unit Size (also the minimum trade size) For all FX instruments - tag 53 in collateral report(35=BA) For each CFD - tag 228 in Trading Session Status ( 35=h)

* How can i get Positions side and quantity from open position report:

First you need to send position report request 35=AN with 724=0 (open position), 724=1 is for closed positions. 
if you have open position you will get positon report 35=AP for each open position. If you don’t have it you will receive “no open position” in message AO instead of 35=AP message.
In the positon report, you need to look at 704 (LongQty) or 705(shortQty)
If you see 704 it is long order (buy order), if you see 705 it is short order (sell order).

	8=FIX.4.4|9=149|35=AN|34=5|49=d101968168_client1|52=20151111-21:01:12.396|56=FXCM|57=U100D1|1=01958448|60=20151111-21:01:12.395|263=1|581=6|710=4|715=20151111|724=0|10=085|

	8=FIX.4.4|9=565|35=AP|34=8|49=FXCM|50=U100D1|52=20151111-20:19:59.929|56=d101968168_client1|1=01958448|11=FIX.4.4:d101968168_client1->FXCM/U100D1-1437981786837-10|15=EUR|37=207486895|55=EUR/USD|60=20150727-07:23:08|325=N|336=FXCM|526=fix_example_test|581=6|625=U100D1|710=4|715=20151111|721=3684204026|724=0|727=2|728=0|730=1.10728|731=1|734=0|912=N|9000=1|9038=260|9040=-21.16|9041=80775478|9042=20150727-07:23:08|9053=0.8|453=1|448=FXCM ID|447=D|452=3|802=4|523=32|803=26|523=d101968168|803=2|523=fix-test112|803=22|523=1958448|803=10|702=1|703=TQ|704=10000|753=1|707=CASH|708=0|10=137|

* How can i get closed position:

First you need to send position request with 724=1

	8=FIX.4.4|9=177|35=AN|34=6|49=d101968168_client1|52=20151111-21:01:12.400|56=FXCM|57=U100D1|1=01958448|60=20151111-21:01:12.400|263=1|581=6|710=5|715=20151111|724=1|9012=20150311|9014=20151112|10=110|
	9043       Closing price
	730         Open price
	9052       PnL for the closed position
	9053       Commission 
	9040       Interest  fee associated whit this position
	9038       used margin

	8=FIX.4.4|9=702|35=AP|34=20|49=FXCM|50=U100D1|52=20151111-21:01:11.936|56=d101968168_client1|1=01958448|11=FIX.4.4:d101968168_client1->FXCM/U100D1-1428599035518-4|15=EUR|37=202027586|55=EUR/USD|60=20150519-03:30:43|325=N|336=FXCM|526=fix_example_test|581=6|625=U100D1|710=5|715=20151111|721=3533878441|724=1|727=13|728=0|730=1.06572|731=1|734=0|912=Y|9000=1|9040=-6.08|9041=78911063|9042=20150409-17:03:56|9043=1.12979|9044=20150519-03:30:43|9048=U100D1_16679142D2EE08ABE053142B3C0A452A_05192015032653174913_QCV-127|9049=FXTS|9052=640.7|9053=0.8|9054=204437509|453=1|448=FXCM ID|447=D|452=3|802=4|523=32|803=26|523=d101968168|803=2|523=fix-test112|803=22|523=1958448|803=10|702=1|703=TQ|704=10000|753=1|707=CASH|708=0|10=042|



	High Risk Investment Warning: Trading foreign exchange and/or contracts for differences on margin carries a high level of risk, and may not be suitable for all investors. The possibility exists that you could sustain a loss in excess of your deposited funds and therefore, you should not speculate with capital that you cannot afford to lose. Before deciding to trade the products offered by FXCM you should carefully consider your objectives, financial situation, needs and level of experience. You should be aware of all the risks associated with trading on margin. FXCM provides general advice that does not take into account your objectives, financial situation or needs. The content of this Website must not be construed as personal advice. FXCM recommends you seek advice from a separate financial advisor. 
