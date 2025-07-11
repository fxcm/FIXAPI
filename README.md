## FXCM FIX API Overview
The FXCM FIX API, built on the Financial Information eXchange (FIX) Protocol 4.4, is a high-performance, real-time solution tailored for institutional clients. It supports up to 200 price updates per second, offering unmatched speed and reliability compared to other APIs. This API provides access to FXCM's full suite of trading order types, making it our most advanced and widely adopted connectivity option.

### Key Features
*	High-Speed Performance: Delivers up to 200 price updates per second for real-time market data.
*	Comprehensive Order Types: Access to all FXCM trading order types, enabling flexible and sophisticated trading strategies.
*	Industry-Standard Protocol: Utilizes FIX Protocol 4.4, ensuring compatibility with institutional trading systems.
*	Robust and Scalable: Designed to meet the demanding requirements of high-frequency trading environments.

### Connectivity Requirements
*	To establish and maintain FIX connectivity, clients must deploy an application capable of managing network connections and sending/receiving FIX messages. This application, commonly known as a FIX engine, can be a commercial solution or an open-source option, such as QuickFIX, a widely recognized and reliable choice.
*	Trading Session Schedule
  FXCM trading sessions reset weekly, with the following schedule:
  Opening: Sundays between 5:00 PM and 5:15 PM Eastern Time (ET).
  Closing: Fridays at 4:55 PM ET.

### Getting Started
*	To integrate the FXCM FIX API, ensure your FIX engine is configured to support FIX Protocol 4.4 and establish a secure network connection with FXCM's servers. For detailed setup instructions and technical specifications, please refer to FXCM's official API documentation or contact our support team.
*	First open a demo TSII account at [here](https://www.fxcm.com/uk/algorithmic-trading/api-trading/)
*	Send your login to api@fxcm.com to get FIX credentials. 
*	Request documentation by signing our [EULA](https://www.fxcm.com/forms/eula/). FXCM data dictionary [FIXFXCM10.xml](https://apiwiki.fxcorporate.com/api/fix/docs/FIXFXCM10.xml)
*	Install software development environment (SDE). For Java, you can try [Eclipse](https://www.eclipse.org/downloads/) or [NetBeans](https://netbeans.org/downloads/),  For C++/C#, please download [Visual studio](https://visualstudio.microsoft.com/downloads/)
*	You also need to download FIX protocol package. [QuickFix/J or QuickFix/N](http://www.quickfixj.org/)

## Performance improvment release 
*	We did some performance improvement and released to Demo. It should transparent to API users.
*	However, you are welcome to test your current setting on Demo and contact api@fxcm.com if you experience any issues.
*	If everything goes well, we plan to release to Production by the end of next week. Dec 17 2022.

## Disclaimer:
Stratos Group is a holding company of Stratos Markets Limited, Stratos Europe Limited, Stratos Trading Pty. Limited, Stratos South Africa (Pty) Ltd, Stratos Global LLC and all affiliates of aforementioned firms, or other firms under the Stratos Group of companies (collectively "Stratos Group").
The Stratos Group is headquartered at 20 Gresham Street, 4th Floor, London EC2V 7JE, United Kingdom. Stratos Markets Limited is authorised and regulated in the UK by the Financial Conduct Authority. Registration number 217689. Registered in England and Wales with Companies House company number 04072877. Stratos Europe Limited (trading as "FXCM"), is a Cyprus Investment Firm ("CIF") registered with the Cyprus Department of Registrar of Companies (HE 405643) and authorised and regulated by the Cyprus Securities and Exchange Commission ("CySEC") under license number 392/20. Stratos Trading Pty. Limited (trading as "FXCM") (AFSL 309763, ABN 31 121 934 432) is regulated by the Australian Securities and Investments Commission. The information provided by FXCM is intended for residents of Australia and is not directed at any person in any country or jurisdiction where such distribution or use would be contrary to local law or regulation. Please read the full Terms and Conditions. Stratos South Africa (Pty) Ltd is an authorized Financial Services Provider and is regulated by the Financial Sector Conduct Authority under registration number 46534. Stratos Global LLC ("FXCM") is incorporated in St Vincent and the Grenadines with company registration No. 1776 LLC 2022 and is an operating subsidiary within the Stratos Group. FXCM is not required to hold any financial services license or authorization in St Vincent and the Grenadines to offer its products and services. Stratos Global Services, LLC is an operating subsidiary within the Stratos Group. Stratos Global Services, LLC is not regulated and not subject to regulatory oversight.
Stratos Markets Limited: CFDs are complex instruments and come with a high risk of losing money rapidly due to leverage. 65% of retail investor accounts lose money when trading CFDs with this provider. You should consider whether you understand how CFDs work and whether you can afford to take the high risk of losing your money.
Stratos Europe Limited: CFDs are complex instruments and come with a high risk of losing money rapidly due to leverage. 66% of retail investor accounts lose money when trading CFDs with this provider. You should consider whether you understand how CFDs work and whether you can afford to take the high risk of losing your money.
Stratos Trading Pty. Limited: Trading CFDs on margin and margin FX carries a high level of risk, and may not be suitable for all investors. Retail clients could sustain a total loss of the deposited funds, but wholesale clients could sustain losses in excess of deposits. Trading CFDs on margin and margin FX does not give you any entitlements or rights to the underlying instruments.
Stratos Global LLC: Our products are traded on leverage which means they carry a high level of risk and you could lose more than your deposits. These products are not suitable for all investors. Please ensure you fully understand the risks and carefully consider your financial situation and trading experience before trading. Seek independent advice if necessary.
Past Performance: Past Performance is not an indicator of future results.
