<!--yml
category: 未分类
date: 2024-05-29 12:37:59
-->

# What Is Green Software and Why Do We Need It? - IEEE Spectrum

> 来源：[https://spectrum.ieee.org/green-software](https://spectrum.ieee.org/green-software)

**[Software may be]** [eating the world](https://a16z.com/why-software-is-eating-the-world/), but it is also heating it.

In December 2023, representatives from nearly 200 countries gathered in Dubai for COP28, the U.N.’s climate-change conference, to discuss the urgent need to lower emissions. Meanwhile, [COP28’s website](https://spectrum.ieee.org/internet-carbon-emissions) produced [3.69 grams of carbon dioxide](https://ecograder.com/report/bKjho1BxM1MHJRunAgpDGhSC) (CO[2]) per page load, according to the website sustainability scoring tool [Ecograder](https://ecograder.com/). That appears to be a tiny amount, but if the site gets 10,000 views each month for a year, its emissions would be a little over that of a one-way flight from San Francisco to Toronto.

This was not inevitable. Based on Ecograder’s analysis, unused code, improperly sized images, and third-party scripts, among other things, affect the COP28 website’s emissions. These all factor into the energy used for data transfer, loading, and processing, consuming a lot of power on users’ devices. Fixing and optimizing these things could chop a whopping 93 percent from the website’s per-page-load emissions, Ecograder notes.

While software on its own doesn’t release any emissions, it runs on hardware in data centers and steers data through transmission networks, which account for about [1 percent of energy-related greenhouse gas emissions](https://www.iea.org/energy-system/buildings/data-centres-and-data-transmission-networks) each. The information and communications technology sector as a whole is responsible for an estimated [2 to 4 percent of global](https://www.sciencedirect.com/science/article/pii/S2666389921001884) greenhouse gas emissions. By 2040, that number could reach [14 percent](https://doi.org/10.1016/j.jclepro.2017.12.239)—almost as much carbon as that emitted by [air, land, and sea transport](https://www.wri.org/insights/4-charts-explain-greenhouse-gas-emissions-countries-and-sectors) combined.

Within the sphere of software, [artificial intelligence](https://spectrum.ieee.org/topic/artificial-intelligence/) has its own sustainability issues. AI company [Hugging Face](https://huggingface.co/) [estimated the carbon footprint of its BLOOM large language model](https://www.jmlr.org/papers/volume24/23-0069/23-0069.pdf) across its entire life cycle, from equipment manufacturing to deployment. The company found that BLOOM’s final training emitted 50 tonnes of CO[2]—equivalent to about a dozen flights from New York City to Sydney.

Green software engineering is an emerging discipline consisting of best practices to build applications that reduce carbon emissions. The green software movement is fast gaining momentum. Companies like [Salesforce](https://www.salesforce.com/) have launched their own [software sustainability initiatives](https://www.salesforce.com/news/stories/green-code-software/), while the [Green Software Foundation](https://greensoftware.foundation/) now comprises 64 member organizations, including tech giants [Google](https://spectrum.ieee.org/tag/google), [Intel](https://spectrum.ieee.org/tag/intel), and [Microsoft](https://spectrum.ieee.org/search/?q=microsoft). But the sector will have to embrace these practices even more broadly if they are to prevent worsening emissions from developing and using software.

## What Is Green Software Engineering?

The path to green software began more than 10 years ago. The [Sustainable Web Design Community Group](https://www.w3.org/community/sustyweb/) of the [World Wide Web Consortium](https://www.w3.org/) (W3C) was established in 2013, while the [Green Web Foundation](https://www.thegreenwebfoundation.org/) began in 2006 as a way to understand the kinds of energy that power the Internet. Now, the Green Web Foundation is working toward the ambitious goal of a fossil-free Internet by 2030.

“There’s an already existing large segment of the software-development ecosystem that cares about this space—they just haven’t known what to do,” says [Asim Hussain](https://asim.dev/), chairperson and executive director of the Green Software Foundation and former director of green software and ecosystems at [Intel](https://spectrum.ieee.org/tag/intel).

What to do, according to Hussain, falls under [three main pillars](https://greensoftware.foundation/articles/what-is-green-software): energy efficiency, or using less energy; hardware efficiency, or using fewer physical resources; and carbon-aware computing, or using energy more intelligently. Carbon-aware computing, Hussain adds, is about doing more with your applications during the periods when the electricity comes from clean or low-carbon sources—such as when wind and solar power are available—and doing less when it doesn’t.

## The Case for Sustainable Software

So why should programmers care about making their software sustainable? For one, green software is efficient software, allowing coders to cultivate faster, higher-quality systems, says [Kaspar Kinsiveer](https://ee.linkedin.com/in/kaspar-kinsiveer), a team lead and sustainable-software strategist at the software-development firm [Helmes](https://www.helmes.com/).

These efficient systems could also mean lower costs for companies. “One of the main misconceptions about green software is that you have to do something extra, and it will cost extra,” Kinsiveer says. “It doesn’t cost extra—you just have to do things right.”

Green software is efficient software, allowing coders to cultivate faster, higher-quality systems.

Other motivating factors, especially on the business side of software, are the upcoming legislation and regulations related to sustainability. In the European Union, for instance, the [Corporate Sustainability Reporting Directive](https://finance.ec.europa.eu/capital-markets-union-and-financial-markets/company-reporting-and-auditing/company-reporting/corporate-sustainability-reporting_en) requires companies to report more on their environmental footprint, energy usage, and emissions, including the [emissions related to the use of their products](https://www.ibm.com/topics/scope-3-emissions).

Yet other developers may be motivated by the [climate crisis](https://spectrum.ieee.org/collections/climate-change/) itself, wanting to play their part in fostering a habitable planet for the coming generations. And software engineers have tremendous influence on the actual purpose and emissions of what they build.

“It’s not just lines of code. Those lines have an impact on human beings,” says [June Sallou](https://jnsll.github.io/), a postdoctoral researcher specializing in sustainable-software engineering at the [Delft University of Technology](https://www.tudelft.nl/), in the Netherlands. Because of AI’s societal impact in particular, she adds, developers have a responsibility to ensure that what they’re creating isn’t damaging the environment.

## Building Greener Websites and Apps

The makers of COP28’s website could have taken a page from directories like [Lowwwcarbon](https://lowwwcarbon.com/), which highlights examples of existing low-carbon websites. The company website of the Netherlands-based Web design and branding firm [Tijgerbrood](https://tijgerbrood.nl/en/), for instance, emits less than [0.1 grams of carbon](https://lowwwcarbon.com/case-study/tijgerbrood/) per page view.

Creating sustainable websites like Tijgerbrood’s is a team effort that involves different roles—from business analysts who define software requirements to designers, architects, and those in charge of operations—and includes green practices that can be applied at each stage of the software-development process.

First, analysts will have to consider if the feature, app, or software they’re designing should even be developed in the first place. Tech is often about creating the next new thing, but making software sustainable also entails decisions on what not to build, and that may require a shift in mind-set.

The design stage is all about choosing efficient algorithms and architectures. “Think about sustainability before going into the solution—and not after,” says [Chiara Lanza](https://www.cttc.cat/people/chiara-lanza/), a researcher at the [Sustainable AI unit](https://www.cttc.cat/sustainable-artificial-intelligence-sai/) of the [Centre Tecnològic de Telecomunicacions de Catalunya](https://www.cttc.cat/), in Barcelona.

During the development stage, programmers need to focus on optimizing code. “We need the overall amount of energy we’re using to run software to go down. Some of that will come from writing [code] efficiently,” says [Hannah Smith](https://opcan.co.uk/), a sustainable digital tech consultant and director of operations at the Green Web Foundation.

[Tijgerbrood](https://tijgerbrood.nl/en/)’s website optimized the company’s code by using low-resolution images and modern image formats, loading animations only when a user scrolls them into view, and removing unnecessary code. These techniques help speed up data transfer, loading, and processing on a user’s device. The website also uses minimal JavaScript. “When a user loads a website [with] a lot of JavaScript, it causes them to use a lot more energy on their own device because their device is having to do all the work of reading the JavaScript and running [it],” explains Smith.

When it comes to operations, one of the most impactful actions you can take is to select a sustainable Web hosting or cloud-computing provider. The Green Web Foundation has a tool to [check if your website runs on green energy](https://www.thegreenwebfoundation.org/green-web-check/), as well as a [directory of hosting providers powered by renewable energy](https://www.thegreenwebfoundation.org/tools/directory/). You can also ask your hosting provider if you can scale how your software runs in the cloud so that peak usage is powered by green energy or pause or switch off certain services during nonpeak hours.

## AI the Green Way

Programmers can apply green software strategies when developing AI as well. Trimming training data is one of the major ways to make AI systems greener. Starting with data collection and preprocessing, it’s worth thinking about how much data is really needed to do the job. It may pay to clean the dataset to remove unnecessary data, or select only a subset of the dataset for training.

“The larger your dataset is, the more time and computation it will take for the algorithm to go through all the data,” hence using up more energy, says [Sallou](https://jnsll.github.io/).

For instance, in a [study](https://www.computer.org/csdl/proceedings-article/ict4s/2022/828600a035/1F8ztEmMGYM) of six different AI algorithms that detect SMS spam messages, Sallou and her colleagues found that the [random forest algorithm](https://www.ibm.com/topics/random-forest), which combines the output of a collection of decision trees to make a prediction, was the most energy-greedy algorithm. But reducing the size of the training dataset to 20 percent—only 1,000 data points out of 5,000—dropped the energy consumption of training by nearly 75 percent, with only a 0.06 percent loss in accuracy.

Choosing a greener algorithm could also save carbon. Tools like [CodeCarbon](https://codecarbon.io/) and [ML CO[2] Impact](https://mlco2.github.io/impact/) can help make the choice by estimating the energy usage and carbon footprint of training different AI models.

## Tools for Measuring Software’s Carbon Footprint

To write green code, developers need a way of measuring the actual carbon emissions across a system’s entire life cycle. It’s a complex feat, given the myriad processes involved. If we take AI as an example, its life cycle encompasses raw material extraction, materials manufacturing, hardware manufacturing, model training, model deployment, and disposal—and not all of these stages have available data.

“We don’t understand huge parts of the ecosystem at the moment, and access to reliable data is tough,” Smith says. The biggest need, she adds, is “open data that we can rely on and trust” from big tech data-center operators and cloud providers like [Amazon](https://spectrum.ieee.org/tag/amazon), [Google](https://spectrum.ieee.org/tag/google), and [Microsoft](https://spectrum.ieee.org/tag/microsoft).

Until that data surfaces, a more practical approach would be to measure how much power software consumes. “Just knowing the energy consumption of running a piece of software can impact how software engineers can improve the code,” Sallou says.

Developers themselves are heeding the call for more measurement, and they’re building tools to meet this demand. The W3C’s Sustainable Web Design Community Group, for instance, plans to provide a test suite to measure the impacts of implementing its Web sustainability guidelines. Similarly, the Green Software Foundation wrote a [specification](https://github.com/Green-Software-Foundation/sci) to calculate the carbon intensity of software systems. For accurate measurements, Lanza suggests isolating the hardware in which a system runs from any other operations and to avoid running any other programs that could influence measurements.

Other tools developers can use to measure the impact of green software engineering practices include dashboards that give an overview of the estimated carbon emissions associated with cloud workloads, such as the [AWS Customer Carbon Footprint Tool](https://aws.amazon.com/aws-cost-management/aws-customer-carbon-footprint-tool/) and [Microsoft’s Azure Emissions Impact Dashboard](https://www.microsoft.com/en-us/sustainability/emissions-impact-dashboard); energy profilers or power monitors like [Intel’s Performance Counter Monitor](https://github.com/intel/pcm); and tools that help calculate the carbon footprint of websites, such as [Ecograder](https://ecograder.com/), [Firefox Profiler](https://profiler.firefox.com/), and [Website Carbon Calculator](https://www.websitecarbon.com/).

## The Future Is Green

Green software engineering is growing and evolving, but we need more awareness to help the discipline to become more widespread. This is why, in addition to its [Green Software for Practitioners course](https://learn.greensoftware.foundation/), the Green Software Foundation aims to create more training courses, some of which may even lead to certifications. Likewise, Sallou coteaches a graduate course in [sustainable software engineering](https://luiscruz.github.io/course_sustainableSE/2023/), whose syllabus is open and can be used as a foundation for anyone looking to build a similar course. Providing this knowledge to students early on, she says, could ensure they bring it to their workplaces as future software engineers.

In the realm of artificial intelligence, [Navveen Balani](https://navveenbalani.dev/), an AI expert and Google Cloud Certified Fellow who also serves on the Green Software Foundation’s steering committee, notes that AI could inherently include green AI principles in the coming years, much like how security considerations are now an integral part of software development. “This shift will align AI innovation with environmental sustainability, making green AI not just a specialty but an implied standard in the field,” he says.

As for the Web, Smith hopes the Green Web Foundation will cease to exist by 2030\. “Our dream as an organization is that we’re not needed, we meet our goal, and the Internet is green by default,” she says.

Kinsiveer has observed that in the past, software had to be optimized and built well because hardware then was lacking. As hardware performance and innovation leveled up, “the quality of programming itself went down,” he says. But now, the industry is coming full circle, going back to its efficiency roots and adding sustainability to the mix.

“The future is green software,” Kinsiveer says. “I cannot imagine it any other way.”

From Your Site Articles

Related Articles Around the Web