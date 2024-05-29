<!--yml
category: 未分类
date: 2024-05-27 15:11:48
-->

# Novel camera system lets us see the world through eyes of birds and bees | Ars Technica

> 来源：[https://arstechnica.com/science/2024/01/novel-camera-system-lets-us-see-the-world-through-eyes-of-birds-and-bees/](https://arstechnica.com/science/2024/01/novel-camera-system-lets-us-see-the-world-through-eyes-of-birds-and-bees/)

A new camera system and software package allows researchers and filmmakers to capture animal-view videos. Credit: Vasas et al., 2024.

Who among us hasn't wondered about how animals perceive the world, which is often different from how humans do so? There are various methods by which scientists, photographers, filmmakers, and others attempt to reconstruct, say, the colors that a bee sees as it hunts for a flower ripe for pollinating. Now an interdisciplinary team has developed an innovative camera system that is faster and more flexible in terms of lighting conditions than existing systems, allowing it to capture moving images of animals in their natural setting, according to a [new paper](http://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.3002444) published in the journal PLoS Biology.

“We’ve long been fascinated by how animals see the world. Modern techniques in sensory ecology allow us to infer how static scenes might appear to an animal," [said co-author Daniel Hanley](https://www.eurekalert.org/news-releases/1031259?), a biologist at George Mason University in Fairfax, Virginia. "However, animals often make crucial decisions on moving targets (e.g., detecting food items, evaluating a potential mate’s display, etc.). Here, we introduce hardware and software tools for ecologists and filmmakers that can capture and display animal-perceived colors in motion.”

Per Hanley and his co-authors, different animal species possess unique sets of photoreceptors that are sensitive to a wide range of wavelengths, from ultraviolet to the infrared, dependent on each animal's specific ecological needs. Some animals can even detect polarized light. So every species will perceive color a bit differently. Honeybees and birds, for instance, are sensitive to UV light, which isn't visible to human eyes. "As neither our eyes nor commercial cameras capture such variations in light, wide swaths of visual domains remain unexplored," the authors wrote. "This makes false color imagery of animal vision powerful and compelling."

However, the authors contend that current techniques for producing false color imagery can't quantify the colors animals see while in motion, an important factor since movement is crucial to how different animals communicate and navigate the world around them via color appearance and signal detection. Traditional spectrophotometry, for instance, relies on object-reflected light to estimate how a given animal's photoreceptors will process that light, but it's a time-consuming method, and much spatial and temporal information is lost.

Peacock feathers through eyes of four different animals: (a) a peafowl; (b) humans; (c) honeybees; and (d) dogs. Credit: Vasas et al., 2024.

Multispectral photography takes a series of photos across various wavelengths (including UV and infrared) and stacks them into different color channels to derive camera-independent measurements of color. This method trades some accuracy for better spatial information and is well-suited for studying animal signals, for instance, but it only works on still objects, so temporal information is lacking.

That's a shortcoming because "animals present and perceive signals from complex shapes that cast shadows and generate highlights," the authors wrote. 'These signals vary under continuously changing illumination and vantage points. Information on this interplay among background, illumination, and dynamic signals is scarce. Yet it forms a crucial aspect of the ways colors are used, and therefore perceived, by free-living organisms in natural settings."

So Hanley and his co-authors set out to develop a camera system capable of producing high-precision animal-view videos that capture the full complexity of visual signals as they would be perceived by an animal in a natural setting. They combined existing methods of multispectral photography with new hardware and software designs. The camera records video in four color channels simultaneously (blue, green, red, and UV). Once that data has been processed into "perceptual units," the result is an accurate video of how a colorful scene would be perceived by various animals, based on what we know about which photoreceptors they possess. The team's system predicts the perceived colors with 92 percent accuracy. The cameras are commercially available, and the software is open source so that others can freely use and build on it.

The video at the top of this article depicts the colors perceived by honeybees watching fellow bees foraging and interacting (even fighting) on flowers—an example of the camera system's ability to capture behavior in a natural setting. Below, Hanley applies UV-blocking sunscreen in the field. His light-toned skin looks roughly the same in human vision and honeybee false color vision "because skin reflectance increases progressively at longer wavelengths," the authors wrote.