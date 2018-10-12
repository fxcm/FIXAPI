# FIX API

FIX API using FIX Protocol 4.4 designed for real-time, custom institutional interface which push up to 250 price update per second (not available on other APIs). It is our fastest and most popular option. You will get full range of trading order types available at FXCM.

In order to establish and maintain FIX connectivity, you must have an application that manages a network connection and which sends/receives FIX messages. An application that does this is referred to as a FIX engine. Today there are numerous commercial FIX engines as well as open-source alternatives, the most known of which is [QuickFIX](http://www.quickfixengine.org/)

FXCM trading session reset weekly, it opens on Sundays between 5:00 PM ET and 5:15 PM ET. and closes on Fridays at 4:55 PM ET

## How to start:
1) FIX quick start guide at [here](https://github.com/fxcm/FIXAPI/blob/master/FIX_quick_start_guide.docx)
2) In order to obtain live access, FXCM requires a live account minimum balance of $5,000 USD.
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

If you want shrink price data which is 60% of current market price, please take a look at [here](https://docs.fxcorporate.com/api-message-info.pdf)

## Placing order:
Please set account number on tag 1, 1=00648329 when you place orders. Otherwise you will get error "No Account specified".

## Suggested FIX Protocol package.
C++ - Windows, Linux, Mac - <a href="http://www.quickfixengine.org/">QuickFix</a>

C# - Windows – <a href="http://quickfixn.org/">QuickFix/N</a>

Java – any OS with Java VM – <a href="http://www.quickfixj.org/">QuickFix/J</a>

## Note:
o	This is for personal use and abides by our [EULA](https://www.fxcm.com/uk/forms/eula/)

o	For more information, you may contact us: api@fxcm.com

## Disclaimer:

Trading forex/CFDs on margin carries a high level of risk and may not be suitable for all investors as you could sustain losses in excess of deposits. Leverage can work against you. The products are intended for retail and professional clients. Due to the certain restrictions imposed by the local law and regulation, German resident retail client(s) could sustain a total loss of deposited funds but are not subject to subsequent payment obligations beyond the deposited funds. Be aware and fully understand all risks associated with the market and trading. Prior to trading any products, carefully consider your financial situation and experience level. If you decide to trade products offered by FXCM Australia Pty. Limited (“FXCM AU”) (AFSL 309763), you must read and understand the [Financial Services Guide](https://docs.fxcorporate.com/financial-services-guide-au.pdf), [Product Disclosure Statement](https://www.fxcm.com/au/legal/product-disclosure-statements/), and [Terms of Business](https://docs.fxcorporate.com/tob_au_en.pdf). Any opinions, news, research, analyses, prices, or other information is provided as general market commentary, and does not constitute investment advice. FXCM will not accept liability for any loss or damage, including without limitation to, any loss of profit, which may arise directly or indirectly from use of or reliance on such information. FXCM will not accept liability for any loss or damage, including without limitation to, any loss of profit, which may arise directly or indirectly from use of or reliance on such information.
