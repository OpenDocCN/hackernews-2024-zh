<!--yml
category: 未分类
date: 2024-05-27 14:34:08
-->

# What does the Cerebellum Do Anyway? - by Sarah Constantin

> 来源：[https://sarahconstantin.substack.com/p/what-does-the-cerebellum-do-anyway](https://sarahconstantin.substack.com/p/what-does-the-cerebellum-do-anyway)

This is the cerebellum. Its name means “little brain” — it’s a whole other brain under your “big” one.

If you vaguely remember something about what the cerebellum does, you’re probably thinking something to do with balance. Medical students have to learn the “cerebellar gait” that results from cerebellar injury (it’s the same staggering gait that drunk people have, because alcohol impairs the cerebellum.)

A more detailed neurological exam of a patient with cerebellar disease shows a wider variety of motor problems.

Here you’ll notice that the patient can’t bring his finger to his nose or clap his hands without a wobbling back-and-forth motion; that his eyes “wobble” back and forth (which is called [nystagmus](https://en.wikipedia.org/wiki/Nystagmus)); and that he wobbles backwards and forwards while standing or walking, so that he nearly falls over and needs a broad-based gait to support himself.

In cerebellar disease, muscle tone is diminished (people are “floppy”), movements are not fluent (each individual sub-movement is separate), there’s [dysmetria](https://en.wikipedia.org/wiki/Dysmetria) (failure to “aim” or estimate the distance to move, overshoot or undershoot) and there’s “[intention tremor](https://www.ncbi.nlm.nih.gov/books/NBK560642/#:~:text=Intention%20tremor%20is%20defined%20as,worsening%20before%20reaching%20the%20endpoint.)” (high amplitude, relatively slow wobbles that arise when the patient starts to move, as contrasted with the “resting tremor” characteristic of e.g. Parkinson’s disease.)

Clearly, the cerebellum does something to control movement, and movement is impaired when it is damaged. But *why do we need a whole other “little brain” to control these aspects of movement*?

There are already regions of the cerebrum (or “forebrain”) dedicated to movement, like the motor cortex and the basal ganglia. And you can do a lot of movement using just those!

Even in the rare cases known as cerebellar agenesis, where a person is born totally lacking a cerebellum, movement is still possible, just impaired: slow motor and speech development in childhood, abnormal spoken pronunciation, wobbly limb movements, and mild-to-moderate intellectual disability. But *not* paralysis, and not even particularly bad disability overall — a lot of these people were able to live independently and work at jobs.

So…that’s weird.

What is the cerebellum’s job? It seems weird to have a *whole separate organ* for “make motor and cognitive skills work somewhat better.”

The other weird thing about the cerebellum is anatomical.

These very large, complex neurons are the Purkinje cells, which exist only in the cerebellum.

Illustration by Santiago Ramon y Cajal

Notice how the Purkinje cell (e) is much more complex than the other neuron types.

They have hundreds of synapses each, unlike the neurons of the cerebrum which only have a few.

Most of the other cells in the cerebellum are the small granule cells — in fact, they are so numerous that they comprise more than half of all neurons in the whole human brain. In total, the cerebellum contains 80% of all neurons!

If you were an alien with a microscope who knew nothing about neurology, your first assumption would be “ah yes, the thinking happens in the cerebellum.”

Why is there so much neuronal complexity dedicated to….making movement a bit smoother and “higher” cognition a bit better?

The third weird fact is that the *size* of the cerebellum has been growing throughout primate evolution and human prehistory, *faster* than overall brain size.

Great ape brains are distinguished from monkey brains by their larger frontal *and cerebellar* lobes. The Neanderthals had bigger brains than us but smaller cerebella. And, most strikingly, modern humans have much bigger cerebella than “anatomically modern” Cro-Magnon humans of only 50,000 years ago (but relatively *smaller* cerebral hemispheres!)

An alien paleontologist could be forgiven for assuming “ah yes, the cerebellum, the seat of the higher intellect.”

The cerebellum *looks* like it should have some crucial unique function. Something key to “what makes us human.” But what could it be?

Some of Ivan Pavlov’s famous dogs

Classical conditioning — the thing that makes a dog salivate when it hears a bell it’s learned to associate with food — is a very low-level process.

You don’t need a lot of brain for classical conditioning.

You can classically condition the sea slug *Aplysia,* which has only 20,000 neurons, to flinch from a neutral sensation it’s learned to associate with a painful one.

*Aplysia californica* releasing a toxic cloud in self-defense

Even single cells can exhibit learning. The giant slime mold amoeba *Physarum* can be “trained” to cross a noxious part of a petri dish to reach food on the other side.

However, in vertebrates, classical conditioning is highly localized in the nervous system: the cerebellum is *necessary and sufficient* for learning conditioned responses.

A standard test of classical conditioning is the eyeblink test: can the subject learn to associate a neutral stimulus with an unpleasant one (like a puff of air to the eye) and automatically blink in response to the neutral stimulus.

If you lesion the cerebellum in animals, they no longer exhibit eyeblink conditioning. Humans with cerebellar damage also have no eyeblink conditioning.

Conversely, if you want classical conditioning, *all you need* is a cerebellum — in fact, all you need for classical conditioning is the Purkinje cells!

The eyeblink response is governed by the Purkinje cells. When an animal feels a puff of air to the eye, the Purkinje cells stop firing temporarily, resulting in an eyeblink.

So, in 2007, some Swedish researchers decided to study this response in isolation. Here’s how it works.

You take a “decerebrate” ferret — i.e. a ferret with the whole cerebrum severed from the rest of the brain (consisting of the brainstem and cerebellum). You condition individual Purkinje cells with electrode stimulation of two types of neurons that form their inputs: the climbing fibers (unconditioned stimulus) and the mossy fibers (conditioned stimulus.) Stimulating the climbing fibers (the same thing that happens naturally when a puff of air hits the eye) causes a temporary suppression of Purkinje cell firing; stimulating the mossy fibers does not. *But,* if the mossy fibers are stimulated *right before* the climbing fibers, the Purkinje neurons *learn* to anticipate the association and suppress their firing in response to the mossy fiber stimulation. This is a textbook example of classical conditioning.

Are cerebellar Purkinje cells the only individual neurons capable of single-cell learning?

Not in all animals; the sensory neurons of the sea slug *Aplysia* can be classically conditioned. But I haven’t been able to find a study in vertebrates of single-cell classical conditioning or associative learning outside the cerebellum. (Other regions of the brain, of course, are involved in learning, but the learning might be occurring through *multicellular* information like the relationships between nearby neurons’ firing patterns.)

Why do we care?

First of all, this gives us at least one “unique job” for the cerebellum: it is the place in the brain where conditioned associations are learned.

Second of all, it *disproves* the [long-term potentiation](https://en.wikipedia.org/wiki/Long-term_potentiation) theory that learning in the brain happens exclusively through strengthening synaptic connections between neurons. (The old dictum that “neurons that fire together, wire together.”) The brain is *not* like a neural network where the only thing that is “learned” or “updated” is the weights between neurons. At least some learning evidently happens *within individual neurons.*

That’s bad news for anyone hoping to simulate a brain digitally. It means there’s a lot more relevant stuff to simulate (like the learning that goes on within cells) than the [connectionist](https://en.wikipedia.org/wiki/Connectionism) paradigm of treating each biological neuron like a neural-net “neuron” would imply, and thus the computational requirements of simulating a brain are higher — maybe vastly higher — than connectionists hope.

On the other hand, intracellular learning is *great* news for neuroscientists trying to discover exactly how learning works inside a brain. If you’re studying an example of learning (or classical conditioning) that occurs within a single neuron, then the long-hypothesized “engram”, or physical object corresponding to a piece of newly learned information, has to reside *inside that neuron*. Something needs to physically or chemically *change* in that neuron, representing the newly learned information, and causing the corresponding change in the neuron’s firing behavior.

Not only can individual Purkinje cells in the cerebellum be classically conditioned, they can also learn information about the *timing* of stimuli.

If the time interval between the conditioned stimulus and unconditioned stimulus is varied, the Purkinje cells learn to suppress their firing at *different times* to match.

In other words, if the Purkinje cell has “learned” from experience that an aversive stimulus will typically follow the neutral stimulus in exactly 50 milliseconds, then the Purkinje cell will delay roughly that long after receiving the neutral stimulus before pausing its firing. If the Purkinje cell “learns” a longer delay, it’ll wait longer before the pause.

This means that something inside the Purkinje cell is capable of representing *number* or *quantity*.

**Cerebellar Purkinje cells, in other words, can count. (Or measure.)**

The fact that individual cerebellar Purkinje neurons contain the ability to measure quantities is highly suggestive about the function of the cerebellum, given the symptoms of cerebellar disease.

Dysmetria, or failure to estimate the right distance and/or speed to move, is a typical symptom of damage to the cerebellum. If *measurement* (especially motor timing) is localized to the cerebellum, then dysmetria as a cerebellar deficit makes perfect sense.

Other symptoms of cerebellar damage are easy to understand as consequences of dysmetria. Intention tremor (those big back-and-forth wobbles when the patient tries to move) might simply be what happens when dysmetria causes the movement to initially overshoot, and then an attempt to correct the overshoot itself overshoots and swings in the other direction, and so on. Nystagmus could be the same thing, with the muscles of the eye.

Even the non-motor symptoms of cerebellar damage, like poor performance on cognitive tests, have sometimes been referred to as a kind of “dysmetria of thought.”

Cerebellar patients have trouble with planning tasks (like the [Tower of Hanoi](https://en.wikipedia.org/wiki/Tower_of_Hanoi) puzzle), spatial reasoning tasks, and executive function tasks; you might suppose that mentally “gauging” how long to spend doing things, or “relating” subtasks to the whole, as well as understanding how objects fit together in space, are aspects of mental “measurement”.

Cerebellar patients have normal abilities to make grammatical sentences, but make strange errors in generating *logical* sentences. Things like:

*   “It’s a big job, but it’s not easy.”

*   “If you drive, it will be less crowded”

*   “Although it’s the wrong size for you, I’ll ask to have one that will fit you.”

*   “I never thought I would meet you here; Nor did I, because everything seems so fresh here to buy.”

Conjunctions (like “but”, “although”, “because”, “if”) represent particular logical relationships between parts of sentences. The cerebellar patients’ errors imply a specific impairment in the ability to make those conjunctional relationships.

A sentence that begins “it’s a big job, *but*” ought to end with something that would ordinarily seem to conflict with the claim “it’s a big job”; the cerebellar patient instead ended the sentence with something that typically *reinforces* the claim (“big jobs” are *usually* “not easy”). This error violates the expected logical relationship between clauses.

In a sense this is analogous to a problem in “spatial” or “part-whole” relating — the cerebellar patient has trouble constructing sentences whose clauses relate in the right way to match the conjunction between them. It’s analogous to the way cerebellar patients have the basic sensorimotor ability to do all the same individual movements that healthy people do, but they have trouble sequencing them fluently, “putting the pieces together” in the right way.

Of course, other areas of the brain are involved in “measurement” or “quantity” activities too. Visual areas of the brain “measure” spatial distances, auditory areas “measure” pitch and rhythm, various parts of the cerebral cortex are active in mental arithmetic, etc. So it’s not that the cerebellum is the sole region that “measures” or “relates” things. But there is something measurement-related going on.

Another broad theory of what the cerebellum is “for” is *anticipation* or *preparation*.

This fits well with the fact that classical conditioning happens in the cerebellum. Classical conditioning consists of learning to *expect* one stimulus to follow another, and respond *in anticipation* of the expected stimulus.

It also fits well with the cerebellum’s role in motor planning and sequencing. Fluent movement requires unconsciously, rapidly forming intentions to do *multiple different things in sequence* without having to stop to think between steps.

> Gordon Holmes (1939) quotes a cerebellar-lesioned patient as saying, "The movements of my left (unaffected) arm are done subconsciously, but I have to think out each movement of the right (affected) arm. I come to a dead stop in turning and have to think before I start again."

This kind of cerebellar damage symptom represents a failure of the *anticipatory* or *preparatory* function that automatically “gets ready” for the next step before the last one is complete.

Patients with cerebellar damage have impaired ability to switch their attentional focus (for instance, between a task that requires watching for visual cues and one that requires listening for auditory cues) but unimpaired ability to perform similar tasks that don’t require shifting focus. Patients with brain lesions *outside* the cerebellum didn’t have problems with task-switching.

This also suggests that the cerebellum is involved in the “readiness” or “anticipation” prior to making a mental action.

Also, experimentally stimulating the cerebellum in animals makes them more sensitive to subsequent sensory stimulus, further supporting the hypothesis that the cerebellum helps organisms “prepare to pay attention”.

The Purkinje neurons of the cerebellum, in particular, are involved in “preparatory” motor adjustments, such as altering one’s grip on an object in anticipation of the experimenter moving it.

If the function of the cerebellum is fully general “anticipatory” or “predictive” modeling, this would explain why it’s so important, especially in primate and hominid evolution. Dexterity (tool use, throwing) has obviously been selected for in our primate and hominid ancestors, and so have the general cognitive abilities to predict and make sense of a changing world.

It also would explain why people without a cerebellum are still capable of most of the same tasks as healthy people, just with worse performance.

People who lack a cerebellum are *wholly* incapable of eyeblink classical conditioning, but they’re not wholly incapable of learning or memory; they learn new skills *slowly* but they do learn them and they don’t have total amnesia.

This would make sense if they are unable to learn to *instantly anticipate* context to prepare for upcoming actions, but they are able to learn and form memories through a slower, noisier route using the cerebrum. They can perform most of the same skills as healthy people, but they have to adjust “by trial and error” instead of using anticipation to get things right the first time, so they’re slower and less accurate.

Another way of phrasing this is that the cerebellum is a *forward model* of the consequences of action.

The cerebellum receives an [efference copy](https://en.wikipedia.org/wiki/Efference_copy#:~:text=In%20physiology%2C%20an%20efference%20copy,by%20an%20organism's%20motor%20system.) of motor commands generated by the motor cortex, and uses its model to predict the sensory consequences; it can then compare predicted vs. actual data and error-correct. A fast, subconscious path for sensorimotor prediction and learning (including classical conditioning) is confined to the cerebellum alone; a slower path includes both the cerebellum and cerebrum.

It takes hundreds of milliseconds to consciously perceive sensory information — far too slow a timescale to allow finely tuned and responsive motion that adapts to sensory feedback. Motion in real time has to be shaped and controlled by something faster than sensory processing through the long chains of neurons used for e.g. image recognition in the cortex — but moving totally “blind” without feedback control from *anything* would result in unacceptably crude, sloppy, choppy movement. The solution, perhaps, is the “virtual reality” generated by the cerebellum, a predictive model of the world that runs faster than the senses.

All vertebrates have a cerebellum.

If you remove the cerebellum from a dogfish, it can still swim, but it has a tendency to “stall” and difficulty judging its turns, so that it often bumps into the sides of the tank.

The cerebellum is especially enlarged in fish with electrolocation abilities, like the Peters’s elephant-nose fish *Gnathonemus petersii,* an African freshwater fish that uses electricity-sensing receptors all over its body to navigate around obstacles.

Peter’s elephant-nose fish

This fish uses its huge cerebellum to monitor the electrical signatures of moving objects in the water around it. Particular Purkinje cells in the elephant-nose fish cerebellum are sensitive to particular target distances and speeds, just as particular neurons in the mammalian visual cortex are sensitive to the shape, location, and angle of visual features.

This unusual electrosensing function reinforces that the cerebellum is not just a motor control organ, but has a more general function related to spatial and environmental awareness.

The platypus also uses its cerebellum to keep track of electrical signals

Monotremes such as platypuses also have an unusually large cerebellum; like the elephant-nose fish, the platypus is electrosensitive (in its beak) which it uses to detect swimming prey in the water.

The fin whale, whose cerebellum is massively expanded

Aquatic mammals — whales, dolphins, seals, and sea lions — are also notable for their enlarged cerebellums, especially the baleen whales. In marine mammals, the cerebellum is used for echolocation. Echolocating bats also use the cerebellum for navigation, though their overall cerebellum size relative to brain size is small.

In many mammals, large areas of the cerebellum are devoted to processing sensory and motor information for parts of the body that are particularly dexterous and used in exploration: the snout in rodents, the hands in primates, the tip of the tail in arboreal monkeys.

Across animals, the cerebellum seems to be involved in both motion and sensory perception, and intriguingly, seems to be particularly enlarged in animals that use echolocation or electrosensing in the water, for spatial awareness of object locations in all directions.

This is suggestive of something like “spatial world modeling” going on in the cerebellum, and is consistent with the theory that the cerebellum’s job is anticipation and preparation.

Echolocation and electrosensing are both sensory modalities that involve an organism generating a “field” around itself (of electric or sound waves) and perceiving objects in the patterns of disruption in that field. Unlike vision, hearing, and smell, they are “[active sensory systems](https://en.wikipedia.org/wiki/Active_sensory_systems)”, in which the organism can control the intensity, direction, and timing of the “probe” signal (the sound or electric signal they emit).

Touch can have an analogous “actively controllable” quality, as the animal reaches out a snout, hand, or tail to explore its nearby surroundings. But truly active sensory systems, unlike touch, allow an animal to explore and probe at a distance, and gain a 3D model of its whole surrounding world.

While humans don’t have these kinds of sensory systems, it may provide some intuition for what our cerebellum is doing; perhaps building implicit anticipations of where everything in the physical world around us, and how it will respond if “poked”.

If you try to map regions of the cerebellum by function and by functional connectivity you get a close one-to-one match with the cerebrum, even down to the localization of language in one hemisphere (the left hemisphere of the cerebrum and the right hemisphere of the cerebellum; everything’s flipped.)

The only parts of the cerebral cortex *without* a corresponding cerebellar region are the auditory and visual cortices. (Suggestive, given that hearing and vision are *passive* senses, and my hypothesis in the previous section that the cerebellum has something to do with *active* sensing.)

The cerebellum has a repeated, almost crystal-like neural structure: it’s divided into multiple identical parallel modules.

The Purkinje cells (PC, in orange) project down into the core of the cerebellum, where they connect to deep nuclei. Climbing fibers (red) feed back up to the Purkinje cells. Mossy fibers (yellow) also feed into the Purkinje cells indirectly, via the granular cells (beige), and the Golgi, stellate, and basket cells (blue). The blue cells inhibit the Purkinje cells, while the red, yellow, and beige ones are excitatory.

Basically, this is a feedforward excitatory chain, plus inhibitory feedback loops. The main chain goes:

but then there are lots of additional loops where this pathway can be self-inhibiting.

The primary feedforward chain, though, is a reasonable candidate for the mechanism behind the super-fast “forward model” that generates predictions to inform action faster than sensory processing can generate conscious perceptions.

On a slightly larger scale, nearby patches of neurons in the cerebellum form pretty much self-contained modules without much connection to cells in other modules.

This is in contrast to the cerebral cortex, which varies a lot in cell composition between regions, has lots of recurrent loops, and lots of cross-connections between neurons from different local columns. The cerebellum is “one and done” — information goes *in*, *through* the Purkinje cells, and *out.*

There are lots of different such modules, looping the cerebellum together with different parts of the cerebrum.

*   Loops through the parietal lobes are involved in visual-motor coordination (like reaching the hand out to grasp something.)

*   Loops through the oculomotor cortex are involved in controlling eye movements.

*   Loops through the prefrontal cortex are involved in control of attention and working memory, fear extinction learning, mental preparation of imminent actions, and procedural learning.

*   There are other loops (including through the basal ganglia and limbic system) but these have less well understood functions.

The independence of cerebellar modules makes sense given the need for speed — you can’t have long chains of interconnected neurons messing around if you want to give near-real-time model responses to control immediate action.

People often talk as though “higher” intelligence lives in the cortex, especially the frontal lobes. The “hindbrain” is for boring, animal stuff, like controlling heart rate and hormones. Real thinking happens behind your noble brow.

This picture, it turns out, is wrong.

Modern *Homo sapiens* is as much characterized by our big cerebellums as by our big frontal lobes.

Even the most iconically “thought-like” thinking — the phonological loop, i.e. imagined or inner speech — passes through both the cerebrum *and* cerebellum. (Speech, after all, is motion, and imagined speech is simulated motion. I can literally feel subtle movements and tension in my tongue and jaw when I think.)

Most things we do have a cerebellar component, what some neuroscientists call a cerebellar *transform*, smoothing and tuning and relating and fluently switching between the basic building-block abilities (sensory perception, motion, comprehension) that reside in the cerebrum.

The cerebellum regulates rate, rhythm, speed, contextual appropriateness; damage it, and the same building block actions are still possible, but all out of whack, clumsy and disproportionate.

This is consistent with the “embodied cognition” worldview where sensorimotor functions are *on a continuum with* or *of the same kind as* cognitive functions; thought is just “inner” motion and/or “inner” sensation. (And even abstract thought is built up of motions and sensations, via analogy or metaphor.)

The cerebellum may also inspire artificial-intelligence approaches somewhat, especially approaches to robotics or other control, in that it may be be beneficial to include a fast feedforward-only predictive modeling step to control real-time actions, alongside a slower training/updating pathway for model retraining. (I may formalize this more in a later post).