<!--yml
category: 未分类
date: 2024-05-27 14:34:03
-->

# 1.55 R⊕ habitable-zone planet hosted by TOI-715, an M4 star near the ecliptic South Pole | Monthly Notices of the Royal Astronomical Society | Oxford Academic

> 来源：[https://academic.oup.com/mnras/article/527/1/35/7172075](https://academic.oup.com/mnras/article/527/1/35/7172075)

## ABSTRACT

A new generation of observatories is enabling detailed study of exoplanetary atmospheres and the diversity of alien climates, allowing us to seek evidence for extraterrestrial biological and geological processes. Now is therefore the time to identify the most unique planets to be characterized with these instruments. In this context, we report on the discovery and validation of TOI-715 b, a |$R_{\rm b}=1.55\pm 0.06\rm R_{\oplus }$| planet orbiting its nearby (42 pc) M4 host (TOI-715/TIC 271971130) with a period |$P_{\rm b} = 19.288004_{-0.000024}^{+0.000027}$| d. TOI-715 b was first identified by *TESS* and validated using ground-based photometry, high-resolution imaging and statistical validation. The planet’s orbital period combined with the stellar effective temperature |$T_{\rm eff}=3075\pm 75~\rm K$| give this planet an installation |$S_{\rm b} = 0.67_{-0.20}^{+0.15}~\rm S_\oplus$|⁠, placing it within the most conservative definitions of the habitable zone for rocky planets. TOI-715 b’s radius falls exactly between two measured locations of the M-dwarf radius valley; characterizing its mass and composition will help understand the true nature of the radius valley for low-mass stars. We demonstrate TOI-715 b is amenable for characterization using precise radial velocities and transmission spectroscopy. Additionally, we reveal a second candidate planet in the system, TIC 271971130.02, with a potential orbital period of |$P_{02} = 25.60712_{-0.00036}^{+0.00031}$| d and a radius of |$R_{02} = 1.066\pm 0.092\, \rm R_{\oplus }$|⁠, just inside the outer boundary of the habitable zone, and near a 4:3 orbital period commensurability. Should this second planet be confirmed, it would represent the smallest habitable zone planet discovered by *TESS* to date.

## 1 INTRODUCTION

At long last the era of *JWST* has arrived, and with it the age of detailed exoplanetary atmospheric characterization (The *JWST* Transiting Exoplanet Community Early Release Science Team 2023). This achievement was unlocked just as the community hit another significant milestone: the discovery of the 5000th planet beyond the solar system.¹ This ever-growing sample combined with the might of *JWST* is now set to deepen our understanding of planets, including those in our solar system, on a sub-population level.

One sub-population of particularly enduring interest consists of small, potentially habitable planets orbiting cool M-type stars. With current instrumentation, M dwarfs represent our best hope of finding temperate terrestrial planets, the best example being the seven Earth-sized planets of the TRAPPIST-1 system orbiting an M8V type star (Gillon et al. 2017). The reduced radii of M dwarfs enable small planets to produce large transit depths; this combined with the shorter orbital periods of these more compact systems makes photometric monitoring from ground and space-based facilities considerably more feasible (e.g. Nutzman & Charbonneau 2008; Reiners et al. 2018; Sebastian et al. 2021; Triaud 2021). In the context of atmospheric characterization by transmission spectroscopy, bright, nearby M dwarfs are ideal planetary hosts as small temperate planets will transit frequently, enabling high signal-to-noise detections of atmospheric features with fewer hours of telescope time (Charbonneau & Deming 2007; Dressing & Charbonneau 2015; Morley et al. 2017). Additionally, these low mass stars appear more likely to host small planets from a theoretical and observational point of view (Montgomery & Laughlin 2009; Bonfils et al. 2013; Dressing & Charbonneau 2013; Mulders, Pascucci & Apai 2015; Alibert & Benz 2017; He, Triaud & Gillon 2017; Sabotta et al. 2021).

The so-called ‘habitable zone’ is a circumstellar region where an Earth-like planet could sustain liquid water on its surface (Kasting, Whitmire & Reynolds 1993). With sometimes contradictory definitions of its boundaries, which depend on stellar spectral type, planetary albedo, mass and even cloud cover (Underwood, Jones & Sleep 2003; Kopparapu et al. 2014; Ramirez & Kaltenegger 2016), it can be challenging to categorise planets using this metric. However, the most widely applicable and conservative definition comes from Kopparapu et al. (2013), stating that rocky planets receiving between |$0.42\,\mathrm{ and}\,0.842\, S_{\oplus }$| are in their star’s HZ, irrespective of all other factors.

The activity of the M-dwarf host stars themselves is an additional factor to consider in the habitability of planets orbiting M dwarfs, as the effect of their frequent flaring is not fully known (O’Malley-James & Kaltenegger 2017). Stellar flares could destroy a nascent atmosphere entirely (Davenport et al. 2016; Dong et al. 2017), or they might provide the energy needed to catalyse biological processes (Buccino, Lemarchand & Mauas 2007; Patel et al. 2015; Lingam & Loeb 2017; Rimmer et al. 2018). Additionally, we must look beyond our own terrestrial version of habitability, by exploring the possibility that biology might arise on water-worlds (Madhusudhan, Piette & Constantinou 2021) or temperate sub-Neptunes (Seager et al. 2021), as both of these may have different HZ boundaries.

We owe much of our current understanding of exoplanetary demographics to the *Kepler* mission (Borucki et al. 2010), and one of the most influential results to come out of statistical studies of the *Kepler* sample was the bimodal radius distribution of sub-Neptune sized planets, with a gap between |$1.5\,\mathrm{ and}\,2\, R_{\oplus }$| (Fulton et al. 2017; Van Eylen et al. 2018). The current best explanations for this bimodality are core-powered mass loss (Lopez & Fortney 2013; Ginzburg, Schlichting & Sari 2018; Gupta & Schlichting 2019), photo-evaporation (Owen & Wu 2013, 2017) and volatile-poor formation (Lee, Chiang & Ormel 2014; Venturini & Helled 2017). While the exact location of this so-called ‘radius valley’ depends on stellar mass among other things (Wu 2019; Gupta & Schlichting 2020), it is still unclear whether it is present around M dwarfs or not (Cloutier & Menou 2020). Therefore, planets with sizes within the radius valley (e.g. Cloutier et al. 2020, 2021; Luque et al. 2022) help us to understand the shape and depth of this gap around M dwarfs. A recent study by Luque & Pallé (2022), however, indicates that M-dwarf planets may have a density gap rather than a radius gap separating two populations of small planets (rocky and water worlds), and that these observations are also well explained by formation pathways that include disc-driven migration (Venturini et al. 2020).

At present *TESS* (Ricker et al. 2015) is providing the community with ample new planets to improve our understanding of exoplanetary demographics, including by populating the habitable zone. Notably, while several ‘habitable zone’ planets discovered by *TESS* have been confirmed (e.g. Gilbert et al. 2020; Vach et al. 2022), none yet have fallen within the conservative habitable zone as described by Kopparapu et al. (2013) – until now.

In this context, we report on the discovery of TOI-715 b, a small planet orbiting a nearby M4 star (TOI-715/TIC 271971130). The planet’s relatively long orbital period and cool host provide it with a mild installation, placing TOI-715 b comfortably within its star’s conservative habitable zone. We additionally find a second candidate in the system that, if confirmed, could be *TESS*’s smallest habitable zone planet discovered to date.

Our paper is organised as follows: we begin by characterizing the host star using its spectral energy distribution and reconnaissance spectroscopy in Section [2](#sec2). We then describe the identification of planetary candidates in the data, first by *TESS* followed by our own search, in Section [3](#sec3). In Section [4](#sec4), we describe our ground-based follow-up campaign and our procedures to validate the system, and in Section [5](#sec5) we detail our global analysis of all available data. Finally, we contextualize our results in Section [6](#sec6) and conclude in Section [7](#sec7).

## 2 STELLAR CHARACTERIZATION

TOI-715 (TIC 271971130) is a nearby (⁠|$\rm 42\, pc$|⁠; Bailer-Jones et al. 2021) M dwarf of spectral type M4\. It is a high-proper-motion target with right ascension 07:35:24.56 hms and declination −73:34:38.67 dms (J2000, epoch 2015.5), placing it within *TESS*’s continuous viewing zone (CVZ). As all our planetary information will be derived using the host star’s parameters, we begin in the section that follows by characterizing TOI-715\. All photometry and stellar parameters adopted for this work can be found in Table 1.

Table 1.

Stellar parameters adopted for this work.

| Designations . | TOI-715, TIC 271971130, 2MASS J07352425−7334388, APASS 33649915, Gaia DR2 5262666416118954368, UCAC4 083–012601, *WISE* J073524.46−733438.7 . |
| --- | --- |
| Parameter . | Value . | Source . |
| --- | --- | --- |
| α | 07:35:24.56 | Gaia Collaboration (2022) |
| δ | −73:34:38.67 | Gaia Collaboration (2022) |
| Distance | 42.46 ± 0.03 pc | Bailer-Jones et al. (2021) |
| μ[α] | &#124;$\rm 82.67\pm 0.02\, mas\, yr^{-1}$&#124; | Gaia Collaboration (2022) |
| μ[δ] | &#124;$\rm 9.92\pm 0.02\, mas\, yr^{-1}$&#124; | Gaia Collaboration (2022) |
| *RV* | &#124;$\rm +55.8\pm 2.7\, km\, s^{-1}$&#124; | Gaia Collaboration (2022) |
| *U* | &#124;$\rm +29.7\pm 0.6\, km\, s^{-1}$&#124; | This work |
| *V* | &#124;$\rm -54.8\pm 2.1\, km\, s^{-1}$&#124; | This work |
| *W* | &#124;$\rm +0.7\pm 0.9\, km\, s^{-1}$&#124; | This work |
| SpT | M4 | This work |
| *R*[⋆] | 0.240 ± 0.012*R*[⊙] | This work |
| *M*[⋆] | 0.225 ± 0.012M[⊙] | This work |
| T[eff] | 3075 ± 75 K | This work |
| log *g*[⋆] | 5.0 ± 0.2 | This work |
| &#124;$\rm [Fe/H]$&#124; |  + 0.09 ± 0.20 dex | This work (spectroscopy) |
|  | −0.25 ± 0.25 dex | This work (SED) |
| Age | 6.2&#124;$^{+3.2}_{-2.2}$&#124; Gyr | This work |
| *T* mag | 13.5308 ± 0.0073 | Stassun et al. (2019) |
| *B* mag | 18.14 ± 0.16 | Zacharias et al. (2013) |
| *V* mag | 16.24 ± 0.01 | Zacharias et al. (2013) |
| *G* mag | 14.8940 ± 0.0007 | Gaia Collaboration (2022) |
| *J* mag | 11.808 ± 0.024 | Cutri et al. (2003) |
| *H* mag | 11.264 ± 0.026 | Cutri et al. (2003) |
| *K* mag | 10.917 ± 0.019 | Cutri et al. (2003) |
| *W*1 mag | 10.753 ± 0.023 | Cutri et al. (2021) |
| *W*2 mag | 10.571 ± 0.020 | Cutri et al. (2021) |
| *W*3 mag | 10.387 ± 0.049 | Cutri et al. (2021) |
| *W*4 mag | > 8.92 | Cutri et al. (2021) |

| Designations . | TOI-715, TIC 271971130, 2MASS J07352425−7334388, APASS 33649915, Gaia DR2 5262666416118954368, UCAC4 083–012601, *WISE* J073524.46−733438.7 . |
| --- | --- |
| Parameter . | Value . | Source . |
| --- | --- | --- |
| α | 07:35:24.56 | Gaia Collaboration (2022) |
| δ | −73:34:38.67 | Gaia Collaboration (2022) |
| Distance | 42.46 ± 0.03 pc | Bailer-Jones et al. (2021) |
| μ[α] | &#124;$\rm 82.67\pm 0.02\, mas\, yr^{-1}$&#124; | Gaia Collaboration (2022) |
| μ[δ] | &#124;$\rm 9.92\pm 0.02\, mas\, yr^{-1}$&#124; | Gaia Collaboration (2022) |
| *RV* | &#124;$\rm +55.8\pm 2.7\, km\, s^{-1}$&#124; | Gaia Collaboration (2022) |
| *U* | &#124;$\rm +29.7\pm 0.6\, km\, s^{-1}$&#124; | This work |
| *V* | &#124;$\rm -54.8\pm 2.1\, km\, s^{-1}$&#124; | This work |
| *W* | &#124;$\rm +0.7\pm 0.9\, km\, s^{-1}$&#124; | This work |
| SpT | M4 | This work |
| *R*[⋆] | 0.240 ± 0.012*R*[⊙] | This work |
| *M*[⋆] | 0.225 ± 0.012M[⊙] | This work |
| T[eff] | 3075 ± 75 K | This work |
| log *g*[⋆] | 5.0 ± 0.2 | This work |
| &#124;$\rm [Fe/H]$&#124; |  + 0.09 ± 0.20 dex | This work (spectroscopy) |
|  | −0.25 ± 0.25 dex | This work (SED) |
| Age | 6.2&#124;$^{+3.2}_{-2.2}$&#124; Gyr | This work |
| *T* mag | 13.5308 ± 0.0073 | Stassun et al. (2019) |
| *B* mag | 18.14 ± 0.16 | Zacharias et al. (2013) |
| *V* mag | 16.24 ± 0.01 | Zacharias et al. (2013) |
| *G* mag | 14.8940 ± 0.0007 | Gaia Collaboration (2022) |
| *J* mag | 11.808 ± 0.024 | Cutri et al. (2003) |
| *H* mag | 11.264 ± 0.026 | Cutri et al. (2003) |
| *K* mag | 10.917 ± 0.019 | Cutri et al. (2003) |
| *W*1 mag | 10.753 ± 0.023 | Cutri et al. (2021) |
| *W*2 mag | 10.571 ± 0.020 | Cutri et al. (2021) |
| *W*3 mag | 10.387 ± 0.049 | Cutri et al. (2021) |
| *W*4 mag | > 8.92 | Cutri et al. (2021) |

Table 1.

Stellar parameters adopted for this work.

| Designations . | TOI-715, TIC 271971130, 2MASS J07352425−7334388, APASS 33649915, Gaia DR2 5262666416118954368, UCAC4 083–012601, *WISE* J073524.46−733438.7 . |
| --- | --- |
| Parameter . | Value . | Source . |
| --- | --- | --- |
| α | 07:35:24.56 | Gaia Collaboration (2022) |
| δ | −73:34:38.67 | Gaia Collaboration (2022) |
| Distance | 42.46 ± 0.03 pc | Bailer-Jones et al. (2021) |
| μ[α] | &#124;$\rm 82.67\pm 0.02\, mas\, yr^{-1}$&#124; | Gaia Collaboration (2022) |
| μ[δ] | &#124;$\rm 9.92\pm 0.02\, mas\, yr^{-1}$&#124; | Gaia Collaboration (2022) |
| *RV* | &#124;$\rm +55.8\pm 2.7\, km\, s^{-1}$&#124; | Gaia Collaboration (2022) |
| *U* | &#124;$\rm +29.7\pm 0.6\, km\, s^{-1}$&#124; | This work |
| *V* | &#124;$\rm -54.8\pm 2.1\, km\, s^{-1}$&#124; | This work |
| *W* | &#124;$\rm +0.7\pm 0.9\, km\, s^{-1}$&#124; | This work |
| SpT | M4 | This work |
| *R*[⋆] | 0.240 ± 0.012*R*[⊙] | This work |
| *M*[⋆] | 0.225 ± 0.012M[⊙] | This work |
| T[eff] | 3075 ± 75 K | This work |
| log *g*[⋆] | 5.0 ± 0.2 | This work |
| &#124;$\rm [Fe/H]$&#124; |  + 0.09 ± 0.20 dex | This work (spectroscopy) |
|  | −0.25 ± 0.25 dex | This work (SED) |
| Age | 6.2&#124;$^{+3.2}_{-2.2}$&#124; Gyr | This work |
| *T* mag | 13.5308 ± 0.0073 | Stassun et al. (2019) |
| *B* mag | 18.14 ± 0.16 | Zacharias et al. (2013) |
| *V* mag | 16.24 ± 0.01 | Zacharias et al. (2013) |
| *G* mag | 14.8940 ± 0.0007 | Gaia Collaboration (2022) |
| *J* mag | 11.808 ± 0.024 | Cutri et al. (2003) |
| *H* mag | 11.264 ± 0.026 | Cutri et al. (2003) |
| *K* mag | 10.917 ± 0.019 | Cutri et al. (2003) |
| *W*1 mag | 10.753 ± 0.023 | Cutri et al. (2021) |
| *W*2 mag | 10.571 ± 0.020 | Cutri et al. (2021) |
| *W*3 mag | 10.387 ± 0.049 | Cutri et al. (2021) |
| *W*4 mag | > 8.92 | Cutri et al. (2021) |

| Designations . | TOI-715, TIC 271971130, 2MASS J07352425−7334388, APASS 33649915, Gaia DR2 5262666416118954368, UCAC4 083–012601, *WISE* J073524.46−733438.7 . |
| --- | --- |
| Parameter . | Value . | Source . |
| --- | --- | --- |
| α | 07:35:24.56 | Gaia Collaboration (2022) |
| δ | −73:34:38.67 | Gaia Collaboration (2022) |
| Distance | 42.46 ± 0.03 pc | Bailer-Jones et al. (2021) |
| μ[α] | &#124;$\rm 82.67\pm 0.02\, mas\, yr^{-1}$&#124; | Gaia Collaboration (2022) |
| μ[δ] | &#124;$\rm 9.92\pm 0.02\, mas\, yr^{-1}$&#124; | Gaia Collaboration (2022) |
| *RV* | &#124;$\rm +55.8\pm 2.7\, km\, s^{-1}$&#124; | Gaia Collaboration (2022) |
| *U* | &#124;$\rm +29.7\pm 0.6\, km\, s^{-1}$&#124; | This work |
| *V* | &#124;$\rm -54.8\pm 2.1\, km\, s^{-1}$&#124; | This work |
| *W* | &#124;$\rm +0.7\pm 0.9\, km\, s^{-1}$&#124; | This work |
| SpT | M4 | This work |
| *R*[⋆] | 0.240 ± 0.012*R*[⊙] | This work |
| *M*[⋆] | 0.225 ± 0.012M[⊙] | This work |
| T[eff] | 3075 ± 75 K | This work |
| log *g*[⋆] | 5.0 ± 0.2 | This work |
| &#124;$\rm [Fe/H]$&#124; |  + 0.09 ± 0.20 dex | This work (spectroscopy) |
|  | −0.25 ± 0.25 dex | This work (SED) |
| Age | 6.2&#124;$^{+3.2}_{-2.2}$&#124; Gyr | This work |
| *T* mag | 13.5308 ± 0.0073 | Stassun et al. (2019) |
| *B* mag | 18.14 ± 0.16 | Zacharias et al. (2013) |
| *V* mag | 16.24 ± 0.01 | Zacharias et al. (2013) |
| *G* mag | 14.8940 ± 0.0007 | Gaia Collaboration (2022) |
| *J* mag | 11.808 ± 0.024 | Cutri et al. (2003) |
| *H* mag | 11.264 ± 0.026 | Cutri et al. (2003) |
| *K* mag | 10.917 ± 0.019 | Cutri et al. (2003) |
| *W*1 mag | 10.753 ± 0.023 | Cutri et al. (2021) |
| *W*2 mag | 10.571 ± 0.020 | Cutri et al. (2021) |
| *W*3 mag | 10.387 ± 0.049 | Cutri et al. (2021) |
| *W*4 mag | > 8.92 | Cutri et al. (2021) |

### 2.1 Reconnaissance spectroscopy

We collected an optical spectrum of TOI-715 on 2022 January 7 (ut) using the Low Dispersion Survey Spectrograph on the 6.5-m *Magellan*ii (Clay) Telescope at Las Campanas Observatory in Chile. With its upgraded red-sensitive CCD (Stevenson et al. 2016), this instrument is now known as ‘LDSS-3C’. We used LDSS-3C in long-slit mode with the standard setup (fast readout speed, low gain, and 1 × 1 binning) and the VPH-Red grism, OG-590 blocking filter, and the 0|${_{.}^{\prime\prime}}$|75 × 0|${_{.}^{\prime}}$|4 center slit. This setup provides spectra covering 6000–10 000 Å with a resolution of *R* ∼ 1810, which is insufficient to produce a notable constraint on *v*sin *i*[⋆].

We observed TOI-715 during clear conditions with seeing of 0|${_{.}^{\prime\prime}}$|5\. We collected six exposures of 300 s each, totaling 30 min on-source, at an average airmass of 1.445\. Afterwards, we collected three 1-s exposures of the nearby F8 V standard star HR 2283 (Maiolino, Rieke & Rieke 1996) at an average airmass of 1.360\. At each pointing, we collected a 1-s HeNeAr arc lamp exposure and three, 10-s flat fields with the ‘quartz high’ lamp. We reduced the data with a custom, python-based pipeline, which includes bias removal, flat field correction, and spectral extraction. For wavelength calibration, the HeNeAr arc exposure was used, which was extracted similarly to the science spectrum. For flux calibration, we used the ratio of the spectrum of the flux standard HR 2283 with an F8 V template from Pickles (1998) to compute a relative flux correction; no correction was made to address telluric absorption. The final science spectrum has a maximum SNR per resolution element of 193 at 9202 Å and a mean SNR per resolution element of 124 in the 6000–10 000 Å range.

The reduced spectrum (Fig. 1) was analysed using kastredux.² We compared the spectrum to Sloan Digital Sky Survey templates from Kesseli et al. (2017), finding a best match to an M4 dwarf. This classification was verified through spectral classification indices from Reid, Hawley & Gizis (1995), Lépine, Rich & Shara (2003), and Riddick, Roche & Lucas (2007), all of which measure consistent spectral classifications of M4\. We evaluated the ζ metallicity index (Lépine, Rich & Shara 2007; Lépine et al. 2013), determining a value of 1.066 ± 0.002 which corresponds to a metallicity [Fe/H] = +0.09 ± 0.20 dex based on the empirical calibration of Mann et al. (2013). There is no significant evidence of Hα emission, with an equivalent width limit |*EW*| < 1.2 Å corresponding to log[10]*L*[H α]/*L*[bol] < −6.5 (Douglas et al. 2014). We also find no evidence of Li I absorption at 6708 Å (*EW* < 0.7 Å), ruling out a young age and substellar mass (Magazzu, Martin & Rebolo 1993).

Figure 1.

LDSS-3C red optical spectrum of TOI-715 (black line), compared to its best-fitting M4 template (Kesseli et al. 2017, magenta line). Spectra are normalized in the 7400–7500 Å region, and major absorption features are labeled, including regions of strong telluric absorption (⊕). The inset box shows a close-up of the region encompassing H α (6563 Å) and Li i (6708 Å) features, neither of which is detected in the data.

### 2.2 Spectral energy distribution

As an independent determination of the basic stellar parameters, we performed an analysis of the broadband spectral energy distribution (SED) of the star together with the *Gaia* DR3 parallax (with no systematic offset applied; see e.g. Stassun & Torres 2021), to determine an empirical measurement of the stellar radius, following the procedures described in Stassun & Torres (2016) and Stassun, Collins & Gaudi (2017); Stassun et al. (2018). We utilized the highest quality broadband photometry available, which were the *JHK[S]* magnitudes from *2MASS*, the *W*1–*W*3 magnitudes from *WISE*, and the *GG*[BP]*G*[RP] magnitudes from *Gaia*. Together, the available photometry spans the full stellar SED over the wavelength range 0.4–10 μm (see Fig. 2).

Figure 2.

Spectral energy distribution of TOI-715\. Red symbols represent the observed photometric measurements, where the horizontal bars represent the effective width of the passband. Blue symbols are the model fluxes from the best-fit NextGen atmosphere model (black).

We performed a fit using NextGen stellar atmosphere models (Hauschildt, Allard & Baron 1999), with the free parameters being the effective temperature (*T*[eff]), surface gravity (log *g*), and metallicity ([Fe/H]). The remaining free parameter is the extinction *A[V]*, which we fixed at zero due to the star’s proximity. The resulting fit (Fig. 2) has a reduced χ² of 1.3, with best-fitting *T*[eff] = 3075 ± 75 K, log *g* = 5.0 ± 0.2, [Fe/H]  = −0.25 ± 0.25\. Integrating the (unreddened) model SED gives the bolometric flux at Earth, *F*[bol] = 8.21 ± 0.19 × 10^(−11) erg s^(−1) cm^(−2). Taking the *F*[bol] and *T*[eff] together with the *Gaia* parallax, gives the stellar radius, *R*[⋆] = 0.240 ± 0.012 R[⊙]. In addition, we can estimate the stellar mass from the empirical relations of Mann et al. (2019), giving *M*[⋆] = 0.225 ± 0.012 M[⊙], which is consistent with the value of 0.21 ± 0.05 M[⊙] determined via log *g* and *R*[⋆].

### 2.3 Estimated age

The lack of detectable H α emission in its optical spectrum, and absence of any significant flaring activity in its *TESS* light curve, both suggest a relatively old age for TOI-715 given the persistent emission of most mid-type M dwarfs (Newton et al. 2017; Kiman et al. 2019). We estimated the age of TOI-715 by comparing its *UVW* velocities and metallicity to local stars with previously determined ages from isochrone fitting and metallicities from high resolution spectroscopy (cf. Burgasser & Mamajek 2017; Delrez et al. 2022). Our comparison sample was drawn from the GALAH Data Release 3 catalogue (Buder et al. 2021), for which ages are estimated using the Bayesian Stellar Parameter Estimation (bstep) code (Sharma et al. 2018). A matched GALAH sample was selected by requiring a distance ≤200 pc, and agreement with TOI-715 in individual *UVW* velocities to within 10 km s^(−1) and metallicity to within 0.20 dex. The age distributions of the full GALAH sample and matched stars are shown in Fig. 3. The latter shows a broad peak with a maximum probability at 7 Gyr; the median age of the distribution and 25 and 75  per cent quantiles yields an age estimate of 6.6|$^{+3.2}_{-2.2}$| Gyr. TOI-715 is likely to be as old or older than the Sun, consistent with its low degree of magnetic activity.

Figure 3.

Distribution of ages for all sources in GALAH Data Release 3 (light grey histogram) and those GALAH sources that match the *UVW* velocities and metallicity of TOI-715\. (dark grey histogram). The latter distribution is consistent with a median age of 6.6|$^{+3.2}_{-2.2}$| Gyr (green shaded region, 25 and 75 per cent quantiles), which we adopt as the age of TOI-715.

Analyses of ground-based and Kepler photometry by Newton et al. (2016) and McQuillan, Mazeh & Aigrain (2014), respectively, show that typical rotational periods of old M4 dwarfs (⁠|$0.4\!-\!0.5\, \rm M_\odot$|⁠) are roughly 15–50 d, most likely at the longer end of this range (and possibly beyond; longer periods in Newton et al. (2016) are ‘grade B’ detections due to low amplitude variations). This range falls within the detectable period range for the full *TESS* data set; however, the absence of detectable H α emission suggests a lack of significant spotting, so even if the period were within this range *TESS* measurements may not be sufficient to detect variability. In either case, the lack of H α emission and rotationally induced variability on a time-scale ≤50 d are both consistent with an old age.

## 3 IDENTIFICATION OF PLANETARY CANDIDATES

TOI-715 is in *TESS*’s southern CVZ, meaning that in principle, it would be observed in all southern sectors. In practice, it was not optimally placed on the CCD during sectors 28 and 38, so at the time of writing, there are 24 sectors of short-cadence (2 min) data for this target.³

In the following sections, we describe first the initial candidate identification in the *TESS* data, followed by our own search for further candidates.

### 3.1 *TESS* Candidate Identification

All *TESS* 2-minute cadence data are processed in the first instance by the spoc (Science Processing Operations Center) pipeline, presented in Jenkins et al. (2016). Data products are then available to the community in the form of Simple Aperture Photometry (SAP; Twicken et al. 2010; Morris et al. 2020) or Presearch Data Conditioning Simple Aperture Photometry (PDCSAP; Smith et al. 2012; Stumpe et al. 2012, 2014), the latter having been corrected for instrument systematics and for crowding effects. Data products are downloadable from the NASA Mikulski Archive for Space Telescopes (MAST) and available via the Lightkurve package (Lightkurve Collaboration 2018). All light curves are additionally searched by spoc for periodic transit-like signals and candidates with |$\rm SNR\, \gt \, 7.1$| are reported as threshold crossing events (TCEs).

TOI-715.01 was first reported as a candidate on 2019 May 24 (Guerrero et al. 2021) following a multisector transit search (Jenkins 2002; Jenkins et al. 2010, 2020) conducted on 2019 May 5 for sectors 1–9\. The candidate was already identified in the multi-sector search of Sectors 1–6 conducted on 2019 April 18, but did not at this time pass the necessary tests to be reported as a TOI. The transit signal was fitted with an initial limb-darkened transit model (Li et al. 2019) and subjected to a suite of diagnostic tests (Twicken et al. 2018) to help determine whether the signature is from an exoplanet. The transit signature passed all of the tests reported in the Data Validation report for Sectors 1–9⁴, and the difference image centroiding test located the source of the transit signal to within 3|${_{.}^{\prime\prime}}$|7 ± 3|${_{.}^{\prime\prime}}$|5 in the subsequent data validation reports for the multisector searches of Sectors 1–13 and 27–39.

At the time of the first reported TCE⁵, the *TESS* Input Catalog (TIC) in use was version 7, which had not yet incorporated the *Gaia*-DR2 data; as such, TOI-715 did not have a stellar radius and the planet candidate was reported as a |$\rm 7.1\, R_{\oplus }$| object with a period of |$\rm \sim 19.2\, d$|⁠. Following the update to the TICv8 (Stassun et al. 2019) the planet candidate’s radius estimate was revised, demonstrating that TOI-715.01 was likely a super-Earth and thus a high priority target.

spoc TCEs are subject to the TESS-ExoClass⁶ automated classifier that reduces the number of TCEs that undergo the manual TOI vetting procedure. TESS-ExoClass applies a series of tests that are similar to the *Kepler*Robovetter (Coughlin et al. 2016; Thompson et al. 2018). TOI 715.01 passes all the tests of TESS-ExoClass and has been placed in the Tier 1 (highest quality candidate) category, and has been classified as a Tier 1 candidate in the subsequent spoc multi-sector 1–13, 1–36, and 1–39 searches. Further details for the TOI assignment process are available in Guerrero et al. (2021).

We present the PDCSAP *TESS* light curves for all 24 sectors of 2-min cadence data in Fig. 4; the timings of the 29 transits are indicated with dark pink arrows.

Figure 4.

PDCSAP flux extracted from the short (2 min) cadence data of the 24 sectors (1–13, 27, 29–37, 39) in which TOI-715 was observed. Light grey point show the |$120\, \rm s$| exposures and the purple line shows the flux in |$3\, \rm min$| bins. The transit events of TOI-715.01 are shown with the dark pink arrows and the locations of possible transits of TIC 271971130.02 are indicated with green arrows; we note that individual transits are not readily apparent by eye.

### 3.2 Search for additional candidates

We make use of the custom pipeline sherlock⁷, presented in Pozuelos et al. (2020) and Demory et al. (2020), to search the *TESS* data for additional transiting candidates, as was done in Dransfield et al. (2022a). sherlock downloads all light curves from MAST and, using Wotan (Hippke et al. 2019), applies a bi-weight function with varying window sizes to detrend the data. Each detrended light curve and the original PDCSAP light curve are then searched for transit-like signals using Transit Least Squares (Hippke & Heller 2019). We use only the 2-min cadence in our candidate search, applying a Savitzky–Golay (SG) digital filter (Savitzky & Golay 1964) previously following the strategy described in Delrez et al. (2022)⁸ and we test 11 window sizes between 0.19 and 1.9 d when detrending with the bi-weight filter. We thus carry out our transit search on the PDCSAP light curve and the 11 detrended light curves and only consider signals with SNR > 7 for further investigation.

We recover TOI-715.01 at a period of |$\rm 19.29\, d$| in the first instance in all 12 light curves, and we find that the highest SNR and SDE are achieved in the PDCSAP flux without any detrending with Wotan.

We additionally find two other periodic signals in the data. Of these, the signal with highest SNR is at a period of |$\rm 25.61\, d$|⁠, putting it within 0.4  per cent of the first order 4:3 commensurability. The signal is detected in nine of the searched light curves with a maximum SNR of 13.81 and SDE of 11.6; it has a depth of |$\rm \sim \, 1\, ppt$|⁠, making this a |$\rm \sim \, 1.16\, R_{\oplus }$| candidate. In order to ensure this signal is not an artefact produced by the SG filter, we additionally search the untreated PDCSAP light curve and recover the signal with an SNR of 6.81\. We adopt this candidate as TIC 271971130.02⁹ and highlight the positions of transit events on Fig. 4 with dark green arrows. In Fig. 5, we present the *TESS* light curve folded on this signal, along with the Lomb–Scargle periodogram.

Figure 5.

Results of our search for additional candidates in the data, using Transit Least Squares as implemented by Sherlock. **Upper panel:**tls periodogram showing the detected 25.61-d period of TIC 271971130.02 and its harmonics. **Lower panel:***TESS* PDCSAP light curve phase-folded on this period, with a transit model overplotted. We note that the errorbars on the binned points are smaller than the markers, and that the light curve shown here is the one treated with the SG Filter.

Given the large amount of data in Fig. 4, it is not apparent by eye how much pre- and post-transit baseline each transit has. We therefore note that the total number of in-transit points for TIC 271971130.02 is 1371, while the total number of points before and after the transits are 1349 and 1369 respectively (counted up to 1 transit duration). Thus on average the pre-transit baseline is 98.4  per cent of a transit duration, while the post-transit baseline is 99.9  per cent.

The SPOC ran the Data Validation module at the ephemeris for the second signal and obtained an SNR of 5.5σ, and additionally identified other sub-threshold transit-like signals at higher SNR.

The second signal we recover has a period of |$\rm 7.17\, d$| and a depth of |$\lt 1\, \rm ppt$|⁠; it has a maximum SNR of 9.63 and SDE of 8.17\. The very low SNR of the signal makes it challenging to discern by eye whether or not the shape is consistent with that of a planetary transit. We also note that this signal is only found in four of the detrended light curves (with window sizes between 0.38–0.65 d, but not the PDCSAP light curve, indicating that the signal could be dependent on detrending. The duration of a transit at this period on a circular orbit is should be of order 0.065 d, and all tested windows are at least 3 × this duration. We therefore do not believe the detrending will have suppressed the transit in other light curves if it is real. When *TESS* returns to the southern skies in its second extended mission, the additional photometry will provide further insight into the nature of this signal.

It is important to note that the SNR and SDE yielded by sherlock are inherited from the transit least squares algorithm, which uses a simple estimation that cannot be compared with values provided by other pipelines, such as SPOC. Instead, these values are used internally in sherlock to compare the signals found and select the most prominent among them.

## 4 VETTING AND VALIDATION

In this section we describe the results of the multifacility follow-up campaign conducted between May 2020 and April 2022\. We begin with the high-resolution imaging observations, and then outline the photometric observations collected from five southern observatories. Finally, we describe how all our follow-up observations were used to validate the planetary nature of TOI-715.01 and TIC 271971130.02.

All follow-up observations are summarized in Table 2.

Table 2.

Summary of ground-based follow-up observations carried out for the validation of TOI-715.01 and TIC 271971130.02.

| Follow-up Observations . |
| --- |
| High resolution imaging . |
| --- |
| Observatory . | Filter . | Date . | Sensitivity limit . | Result . |
| --- | --- | --- | --- | --- |
| Gemini South | 562 nm | 2020 December 26 | Δ*m* = 4.69 at 0.5″ | No sources detected |
| Gemini South | 832 nm | 2020 December 26 | Δ*m* = 5.07 at 0.5″ | No sources detected |
| Photometric Follow-up |
| LCO-SAAO | *Sloan-i′* | 2020 May 13 | Ingress | Transit ruled out on or off target during the time covered |
| LCO-SAAO | *Sloan-i′* | 2020 October 15 | (.01) Ingress | Transit detected on target |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2021 February 26 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2021 April 25 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2021 April 25 | (.01) Full | Detection |
| LCO-CTIO | *Sloan-i′* | 2021 April 25 | (.01) Full | Detection |
| SSO-Callisto | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Io | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Europa | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Ganymede | *Sloan-r′* | 2021 September 26 | (.01) Gapped | Interruption due to weather – ingress detected |
| TRAPPIST-South | *I + *z*′* | 2021 September 26 | (.01) Full | Detection |
| OACC-CAO | *Sloan-i’2* | 2021 November 24 | (.01) Full | Detection |
| SSO-Callisto | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| SSO-Io | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| SSO-Ganymede | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2022 February 08 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2022 April 07 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2022 April 07 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2022 October 25 | (.02) Egress | Inconclusive (high airmass) |
| Spectroscopic Observations |
| Instrument | Wavelength range | Date | Number of spectra | Use |
| Magellan/LDSS3 | &#124;$380\!-\!1000~\rm nm$&#124; | 2022 January 06 | 1 | Stellar characterization |

| Follow-up Observations . |
| --- |
| High resolution imaging . |
| --- |
| Observatory . | Filter . | Date . | Sensitivity limit . | Result . |
| --- | --- | --- | --- | --- |
| Gemini South | 562 nm | 2020 December 26 | Δ*m* = 4.69 at 0.5″ | No sources detected |
| Gemini South | 832 nm | 2020 December 26 | Δ*m* = 5.07 at 0.5″ | No sources detected |
| Photometric Follow-up |
| LCO-SAAO | *Sloan-i′* | 2020 May 13 | Ingress | Transit ruled out on or off target during the time covered |
| LCO-SAAO | *Sloan-i′* | 2020 October 15 | (.01) Ingress | Transit detected on target |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2021 February 26 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2021 April 25 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2021 April 25 | (.01) Full | Detection |
| LCO-CTIO | *Sloan-i′* | 2021 April 25 | (.01) Full | Detection |
| SSO-Callisto | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Io | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Europa | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Ganymede | *Sloan-r′* | 2021 September 26 | (.01) Gapped | Interruption due to weather – ingress detected |
| TRAPPIST-South | *I + *z*′* | 2021 September 26 | (.01) Full | Detection |
| OACC-CAO | *Sloan-i’2* | 2021 November 24 | (.01) Full | Detection |
| SSO-Callisto | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| SSO-Io | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| SSO-Ganymede | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2022 February 08 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2022 April 07 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2022 April 07 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2022 October 25 | (.02) Egress | Inconclusive (high airmass) |
| Spectroscopic Observations |
| Instrument | Wavelength range | Date | Number of spectra | Use |
| Magellan/LDSS3 | &#124;$380\!-\!1000~\rm nm$&#124; | 2022 January 06 | 1 | Stellar characterization |

Table 2.

Summary of ground-based follow-up observations carried out for the validation of TOI-715.01 and TIC 271971130.02.

| Follow-up Observations . |
| --- |
| High resolution imaging . |
| --- |
| Observatory . | Filter . | Date . | Sensitivity limit . | Result . |
| --- | --- | --- | --- | --- |
| Gemini South | 562 nm | 2020 December 26 | Δ*m* = 4.69 at 0.5″ | No sources detected |
| Gemini South | 832 nm | 2020 December 26 | Δ*m* = 5.07 at 0.5″ | No sources detected |
| Photometric Follow-up |
| LCO-SAAO | *Sloan-i′* | 2020 May 13 | Ingress | Transit ruled out on or off target during the time covered |
| LCO-SAAO | *Sloan-i′* | 2020 October 15 | (.01) Ingress | Transit detected on target |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2021 February 26 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2021 April 25 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2021 April 25 | (.01) Full | Detection |
| LCO-CTIO | *Sloan-i′* | 2021 April 25 | (.01) Full | Detection |
| SSO-Callisto | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Io | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Europa | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Ganymede | *Sloan-r′* | 2021 September 26 | (.01) Gapped | Interruption due to weather – ingress detected |
| TRAPPIST-South | *I + *z*′* | 2021 September 26 | (.01) Full | Detection |
| OACC-CAO | *Sloan-i’2* | 2021 November 24 | (.01) Full | Detection |
| SSO-Callisto | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| SSO-Io | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| SSO-Ganymede | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2022 February 08 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2022 April 07 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2022 April 07 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2022 October 25 | (.02) Egress | Inconclusive (high airmass) |
| Spectroscopic Observations |
| Instrument | Wavelength range | Date | Number of spectra | Use |
| Magellan/LDSS3 | &#124;$380\!-\!1000~\rm nm$&#124; | 2022 January 06 | 1 | Stellar characterization |

| Follow-up Observations . |
| --- |
| High resolution imaging . |
| --- |
| Observatory . | Filter . | Date . | Sensitivity limit . | Result . |
| --- | --- | --- | --- | --- |
| Gemini South | 562 nm | 2020 December 26 | Δ*m* = 4.69 at 0.5″ | No sources detected |
| Gemini South | 832 nm | 2020 December 26 | Δ*m* = 5.07 at 0.5″ | No sources detected |
| Photometric Follow-up |
| LCO-SAAO | *Sloan-i′* | 2020 May 13 | Ingress | Transit ruled out on or off target during the time covered |
| LCO-SAAO | *Sloan-i′* | 2020 October 15 | (.01) Ingress | Transit detected on target |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2021 February 26 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2021 April 25 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2021 April 25 | (.01) Full | Detection |
| LCO-CTIO | *Sloan-i′* | 2021 April 25 | (.01) Full | Detection |
| SSO-Callisto | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Io | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Europa | *Sloan-r′* | 2021 September 26 | (.01) Full | Detection |
| SSO-Ganymede | *Sloan-r′* | 2021 September 26 | (.01) Gapped | Interruption due to weather – ingress detected |
| TRAPPIST-South | *I + *z*′* | 2021 September 26 | (.01) Full | Detection |
| OACC-CAO | *Sloan-i’2* | 2021 November 24 | (.01) Full | Detection |
| SSO-Callisto | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| SSO-Io | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| SSO-Ganymede | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2021 November 24 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2022 February 08 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2022 April 07 | (.01) Full | Detection |
| ExTrA | *&#124;$\rm 1.21\,\mu m$&#124;* | 2022 April 07 | (.01) Full | Detection |
| TRAPPIST-South | *I + *z*′* | 2022 October 25 | (.02) Egress | Inconclusive (high airmass) |
| Spectroscopic Observations |
| Instrument | Wavelength range | Date | Number of spectra | Use |
| Magellan/LDSS3 | &#124;$380\!-\!1000~\rm nm$&#124; | 2022 January 06 | 1 | Stellar characterization |

### 4.1 High resolution imaging

Close stellar companions (bound or in the line of sight) can confound derived exoplanet properties in a number of ways. The detected transit signal might be a false positive due to a background eclipsing binary and even real planet discoveries will yield incorrect stellar and exoplanet parameters if a close companion exists and is unaccounted for (e.g. Ciardi et al. 2015; Furlan & Howell 2017, 2020). Additionally, the presence of a close companion star leads to the non-detection of small planets residing within the same exoplanetary system (Lester et al. 2021). Approximately 25  per cent of M dwarfs are part of binary or multiple star systems (e.g. Cortes Contreras et al. 2015; Winters et al. 2019), though fewer than 5  per cent of known spectroscopic binaries include an M-dwarf primary star (Pourbaix et al. 2004).^(10) None the less, high resolution imaging provides crucial information towards our understanding of exoplanetary formation, dynamics, and evolution (Howell et al. 2021).

TOI-715 was observed on 2020 December 26 ut using the Zorro speckle instrument on the Gemini South 8-m telescope (Scott et al. 2021). Zorro provides simultaneous speckle imaging in two bands (⁠|$562~\rm nm$| and |$832~\rm nm$|⁠) with output data products including a reconstructed image with robust contrast limits on companion detection (see Howell & Furlan 2022). TOI-715 was found to be a single star to within the angular and brightness contrast levels achieved. Eight sets of 1000 × 0.06 s images were obtained and processed by our standard reduction pipeline (Howell et al. 2011). Fig. 6 shows our final contrast curves and the |$832~\rm nm$| reconstructed speckle image. These high-resolution observations revealed no companion star brighter than 5 magnitudes below that of the target star from the 8-m telescope diffraction limit (⁠|$20~\rm mas$|⁠) out to 1.2 arcsec. At the distance of TOI-715 (⁠|$42.4~\rm pc$|⁠) these angular limits correspond to spatial limits of 0.84 to |$50.9~\rm au$|⁠.

Figure 6.

Plot showing the 5σ speckle imaging contrast curves in both filters as a function of the angular separation out to 1.2 arcsec, the end of speckle coherence. The inset shows the reconstructed |$832~\rm nm$| image with a 1 arcsec scale bar. The star, TOI-715, was found to have no close companions to within the angular and brightness contrast levels achieved.

### 4.2 Photometric follow-up

#### 4.2.1 Las Cumbres Observatory

We used the Las Cumbres Observatory Global Telescopes (LCOGT; Brown et al. 2013) 1.0-m network nodes at South Africa Astronomical Observatory (SAAO) and Cerro Tololo Inter-American Observatory to observe three transits of TOI-715.01\. First and second transits were observed with LCO-SAAO on 2020 May 13 and 2020 October 15, and third was observed with LCO-CTIO on 2021 April 25\. All observations were carried out with the Sloan-*i*′ and an exposure time of 180 s.

Photometric brightness was measured using an uncontaminated target aperture of 4.3 arcsec (11 pixels). We used the *TESS* transit Finder, which is a customized version of the Tapir software package (Jensen 2013), to schedule our photometric time series. The 1.0-m telescopes are equipped with 4096 × 4096 SINISTRO cameras with an image scale of 0|${_{.}^{\prime\prime}}$|389 per pixel, resulting in a 26 arcmin × 26 arcmin field of view. The raw images were calibrated with the standard LCO banzai pipeline (McCully et al. 2018), and photometric data were extracted with astroimageJ (Collins et al. 2017).

The observation of 2020 May 13 only provided |$\sim 20~{{\ \rm per\ cent}}$| in-transit coverage as well as 1.2 h of pre-transit baseline. The transit was ruled out on or off target during the window covered by this observation.^(11) The subsequent observation of 2020 October 15 confirmed the transit event on target with the detection of an ingress that was 29 min late relative to the ephemeris. The observation of 2021 April 25 resulted in the detection of a ful transit.

#### 4.2.2 SPECULOOS-Southern Observatory

The SPECULOOS Southern Observatory (SSO) is comprised of four Ritchey-Chrétien 1.0 m-class telescopes installed at ESO Paranal in the Atacama desert (Delrez et al. 2018). Designed to hunt for small, habitable-zone planets orbiting ultra-cool stars (Sebastian et al. 2021), all four telescopes are equipped with a deep-depletion Andor CCD camera with 2048 × 2048 13-|$\rm{\mu m}$| pixels. Each telescope therefore has a field of view of |$12\,\mathrm{arcmin} \, \times \, 12\,\mathrm{arcmin}$| and a pixel scale of 0|${_{.}^{\prime\prime}}$|35 (Burdanov et al. 2018).

All SPECULOOS observations are scheduled using the python package spock.^(12) (Sebastian et al. 2021), and processed in the first instance by an automatic data reduction pipeline, described in detail in Murray et al. (2020). Successful observations of *TESS* targets are then reprocessed using prose, a publicly available python framework for processing astronomical images^(13) as described in Garcia et al. (2021, 2022). Images are calibrated and aligned before performing aperture photometry on the 500 brightest sources detected; prose then performs differential photometry (Broeg, Fernández & Neuhäuser 2005) on the target star to extract the light curve. Individual light curves are detrended for airmass, sky background and FWHM (full width at half maximum) using second order polynomials in time, and they are then modelled using exoplanet (Foreman-Mackey et al. 2021b).

We observed TOI-715.01 for the first time on the night of 2021 September 26 with all four telescopes simultaneously (Io, Europa, Ganymede, Callisto) using the *Sloan-r′* filter and |$\rm 120$|-s exposures. Cloudy skies during the transit caused some large systematics in three out of four telescopes, although this did not prevent the transit from being detected. The fourth telescope, Ganymede, closed during the weather alert, causing only the ingress to be detected.

We re-observed TOI-715.01 on the night of 2021 November 24 with three of our telescopes. We collected |$\rm 13$|-s exposures using the custom filter *I* + *z*′ (Sebastian et al. 2021). Observations were once again hampered by the untimely appearance of clouds, which affected the in-transit precision of our observations. Fortunately, the transit was still detected with all three instruments despite the overcast conditions.

#### 4.2.3 TRAPPIST-South

We observed four transits of TOI-715.01 with TRAPPIST-South (TS; Gillon et al. 2011; Jehin et al. 2011), located in ESO La Silla Observatory in Chile. This 0.6-m telescope is equipped with a FLI ProLine PL3041-BB camera and a back-illuminated CCD with a pixel size of 0|${_{.}^{\prime\prime}}$|64, providing a total field of view of |$22\,\mathrm{arcmin} \, \times \, 22\,\mathrm{arcmin}$| for an array of |$2048\, \times \, 2048$| pixels. TS is a Ritchey–Chrétien telescope with F/8 and is equipped with a German equatorial mount.

All four transits were observed with the custom *I* + *z*′ filter to maximize the photometric precision. The observations took place on 2021 April 25, 2021 September 26, 2021 November 24, and 2022 April 7, with exposure times of 50s, 70s, 120s, and 90s, respectively. We reduced the images using prose pipeline (Garcia et al. 2021, 2022) to extract optimal light curves. Individual light curves were then modeled using the exoplanet (Foreman-Mackey et al. 2021b) package and detrended using a second order polynomial of airmass, FWHM and sky background. None of the observations suffered weather losses.

We additionally observed TOI-715 on 2022 October 25 during a partial transit of the second candidate (.02) but the target was too low during the event. The observation was therefore inconclusive.

#### 4.2.4 ExTrA

ExTrA (Bonfils et al. 2015) is a near-infrared (0.85–1.55 |$\rm \,\mu m$|⁠) multi-object spectrograph fed by three 0.6 m telescopes located at La Silla observatory. We observed four full transits of TOI 715.01 on 2021 February 26, 2021 April 25, 2022 February 8, and 2022 April 7\. We observed the first three transits using two telescopes, and the fourth using three; all observations were carried out with 8 arcsec aperture fibers. For both nights, we used the spectrograph’s low resolution mode (*R* ∼ 20) and 60s exposures. Five fibre positioners are used at the focal plane of each telescope to collect light from the target and four comparison stars. As comparison stars, we also observed 2MASS J07381369−7347351, 2MASS J07410628−7341297, 2MASS J07393394−7330535, and 2MASS J07372328−7321411 with *J*-magnitude (Skrutskie et al. 2006) and *T*[eff] (Gaia Collaboration 2021) similar to TOI-715\. The resulting ExTrA data were analyzed with custom data reduction software, detailed in Cointepas et al. (2021).

#### 4.2.5 OACC-CAO

We observed a transit of TOI-715.01 with Campo Catino Austral Observatory (OACC-CAO), located in El Sauce Observatory in the Atacama desert in Chile, on the night of 2021 November 24\. The telescope is a |$\rm 0.6\, m$| Planewave CDK 24 arcsec with a Planewave L600 mount, equipped with an FLI filter wheel with *Sloan* filters and an FLI PL16803 camera with 4096 × 4096 |$9~\rm \,\mu m$| pixels. It has a field of view of 32 arcmin and a pixel scale of 0|${_{.}^{\prime\prime}}$|48.

TOI-715.01 was observed with the *Sloan-i’2* filter using 180s exposures. The images were reduced using AstroImageJ (Collins et al. 2017) and we detect the full transit in an uncontaminated aperture.

### 4.3 Statistical validation

We make use of the statistical validation package triceratops^(14) (Giacalone & Dressing 2020; Giacalone et al. 2021) to assess the probability of the planet hypothesis for TOI-715.01 and TIC 271971130.02\. Triceratops calculates the flux contribution from nearby stars to check if any could be responsible for the transit signal. It then calculates the relative probabilities of a range of transiting planet (TP) and eclipsing binary (EB) scenarios using light curve models fitted to the phase-folded photometry. In the first instance we find that the false positive probabilities (FPP) are 0.403 and 0.592 for candidates .01 and .02, respectively when examining the *TESS* data, while the threshold for validation is 0.015\. For candidate .01, we make use of our follow-up photometry by folding in the four TRAPPIST-South light curves. We choose these as they were observed with the custom *I + z’* filter (which is very similar to the *TESS* bandpass) and all four are at different epochs.

With this phase-folded subset of our follow-up photometry, we find that the false positive probability reduces to 0.0107, placing it below the threshold for statistical validation. To similarly reduce the FPP for TIC 271971130.02, we will need to collect ground-based photometry to confirm the event on-target or rule out events on nearby stars (such an observation was scheduled from Hazelwood Observatory but cancelled due to bad weather).

#### 4.3.1 Statistical validation conclusions

With a FPP of 0.0107 TOI-715.01 is now below the customary threshold for statistical validation (0.015; Giacalone et al. 2021); we therefore consider the planet validated and refer to it as TOI-715 b. TIC 271971130.02 does not yet meet the criterion for statistical validation, and is therefore classified as a ‘likely transiting planet’ until more evidence such as ground-based photometry is collected.

## 5 GLOBAL PHOTOMETRIC ANALYSIS

We carried out a global photometric analysis of the *TESS* photometry and the data sets described in Section [4.3](#sec4-3) using allesfitter (Günther & Daylan 2019, 2021), a flexible and publicly available python-based inference package. We exclude all partial transits, as well as the transit obtained from OACC (due to large scatter in the data) from our analysis, leaving 15 full transits from our ground-based facilities. Allesfitter generates light curve models using Ellc (Maxted 2016), and Gaussian Process (GP) models using Celerite (Foreman-Mackey et al. 2017). The best fitting models are then chosen using either a nested sampling algorithm via Dynesty (Speagle 2020), or MCMC sampling with Emcee (Foreman-Mackey et al. 2013). For all modeling in this paper, we make use of the nested sampling algorithm as it calculates the Bayesian evidence at each step in the sampling; this allows us to compare the Bayesian evidence for different models by calculating the Bayes Factor (Kass & Raftery 1995). All prior distributions and their bounds can be found in Table 3.

Table 3.

Priors used in our fit, along with fitted and derived parameters. Uniform priors are indicated as |$\rm \mathcal {U}(lower~bound,~upper~bound)$| and normal priors are indicated as |$\rm \mathcal {N}(mean,~standard~deviation)$|⁠. ^⋆Equillibrium temperature is calculated assuming an albedo of 0.3 and emissivity of 1.

|  | TOI-715 b | TIC 271971130.02 |  |
| **Fit parametrization and priors** |
| Transit depth; *R*[p]/*R*[⋆] | &#124;$\mathcal {U}(0.01, 0.1)$&#124; | &#124;$\mathcal {U}(0.01, 0.1)$&#124; |  |
| Inverse scaled semimajor axis; (*R*[⋆] + *R*[p])/*a* | &#124;$\mathcal {U}(0.01, 0.05)$&#124; | &#124;$\mathcal {U}(0.008, 0.04)$&#124; |  |
| Orbital inclination; cos *i* | &#124;$\mathcal {U}(0.000, 0.04)$&#124; | &#124;$\mathcal {U}(0.000, 0.04)$&#124; |  |
| Transit epoch; *T*[0] (BJD) | &#124;$\mathcal {U}(2\, 458\, 327.3, 2\, 458\, 327.7)$&#124; | &#124;$\mathcal {U}(2\, 458\, 342.0, 2\, 458\, 342.4)$&#124; |  |
| Period; *P* (d) | &#124;$\mathcal {U}(19.0, 19.4)$&#124; | &#124;$\mathcal {U}(25.4, 25.8)$&#124; |  |
| **Limb-darkening coefficients** |
| *TESS*&#124;$\rm u_1$&#124; | 0.3322 ± 0.0013 | *TESS*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.413, 0.050)$&#124; |
| *TESS*&#124;$\rm u_2$&#124; | 0.3103 ± 0.0047 | *TESS*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.259, 0.050)$&#124; |
| *I + z*&#124;$\rm u_1$&#124; | 0.2942 ± 0.0014 | *I + z*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.453, 0.050)$&#124; |
| *I + z*&#124;$\rm u_2$&#124; | 0.3791 ± 0.0052 | *I + z*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.218, 0.050)$&#124; |
| *Sloan-i’*&#124;$\rm u_1$&#124; | 0.3863 ± 0.0018 | *Sloan-i’*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.532, 0.050)$&#124; |
| *Sloan-i’*&#124;$\rm u_2$&#124; | 0.3431 ± 0.0060 | *Sloan-i’*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.265, 0.050)$&#124; |
| *Sloan-r’*&#124;$\rm u_1$&#124; | 0.6408 ± 0.0032 | *Sloan-r’*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.768, 0.050)$&#124; |
| *Sloan-r’*&#124;$\rm u_2$&#124; | 0.2273 ± 0.0079 | *Sloan-r’*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.218, 0.050)$&#124; |
| *ExTra (1.2 μm)*&#124;$\rm u_1$&#124; | 0.2058 ± 0.0007 | *ExTra (1.2μm)*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.142, 0.050)$&#124; |
| *ExTra (1.2 μm)*&#124;$\rm u_2$&#124; | 0.1708 ± 0.0028 | *ExTra (1.2μm)*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.273, 0.050)$&#124; |
| **External priors** | **GP priors** |
| Stellar Mass; *M*[⋆] (⁠&#124;$\mathrm{{\rm M}_{\odot }}$&#124;⁠) | &#124;$\mathcal {N}(0.225, 0.012)$&#124; | Amplitude scale &#124;$\mathrm{GP \ln \sigma (flux)}$&#124; | &#124;$\mathcal {U}(-7, -7)$&#124; |
| Stellar Radius; *R*[⋆] (R[⊙]) | &#124;$\mathcal {N}(0.240, 0.012)$&#124; | Length scale &#124;$\mathrm{GP \ln \rho (flux)}$&#124; | &#124;$\mathcal {U}(-7, 2)$&#124; |
| Stellar Effective Temperature; *T*[eff] (K) | &#124;$\mathcal {N}(3075, 75)$&#124; |  |  |
| **Fitted parameters** | **Source** |
| *R*[p]/*R*[⋆] | 0.0618 ± 0.0017 | 0.0425 ± 0.0034 | 2-planet linear fit |
| (*R*[⋆] + *R*[p])/*a* | 0.01367 ± 0.00017 | &#124;$0.01129_{-0.00047}^{+0.00055}$&#124; | 2-planet linear fit |
| cos *i* | &#124;$0.00252_{-0.00032}^{+0.00030}$&#124; | &#124;$0.0048_{-0.0024}^{+0.0021}$&#124; | 2-planet linear fit |
| *T*[0] (BJD) | &#124;$2\, 459\, 002.63051_{-0.00074}^{+0.00070}$&#124; | &#124;$2\, 459\, 007.9879_{-0.0041}^{+0.0045}$&#124; | 2-planet linear fit |
| *P* (d) | &#124;$19.288004_{-0.000024}^{+0.000027}$&#124; | &#124;$25.60712_{-0.00036}^{+0.00031}$&#124; | 2-planet linear fit |
| **Fitted GP hyperparameters** | **Source** |
| ExTrA 2 | &#124;$\mathrm{gp \ln \sigma }$&#124;&#124;$-3.77_{-0.19}^{+0.24}$&#124; | &#124;$\mathrm{gp \ln \rho }$&#124;&#124;$-2.73_{-0.17}^{+0.26}$&#124; | 2-planet linear fit |
| ExTrA 3 | &#124;$\mathrm{gp \ln \sigma }$&#124;&#124;$-4.60_{-0.19}^{+0.22}$&#124; | &#124;$\mathrm{gp \ln \rho }$&#124;&#124;$-2.77_{-0.15}^{+0.24}$&#124; | 2-planet linear fit |
| **Derived parameters** | **Source** |
| Companion radius; *R*[p] (R[⊕]) | 1.550 ± 0.064 | 1.066 ± 0.092 | 2-planet linear fit |
| Instellation; *S*[p] (S[⊕]) | &#124;$0.67_{-0.20}^{+0.15}$&#124; | &#124;$0.48_{-0.17}^{+0.12}$&#124; | 2-planet linear fit |
| Semi-major axis; *a* (au) | 0.0830 ± 0.0027 | 0.0986 ± 0.0054 | 2-planet linear fit |
| Inclination; *i* (deg) | &#124;$89.856_{-0.017}^{+0.018}$&#124; | &#124;$89.72_{-0.12}^{+0.14}$&#124; | 2-planet linear fit |
| Impact parameter; *b* | &#124;$0.195_{-0.024}^{+0.023}$&#124; | &#124;$0.44_{-0.22}^{+0.18}$&#124; | 2-planet linear fit |
| Total transit duration; *T*[tot] (h) | 1.980 ± 0.025 | &#124;$1.99_{-0.21}^{+0.14}$&#124; | 2-planet linear fit |
| Full-transit duration; *T*[full] (h) | 1.741 ± 0.022 | &#124;$1.79_{-0.25}^{+0.15}$&#124; | 2-planet linear fit |
| Equilibrium temperature^⋆; *T*[eq] (K) | 234 ± 12 | 215 ± 12 | 2-planet linear fit |
| Transit depth *TESS*; δ[tr; TESS] (ppt) | &#124;$4.48_{-0.23}^{+0.26}$&#124; | &#124;$2.03_{-0.27}^{+0.31}$&#124; | 2-planet linear fit |
| Transit depth *i + z*; δ[tr; i + z] (ppt) | &#124;$4.48_{-0.48}^{+0.54}$&#124; | - | 2-planet linear fit |
| Transit depth *Sloan-i’*; &#124;$\delta _\mathrm{tr; Sloan-i^{\prime }}$&#124; (ppt) | &#124;$4.60_{-1.20}^{+1.50}$&#124; | - | 2-planet linear fit |
| Transit depth *Sloan-r’*; &#124;$\delta _\mathrm{tr; Sloan-r^{\prime }}$&#124; (ppt) | &#124;$4.81_{-0.90}^{+0.78}$&#124; | - | 2-planet linear fit |
| Transit depth *ExTra* (⁠&#124;$1.2~\rm \,\mu m$&#124;⁠); δ[tr; ExTra(1.2μm)] (ppt) | &#124;$4.20_{-0.96}^{+1.10}$&#124; | - | 2-planet linear fit |

|  | TOI-715 b | TIC 271971130.02 |  |
| **Fit parametrization and priors** |
| Transit depth; *R*[p]/*R*[⋆] | &#124;$\mathcal {U}(0.01, 0.1)$&#124; | &#124;$\mathcal {U}(0.01, 0.1)$&#124; |  |
| Inverse scaled semimajor axis; (*R*[⋆] + *R*[p])/*a* | &#124;$\mathcal {U}(0.01, 0.05)$&#124; | &#124;$\mathcal {U}(0.008, 0.04)$&#124; |  |
| Orbital inclination; cos *i* | &#124;$\mathcal {U}(0.000, 0.04)$&#124; | &#124;$\mathcal {U}(0.000, 0.04)$&#124; |  |
| Transit epoch; *T*[0] (BJD) | &#124;$\mathcal {U}(2\, 458\, 327.3, 2\, 458\, 327.7)$&#124; | &#124;$\mathcal {U}(2\, 458\, 342.0, 2\, 458\, 342.4)$&#124; |  |
| Period; *P* (d) | &#124;$\mathcal {U}(19.0, 19.4)$&#124; | &#124;$\mathcal {U}(25.4, 25.8)$&#124; |  |
| **Limb-darkening coefficients** |
| *TESS*&#124;$\rm u_1$&#124; | 0.3322 ± 0.0013 | *TESS*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.413, 0.050)$&#124; |
| *TESS*&#124;$\rm u_2$&#124; | 0.3103 ± 0.0047 | *TESS*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.259, 0.050)$&#124; |
| *I + z*&#124;$\rm u_1$&#124; | 0.2942 ± 0.0014 | *I + z*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.453, 0.050)$&#124; |
| *I + z*&#124;$\rm u_2$&#124; | 0.3791 ± 0.0052 | *I + z*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.218, 0.050)$&#124; |
| *Sloan-i’*&#124;$\rm u_1$&#124; | 0.3863 ± 0.0018 | *Sloan-i’*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.532, 0.050)$&#124; |
| *Sloan-i’*&#124;$\rm u_2$&#124; | 0.3431 ± 0.0060 | *Sloan-i’*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.265, 0.050)$&#124; |
| *Sloan-r’*&#124;$\rm u_1$&#124; | 0.6408 ± 0.0032 | *Sloan-r’*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.768, 0.050)$&#124; |
| *Sloan-r’*&#124;$\rm u_2$&#124; | 0.2273 ± 0.0079 | *Sloan-r’*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.218, 0.050)$&#124; |
| *ExTra (1.2 μm)*&#124;$\rm u_1$&#124; | 0.2058 ± 0.0007 | *ExTra (1.2μm)*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.142, 0.050)$&#124; |
| *ExTra (1.2 μm)*&#124;$\rm u_2$&#124; | 0.1708 ± 0.0028 | *ExTra (1.2μm)*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.273, 0.050)$&#124; |
| **External priors** | **GP priors** |
| Stellar Mass; *M*[⋆] (⁠&#124;$\mathrm{{\rm M}_{\odot }}$&#124;⁠) | &#124;$\mathcal {N}(0.225, 0.012)$&#124; | Amplitude scale &#124;$\mathrm{GP \ln \sigma (flux)}$&#124; | &#124;$\mathcal {U}(-7, -7)$&#124; |
| Stellar Radius; *R*[⋆] (R[⊙]) | &#124;$\mathcal {N}(0.240, 0.012)$&#124; | Length scale &#124;$\mathrm{GP \ln \rho (flux)}$&#124; | &#124;$\mathcal {U}(-7, 2)$&#124; |
| Stellar Effective Temperature; *T*[eff] (K) | &#124;$\mathcal {N}(3075, 75)$&#124; |  |  |
| **Fitted parameters** | **Source** |
| *R*[p]/*R*[⋆] | 0.0618 ± 0.0017 | 0.0425 ± 0.0034 | 2-planet linear fit |
| (*R*[⋆] + *R*[p])/*a* | 0.01367 ± 0.00017 | &#124;$0.01129_{-0.00047}^{+0.00055}$&#124; | 2-planet linear fit |
| cos *i* | &#124;$0.00252_{-0.00032}^{+0.00030}$&#124; | &#124;$0.0048_{-0.0024}^{+0.0021}$&#124; | 2-planet linear fit |
| *T*[0] (BJD) | &#124;$2\, 459\, 002.63051_{-0.00074}^{+0.00070}$&#124; | &#124;$2\, 459\, 007.9879_{-0.0041}^{+0.0045}$&#124; | 2-planet linear fit |
| *P* (d) | &#124;$19.288004_{-0.000024}^{+0.000027}$&#124; | &#124;$25.60712_{-0.00036}^{+0.00031}$&#124; | 2-planet linear fit |
| **Fitted GP hyperparameters** | **Source** |
| ExTrA 2 | &#124;$\mathrm{gp \ln \sigma }$&#124;&#124;$-3.77_{-0.19}^{+0.24}$&#124; | &#124;$\mathrm{gp \ln \rho }$&#124;&#124;$-2.73_{-0.17}^{+0.26}$&#124; | 2-planet linear fit |
| ExTrA 3 | &#124;$\mathrm{gp \ln \sigma }$&#124;&#124;$-4.60_{-0.19}^{+0.22}$&#124; | &#124;$\mathrm{gp \ln \rho }$&#124;&#124;$-2.77_{-0.15}^{+0.24}$&#124; | 2-planet linear fit |
| **Derived parameters** | **Source** |
| Companion radius; *R*[p] (R[⊕]) | 1.550 ± 0.064 | 1.066 ± 0.092 | 2-planet linear fit |
| Instellation; *S*[p] (S[⊕]) | &#124;$0.67_{-0.20}^{+0.15}$&#124; | &#124;$0.48_{-0.17}^{+0.12}$&#124; | 2-planet linear fit |
| Semi-major axis; *a* (au) | 0.0830 ± 0.0027 | 0.0986 ± 0.0054 | 2-planet linear fit |
| Inclination; *i* (deg) | &#124;$89.856_{-0.017}^{+0.018}$&#124; | &#124;$89.72_{-0.12}^{+0.14}$&#124; | 2-planet linear fit |
| Impact parameter; *b* | &#124;$0.195_{-0.024}^{+0.023}$&#124; | &#124;$0.44_{-0.22}^{+0.18}$&#124; | 2-planet linear fit |
| Total transit duration; *T*[tot] (h) | 1.980 ± 0.025 | &#124;$1.99_{-0.21}^{+0.14}$&#124; | 2-planet linear fit |
| Full-transit duration; *T*[full] (h) | 1.741 ± 0.022 | &#124;$1.79_{-0.25}^{+0.15}$&#124; | 2-planet linear fit |
| Equilibrium temperature^⋆; *T*[eq] (K) | 234 ± 12 | 215 ± 12 | 2-planet linear fit |
| Transit depth *TESS*; δ[tr; TESS] (ppt) | &#124;$4.48_{-0.23}^{+0.26}$&#124; | &#124;$2.03_{-0.27}^{+0.31}$&#124; | 2-planet linear fit |
| Transit depth *i + z*; δ[tr; i + z] (ppt) | &#124;$4.48_{-0.48}^{+0.54}$&#124; | - | 2-planet linear fit |
| Transit depth *Sloan-i’*; &#124;$\delta _\mathrm{tr; Sloan-i^{\prime }}$&#124; (ppt) | &#124;$4.60_{-1.20}^{+1.50}$&#124; | - | 2-planet linear fit |
| Transit depth *Sloan-r’*; &#124;$\delta _\mathrm{tr; Sloan-r^{\prime }}$&#124; (ppt) | &#124;$4.81_{-0.90}^{+0.78}$&#124; | - | 2-planet linear fit |
| Transit depth *ExTra* (⁠&#124;$1.2~\rm \,\mu m$&#124;⁠); δ[tr; ExTra(1.2μm)] (ppt) | &#124;$4.20_{-0.96}^{+1.10}$&#124; | - | 2-planet linear fit |

Table 3.

Priors used in our fit, along with fitted and derived parameters. Uniform priors are indicated as |$\rm \mathcal {U}(lower~bound,~upper~bound)$| and normal priors are indicated as |$\rm \mathcal {N}(mean,~standard~deviation)$|⁠. ^⋆Equillibrium temperature is calculated assuming an albedo of 0.3 and emissivity of 1.

|  | TOI-715 b | TIC 271971130.02 |  |
| **Fit parametrization and priors** |
| Transit depth; *R*[p]/*R*[⋆] | &#124;$\mathcal {U}(0.01, 0.1)$&#124; | &#124;$\mathcal {U}(0.01, 0.1)$&#124; |  |
| Inverse scaled semimajor axis; (*R*[⋆] + *R*[p])/*a* | &#124;$\mathcal {U}(0.01, 0.05)$&#124; | &#124;$\mathcal {U}(0.008, 0.04)$&#124; |  |
| Orbital inclination; cos *i* | &#124;$\mathcal {U}(0.000, 0.04)$&#124; | &#124;$\mathcal {U}(0.000, 0.04)$&#124; |  |
| Transit epoch; *T*[0] (BJD) | &#124;$\mathcal {U}(2\, 458\, 327.3, 2\, 458\, 327.7)$&#124; | &#124;$\mathcal {U}(2\, 458\, 342.0, 2\, 458\, 342.4)$&#124; |  |
| Period; *P* (d) | &#124;$\mathcal {U}(19.0, 19.4)$&#124; | &#124;$\mathcal {U}(25.4, 25.8)$&#124; |  |
| **Limb-darkening coefficients** |
| *TESS*&#124;$\rm u_1$&#124; | 0.3322 ± 0.0013 | *TESS*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.413, 0.050)$&#124; |
| *TESS*&#124;$\rm u_2$&#124; | 0.3103 ± 0.0047 | *TESS*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.259, 0.050)$&#124; |
| *I + z*&#124;$\rm u_1$&#124; | 0.2942 ± 0.0014 | *I + z*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.453, 0.050)$&#124; |
| *I + z*&#124;$\rm u_2$&#124; | 0.3791 ± 0.0052 | *I + z*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.218, 0.050)$&#124; |
| *Sloan-i’*&#124;$\rm u_1$&#124; | 0.3863 ± 0.0018 | *Sloan-i’*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.532, 0.050)$&#124; |
| *Sloan-i’*&#124;$\rm u_2$&#124; | 0.3431 ± 0.0060 | *Sloan-i’*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.265, 0.050)$&#124; |
| *Sloan-r’*&#124;$\rm u_1$&#124; | 0.6408 ± 0.0032 | *Sloan-r’*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.768, 0.050)$&#124; |
| *Sloan-r’*&#124;$\rm u_2$&#124; | 0.2273 ± 0.0079 | *Sloan-r’*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.218, 0.050)$&#124; |
| *ExTra (1.2 μm)*&#124;$\rm u_1$&#124; | 0.2058 ± 0.0007 | *ExTra (1.2μm)*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.142, 0.050)$&#124; |
| *ExTra (1.2 μm)*&#124;$\rm u_2$&#124; | 0.1708 ± 0.0028 | *ExTra (1.2μm)*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.273, 0.050)$&#124; |
| **External priors** | **GP priors** |
| Stellar Mass; *M*[⋆] (⁠&#124;$\mathrm{{\rm M}_{\odot }}$&#124;⁠) | &#124;$\mathcal {N}(0.225, 0.012)$&#124; | Amplitude scale &#124;$\mathrm{GP \ln \sigma (flux)}$&#124; | &#124;$\mathcal {U}(-7, -7)$&#124; |
| Stellar Radius; *R*[⋆] (R[⊙]) | &#124;$\mathcal {N}(0.240, 0.012)$&#124; | Length scale &#124;$\mathrm{GP \ln \rho (flux)}$&#124; | &#124;$\mathcal {U}(-7, 2)$&#124; |
| Stellar Effective Temperature; *T*[eff] (K) | &#124;$\mathcal {N}(3075, 75)$&#124; |  |  |
| **Fitted parameters** | **Source** |
| *R*[p]/*R*[⋆] | 0.0618 ± 0.0017 | 0.0425 ± 0.0034 | 2-planet linear fit |
| (*R*[⋆] + *R*[p])/*a* | 0.01367 ± 0.00017 | &#124;$0.01129_{-0.00047}^{+0.00055}$&#124; | 2-planet linear fit |
| cos *i* | &#124;$0.00252_{-0.00032}^{+0.00030}$&#124; | &#124;$0.0048_{-0.0024}^{+0.0021}$&#124; | 2-planet linear fit |
| *T*[0] (BJD) | &#124;$2\, 459\, 002.63051_{-0.00074}^{+0.00070}$&#124; | &#124;$2\, 459\, 007.9879_{-0.0041}^{+0.0045}$&#124; | 2-planet linear fit |
| *P* (d) | &#124;$19.288004_{-0.000024}^{+0.000027}$&#124; | &#124;$25.60712_{-0.00036}^{+0.00031}$&#124; | 2-planet linear fit |
| **Fitted GP hyperparameters** | **Source** |
| ExTrA 2 | &#124;$\mathrm{gp \ln \sigma }$&#124;&#124;$-3.77_{-0.19}^{+0.24}$&#124; | &#124;$\mathrm{gp \ln \rho }$&#124;&#124;$-2.73_{-0.17}^{+0.26}$&#124; | 2-planet linear fit |
| ExTrA 3 | &#124;$\mathrm{gp \ln \sigma }$&#124;&#124;$-4.60_{-0.19}^{+0.22}$&#124; | &#124;$\mathrm{gp \ln \rho }$&#124;&#124;$-2.77_{-0.15}^{+0.24}$&#124; | 2-planet linear fit |
| **Derived parameters** | **Source** |
| Companion radius; *R*[p] (R[⊕]) | 1.550 ± 0.064 | 1.066 ± 0.092 | 2-planet linear fit |
| Instellation; *S*[p] (S[⊕]) | &#124;$0.67_{-0.20}^{+0.15}$&#124; | &#124;$0.48_{-0.17}^{+0.12}$&#124; | 2-planet linear fit |
| Semi-major axis; *a* (au) | 0.0830 ± 0.0027 | 0.0986 ± 0.0054 | 2-planet linear fit |
| Inclination; *i* (deg) | &#124;$89.856_{-0.017}^{+0.018}$&#124; | &#124;$89.72_{-0.12}^{+0.14}$&#124; | 2-planet linear fit |
| Impact parameter; *b* | &#124;$0.195_{-0.024}^{+0.023}$&#124; | &#124;$0.44_{-0.22}^{+0.18}$&#124; | 2-planet linear fit |
| Total transit duration; *T*[tot] (h) | 1.980 ± 0.025 | &#124;$1.99_{-0.21}^{+0.14}$&#124; | 2-planet linear fit |
| Full-transit duration; *T*[full] (h) | 1.741 ± 0.022 | &#124;$1.79_{-0.25}^{+0.15}$&#124; | 2-planet linear fit |
| Equilibrium temperature^⋆; *T*[eq] (K) | 234 ± 12 | 215 ± 12 | 2-planet linear fit |
| Transit depth *TESS*; δ[tr; TESS] (ppt) | &#124;$4.48_{-0.23}^{+0.26}$&#124; | &#124;$2.03_{-0.27}^{+0.31}$&#124; | 2-planet linear fit |
| Transit depth *i + z*; δ[tr; i + z] (ppt) | &#124;$4.48_{-0.48}^{+0.54}$&#124; | - | 2-planet linear fit |
| Transit depth *Sloan-i’*; &#124;$\delta _\mathrm{tr; Sloan-i^{\prime }}$&#124; (ppt) | &#124;$4.60_{-1.20}^{+1.50}$&#124; | - | 2-planet linear fit |
| Transit depth *Sloan-r’*; &#124;$\delta _\mathrm{tr; Sloan-r^{\prime }}$&#124; (ppt) | &#124;$4.81_{-0.90}^{+0.78}$&#124; | - | 2-planet linear fit |
| Transit depth *ExTra* (⁠&#124;$1.2~\rm \,\mu m$&#124;⁠); δ[tr; ExTra(1.2μm)] (ppt) | &#124;$4.20_{-0.96}^{+1.10}$&#124; | - | 2-planet linear fit |

|  | TOI-715 b | TIC 271971130.02 |  |
| **Fit parametrization and priors** |
| Transit depth; *R*[p]/*R*[⋆] | &#124;$\mathcal {U}(0.01, 0.1)$&#124; | &#124;$\mathcal {U}(0.01, 0.1)$&#124; |  |
| Inverse scaled semimajor axis; (*R*[⋆] + *R*[p])/*a* | &#124;$\mathcal {U}(0.01, 0.05)$&#124; | &#124;$\mathcal {U}(0.008, 0.04)$&#124; |  |
| Orbital inclination; cos *i* | &#124;$\mathcal {U}(0.000, 0.04)$&#124; | &#124;$\mathcal {U}(0.000, 0.04)$&#124; |  |
| Transit epoch; *T*[0] (BJD) | &#124;$\mathcal {U}(2\, 458\, 327.3, 2\, 458\, 327.7)$&#124; | &#124;$\mathcal {U}(2\, 458\, 342.0, 2\, 458\, 342.4)$&#124; |  |
| Period; *P* (d) | &#124;$\mathcal {U}(19.0, 19.4)$&#124; | &#124;$\mathcal {U}(25.4, 25.8)$&#124; |  |
| **Limb-darkening coefficients** |
| *TESS*&#124;$\rm u_1$&#124; | 0.3322 ± 0.0013 | *TESS*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.413, 0.050)$&#124; |
| *TESS*&#124;$\rm u_2$&#124; | 0.3103 ± 0.0047 | *TESS*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.259, 0.050)$&#124; |
| *I + z*&#124;$\rm u_1$&#124; | 0.2942 ± 0.0014 | *I + z*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.453, 0.050)$&#124; |
| *I + z*&#124;$\rm u_2$&#124; | 0.3791 ± 0.0052 | *I + z*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.218, 0.050)$&#124; |
| *Sloan-i’*&#124;$\rm u_1$&#124; | 0.3863 ± 0.0018 | *Sloan-i’*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.532, 0.050)$&#124; |
| *Sloan-i’*&#124;$\rm u_2$&#124; | 0.3431 ± 0.0060 | *Sloan-i’*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.265, 0.050)$&#124; |
| *Sloan-r’*&#124;$\rm u_1$&#124; | 0.6408 ± 0.0032 | *Sloan-r’*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.768, 0.050)$&#124; |
| *Sloan-r’*&#124;$\rm u_2$&#124; | 0.2273 ± 0.0079 | *Sloan-r’*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.218, 0.050)$&#124; |
| *ExTra (1.2 μm)*&#124;$\rm u_1$&#124; | 0.2058 ± 0.0007 | *ExTra (1.2μm)*&#124;$\rm q_1$&#124; | &#124;$\mathcal {N}(0.142, 0.050)$&#124; |
| *ExTra (1.2 μm)*&#124;$\rm u_2$&#124; | 0.1708 ± 0.0028 | *ExTra (1.2μm)*&#124;$\rm q_2$&#124; | &#124;$\mathcal {N}(0.273, 0.050)$&#124; |
| **External priors** | **GP priors** |
| Stellar Mass; *M*[⋆] (⁠&#124;$\mathrm{{\rm M}_{\odot }}$&#124;⁠) | &#124;$\mathcal {N}(0.225, 0.012)$&#124; | Amplitude scale &#124;$\mathrm{GP \ln \sigma (flux)}$&#124; | &#124;$\mathcal {U}(-7, -7)$&#124; |
| Stellar Radius; *R*[⋆] (R[⊙]) | &#124;$\mathcal {N}(0.240, 0.012)$&#124; | Length scale &#124;$\mathrm{GP \ln \rho (flux)}$&#124; | &#124;$\mathcal {U}(-7, 2)$&#124; |
| Stellar Effective Temperature; *T*[eff] (K) | &#124;$\mathcal {N}(3075, 75)$&#124; |  |  |
| **Fitted parameters** | **Source** |
| *R*[p]/*R*[⋆] | 0.0618 ± 0.0017 | 0.0425 ± 0.0034 | 2-planet linear fit |
| (*R*[⋆] + *R*[p])/*a* | 0.01367 ± 0.00017 | &#124;$0.01129_{-0.00047}^{+0.00055}$&#124; | 2-planet linear fit |
| cos *i* | &#124;$0.00252_{-0.00032}^{+0.00030}$&#124; | &#124;$0.0048_{-0.0024}^{+0.0021}$&#124; | 2-planet linear fit |
| *T*[0] (BJD) | &#124;$2\, 459\, 002.63051_{-0.00074}^{+0.00070}$&#124; | &#124;$2\, 459\, 007.9879_{-0.0041}^{+0.0045}$&#124; | 2-planet linear fit |
| *P* (d) | &#124;$19.288004_{-0.000024}^{+0.000027}$&#124; | &#124;$25.60712_{-0.00036}^{+0.00031}$&#124; | 2-planet linear fit |
| **Fitted GP hyperparameters** | **Source** |
| ExTrA 2 | &#124;$\mathrm{gp \ln \sigma }$&#124;&#124;$-3.77_{-0.19}^{+0.24}$&#124; | &#124;$\mathrm{gp \ln \rho }$&#124;&#124;$-2.73_{-0.17}^{+0.26}$&#124; | 2-planet linear fit |
| ExTrA 3 | &#124;$\mathrm{gp \ln \sigma }$&#124;&#124;$-4.60_{-0.19}^{+0.22}$&#124; | &#124;$\mathrm{gp \ln \rho }$&#124;&#124;$-2.77_{-0.15}^{+0.24}$&#124; | 2-planet linear fit |
| **Derived parameters** | **Source** |
| Companion radius; *R*[p] (R[⊕]) | 1.550 ± 0.064 | 1.066 ± 0.092 | 2-planet linear fit |
| Instellation; *S*[p] (S[⊕]) | &#124;$0.67_{-0.20}^{+0.15}$&#124; | &#124;$0.48_{-0.17}^{+0.12}$&#124; | 2-planet linear fit |
| Semi-major axis; *a* (au) | 0.0830 ± 0.0027 | 0.0986 ± 0.0054 | 2-planet linear fit |
| Inclination; *i* (deg) | &#124;$89.856_{-0.017}^{+0.018}$&#124; | &#124;$89.72_{-0.12}^{+0.14}$&#124; | 2-planet linear fit |
| Impact parameter; *b* | &#124;$0.195_{-0.024}^{+0.023}$&#124; | &#124;$0.44_{-0.22}^{+0.18}$&#124; | 2-planet linear fit |
| Total transit duration; *T*[tot] (h) | 1.980 ± 0.025 | &#124;$1.99_{-0.21}^{+0.14}$&#124; | 2-planet linear fit |
| Full-transit duration; *T*[full] (h) | 1.741 ± 0.022 | &#124;$1.79_{-0.25}^{+0.15}$&#124; | 2-planet linear fit |
| Equilibrium temperature^⋆; *T*[eq] (K) | 234 ± 12 | 215 ± 12 | 2-planet linear fit |
| Transit depth *TESS*; δ[tr; TESS] (ppt) | &#124;$4.48_{-0.23}^{+0.26}$&#124; | &#124;$2.03_{-0.27}^{+0.31}$&#124; | 2-planet linear fit |
| Transit depth *i + z*; δ[tr; i + z] (ppt) | &#124;$4.48_{-0.48}^{+0.54}$&#124; | - | 2-planet linear fit |
| Transit depth *Sloan-i’*; &#124;$\delta _\mathrm{tr; Sloan-i^{\prime }}$&#124; (ppt) | &#124;$4.60_{-1.20}^{+1.50}$&#124; | - | 2-planet linear fit |
| Transit depth *Sloan-r’*; &#124;$\delta _\mathrm{tr; Sloan-r^{\prime }}$&#124; (ppt) | &#124;$4.81_{-0.90}^{+0.78}$&#124; | - | 2-planet linear fit |
| Transit depth *ExTra* (⁠&#124;$1.2~\rm \,\mu m$&#124;⁠); δ[tr; ExTra(1.2μm)] (ppt) | &#124;$4.20_{-0.96}^{+1.10}$&#124; | - | 2-planet linear fit |

We adopt the signal parameters from Section [3](#sec3) as uniform priors, and the stellar parameters from Section [2](#sec2) as normal priors and fit for all planetary parameters (*R*[p]/*R*[⋆], (*R*[⋆] + *R*[p])/*a*, |$\cos \, i$|⁠, *T*[0], and *P*). We fit two models in the first instance: one where eccentricity is constrained to zero and another where eccentricity is allowed to vary, parametrized as |$\sqrt{e_{\rm b}} \cos {\omega _{\rm b}}$| and |$\sqrt{e_{\rm b}} \sin {\omega _{\rm b}}$| (as in Triaud et al. 2011).

The observations described in Section [4.3](#sec4-3) span several photometric filters from the blue to the near-infrared ends of the spectrum. To allow the colour-dependent transit depth to be a free parameter in our models, we also fit each light curve for a dilution parameter (*D*) which we give a uniform prior between −1 and 1\. We fix the dilution of the *TESS* observations at 0 as the PDCSAP light curve is already corrected for crowding (Stumpe et al. 2012), and use the transit depth as the reference depth; we then use Allesfitter’s ‘|$\rm coupled\_with$|’ functionality to ensure that the dilution and quadratic limb-darkening coefficients are fitted together for all observations taken in the same photometric band.

We use the python package PyLDTK (Parviainen & Aigrain 2015) and the PHOENIX stellar atmosphere library (Husser et al. 2013) to calculate quadratic limb-darkening coefficients for each photometric band. These are also adopted as normal priors in our models after we reparameterise them in the Kipping (2013) parametrization by converting from *u*[1], *u*[2] to *q*[1], *q*[2].

Finally, due to the complex instrument systematics present in the data obtained with the ExTrA telescopes, we model the correlated ‘red’ noise in these observations using a GP. We select the *Matérn 3/2* kernel as implemented by Celerite due to its versatility and its ability to model both long and short term trends. We fit for two GP hyperparameters for each ExTrA telescope: the amplitude scale, σ, and the length scale, ρ. We place wide uniform priors on these terms. For all other telescopes, we make use of a hybrid spline to model the baseline. Allesfitter also fits an ‘error scaling’ term to account for white noise in the data.

The initial fits (1-planet circular, and 1-planet eccentric) both confirm that TOI-715 b is a |$\rm 1.550\pm 0.061\, \rm R_{\oplus }$| planet with an equilibrium temperature of |$\rm 234\pm 12\, \rm K$|⁠. We find that the eccentric model has slightly higher evidence, with a logged Bayes Factor of ∼6.5 ± 1.1, although the eccentricity derived from this more complex model, |$e = 0.31_{-0.20}^{+0.38}$|⁠, is consistent with zero at the 2σ level, and we do not consider this a detection of orbital eccentricity. All modelled ground-based transits of TOI-715 b, as well as the phase-folded *TESS* photometry, can be found in Figs 7 and 8. In Fig. 9 we present the averaged depths, in each photometric band, demonstrating the achromaticity of the observed transits.

Figure 7.

Photometry of TOI-715 b along with best fitting models. Grey points are raw flux and dark pink circles are 15-min binned points. The dark pink lines are 20 fair draws from the posterior transit model. The flux and the transit models have been corrected by subtracting the baseline models. The *TESS*, ExTrA 2 and ExTrA 3 photometry are phase-folded, while the others are single transits.

Figure 8.

Photometry of TOI-715 b along with best-fitting models. Grey points are raw flux and dark pink circles are 15-min binned points. The dark pink lines are 20 fair draws from the posterior transit model. The flux and the transit models have been corrected by subtracting the baseline models. All transits shown here are single transits. For the TRAPPIST light curves, the number in the figure titles denotes the visit number. These are treated as single transits due to the different exposure times of each visit. Increased scatter during the transit observed by SSO-Io (bottom right panel) cause the middle binned point to have a large errorbar.

Figure 9.

Measured transit depths versus wavelength. The dark grey horizontal line indicates the depth of the *TESS* transits, while other depths are highlighted with circles for comparison.

We remind the reader that in Section [3](#sec3), we identified a second transiting planet candidate in the *TESS* data, which we now incorporate into the model, and test this two-planet hypothesis. We test two further models to seek evidence of a second planet in the system, starting by fixing the linear ephemerides of planet b and allowing the mid-time of each individual transit to vary. This model is motivated by the second candidate’s proximity to a 1^(st) order 4:3 mean-motion resonance with the first planet. If the second candidate is real, we might expect for these planets to experience mutual gravitational interactions exciting transit timing variations (TTVs).

We place a wide uniform prior on each transit mid-time, corresponding to the linear predicted mid-time |$\pm \, 60$| min. We find that due to the low SNR of individual transits in the *TESS* data, the transit mid-times are not well constrained for events observed only by *TESS*. However, transits observed simultaneously by several ground-based observatories have significantly less scatter and smaller error bars. In these events, we see evidence suggesting TTVs of up to 5 min, although the Bayesian evidence for this model is lower than for the two linear models tested. Models with linear ephemerides are preferred, with Bayes Factors of ∼9.5 and ∼16 for the circular and eccentric models respectively. We note that we did not test a fixed linear ephemerides fit with a 2-planet model as the individual transits in *TESS* for the second candidate have even lower SNR and we have no ground-based transits at the time of writing.

The final model we test is a 2-planet circular Keplerian model. We fit for all the same parameters as before, but now additionally include the transit parameters for the second candidate. The phase-folded *TESS* photometry for the second candidate is presented in Fig. 10; the fitted and derived parameters for planet b resulting from the 2-planet model are consistent with those emerging from all previous models. Also noteworthy is that the host density as calculated by allesfitter following Seager & Mallén-Ornelas (2003) with the transit parameters of the second candidate is |$\rm 22.7\pm 3.0\, \rm g\, cm^{-3}$|⁠, which is within 1σ of the stellar density prior of |$\rm 23.1\pm 3.2\, \rm g\, cm^{-3}$|⁠, suggesting that both signals are produced over the same star. The results of this fit indicate that if TIC 271971130.02 is confirmed, it is likely a |$1.066\pm 0.092 ~\rm R_\oplus$| planet with an orbital period of |$25.60712_{-0.00036}^{+0.00031}$| and an equilibrium temperature of |$215\pm 12~\rm K$|⁠.

Figure 10.

Phase folded *TESS* photometry of TIC 271971130.02\. Grey points are raw flux and green circles are 15-min binned points. The green lines are 20 fair draws from the posterior transit model. The flux and the transit models have been corrected by subtracting the baseline models.

All fitted and derived parameters from the linear 2-planet circular model are presented in Table 3.

## 6 DISCUSSION

TOI-715 is host to at least one planet (TOI-715 b), with |$R_{\rm b}= \rm 1.550\pm 0.064\, R_{\oplus }$|⁠, receiving an installation |$S_{\rm b} = \rm 0.67_{-0.20}^{+0.15}\, S_{\oplus }$|⁠, which places it within the ‘conservative habitable zone’ defined by Kopparapu et al. (2013) as the circumstellar region where a rocky planet receives and installation flux of |$0.42-0.842\, S_{\oplus }$|⁠. Our model suggests that there is also a possibly second, smaller planet in the system, with size |$R_{02} = 1.066\pm 0.092 ~\rm R_\oplus$|⁠, just within the outer edge of the host’s habitable zone, at an installation |$S_{\rm 02} = \rm 0.48_{-0.17}^{+0.12}\, S_{\oplus }$|⁠.

Expectation of planet yield by *TESS* prior to launch showed *TESS* would detect of order 70 ± 9 Earth-sized planets (⁠|$\lt 1.25 ~\rm R_\oplus$|⁠) and 14 ± 4 conservative ‘habitable zone’ planets |$\lt 2~\rm R_\oplus$| (as in Kopparapu et al. 2013), but that *TESS* would identify of order 0 Earth-sized (⁠|$\lt 1.25 ~\rm R_\oplus$|⁠) ‘conservative habitable zone planets’ (Sullivan et al. 2015). More recent yield estimates (post-launch) by Kunimoto et al. (2022) appear more pessimistic, however, finding only 5 ± 2 ‘conservative habitable zone planets’ |$\lt 2~\rm R_\oplus$| would be detected within the primary mission and first extended mission of *TESS*, a factor nearly three times lower than Sullivan et al. (2015). Should the second planet candidate, TIC 271971130.02 be fully confirmed, its existence would thus represent an unexpectedly important discovery made possible by the *TESS* mission, with the contributions of many ground-based facilities.

In this section, we contextualize the system by examining other factors required for habitability, and assessing its suitability for further in-depth characterization. We begin by revisiting the radius valley to understand what can be learned about the mass of TOI-715 b from current mass–radius relations. We then explore the prospects for a precise mass measurement of this planet with current spectroscopic instrumentation, followed by a discussion of the potential for atmospheric characterization with *HST* and *JWST*.

### 6.1 TOI-715 b and the radius valley

The radius valley, as identified in the *Kepler* sample by Fulton et al. (2017), shows a lack of close-in (⁠|$P\lt 100~\rm d$|⁠) planets with sizes between |$1.5-2~\rm R_{\oplus }$|⁠; TOI-715 b’s |$1.55~\rm R_\oplus$| appears just inside this underpopulated region of parameter space. The importance of the radius valley lies in its potential to teach us about planetary formation and post-formation evolution (e.g. Owen & Wu 2017; Ginzburg et al. 2018; Venturini et al. 2020), and hence planets inside this gap are crucial in furthering our understanding of the factors that sculpt it.

The location and slope of the radius valley has been shown to depend on the properties of the host star (e.g. Fulton & Petigura 2018; Gupta & Schlichting 2019; Berger et al. 2020), and Cloutier & Menou (2020) and Van Eylen et al. (2021) recently re-examined this for low-mass stars. Cloutier & Menou (2020) looked at the small, close-in planet distribution around stars cooler than |$4700\, \rm K$| and found that the slope is opposite in sign compared with FGK stars, and also that the peaks of the planet radius distributions are shifted toward smaller planets. According to this, the center of the radius valley also shifts towards smaller radii. The sample examined in this study contained planets discovered by *Kepler* and *K2*, and was corrected for completeness. This work states that planets falling between their measured radius valley and the one measured by Martinez et al. (2019) for Sun-like stars are so-called ‘keystone planets’, crucial for further understanding the radius valley around low-mass stars. By this metric TOI-715 b’s radius of |$\rm 1.55\, R_{\oplus }$| is just below the boundary of |$\rm 1.66\, R_{\oplus }$| for a period of 19.288 d, falling within the super-Earth category.

Van Eylen et al. (2021), however, used a significantly smaller sample, restricted to confirmed and well-characterized planets, with masses and radii measured to at least 20 per cent precision, orbiting stars cooler than 4000 K. They found a slope in radius-period space opposite in sign compared with Cloutier & Menou (2020), and therefore consistent in sign with FGK stars as measured by, for instance, Martinez et al. (2019). They do, however, find that the valley shifts towards smaller radii for later spectral types. Contrary to the work of Cloutier & Menou (2020), the functional form of the radius valley derived in this study places TOI-715 b above the radius valley and with the population of sub-Neptunes.

In Fig. 11 we present the current sample of exoplanets orbiting stars cooler than |$T_\mathrm{eff}\, =\, 4000\, \rm K$| with planetary radii measured to better than 10  per cent precision^(15) in radius-period space. We highlight the positions of the radius valleys as measured by Martinez et al. (2019); Cloutier & Menou (2020) and Van Eylen et al. (2021) and the positions of TOI-715 b and TIC 271971130.02\. In this sample, a clear radius bimodality for planets orbiting M dwarfs is not visible. What is evident is that precise characterization of planets such as TOI-715 b that fall *between* the various definitions of the low-mass star radius valley is essential to understand whether or not this bimodality will eventually be borne out by the data.

Figure 11.

Plot of the current sample of planets orbiting hosts cooler than 4000 K in radius-period space. The grey circles are confirmed planets with radii measured to better than 10 per cent precision, and the dark pink and green circles are TOI-715 b and TIC 271971130.02, respectively. We note that some errorbars are smaller than the markers. The blue line indicates the measured radius valley according to Van Eylen et al. (2021) for hosts cooler than 4000 K. The black dashed line is the low-mass star radius valley as measured by Cloutier & Menou (2020) for hosts cooler than 4700 K, while the dotted black line indicated the radius valley for Sun-like stars as measured by Martinez et al. (2019). The yellow shaded area indicates where planets referred to as ‘keystone planets’ can be found. The histogram in the right-hand panel does not include TOI-715 b or TIC 271971130.02.

### 6.2 TOI-715 b and the density gap

Luque & Pallé (2022) recently demonstrated using a sample of planets with precisely measured radii (to at least <8 per cent) and masses (to at least <25 per cent) that small planets orbiting low-mass stars in fact likely have a density gap rather than a radius valley. The results of that study indicate that small planets come in three flavours: rocky, water-worlds and gassy. For planets smaller than |$2\, \rm R_{\oplus }$| they find a clear bi-modality in density which allow us to estimate the mass of TOI-715 b in both the rocky (⁠|$\rm \sim \, 7M_{\oplus }$|⁠) and water-world (⁠|$\rm \sim \, 2M_{\oplus }$|⁠) scenarios; both different from the Chen & Kipping (2017) estimate of |$\rm \sim \, 3.5M_{\oplus }$| and the rocky and volatile-rich predictions from Otegi, Bouchy & Helled (2020) (⁠|$\rm \sim \, 4.1M_{\oplus }$| and |$\rm \sim \, 3.5M_{\oplus }$| respectively). While this result is very promising for our understanding of the composition of small planets around low-mass stars, their sample contained only 34 planets; therefore continued systematic characterization of these systems with consistent sample selection criteria remains crucial to understand this distribution and ultimately the composition of planets around M dwarfs.

### 6.3 Prospects for a mass measurement of TOI-715 b

Given the faintness of TOI-715, the only southern hemisphere instrument currently capable of the precision required to measure the mass of planet b is ESPRESSO at the VLT (Very Large Telescope Pepe et al. 2010). If the planet is rocky and has a mass of |$\rm \sim 7\, M_{\oplus }$|⁠, then we calculate a predicted RV semi-amplitude of |$\rm 4.5\, m\, s^{-1}$|⁠. From ESPRESSO’s exposure time calculator (ETC) we find that we could achieve a precision of |$\rm 7.4\, m\, s^{-1}$| per measurement, meaning that for a 25 per cent precision on the mass of the planet, 44 spectra need to be collected, with |$\rm 1800\, s$| exposures and SNR |$\rm \sim$| 14\. For this, we assume that TOI-715 is a relatively slow rotator (⁠|$\rm vsini \lesssim 2\, km\, s^{-1}$|⁠) which is not unusual for old M dwarfs that do not show significant stellar activity (Moutou et al. 2017; Reiners et al. 2022).

However the ETC is usually pessimistic. Based on observed radial-velocity precision of mid-M type planet-host stars obtained with ESPRESSO we expect a slightly better performance for this instrument. Observations of Proxima Cen (M5V) resulted in a mean precision of |$\rm 0.6\, m\, s^{-1}$| (Suárez Mascareño et al. 2020), which is about 40 per cent better than the ETC predicts. A similar case is LHS 1140 (M3V), for which a mean precision of |$\rm 0.8\, m\, s^{-1}$| has been achieved (Lillo-Box et al. 2020), which is a precision twice better than expected. Empirical results^(16) for M dwarfs using the MAROON-X spectrograph at Gemini North (Seifahrt et al. 2020) found a more than 40  per cent increase in precision between M0 and M6 dwarf stars. The ESPRESSO ETC assumes an M2 spectral type to predict the RV precision. Thus, this apparent increase in RV precision for mid-M dwarfs, compared to early M dwarfs can explain the observed difference in performance.

Since TOI-715 is an M4 dwarf, we can conservatively expect a 20 per cent better RV precision compared to the ETC (thus a precision of |$\rm \sim 5.9\, m\, s^{-1}$|⁠). This means a 25 per cent precision on the mass of the planet can be reached with only |$\rm \sim 30$| spectra (⁠|$\rm 1800\, s$| exposures; SNR ∼ 14). This amounts to 15 h of telescope time, which is not unrealistic for a planet of this importance.

If however, TOI-715 b is in fact a ‘water world’, then a mass of |$\rm \sim 2\, M_{\oplus }$| would produce an RV semi-amplitude of just |$\rm 1.3\, m\, s^{-1}$|⁠. In this case 150 h of telescope time are needed to constrain the mass with 25 per cent precision. Should the density bi-modality for small planets around M dwarfs suggested by Luque & Pallé (2022) be confirmed, a non-detection might enable us to infer the planet’s mass.

In Section [5](#sec5), we found that there were small hints of transit timing variations (TTVs) in the ground-based transits of TOI-715 b. If the second planet is confirmed and the orbits are commensurate, the masses of both planets could be suitable for characterization using dynamical modelling. Given the low SNR of individual transits and the long orbital periods, such a campaign would be well suited to ASTEP (Antarctic Search for Transiting Exoplanets, Daban et al. 2010; Schmider et al. 2022), given its convenient Antarctic location (Dransfield et al. 2022b).

### 6.4 Prospects for detailed atmospheric characterization of TOI-715 b

The Transmission Spectroscopy Metric (TSM) as established by Kempton et al. (2018) is often used to quantify the suitability of planets for atmospheric characterization via transmission spectroscopy. We calculate the TSM for TOI-715 b in both mass limits^(17) and for TIC 271971130.02, as well as the published sample of planets with |$R_p\, \lt \, 2~\rm R_{\oplus }$| and |$S_p\, \lt \, 1.6~\rm S_{\oplus }$|^(18) The comparison sample contained 34 planets initially; we then removed the seven planets discovered via radial velocity as these are not known to transit. In Fig. 12, we present this sample of small habitable zone and temperate exoplanets, highlighting the location of the conservative habitable zone (Kopparapu et al. 2013). We also highlight the positions of the five TRAPPIST-1 planets that fall in this parameter space (d-h) (Gillon et al. 2017), the recently discovered LP 890–9 c (SPECULOOS-2 c/TOI-4306 c; Delrez et al. 2022), K2-3 d (Crossfield et al. 2015), and the well characterized mini-Neptune, LHS 1140 b (Dittmann et al. 2017).

Figure 12.

Transmission spectroscopy metric verus installation for transiting planets orbiting hosts cooler than 4000 K. The blue shaded area shows the conservative habitable zone as defined by Kopparapu et al. (2013). The two dark pink circles highlight the positions of TOI-715 b in the rocky and water world scenarios, while the green circle indicates the position of TIC 271971130.02\. We highlight in yellow the positions of the seven planets of TRAPPIST-1\. The sizes of the circles are scaled with planetary radius.

In the case of a ‘water world’ composition, TOI-715 b could have a transmission signal comparable to LHS-1140 b if it does indeed have detectable atmospheric features. In the higher mass limit we find that TOI-715 b has a TSM of approximately 1.9, below the suggested cutoff of 12 for follow-up of terrestrial planets.

We use Tierra (Niraula et al. 2022) to calculate simulated transmission spectra for TOI-715 b in the ‘rocky’ (⁠|$\sim 7\, \rm M_\oplus$|⁠) and ‘water-world’ (⁠|$\sim 2\, \rm M_\oplus$|⁠) mass scenarios in the case of a primordial hydrogen/helium-dominated atmosphere. We then use Pandexo (Batalha et al. 2017) to simulate *JWST* observations in both cases to assess the detectability of atmospheric features. We find that in the low-mass case atmospheric features could be detectable with a single *JWST* transit if the atmosphere is cloudless. In the higher mass case, we find that extracting meaningful features would require at least five transits, also in the cloud-free case. A secondary atmosphere with higher mean molecular mass could suppress the scale height by more than an order of magnitude, in turn making it challenging to detect atmospheric features of these planets (de Wit et al. 2016).

In both mass cases, the carbon dioxide feature centered on |$\rm 4.5\, \,\mu m$|⁠, the methane feature at |$\rm 3.3\, \,\mu m$| and multiple water bands are some of the most accessible atmospheric features with the NIRSpec PRISM, the most suitable instrument for *JWST* follow-up given the similar brightness of TOI-715 compared to TRAPPIST-1\. Detection of these features will constrain the metallicity of the putative planetary atmospheres, providing first hints of the carbon-chemistry of the atmosphere itself (Madhusudhan 2012), and insights into their formation history (Öberg, Murray-Clay & Bergin 2011).

### 6.5 Flares and habitability

Both TOI-715 b and the candidate TIC 271971130.02 lie in an interesting location of the parameter space with regard to insolation, radius, and potential rocky composition, naturally posing questions of their habitability. Many recent prebiotic chemistry and astrobiology studies investigated how life may have originated on Earth and other planets (see e.g. Patel et al. 2015; Airapetian et al. 2016; Xu et al. 2018 and reviews by Sutherland 2017; Kitadai & Maruyama 2018). The first processes leading from inorganic compounds towards RNA precursors likely require a liquid solution (e.g. liquid surface water) and an energy source. For the latter, the host star’s flaring may provide the necessary UV radiation (in the 200–280 nm range) to trigger these early steps (e.g. Rimmer et al. 2018; Todd et al. 2018; Rimmer et al. 2021).

On the other hand, stellar activity can also pose a significant danger to exoplanets. Strong stellar winds and XUV outbursts can contribute to atmospheric loss (e.g. Atri & Mogan 2020). Even if the atmosphere prevails, coronal mass ejections (charged particle streams) may interact with and dissociate atmospheric ozone (e.g. Tilley et al. 2019). Without the major atmospheric absorber of harmful UV radiation, the next flares might sterilize existing surface biology.

We investigate TOI-715’s 24 sectors of TESS data for signs of stellar flaring. We find an average rate of 7 flares per 100 d of observations, with a maximum flare amplitude of 5 per cent in relative flux over nearly 2 yr of data. This analysis was performed using the stella neural network (Feinstein et al. 2020) and as part of the TESS flare catalog (Günther et al., in preparation), building on (Günther et al. 2020). We also compare this flare frequency distribution with the theoretical potential for ozone sterilization (Tilley et al. 2019) and laboratory thresholds for prebiotic chemistry (Rimmer et al. 2018). We find that the flaring is likely too rare and not energetic enough to influence either of these effects.

As TOI-715 is an older star (6.6|$^{+3.2}_{-2.2}$| Gyr), its flaring is likely not as prominent as in younger years, when its planets were forming and prebiotic chemistry steps might have been triggered. Demographic studies of young (<100 Myr) and older M dwarfs show that flaring activity decreases with stellar age, likely due to spin-down and a decreasing stellar dynamo (e.g. Feinstein et al. 2020; Günther et al. 2020). Thus, one could carefully speculate: if stronger flaring or other processes (volcanoes, impacts, or lightning) had led to astrobiology on TOI-715 b or TIC 271971130.02 several Gyr ago, there might be a chance it could still exist (pending, of course, all other criteria such as atmospheric composition, liquid water, etc. are fulfilled).

## 7 CONCLUSIONS

In this work, we have presented the discovery, validation and characterization of TOI-715 b, a |$R_{\rm b}= \rm 1.550\pm 0.064\, R_{\oplus }$| habitable zone planet orbiting an M4 star with a period of |$19.288004_{-0.000024}^{+0.000027}$| d. We also demonstrated that there is possibly a second, smaller planet with radius |$R_{02} = 1.066\pm 0.092 ~\rm R_\oplus$| at a period of |$25.60712_{-0.00036}^{+0.00031}$| d, placing it just inside the outer edge of the circumstellar habitable zone. This system represents the first *TESS* discovery to fall within this most conservative and widely applicable ‘habitable zone’.

We have also demonstrated that TOI-715 b is amenable to further characterization with precise radial velocities and transmission spectroscopy; detailed follow-up of this planet is crucial to further our understanding of the formation and evolution of small, close-in planets. Given its location in radius-installation space, TOI-715 b could also help us understand the characteristics of the M-dwarf radius valley.

Confirmation of the existence of the TIC 271971130.02 in the coming months will also be crucial in planning further characterization of planet b, as disentangling the two planets in a putative radial velocity campaign could prove challenging if the orbit of a second planet is not well known.

## ACKNOWLEDGEMENTS

We thank the reviewer for their comments and feedback as these helped clarify and streamline the manuscript significantly. Funding for the *TESS* mission is provided by NASA’s Science Mission Directorate. We acknowledge the use of public *TESS* data from pipelines at the *TESS* Science Office and at the *TESS* Science Processing Operations Center. This research has made use of the Exoplanet Follow-up Observation Program website, which is operated by the California Institute of Technology, under contract with the National Aeronautics and Space Administration under the Exoplanet Exploration Program. This paper includes data collected by the *TESS* mission that are publicly available from the Mikulski Archive for Space Telescopes (MAST). Based on data collected by the SPECULOOS-South Observatory at the ESO Paranal Observatory in Chile.The ULiege’s contribution to SPECULOOS has received funding from the European Research Council under the European Union’s Seventh Framework Programme (FP/2007-2013) (grant agreement no. 336480/SPECULOOS), from the Balzan Prize and Francqui Foundations, from the Belgian Scientific Research Foundation (F.R.S.-FNRS; grant no. T.0109.20), from the University of Liege, and from the ARC grant for Concerted Research Actions financed by the Wallonia–Brussels Federation. This work is supported by a grant from the Simons Foundation (PI Queloz, grant number 327127). This research is in part funded by the European Union’s Horizon 2020 research and innovation programme (grants agreements no. 803193/BEBOP), and from the Science and Technology Facilities Council (STFC; grant no. ST/S00193X/1 and ST/W000385/1). The material is based upon work supported by NASA under award number 80GSFC21M0002 Based on data collected by the TRAPPIST-South telescope at the ESO La Silla Observatory. TRAPPIST is funded by the Belgian Fund for Scientific Research (Fond National de la Recherche Scientifique, FNRS) under the grant FRFC 2.5.594.09.F, with the participation of the Swiss National Science Fundation (SNF). Based on data collected under the ExTrA project at the ESO La Silla Paranal Observatory. ExTrA is a project of Institut de Planétologie et d’Astrophysique de Grenoble (IPAG/CNRS/UGA), funded by the European Research Council under the ERC Grant Agreement n. 337591-ExTrA. This work has been supported by a grant from Labex OSUG@2020 (Investissements d’avenir – ANR10 LABX56). This work has been carried out within the framework of the NCCR PlanetS supported by the Swiss National Science Foundation. Some of the observations in the paper made use of the High-Resolution Imaging instrument Zorro obtained under Gemini LLP Proposal Number: GN/S-2021A-LP-105\. Zorro was funded by the NASA Exoplanet Exploration Program and built at the NASA Ames Research Center by Steve B. Howell, Nic Scott, Elliott P. Horch, and Emmett Quigley. Zorro was mounted on the Gemini South telescope of the international Gemini Observatory, a program of NSF’s OIR Lab, which is managed by the Association of Universities for Research in Astronomy (AURA) under a cooperative agreement with the National Science Foundation on behalf of the Gemini partnership: the National Science Foundation (United States), National Research Council (Canada), Agencia Nacional de Investigación y Desarrollo (Chile), Ministerio de Ciencia, Tecnología e Innovación (Argentina), Ministério da Ciencia, Tecnología, Inovações e Comunicações (Brazil), and Korea Astronomy and Space Science Institute (Republic of Korea). The Digitized Sky Surveys were produced at the Space Telescope Science Institute under U.S. Government grant NAG W-2166\. The images of these surveys are based on photographic data obtained using the Oschin Schmidt Telescope on Palomar Mountain and the UK Schmidt Telescope. The plates were processed into the present compressed digital form with the permission of these institutions. This work makes use of observations from the LCOGT network. Part of the LCOGT telescope time was granted by NOIRLab through the Mid-Scale Innovations Program (MSIP). MSIP is funded by NSF. This publication benefits from the support of the French Community of Belgium in the context of the FRIA Doctoral Grant awarded to MT. MNG acknowledges support from the European Space Agency (ESA) as an ESA Research Fellow. BVR thanks the Heising–Simons Foundation for support. MG is F.R.S.-FBRS Research Director. YGMC acknowledges support from UNAM-PAPIIT-IG101321 FJP acknowledges financial support from the grant CEX2021-001131-S funded by MCIN/AEI/10.13039/501100011033\. This research made use of Lightkurve, a Python package for Kepler and *TESS* data analysis (Lightkurve Collaboration, 2018). This work made use of astropy:^(19) a community-developed core Python package and an ecosystem of tools and resources for astronomy (Astropy Collaboration 2013, 2018, 2022). This research has made use of the NASA Exoplanet Archive, which is operated by the California Institute of Technology, under contract with the National Aeronautics and Space Administration under the Exoplanet Exploration Program. This research made use of exoplanet (Foreman-Mackey et al. 2021a) and its dependencies (Astropy Collaboration 2013, 2018; Salvatier, Wiecki & Fonnesbeck 2016; Theano Development Team 2016; Kumar et al. 2019; Luger et al. 2019; Agol, Luger & Foreman-Mackey 2020). Resources supporting this work were provided by the NASA High-End Computing (HEC) Program through the NASA Advanced Supercomputing (NAS) Division at Ames Research Center for the production of the SPOC data products.

## DATA AVAILABILITY

*TESS* data products are available via the MAST portal at [https://mast.stsci.edu/portal/Mashup/Clients/Mast/Portal.html](https://mast.stsci.edu/portal/Mashup/Clients/Mast/Portal.html). Follow-up photometry and high resolution imaging data for TOI-715 are available on ExoFOP at [https://exofop.ipac.caltech.edu/tess/target.php?id = 271971130](https://exofop.ipac.caltech.edu/tess/target.php?id=271971130). These data are freely accessible to ExoFOP members immediately and are publicly available following a one-year proprietary period.

## References

Agol

E.

, Luger

R. , Foreman-Mackey

D. ,

2020

,

AJ

,

159

,

123

Airapetian

V. S.

, Glocer

A. , Gronoff

G. , Hébrard

E. , Danchi

W. ,

2016

,

Nat. Geosci.

,

9

,

452

Alibert

Y.

, Benz

W. ,

2017

,

A&A

,

598

,

L5

Astropy Collaboration

,

2013

,

A&A

,

558

,

A33

Astropy Collaboration

,

2018

,

AJ

,

156

,

123

Astropy Collaboration

,

2022

,

ApJ

,

935

,

167

Atri

D.

, Mogan

S. R. C. ,

2020

,

MNRAS

,

500

,

L1

Bailer-Jones

C. A. L.

, Rybizki

J. , Fouesneau

M. , Demleitner

M. , Andrae

R. ,

2021

,

AJ

,

161

,

147

Batalha

N. E.

et al. ,

2017

,

PASP

,

129

,

064501

Berger

T. A.

, Huber

D. , Gaidos

E. , van Saders

J. L. , Weiss

L. M. ,

2020

,

AJ

,

160

,

108

Bonfils

X.

et al. ,

2013

,

A&A

,

549

,

A109

Bonfils

X.

et al. ,

2015

, in

 Shaklan

S. 

ed.,

Proc. SPIE Conf. Ser. Vol. 9605, Techniques and Instrumentation for Detection of Exoplanets VII

.

SPIE

,

Bellingham

, p.

96051L

Borucki

W. J.

et al. ,

2010

,

Science

,

327

,

977

Broeg

C.

, Fernández

M. , Neuhäuser

R. ,

2005

,

Astron. Nachr.

,

326

,

134

Brown

T. M.

et al. ,

2013

,

PASP

,

125

,

1031

Buccino

A. P.

, Lemarchand

G. A. , Mauas

P. J. D. ,

2007

,

Icarus

,

192

,

582

Buder

S.

et al. ,

2021

,

MNRAS

,

506

,

150

Burdanov

A.

, Delrez

L. , Gillon

M. , Jehin

E. ,

2018

, in

 Deeg

H. J. , Belmonte

J. A. 

, eds,

Handbook of Exoplanets

.

Springer Cham

,

New York

, p.

130

Burgasser

A. J.

, Mamajek

E. E. ,

2017

,

ApJ

,

845

,

110

Charbonneau

D.

, Deming

D. ,

2007

, (

)

Chen

J.

, Kipping

D. ,

2017

,

ApJ

,

834

,

17

Ciardi

D. R.

, Beichman

C. A. , Horch

E. P. , Howell

S. B. ,

2015

,

ApJ

,

805

,

16

Cloutier

R.

, Menou

K. ,

2020

,

AJ

,

159

,

211

Cloutier

R.

et al. ,

2020

,

AJ

,

160

,

22

Cloutier

R.

et al. ,

2021

,

AJ

,

162

,

79

Cointepas

M.

et al. ,

2021

,

A&A

,

650

,

A145

Collins

K. A.

, Kielkopf

J. F. , Stassun

K. G. , Hessman

F. V. ,

2017

,

AJ

,

153

,

77

Cortes Contreras

M.

et al. ,

2015

,

18th Cambridge Workshop on Cool Stars, Stellar Systems, and the Sun

.

Lowell Observatory

, p.

805

Coughlin

J. L.

et al. ,

2016

,

ApJS

,

224

,

12

Crossfield

I. J. M.

et al. ,

2015

,

ApJ

,

804

,

10

Cutri

R. M.

et al. ,

2003

,

VizieR Online Data Catalog

, p.

II/246

Cutri

R. M.

et al. ,

2021

,

VizieR Online Data Catalog

, p.

II/328

Daban

J.-B.

et al. ,

2010

, in

 Stepp

L. M. , Gilmozzi

R. , Hall

H. J. 

, eds,

Proc. SPIE Conf. Ser. Vol. 7733, Ground-based and Airborne Telescopes III

.

SPIE

,

Bellingham

, p.

77334T

Davenport

J. R. A.

, Kipping

D. M. , Sasselov

D. , Matthews

J. M. , Cameron

C. ,

2016

,

ApJ

,

829

,

L31

de Wit

J.

et al. ,

2016

,

Nature

,

537

,

69

Delrez

L.

et al. ,

2018

, in

 Marshall

H. K. , Spyromilio

J. 

, eds,

Proc. SPIE Conf. Ser. Vol. 10700, Ground-based and Airborne Telescopes VII

.

SPIE

,

Bellingham

, p.

107001I

Delrez

L.

et al. ,

2022

,

A&A

,

667

,

A59

Demory

B. O.

et al. ,

2020

,

A&A

,

642

,

A49

Dittmann

J. A.

et al. ,

2017

,

Nature

,

544

,

333

Dong

C.

, Lingam

M. , Ma

Y. , Cohen

O. ,

2017

,

ApJ

,

837

,

L26

Douglas

S. T.

et al. ,

2014

,

ApJ

,

795

,

161

Dransfield

G.

et al. ,

2022a

,

MNRAS

,

515

,

1328

Dransfield

G.

et al. ,

2022b

, in

 Adler

D. S. , Seaman

R. L. , Benn

C. R. 

, eds,

Proc. SPIE Conf. Ser. Vol. 12186, Observatory Operations: Strategies, Processes, and Systems IX

.

SPIE

,

Bellingham

, p.

121861F

Dressing

C. D.

, Charbonneau

D. ,

2013

,

ApJ

,

767

,

95

Dressing

C. D.

, Charbonneau

D. ,

2015

,

ApJ

,

807

,

45

Feinstein

A. D.

, Montet

B. T. , Ansdell

M. , Nord

B. , Bean

J. L. , Günther

M. N. , Gully-Santiago

M. A. , Schlieder

J. E. ,

2020

,

AJ

,

160

,

219

Foreman-Mackey

D.

, Hogg

D. W. , Lang

D. , Goodman

J. ,

2013

,

PASP

,

125

,

306

Foreman-Mackey

D.

, Agol

E. , Ambikasaran

S. , Angus

R. ,

2017

,

Astrophysics Source Code Library, record ascl:1709.008

.

Foreman-Mackey

D.

et al. ,

2021a

,

exoplanet-dev/exoplanet v0.5.0

.

Foreman-Mackey

D.

et al. ,

2021b

,

JOSS

,

6

,

3285

Fulton

B. J.

, Petigura

E. A. ,

2018

,

AJ

,

156

,

264

Fulton

B. J.

et al. ,

2017

,

AJ

,

154

,

109

Furlan

E.

, Howell

S. B. ,

2017

,

AJ

,

154

,

66

Furlan

E.

, Howell

S. B. ,

2020

,

ApJ

,

898

,

47

Gaia Collaboration

,

2022

, (

)

Gaia Collaboration

,

2021

,

A&A

,

650

,

C3

Garcia

L. J.

, Timmermans

M. , Pozuelos

F. J. , Ducrot

E. , Gillon

M. , Delrez

L. , Wells

R. D. , Jehin

E. ,

2021

,

Astrophysics Source Code Library

, ascl:2111.006

Garcia

L. J.

, Timmermans

M. , Pozuelos

F. J. , Ducrot

E. , Gillon

M. , Delrez

L. , Wells

R. D. , Jehin

E. ,

2022

,

MNRAS

,

509

,

4817

Giacalone

S.

, Dressing

C. D. ,

2020

,

Astrophysics Source Code Library

,

Giacalone

S.

et al. ,

2021

,

AJ

,

161

,

24

Gilbert

E. A.

et al. ,

2020

,

AJ

,

160

,

116

Gillon

M.

, Jehin

E. , Magain

P. , Chantry

V. , Hutsemékers

D. , Manfroid

J. , Queloz

D. , Udry

S. ,

2011

,

EPJ Web Conf.

,

11

,

06002

Gillon

M.

et al. ,

2017

,

Nature

,

542

,

456

Ginzburg

S.

, Schlichting

H. E. , Sari

R. ,

2018

,

MNRAS

,

476

,

759

Guerrero

N. M.

et al. ,

2021

,

ApJS

,

254

,

39

Günther

M. N.

, Daylan

T. ,

2019

,

Astrophysics Source Code Library

, ascl:1903.003

Günther

M. N.

, Daylan

T. ,

2021

,

ApJS

,

254

,

13

Günther

M. N.

et al. ,

2020

,

AJ

,

159

,

60

Gupta

A.

, Schlichting

H. E. ,

2019

,

MNRAS

,

487

,

24

Gupta

A.

, Schlichting

H. E. ,

2020

,

MNRAS

,

493

,

792

Hauschildt

P. H.

, Allard

F. , Baron

E. ,

1999

,

ApJ

,

512

,

377

He

M. Y.

, Triaud

A. H. M. J. , Gillon

M. ,

2017

,

MNRAS

,

464

,

2687

Hippke

M.

, Heller

R. ,

2019

,

Astrophysics Source Code Library

, ascl:1910.007

Hippke

M.

, David

T. J. , Mulders

G. D. , Heller

R. ,

2019

,

AJ

,

158

,

143

Howell

S. B.

, Furlan

E. ,

2022

,

Front. Astron. Space Sci.

,

9

,

871163

Howell

S. B.

, Everett

M. E. , Sherry

W. , Horch

E. , Ciardi

D. R. ,

2011

,

AJ

,

142

,

19

Howell

S. B.

, Scott

N. J. , Matson

R. A. , Everett

M. E. , Furlan

E. , Gnilka

C. L. , Ciardi

D. R. , Lester

K. V. ,

2021

,

Front. Astron. Space Sci.

,

8

,

10

Husser

T. O.

, Wende-von Berg

S. , Dreizler

S. , Homeier

D. , Reiners

A. , Barman

T. , Hauschildt

P. H. ,

2013

,

A&A

,

553

,

A6

Jehin

E.

et al. ,

2011

,

The Messenger

,

145

,

2

Jenkins

J. M.

,

2002

,

ApJ

,

575

,

493

Jenkins

J. M.

et al. ,

2010

, in

 Radziwill

N. M. , Bridger

A. 

, eds,

Proc. SPIE Conf. Ser. Vol. 7740, Software and Cyberinfrastructure for Astronomy

.

SPIE

,

Bellingham

, p.

77400D

Jenkins

J. M.

et al. ,

2016

, in

 Chiozzi

G. , Guzman

J. C. 

, eds,

Proc. SPIE Conf. Ser. Vol. 9913, Software and Cyberinfrastructure for Astronomy IV

.

SPIE

,

Bellingham

, p.

99133E

Jenkins

J. M.

, Tenenbaum

P. , Seader

S. , Burke

C. J. , McCauliff

S. D. , Smith

J. C. , Twicken

J. D. , Chandrasekaran

H. ,

2020

,

Kepler Data Processing Handbook: Transiting Planet Search, Kepler Science Document KSCI-19081-003

.

Jensen

E.

,

2013

,

Astrophysics Source Code Library

. ascl:1306.007

Kass

R. E.

, Raftery

A. E. ,

1995

,

J. Am. Stat. Assoc.

,

90

,

773

Kasting

J. F.

, Whitmire

D. P. , Reynolds

R. T. ,

1993

,

Icarus

,

101

,

108

Kempton

E. M. R.

et al. ,

2018

,

PASP

,

130

,

114401

Kesseli

A. Y.

, West

A. A. , Veyette

M. , Harrison

B. , Feldman

D. , Bochanski

J. J. ,

2017

,

ApJS

,

230

,

16

Kiman

R.

, Schmidt

S. J. , Angus

R. , Cruz

K. L. , Faherty

J. K. , Rice

E. ,

2019

,

AJ

,

157

,

231

Kipping

D. M.

,

2013

,

MNRAS

,

435

,

2152

Kitadai

N.

, Maruyama

S. ,

2018

,

Geosci. Front.

,

9

,

1117

Kopparapu

R. K.

et al. ,

2013

,

ApJ

,

765

,

131

Kopparapu

R. K.

, Ramirez

R. M. , SchottelKotte

J. , Kasting

J. F. , Domagal-Goldman

S. , Eymet

V. ,

2014

,

ApJ

,

787

,

L29

Kumar

R.

, Carroll

C. , Hartikainen

A. , Martin

O. A. ,

2019

,

J. Open Source Softw.

,

4

,

1143

Kunimoto

M.

, Winn

J. , Ricker

G. R. , Vanderspek

R. K. ,

2022

,

AJ

,

163

,

290

Lee

E. J.

, Chiang

E. , Ormel

C. W. ,

2014

,

ApJ

,

797

,

95

Lépine

S.

, Rich

R. M. , Shara

M. M. ,

2003

,

AJ

,

125

,

1598

Lépine

S.

, Rich

R. M. , Shara

M. M. ,

2007

,

ApJ

,

669

,

1235

Lépine

S.

, Hilton

E. J. , Mann

A. W. , Wilde

M. , Rojas-Ayala

B. , Cruz

K. L. , Gaidos

E. ,

2013

,

AJ

,

145

,

102

Lester

K. V.

et al. ,

2021

,

AJ

,

162

,

75

Li

J.

, Tenenbaum

P. , Twicken

J. D. , Burke

C. J. , Jenkins

J. M. , Quintana

E. V. , Rowe

J. F. , Seader

S. E. ,

2019

,

PASP

,

131

,

024506

Lightkurve Collaboration

,

2018

,

Astrophysics Source Code Library

. ascl:1812.013

Lillo-Box

J.

et al. ,

2020

,

A&A

,

642

,

A121

Lingam

M.

, Loeb

A. ,

2017

,

ApJ

,

848

,

41

Lopez

E. D.

, Fortney

J. J. ,

2013

,

ApJ

,

776

,

2

Luger

R.

, Agol

E. , Foreman-Mackey

D. , Fleming

D. P. , Lustig-Yaeger

J. , Deitrick

R. ,

2019

,

AJ

,

157

,

64

Luque

R.

, Pallé

E. ,

2022

,

Science

,

377

,

1211

Luque

R.

et al. ,

2022

,

A&A

,

666

,

A154

Madhusudhan

N.

,

2012

,

ApJ

,

758

,

36

Madhusudhan

N.

, Piette

A. A. A. , Constantinou

S. ,

2021

,

ApJ

,

918

,

1

Magazzu

A.

, Martin

E. L. , Rebolo

R. ,

1993

,

ApJ

,

404

,

L17

Maiolino

R.

, Rieke

G. H. , Rieke

M. J. ,

1996

,

AJ

,

111

,

537

Mann

A. W.

, Brewer

J. M. , Gaidos

E. , Lépine

S. , Hilton

E. J. ,

2013

,

AJ

,

145

,

52

Mann

A. W.

et al. ,

2019

,

ApJ

,

871

,

63

Martinez

C. F.

, Cunha

K. , Ghezzi

L. , Smith

V. V. ,

2019

,

ApJ

,

875

,

29

Maxted

P. F. L.

,

2016

,

A&A

,

591

,

A111

McCully

C.

, Volgenau

N. H. , Harbeck

D.-R. , Lister

T. A. , Saunders

E. S. , Turner

M. L. , Siiverd

R. J. , Bowman

M. ,

2018

, in

 Guzman

J. C. , Ibsen

J. 

, eds,

Proc. SPIE Conf. Ser. Vol. 10707, Software and Cyberinfrastructure for Astronomy V

.

SPIE

,

Bellingham

, p.

107070K

McQuillan

A.

, Mazeh

T. , Aigrain

S. ,

2014

,

ApJS

,

211

,

24

Montgomery

R.

, Laughlin

G. ,

2009

,

Icarus

,

202

,

1

Morley

C. V.

, Kreidberg

L. , Rustamkulov

Z. , Robinson

T. , Fortney

J. J. ,

2017

,

ApJ

,

850

,

121

Morris

R. L.

, Twicken

J. D. , Smith

J. C. , Clarke

B. D. , Jenkins

J. M. , Bryson

S. T. , Girouard

F. , Klaus

T. C. ,

2020

,

Kepler Data Processing Handbook: Photometric Analysis, Kepler Science Document KSCI-19081-003

.

Moutou

C.

et al. ,

2017

,

MNRAS

,

472

,

4563

Mulders

G. D.

, Pascucci

I. , Apai

D. ,

2015

,

ApJ

,

814

,

130

Murray

C. A.

et al. ,

2020

,

MNRAS

,

495

,

2446

Newton

E. R.

, Irwin

J. , Charbonneau

D. , Berta-Thompson

Z. K. , Dittmann

J. A. , West

A. A. ,

2016

,

ApJ

,

821

,

93

Newton

E. R.

, Irwin

J. , Charbonneau

D. , Berlind

P. , Calkins

M. L. , Mink

J. ,

2017

,

ApJ

,

834

,

85

Niraula

P.

, de Wit

J. , Gordon

I. E. , Hargreaves

R. J. , Sousa-Silva

C. , Kochanov

R. V. ,

2022

,

Nat. Astron.

,

6

,

1287

Nutzman

P.

, Charbonneau

D. ,

2008

,

PASP

,

120

,

317

Öberg

K. I.

, Murray-Clay

R. , Bergin

E. A. ,

2011

,

ApJ

,

743

,

L16

O’Malley-James

J. T.

, Kaltenegger

L. ,

2017

,

MNRAS

,

469

,

L26

Otegi

J. F.

, Bouchy

F. , Helled

R. ,

2020

,

A&A

,

634

,

A43

Owen

J. E.

, Wu

Y. ,

2013

,

ApJ

,

775

,

105

Owen

J. E.

, Wu

Y. ,

2017

,

ApJ

,

847

,

29

Parviainen

H.

, Aigrain

S. ,

2015

,

MNRAS

,

453

,

3821

Patel

B. H.

, Percivalle

C. , Ritson

D. J. , Duffy

C. D. , Sutherland

J. D. ,

2015

,

Nat. Chem.

,

7

,

301

Pepe

F. A.

et al. ,

2010

, in

 McLean

I. S. , Ramsay

S. K. , Takami

H. 

, eds,

Proc. SPIE Conf. Ser. Vol. 7735, Ground-based and Airborne Instrumentation for Astronomy III

.

SPIE

,

Bellingham

, p.

77350F

Pickles

A. J.

,

1998

,

PASP

,

110

,

863

Pourbaix

D.

et al. ,

2004

,

A&A

,

424

,

727

Pozuelos

F. J.

et al. ,

2020

,

A&A

,

641

,

A23

Ramirez

R. M.

, Kaltenegger

L. ,

2016

,

ApJ

,

823

,

6

Reid

I. N.

, Hawley

S. L. , Gizis

J. E. ,

1995

,

AJ

,

110

,

1838

Reiners

A.

et al. ,

2018

,

A&A

,

609

,

L5

Reiners

A.

et al. ,

2022

,

A&A

,

662

,

A41

Ricker

G. R.

et al. ,

2015

,

J. Astron. Telesc. Instrum. Syst.

,

1

,

014003

Riddick

F. C.

, Roche

P. F. , Lucas

P. W. ,

2007

,

MNRAS

,

381

,

1067

Rimmer

P. B.

, Xu

J. , Thompson

S. J. , Gillen

E. , Sutherland

J. D. , Queloz

D. ,

2018

,

Sci. Adv.

,

4

,

eaar3302

Rimmer

P. B.

, Thompson

S. J. , Xu

J. , Russell

D. A. , Green

N. J. , Ritson

D. J. , Sutherland

J. D. , Queloz

D. P. ,

2021

,

Astrobiology

,

21

,

1099

Sabotta

S.

et al. ,

2021

,

A&A

,

653

,

A114

Salvatier

J.

, Wiecki

T. V. , Fonnesbeck

C. ,

2016

,

PeerJ Comput. Sci.

,

2

,

e55

Savitzky

A.

, Golay

M. J. E. ,

1964

,

Anal. Chem.

,

36

,

1627

Schmider

F.-X.

et al. ,

2022

, in

 Marshall

H. K. , Spyromilio

J. , Usuda

T. 

, eds,

Proc. SPIE Conf. Ser. Vol. 12182, Ground-based and Airborne Telescopes IX

.

SPIE

,

Bellingham

, p.

121822O

Scott

N. J.

et al. ,

2021

,

Front. Astron. Space Sci.

,

8

,

138

Seager

S.

, Mallén-Ornelas

G. ,

2003

,

ApJ

,

585

,

1038

Seager

S.

, Petkowski

J. J. , Günther

M. N. , Bains

W. , Mikal-Evans

T. , Deming

D. ,

2021

,

Universe

,

7

,

172

Sebastian

D.

et al. ,

2021

,

A&A

,

645

,

A100

Seifahrt

A.

et al. ,

2020

, in

 Evans

C. J. , Bryant

J. J. , Motohara

K. 

, eds,

Proc. SPIE Conf. Ser. Vol. 11447, Ground-based and Airborne Instrumentation for Astronomy VIII

.

SPIE

,

Bellingham

, p.

114471F

Sharma

S.

et al. ,

2018

,

MNRAS

,

473

,

2004

Skrutskie

M. F.

et al. ,

2006

,

AJ

,

131

,

1163

Smith

J. C.

et al. ,

2012

,

PASP

,

124

,

1000

Speagle

J. S.

,

2020

,

MNRAS

,

493

,

3132

Stassun

K. G.

, Torres

G. ,

2016

,

AJ

,

152

,

180

Stassun

K. G.

, Torres

G. ,

2021

,

ApJ

,

907

,

L33

Stassun

K. G.

, Collins

K. A. , Gaudi

B. S. ,

2017

,

AJ

,

153

,

136

Stassun

K. G.

, Corsaro

E. , Pepper

J. A. , Gaudi

B. S. ,

2018

,

AJ

,

155

,

22

Stassun

K. G.

et al. ,

2019

,

AJ

,

158

,

138

Stevenson

K. B.

, Bean

J. L. , Seifahrt

A. , Gilbert

G. J. , Line

M. R. , Désert

J.-M. , Fortney

J. J. ,

2016

,

ApJ

,

817

,

141

Stumpe

M. C.

et al. ,

2012

,

PASP

,

124

,

985

Stumpe

M. C.

, Smith

J. C. , Catanzarite

J. H. , Van Cleve

J. E. , Jenkins

J. M. , Twicken

J. D. , Girouard

F. R. ,

2014

,

PASP

,

126

,

100

Suárez Mascareño

A.

et al. ,

2020

,

A&A

,

639

,

A77

Sullivan

P. W.

et al. ,

2015

,

ApJ

,

809

,

77

Sutherland

J. D.

,

2017

,

Nat. Rev. Chem.

,

1

,

1

The JWST Transiting Exoplanet Community Early Release Science Team

,

2023

,

Nature

,

614

,

649

Theano Development Team

,

2016

, (

)

Thompson

S. E.

et al. ,

2018

,

ApJS

,

235

,

38

Tilley

M. A.

, Segura

A. , Meadows

V. , Hawley

S. , Davenport

J. ,

2019

,

Astrobiology

,

19

,

64

Todd

Z. R.

, Fahrenbach

A. C. , Magnani

C. J. , Ranjan

S. , Björkbom

A. , Szostak

J. W. , Sasselov

D. D. ,

2018

,

Chem. Commun.

,

54

,

1121

Triaud

A. H. M. J.

,

2021

, in

 Madhusudhan

N. 

ed.,

ExoFrontiers; Big Questions in Exoplanetary Science

.

IOP Publishing

,

Bristol

, p.

6

Triaud

A. H. M. J.

et al. ,

2011

,

A&A

,

531

,

A24

Twicken

J. D.

, Clarke

B. D. , Bryson

S. T. , Tenenbaum

P. , Wu

H. , Jenkins

J. M. , Girouard

F. , Klaus

T. C. ,

2010

, in

 Radziwill

N. M. , Bridger

A. 

, eds,

Proc. SPIE Conf. Ser. Vol. 7740, Software and Cyberinfrastructure for Astronomy

.

SPIE

,

Bellingham

, p.

774023

Twicken

J. D.

et al. ,

2018

,

PASP

,

130

,

064502

Underwood

D. R.

, Jones

B. W. , Sleep

P. N. ,

2003

,

Int. J. Astrobiol.

,

2

,

289

Vach

S.

et al. ,

2022

,

AJ

,

164

,

71

Van Eylen

V.

, Agentoft

C. , Lundkvist

M. S. , Kjeldsen

H. , Owen

J. E. , Fulton

B. J. , Petigura

E. , Snellen

I. ,

2018

,

MNRAS

,

479

,

4786

Van Eylen

V.

et al. ,

2021

,

MNRAS

,

507

,

2154

Venturini

J.

, Helled

R. ,

2017

,

ApJ

,

848

,

95

Venturini

J.

, Guilera

O. M. , Haldemann

J. , Ronco

M. P. , Mordasini

C. ,

2020

,

A&A

,

643

,

L1

Winters

J. G.

et al. ,

2019

,

AJ

,

157

,

216

Wu

Y.

,

2019

,

ApJ

,

874

,

91

Xu

J.

, Ritson

D. J. , Ranjan

S. , Todd

Z. R. , Sasselov

D. D. , Sutherland

J. D. ,

2018

,

Chem. Commun.

,

54

,

5566

Zacharias

N.

, Finch

C. T. , Girard

T. M. , Henden

A. , Bartlett

J. L. , Monet

D. G. , Zacharias

M. I. ,

2013

,

AJ

,

145

,

44

## Author notes

© 2023 The Author(s). Published by Oxford University Press on behalf of Royal Astronomical Society

This is an Open Access article distributed under the terms of the Creative Commons Attribution License (

[https://creativecommons.org/licenses/by/4.0/](https://creativecommons.org/licenses/by/4.0/)

), which permits unrestricted reuse, distribution, and reproduction in any medium, provided the original work is properly cited.