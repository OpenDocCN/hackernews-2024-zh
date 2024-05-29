<!--yml
category: 未分类
date: 2024-05-27 15:04:53
-->

# ads.txt - Wikipedia

> 来源：[https://en.wikipedia.org/wiki/Ads.txt](https://en.wikipedia.org/wiki/Ads.txt)

Text file format used in online advertising management

**ads.txt** (**Authorized Digital Sellers**) is an initiative from [IAB](/wiki/Interactive_Advertising_Bureau "Interactive Advertising Bureau") Technology Laboratory. It specifies a [text file](/wiki/Text_file "Text file") that companies can host on their [web servers](/wiki/Web_server "Web server"), listing the other companies authorized to sell their products or services. This is designed to allow online buyers to check the validity of the sellers from whom they buy, for the purposes of [internet fraud prevention](/wiki/Internet_fraud_prevention "Internet fraud prevention").

## State of adoption[[edit](/w/index.php?title=Ads.txt&action=edit&section=1 "Edit section: State of adoption")]

By November 2017, more than 44% of publishers had `ads.txt` files.^([[1]](#cite_note-Ad_Ops_Insider-1)) More than 90,000 sites were using `ads.txt`, up from 3,500 in September 2017, according to Pixalate. Among the top 1,000 sites that sold programmatic ads, 57 percent had `ads.txt` files, compared to 16 percent in September, per Pixalate.^([[2]](#cite_note-Digiday-2))

Latest adoption data per FirstImpression.io's Ads.txt Industry Dashboard:^([[3]](#cite_note-3))

| Websites Tier | Jan 30th, 2020 | Jan 30th, 2019 | Jan 30th, 2018 |
| Alexa Top 1,000 | 44.20% | 40.90% | 33.70% |
| Alexa Top 5,000 | 39.58% | 36.00% | 29.06% |
| Alexa Top 10,000 | 36.66% | 32.75% | 25.18% |
| Alexa Top 30,000 | 31.71% | 26.34% | 18.52% |

[Google](/wiki/Google "Google") has been an active proponent of `ads.txt` and pushing for faster, widespread adoption by publishers.^([[4]](#cite_note-4)) From the end of October 2017 Google Display & Video 360 only buys inventory from sources identified as authorized sellers in a publisher's `ads.txt` file, when a file is available. `ads.txt` may become a requirement for Display & Video 360.^([[5]](#cite_note-MarTechToday-5))

## File format[[edit](/w/index.php?title=Ads.txt&action=edit&section=2 "Edit section: File format")]

The IAB's ads.txt specification^([[6]](#cite_note-IABTechLab-6)) dictates the formatting of ads.txt files, which can contain three types of record; data records, variables and comments. An ads.txt file can include any number of records, each placed on their own line.

Since the ads.txt file format must be adhered to, a range of validation,^([[7]](#cite_note-IABTechLab_Resources-7)) management and collaboration tools have become available to help ensure ads.txt files are created correctly.

The original specification, v1.0, was issued in 2017.^([[6]](#cite_note-IABTechLab-6)) with version 1.0.1 making small changes based on community feedback later that year. In 2019, version 1.0.2 made further small amendments and recommended using a placeholder record to indicate the intent of an empty ads.txt file:^([[8]](#cite_note-IABTechLab-spec-8)) `placeholder.example.com, placeholder, DIRECT, placeholder`.

In 2021, version 1.0.3 added the `inventorypartnerdomain` directive, with version 1.1 adding `managerdomain` and `ownerdomain` in 2022.

## See also[[edit](/w/index.php?title=Ads.txt&action=edit&section=3 "Edit section: See also")]

## References[[edit](/w/index.php?title=Ads.txt&action=edit&section=4 "Edit section: References")]

## External links[[edit](/w/index.php?title=Ads.txt&action=edit&section=5 "Edit section: External links")]