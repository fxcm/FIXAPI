# FIX API

FIX API using FIX Protocol 4.4 designed for real-time, custom institutional interface which push up to 250 price update per second (not available on other APIs). It is our fastest and most popular option. You will get full range of trading order types available at FXCM. An FXCM TSII account with a $5,000 minimum balance is required.

In order to establish and maintain FIX connectivity, you must have an application that manages a network connection and which sends/receives FIX messages. An application that does this is referred to as a FIX engine. Today there are numerous commercial FIX engines as well as open-source alternatives, the most known of which is [QuickFIX](http://www.quickfixengine.org/)

FXCM trading session reset weekly, it opens on Sundays between 5:00 PM ET and 5:15 PM ET. and closes on Fridays at 4:55 PM ET

## How to start:
1) A FXCM TSII account. open account at [here](https://www.fxcm.com/uk/algorithmic-trading/api-trading/)
2) In order to obtain live access, FXCM requires a live account minimum balance of $5,000 USD. Please contact [FXCM client support](https://www.fxcm.com/support/contact-client-support/) for requesting FIX credentials and more.
3) You can request documentation by signing our [EULA](https://www.fxcm.com/forms/eula/)
4) FXCM data dictionary [FIXFXCM10.xml](https://apiwiki.fxcorporate.com/api/fix/docs/FIXFXCM10.xml)
5) Sample programs in C++/C#/Java are [here](https://github.com/fxcm/FIXAPI/tree/master/Sample%20Projects)
6) Want to test on demo? please send request to API support api@fxcm.com.

## Getting Connected:
There are two ways to Logon, one is within the Logon message should include your Username(553) and Password(554). The other one is send Username(553) and Password(554) on User Request (35=BE). It is also important to note here that FXCM requires the TargetSubID on all messages, including your Logon. If you are not receiving any responses to your Logon message, it is likely because you have not included TargetSubID

	Example Logon Messages
	
	//Send Username/Password on Logon (35=A)
	8=FIX.4.4|9=114| 35=A |34=1| 49=fx1294946_client1| 52=20120927-13:15:34.754| 56=FXCM |57=U100D1| 553=fx1294946| 554=123| 98=0 |108=30 |141=Y| 10=146|

	//Send Username/Password on User Request(35=BE)
	8=FIX.4.4|9=114|35=BE|34=2|49=fx1294946_client1|52=20140515-00:29:11.372|56=FXCM|57=U100D1|553=fx1294946|554=1234|923=1|924=1|10=150|

	//FXCM Logon response 
	8=FIX.4.4|9=92| 35=A| 34=1| 49=FXCM| 50=U100D1 |52=20120927-13:15:34.810| 56=fx1294946_client1| 98=0| 108=30| 141=Y| 10=187|
  

## Requesting Market Data:
The MarketDataSnapshotFullRefresh(W) message contains the updates to market data. It is obtained as a response to the MarketDataRequest(V) message. FIX connections are then subscription based for the market data; meaning, you must request it to receive it.

The types of data you can receive, such as the Bid price or Offer price, are referred to as MDEntryTypes in FIX. FXCM supports the following MDEntryTypes in each message: Bid(0), Offer(1), High Price(7), and Low Price(8). Additional MDEntryTypes such as MDEntryDate, MDEntryTime, QuoteCondition, etc., are found only once within the first repeating group of the message.

Also we don't have liquidity size information and depth information, only display BBO (best bid offer). 

## Suggested FIX Protocol package.
C++ - Windows, Linux, Mac - <a href="http://www.quickfixengine.org/">QuickFix</a>

C# - Windows – <a href="http://quickfixn.org/">QuickFix/N</a>

Java – any OS with Java VM – <a href="http://www.quickfixj.org/">QuickFix/J</a>

## Note:
o	This is for personal use and abides by our [EULA](https://www.fxcm.com/uk/forms/eula/)

o	For more information, you may contact us: api@fxcm.com
