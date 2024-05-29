<!--yml
category: 未分类
date: 2024-05-27 14:47:06
-->

# electromagnetism - Can tin foil hats block anything? - Physics Stack Exchange

> 来源：[https://physics.stackexchange.com/questions/208516/can-tin-foil-hats-block-anything](https://physics.stackexchange.com/questions/208516/can-tin-foil-hats-block-anything)

> Can tin foil hats actually block anything?

*Anything*? Sure. As already noted by Daniel Griscom's answer, tin foil can block several "things" including rain, alpha rays, and electromagnetic radiation with small enough wave length that the radiation cannot diffract around the edges of the hat.

> If they can, what frequencies?

Since you mention frequency let's really focus on electromagnetic radiation. Foil, being a metal, is very good at reflecting electromagnetic radiation. You can see this quite easily by looking at a sheet of aluminum/tin foil: it is shiny and you may even see a reflection of yourself. Physically this happens because electrons can move easily within the foil. Electromagnetic radiation impinging on the foil causes these electrons to move, and that motion produces new electromagnetic radiation which generates new radiation which destructively interferes with the original incoming radiation. This leads to cancellation of the *incoming* wave so that it does not penetrate the foil, and creation of a new *outgoing* wave which is the reflection you see.

Now this isn't the whole story. Radiation with a wavelength at about the same size as the hat actually winds up resonating inside the hat. In an effect similar to a vibration of a violin/guitar string, radiation that fits nicely inside the metal cavity formed by the hat actually winds up creating electromagnetic fields which are *bigger* than the incoming field. You can get a rough idea of how this works on the Wikipedia page for [resonance](https://en.wikipedia.org/wiki/Resonance). It's the same effect as when you blow gently over a bottle and hear a loud hum sound.

> Is there any research into tin or aluminum foil and radio blocking or amplifying abilities when shaped into a hat?

Yes, there is a [sort of famous study from MIT](http://web.archive.org/web/20100708230258/http://people.csail.mit.edu/rahimi/helmet/) where a student looked into this. They found the resonance effect mentioned above actually made the electromagnetic field strength inside that hats larger:

> *"For all helmets, we noticed a 30 db amplification at 2.6 Ghz and a 20 db amplification at 1.2 Ghz, regardless of the position of the antenna on the cranium. In addition, all helmets exhibited a marked 20 db attenuation at around 1.5 Ghz, with no significant attenuation beyond 10 db anywhere else."*

Note that resonance is not the same thing as amplification. Amplification means an increase in power; you can only get that if you have a power source, as in a radio receiver which amplifies the signal coming in over the air waves. Resonance is an effect wherein (at least in this case) the incoming flow of energy winds up being sort of piled up into the hat so that the fields in the hat are larger than the fields coming in. It doesn't add new energy to the system, it just concentrates it in the hat.

> If they really don't do anything, what would be better? Radio blocking in a hat design that is, not a Faraday cage suit (with matching tie).

Radio waves have pretty big wave length. For example, a 100 MHz wave has a length of 3 meters. If you want to block that out you need something like a [Faraday cage](https://en.wikipedia.org/wiki/Faraday_cage) with holes a substantial fraction smaller than that wave length.

P.S. The conclusions in that MIT study are hilarious and I recommend reading it.