<!--yml
category: 未分类
date: 2024-05-27 14:59:15
-->

# proxy.c « examples - liburing - io_uring library

> 来源：[https://git.kernel.dk/cgit/liburing/tree/examples/proxy.c](https://git.kernel.dk/cgit/liburing/tree/examples/proxy.c)

```
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136
137
138
139
140
141
142
143
144
145
146
147
148
149
150
151
152
153
154
155
156
157
158
159
160
161
162
163
164
165
166
167
168
169
170
171
172
173
174
175
176
177
178
179
180
181
182
183
184
185
186
187
188
189
190
191
192
193
194
195
196
197
198
199
200
201
202
203
204
205
206
207
208
209
210
211
212
213
214
215
216
217
218
219
220
221
222
223
224
225
226
227
228
229
230
231
232
233
234
235
236
237
238
239
240
241
242
243
244
245
246
247
248
249
250
251
252
253
254
255
256
257
258
259
260
261
262
263
264
265
266
267
268
269
270
271
272
273
274
275
276
277
278
279
280
281
282
283
284
285
286
287
288
289
290
291
292
293
294
295
296
297
298
299
300
301
302
303
304
305
306
307
308
309
310
311
312
313
314
315
316
317
318
319
320
321
322
323
324
325
326
327
328
329
330
331
332
333
334
335
336
337
338
339
340
341
342
343
344
345
346
347
348
349
350
351
352
353
354
355
356
357
358
359
360
361
362
363
364
365
366
367
368
369
370
371
372
373
374
375
376
377
378
379
380
381
382
383
384
385
386
387
388
389
390
391
392
393
394
395
396
397
398
399
400
401
402
403
404
405
406
407
408
409
410
411
412
413
414
415
416
417
418
419
420
421
422
423
424
425
426
427
428
429
430
431
432
433
434
435
436
437
438
439
440
441
442
443
444
445
446
447
448
449
450
451
452
453
454
455
456
457
458
459
460
461
462
463
464
465
466
467
468
469
470
471
472
473
474
475
476
477
478
479
480
481
482
483
484
485
486
487
488
489
490
491
492
493
494
495
496
497
498
499
500
501
502
503
504
505
506
507
508
509
510
511
512
513
514
515
516
517
518
519
520
521
522
523
524
525
526
527
528
529
530
531
532
533
534
535
536
537
538
539
540
541
542
543
544
545
546
547
548
549
550
551
552
553
554
555
556
557
558
559
560
561
562
563
564
565
566
567
568
569
570
571
572
573
574
575
576
577
578
579
580
581
582
583
584
585
586
587
588
589
590
591
592
593
594
595
596
597
598
599
600
601
602
603
604
605
606
607
608
609
610
611
612
613
614
615
616
617
618
619
620
621
622
623
624
625
626
627
628
629
630
631
632
633
634
635
636
637
638
639
640
641
642
643
644
645
646
647
648
649
650
651
652
653
654
655
656
657
658
659
660
661
662
663
664
665
666
667
668
669
670
671
672
673
674
675
676
677
678
679
680
681
682
683
684
685
686
687
688
689
690
691
692
693
694
695
696
697
698
699
700
701
702
703
704
705
706
707
708
709
710
711
712
713
714
715
716
717
718
719
720
721
722
723
724
725
726
727
728
729
730
731
732
733
734
735
736
737
738
739
740
741
742
743
744
745
746
747
748
749
750
751
752
753
754
755
756
757
758
759
760
761
762
763
764
765
766
767
768
769
770
771
772
773
774
775
776
777
778
779
780
781
782
783
784
785
786
787
788
789
790
791
792
793
794
795
796
797
798
799
800
801
802
803
804
805
806
807
808
809
810
811
812
813
814
815
816
817
818
819
820
821
822
823
824
825
826
827
828
829
830
831
832
833
834
835
836
837
838
839
840
841
842
843
844
845
846
847
848
849
850
851
852
853
854
855
856
857
858
859
860
861
862
863
864
865
866
867
868
869
870
871
872
873
874
875
876
877
878
879
880
881
882
883
884
885
886
887
888
889
890
891
892
893
894
895
896
897
898
899
900
901
902
903
904
905
906
907
908
909
910
911
912
913
914
915
916
917
918
919
920
921
922
923
924
925
926
927
928
929
930
931
932
933
934
935
936
937
938
939
940
941
942
943
944
945
946
947
948
949
950
951
952
953
954
955
956
957
958
959
960
961
962
963
964
965
966
967
968
969
970
971
972
973
974
975
976
977
978
979
980
981
982
983
984
985
986
987
988
989
990
991
992
993
994
995
996
997
998
999
1000
1001
1002
1003
1004
1005
1006
1007
1008
1009
1010
1011
1012
1013
1014
1015
1016
1017
1018
1019
1020
1021
1022
1023
1024
1025
1026
1027
1028
1029
1030
1031
1032
1033
1034
1035
1036
1037
1038
1039
1040
1041
1042
1043
1044
1045
1046
1047
1048
1049
1050
1051
1052
1053
1054
1055
1056
1057
1058
1059
1060
1061
1062
1063
1064
1065
1066
1067
1068
1069
1070
1071
1072
1073
1074
1075
1076
1077
1078
1079
1080
1081
1082
1083
1084
1085
1086
1087
1088
1089
1090
1091
1092
1093
1094
1095
1096
1097
1098
1099
1100
1101
1102
1103
1104
1105
1106
1107
1108
1109
1110
1111
1112
1113
1114
1115
1116
1117
1118
1119
1120
1121
1122
1123
1124
1125
1126
1127
1128
1129
1130
1131
1132
1133
1134
1135
1136
1137
1138
1139
1140
1141
1142
1143
1144
1145
1146
1147
1148
1149
1150
1151
1152
1153
1154
1155
1156
1157
1158
1159
1160
1161
1162
1163
1164
1165
1166
1167
1168
1169
1170
1171
1172
1173
1174
1175
1176
1177
1178
1179
1180
1181
1182
1183
1184
1185
1186
1187
1188
1189
1190
1191
1192
1193
1194
1195
1196
1197
1198
1199
1200
1201
1202
1203
1204
1205
1206
1207
1208
1209
1210
1211
1212
1213
1214
1215
1216
1217
1218
1219
1220
1221
1222
1223
1224
1225
1226
1227
1228
1229
1230
1231
1232
1233
1234
1235
1236
1237
1238
1239
1240
1241
1242
1243
1244
1245
1246
1247
1248
1249
1250
1251
1252
1253
1254
1255
1256
1257
1258
1259
1260
1261
1262
1263
1264
1265
1266
1267
1268
1269
1270
1271
1272
1273
1274
1275
1276
1277
1278
1279
1280
1281
1282
1283
1284
1285
1286
1287
1288
1289
1290
1291
1292
1293
1294
1295
1296
1297
1298
1299
1300
1301
1302
1303
1304
1305
1306
1307
1308
1309
1310
1311
1312
1313
1314
1315
1316
1317
1318
1319
1320
1321
1322
1323
1324
1325
1326
1327
1328
1329
1330
1331
1332
1333
1334
1335
1336
1337
1338
1339
1340
1341
1342
1343
1344
1345
1346
1347
1348
1349
1350
1351
1352
1353
1354
1355
1356
1357
1358
1359
1360
1361
1362
1363
1364
1365
1366
1367
1368
1369
1370
1371
1372
1373
1374
1375
1376
1377
1378
1379
1380
1381
1382
1383
1384
1385
1386
1387
1388
1389
1390
1391
1392
1393
1394
1395
1396
1397
1398
1399
1400
1401
1402
1403
1404
1405
1406
1407
1408
1409
1410
1411
1412
1413
1414
1415
1416
1417
1418
1419
1420
1421
1422
1423
1424
1425
1426
1427
1428
1429
1430
1431
1432
1433
1434
1435
1436
1437
1438
1439
1440
1441
1442
1443
1444
1445
1446
1447
1448
1449
1450
1451
1452
1453
1454
1455
1456
1457
1458
1459
1460
1461
1462
1463
1464
1465
1466
1467
1468
1469
1470
1471
1472
1473
1474
1475
1476
1477
1478
1479
1480
1481
1482
1483
1484
1485
1486
1487
1488
1489
1490
1491
1492
1493
1494
1495
1496
1497
1498
1499
1500
1501
1502
1503
1504
1505
1506
1507
1508
1509
1510
1511
1512
1513
1514
1515
1516
1517
1518
1519
1520
1521
1522
1523
1524
1525
1526
1527
1528
1529
1530
1531
1532
1533
1534
1535
1536
1537
1538
1539
1540
1541
1542
1543
1544
1545
1546
1547
1548
1549
1550
1551
1552
1553
1554
1555
1556
1557
1558
1559
1560
1561
1562
1563
1564
1565
1566
1567
1568
1569
1570
1571
1572
1573
1574
1575
1576
1577
1578
1579
1580
1581
1582
1583
1584
1585
1586
1587
1588
1589
1590
1591
1592
1593
1594
1595
1596
1597
1598
1599
1600
1601
1602
1603
1604
1605
1606
1607
1608
1609
1610
1611
1612
1613
1614
1615
1616
1617
1618
1619
1620
1621
1622
1623
1624
1625
1626
1627
1628
1629
1630
1631
1632
1633
1634
1635
1636
1637
1638
1639
1640
1641
1642
1643
1644
1645
1646
1647
1648
1649
1650
1651
1652
1653
1654
1655
1656
1657
1658
1659
1660
1661
1662
1663
1664
1665
1666
1667
1668
1669
1670
1671
1672
1673
1674
1675
1676
1677
1678
1679
1680
1681
1682
1683
1684
1685
1686
1687
1688
1689
1690
1691
1692
1693
1694
1695
1696
1697
1698
1699
1700
1701
1702
1703
1704
1705
1706
1707
1708
1709
1710
1711
1712
1713
1714
1715
1716
1717
1718
1719
1720
1721
1722
1723
1724
1725
1726
1727
1728
1729
1730
1731
1732
1733
1734
1735
1736
1737
1738
1739
1740
1741
1742
1743
1744
1745
1746
1747
1748
1749
1750
1751
1752
1753
1754
1755
1756
1757
1758
1759
1760
1761
1762
1763
1764
1765
1766
1767
1768
1769
1770
1771
1772
1773
1774
1775
1776
1777
1778
1779
1780
1781
1782
1783
1784
1785
1786
1787
1788
1789
1790
1791
1792
1793
1794
1795
1796
1797
1798
1799
1800
1801
1802
1803
1804
1805
1806
1807
1808
1809
1810
1811
1812
1813
1814
1815
1816
1817
1818
1819
1820
1821
1822
1823
1824
1825
1826
1827
1828
1829
1830
1831
1832
1833
1834
1835
1836
1837
1838
1839
1840
1841
1842
1843
1844
1845
1846
1847
1848
1849
1850
1851
1852
1853
1854
1855
1856
1857
1858
1859
1860
1861
1862
1863
1864
1865
1866
1867
1868
1869
1870
1871
1872
1873
1874
1875
1876
1877
1878
1879
1880
1881
1882
1883
1884
1885
1886
1887
1888
1889
1890
1891
1892
1893
1894
1895
1896
1897
1898
1899
1900
1901
1902
1903
1904
1905
1906
1907
1908
1909
1910
1911
1912
1913
1914
1915
1916
1917
1918
1919
1920
1921
1922
1923
1924
1925
1926
1927
1928
1929
1930
1931
1932
1933
1934
1935
1936
1937
1938
1939
1940
1941
1942
1943
1944
1945
1946
1947
1948
1949
1950
1951
1952
1953
1954
1955
1956
1957
1958
1959
1960
1961
1962
1963
1964
1965
1966
1967
1968
1969
1970
1971
1972
1973
1974
1975
1976
1977
1978
1979
1980
1981
1982
1983
1984
1985
1986
1987
1988
1989
1990
1991
1992
1993
1994
1995
1996
1997
1998
1999
2000
2001
2002
2003
2004
2005
2006
2007
2008
2009
2010
2011
2012
2013
2014
2015
2016
2017
2018
2019
2020
2021
2022
2023
2024
2025
2026
2027
2028
2029
2030
2031
2032
2033
2034
2035
2036
2037
2038
2039
2040
2041
2042
2043
2044
2045
2046
2047
2048
2049
2050
2051
2052
2053
2054
2055
2056
2057
2058
2059
2060
2061
2062
2063
2064
2065
2066
2067
2068
2069
2070
2071
2072
2073
2074
2075
2076
2077
2078
2079
2080
2081
2082
2083
2084
2085
2086
2087
2088
2089
2090
2091
2092
2093
2094
2095
2096
2097
2098
2099
2100
2101
2102
2103
2104
2105
2106
2107
2108
2109
2110
2111
2112
2113
2114
2115
2116
2117
2118
2119
2120
2121
2122
2123
2124
2125
2126
2127
2128
2129
2130
2131
2132
2133
2134
2135
2136
2137
2138
2139
2140
2141
2142
2143
2144
2145
2146
2147
2148
2149
2150
2151
2152
2153
2154
2155
2156
2157
2158
2159
2160
2161
2162
2163
2164
2165
2166
2167
2168
2169
2170
2171
2172
2173
2174
2175
2176
2177
2178
2179
2180
2181
2182
2183
2184
2185
2186
2187
2188
2189
2190
2191
2192
2193
2194
2195
2196
2197
2198
2199
2200
2201
2202
2203
2204
2205
2206
2207
2208
2209
2210
2211
2212
2213
2214
2215
2216
2217
2218
2219
2220
2221
2222
2223
2224
2225
2226
2227
2228
2229
2230
2231
2232
2233
2234
2235
2236
2237
2238
2239
2240
2241
2242
2243
2244
2245
2246
2247
2248
2249
2250
2251
2252
2253
2254
2255
2256
2257
2258
2259
2260
2261
2262
2263
2264
2265
2266
2267
2268
2269
2270
2271
2272
2273
2274
2275
2276
2277
2278
2279
2280
2281
2282
2283
2284
2285
2286
2287
2288
2289
2290
2291
2292
2293
2294
2295
2296
2297
2298
2299
2300
2301
2302
2303
2304
2305
2306
2307
2308
2309
2310
2311
2312
2313
2314
2315
2316
2317
2318
2319
2320
2321
2322
2323
2324
2325
2326
2327
2328
2329
2330
2331
2332
2333
2334
2335
2336
2337
2338
2339
2340
2341
2342
2343
2344
2345
2346
2347
2348
2349
2350
2351
2352
2353
2354
2355
2356
2357
2358
2359
2360
2361
2362
2363
2364
2365
2366
2367
2368
2369
2370
2371
2372
2373
2374
2375
2376
2377
2378
2379
2380
2381
2382
2383
2384
2385
2386
2387
2388
2389
2390
2391
2392
2393
2394
2395
2396
2397
2398
2399
2400
2401
2402
2403
2404
2405
2406
2407
2408
2409
2410

```

```
/* SPDX-License-Identifier: MIT */
/*
 * Sample program that can act either as a packet sink, where it just receives
 * packets and doesn't do anything with them, or it can act as a proxy where it
 * receives packets and then sends them to a new destination. The proxy can
 * be unidirectional (-B0), or bi-direction (-B1).
 * 
 * Examples:
 *
 * Act as a proxy, listening on port 4444, and send data to 192.168.2.6 on port
 * 4445\. Use multishot receive, DEFER_TASKRUN, and fixed files
 *
 * 	./proxy -m1 -r4444 -H 192.168.2.6 -p4445
 *
 * Same as above, but utilize send bundles (-C1, requires -u1 send_ring) as well
 * with ring provided send buffers, and recv bundles (-c1).
 *
 * 	./proxy -m1 -c1 -u1 -C1 -r4444 -H 192.168.2.6 -p4445
 *
 * Act as a bi-directional proxy, listening on port 8888, and send data back
 * and forth between host and 192.168.2.6 on port 22\. Use multishot receive,
 * DEFER_TASKRUN, fixed files, and buffers of size 1500.
 *
 * 	./proxy -m1 -B1 -b1500 -r8888 -H 192.168.2.6 -p22
 *
 * Act a sink, listening on port 4445, using multishot receive, DEFER_TASKRUN,
 * and fixed files:
 *
 * 	./proxy -m1 -s1 -r4445
 *
 * Run with -h to see a list of options, and their defaults.
 *
 * (C) 2024 Jens Axboe <axboe@kernel.dk>
 *
 */
#include <fcntl.h>
#include <stdint.h>
#include <netinet/in.h>
#include <netinet/tcp.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <sys/time.h>
#include <unistd.h>
#include <sys/mman.h>
#include <linux/mman.h>
#include <locale.h>
#include <assert.h>
#include <pthread.h>
#include <liburing.h>

#include "proxy.h"
#include "helpers.h"

/*
 * Will go away once/if bundles are upstreamed and we put the generic
 * definitions in the kernel header.
 */
#ifndef IORING_RECVSEND_BUNDLE
#define IORING_RECVSEND_BUNDLE		(1U << 4)
#endif
#ifndef IORING_FEAT_SEND_BUF_SELECT
#define IORING_FEAT_SEND_BUF_SELECT	(1U << 14)
#endif

static int cur_bgid = 1;
static int nr_conns;
static int open_conns;
static long page_size;

static unsigned long event_loops;
static unsigned long events;

static int recv_mshot = 1;
static int sqpoll;
static int defer_tw = 1;
static int is_sink;
static int fixed_files = 1;
static char *host = "192.168.3.2";
static int send_port = 4445;
static int receive_port = 4444;
static int buf_size = 32;
static int bidi;
static int ipv6;
static int napi;
static int napi_timeout;
static int wait_batch = 1;
static int wait_usec = 1000000;
static int rcv_msg;
static int snd_msg;
static int snd_zc;
static int send_ring = -1;
static int snd_bundle;
static int rcv_bundle;
static int use_huge;
static int ext_stat;
static int verbose;

static int nr_bufs = 256;
static int br_mask;

static int ring_size = 128;

static pthread_mutex_t thread_lock;
static struct timeval last_housekeeping;

/*
 * For sendmsg/recvmsg. recvmsg just has a single vec, sendmsg will have
 * two vecs - one that is currently submitted and being sent, and one that
 * is being prepared. When a new sendmsg is issued, we'll swap which one we
 * use. For send, even though we don't pass in the iovec itself, we use the
 * vec to serialize the sends to avoid reordering.
 */
struct msg_vec {
	struct iovec *iov;
	/* length of allocated vec */
	int vec_size;
	/* length currently being used */
	int iov_len;
	/* only for send, current index we're processing */
	int cur_iov;
};

struct io_msg {
	struct msghdr msg;
	struct msg_vec vecs[2];
	/* current msg_vec being prepared */
	int vec_index;
};

/*
 * Per socket stats per connection. For bi-directional, we'll have both
 * sends and receives on each socket, this helps track them seperately.
 * For sink or one directional, each of the two stats will be only sends
 * or receives, not both.
 */
struct conn_dir {
	int index;

	int pending_shutdown;
	int pending_send;
	int pending_recv;

	int snd_notif;

	int out_buffers;

	int rcv, rcv_shrt, rcv_enobufs, rcv_mshot;
	int snd, snd_shrt, snd_enobufs, snd_busy, snd_mshot;

	int snd_next_bid;
	int rcv_next_bid;

	int *rcv_bucket;
	int *snd_bucket;

	unsigned long in_bytes, out_bytes;

	/* only ever have a single recv pending */
	struct io_msg io_rcv_msg;

	/* one send that is inflight, and one being prepared for the next one */
	struct io_msg io_snd_msg;
};

enum {
	CONN_F_STARTED		= 1,
	CONN_F_DISCONNECTING	= 2,
	CONN_F_DISCONNECTED	= 4,
	CONN_F_PENDING_SHUTDOWN	= 8,
	CONN_F_STATS_SHOWN	= 16,
	CONN_F_END_TIME		= 32,
	CONN_F_REAPED		= 64,
};

/*
 * buffer ring belonging to a connection
 */
struct conn_buf_ring {
	struct io_uring_buf_ring *br;
	void *buf;
	int bgid;
};

struct conn {
	struct io_uring ring;

	/* receive side buffer ring, new data arrives here */
	struct conn_buf_ring in_br;
	/* if send_ring is used, outgoing data to send */
	struct conn_buf_ring out_br;

	int tid;
	int in_fd, out_fd;
	int pending_cancels;
	int flags;

	struct conn_dir cd[2];

	struct timeval start_time, end_time;

	union {
		struct sockaddr_in addr;
		struct sockaddr_in6 addr6;
	};

	pthread_t thread;
	pthread_barrier_t startup_barrier;
};

#define MAX_CONNS	1024
static struct conn conns[MAX_CONNS];

#define vlog(str, ...) do {						\
	if (verbose)							\
		printf(str, ##__VA_ARGS__);				\
} while (0)

static int prep_next_send(struct io_uring *ring, struct conn *c,
			  struct conn_dir *cd, int fd);
static void *thread_main(void *data);

static struct conn *cqe_to_conn(struct io_uring_cqe *cqe)
{
	struct userdata ud = { .val = cqe->user_data };

	return &conns[ud.op_tid & TID_MASK];
}

static struct conn_dir *cqe_to_conn_dir(struct conn *c,
					struct io_uring_cqe *cqe)
{
	int fd = cqe_to_fd(cqe);

	return &c->cd[fd != c->in_fd];
}

static int other_dir_fd(struct conn *c, int fd)
{
	if (c->in_fd == fd)
		return c->out_fd;
	return c->in_fd;
}

/* currently active msg_vec */
static struct msg_vec *msg_vec(struct io_msg *imsg)
{
	return &imsg->vecs[imsg->vec_index];
}

static struct msg_vec *snd_msg_vec(struct conn_dir *cd)
{
	return msg_vec(&cd->io_snd_msg);
}

/*
 * Goes from accept new connection -> create socket, connect to end
 * point, prepare recv, on receive do send (unless sink). If either ends
 * disconnects, we transition to shutdown and then close.
 */
enum {
	__ACCEPT	= 1,
	__SOCK		= 2,
	__CONNECT	= 3,
	__RECV		= 4,
	__RECVMSG	= 5,
	__SEND		= 6,
	__SENDMSG	= 7,
	__SHUTDOWN	= 8,
	__CANCEL	= 9,
	__CLOSE		= 10,
	__FD_PASS	= 11,
	__NOP		= 12,
	__STOP		= 13,
};

struct error_handler {
	const char *name;
	int (*error_fn)(struct error_handler *, struct io_uring *, struct io_uring_cqe *);
};

static int recv_error(struct error_handler *err, struct io_uring *ring,
		      struct io_uring_cqe *cqe);
static int send_error(struct error_handler *err, struct io_uring *ring,
		      struct io_uring_cqe *cqe);

static int default_error(struct error_handler *err,
			 struct io_uring __attribute__((__unused__)) *ring,
			 struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);

	fprintf(stderr, "%d: %s error %s\n", c->tid, err->name, strerror(-cqe->res));
	fprintf(stderr, "fd=%d, bid=%d\n", cqe_to_fd(cqe), cqe_to_bid(cqe));
	return 1;
}

/*
 * Move error handling out of the normal handling path, cleanly seperating
 * them. If an opcode doesn't need any error handling, set it to NULL. If
 * it wants to stop the connection at that point and not do anything else,
 * then the default handler can be used. Only receive has proper error
 * handling, as we can get -ENOBUFS which is not a fatal condition. It just
 * means we need to wait on buffer replenishing before re-arming the receive.
 */
static struct error_handler error_handlers[] = {
	{ .name = "NULL",	.error_fn = NULL, },
	{ .name = "ACCEPT",	.error_fn = default_error, },
	{ .name = "SOCK",	.error_fn = default_error, },
	{ .name = "CONNECT",	.error_fn = default_error, },
	{ .name = "RECV",	.error_fn = recv_error, },
	{ .name = "RECVMSG",	.error_fn = recv_error, },
	{ .name = "SEND",	.error_fn = send_error, },
	{ .name = "SENDMSG",	.error_fn = send_error, },
	{ .name = "SHUTDOWN",	.error_fn = NULL, },
	{ .name = "CANCEL",	.error_fn = NULL, },
	{ .name = "CLOSE",	.error_fn = NULL, },
	{ .name = "FD_PASS",	.error_fn = default_error, },
	{ .name = "NOP",	.error_fn = NULL, },
	{ .name = "STOP",	.error_fn = default_error, },
};

static void free_buffer_ring(struct io_uring *ring, struct conn_buf_ring *cbr)
{
	if (!cbr->br)
		return;

	io_uring_free_buf_ring(ring, cbr->br, nr_bufs, cbr->bgid);
	cbr->br = NULL;
	if (use_huge)
		munmap(cbr->buf, buf_size * nr_bufs);
	else
		free(cbr->buf);
}

static void free_buffer_rings(struct io_uring *ring, struct conn *c)
{
	free_buffer_ring(ring, &c->in_br);
	free_buffer_ring(ring, &c->out_br);
}

/*
 * Setup a ring provided buffer ring for each connection. If we get -ENOBUFS
 * on receive, for multishot receive we'll wait for half the provided buffers
 * to be returned by pending sends, then re-arm the multishot receive. If
 * this happens too frequently (see enobufs= stat), then the ring size is
 * likely too small. Use -nXX to make it bigger. See recv_enobufs().
 *
 * The alternative here would be to use the older style provided buffers,
 * where you simply setup a buffer group and use SQEs with
 * io_urign_prep_provide_buffers() to add to the pool. But that approach is
 * slower and has been deprecated by using the faster ring provided buffers.
 */
static int setup_recv_ring(struct io_uring *ring, struct conn *c)
{
	struct conn_buf_ring *cbr = &c->in_br;
	int ret, i;
	size_t len;
	void *ptr;

	len = buf_size * nr_bufs;
	if (use_huge) {
		cbr->buf = mmap(NULL, len, PROT_READ|PROT_WRITE,
				MAP_PRIVATE|MAP_HUGETLB|MAP_HUGE_2MB|MAP_ANONYMOUS,
				-1, 0);
		if (cbr->buf == MAP_FAILED) {
			perror("mmap");
			return 1;
		}
	} else {
		if (posix_memalign(&cbr->buf, page_size, len)) {
			perror("posix memalign");
			return 1;
		}
	}
	cbr->br = io_uring_setup_buf_ring(ring, nr_bufs, cbr->bgid, 0, &ret);
	if (!cbr->br) {
		fprintf(stderr, "Buffer ring register failed %d\n", ret);
		return 1;
	}

	ptr = cbr->buf;
	for (i = 0; i < nr_bufs; i++) {
		vlog("%d: add bid %d, data %p\n", c->tid, i, ptr);
		io_uring_buf_ring_add(cbr->br, ptr, buf_size, i, br_mask, i);
		ptr += buf_size;
	}
	io_uring_buf_ring_advance(cbr->br, nr_bufs);
	printf("%d: recv buffer ring bgid %d, bufs %d\n", c->tid, cbr->bgid, nr_bufs);
	return 0;
}

/*
 * If 'send_ring' is used and the kernel supports it, we can skip serializing
 * sends as the data will be ordered regardless. This reduces the send handling
 * complexity, as buffers can always be added to the outgoing ring and will be
 * processed in the order in which they were added.
 */
static int setup_send_ring(struct io_uring *ring, struct conn *c)
{
	struct conn_buf_ring *cbr = &c->out_br;
	int ret;

	cbr->br = io_uring_setup_buf_ring(ring, nr_bufs, cbr->bgid, 0, &ret);
	if (!cbr->br) {
		fprintf(stderr, "Buffer ring register failed %d\n", ret);
		return 1;
	}

	printf("%d: send buffer ring bgid %d, bufs %d\n", c->tid, cbr->bgid, nr_bufs);
	return 0;
}

static int setup_send_zc(struct io_uring *ring, struct conn *c)
{
	struct iovec *iovs;
	void *buf;
	int i, ret;

	if (snd_msg)
		return 0;

	buf = c->in_br.buf;
	iovs = calloc(nr_bufs, sizeof(struct iovec));
	for (i = 0; i < nr_bufs; i++) {
		iovs[i].iov_base = buf;
		iovs[i].iov_len = buf_size;
		buf += buf_size;
	}

	ret = io_uring_register_buffers(ring, iovs, nr_bufs);
	if (ret) {
		fprintf(stderr, "failed registering buffers: %d\n", ret);
		free(iovs);
		return ret;
	}
	free(iovs);
	return 0;
}

/*
 * Setup an input and output buffer ring.
 */
static int setup_buffer_rings(struct io_uring *ring, struct conn *c)
{
	int ret;

	/* no locking needed on cur_bgid, parent serializes setup */
	c->in_br.bgid = cur_bgid++;
	c->out_br.bgid = cur_bgid++;
	c->out_br.br = NULL;

	ret = setup_recv_ring(ring, c);
	if (ret)
		return ret;
	if (is_sink)
		return 0;
	if (snd_zc) {
		ret = setup_send_zc(ring, c);
		if (ret)
			return ret;
	}
	if (send_ring) {
		ret = setup_send_ring(ring, c);
		if (ret) {
			free_buffer_ring(ring, &c->in_br);
			return ret;
		}
	}

	return 0;
}

static void show_buckets(struct conn_dir *cd)
{
	int i;

	if (!cd->rcv_bucket || !cd->snd_bucket)
		return;

	printf("\t Packets per recv/send:\n");
	for (i = 0; i < nr_bufs; i++) {
		if (!cd->rcv_bucket[i] && !cd->snd_bucket[i])
			continue;
		printf("\t bucket(%3d): rcv=%u snd=%u\n", i, cd->rcv_bucket[i],
							     cd->snd_bucket[i]);
	}
}

static void __show_stats(struct conn *c)
{
	unsigned long msec, qps;
	unsigned long bytes, bw;
	struct conn_dir *cd;
	int i;

	if (c->flags & (CONN_F_STATS_SHOWN | CONN_F_REAPED))
		return;
	if (!(c->flags & CONN_F_STARTED))
		return;

	if (!(c->flags & CONN_F_END_TIME))
		gettimeofday(&c->end_time, NULL);

	msec = (c->end_time.tv_sec - c->start_time.tv_sec) * 1000;
	msec += (c->end_time.tv_usec - c->start_time.tv_usec) / 1000;

	qps = 0;
	for (i = 0; i < 2; i++)
		qps += c->cd[i].rcv + c->cd[i].snd;

	if (!qps)
		return;

	if (msec)
		qps = (qps * 1000) / msec;

	printf("Conn %d/(in_fd=%d, out_fd=%d): qps=%lu, msec=%lu\n", c->tid,
					c->in_fd, c->out_fd, qps, msec);

	bytes = 0;
	for (i = 0; i < 2; i++) {
		cd = &c->cd[i];

		if (!cd->in_bytes && !cd->out_bytes && !cd->snd && !cd->rcv)
			continue;

		bytes += cd->in_bytes;
		bytes += cd->out_bytes;

		printf("\t%3d: rcv=%u (short=%u, enobufs=%d), snd=%u (short=%u,"
			" busy=%u, enobufs=%d)\n", i, cd->rcv, cd->rcv_shrt,
			cd->rcv_enobufs, cd->snd, cd->snd_shrt, cd->snd_busy,
			cd->snd_enobufs);
		printf("\t   : in_bytes=%lu (Kb %lu), out_bytes=%lu (Kb %lu)\n",
			cd->in_bytes, cd->in_bytes >> 10,
			cd->out_bytes, cd->out_bytes >> 10);
		printf("\t   : mshot_rcv=%d, mshot_snd=%d\n", cd->rcv_mshot,
			cd->snd_mshot);
		show_buckets(cd);

	}
	if (msec) {
		bytes *= 8UL;
		bw = bytes / 1000;
		bw /= msec;
		printf("\tBW=%'luMbit\n", bw);
	}

	c->flags |= CONN_F_STATS_SHOWN;
}

static void show_stats(void)
{
	float events_per_loop = 0.0;
	static int stats_shown;
	int i;

	if (stats_shown)
		return;

	if (events)
		events_per_loop = (float) events / (float) event_loops;

	printf("Event loops: %lu, events %lu, events per loop %.2f\n", event_loops,
							events, events_per_loop);

	for (i = 0; i < MAX_CONNS; i++) {
		struct conn *c = &conns[i];

		__show_stats(c);
	}
	stats_shown = 1;
}

static void sig_int(int __attribute__((__unused__)) sig)
{
	printf("\n");
	show_stats();
	exit(1);
}

/*
 * Special cased for SQPOLL only, as we don't control when SQEs are consumed if
 * that is used. Hence we may need to wait for the SQPOLL thread to keep up
 * until we can get a new SQE. All other cases will break immediately, with a
 * fresh SQE.
 *
 * If we grossly undersized our SQ ring, getting a NULL sqe can happen even
 * for the !SQPOLL case if we're handling a lot of CQEs in our event loop
 * and multishot isn't used. We can do io_uring_submit() to flush what we
 * have here. Only caveat here is that if linked requests are used, SQEs
 * would need to be allocated upfront as a link chain is only valid within
 * a single submission cycle.
 */
static struct io_uring_sqe *get_sqe(struct io_uring *ring)
{
	struct io_uring_sqe *sqe;

	do {
		sqe = io_uring_get_sqe(ring);
		if (sqe)
			break;
		if (!sqpoll)
			io_uring_submit(ring);
		else
			io_uring_sqring_wait(ring);
	} while (1);

	return sqe;
}

/*
 * See __encode_userdata() for how we encode sqe->user_data, which is passed
 * back as cqe->user_data at completion time.
 */
static void encode_userdata(struct io_uring_sqe *sqe, struct conn *c, int op,
			    int bid, int fd)
{
	__encode_userdata(sqe, c->tid, op, bid, fd);
}

static void __submit_receive(struct io_uring *ring, struct conn *c,
			     struct conn_dir *cd, int fd)
{
	struct conn_buf_ring *cbr = &c->in_br;
	struct io_uring_sqe *sqe;

	vlog("%d: submit receive fd=%d\n", c->tid, fd);

	assert(!cd->pending_recv);
	cd->pending_recv = 1;

	/*
	 * For both recv and multishot receive, we use the ring provided
	 * buffers. These are handed to the application ahead of time, and
	 * are consumed when a receive triggers. Note that the address and
	 * length of the receive are set to NULL/0, and we assign the
	 * sqe->buf_group to tell the kernel which buffer group ID to pick
	 * a buffer from. Finally, IOSQE_BUFFER_SELECT is set to tell the
	 * kernel that we want a buffer picked for this request, we are not
	 * passing one in with the request.
	 */
	sqe = get_sqe(ring);
	if (rcv_msg) {
		struct io_msg *imsg = &cd->io_rcv_msg;
		struct msghdr *msg = &imsg->msg;

		memset(msg, 0, sizeof(*msg));
		msg->msg_iov = msg_vec(imsg)->iov;
		msg->msg_iovlen = msg_vec(imsg)->iov_len;

		if (recv_mshot) {
			cd->rcv_mshot++;
			io_uring_prep_recvmsg_multishot(sqe, fd, &imsg->msg, 0);
		} else {
			io_uring_prep_recvmsg(sqe, fd, &imsg->msg, 0);
		}
	} else {
		if (recv_mshot) {
			cd->rcv_mshot++;
			io_uring_prep_recv_multishot(sqe, fd, NULL, 0, 0);
		} else {
			io_uring_prep_recv(sqe, fd, NULL, 0, 0);
		}
	}
	encode_userdata(sqe, c, __RECV, 0, fd);
	sqe->buf_group = cbr->bgid;
	sqe->flags |= IOSQE_BUFFER_SELECT;
	if (fixed_files)
		sqe->flags |= IOSQE_FIXED_FILE;
	if (rcv_bundle)
		sqe->ioprio |= IORING_RECVSEND_BUNDLE;
}

/*
 * One directional just arms receive on our in_fd
 */
static void submit_receive(struct io_uring *ring, struct conn *c)
{
	__submit_receive(ring, c, &c->cd[0], c->in_fd);
}

/*
 * Bi-directional arms receive on both in and out fd
 */
static void submit_bidi_receive(struct io_uring *ring, struct conn *c)
{
	__submit_receive(ring, c, &c->cd[0], c->in_fd);
	__submit_receive(ring, c, &c->cd[1], c->out_fd);
}

/*
 * We hit -ENOBUFS, which means that we ran out of buffers in our current
 * provided buffer group. This can happen if there's an imbalance between the
 * receives coming in and the sends being processed, particularly with multishot
 * receive as they can trigger very quickly. If this happens, defer arming a
 * new receive until we've replenished half of the buffer pool by processing
 * pending sends.
 */
static void recv_enobufs(struct io_uring *ring, struct conn *c,
			 struct conn_dir *cd, int fd)
{
	vlog("%d: enobufs hit\n", c->tid);

	cd->rcv_enobufs++;

	/*
	 * If we're a sink, mark rcv as rearm. If we're not, then mark us as
	 * needing a rearm for receive and send. The completing send will
	 * kick the recv rearm.
	 */
	if (!is_sink) {
		int do_recv_arm = 1;

		if (!cd->pending_send)
			do_recv_arm = !prep_next_send(ring, c, cd, fd);
		if (do_recv_arm)
			__submit_receive(ring, c, &c->cd[0], c->in_fd);
	} else {
		__submit_receive(ring, c, &c->cd[0], c->in_fd);
	}
}

/*
 * Kill this socket - submit a shutdown and link a close to it. We don't
 * care about shutdown status, so mark it as not needing to post a CQE unless
 * it fails.
 */
static void queue_shutdown_close(struct io_uring *ring, struct conn *c, int fd)
{
	struct io_uring_sqe *sqe1, *sqe2;

	/*
	 * On the off chance that we run out of SQEs after the first one,
	 * grab two upfront. This it to prevent our link not working if
	 * get_sqe() ends up doing submissions to free up an SQE, as links
	 * are not valid across separate submissions.
	 */
	sqe1 = get_sqe(ring);
	sqe2 = get_sqe(ring);

	io_uring_prep_shutdown(sqe1, fd, SHUT_RDWR);
	if (fixed_files)
		sqe1->flags |= IOSQE_FIXED_FILE;
	sqe1->flags |= IOSQE_IO_LINK | IOSQE_CQE_SKIP_SUCCESS;
	encode_userdata(sqe1, c, __SHUTDOWN, 0, fd);

	if (fixed_files)
		io_uring_prep_close_direct(sqe2, fd);
	else
		io_uring_prep_close(sqe2, fd);
	encode_userdata(sqe2, c, __CLOSE, 0, fd);
}

/*
 * This connection is going away, queue a cancel for any pending recv, for
 * example, we have pending for this ring. For completeness, we issue a cancel
 * for any request we have pending for both in_fd and out_fd.
 */
static void queue_cancel(struct io_uring *ring, struct conn *c)
{
	struct io_uring_sqe *sqe;
	int flags = 0;

	if (fixed_files)
		flags |= IORING_ASYNC_CANCEL_FD_FIXED;

	sqe = get_sqe(ring);
	io_uring_prep_cancel_fd(sqe, c->in_fd, flags);
	encode_userdata(sqe, c, __CANCEL, 0, c->in_fd);
	c->pending_cancels++;

	if (c->out_fd != -1) {
		sqe = get_sqe(ring);
		io_uring_prep_cancel_fd(sqe, c->out_fd, flags);
		encode_userdata(sqe, c, __CANCEL, 0, c->out_fd);
		c->pending_cancels++;
	}

	io_uring_submit(ring);
}

static int pending_shutdown(struct conn *c)
{
	return c->cd[0].pending_shutdown + c->cd[1].pending_shutdown;
}

static bool should_shutdown(struct conn *c)
{
	int i;

	if (!pending_shutdown(c))
		return false;
	if (is_sink)
		return true;
	if (!bidi)
		return c->cd[0].in_bytes == c->cd[1].out_bytes;

	for (i = 0; i < 2; i++) {
		if (c->cd[0].rcv != c->cd[1].snd)
			return false;
		if (c->cd[1].rcv != c->cd[0].snd)
			return false;
	}

	return true;
}

/*
 * Close this connection - send a ring message to the connection with intent
 * to stop. When the client gets the message, it will initiate the stop.
 */
static void __close_conn(struct io_uring *ring, struct conn *c)
{
	struct io_uring_sqe *sqe;
	uint64_t user_data;

	printf("Client %d: queueing stop\n", c->tid);

	user_data = __raw_encode(c->tid, __STOP, 0, 0);
	sqe = io_uring_get_sqe(ring);
	io_uring_prep_msg_ring(sqe, c->ring.ring_fd, 0, user_data, 0);
	encode_userdata(sqe, c, __NOP, 0, 0);
	io_uring_submit(ring);
}

static void close_cd(struct conn *c, struct conn_dir *cd)
{
	cd->pending_shutdown = 1;

	if (cd->pending_send)
		return;

	if (!(c->flags & CONN_F_PENDING_SHUTDOWN)) {
		gettimeofday(&c->end_time, NULL);
		c->flags |= CONN_F_PENDING_SHUTDOWN | CONN_F_END_TIME;
	}
}

/*
 * We're done with this buffer, add it back to our pool so the kernel is
 * free to use it again.
 */
static int replenish_buffer(struct conn_buf_ring *cbr, int bid, int offset)
{
	void *this_buf = cbr->buf + bid * buf_size;

	assert(bid < nr_bufs);

	io_uring_buf_ring_add(cbr->br, this_buf, buf_size, bid, br_mask, offset);
	return buf_size;
}

/*
 * Iterate buffers from '*bid' and with a total size of 'bytes' and add them
 * back to our receive ring so they can be reused for new receives.
 */
static int replenish_buffers(struct conn *c, int *bid, int bytes)
{
	struct conn_buf_ring *cbr = &c->in_br;
	int nr_packets = 0;

	while (bytes) {
		int this_len = replenish_buffer(cbr, *bid, nr_packets);

		if (this_len > bytes)
			this_len = bytes;
		bytes -= this_len;

		*bid = (*bid + 1) & (nr_bufs - 1);
		nr_packets++;
	}

	io_uring_buf_ring_advance(cbr->br, nr_packets);
	return nr_packets;
}

static void free_mvec(struct msg_vec *mvec)
{
	free(mvec->iov);
	mvec->iov = NULL;
}

static void init_mvec(struct msg_vec *mvec)
{
	memset(mvec, 0, sizeof(*mvec));
	mvec->iov = malloc(sizeof(struct iovec));
	mvec->vec_size = 1;
}

static void init_msgs(struct conn_dir *cd)
{
	memset(&cd->io_snd_msg, 0, sizeof(cd->io_snd_msg));
	memset(&cd->io_rcv_msg, 0, sizeof(cd->io_rcv_msg));
	init_mvec(&cd->io_snd_msg.vecs[0]);
	init_mvec(&cd->io_snd_msg.vecs[1]);
	init_mvec(&cd->io_rcv_msg.vecs[0]);
}

static void free_msgs(struct conn_dir *cd)
{
	free_mvec(&cd->io_snd_msg.vecs[0]);
	free_mvec(&cd->io_snd_msg.vecs[1]);
	free_mvec(&cd->io_rcv_msg.vecs[0]);
}

/*
 * Multishot accept completion triggered. If we're acting as a sink, we're
 * good to go. Just issue a receive for that case. If we're acting as a proxy,
 * then start opening a socket that we can use to connect to the other end.
 */
static int handle_accept(struct io_uring *ring, struct io_uring_cqe *cqe)
{
	struct conn *c;
	int i;

	if (nr_conns == MAX_CONNS) {
		fprintf(stderr, "max clients reached %d\n", nr_conns);
		return 1;
	}

	/* main thread handles this, which is obviously serialized */
	c = &conns[nr_conns];
	c->tid = nr_conns++;
	c->in_fd = -1;
	c->out_fd = -1;

	for (i = 0; i < 2; i++) {
		struct conn_dir *cd = &c->cd[i];

		cd->index = i;
		cd->snd_next_bid = -1;
		cd->rcv_next_bid = -1;
		if (ext_stat) {
			cd->rcv_bucket = calloc(nr_bufs, sizeof(int));
			cd->snd_bucket = calloc(nr_bufs, sizeof(int));
		}
		init_msgs(cd);
	}

	printf("New client: id=%d, in=%d\n", c->tid, c->in_fd);
	gettimeofday(&c->start_time, NULL);

	pthread_barrier_init(&c->startup_barrier, NULL, 2);
	pthread_create(&c->thread, NULL, thread_main, c);

	/*
	 * Wait for thread to have its ring setup, then either assign the fd
	 * if it's non-fixed, or pass the fixed one
	 */
	pthread_barrier_wait(&c->startup_barrier);
	if (!fixed_files) {
		c->in_fd = cqe->res;
	} else {
		struct io_uring_sqe *sqe;
		uint64_t user_data;

		/*
		 * Ring has just been setup, we'll use index 0 as the descriptor
		 * value.
		 */
		user_data = __raw_encode(c->tid, __FD_PASS, 0, 0);
		sqe = io_uring_get_sqe(ring);
		io_uring_prep_msg_ring_fd(sqe, c->ring.ring_fd, cqe->res, 0,
						user_data, 0);
		encode_userdata(sqe, c, __NOP, 0, cqe->res);
	}

	return 0;
}

/*
 * Our socket request completed, issue a connect request to the other end.
 */
static int handle_sock(struct io_uring *ring, struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);
	struct io_uring_sqe *sqe;
	int ret;

	vlog("%d: sock: res=%d\n", c->tid, cqe->res);

	c->out_fd = cqe->res;

	if (ipv6) {
		memset(&c->addr6, 0, sizeof(c->addr6));
		c->addr6.sin6_family = AF_INET6;
		c->addr6.sin6_port = htons(send_port);
		ret = inet_pton(AF_INET6, host, &c->addr6.sin6_addr);
	} else {
		memset(&c->addr, 0, sizeof(c->addr));
		c->addr.sin_family = AF_INET;
		c->addr.sin_port = htons(send_port);
		ret = inet_pton(AF_INET, host, &c->addr.sin_addr);
	}
	if (ret <= 0) {
		if (!ret)
			fprintf(stderr, "host not in right format\n");
		else
			perror("inet_pton");
		return 1;
	}

	sqe = get_sqe(ring);
	if (ipv6) {
		io_uring_prep_connect(sqe, c->out_fd,
					(struct sockaddr *) &c->addr6,
					sizeof(c->addr6));
	} else {
		io_uring_prep_connect(sqe, c->out_fd,
					(struct sockaddr *) &c->addr,
					sizeof(c->addr));
	}
	encode_userdata(sqe, c, __CONNECT, 0, c->out_fd);
	if (fixed_files)
		sqe->flags |= IOSQE_FIXED_FILE;
	return 0;
}

/*
 * Connection to the other end is done, submit a receive to start receiving
 * data. If we're a bidirectional proxy, issue a receive on both ends. If not,
 * then just a single recv will do.
 */
static int handle_connect(struct io_uring *ring, struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);

	pthread_mutex_lock(&thread_lock);
	open_conns++;
	pthread_mutex_unlock(&thread_lock);

	if (bidi)
		submit_bidi_receive(ring, c);
	else
		submit_receive(ring, c);

	return 0;
}

/*
 * Append new segment to our currently active msg_vec. This will be submitted
 * as a sendmsg (with all of it), or as separate sends, later. If we're using
 * send_ring, then we won't hit this path. Instead, outgoing buffers are
 * added directly to our outgoing send buffer ring.
 */
static void send_append_vec(struct conn_dir *cd, void *data, int len)
{
	struct msg_vec *mvec = snd_msg_vec(cd);

	if (mvec->iov_len == mvec->vec_size) {
		mvec->vec_size <<= 1;
		mvec->iov = realloc(mvec->iov, mvec->vec_size * sizeof(struct iovec));
	}

	mvec->iov[mvec->iov_len].iov_base = data;
	mvec->iov[mvec->iov_len].iov_len = len;
	mvec->iov_len++;
}

/*
 * Queue a send based on the data received in this cqe, which came from
 * a completed receive operation.
 */
static void send_append(struct conn *c, struct conn_dir *cd, void *data,
			int bid, int len)
{
	vlog("%d: send %d (%p, bid %d)\n", c->tid, len, data, bid);

	assert(bid < nr_bufs);

	/* if using provided buffers for send, add it upfront */
	if (send_ring) {
		struct conn_buf_ring *cbr = &c->out_br;

		io_uring_buf_ring_add(cbr->br, data, len, bid, br_mask, 0);
		io_uring_buf_ring_advance(cbr->br, 1);
	} else {
		send_append_vec(cd, data, len);
	}
}

/*
 * For non recvmsg && multishot, a zero receive marks the end. For recvmsg
 * with multishot, we always get the header regardless. Hence a "zero receive"
 * is the size of the header.
 */
static int recv_done_res(int res)
{
	if (!res)
		return 1;
	if (rcv_msg && recv_mshot && res == sizeof(struct io_uring_recvmsg_out))
		return 1;
	return 0;
}

/*
 * Any receive that isn't recvmsg with multishot can be handled the same way.
 * Iterate from '*bid' and 'in_bytes' in total, and append the data to the
 * outgoing queue.
 */
static int recv_bids(struct conn *c, struct conn_dir *cd, int *bid, int in_bytes)
{
	struct conn_buf_ring *cbr = &c->out_br;
	struct conn_buf_ring *in_cbr = &c->in_br;
	struct io_uring_buf *buf;
	int nr_packets = 0;

	while (in_bytes) {
		int this_bytes;
		void *data;

		buf = &in_cbr->br->bufs[*bid];
		data = (void *) (unsigned long) buf->addr;
		this_bytes = buf->len;
		if (this_bytes > in_bytes)
			this_bytes = in_bytes;

		in_bytes -= this_bytes;

		if (send_ring)
			io_uring_buf_ring_add(cbr->br, data, this_bytes, *bid,
						br_mask, nr_packets);
		else
			send_append(c, cd, data, *bid, this_bytes);

		*bid = (*bid + 1) & (nr_bufs - 1);
		nr_packets++;
	}

	if (send_ring)
		io_uring_buf_ring_advance(cbr->br, nr_packets);

	return nr_packets;
}

/*
 * Special handling of recvmsg with multishot
 */
static int recv_mshot_msg(struct conn *c, struct conn_dir *cd, int *bid,
			  int in_bytes)
{
	struct conn_buf_ring *cbr = &c->out_br;
	struct conn_buf_ring *in_cbr = &c->in_br;
	struct io_uring_buf *buf;
	int nr_packets = 0;

	while (in_bytes) {
		struct io_uring_recvmsg_out *pdu;
		int this_bytes;
		void *data;

		buf = &in_cbr->br->bufs[*bid];

		/*
		 * multishot recvmsg puts a header in front of the data - we
		 * have to take that into account for the send setup, and
		 * adjust the actual data read to not take this metadata into
		 * account. For this use case, namelen and controllen will not
		 * be set. If they were, they would need to be factored in too.
		 */
		buf->len -= sizeof(struct io_uring_recvmsg_out);
		in_bytes -= sizeof(struct io_uring_recvmsg_out);

		pdu = (void *) (unsigned long) buf->addr;
		vlog("pdu namelen %d, controllen %d, payload %d flags %x\n",
				pdu->namelen, pdu->controllen, pdu->payloadlen,
				pdu->flags);
		data = (void *) (pdu + 1);

		this_bytes = pdu->payloadlen;
		if (this_bytes > in_bytes)
			this_bytes = in_bytes;

		in_bytes -= this_bytes;

		if (send_ring)
			io_uring_buf_ring_add(cbr->br, data, this_bytes, *bid,
						br_mask, nr_packets);
		else
			send_append(c, cd, data, *bid, this_bytes);

		*bid = (*bid + 1) & (nr_bufs - 1);
		nr_packets++;
	}

	if (send_ring)
		io_uring_buf_ring_advance(cbr->br, nr_packets);

	return nr_packets;
}

static int __handle_recv(struct io_uring *ring, struct conn *c,
			 struct conn_dir *cd, struct io_uring_cqe *cqe)
{
	struct conn_dir *ocd = &c->cd[!cd->index];
	int bid, nr_packets;

	/*
	 * Not having a buffer attached should only happen if we get a zero
	 * sized receive, because the other end closed the connection. It
	 * cannot happen otherwise, as all our receives are using provided
	 * buffers and hence it's not possible to return a CQE with a non-zero
	 * result and not have a buffer attached.
	 */
	if (!(cqe->flags & IORING_CQE_F_BUFFER)) {
		cd->pending_recv = 0;

		if (!recv_done_res(cqe->res)) {
			fprintf(stderr, "no buffer assigned, res=%d\n", cqe->res);
			return 1;
		}
start_close:
		prep_next_send(ring, c, ocd, other_dir_fd(c, cqe_to_fd(cqe)));
		close_cd(c, cd);
		return 0;
	}

	if (cqe->res && cqe->res < buf_size)
		cd->rcv_shrt++;

	bid = cqe->flags >> IORING_CQE_BUFFER_SHIFT;

	/*
	 * BIDI will use the same buffer pool and do receive on both CDs,
	 * so can't reliably check. TODO.
	 */
	if (!bidi && cd->rcv_next_bid != -1 && bid != cd->rcv_next_bid) {
		fprintf(stderr, "recv bid %d, wanted %d\n", bid, cd->rcv_next_bid);
		goto start_close;
	}

	vlog("%d: recv: bid=%d, res=%d, cflags=%x\n", c->tid, bid, cqe->res, cqe->flags);
	/*
	 * If we're a sink, we're done here. Just replenish the buffer back
	 * to the pool. For proxy mode, we will send the data to the other
	 * end and the buffer will be replenished once the send is done with
	 * it.
	 */
	if (is_sink)
		nr_packets = replenish_buffers(c, &bid, cqe->res);
	else if (rcv_msg && recv_mshot)
		nr_packets = recv_mshot_msg(c, ocd, &bid, cqe->res);
	else
		nr_packets = recv_bids(c, ocd, &bid, cqe->res);

	if (cd->rcv_bucket)
		cd->rcv_bucket[nr_packets]++;

	if (!is_sink) {
		ocd->out_buffers += nr_packets;
		assert(ocd->out_buffers <= nr_bufs);
	}

	cd->rcv++;
	cd->rcv_next_bid = bid;

	/*
	 * If IORING_CQE_F_MORE isn't set, then this is either a normal recv
	 * that needs rearming, or it's a multishot that won't post any further
	 * completions. Setup a new one for these cases.
	 */
	if (!(cqe->flags & IORING_CQE_F_MORE)) {
		cd->pending_recv = 0;
		if (recv_done_res(cqe->res))
			goto start_close;
		if (is_sink)
			__submit_receive(ring, c, &c->cd[0], c->in_fd);
	}

	/*
	 * Submit a send if we won't get anymore notifications from this
	 * recv, or if we have nr_bufs / 2 queued up. If BIDI mode, send
	 * every buffer. We assume this is interactive mode, and hence don't
	 * delay anything.
	 */
	if (((!ocd->pending_send && (bidi || (ocd->out_buffers >= nr_bufs / 2))) ||
	    !(cqe->flags & IORING_CQE_F_MORE)) && !is_sink)
		prep_next_send(ring, c, ocd, other_dir_fd(c, cqe_to_fd(cqe)));

	if (!recv_done_res(cqe->res))
		cd->in_bytes += cqe->res;
	return 0;
}

static int handle_recv(struct io_uring *ring, struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);
	struct conn_dir *cd = cqe_to_conn_dir(c, cqe);

	return __handle_recv(ring, c, cd, cqe);
}

static int recv_error(struct error_handler *err, struct io_uring *ring,
		      struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);
	struct conn_dir *cd = cqe_to_conn_dir(c, cqe);

	cd->pending_recv = 0;

	if (cqe->res != -ENOBUFS)
		return default_error(err, ring, cqe);

	recv_enobufs(ring, c, cd, other_dir_fd(c, cqe_to_fd(cqe)));
	return 0;
}

static void submit_send(struct io_uring *ring, struct conn *c,
			struct conn_dir *cd, int fd, void *data, int len,
			int bid, int flags)
{
	struct io_uring_sqe *sqe;
	int bgid = c->out_br.bgid;

	if (cd->pending_send)
		return;
	cd->pending_send = 1;

	flags |= MSG_WAITALL | MSG_NOSIGNAL;

	sqe = get_sqe(ring);
	if (snd_msg) {
		struct io_msg *imsg = &cd->io_snd_msg;

		if (snd_zc) {
			io_uring_prep_sendmsg_zc(sqe, fd, &imsg->msg, flags);
			cd->snd_notif++;
		} else {
			io_uring_prep_sendmsg(sqe, fd, &imsg->msg, flags);
		}
	} else if (send_ring) {
		io_uring_prep_send(sqe, fd, NULL, 0, flags);
	} else if (!snd_zc) {
		io_uring_prep_send(sqe, fd, data, len, flags);
	} else {
		io_uring_prep_send_zc(sqe, fd, data, len, flags, 0);
		sqe->ioprio |= IORING_RECVSEND_FIXED_BUF;
		sqe->buf_index = bid;
		cd->snd_notif++;
	}
	encode_userdata(sqe, c, __SEND, bid, fd);
	if (fixed_files)
		sqe->flags |= IOSQE_FIXED_FILE;
	if (send_ring) {
		sqe->flags |= IOSQE_BUFFER_SELECT;
		sqe->buf_group = bgid;
	}
	if (snd_bundle) {
		sqe->ioprio |= IORING_RECVSEND_BUNDLE;
		cd->snd_mshot++;
	} else if (send_ring)
		cd->snd_mshot++;
}

/*
 * Prepare the next send request, if we need to. If one is already pending,
 * or if we're a sink and we don't need to do sends, then there's nothing
 * to do.
 *
 * Return 1 if another send completion is expected, 0 if not.
 */
static int prep_next_send(struct io_uring *ring, struct conn *c,
			   struct conn_dir *cd, int fd)
{
	int bid;

	if (cd->pending_send || is_sink)
		return 0;
	if (!cd->out_buffers)
		return 0;

	bid = cd->snd_next_bid;
	if (bid == -1)
		bid = 0;

	if (send_ring) {
		/*
		 * send_ring mode is easy, there's nothing to do but submit
		 * our next send request. That will empty the entire outgoing
		 * queue.
		 */
		submit_send(ring, c, cd, fd, NULL, 0, bid, 0);
		return 1;
	} else if (snd_msg) {
		/*
		 * For sendmsg mode, submit our currently prepared iovec, if
		 * we have one, and swap our iovecs so that any further
		 * receives will start preparing that one.
		 */
		struct io_msg *imsg = &cd->io_snd_msg;

		if (!msg_vec(imsg)->iov_len)
			return 0;
		imsg->msg.msg_iov = msg_vec(imsg)->iov;
		imsg->msg.msg_iovlen = msg_vec(imsg)->iov_len;
		msg_vec(imsg)->iov_len = 0;
		imsg->vec_index = !imsg->vec_index;
		submit_send(ring, c, cd, fd, NULL, 0, bid, 0);
		return 1;
	} else {
		/*
		 * send without send_ring - submit the next available vec,
		 * if any. If this vec is the last one in the current series,
		 * then swap to the next vec. We flag each send with MSG_MORE,
		 * unless this is the last part of the current vec.
		 */
		struct io_msg *imsg = &cd->io_snd_msg;
		struct msg_vec *mvec = msg_vec(imsg);
		int flags = !snd_zc ? MSG_MORE : 0;
		struct iovec *iov;

		if (mvec->iov_len == mvec->cur_iov)
			return 0;
		imsg->msg.msg_iov = msg_vec(imsg)->iov;
		iov = &mvec->iov[mvec->cur_iov];
		mvec->cur_iov++;
		if (mvec->cur_iov == mvec->iov_len) {
			mvec->iov_len = 0;
			mvec->cur_iov = 0;
			imsg->vec_index = !imsg->vec_index;
			flags = 0;
		}
		submit_send(ring, c, cd, fd, iov->iov_base, iov->iov_len, bid, flags);
		return 1;
	}
}

/*
 * Handling a send with an outgoing send ring. Get the buffers from the
 * receive side, and add them to the ingoing buffer ring again.
 */
static int handle_send_ring(struct conn *c, struct conn_dir *cd,
			    int bid, int bytes)
{
	struct conn_buf_ring *in_cbr = &c->in_br;
	struct conn_buf_ring *out_cbr = &c->out_br;
	int i = 0;

	while (bytes) {
		struct io_uring_buf *buf = &out_cbr->br->bufs[bid];
		int this_bytes;
		void *this_buf;

		this_bytes = buf->len;
		if (this_bytes > bytes)
			this_bytes = bytes;

		cd->out_bytes += this_bytes;

		vlog("%d: send: bid=%d, len=%d\n", c->tid, bid, this_bytes);

		this_buf = in_cbr->buf + bid * buf_size;
		io_uring_buf_ring_add(in_cbr->br, this_buf, buf_size, bid, br_mask, i);
		/*
		 * Find the provided buffer that the receive consumed, and
		 * which we then used for the send, and add it back to the
		 * pool so it can get picked by another receive. Once the send
		 * is done, we're done with it.
		 */
		bid = (bid + 1) & (nr_bufs - 1);
		bytes -= this_bytes;
		i++;
	}
	cd->snd_next_bid = bid;
	io_uring_buf_ring_advance(in_cbr->br, i);

	if (pending_shutdown(c))
		close_cd(c, cd);

	return i;
}

/*
 * sendmsg, or send without a ring. Just add buffers back to the ingoing
 * ring for receives.
 */
static int handle_send_buf(struct conn *c, struct conn_dir *cd, int bid,
			   int bytes)
{
	struct conn_buf_ring *in_cbr = &c->in_br;
	int i = 0;

	while (bytes) {
		struct io_uring_buf *buf = &in_cbr->br->bufs[bid];
		int this_bytes;

		this_bytes = bytes;
		if (this_bytes > buf->len)
			this_bytes = buf->len;

		vlog("%d: send: bid=%d, len=%d\n", c->tid, bid, this_bytes);

		cd->out_bytes += this_bytes;
		/* each recvmsg mshot package has this overhead */
		if (rcv_msg && recv_mshot)
			cd->out_bytes += sizeof(struct io_uring_recvmsg_out);
		replenish_buffer(in_cbr, bid, i);
		bid = (bid + 1) & (nr_bufs - 1);
		bytes -= this_bytes;
		i++;
	}
	io_uring_buf_ring_advance(in_cbr->br, i);
	cd->snd_next_bid = bid;
	return i;
}

static int __handle_send(struct io_uring *ring, struct conn *c,
			 struct conn_dir *cd, struct io_uring_cqe *cqe)
{
	struct conn_dir *ocd;
	int bid, nr_packets;

	if (send_ring) {
		if (!(cqe->flags & IORING_CQE_F_BUFFER)) {
			fprintf(stderr, "no buffer in send?! %d\n", cqe->res);
			return 1;
		}
		bid = cqe->flags >> IORING_CQE_BUFFER_SHIFT;
	} else {
		bid = cqe_to_bid(cqe);
	}

	/*
	 * CQE notifications only happen with send/sendmsg zerocopy. They
	 * tell us that the data has been acked, and that hence the buffer
	 * is now free to reuse. Waiting on an ACK for each packet will slow
	 * us down tremendously, so do all of our sends and then wait for
	 * the ACKs to come in. They tend to come in bundles anyway. Once
	 * all acks are done (cd->snd_notif == 0), then fire off the next
	 * receive.
	 */
	if (cqe->flags & IORING_CQE_F_NOTIF) {
		cd->snd_notif--;
	} else {
		if (cqe->res && cqe->res < buf_size)
			cd->snd_shrt++;

		/*
		 * BIDI will use the same buffer pool and do sends on both CDs,
		 * so can't reliably check. TODO.
		 */
		if (!bidi && send_ring && cd->snd_next_bid != -1 &&
		    bid != cd->snd_next_bid) {
			fprintf(stderr, "send bid %d, wanted %d at %lu\n", bid,
					cd->snd_next_bid, cd->out_bytes);
			goto out_close;
		}

		assert(bid <= nr_bufs);

		vlog("send: got %d, %lu\n", cqe->res, cd->out_bytes);

		if (send_ring)
			nr_packets = handle_send_ring(c, cd, bid, cqe->res);
		else
			nr_packets = handle_send_buf(c, cd, bid, cqe->res);

		if (cd->snd_bucket)
			cd->snd_bucket[nr_packets]++;

		cd->out_buffers -= nr_packets;
		assert(cd->out_buffers >= 0);

		cd->snd++;
	}

	if (!(cqe->flags & IORING_CQE_F_MORE)) {
		int do_recv_arm;

		cd->pending_send = 0;

		/*
		 * send done - see if the current vec has data to submit, and
		 * do so if it does. if it doesn't have data yet, nothing to
		 * do.
		 */
		do_recv_arm = !prep_next_send(ring, c, cd, cqe_to_fd(cqe));

		ocd = &c->cd[!cd->index];
		if (!cd->snd_notif && do_recv_arm && !ocd->pending_recv) {
			int fd = other_dir_fd(c, cqe_to_fd(cqe));

			__submit_receive(ring, c, ocd, fd);
		}
out_close:
		if (pending_shutdown(c))
			close_cd(c, cd);
	}

	vlog("%d: pending sends %d\n", c->tid, cd->pending_send);
	return 0;
}

static int handle_send(struct io_uring *ring, struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);
	struct conn_dir *cd = cqe_to_conn_dir(c, cqe);

	return __handle_send(ring, c, cd, cqe);
}

static int send_error(struct error_handler *err, struct io_uring *ring,
		      struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);
	struct conn_dir *cd = cqe_to_conn_dir(c, cqe);

	cd->pending_send = 0;

	/* res can have high bit set */
	if (cqe->flags & IORING_CQE_F_NOTIF)
		return handle_send(ring, cqe);
	if (cqe->res != -ENOBUFS)
		return default_error(err, ring, cqe);

	cd->snd_enobufs++;
	return 0;
}

/*
 * We don't expect to get here, as we marked it with skipping posting a
 * CQE if it was successful. If it does trigger, than means it fails and
 * that our close has not been done. Log the shutdown error and issue a new
 * separate close.
 */
static int handle_shutdown(struct io_uring *ring, struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);
	struct io_uring_sqe *sqe;
	int fd = cqe_to_fd(cqe);

	fprintf(stderr, "Got shutdown notication on fd %d\n", fd);

	if (!cqe->res)
		fprintf(stderr, "Unexpected success shutdown CQE\n");
	else if (cqe->res < 0)
		fprintf(stderr, "Shutdown got %s\n", strerror(-cqe->res));

	sqe = get_sqe(ring);
	if (fixed_files)
		io_uring_prep_close_direct(sqe, fd);
	else
		io_uring_prep_close(sqe, fd);
	encode_userdata(sqe, c, __CLOSE, 0, fd);
	return 0;
}

/*
 * Final stage of a connection, the shutdown and close has finished. Mark
 * it as disconnected and let the main loop reap it.
 */
static int handle_close(struct io_uring *ring, struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);
	int fd = cqe_to_fd(cqe);

	printf("Closed client: id=%d, in_fd=%d, out_fd=%d\n", c->tid, c->in_fd, c->out_fd);
	if (fd == c->in_fd)
		c->in_fd = -1;
	else if (fd == c->out_fd)
		c->out_fd = -1;

	if (c->in_fd == -1 && c->out_fd == -1) {
		c->flags |= CONN_F_DISCONNECTED;

		pthread_mutex_lock(&thread_lock);
		__show_stats(c);
		open_conns--;
		pthread_mutex_unlock(&thread_lock);
		free_buffer_rings(ring, c);
		free_msgs(&c->cd[0]);
		free_msgs(&c->cd[1]);
		free(c->cd[0].rcv_bucket);
		free(c->cd[0].snd_bucket);
	}

	return 0;
}

static int handle_cancel(struct io_uring *ring, struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);
	int fd = cqe_to_fd(cqe);

	c->pending_cancels--;

	vlog("%d: got cancel fd %d, refs %d\n", c->tid, fd, c->pending_cancels);

	if (!c->pending_cancels) {
		queue_shutdown_close(ring, c, c->in_fd);
		if (c->out_fd != -1)
			queue_shutdown_close(ring, c, c->out_fd);
		io_uring_submit(ring);
	}

	return 0;
}

static void open_socket(struct conn *c)
{
	if (is_sink) {
		pthread_mutex_lock(&thread_lock);
		open_conns++;
		pthread_mutex_unlock(&thread_lock);

		submit_receive(&c->ring, c);
	} else {
		struct io_uring_sqe *sqe;
		int domain;

		if (ipv6)
			domain = AF_INET6;
		else
			domain = AF_INET;

		/*
		 * If fixed_files is set, proxy will use fixed files for any new
		 * file descriptors it instantiates. Fixd files, or fixed
		 * descriptors, are io_uring private file descriptors. They
		 * cannot be accessed outside of io_uring. io_uring holds a
		 * fixed reference to them, which means that we do not need to
		 * grab per-request references to them. Particularly for
		 * threaded applications, grabbing and dropping file references
		 * for each operation can be costly as the file table is shared.
		 * This generally shows up as fget/fput related overhead in any
		 * workload profiles.
		 *
		 * Fixed descriptors are passed in via the 'fd' field just like
		 * regular descriptors, and then marked as such by setting the
		 * IOSQE_FIXED_FILE flag in the sqe->flags field. Some helpers
		 * do that automatically, like the below, others will need it
		 * set manually if they don't have a *direct*() helper.
		 *
		 * For operations that instantiate them, like the opening of a
		 * direct socket, the application may either ask the kernel to
		 * find a free one (as is done below), or the application may
		 * manage the space itself and pass in an index for a currently
		 * free slot in the table. If the kernel is asked to allocate a
		 * free direct descriptor, note that io_uring does not abide by
		 * the POSIX mandated "lowest free must be returned". It may
		 * return any free descriptor of its choosing.
		 */
		sqe = get_sqe(&c->ring);
		if (fixed_files)
			io_uring_prep_socket_direct_alloc(sqe, domain, SOCK_STREAM, 0, 0);
		else
			io_uring_prep_socket(sqe, domain, SOCK_STREAM, 0, 0);
		encode_userdata(sqe, c, __SOCK, 0, 0);
	}
}

/*
 * Start of connection, we got our in descriptor.
 */
static int handle_fd_pass(struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);
	int fd = cqe_to_fd(cqe);

	vlog("%d: got fd pass %d\n", c->tid, fd);
	c->in_fd = fd;
	open_socket(c);
	return 0;
}

static int handle_stop(struct io_uring_cqe *cqe)
{
	struct conn *c = cqe_to_conn(cqe);

	printf("Client %d: queueing shutdown\n", c->tid);
	queue_cancel(&c->ring, c);
	return 0;
}

/*
 * Called for each CQE that we receive. Decode the request type that it
 * came from, and call the appropriate handler.
 */
static int handle_cqe(struct io_uring *ring, struct io_uring_cqe *cqe)
{
	int ret;

	/*
	 * Unlikely, but there's an error in this CQE. If an error handler
	 * is defined, call it, and that will deal with it. If no error
	 * handler is defined, the opcode handler either doesn't care or will
	 * handle it on its own.
	 */
	if (cqe->res < 0) {
		struct error_handler *err = &error_handlers[cqe_to_op(cqe)];

		if (err->error_fn)
			return err->error_fn(err, ring, cqe);
	}

	switch (cqe_to_op(cqe)) {
	case __ACCEPT:
		ret = handle_accept(ring, cqe);
		break;
	case __SOCK:
		ret = handle_sock(ring, cqe);
		break;
	case __CONNECT:
		ret = handle_connect(ring, cqe);
		break;
	case __RECV:
	case __RECVMSG:
		ret = handle_recv(ring, cqe);
		break;
	case __SEND:
	case __SENDMSG:
		ret = handle_send(ring, cqe);
		break;
	case __CANCEL:
		ret = handle_cancel(ring, cqe);
		break;
	case __SHUTDOWN:
		ret = handle_shutdown(ring, cqe);
		break;
	case __CLOSE:
		ret = handle_close(ring, cqe);
		break;
	case __FD_PASS:
		ret = handle_fd_pass(cqe);
		break;
	case __STOP:
		ret = handle_stop(cqe);
		break;
	case __NOP:
		ret = 0;
		break;
	default:
		fprintf(stderr, "bad user data %lx\n", (long) cqe->user_data);
		return 1;
	}

	return ret;
}

static void house_keeping(struct io_uring *ring)
{
	static unsigned long last_bytes;
	unsigned long bytes, elapsed;
	struct conn *c;
	int i, j;

	vlog("House keeping entered\n");

	bytes = 0;
	for (i = 0; i < nr_conns; i++) {
		c = &conns[i];

		for (j = 0; j < 2; j++) {
			struct conn_dir *cd = &c->cd[j];

			bytes += cd->in_bytes + cd->out_bytes;
		}
		if (c->flags & CONN_F_DISCONNECTED) {
			vlog("%d: disconnected\n", i);

			if (!(c->flags & CONN_F_REAPED)) {
				void *ret;

				pthread_join(c->thread, &ret);
				c->flags |= CONN_F_REAPED;
			}
			continue;
		}
		if (c->flags & CONN_F_DISCONNECTING)
			continue;

		if (should_shutdown(c)) {
			__close_conn(ring, c);
			c->flags |= CONN_F_DISCONNECTING;
		}
	}

	elapsed = mtime_since_now(&last_housekeeping);
	if (bytes && elapsed >= 900) {
		unsigned long bw;

		bw = (8 * (bytes - last_bytes) / 1000UL) / elapsed;
		if (bw) {
			if (open_conns)
				printf("Bandwidth (threads=%d): %'luMbit\n", open_conns, bw);
			gettimeofday(&last_housekeeping, NULL);
			last_bytes = bytes;
		}
	}
}

/*
 * Event loop shared between the parent, and the connections. Could be
 * split in two, as they don't handle the same types of events. For the per
 * connection loop, 'c' is valid. For the main loop, it's NULL.
 */
static int __event_loop(struct io_uring *ring, struct conn *c)
{
	struct __kernel_timespec active_ts, idle_ts;
	int flags;

	idle_ts.tv_sec = 0;
	idle_ts.tv_nsec = 100000000LL;
	active_ts = idle_ts;
	if (wait_usec > 1000000) {
		active_ts.tv_sec = wait_usec / 1000000;
		wait_usec -= active_ts.tv_sec * 1000000;
	}
	active_ts.tv_nsec = wait_usec * 1000;

	gettimeofday(&last_housekeeping, NULL);

	flags = 0;
	while (1) {
		struct __kernel_timespec *ts = &idle_ts;
		struct io_uring_cqe *cqe;
		unsigned int head;
		int ret, i, to_wait;

		/*
		 * If wait_batch is set higher than 1, then we'll wait on
		 * that amount of CQEs to be posted each loop. If used with
		 * DEFER_TASKRUN, this can provide a substantial reduction
		 * in context switch rate as the task isn't woken until the
		 * requested number of events can be returned.
		 *
		 * Can be used with -t to set a wait_usec timeout as well.
		 * For example, if an application can deal with 250 usec
		 * of wait latencies, it can set -w8 -t250 which will cause
		 * io_uring to return when either 8 events have been received,
		 * or if 250 usec of waiting has passed.
		 *
		 * If we don't have any open connections, wait on just 1
		 * always.
		 */
		to_wait = 1;
		if (open_conns && !flags) {
			ts = &active_ts;
			to_wait = wait_batch;
		}

		vlog("Submit and wait for %d\n", to_wait);
		ret = io_uring_submit_and_wait_timeout(ring, &cqe, to_wait, ts, NULL);

		if (*ring->cq.koverflow)
			printf("overflow %u\n", *ring->cq.koverflow);
		if (*ring->sq.kflags &  IORING_SQ_CQ_OVERFLOW)
			printf("saw overflow\n");

		vlog("Submit and wait: %d\n", ret);

		i = flags = 0;
		io_uring_for_each_cqe(ring, head, cqe) {
			if (handle_cqe(ring, cqe))
				return 1;
			flags |= cqe_to_conn(cqe)->flags;
			++i;
		}

		vlog("Handled %d events\n", i);

		/*
		 * Advance the CQ ring for seen events when we've processed
		 * all of them in this loop. This can also be done with
		 * io_uring_cqe_seen() in each handler above, which just marks
		 * that single CQE as seen. However, it's more efficient to
		 * mark a batch as seen when we're done with that batch.
		 */
		if (i) {
			io_uring_cq_advance(ring, i);
			events += i;
		}

		event_loops++;
		if (c) {
			if (c->flags & CONN_F_DISCONNECTED)
				break;
		} else {
			house_keeping(ring);
		}
	}

	return 0;
}

/*
 * Main event loop, Submit our multishot accept request, and then just loop
 * around handling incoming connections.
 */
static int parent_loop(struct io_uring *ring, int fd)
{
	struct io_uring_sqe *sqe;

	/*
	 * proxy provides a way to use either multishot receive or not, but
	 * for accept, we always use multishot. A multishot accept request
	 * needs only be armed once, and then it'll trigger a completion and
	 * post a CQE whenever a new connection is accepted. No need to do
	 * anything else, unless the multishot accept terminates. This happens
	 * if it encounters an error. Applications should check for
	 * IORING_CQE_F_MORE in cqe->flags - this tells you if more completions
	 * are expected from this request or not. Non-multishot never have
	 * this set, where multishot will always have this set unless an error
	 * occurs.
	 */
	sqe = get_sqe(ring);
	if (fixed_files)
		io_uring_prep_multishot_accept_direct(sqe, fd, NULL, NULL, 0);
	else
		io_uring_prep_multishot_accept(sqe, fd, NULL, NULL, 0);
	__encode_userdata(sqe, 0, __ACCEPT, 0, fd);

	return __event_loop(ring, NULL);
}

static int init_ring(struct io_uring *ring, int nr_files)
{
	struct io_uring_params params;
	int ret;

	/*
	 * By default, set us up with a big CQ ring. Not strictly needed
	 * here, but it's very important to never overflow the CQ ring.
	 * Events will not be dropped if this happens, but it does slow
	 * the application down in dealing with overflown events.
	 *
	 * Set SINGLE_ISSUER, which tells the kernel that only one thread
	 * is doing IO submissions. This enables certain optimizations in
	 * the kernel.
	 */
	memset(&params, 0, sizeof(params));
	params.flags |= IORING_SETUP_SINGLE_ISSUER | IORING_SETUP_CLAMP;
	params.flags |= IORING_SETUP_CQSIZE;
	params.cq_entries = 1024;

	/*
	 * If use_huge is set, setup the ring with IORING_SETUP_NO_MMAP. This
	 * means that the application allocates the memory for the ring, and
	 * the kernel maps it. The alternative is having the kernel allocate
	 * the memory, and then liburing will mmap it. But we can't really
	 * support huge pages that way. If this fails, then ensure that the
	 * system has huge pages set aside upfront.
	 */
	if (use_huge)
		params.flags |= IORING_SETUP_NO_MMAP;

	/*
	 * DEFER_TASKRUN decouples async event reaping and retrying from
	 * regular system calls. If this isn't set, then io_uring uses
	 * normal task_work for this. task_work is always being run on any
	 * exit to userspace. Real applications do more than just call IO
	 * related system calls, and hence we can be running this work way
	 * too often. Using DEFER_TASKRUN defers any task_work running to
	 * when the application enters the kernel anyway to wait on new
	 * events. It's generally the preferred and recommended way to setup
	 * a ring.
	 */
	if (defer_tw) {
		params.flags |= IORING_SETUP_DEFER_TASKRUN;
		sqpoll = 0;
	}

	/*
	 * SQPOLL offloads any request submission and retry operations to a
	 * dedicated thread. This enables an application to do IO without
	 * ever having to enter the kernel itself. The SQPOLL thread will
	 * stay busy as long as there's work to do, and go to sleep if
	 * sq_thread_idle msecs have passed. If it's running, submitting new
	 * IO just needs to make them visible to the SQPOLL thread, it needs
	 * not enter the kernel. For submission, the application will only
	 * enter the kernel if the SQPOLL has been idle long enough that it
	 * has gone to sleep.
	 *
	 * Waiting on events still need to enter the kernel, if none are
	 * available. The application may also use io_uring_peek_cqe() to
	 * check for new events without entering the kernel, as completions
	 * will be continually produced to the CQ ring by the SQPOLL thread
	 * as they occur.
	 */
	if (sqpoll) {
		params.flags |= IORING_SETUP_SQPOLL;
		params.sq_thread_idle = 1000;
		defer_tw = 0;
	}

	/*
	 * If neither DEFER_TASKRUN or SQPOLL is used, set COOP_TASKRUN. This
	 * avoids heavy signal based notifications, which can force an
	 * application to enter the kernel and process it as soon as they
	 * occur.
	 */
	if (!sqpoll && !defer_tw)
		params.flags |= IORING_SETUP_COOP_TASKRUN;

	/*
	 * The SQ ring size need not be larger than any batch of requests
	 * that need to be prepared before submit. Normally in a loop we'd
	 * only need a few, if any, particularly if multishot is used.
	 */
	ret = io_uring_queue_init_params(ring_size, ring, &params);
	if (ret) {
		fprintf(stderr, "%s\n", strerror(-ret));
		return 1;
	}

	/*
	 * If send serialization is available and no option was given to use
	 * it or not, default it to on. If it was turned on and the kernel
	 * doesn't support it, turn it off.
	 */
	if (params.features & IORING_FEAT_SEND_BUF_SELECT) {
		if (send_ring == -1)
			send_ring = 1;
	} else {
		if (send_ring == 1) {
			fprintf(stderr, "Kernel doesn't support ring provided "
				"buffers for sends, disabled\n");
		}
		send_ring = 0;
	}

	if (!send_ring && snd_bundle) {
		fprintf(stderr, "Can't use send bundle without send_ring\n");
		snd_bundle = 0;
	}

	if (fixed_files) {
		/*
		 * If fixed files are used, we need to allocate a fixed file
		 * table upfront where new direct descriptors can be managed.
		 */
		ret = io_uring_register_files_sparse(ring, nr_files);
		if (ret) {
			fprintf(stderr, "file register: %d\n", ret);
			return 1;
		}

		/*
		 * If fixed files are used, we also register the ring fd. See
		 * comment near io_uring_prep_socket_direct_alloc() further
		 * down. This avoids the fget/fput overhead associated with
		 * the io_uring_enter(2) system call itself, which is used to
		 * submit and wait on events.
		 */
		ret = io_uring_register_ring_fd(ring);
		if (ret != 1) {
			fprintf(stderr, "ring register: %d\n", ret);
			return 1;
		}
	}

	if (napi) {
		struct io_uring_napi n = {
			.prefer_busy_poll = napi > 1 ? 1 : 0,
			.busy_poll_to = napi_timeout,
		};

		ret = io_uring_register_napi(ring, &n);
		if (ret) {
			fprintf(stderr, "io_uring_register_napi: %d\n", ret);
			if (ret != -EINVAL)
				return 1;
			fprintf(stderr, "NAPI not available, turned off\n");
		}
	}

	return 0;
}

static void *thread_main(void *data)
{
	struct conn *c = data;
	int ret;

	c->flags |= CONN_F_STARTED;

	/* we need a max of 4 descriptors for each client */
	ret = init_ring(&c->ring, 4);
	if (ret)
		goto done;

	if (setup_buffer_rings(&c->ring, c))
		goto done;

	/*
	 * If we're using fixed files, then we need to wait for the parent
	 * to install the c->in_fd into our direct descriptor table. When
	 * that happens, we'll set things up. If we're not using fixed files,
	 * we can set up the receive or connect now.
	 */
	if (!fixed_files)
		open_socket(c);

	/* we're ready */
	pthread_barrier_wait(&c->startup_barrier);

	__event_loop(&c->ring, c);
done:
	return NULL;
}

static void usage(const char *name)
{
	printf("%s:\n", name);
	printf("\t-m:\t\tUse multishot receive (%d)\n", recv_mshot);
	printf("\t-d:\t\tUse DEFER_TASKRUN (%d)\n", defer_tw);
	printf("\t-S:\t\tUse SQPOLL (%d)\n", sqpoll);
	printf("\t-f:\t\tUse only fixed files (%d)\n", fixed_files);
	printf("\t-a:\t\tUse huge pages for the ring (%d)\n", use_huge);
	printf("\t-t:\t\tTimeout for waiting on CQEs (usec) (%d)\n", wait_usec);
	printf("\t-w:\t\tNumber of CQEs to wait for each loop (%d)\n", wait_batch);
	printf("\t-B:\t\tUse bi-directional mode (%d)\n", bidi);
	printf("\t-s:\t\tAct only as a sink (%d)\n", is_sink);
	printf("\t-q:\t\tRing size to use (%d)\n", ring_size);
	printf("\t-H:\t\tHost to connect to (%s)\n", host);
	printf("\t-r:\t\tPort to receive on (%d)\n", receive_port);
	printf("\t-p:\t\tPort to connect to (%d)\n", send_port);
	printf("\t-6:\t\tUse IPv6 (%d)\n", ipv6);
	printf("\t-N:\t\tUse NAPI polling (%d)\n", napi);
	printf("\t-T:\t\tNAPI timeout (usec) (%d)\n", napi_timeout);
	printf("\t-b:\t\tSend/receive buf size (%d)\n", buf_size);
	printf("\t-n:\t\tNumber of provided buffers (pow2) (%d)\n", nr_bufs);
	printf("\t-u:\t\tUse provided buffers for send (%d)\n", send_ring);
	printf("\t-C:\t\tUse bundles for send (%d)\n", snd_bundle);
	printf("\t-z:\t\tUse zerocopy send (%d)\n", snd_zc);
	printf("\t-c:\t\tUse bundles for recv (%d)\n", snd_bundle);
	printf("\t-M:\t\tUse sendmsg (%d)\n", snd_msg);
	printf("\t-M:\t\tUse recvmsg (%d)\n", rcv_msg);
	printf("\t-x:\t\tShow extended stats (%d)\n", ext_stat);
	printf("\t-V:\t\tIncrease verbosity (%d)\n", verbose);
}

/*
 * Options parsing the ring / net setup
 */
int main(int argc, char *argv[])
{
	struct io_uring ring;
	struct sigaction sa = { };
	const char *optstring;
	int opt, ret, fd;

	setlocale(LC_NUMERIC, "en_US");

	page_size = sysconf(_SC_PAGESIZE);
	if (page_size < 0) {
		perror("sysconf(_SC_PAGESIZE)");
		return 1;
	}

	pthread_mutex_init(&thread_lock, NULL);

	optstring = "m:d:S:s:b:f:H:r:p:n:B:N:T:w:t:M:R:u:c:C:q:a:x:z:6Vh?";
	while ((opt = getopt(argc, argv, optstring)) != -1) {
		switch (opt) {
		case 'm':
			recv_mshot = !!atoi(optarg);
			break;
		case 'S':
			sqpoll = !!atoi(optarg);
			break;
		case 'd':
			defer_tw = !!atoi(optarg);
			break;
		case 'b':
			buf_size = atoi(optarg);
			break;
		case 'n':
			nr_bufs = atoi(optarg);
			break;
		case 'u':
			send_ring = !!atoi(optarg);
			break;
		case 'c':
			rcv_bundle = !!atoi(optarg);
			break;
		case 'C':
			snd_bundle = !!atoi(optarg);
			break;
		case 'w':
			wait_batch = atoi(optarg);
			break;
		case 't':
			wait_usec = atoi(optarg);
			break;
		case 's':
			is_sink = !!atoi(optarg);
			break;
		case 'f':
			fixed_files = !!atoi(optarg);
			break;
		case 'H':
			host = strdup(optarg);
			break;
		case 'r':
			receive_port = atoi(optarg);
			break;
		case 'p':
			send_port = atoi(optarg);
			break;
		case 'B':
			bidi = !!atoi(optarg);
			break;
		case 'N':
			napi = !!atoi(optarg);
			break;
		case 'T':
			napi_timeout = atoi(optarg);
			break;
		case '6':
			ipv6 = true;
			break;
		case 'M':
			snd_msg = !!atoi(optarg);
			break;
		case 'z':
			snd_zc = !!atoi(optarg);
			break;
		case 'R':
			rcv_msg = !!atoi(optarg);
			break;
		case 'q':
			ring_size = atoi(optarg);
			break;
		case 'a':
			use_huge = !!atoi(optarg);
			break;
		case 'x':
			ext_stat = !!atoi(optarg);
			break;
		case 'V':
			verbose++;
			break;
		case 'h':
		default:
			usage(argv[0]);
			return 1;
		}
	}

	if (bidi && is_sink) {
		fprintf(stderr, "Can't be both bidi proxy and sink\n");
		return 1;
	}
	if (snd_msg && sqpoll) {
		fprintf(stderr, "SQPOLL with msg variants disabled\n");
		snd_msg = 0;
	}
	if (rcv_msg && rcv_bundle) {
		fprintf(stderr, "Can't use bundles with recvmsg\n");
		rcv_msg = 0;
	}
	if (snd_msg && snd_bundle) {
		fprintf(stderr, "Can't use bundles with sendmsg\n");
		snd_msg = 0;
	}
	if (snd_msg && send_ring) {
		fprintf(stderr, "Can't use send ring sendmsg\n");
		snd_msg = 0;
	}
	if (snd_zc && (send_ring || snd_bundle)) {
		fprintf(stderr, "Can't use send zc with bundles or ring\n");
		send_ring = snd_bundle = 0;
	}
	/*
	 * For recvmsg w/multishot, we waste some data at the head of the
	 * packet every time. Adjust the buffer size to account for that,
	 * so we're still handing 'buf_size' actual payload of data.
	 */
	if (rcv_msg && recv_mshot) {
		fprintf(stderr, "Adjusted buf size for recvmsg w/multishot\n");
		buf_size += sizeof(struct io_uring_recvmsg_out);
	}

	br_mask = nr_bufs - 1;

	fd = setup_listening_socket(receive_port, ipv6);
	if (is_sink)
		send_port = -1;

	if (fd == -1)
		return 1;

	atexit(show_stats);
	sa.sa_handler = sig_int;
	sa.sa_flags = SA_RESTART;
	sigaction(SIGINT, &sa, NULL);

	ret = init_ring(&ring, MAX_CONNS * 3);
	if (ret)
		return ret;

	printf("Backend: sqpoll=%d, defer_tw=%d, fixed_files=%d, "
		"is_sink=%d, buf_size=%d, nr_bufs=%d, host=%s, send_port=%d, "
		"receive_port=%d, napi=%d, napi_timeout=%d, huge_page=%d\n",
			sqpoll, defer_tw, fixed_files, is_sink,
			buf_size, nr_bufs, host, send_port, receive_port,
			napi, napi_timeout, use_huge);
	printf(" recv options: recvmsg=%d, recv_mshot=%d, recv_bundle=%d\n",
			rcv_msg, recv_mshot, rcv_bundle);
	printf(" send options: sendmsg=%d, send_ring=%d, send_bundle=%d, "
		"send_zerocopy=%d\n", snd_msg, send_ring, snd_bundle,
			snd_zc);

	return parent_loop(&ring, fd);
} 
```