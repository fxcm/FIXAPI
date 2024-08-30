# FIX API
> *Based on FIX designed for institutional traders.*

FIX API using FIX Protocol 4.4 designed for real-time, custom institutional interface which push up to 200 price update per second (not available on other APIs). It is our fastest and most popular solution. You will get full range of trading order types available at FXCM.

In order to establish and maintain FIX connectivity, you must have an application that manages a network connection and which sends/receives FIX messages. An application that does this is referred to as a FIX engine. Today there are numerous commercial FIX engines as well as open-source alternatives, the most known of which is QuickFIX

FXCM trading session reset weekly, it opens on Sundays between 5:00 PM ET and 5:15 PM ET. and closes on Fridays at 4:55 PM ET.

Please refer to our [github page](https://github.com/fxcm/FIXAPI)

## Contents
* [Prerequisites](#Prerequisites)
* [Connecting](#Connecting)
* [Marketdata](#Marketdata)
* [Table](#Table)
* [Order](#Order)
* [Sample](#Sample)
* [Disclaimer](#Disclaimer)


## Prerequisites

*	First open a demo TSII account at [here](https://www.fxcm.com/uk/algorithmic-trading/api-trading/)
*	Send your login to api@fxcm.com to get FIX credentials. 
*	Request documentation by signing our [EULA](https://www.fxcm.com/forms/eula/). FXCM data dictionary [FIXFXCM10.xml](https://apiwiki.fxcorporate.com/api/fix/docs/FIXFXCM10.xml)
*	Install software development environment (SDE). For Java, you can try [Eclipse](https://www.eclipse.org/downloads/) or [NetBeans](https://netbeans.org/downloads/),  For C++/C#, please download [Visual studio](https://visualstudio.microsoft.com/downloads/)
*	You also need to download FIX protocol package. [QuickFix/J or QuickFix/N](http://www.quickfixj.org/)


## Connecting
There are two ways to Logon, one is within the Logon message should include your Username(553) and Password(554). The other one is send Username(553) and Password(554) on User Request (35=BE). It is also important to note here that FXCM requires the TargetSubID on all messages, including your Logon.

```
Example Logon Messages
//Send Username/Password on Logon (35=A)
8=FIX.4.4|9=114| 35=A |34=1| 49=fx1294946_client1| 52=20120927-13:15:34.754| 56=FXCM |57=U100D1| 553=fx1294946| 554=123| 98=0 |108=30 |141=Y| 10=146|

//Send Username/Password on User Request(35=BE)
8=FIX.4.4|9=114|35=BE|34=2|49=fx1294946_client1|52=20140515-00:29:11.372|56=FXCM|57=U100D1|553=fx1294946|554=1234|923=1|924=1|10=150|

//FXCM Logon response 
8=FIX.4.4|9=92| 35=A| 34=1| 49=FXCM| 50=U100D1 |52=20120927-13:15:34.810| 56=fx1294946_client1| 98=0| 108=30| 141=Y| 10=187|
```

## Marketdata
*	Subscribe market price with 35=V MarketDataRequest. You will get MarketDataSnapshotFullRefresh(W) 35=W.
*	The MarketDataSnapshotFullRefresh(W) message contains the updates to market data. It is obtained as a response to the MarketDataRequest(V) message. FIX connections are then subscription based for the market data; meaning, you must request it to receive it.

*	The types of data you can receive, such as the Bid price or Offer price, are referred to as MDEntryTypes in FIX. FXCM supports the following MDEntryTypes in each message: Bid(0), Offer(1), High Price(7), and Low Price(8). Additional MDEntryTypes such as MDEntryDate, MDEntryTime, QuoteCondition, etc., are found only once within the first repeating group of the message.

*	Also we don't have liquidity size information and depth information, only display BBO (best bid offer).

*	If you want shrink price data which is 60% of current market price, please take a look at [here](https://apiwiki.fxcorporate.com/api/fix/FIXShorteneMessage.docx)
```
8=FIX.4.4|9=497|35=W|34=4|49=FXCM|50=U100D1|52=20180817-13:44:06.495|56=MD_d101968187_client1|55=EUR/JPY|228=1|231=1|460=4|9001=3|9002=0.01|9005=10|9011=0|9020=0|9080=1|9090=0|9091=0|9092=0|9093=0|9094=50000000|9095=1|9096=O|268=4|269=0|270=126.085|271=0|272=20180817|273=13:44:06.000|336=FXCM|625=PSFX|276=A|282=PSFX_DESK|299=FXCM-EURJPY-19288641|537=1|269=1|270=126.093|271=0|272=20180817|273=13:44:06.000|336=FXCM|625=PSFX|276=A|282=PSFX_DESK|299=FXCM-EURJPY-19288641|537=1|269=7|270=126.448|269=8|270=125.567|10=117|
```
*	If you want subscribe once and get update been pushed whenever there is update, you need set Subscription request type 263=1 which is snapshot + update.  If 0, you will only get one snapshot. 
*	You can also subscribe security in list, instead of just one security. 
```
8=FIX.4.4|9=134|35=V|34=5|49=d101970033_client2|52=20180822-19:20:49.137|56=FXCM|57=U100D1|262=4|263=1|264=3|265=0|146=1|55=EUR/USD|267=2|269=0|269=1|10=189|
8=FIX.4.4|9=808|35=V|34=5|49=MD_d101968187_client1|52=20180817-13:44:05.840|56=FXCM|57=U100D1|262=4|263=1|264=0|265=0|146=65|55=USOil|55=AUD/JPY|55=NZD/CAD|55=EUR/CAD|55=USD/ZAR|55=AUS200|55=UKOil|55=EUR/NOK|55=NGAS|55=EUR/AUD|55=USD/HKD|55=EUSTX50|55=GBP/CAD|55=USD/CAD|55=GER30|55=CAD/CHF|55=USD/TRY|55=EUR/TRY|55=Copper|55=HKG33|55=USOilF2|55=GBP/AUD|55=NAS100|55=EUR/CHF|55=TRY/JPY|55=AUD/NZD|55=USD/CHF|55=XAU/USD|55=FRA40|55=USOilF|55=AUD/USD|55=NZD/JPY|55=USD/MXN|55=USDOLLAR|55=CHN50|55=ESP35|55=EUR/NZD|55=UKOilF|55=ZAR/JPY|55=GBP/CHF|55=NZD/USD|55=USD/JPY|55=GBP/NZD|55=SPX500|55=CHF/JPY|55=UK100|55=EUR/USD|55=SOYF|55=GBP/USD|55=EUR/JPY|55=AUD/CHF|55=EUR/GBP|55=XAG/USD|55=US30|55=GBP/JPY|55=NZD/CHF|55=USD/NOK|55=CAD/JPY|55=AUD/CAD|55=Bund|55=USD/SEK|55=EUR/SEK|55=USD/CNH|55=JPN225|55=UKOilF2|267=2|269=0|269=1|10=004|
```

## Table
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

## Order

*	Please set account number on tag 1, 1=00648329 when you place orders. Otherwise you will get error "No Account specified".
*	Place market order via 35=D. you will get execution report in 35=8

```
Open market position:
20160411-06:16:50.909 : 8=FIX.4.4 9=163 35=D 34=7 49=D101546502001_client1 52=20160411-06:16:50.909 56=FXCM 57=U100D1 1=01537581 11=635959630109097564 38=10 40=1 54=1 55=SPX500 59=1 60=20160411-06:16:50 10=054

Sample execution report 35=8
20160411-06:16:51.399 : 8=FIX.4.4 9=478 35=8 34=15 49=FXCM 50=U100D1 52=20160411-06:16:51.177 56=D101546502001_client1 1=01537581 6=2047.53 11=635959630109097564 14=10 15=USD 17=821172034 31=2047.53 32=10 37=225909074 38=10 39=2 40=1 44=2047.53 54=1 55=SPX500 58=Executed 59=1 60=20160411-06:16:51 99=0 150=F 151=0 211=0 336=FXCM 625=U100D1 835=0 836=0 1094=0 9000=1010 9041=89603919 9050=OM 9051=F 9061=0 453=1 448=FXCM ID 447=D 452=3 802=4 523=1537581 803=10 523=d101546502001 803=2 523=Halpert 803=22 523=32 803=26 10=088

```

## Limit Order

*	There are two cases of limit order base on TIF and they are different
*	Case 1: TIF tag59 as GTC/GTD limit order: it work like entry order, because order will be executed in the future. If long limit price <= Ask. 

```
DEBUG (2016-01-18 21:15:54,015) [QF/J Session dispatcher: FIX.4.4:FXCM/MINIREAL->1601094176_client2] (app) - <<< app message from counterparty: 8=FIX.4.4|9=182|35=D|34=522|49=1601094176_client2|52=20160119-02:15:54|56=FXCM|57=MINIREAL|1=1601094176|11=F029d160118t211554L0015|38=21000|40=2|44=0.68657|54=1|55=AUD/USD|59=1|60=20160119-02:15:54|10=022|

```

*	Case 2: TIF as IOC/FOK. it is executed immediately at market. for buy limit IOC, limit price should >= Ask

```
DEBUG (2016-01-18 23:15:54,053) [QF/J Session dispatcher: FIX.4.4:FXCM/MINIREAL->1601094176_client2] (app) - <<< app message from counterparty: 8=FIX.4.4|9=181|35=D|34=804|49=1601094176_client2|52=20160119-04:15:54|56=FXCM|57=MINIREAL|1=1601094176|11=F085d160118t231554L0044|38=7000|40=2|44=1.45063|54=1|55=USD/CAD|59=3|60=20160119-04:15:54|10=217|

```

## Sample
*	Sample programs in C++/C#/Java are [here](https://github.com/fxcm/FIXAPI/tree/master/Sample%20Projects)

## Performance improvment release 
We did some performance improvement and released to Demo. It should transparent to API users.

However, you are welcome to test your current setting on Demo and contact api@fxcm.com if you experience any issues.

If everything goes well, we plan to release to Production by the end of next week. Dec 17 2022.

## Disclaimer:

Stratos Group is a holding company of Stratos Markets Limited, Stratos Europe Limited, Stratos Trading Pty. Limited, Stratos South Africa (Pty) Ltd, Stratos Global LLC and all affiliates of aforementioned firms, or other firms under the Stratos Group of companies (collectively "Stratos Group").
The Stratos Group is headquartered at 20 Gresham Street, 4th Floor, London EC2V 7JE, United Kingdom. Stratos Markets Limited is authorised and regulated in the UK by the Financial Conduct Authority. Registration number 217689. Registered in England and Wales with Companies House company number 04072877. Stratos Europe Limited (trading as "FXCM"), is a Cyprus Investment Firm ("CIF") registered with the Cyprus Department of Registrar of Companies (HE 405643) and authorised and regulated by the Cyprus Securities and Exchange Commission ("CySEC") under license number 392/20. Stratos Trading Pty. Limited (trading as "FXCM") (AFSL 309763, ABN 31 121 934 432) is regulated by the Australian Securities and Investments Commission. The information provided by FXCM is intended for residents of Australia and is not directed at any person in any country or jurisdiction where such distribution or use would be contrary to local law or regulation. Please read the full Terms and Conditions. Stratos South Africa (Pty) Ltd is an authorized Financial Services Provider and is regulated by the Financial Sector Conduct Authority under registration number 46534. Stratos Global LLC ("FXCM") is incorporated in St Vincent and the Grenadines with company registration No. 1776 LLC 2022 and is an operating subsidiary within the Stratos Group. FXCM is not required to hold any financial services license or authorization in St Vincent and the Grenadines to offer its products and services. Stratos Global Services, LLC is an operating subsidiary within the Stratos Group. Stratos Global Services, LLC is not regulated and not subject to regulatory oversight.
Stratos Markets Limited: CFDs are complex instruments and come with a high risk of losing money rapidly due to leverage. 62% of retail investor accounts lose money when trading CFDs with this provider. You should consider whether you understand how CFDs work and whether you can afford to take the high risk of losing your money.
Stratos Europe Limited: CFDs are complex instruments and come with a high risk of losing money rapidly due to leverage. 59% of retail investor accounts lose money when trading CFDs with this provider. You should consider whether you understand how CFDs work and whether you can afford to take the high risk of losing your money.
Stratos Trading Pty. Limited: Trading CFDs on margin and margin FX carries a high level of risk, and may not be suitable for all investors. Retail clients could sustain a total loss of the deposited funds, but wholesale clients could sustain losses in excess of deposits. Trading CFDs on margin and margin FX does not give you any entitlements or rights to the underlying instruments.
Stratos Global LLC: Our products are traded on leverage which means they carry a high level of risk and you could lose more than your deposits. These products are not suitable for all investors. Please ensure you fully understand the risks and carefully consider your financial situation and trading experience before trading. Seek independent advice if necessary.
Past Performance: Past Performance is not an indicator of future results.
