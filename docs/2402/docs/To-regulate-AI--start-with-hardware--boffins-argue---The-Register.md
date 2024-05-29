<!--yml
category: 未分类
date: 2024-05-27 15:02:18
-->

# To regulate AI, start with hardware, boffins argue • The Register

> 来源：[https://www.theregister.com/2024/02/16/boffins_propose_regulating_ai_hardware/](https://www.theregister.com/2024/02/16/boffins_propose_regulating_ai_hardware/)

In our quest to limit the destructive potential of artificial intelligence, a paper out of the University of Cambridge has suggested baking in remote kill switches and lockouts, like those developed to stop the unauthorized launch of nuclear weapons, into the hardware that powers it.

The paper [[PDF](https://www.cser.ac.uk/media/uploads/files/Computing-Power-and-the-Governance-of-AI.pdf)], which includes voices from numerous academic institutions and several from OpenAI, makes the case that regulating the hardware these models rely on may be the best way to prevent its misuse.

"AI-relevant compute is a particularly effective point of intervention: It is detectable, excludable, and quantifiable, and is produced via an extremely concentrated supply chain," the researchers argue.

Training the most prolific models, believed to exceed a trillion parameters, requires immense physical infrastructure: Tens of thousands of GPUs or accelerators and weeks or even months of processing time. This, the researchers say, makes the existence and relative performance of these resources difficult to hide.

What's more, the most advanced chips used to train these models are produced by a relatively small number of companies, like Nvidia, AMD, and Intel, allowing policymakers to restrict the sale of these goods to persons or countries of concern.

These factors, along with others like supply chain constraints on semiconductor manufacturing, offer policymakers the means to better understand how and where AI infrastructure is deployed, who is and isn't allowed to access it, and enforce penalties for its misuse, the paper contends.

### Controlling the infrastructure

The paper highlights numerous ways policymakers might approach AI hardware regulation. Many of the suggestions – including those designed to improve visibility and limit the sale of AI accelerators – are already playing out at a national level.

Last year US president Joe Biden put forward an [executive order](https://www.theregister.com/2023/10/30/us_president_to_sign_new/) aimed at identifying companies developing large dual-use AI models as well as the infrastructure vendors capable of [training them](https://www.theregister.com/2023/11/05/biden_ai_reporting_thresholds/). If you're not familiar, "dual-use" refers to technologies that can serve double duty in civilian and military applications.

More recently, the US Commerce Department [proposed](https://www.theregister.com/2024/01/29/us_raimondo_ai_cloud_kyc/) regulation that would require American cloud providers to implement more stringent "know-your-customer" policies to prevent persons or countries of concern from getting around export restrictions.

This kind of visibility is valuable, researchers note, as it could help to avoid another arms race, like the one triggered by the missile gap controversy, where erroneous reports led to massive build up of ballistic missiles. While valuable, they warn that executing on these reporting requirements risks invading customer privacy and even lead to sensitive data being leaked.

Meanwhile, on the trade front, the Commerce Department has continued to [step up](https://www.theregister.com/2023/10/19/china_biden_ai/) restrictions, limiting the performance of accelerators sold to China. But, as we've previously reported, while these efforts have made it harder for countries like China to get their hands on American chips, they are far from perfect.

To address these limitations, the researchers have proposed implementing a global registry for AI chip sales that would track them over the course of their lifecycle, even after they've left their country of origin. Such a registry, they suggest, could incorporate a unique identifier into each chip, which could help to combat [smuggling](https://www.theregister.com/2024/02/05/smuggling_ai_chips/) of components.

At the more extreme end of the spectrum, researchers have suggested that kill switches could be baked into the silicon to prevent their use in malicious applications.

Here's how they put it:

They further expanded on that mechanism, and to us it sounds as though they are suggesting the accelerators could self-disable or be remotely disabled by watchdogs:

The academics are clearer elsewhere in their study, proposing that processor functionality could be switched off or dialed down by regulators remotely using digital licensing:

In theory, this could allow watchdogs to respond faster to abuses of sensitive technologies by cutting off access to chips remotely, but the authors warn that doing so isn't without risk. The implication being, if implemented incorrectly, that such a kill switch could become a target for cybercriminals to exploit.

Another proposal would require multiple parties to sign off on potentially risky AI training tasks before they can be deployed at scale. "Nuclear weapons use similar mechanisms called permissive action links," they wrote.

For nuclear weapons, these security locks are designed to prevent one person from going rogue and launching a first strike. For AI however, the idea is that if an individual or company wanted to train a model over a certain threshold in the cloud, they'd first need to get authorization to do so.

Though a potent tool, the researchers observe that this could backfire by preventing the development of desirable AI. The argument seems to be that while the use of nuclear weapons has a pretty clear-cut outcome, AI isn't always so black and white.

But if this feels a little too dystopian for your tastes, the paper dedicates an entire section to reallocating AI resources for the betterment of society as a whole. The idea being that policymakers could come together to make AI compute more accessible to groups unlikely to use it for evil, a concept described as "allocation."

### What's wrong with regulating AI development?

Why go to all this trouble? Well, the paper's authors posit that physical hardware is inherently easier to control.

Compared to hardware, "other inputs and outputs of AI development – data, algorithms, and trained models – are easily shareable, non-rivalrous intangible goods, making them inherently difficult to control," the paper reads.

The argument being that once a model is published, either in the open or leaked, there is no putting genie back in the bottle and stopping its spread across the net.

Researchers also highlighted that efforts to prevent the misuse of models have proven unreliable. In one example, the authors highlighted the ease with which researchers were able to dismantle safeguards in Meta's Llama 2 meant to prevent the model from generating offensive language.

Taken to the extreme, it is feared that a sufficiently advanced dual-use model could be employed to accelerate the [development](https://www.theregister.com/2023/07/28/ai_senate_bioweapon/) of chemical or biological weapons.

The paper concedes that AI hardware regulation isn't a silver bullet and doesn't eliminate the need for regulation in other aspects of the industry.

However, the participation of several OpenAI researchers is hard to ignore considering CEO Sam Altman's [attempts](https://www.theregister.com/2023/05/17/ai_oversight_hearing/) to control the narrative around AI regulation. ®