<!--yml
category: 未分类
date: 2024-05-27 15:24:01
-->

# Satellites Are Becoming the New Cellphone Towers - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/satellite-cellphone-starlink](https://spectrum.ieee.org/satellite-cellphone-starlink)

Starlink sent and received texts over a 4G/LTE connection between mobile phones via its latest generation of satellites, called v2mini, for the [first time this month](https://api.starlink.com/public-files/DIRECT_TO_CELL_FIRST_TEXT_UPDATE.pdf), following similar projects from [Amazon](https://spectrum.ieee.org/tag/amazon), Apple, AST SpaceMobile, Huawei, and Lynk Global. Starlink—the satellite constellation operated by SpaceX—will offer text messaging to subscribers of at least eight different mobile-network operators around the world and may offer voice and data coverage without the need for the ground terminals its customers now use in “coming years,” Starlink’s U.S. partner T-Mobile said in a [statement](https://www.t-mobile.com/news/un-carrier/first-spacex-satellites-launch-for-breakthrough-direct-to-cell-service-with-t-mobile).

The Starlink achievement is the latest example of how satellites and cellular base stations are converging. A handful of companies are exploiting cheaper satellite fabrication and launch costs, as well as adapting existing technologies such as [beamforming](https://spectrum.ieee.org/5g-bytes-beamforming-explained), to bridge the several hundred kilometers between mobile phones and orbiting satellites. Among the many new wrinkles those companies have to iron out is the fact that for the first time, the towers themselves are the mobile component of the network: Low Earth orbit (LEO) satellites move at tens of thousands of kilometers an hour, so they have little time to communicate with any one mobile phone on the Earth’s surface.

The companies competing to solve these problems have so far sent and received text messages on conventional phones via a commercial satellite (Huawei/China Telecom; Lynk Global; Apple/Globalstar) and performed voice and data calls over [5G](https://spectrum.ieee.org/tag/5g) via an experimental satellite (AST SpaceMobile) as *[IEEE Spectrum](https://spectrum.ieee.org/)*[has reported.](https://spectrum.ieee.org/5g-satellite-2665870437) Investors have taken notice: Lynk Global is [going public](https://lynk.world/news/lynk-signs-letter-of-intent-to-become-publicly-listed-leading-satellite-to-phone-company-through-a-business-combination-with-slam-corp/) in a deal that values the company at up to US $800 million while AT&T, [Google](https://spectrum.ieee.org/tag/google), and Vodafone recently invested in AST SpaceMobile, which has a [market capitalization of $674.6 million](https://www.google.com/finance/quote/ASTS:NASDAQ).

“I recall many discussions 10 years ago where the mobile operators told the satellite people, ‘Your price points are way too high.’ This has totally changed due to more availability of the technology, more agile development, and a different approach to failure.” **—Andreas Knopp, University of the Bundeswehr**

Until very recently, satellites could not connect to mobile phones hundreds of kilometers below. The sort of satellite phones people took on expeditions to more remote places have chunky antennas, require clear lines of sight to multiple satellites, and take a while to acquire a signal. Integrating terrestrial and satellite cellular networks isn’t as easy as moving between cell towers and handing off the signal from one to the next.

Indeed, the task is so difficult that one research group [built an experimental application](https://ieeexplore.ieee.org/document/10199206) to help an Internet-connected livestock truck equipped with its own computer switch to an onboard Starlink ground station when the truck loses the cellular network signal. Achieving a seamless integration of terrestrial and nonterrestrial networks is the ultimate goal, says study coauthor [Melisa López](https://vbn.aau.dk/en/persons/melisa-maria-lopez-lechuga), a wireless communications researcher at the University of Aalborg in Denmark.

Starlink doesn’t explain many details of its 4G connection, but existing commercial constellations reveal several of the building blocks of seamless satellite cellular connections, and researchers have published at least one promising lead for future constellations.

## Three Keys to Connecting Phones to Satellites

Instead of redesigning mobile phones to be more like satellite phones, companies are redesigning the satellite network to meet mobile phones more than halfway. They are making the antennas on the satellites much bigger in their scramble to turn satellites into cellphone towers. For example, [AST SpaceMobile’s first satellites](https://spectrum.ieee.org/satellite-cellphone) had antennas with surface areas of 64 square meters, followed by second generation satellites with 128 m² antennas, with plans for going up to 400 m². Starlink’s new v2mini satellite antennas are 6.21 m², but Starlink plans even larger cellular-compatible satellites that it will launch when its larger Starship rocket is available.

Companies are also making their satellites more like cellphone towers by flying them lower than before. For the first few decades of the space age, communications satellites were inserted into geosynchronous orbits much higher above the Earth, where they could cover a large portion of the planet’s surface for a relatively long period of time. However, those satellites handled far fewer devices than exist today.

The advent of smaller and cheaper satellites and cheaper launch costs in the last decade or so have enabled business models that rely on many cheaper satellites flown in low Earth orbit. These new satellites won’t last as long but will be better able to detect the weak signals from mobile phones on the surface and handle their growing traffic.

SpaceX’s Starlink network can connect directly to an off-the-shelf cellphone, potentially eliminating the need for bulky satellite phones.Starlink

“I recall many discussions 10 years ago where the mobile operators told the satellite people, ‘Your price points are way too high.’ This has totally changed due to more availability of the technology, more agile development, and a different approach to failure,” says [Andreas Knopp](https://www.unibw.de/satcom/chair/team/andreas-knopp), a signal processing engineer at the University of the Bundeswehr in Munich.

Another contributor is improved beamforming, which is how a transmitting device calculates the best way to direct its signal to reach a particular recipient, without interfering with other recipients. That may involve bouncing a signal off a building or mountainside, from terrestrial towers, or it may involve precise targeting of a narrow, fast-moving signal, from a satellite moving tens of thousands of kilometers per hour.

More sophisticated beamforming can involve sending the same signals from multiple antennas so that the signal’s reinforce each other, a bit like when sound waves harmonize. Anyone who has tuned a home speaker system to provide the best sound at the point of a particular couch has done beamforming, probably with the help of sophisticated software in the background.

In the future, it may be worth spreading the task of beamforming across even more satellites than now, write the authors of [a pair](https://ieeexplore.ieee.org/abstract/document/10355084) of [recent papers](https://ieeexplore.ieee.org/document/10286242). A scenario floated in one of the studies is the use of more than two dozen tiny satellites flying in close formation to replicate the work done today by one cellular-compatible satellite. “Each of these satellites is now independent, with its own components. The main aspect is this synchronization algorithm, which has to align frequency, phase, and the time for the signals to arrive coherently,” says [Diego Tuzi](https://www.unibw.de/satcom/chair/team/diego-tuzi), a graduate student studying signal processing at the University of the Bundeswehr who, together with Knopp, is a coauthor on one of those recent papers.

For now, what Starlink and its competitors are offering is very little, though it is a necessary step. “It’s better than nothing,” says [Thomas Delamotte](https://www.unibw.de/space-en/members/dr-thomas-delamotte), another coauthor on the recent paper and a signal processing engineer at the University of the Bundeswehr, “but if you want to have a long-term perspective we will need new approaches to make [6G](https://spectrum.ieee.org/tag/6g) ubiquitous.”

From Your Site Articles

Related Articles Around the Web