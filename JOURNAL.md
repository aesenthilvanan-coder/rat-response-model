# eliminating animal testing, i will do it.

Nov 18, 2026 · Aaryan Senthilvanan

## contents

- [dead end 1, just train the biggest thing](#dead-end-1-just-train-the-biggest-thing)
- [dead end 2, atoms up](#dead-end-2-atoms-up)
- [april 2, on a train](#april-2-on-a-train)
- [rattusos vs rattus](#rattusos-vs-rattus)
- [the sandwich](#the-sandwich)
- [the actual object](#the-actual-object)
- [the capability list and what i actually believe about it](#the-capability-list-and-what-i-actually-believe-about-it)
- [R01 genome, R02 molecular](#r01-genome-r02-molecular)
- [R03 cellular, R04 tissue](#r03-cellular-r04-tissue)
- [R05 organ, R06 systemic](#r05-organ-r06-systemic)
- [R07 neural](#r07-neural)
- [R08 cognitive, R09 sensory](#r08-cognitive-r09-sensory)
- [R10 biomechanical, R11 environmental, R12 intervention](#r10-biomechanical-r11-environmental-r12-intervention)
- [shared state and why provenance became a first class citizen](#shared-state-and-why-provenance-became-a-first-class-citizen)
- [time, and why a single timestep is impossible](#time-and-why-a-single-timestep-is-impossible)
- [the molecular band, actual ML](#the-molecular-band-actual-ml)
- [the learned band](#the-learned-band)
- [inference and uncertainty, the machinery](#inference-and-uncertainty-the-machinery)
- [compute](#compute)
- [data, what exists and what doesnt](#data-what-exists-and-what-doesnt)
- [the generalization ladder](#the-generalization-ladder)
- [abstention is the product](#abstention-is-the-product)
- [how not to fool myself](#how-not-to-fool-myself)
- [the roadmap](#the-roadmap)
- [the dopamine slice, in detail](#the-dopamine-slice-in-detail)
- [what ships first](#what-ships-first)
- [governance, and the rules i expect to break](#governance-and-the-rules-i-expect-to-break)
- [what kills this](#what-kills-this)
- [HUMANS](#humans)
- [why the genome swap fails](#why-the-genome-swap-fails)
- [the inference ladder](#the-inference-ladder)
- [the engagement invariance hypothesis](#the-engagement-invariance-hypothesis)
- [the rosetta set](#the-rosetta-set)
- [learning the translation operator](#learning-the-translation-operator)
- [the human data situation is better than the rat one](#the-human-data-situation-is-better-than-the-rat-one)
- [meiosis already ran the knockouts](#meiosis-already-ran-the-knockouts)
- [human PK and the variability that actually matters](#human-pk-and-the-variability-that-actually-matters)
- [where im completely stuck](#where-im-completely-stuck)
- [the counterfactual problem](#the-counterfactual-problem)
- [virtual populations, the near thing](#virtual-populations-the-near-thing)
- [individual twins, the far thing](#individual-twins-the-far-thing)
- [the part that bothers me](#the-part-that-bothers-me)
- [where i actually am](#where-i-actually-am)
- [not going to oversell the ending](#not-going-to-oversell-the-ending)

---

yall wanted to know what im doing the last few months so.

aug?? backdating, only started keeping this properly in oct

went vegan in feb and it wrecked my research, which i did not see coming

not dramatic. nobody took anything from me. its that every number i have ever trained on came out of an animal. IC50 tables. PK curves. behavioral readouts. tox panels. i treat those as data the way you treat a csv as a csv, and then i was cleaning a rat hepatotox set and actually read the column headers. necropsy day. terminal sacrifice.

floor went

obvious move is stop using it. thats stupid though. the data exists, the animals are already dead, refusing to use it un-kills nothing, just means i do worse science carrying the same debt. only move that changes anything is making the NEXT dataset unnecessary

so. bigger problem. been chewing since

reason i dont think its insane is i already build most of the pieces. the interactome thing takes two proteins never seen together and predicts whether and how they interact. thats not a lookup table, it generalizes into pairs nobody has ever assayed. and the bias model does the same one layer up, ligand + receptor -> which pathway it prefers, Gi vs arrestin, which is the entire difference between an analgesic and a respiratory depressant

sat in a session at IWBBIO thinking ok. i can predict a binding event. i can predict pathway bias. what is actually between me and predicting what the animal does

answer was a lot. so much. this is me finding out how much, roughly in order, including the stretches where i was confidently wrong

leaving the dead ends in because the dead ends are the content

some of the tables dont finish. ill get to them

also stop calling it "the rat thing" in emails, people think im doing something bad TO rats

## dead end 1, just train the biggest thing

first instinct obviously. its 2026 and my whole toolkit is large model large corpus hope

pitch i wrote myself in feb, copying out of notes:

take every rat study ever published. every open field, rotarod, forced swim, LD50, tox panel, telemetry trace. pair each with (compound, dose, route, strain, sex, age). train a huge multimodal sequence model. new compound in, behavioral readout out. no more rats

thought this was basically an engineering problem for about three weeks

not a data volume problem, which is what i assumed the wall would be. published rat literature is enormous. DrugMatrix alone is thousands of compounds across tissues with transcriptomic readouts. TG-GATEs has rat liver tox at multiple doses and timepoints. decades of behavioral pharm. fragmented and badly annotated but its there

what kills it is that the model learns the wrong function

think about what the training signal contains. feed it (compound -> outcome) pairs and the cheapest route to low loss is a structural similarity lookup. haloperidol-shaped things reduce locomotion, amphetamine-shaped things increase it. works beautifully until you hand it something new, at which point it interpolates between the two nearest things in chemical space and is wrong

and wrong in exactly the cases i built it for. for a minor analog of something characterized i dont need the model, a med chemist can tell me. for a novel scaffold, the only case where replacing an animal study has value, its extrapolating with no mechanism

the failure is silent too. similarity lookup cannot tell apart "ive seen a hundred close analogs" from "this is 0.3 tanimoto from anything i know". same confident number either way

nightmare case i keep going back to. compound binds the same receptor as a known drug, same affinity, but biased toward arrestin instead of G protein. structurally could be a trivial analog. behaviorally it can be the opposite drug. the similarity model has no way to see it because the thing that matters isnt in the similarity metric, its in the signaling. i know this specifically because the bias model i built exists on the premise that structure does not predict bias

so the lesson that turned into the foundation of everything after:

a model that maps intervention to outcome without representing whats in between cannot know when its out of its depth. the intermediate layers arent for interpretability, theyre the thing that makes the uncertainty estimate mean anything

clever hans framing helped. you can have a system that passes every test for reasons unrelated to what you think youre measuring, and with something this expensive to validate i would not find out for years

dead. keeping the corpus idea though, it comes back in a different shape

## dead end 2, atoms up

natural overcorrection. problem with the first one was no mechanism so: all mechanism. bottom up

most of march here, most fun least productive month of the year

fantasy is clean. you have a genome, you have physics, everything above is implied in principle. simulate proteins, proteins make cells, cells make circuits, circuits make rat. no statistics anywhere. generalization free because physics doesnt care if a molecule is in your training set

this does not fail on compute. everyone assumes compute. its not compute

its parameters you cannot measure, in quantities that are stupid

counting cured me. one neuron. decent biophysical model is multicompartment HH-type, say 200 compartments, ~10 channel types each, each channel 5-10 kinetic parameters, activation inactivation voltage dependence conductance density calcium sensitivity. call it 1e4 for ONE neuron and thats generous to me

rat brain is ~2e8 neurons

| scale | count | params each | total |
| --- | --- | --- | --- |
| neuron biophys | 2e8 | 1e4 | 2e12 |
| synapse | 1e12 | 20 | 2e13 |
| protein species / cell | 1e4 | ? | ? |
| metabolite pools | ? |  |  |
| gave up here |  |  |  |

measured values exist for maybe a few thousand of those. the rest you fit, or copy from a different cell type in a different species at a different temperature, or invent and label biologically plausible

blue brain. i had the lazy version of this opinion and had to go actually read. what they built is real, detailed cortical microcircuit reconstruction, careful work on morphologies and channel distributions. what did not happen is the thing the press implied, a simulated brain that does things

the diagnosis isnt that they were bad at it. its that fidelity at one scale does not compose into function at the next. you can have exquisite single neurons and a network that does nothing, because what makes the network work is connectivity and plasticity rules and neuromodulatory state, and none of those are implied by the neurons

third thing, and this is what actually ended it. the data to constrain it does not exist for rat

this one stung. i assumed rat was the best characterized mammal. its not, mouse is. connectomics money went to mouse. genetic toolkits mouse. cell atlases mouse first. rat has better behavioral and pharmacological literature, decades of it, because rats are big enough for surgery and smart enough for complex tasks. but worse structural data

so for rat i have excellent drug response and behavior, mediocre wiring. meaning a bottom up reconstruction is constrained exactly where i dont need it and unconstrained exactly where i do

so bottom up is dead no. wrong as a global strategy, right as a local one. took another month to see that difference and its the hinge of the whole thing

wrote this at the end of march, still the most useful sentence i have:

i dont have to simulate the rat. i have to simulate the rats response to things. not the same problem and the second one is maybe a thousand times smaller

## april 2, on a train

ok this is the one. this is what turned it from a fantasy into a project

question i had been asking was how do i simulate a rat. unanswerable. no boundary, no point at which youre done

question i should have been asking:

given a rat in state X and an intervention u, what is the distribution over states at t+dt

thats an operator. call it phi. map from (state, intervention) to a distribution over futures. the whole project is approximate phi well enough over a defined domain of u that you dont need the animal

why this does so much work

it bounds the thing. i dont need every state variable, i need the ones the intervention perturbs and the ones the readout depends on. everything else coarse or absorbed into a constant or gone. drug hits D2, i do not need olfactory epithelium at any resolution. scope comes from the question not from the organism

it tells me where mechanism matters. mechanism is expensive, spend it where novelty enters, and novelty enters at exactly one place, the molecular interface. a new compound is new as a molecule. by the time its effect reaches striatal output is suppressed, its not novel anymore. suppressed striatal output is suppressed striatal output whether it came from haloperidol or from something nobody has synthesized

the line i keep coming back to:

novelty is not conserved across scales. its injected at the molecular layer and it dissipates upward. a new drug is a new molecule, it is usually not a new physiology

so the generalization burden is not uniformly distributed. it sits almost entirely on molecular and signaling. the layers above can be learned from existing data because theyre being asked to respond to perturbations that are, in their own coordinates, familiar

huge relief and also the entire architectural thesis

also makes validation piecewise. does molecular predict binding. does signaling predict cAMP. does circuit predict firing rate change. each of those is a real experiment people have run thousands of times on archived tissue or in culture, no new animals

and failure becomes legible. prediction is wrong, i ask which link broke. black box gives you a number and a shrug

flagging for anyone reading who knows the field, this is not a new idea. i spent two weeks thinking i invented something

what i actually did was reinvent, badly, the premise of quantitative systems pharmacology. QSP has done mechanism based multiscale drug response for twenty years. theres commercial PBPK software. theres the virtual population literature. pharma uses it in submissions

briefly devastating then became the best news in the project, because the hard part has precedent and the precedent worked. QSPs limitation is scope. each model hand built for one question one pathway one indication by specialists over years. doesnt scale, doesnt generalize to unseen compounds

which is precisely what modern ML is good at

so the project isnt invent mechanistic pharmacology. its take QSPs architecture and make the parameter and structure problem a learning problem

## rattusos vs rattus

had two designs by may, philosophically incompatible, could not pick. writing both out because the resolution IS the architecture

A. rattusos, the foundation model one. the version i pitch when im excited. ingest everything, published and raw, every modality, behavioral assays telemetry PK tox transcriptomics ephys. harmonize into one latent dynamical system. learn the continuous state space of rat biology directly from observation without ever needing a wiring map. then run it closed loop, it reacts to stimuli, you perturb it, it tells you what happens

the appeal is real. it uses data that already exists, which is the entire moral point. learned latents handle the parts of biology nobody has equations for, which is most of biology. it sidesteps the parameter counting disaster completely, you never specify a channel conductance. and its the only version that could plausibly capture sickness behavior or stress-state dependent drug response, phenomena that are real and matter enormously for interpretation and have no clean mechanistic model at all

B. rattus, mechanistic. the version i pitch when someone senior is in the room. explicit subsystems, genome molecular cellular tissue organ systemic neural cognitive sensory biomechanical environmental intervention. each a real model of a real process. shared state. things propagate because theyre caused, not because a transformer learned a correlation

generalizes by construction where it matters. traceable, which is the only thing a regulator will ever accept. uncertainty attaches to specific mechanisms instead of to the output as a whole. fails loudly, violate mass balance and the sim tells you

couldnt choose because each ones weakness is the others strength, almost suspiciously symmetric

|  | learned | mechanistic |
| --- | --- | --- |
| unseen compound | interpolates silently | extrapolates from physics |
| undescribed biology | fine | cannot represent it |
| data hunger | enormous | moderate, needs the right data |
| traceability | none | complete |
| regulator accepts | no | eventually |
| calibrated uncertainty | hard | natural |
| stress state effects | yes | no |
| cost to build | large | insane |

kept flip flopping based on, embarrassingly, who i talked to most recently

what broke it was realizing the question is malformed. i was asking which paradigm is correct for the organism and no single paradigm is correct for the organism, because the organism is not epistemically uniform

some layers we understand to several decimals. receptor ligand binding thermodynamics. enzyme kinetics. membrane biophysics. cardiac electrophys

some we understand structurally but not quantitatively. signaling topology, we know what phosphorylates what, we do not know the in vivo rate constants

some we have enormous observational data and zero equations. behavior. sickness. stress

so the paradigm varies by layer according to what we actually know about that layer. obvious written down, took six weeks

the rule for which goes where falls out of the novelty argument:

mechanism where novelty enters. learned dynamics where data is dense. hard constraints everywhere, non negotiable

started calling it the mechanistic sandwich which is a terrible name and it stuck

## the sandwich

three bands. assignment is by epistemic status, not by biological scale, which is the part people get wrong when i explain it badly

```
   u      (compound / knockout / stimulus / environment)
            |
      v
[ MECHANISTIC BAND ]
   structure, docking, kon koff, occupancy, efficacy, bias
   real physics, real equations
   novelty is handled HERE and nowhere else
        |
              |      interface = engagement vector
              |      (target, occupancy, efficacy, bias, timecourse, conf)
      v
[ LEARNED BAND ]
   signaling -> cellular -> tissue -> circuit
   neural ODEs, latent SDEs, operator learning
   trained on perturbation data that already exists
   never sees the molecule. on purpose.
              |
              v
[ CONSTRAINED BAND ]
 systemic physiology, PBPK, mass balance, charge balance,
   thermodynamic feasibility, allometry
   hard constraints + classical compartment models
              |
              v
   readouts: firing rate, locomotion, HR, exposure, syllables
```

the critical design decision, and ive now written it in three separate notebooks because i keep needing to remind myself:

the learned band never sees the compound. it only sees the engagement vector. that is the whole trick

if the learned middle took a molecular fingerprint as input it would immediately learn compound identity shortcuts and im back to dead end 1 with extra steps. forcing everything through a low dimensional engagement bottleneck means the only route from new molecule to predicted behavior is through mechanism. the generalization becomes structural instead of hoped for

its an information bottleneck argument. deliberately starve the learned part of the one feature that lets it cheat

consequence i like: two different molecules producing identical engagement vectors MUST produce identical predictions. thats falsifiable and its a real biological claim, and when its wrong, probably off target burden or some membrane effect i havent modeled, the model fails in a way that tells me exactly what mechanism i left out

hard constraints are not decoration. a learned middle will happily produce negative concentrations, non conserved mass, cAMP from nowhere. so positivity by construction, work in log space or softplus the outputs. mass balance as a projection after each integration step. thermodynamic consistency on every loop in the reaction graph, the Wegscheider conditions, which i ignored for a month and paid for. anything that violates gets flagged not clipped. silently clipping is how you end up quietly wrong for months

still undecided whether constraints go in as projection or penalty. penalty trains easier, projection is the only one i trust. probably projection with a penalty warmup. need to ask someone who does scientific ML properly

## the actual object

writing the formalism down properly because i kept being sloppy about it in conversation

computational organism = a dynamically evolving mathematical representation of a biological individual, with enough internal state and enough interacting mechanism to reproduce specified responses under specified interventions. the word specified is doing all the work in that sentence and i want to keep it there

state at time t:

$$
X(t) = \begin{bmatrix} X_m(t) \\ X_c(t) \\ X_t(t) \\ X_o(t) \\ X_n(t) \\ X_b(t) \end{bmatrix}
$$

molecular, cellular, tissue, organ/systemic, neural, behavioral/organism-environment

evolution:

$$
\frac{dX}{dt} = F(X, U, E, G, \Theta, t) + \epsilon(t)
$$

U interventions, E environment, G genotype, Theta individual biological parameters, epsilon the stochastic part plus everything unresolved that i am pretending is noise

that epsilon term is load bearing and slightly dishonest and i want to flag it now. half of what goes in there is genuine biological stochasticity, channel noise, transcriptional bursting, synaptic release probability. the other half is mechanism i havent modeled. those are completely different things and lumping them makes the model look better calibrated than it is. splitting them is most of what the uncertainty section is about

the thing that makes this harder than a textbook ODE system is that biology is not purely continuous

continuous: membrane potential, concentrations, metabolic flux, muscle dynamics, diffusion

discrete: action potentials, vesicle release, cell division, apoptosis, behavioral state transitions, and the interventions themselves

so its a hybrid dynamical system. piecewise continuous flow with event-triggered jumps. formally

$$
\dot{X} = F(X,\cdot) \quad \text{while } g_k(X) > 0, \qquad X^{+} = R_k(X^{-}) \quad \text{when } g_k(X) = 0
$$

g_k are guard functions, R_k are reset maps. a spike is a guard crossing with a reset. so is a cell dividing

the engineering pain here is real and i underestimated it. event detection needs root finding inside the integrator with dense output, you cant just check at step boundaries or you miss events and the whole thing drifts. and if two guards fire in the same step you need a deterministic ordering or the sim is not reproducible, which kills the reproducibility requirement before you even start

spent two weeks on event handling alone. nobody writes papers about event handling

also Zeno behaviour is a genuine risk. a system that triggers infinite events in finite time. happens with oscillators near threshold. currently handled by minimum inter-event time, which is a hack and i know its a hack

## the capability list and what i actually believe about it

wrote out twenty things the finished platform has to do. then went back through and marked where i honestly think each one sits. keeping both columns because the gap between them is the real roadmap

| # | capability | where it is |
| --- | --- | --- |
| 1 | genetically specified rat | mostly solved, its a data problem |
| 2 | major anatomical structures | solved for the atlas level |
| 3 | molecular interactions | strong, this is my home turf |
| 4 | cellular activity | decent and moving fast |
| 5 | intercellular communication | partial, paracrine ok, gap junctions bad |
| 6 | organ specific physiology | good for heart liver kidney, poor elsewhere |
| 7 | whole body integration | PBPK yes, everything else no |
| 8 | neural connectivity + activity | mesoscale only, no synapse level for rat |
| 9 | sensory perception | olfaction is embarrassing. vision ok-ish |
| 10 | coordinated motor activity | tractable, physics engines are good now |
| 11 | learning and memory | plasticity rules exist, which ones apply where does not |
| 12 | adaptation to environment | barely started |
| 13 | pharmacokinetics | solved honestly, oldest part of the field |
| 14 | pharmacodynamics | solved for simple cases, not for chronic |
| 15 | response to unseen interventions | THE point, unproven |
| 16 | longitudinal adaptation | hard, see tolerance below |
| 17 | interindividual variation | rat strains are inbred so this is weirdly harder than human |
| 18 | behavioral consequences | the worst link |
| 19 | uncertainty quantification | designed, not validated |
| 20 | reproducible + independently verifiable | engineering, just tedious |

these are cumulative and thats the part people skip. a convincing behavioral imitation does not establish molecular reconstruction. you can get 18 without 3 through 12 and have nothing

#16 deserves more than one line. chronic effects are where the whole mechanistic story earns its keep or doesnt. repeated exposure changes receptor expression, receptor sensitivity, signaling gain, gene expression, synaptic responsiveness, metabolic set points. tolerance, sensitization, withdrawal. a model that only does acute response is a model of the first dose, and almost nothing clinically interesting happens on the first dose

so the cellular band needs slow adaptive state with its own dynamics, not just fast signaling. thats the min-to-days timescale and its the one that makes multirate integration unavoidable rather than just efficient

scope boundaries, written down so i stop quietly expanding them

first version is Rattus norvegicus, adult, lab strain. the architecture has to eventually carry embryonic development, neonatal, adolescence, sexual maturation, adulthood, aging, disease states, injury and recovery. it does not have to carry them now. but the state representation has to have somewhere to PUT them, which is why developmental stage is a field in the organism state and not a global config flag. learned that one the annoying way after hardcoding adult parameters in about forty places

both sexes. non negotiable and i want to say why because its not just an inclusion checkbox. sex differences in rat drug response are large and mechanistic. different CYP expression profiles, different body composition so different volume of distribution, hormonal modulation of basically every neurotransmitter system, estrous cycle effects on receptor density. collapsing to one average organism doesnt give you an average, it gives you a chimera that matches neither. and historically most behavioral pharm was done in males only, so the data itself is skewed and the model will inherit that skew unless i handle it explicitly

reference organism is one specific documented genotype first, outbred populations later. starting with a population when i cannot yet get one individual right is backwards

## R01 genome, R02 molecular

working through the twelve subsystems properly. these two are the foundation and theyre the two i feel best about

R01, genome engine. the genetic basis for everything above it

needs reference genome representation, chromosomal organisation, gene sequences, exons introns, regulatory sequence, noncoding RNA, mitochondrial DNA, structural variants. then individual variation on top, SNVs indels CNVs allelic variation haplotypes epigenetic marks. then regulation, TF binding promoter enhancer chromatin accessibility methylation histone marks RNA processing

the thing i have to keep hammering on, and the thing most pipelines quietly get wrong: DNA sequence is not gene expression state. a gene being present does not establish that its expressed. transcript abundance does not establish protein abundance. the correlation between mRNA and protein is famously mediocre, somewhere around 0.4 depending on tissue and method, because translation rate and protein half life vary over orders of magnitude

so the genome engine does not output "this gene is on". it outputs a context dependent distribution over expression given tissue, developmental stage, environment, and current cell state. which means its already a dynamical model rather than a lookup, and it has to be coupled to the cellular engine rather than sitting underneath it

validation against independent transcriptomic data, tissue specific and developmental, scored at both individual gene and pathway level. gene level will look bad and pathway level will look fine and i should report both rather than the flattering one

R02, molecular engine. the foundation for intervention generalization, so the most important single subsystem in the project

has to represent small molecules, proteins, peptides, nucleic acids, lipids, carbohydrates, ions, metabolites, protein complexes, membrane associated structures. each with physical and chemical properties rather than an identifier

structural capability: 3D structures, conformational ensembles not single structures, binding site identification, docking, molecular dynamics, solvent, membrane environments, allostery, protein protein interactions, stability. QM where classical approximations break, which is mostly metal centers, covalent chemistry, and proton transfer

the conformational ensemble point matters more than people act like it does. a single crystal structure is a photograph of a thing that moves. for GPCRs especially, the whole functional story is a conformational equilibrium between states that couple preferentially to different transducers. if you dock into one rigid structure you have already thrown away the mechanism of biased agonism

interaction engine simulates or estimates binding affinity, kon, koff, occupancy, conformational transitions, agonism, antagonism, partial agonism, inverse agonism, allosteric modulation, competitive and noncompetitive interactions, covalent where relevant

and the rule that the whole project rests on: a binding event is not functional activation. occupancy, efficacy, signaling bias, and downstream effect are four separate things and conflating them is the single most common error in computational pharmacology. you can have 90% occupancy and zero functional response. thats what an antagonist IS

for GPCRs specifically the target representation needs receptor structure, ligand binding sites, G protein coupling, arrestin pathways, internalization, desensitization, recycling, degradation, dimerization where relevant, tissue specific expression. the trafficking half of that list is usually left out of models and its the half that determines what happens on day fourteen of dosing

output per compound: structural characterization, candidate targets, predicted binding, kinetics, functional activity, off target interactions, confidence intervals, and an explicit list of unresolved molecular uncertainties. that last field is the one that makes it usable

and it has to handle molecules absent from training data, which is the entire reason this band is mechanistic instead of learned

## R03 cellular, R04 tissue

R03. dynamically responsive virtual cells. the layer where molecular events become biology

first pass cell classes, prioritised by what the dopamine slice needs plus what the tox story needs

neural: dopaminergic, GABAergic, glutamatergic, serotonergic, cholinergic glia: astrocytes, microglia, oligodendrocytes peripheral: hepatocytes, cardiomyocytes, renal epithelial, endothelial, immune, smooth muscle, skeletal muscle

each virtual cell represents plasma membrane, nucleus, mitochondria, ER, golgi, ribosomes, cytoskeleton, vesicles, at whatever resolution the question needs

processes: transcription, translation, protein folding, trafficking, degradation, receptor trafficking, ion transport, membrane potential, calcium signaling, intracellular cascades, energy metabolism, oxidative stress, adaptation, apoptosis, autophagy, repair

the thing that makes this hard rather than tedious is context dependence. identical receptor occupancy produces different downstream effects in different cell types because the signaling machinery and regulatory state differ. same D2 receptor, same ligand, same occupancy, completely different outcome in a striatal MSN versus a pituitary lactotroph. so the cell type is not a label on the model, its part of the model

im handling this with a mixture of experts router over cell identity, which is a pragmatic choice rather than a principled one. the principled version would have cell identity emerge from the expression state. i am not there

adaptation over time is the central requirement and the one i keep underestimating. repeated drug exposure alters receptor expression, receptor sensitivity, intracellular signaling, gene expression, synaptic responsiveness, metabolic activity, stress responses. acute and chronic are different regimes and a model that only does acute is a toy

intercellular communication: neurotransmitters, hormones, cytokines, growth factors, extracellular vesicles, electrical coupling, direct contact, ECM interactions. a cell is not an isolated object when its behavior depends on its neighbours, which is always

R04. tissue. integrating cells into spatially organised structures

needs 3D cellular organisation, cell type distributions, ECM, local diffusion, oxygen gradients, nutrient gradients, local signaling, mechanical interactions, remodeling, migration, damage and repair

spatially resolved where it matters: neural tissue, liver lobules, renal structures, cardiac muscle, vascular networks, GI tissue

liver zonation is the clearest case for why you cannot average a tissue. periportal and pericentral hepatocytes have different enzyme complements, different oxygen exposure, different metabolic roles. acetaminophen toxicity is zone specific. model the liver as a well mixed compartment and you get the wrong answer about the most studied hepatotoxin there is

same logic in brain. receptor density varies continuously across striatum, dorsal to ventral, and the behavioral consequence of a D2 antagonist depends on where it binds, not just how much

so cellular processes have to differ by anatomical location, which means the tissue engine is not a container for cells, its a field that modulates them

validation against histology, imaging, spatial transcriptomics, functional measurements. spatial transcriptomics is the one that makes this tractable at all and its only recently good enough

## R05 organ, R06 systemic

R05. every organ gets its own physiological state and talks to the others through the systemic engine

the list, which is long and i am not building most of it any time soon

nervous: brain, spinal cord, PNS cardiovascular: heart, arteries, veins, capillaries respiratory: lungs, airways digestive: mouth, esophagus, stomach, small intestine, large intestine, pancreas, liver, gallbladder renal: kidneys, ureters, bladder endocrine: pituitary, thyroid, adrenals, pancreatic endocrine, gonads, hypothalamic regulation immune and lymphatic: bone marrow, spleen, thymus, lymphatics, immune cell populations musculoskeletal: bones, skeletal muscle, tendons, ligaments, joints integumentary: skin, hair, associated sensory structures

each organ needs anatomical representation, cellular composition, functional state variables, blood supply, neural inputs, hormonal regulation, metabolic activity, injury responses, repair

the requirement i keep forgetting and then rediscovering: it has to support disease states and abnormality, not assume a permanently healthy organism. most physiological models are models of health. the whole point of a tox prediction is the unhealthy case

R06. systemic. the interactions that cannot be understood by modeling organs independently

circulation, blood pressure regulation, oxygen transport, CO2 transport, nutrient distribution, hormonal circulation, immune signaling, temperature regulation, fluid balance, electrolytes, acid base, energy balance, metabolism, waste elimination

PK first because its the most mature thing in the entire project and its what makes the drug story work at all

absorption distribution metabolism excretion. route of administration, absorption rate, plasma concentration, protein binding, tissue distribution, blood brain barrier penetration, metabolic transformation, active metabolites, renal clearance, hepatic clearance, repeated dosing, accumulation, drug drug interactions

PBPK is genuinely solved-ish for rat and thats a gift. physiologically based compartments with real organ volumes and real blood flows, partition coefficients from logP and tissue composition, clearance from in vitro microsomal data scaled up. this part i can basically buy off the shelf conceptually

the part that is not solved is active metabolites. a compound that is inert until the liver makes it into something else is a compound whose entire pharmacology lives downstream of a metabolic step you have to predict. and metabolite prediction is much worse than parent compound prediction

PD connects exposure to response. concentration response, target engagement, efficacy, toxicity, delayed effects, tolerance, withdrawal

the delayed effect problem is underrated. SSRIs occupy the transporter within hours and the clinical effect takes weeks. so the PD model cannot be a static concentration-effect curve, it has to carry slow adaptive state. which again is the chronic problem from the capability list showing up in a different costume

homeostasis last and its what makes it an organism rather than a pile of organs. blood pressure drops, compensatory cardiovascular mechanisms engage. glucose drops, endocrine response. temperature changes, physiological and behavioral response. feedback represented explicitly, not implied

and the feedback is what makes drug effects non-monotonic in ways that break naive models. give a vasodilator, pressure drops, baroreflex kicks in, heart rate rises, and the net effect at two hours looks nothing like the effect at ten minutes

## R07 neural

the subsystem the whole project is really about, because replacing behavioral experimentation is the stated goal and behavior comes out of here

anatomical coverage for the first reconstruction: cerebral cortex, prefrontal, motor, somatosensory, visual, auditory, olfactory structures, hippocampal formation, amygdala, striatum, nucleus accumbens, globus pallidus, substantia nigra, VTA, thalamus, hypothalamus, cerebellum, brainstem. plus spinal cord and PNS

thats the list. im building striatum and its immediate partners and leaving the rest as stubs with documented interfaces

five levels of neural representation, coexisting in one simulation, which is the part that makes it hard

A, biophysical neurons. membrane dynamics, ion channels, electrical activity, synaptic mechanism. multicompartment HH. expensive, accurate, used only where the question demands it

B, reduced models. adaptive exponential integrate and fire, Izhikevich, that family. cheap, captures spiking statistics, throws away dendritic computation

C, neural populations. mean field, firing rate models, neural mass. this is where most of the striatal circuit will live

D, circuits. anatomically and functionally connected networks with real connectivity constraints

E, whole brain. large scale interactions between systems, mostly as a boundary condition on the parts i actually model

different levels coexist in one run. so i need level-crossing interfaces that dont lie. a biophysical neuron embedded in a population model has to receive input that is statistically correct, which means the population model has to emit not just a rate but a correlation structure. getting this wrong produces neurons that fire too regularly and i spent a while confused by that

synaptic mechanisms: excitatory, inhibitory, transmitter release, receptor activation, plasticity, LTP, LTD, short term facilitation, short term depression, neuromodulation, receptor adaptation, synaptic remodeling

short term plasticity matters more than people building rate models want it to. a synapse that depresses at high rates is a low pass filter and it changes what the circuit computes, not just how loudly

transmitter systems, all of which have to interact rather than exist as separate models: dopaminergic, serotonergic, GABAergic, glutamatergic, cholinergic, noradrenergic, histaminergic, opioidergic, endocannabinoid

the interaction requirement is not optional and its where most single-system models fail. dopamine release is modulated by cholinergic interneurons, which are modulated by thalamic input, which is gated by dopamine. you cannot model the dopamine system in isolation because it is not in isolation. the acetylcholine-dopamine interaction in striatum is a genuine closed loop and if you leave it out your D2 antagonist predictions are wrong in a specific, reproducible direction

validation against electrophysiology and imaging: action potential distributions, firing rates, oscillatory activity, synaptic responses, population activity, functional connectivity, sensory responses, changes after pharmacological intervention

and the line i put at the top of the neural engine readme: a convincing brain visualisation is not evidence of a functionally accurate brain model. pretty renders have done real damage to this field's credibility and i am not adding to it

## R08 cognitive, R09 sensory

R08. learning, memory, motivation, decision making, adaptive behavior

hard constraint i put on myself early: this has to emerge from neural dynamics, sensory input, bodily state, and environmental experience. it cannot be a library of behavioral scripts. the moment it becomes a lookup of "drug class X produces behavior Y" the whole project has collapsed back into dead end 1 wearing a costume

capabilities: associative learning, reinforcement learning, habituation, sensitization, spatial learning, memory formation, retrieval, consolidation, reward processing, motivation, action selection, behavioral inhibition, exploration, decision making, social learning, circadian influence

learning architecture needs internal representations of the environment that update through experience. prediction error learning, reward learning, state estimation, memory dependent behavior, adaptive action selection, context dependence

and learning has to modify internal state in a way that changes future responses. otherwise its a controller, not an organism

the part that makes this a biology problem rather than an RL problem: the cognitive engine has to receive internal bodily state. hunger, thirst, fatigue, pain signaling, temperature, stress physiology, hormonal state, reward signaling, prior experience. and those have to actually drive behavioral choice

this is the interoception loop and its the difference between a policy and an animal. a sated rat and a hungry rat facing identical stimuli do different things, and the difference is not in the policy weights, its in the body. most RL-flavoured models of animal behavior quietly skip this and then wonder why motivational manipulations dont reproduce

validation across multiple tasks: locomotion, exploration, spatial navigation, novel object, reward learning, memory tasks, social interaction, motor coordination, sleep wake transitions

and no single task is a complete measure of a cognitive construct. this is the degeneracy problem from the behavior link and its worth restating here because its the thing that makes cognitive validation genuinely hard rather than just laborious

R09. sensory. behavior cannot be reconstructed without the stimuli the organism responds to, which is obvious and i still almost skipped it

vision: light intensity, spatial structure, motion, contrast, visual field, retinal processing. rat vision is poor and dichromatic and i kept over-modeling it because i was thinking like a primate

audition: frequency, amplitude, temporal patterns, localization, processing. rats hear well into ultrasound and use it socially, which matters for any social paradigm

olfaction: chemical detection, odor discrimination, receptor activity, processing. this is the dominant modality for a rat and its the one computational neuroscience has handled worst. roughly 1200 olfactory receptor genes. the combinatorial code is not solved. i am currently representing odor as a learned embedding and calling it a stub, which is unsatisfying

somatosensation: touch, pressure, temperature, mechanical, nociception. whiskers are a whole sensory system and deserve their own model. a rat navigating in the dark is doing whisker-based tactile exploration, not vision

proprioception: joint position, muscle state, movement, body orientation

vestibular: balance, acceleration, head orientation

interoception: internal physiological signals, visceral sensation, metabolic state. ties straight back into R08

and the sensory engine integrates across modalities rather than running them in parallel. the same visual stimulus produces different behavior depending on prior learning, physiological state, and simultaneous auditory or olfactory context. multisensory integration is not a postprocessing step

## R10 biomechanical, R11 environmental, R12 intervention

R10. the physical body and its capacity to interact with the world

skeletal geometry, joint structure, muscle groups, tendons, ligaments, mass distribution, inertial properties, skin and soft tissue, internal organ geometry

has to simulate locomotion, walking, running, turning, jumping, climbing, grooming, feeding, drinking, object manipulation, postural control, balance, motor adaptation

the requirement that makes it part of the organism rather than an animation: movement emerges from neural commands through the motor system, and the neural engine receives sensory feedback from the body's actual simulated movement. closed loop. if the body is driven by a motion library instead of by motor cortex output then the behavioral prediction is theatre

this is also where i get to lean on work that already exists. physics engines are good now, and the virtual rodent result showed that a physically simulated rat body driven by a trained policy produces neural activity structure that matches real recordings. thats the strongest existence proof in the whole field for behavior-level simulation being tractable

difference is theirs learned to move. mine has to respond to a drug it has never seen. harder, same stack

validation against 3D trajectories, joint angles, ground reaction forces, muscle activity, gait characteristics, postural adjustments, motor variability

motor variability specifically. not the mean trajectory, the distribution. a drug that makes movement more variable without changing mean speed is doing something real and a mean-based comparison misses it entirely

R11. environmental. the world the virtual rat exists in

temperature, humidity, lighting, sound, odors, food availability, water availability, physical obstacles, spatial geometry, social presence, novelty, time of day

the critical property is that the organism modifies its own exposure through behavior. a rat moves away from a stimulus, eats, explores an object, seeks shelter. those actions change its subsequent sensory and physiological input

so the environment is interactive rather than a fixed stimulus schedule. and that closes the final loop: behavior changes environment changes physiology changes behavior. without it you cannot model something as basic as a rat avoiding a drug-paired context, which is half of addiction research

time of day deserves its own mention. circadian state modulates drug response substantially, and most rodent work is done during the animals sleep phase because thats when the humans are at work. so a large fraction of the reference literature is measuring drug effects on sleepy animals and not reporting it as a variable. the model has to carry circadian phase explicitly if its going to reproduce that literature at all

R12. intervention. the interface between rattus and the people using it

categories

pharmacological: small molecules, biologics, peptides, agonists, antagonists, enzyme inhibitors, multitarget compounds, combinations genetic: deletion, overexpression, variants, regulatory modification, mutation physiological: hormonal change, metabolic change, altered physiological input environmental: sensory stimuli, temperature, nutrition, stressors, social conditions neurological: altered neural activity, changed neurotransmission, changed synaptic properties

every intervention encoded with identity, chemical or biological structure, target or intended mechanism where known, delivery method, exposure profile, timing, duration, physiological context, known uncertainty

and the rule that defines the whole system: it must not require a known behavioral outcome to accept an intervention. if you have to tell it what the drug does, it isnt predicting anything

novel intervention capability. a compound that never existed in training is accepted if its physical and chemical properties can be represented. the system estimates interactions, propagates, predicts. and when the mechanism cannot be resolved confidently it says so explicitly rather than guessing. the unresolved-interaction list is an output, not a failure mode

## shared state and why provenance became a first class citizen

twelve subsystems that all have to talk. the thing that makes that possible is a single shared representation, and getting it right took three rewrites

every simulated component carries: unique identifier, biological classification, species designation, anatomical location, structural properties, dynamic state variables, associated parameters, known interactions, temporal history, uncertainty estimates, evidence provenance

`rattus/core/state.py`

```python
from dataclasses import dataclass

@dataclass
class OrganismState:
    # every subsystem carries its own clock + its own dt.
    # flattening this into one vector cost me a week.
    molecular: MolecularState   # occupancies, conformer pops   us-ms
    signaling: SignalingState   # cAMP, Ca2+, PKA, phospho      s-min
    cellular:  CellularState    # expression, surface receptor  min-h
    tissue:    TissueState      # gradients, local diffusion    s-h
    circuit:   CircuitState     # rates, population activity    ms
    systemic:  SystemicState    # plasma conc, HR, temp, pH     s-min
    body:      BodyState        # pose, joints, contacts        ms
    env:       EnvState         # light, odor, food, conspecfic

    t: float
    G: Genotype                 # strain, sex, variants
    theta: Params               # individual biological params
    unc: UncertaintyBundle      # per subsystem, PROPAGATED
    prov: dict                  # measured / inferred / assumed

    def assert_physical(self):
        assert (self.molecular.occ >= 0).all()
        assert (self.molecular.occ <= 1).all()
        assert self.systemic.mass_balance_residual() < 1e-6
```

a receptor, a neuron, an organ, a behavioral event all have to be distinct addressable computational entities. and every interaction has to be traceable across scales

concretely what that buys: a predicted motor abnormality traces backward through the responsible circuits, the cellular signaling, the receptor interaction, and the administered compound. that trace IS the regulatory artifact. without it theres nothing to submit

the architecture cannot be strictly linear and this took me a while to internalise. neural influences endocrine, endocrine modifies neural. behavior changes environmental exposure, nutrition, activity, physiology. bidirectional everywhere. so the subsystem graph has cycles, which means no clean topological execution order, which means fixed point iteration or operator splitting at every coupled step. this is why the integrator is complicated

provenance. this started as a nice-to-have and became the thing i care most about

i had an afternoon where i could not tell whether a conductance value in my own code came from a measurement, from a different species, or from me making it up six weeks earlier. that was genuinely alarming. so now every parameter carries its source and its epistemic status

| status | means | how much i trust it |
| --- | --- | --- |
| measured | direct measurement, this species, cited | high |
| measured-xs | measured, different species | medium, flagged |
| inferred | fitted from data in this model | depends on identifiability |
| assumed | i picked it | low, must be in sensitivity analysis |
| placeholder | literally a guess to make it run | must never reach a prediction |

and the model reports, for any prediction, what fraction of the variance traces back to each category. if a behavioral prediction depends mostly on assumed parameters, the output says that in the confidence block rather than burying it

this is also the mechanism for the no-false-completeness rule. i cannot claim an organ is reconstructed when three quarters of its parameters are status assumed. the data structure makes the claim checkable instead of rhetorical

the ontology work underneath all this is unglamorous and enormous. mapping gene identifiers across databases, reconciling anatomical nomenclature between atlases, deciding whether two papers measuring "striatal dopamine" measured the same thing. its most of phase 0 and nobody will ever cite it

## time, and why a single timestep is impossible

biological processes run across about fifteen orders of magnitude in time and this is the single biggest engineering constraint in the project

| process | timescale |
| --- | --- |
| molecular interactions | ns to s |
| ion channel gating | us to s |
| action potentials | ms |
| synaptic transmission | ms to s |
| intracellular signaling | s to h |
| gene expression | min to days |
| metabolic adaptation | min to days |
| learning and plasticity | min to months |
| behavioral adaptation | s to months |
| development and aging | days to years |

one global timestep is not just inefficient, its wrong. step at 10us to resolve channel gating and you need 1e10 steps to see a transcriptional response. step at 1 minute and action potentials dont exist

so multirate integration, with the fast subsystems substepping inside the slow ones

the thing that took me longest and that i got wrong first: coupling has to preserve causal consistency across rates. you cannot sample the fast system at slow-step boundaries and feed that up. it aliases

i generated a beautiful theta-band oscillation this way that was entirely numerical. it looked real. it had the right frequency. i believed it for two days and showed it to someone before i caught it. the fix is passing time-averaged fast state to the slow integrator rather than endpoint state, which sounds obvious and is the kind of obvious you only see after it has eaten two days

stiffness is the other half. biochemical networks are badly stiff, fast equilibria coupled to slow accumulations, so explicit methods either blow up or crawl. implicit solvers in the Rosenbrock family handle it, Rodas5 specifically, and for the mixed case IMEX splitting, implicit on the stiff linear part and explicit on the nonlinear part

and for the discrete molecular stuff, exact stochastic simulation where molecule counts are low, tau-leaping where theyre moderate, deterministic ODEs where theyre high, with automatic switching based on count thresholds. hybrid stochastic-deterministic. the switching logic is a genuine source of artifacts and i have a test suite that is mostly about catching switching artifacts

here is the integrator as it currently stands

`rattus/core/integrate.py`

```python
def step(state, u, env, horizon):
    """multirate. fast substeps inside one slow step.
    slow vars see the TIME AVERAGE of the fast sweep,
    never its endpoint. endpoint aliases oscillations
    and i generated a beautiful fake theta rhythm that
    way and believed it for two days."""
    while state.t < horizon:
        dt_slow = sched.next_slow(state)

        trace, t_loc = [], state.t
        while t_loc < state.t + dt_slow:
            dt_fast = sched.next_fast(state)      # CFL + stiffness
            state.circuit, state.body = integrate_fast(
                    state, u, env, dt_fast)
            trace.append(snapshot(state))
            t_loc += dt_fast

        avg = time_average(trace)                 # <- the fix
        state.signaling = rodas5(f_sig, state, avg, dt_slow)
        state.cellular  = imex_euler(f_cell, state, avg, dt_slow)
        state.systemic  = pbpk_step(state, u, dt_slow)

        state = project_onto_constraints(state)   # not optional
        state.t += dt_slow
    return state
```

the project_onto_constraints call at the bottom is doing real work. after every slow step, project the state back onto the manifold where mass balance and positivity hold. its a constrained least squares, cheap relative to the integration, and it is the difference between a model that stays physical for a simulated week and one that quietly goes negative on hour three

## the molecular band, actual ML

this is the part i can build today because its the part closest to what i already do

ligand representation is an E(3)-equivariant GNN over the molecular graph with 3D coordinates. equivariance matters here for a practical reason rather than an aesthetic one: a rotated pose is the same pose, and a non-equivariant net has to learn that from augmentation, which wastes capacity and never quite works. EGNN-style message passing with coordinate updates, or tensor field networks if you need higher order features. i use EGNN because its cheaper and the gain from higher order irreps has not been worth it on my benchmarks

pocket representation borrows the invariant point attention idea from structure prediction. attention in a local frame per residue so geometry is handled natively instead of through distance features

pose generation as diffusion over SE(3). you denoise translation, rotation, torsion angles jointly on the product manifold. this is much better than scoring-function docking because it samples a distribution over poses rather than committing to one, and the pose distribution is what you need downstream anyway

then the two heads that matter

kinetics head predicts kon and koff separately rather than KD. this is a deliberate choice. KD = koff/kon loses residence time, and residence time is often what determines in vivo efficacy. a compound with the same affinity but ten times the residence time behaves completely differently, particularly for receptors with fast ligand turnover. the field moved toward kinetic selectivity years ago and most docking pipelines still output a single affinity number

bias head is the one that connects to my own prior work. operational model of agonism, Black-Leff, parameterised as transduction coefficients log(tau/KA) per pathway. the reason to use the operational model rather than raw Emax is system dependence: Emax depends on receptor reserve in the specific tissue, tau/KA does not. so transduction coefficients transfer across tissues and Emax does not, which matters enormously when the same receptor sits in striatum and pituitary

bias factor is then the difference of log(tau/KA) between pathways relative to a reference agonist. this is standard pharmacology and the ML contribution is predicting it from structure, which nobody does well yet including me

`rattus/mol/engagement.py`

```python
class MolecularBand(nn.Module):
    """novelty enters HERE and nowhere else.
    E(3)-equivariant so poses dont need augmentation."""
    def __init__(self, d=256):
        self.lig  = EGNN(d, layers=8, equivariant=True)
        self.pkt  = IPAEncoder(d)                # invariant point attn
        self.pose = SE3DiffusionDocker(d)        # diffdock-style
        self.kin  = KineticHead(d)               # kon, koff -> KD
        self.bias = BiasHead(d, paths=("Gi","Gs","Gq","arrestin"))

    def forward(self, mol, targets):
        zl = self.lig(mol.coords, mol.z, mol.bonds)
        out = []
        for t in targets:
            zp = self.pkt(t.struct, t.pocket_mask)
            pose, logp = self.pose.sample(zl, zp, n=64)
            kon, koff = self.kin(zl, zp, pose)
            tau, KA = self.bias.operational(zl, zp)  # black-leff
            out.append(Engagement(
                target=t.uniprot,
                kon=kon, koff=koff, KD=koff / kon,
                efficacy=tau / (1 + tau),
                bias=self.bias(zl, zp),              # softmax over paths
                conf=self.kin.posterior_width(zl, zp)))
        return out
```

training data for the molecular band is the best-off part of the project. ChEMBL, BindingDB, PDSP Ki, and for bias specifically the published BRET and TRUPATH datasets. its still only a few thousand compounds with real pathway-resolved data, so this head is the data bottleneck even though its conceptually the most mature

the off target problem. polypharmacology is not a nuisance, its usually the mechanism. atypical antipsychotics work partly because theyre dirty. so the molecular band has to screen against the full proteome rather than the intended target, and most of those predictions will have wide uncertainty. the output is a ranked engagement list with confidence, not a single target

and the thing that keeps me up: proteome-wide screening means tens of thousands of targets per compound, most with no structural data, so most engagements are predicted from sequence-level models with poor calibration. the off target list is the least trustworthy part of the most important band

## the learned band

signaling through circuit. where most of the actual ML research risk lives

core object is a latent dynamical system. continuous time, because biology is continuous time and because the data is irregularly sampled, which discrete sequence models handle badly

neural ODE for the deterministic part, latent SDE where stochasticity matters. the SDE version is the right one for anything involving low copy number molecular events or synaptic release, where the noise is structural rather than observational

training through the adjoint. memory is the whole problem: backprop through a stiff solver with adaptive steps is brutal, so the adjoint method with checkpointing, and for SDEs the reversible Heun scheme so the backward pass retraces exactly rather than approximately. without reversibility the gradients are wrong in a way that is hard to notice

universal differential equations is the framing i keep coming back to. write down the mechanism you know, leave a neural network term where you dont

$$
\frac{dz}{dt} = f_{\text{known}}(z, u; \theta_{\text{bio}}) + g_{\phi}(z, u)
$$

f_known is the mass action kinetics and conservation laws i can write from the literature. g_phi is a learned residual that absorbs what i cannot. and crucially you can regularise g_phi toward zero, so the model prefers mechanism and only reaches for the learned term when the data demands it

this also gives a discovery channel i did not expect. if g_phi learns a large consistent correction in one region of state space, thats a pointer at missing mechanism. i can go look at what its compensating for. a few times it has been a known pathway i left out, which is reassuring, and once it was a numerical artifact, which was not

cell type conditioning through mixture of experts. shared dynamics backbone, per-cell- type expert routing, so hepatocytes and MSNs share the parts of signaling that are genuinely shared and diverge where they differ. soft routing rather than hard, because cell identity is a continuum and the hard version produced discontinuities at boundaries that were obviously artifactual

operator learning for the parts where i need a map between function spaces rather than a trajectory. DeepONet with branch and trunk, or Fourier neural operators for anything with spatial structure like tissue diffusion fields. FNOs are genuinely good at this and underused outside fluids

`rattus/mid/dynamics.py`

```python
class LearnedBand(nn.Module):
    """takes Engagement ONLY. never a fingerprint, never
    a SMILES, never a compound id. the assert below is in
    the dataloader because i caught myself cheating."""
    def __init__(self, d=512, n_cell=48):
        self.enc   = EngagementEncoder(d)     # set -> vector
        self.moe   = MoERouter(n_cell, d)     # cell-type experts
        self.drift = MLP([d + d, 1024, 1024, d], act="swish")
        self.diff  = DiagDiffusion(d)         # latent SDE noise
        self.dec   = nn.ModuleDict({k: Readout(d) for k in HEADS})

    def f(self, t, z, ctx):
        h = torch.cat([z, ctx], -1)
        return self.drift(h), self.diff(z)

    def forward(self, eng, cell_id, t_eval):
        assert not hasattr(eng, "fingerprint"), "BAND LEAK"
        ctx = self.moe(self.enc(eng), cell_id)
        z0  = self.cell_prior(cell_id)
        zt  = sdeint_adjoint(self.f, z0, t_eval, ctx,
                             method="reversible_heun",
                             adjoint_params=self.parameters())
        return {k: head(zt) for k, head in self.dec.items()}
```

the hard problems in this band, ranked by how much they worry me

cross-study harmonisation. training data comes from hundreds of labs with different protocols, platforms, batch effects. hierarchical Bayesian with study-level random effects, partial pooling so small studies shrink toward the population. without this the model learns lab identity, which is the most boring possible confound and also the easiest to produce accidentally

distribution shift between in vitro training data and in vivo application. the dynamics are learned mostly from cultured cells and applied to tissue. domain adaptation helps some, importance weighting under covariate shift, adversarial domain-invariant features. none of it fully solves it and i should stop pretending it does

long horizon stability. a latent ODE trained on hour-scale data and rolled out for weeks drifts. currently handled with multiple shooting during training and a slow-manifold regulariser, which helps and does not fix

and the result i keep pinned: in the virtual cell challenge, simple regression baselines did extremely well against large deep models. the field is not as far along as the model sizes suggest. i take that as a standing warning about my own middle band

## inference and uncertainty, the machinery

the model is a simulator, so simulation based inference is the natural fit and i should have gone there sooner than i did

the setup: i have a mechanistic simulator with parameters theta i cannot write a likelihood for, because the likelihood involves integrating out every stochastic event in a hybrid system. classical MCMC is dead on arrival. so, neural posterior estimation

sequential NPE, the atomic variant. train a normalising flow to map summary statistics to posterior over theta, sample from the simulator, retrain, truncate the proposal toward the region that matters, repeat. amortised, so once trained you get a posterior for a new observation in one forward pass rather than a fresh MCMC run

flow architecture is a neural spline flow. rational quadratic splines handle the bounded, heavy tailed, multimodal posteriors that biological parameters actually have. masked autoregressive flows were worse in my hands, mostly on the bounded parameters

summary statistics learned rather than hand chosen, trained jointly with the flow under an infomax-style objective. hand chosen summaries throw away information and you never find out how much

`rattus/unc/posterior.py`

```python
def fit_posterior(sim, x_obs, rounds=6, n=20_000):
    """SBI. the model IS a simulator so amortize over it.
    sequential NPE, truncating the proposal each round."""
    flow = NSF(dim=THETA_DIM, ctx=SUMM_DIM, transforms=12,
               bins=10, tail_bound=6.0)
    prior = BiologicalPrior()          # log-uniform, literature
    prop  = prior
    for r in range(rounds):
        theta = prop.sample((n,))
        x = sim.batch(theta)           # 20k rollouts, GPU
        s = summary_net(x)             # learned summaries
        flow.fit(theta, s, atomic_loss=True)   # APT / SNPE-C
        prop = TruncatedProposal(flow, x_obs, quantile=0.99)
    return flow

def predict(state, u, n=512):
    draws = [rollout(state, u, post.sample(), ens.sample())
             for _ in range(n)]
    mu, var = summarize(draws)
    ale, epi = decompose(draws)        # law of total variance
    lo, hi = cqr_interval(draws, alpha=0.1, cal=CAL_SET)
    if epi / (ale + epi) > 0.7:
        return ABSTAIN(reason="structural disagreement")
    return mu, (lo, hi), {"aleatoric": ale, "epistemic": epi}
```

uncertainty decomposition, which i now think is more important than the interval itself

law of total variance to split aleatoric from epistemic. aleatoric is genuine biological stochasticity, it does not shrink with more data, it is a property of the system. epistemic is my ignorance, it does shrink. conflating them produces a model that looks equally uncertain about a coin flip and about a mechanism it has never seen, and those demand completely different responses

and within epistemic, a further split that matters operationally

| source | shrinks with | what to do |
| --- | --- | --- |
| parameter uncertainty | more measurements | design the experiment |
| structural uncertainty | more mechanism | go read, add the pathway |
| numerical | tighter tolerances | cheap, just do it |
| out of distribution | nothing, by definition | abstain |

model discrepancy is its own term and i had been ignoring it. even with perfect parameters the model structure is wrong somewhere. the Kennedy-O'Hagan framing adds an explicit discrepancy function with its own prior, so the model can say my structure is incomplete here rather than silently biasing parameters to compensate. without it, structural error gets absorbed into parameter estimates and you end up with confidently wrong biology that fits

this is the thing that distinguishes increasing model complexity from increasing model correctness. more parameters cannot fix missing mechanism, they just hide it better

calibration via conformal prediction. split conformal for marginal coverage, conformalised quantile regression for adaptive interval width, Mondrian conditioning so coverage holds per compound class rather than only on average. distribution free, which matters because i cannot verify distributional assumptions about rat biology

ensembles over the learned band. deep ensembles beat MC dropout and beat Laplace in every comparison i ran, at five to ten times the cost. i tried to be clever about this and wasnt

and abstention. the model refuses when epistemic fraction exceeds threshold or when the engagement vector falls outside the training manifold. the refusal is the product and i will keep saying that until someone funds it

### 3:40am sept something

cant sleep, writing it down

the model fits. thats the problem

spent today on parameter recovery. synthetic data, known ground truth, fit, see if i get them back. textbook check. i do not get them back. i get a completely different parameter set that fits equally well. not slightly worse. equally

so i did the hessian. eigenvalue spectrum spans about eight orders of magnitude. a few stiff directions and an enormous flat valley everywhere else. sloppy models. i had read about this and nodded along and did not understand it until i watched it happen to something i built

what it means: the data constrains a handful of parameter combinations and says essentially nothing about individual values. so my mechanistic model, whose whole selling point is that its parameters correspond to real biological quantities, has parameters that are not identified by any data i can get

and its worse for me than for a normal modeler. a black box makes no claim about its internals. i am explicitly claiming my internals are biology. if the internals arent identified then the claim is decorative

sitting with this

three ways out

one, dont care, predictions are what matter. sloppy models often predict fine, thats the actual result in that literature. but it torches traceability and traceability is what i tell regulators. so not alone

two, find the stiff directions and measure those. profile likelihood or sobol to identify which combinations ARE constrained, then design experiments to pin exactly those. good, should do it regardless. and the experiments are cell culture and archived tissue, not animals

three, reparameterise onto the identified manifold. stop pretending i know individual rate constants, carry the identified combinations as the actual model parameters, be explicit that the rest float

three is right isnt it

it means writing down that most individual parameters are undetermined, which will make the whole thing look weaker to anyone skimming. but a model that says these six combinations are pinned, these forty float, here is what that does to the interval, is more useful than one reporting forty numbers it cannot justify. and the second one is what most published mechanistic models do

the thing underneath, before i lose it

fitting is not understanding. a model can reproduce every observation you have and still have the wrong internals and goodness of fit cannot tell you. the only thing that separates them is prediction under intervention. which is exactly what im building. so the project is its own test

either profound or 3am. check tomorrow

checked, next day, still think its right

follow up weeks later. sobol run done on the striatal slice. about 11 combinations carry most of the variance. mostly ratios not absolutes. what matters is kon/koff and the D1:D2 density ratio, not any of the four numbers individually. biologically sensible and i should have guessed. profile likelihood agrees, the non-identified directions are exactly the ones where two parameters trade off multiplicatively

reparameterised. model is smaller now. feels like a loss and is a gain

## compute

less interesting than the science but it constrains the science so

four modeling paradigms have to coexist in one system

mechanistic, explicit equations for known processes. data driven, ML for pattern where equations dont exist. agent based, discrete interacting entities, which is right for immune cells, cell populations, some behavioral processes. physics based, molecular dynamics, biomechanics, fluid movement

no single paradigm covers everything and the interfaces between them are where the bugs live

execution needs distributed computing, GPU acceleration, parallel simulation, asynchronous execution, multirate integration, reproducible checkpoints, distributed state sync, numerical error monitoring, deterministic replay

deterministic replay is harder than it sounds in a distributed setting. floating point reduction order varies with thread scheduling, so the same seed gives different results on different runs. fixed-order reductions where it matters, which costs performance. i went back and forth on whether its worth it and landed on yes, because a prediction i cannot reproduce exactly is a prediction i cannot defend

resource allocation by question. not every molecular interaction needs atom-level treatment during a behavioral simulation. the resolution ladder is explicit: start coarse, refine only where sensitivity analysis says the answer depends on it. adaptive fidelity. i have a scheduler that decides this and its one of the parts im actually proud of

numerics that have to be managed explicitly rather than hoped for

numerical stability, integration error, conservation laws, physical constraints, mass balance, charge balance, energy accounting, probability normalisation, time synchronisation

and when a mechanistic model violates a known physical constraint the simulation flags it rather than continuing. this sounds obvious. it is not what most simulation code does. most code clips to zero and carries on, and then the result is subtly wrong in a way that survives review

rough budget per full striatal slice run at useful resolution, currently

| stage | cost |
| --- | --- |
| molecular band, one compound | minutes, GPU |
| signaling, one cell type | seconds |
| circuit, 1e4 neurons, 10s sim | ~an hour |
| behavioral rollout, physics | real time-ish |
| full UQ, 512 draws | the whole thing x512 |

that last row is the problem. uncertainty quantification multiplies everything by the ensemble size and theres no way around it. currently mitigated with a cheap surrogate for the inner loop and full simulation only for the draws that matter, which is itself an approximation i have not validated properly

## data, what exists and what doesnt

inventory from june. depressing in a useful way

exists and is good

binding affinity, ChEMBL BindingDB PDSP Ki, hundreds of thousands of measurements, and PDSP especially for cross receptor panels which is the only systematic polypharmacology source protein structure, PDB plus predicted structures for basically everything rat transcriptomics under drug perturbation, DrugMatrix and TG-GATEs, thousands of compounds, multiple doses, timepoints, tissues. most underused dataset in the field and its free rat genome, well annotated rat PK, decades of it, scattered across papers cell level perturbation response, human not rat, but the architecture transfers. the atlases growing right now are the reason the learned middle is thinkable at all

exists and is mediocre

rat connectivity, mesoscale tract tracing, nothing at synapse level rat ephys, plenty but every lab different prep, essentially unpoolable without heroic harmonisation behavioral video, huge amounts exist, almost none public. this one makes me angry. every one of those recordings is an animal that already died and the data sits on a hard drive in a basement in vivo rate constants, basically nowhere

does not exist

systematic dose response at multiple levels in the same animal paired molecular to cellular to circuit to behavior under one perturbation rat specific signaling parameters for most pathways systematic off target panels for most compounds negative results, failed compounds, anything unpublished

the missing thing that matters most is paired multiscale perturbation data. i need one intervention measured at every level simultaneously so i can validate propagation rather than endpoints. almost nobody runs experiments that way because each level needs different methods and usually different animals. the literature is thousands of studies each measuring one level and you cannot stitch them because strain sex age dose route timing housing all differ

so the irony i have not resolved: the data that would best validate an animal replacement model is data nobody collected, and collecting it the obvious way needs animals

ways around it

archived tissue. enormous amounts banked from completed studies. molecular and cellular validation, zero new animals. underused because its unglamorous reanalysis of published raw data, mostly a harmonisation problem, boring and tractable and nobody is funded for it in vitro multiscale, organotypic slice and MPS give molecular through circuit in one preparation. not behavior, but eight of the eleven links human data for upper layers, which is odd but for some endpoints more relevant than rat anyway, a point that cuts against using rats in the first place

and the one i keep coming back to. tissue from studies happening regardless. i am not going to stop the global pharmaceutical industry from running rat tox. while it happens those animals could yield ten times the data per animal if anyone coordinated sample collection. thats a reduction available right now, independent of whether my model ever works, and it might be the highest expected value thing in this entire document

perturbation database design, separate from the reference database. every observation carries organism identity, strain, sex, age, biological background, intervention, exposure, measurement, collection time, experimental context, outcome, uncertainty, provenance.

negative findings and failed predictions retained permanently

data quality assessment on everything: measurement reliability, batch effects, missingness, experimental bias, sample size, biological variability, replication, provenance, species and strain compatibility. and the database distinguishes direct measurement from computationally inferred value, which sounds trivial and is violated constantly in public resources

governance: versioned datasets, traceable provenance, reproducible preprocessing, documented licenses, access control, privacy, integrity checks. the privacy part is dormant for rat and becomes the whole problem for human

## the generalization ladder

the defining test is predicting responses to interventions absent from training. so i need to be precise about what absent means, because theres a ladder and conflating the rungs is how people claim generalization they dont have

eight categories, progressively harder

1. unseen compound, familiar target. the easy case. new molecule, known receptor, known pathway. honestly most published "generalization" results are this and it is not very impressive, a similarity model does ok here too
2. unseen compound, multiple familiar targets. polypharmacology. harder because the combination of engagements has to compose correctly, and composition is where additive assumptions break
3. unseen combination of familiar interventions. drug A plus drug B, both individually characterised. tests whether interactions emerge from mechanism or have to be memorised
4. unseen exposure profile. same compound, different dosing schedule. tests whether the temporal dynamics are real. continuous infusion versus pulsatile at matched AUC should give different answers and a static model cannot produce that
5. unseen biological context. same intervention, different tissue or cell state or disease background
6. unseen genetic background. different strain, knockout, variant
7. unseen environmental conditions. stress, temperature, social isolation, circadian phase
8. previously uncharacterised mechanism. the real test. a compound acting through something the model does not represent at all

performance reported separately per category, always. a single aggregate number lets category 1 performance hide category 5 failure, and the aggregate is what everyone reports

category 8 is the one i think about most, because strictly the model should fail it. if the mechanism is not represented it cannot propagate. the correct behaviour is not a good prediction, its a confident statement that something is happening that it cannot account for

so category 8 is really a test of abstention rather than prediction. does the model notice. and it should notice because the engagement vector will be unlike anything in the training manifold, or because predicted engagement cannot explain observed proximal signal

that gives a falsification route i like: give it a compound whose mechanism is known but deliberately withheld from the model, check whether it abstains. if it confidently predicts, the uncertainty machinery is broken, and that is a cheap experiment i can run entirely in silico

have run a toy version. removed sigma-1 from the target set, fed it compounds with substantial sigma-1 activity. it did not abstain. it confidently attributed the effect to the nearest represented target. that was sobering and is the reason the OOD detection moved from the molecule to the engagement vector, and then from the engagement vector to a residual check against observed proximal signal

still not solved. a model cannot easily detect the absence of a thing it has no representation for. this might be fundamental and i dont have a clean answer

## abstention is the product

took me a long time to see this and it is now the most important design decision in the whole thing

what makes a model replace an experiment is not accuracy. its knowing when its accurate

a model right 95% of the time with no idea which 5% is useless for safety. a model right 70% of the time that correctly flags the other 30% as run the experiment is immediately useful, because it removes 70% of the experiments at no added risk

so the output is not a number. its a number, an interval, a decomposition, and a decision about whether to emit at all

```
compound:                    RX-0417   (novel scaffold, 0.31 max tanimoto to train)
prediction:                  locomotion -34%    [95% CI    -51 .. -12]
confidence:                  MODERATE


basis
  D2 occupancy 0.78                            high       homologous scaffold in train
    Gi coupling                                high       operational model, tight post
    arrestin bias 0.22                         MEDIUM     sparse bias data this class
    MSN excitability delta                     MEDIUM     cell type undersampled
    circuit -> locomotion                      LOW        degenerate readout, known


variance
  aleatoric                     0.19           irreducible
  epistemic                     0.61           of which structural 0.44
    OOD flag                    engagement within manifold (ok)

limiting link: circuit -> behavior
recommend:                   USE for relative ranking vs reference set
                             DO NOT use for absolute effect size
                             highest value experiment: ex vivo MSN excitability
                             est. interval reduction 58%
```

that last block is what i care about. the model says what measurement would most reduce its own uncertainty. then it stops being a replacement for experiments and becomes an allocator of experiments, which is more useful and much easier to get adopted

thats expected information gain, standard bayesian experimental design, applied to its own posterior. compute EIG over candidate measurements, rank them. the machinery is not novel, pointing it at the models own ignorance is the useful part

the uncomfortable part: calibration can only be verified on data i have. for genuinely novel compounds, the regime i built this for, i cannot verify calibration without doing the experiment. irreducible circularity. the only way out is slow accumulated prospective track record over years and i should stop looking for a shortcut

and the incentive problem, flagging it because it will be real. nobody wants to fund a model that says i dont know. funding pressure, publication pressure, and my own ego all push toward confident point estimates. the calibration machinery has to be structural, load bearing in the architecture, rather than a reporting choice i can quietly drop when someone asks for a cleaner number

## how not to fool myself

the actual risk is not that this fails. its that it appears to work, i believe it, and its wrong. i would not find out for years and meanwhile i would be telling people to skip animal studies on the basis of a broken model. thats a worse outcome than doing nothing

rules, written now while nothings at stake, because the pressure to relax them arrives later when i need a result

a model is not validated by reproducing the data used to build it. retrospective fit counts for zero. i will be able to fit the entire historical dopaminergic literature and that is evidence of flexibility, nothing else

validation at twelve levels, each independently

| level | what gets checked |
| --- | --- |
| molecular | binding, kinetics, conformational behavior |
| cellular | perturbation response, cellular function |
| tissue | spatial and multicellular behavior |
| organ | physiological function |
| systemic | integrated whole body response |
| neural | electrophysiology, circuit response |
| sensory | stimulus response relationships |
| motor | biomechanical accuracy |
| cognitive | learning and memory performance |
| behavioral | naturalistic and measured behavior |
| pharmacological | drug response and toxicity |
| longitudinal | adaptation over time |

correct behavior with wrong signaling is a failure even though it looks like success. especially then

prospective blinded protocol, the full version

receive the intervention spec. establish initial biological state. generate predictions. produce uncertainty estimates. record the simulation configuration, model version, data version, seed, solver, tolerances, config hash. lock it. submit for independent evaluation. only then look

the evaluation set stays inaccessible during development, which means someone else holds it. i cannot be the keeper of my own holdout and i do not trust myself to be

the peek is the whole failure mode. one look and i tune unconsciously. this is not hypothetical, its what happened with the fake theta oscillation

comparators, and they have to be strong

conventional statistical models. established pharmacological models. existing ML approaches. strong simple baselines. relevant animal derived predictions. human derived alternatives where they exist

and predicting the training mean. an embarrassing number of published models lose to that and never report it. if the mechanistic stack doesnt beat the mean predictor on an endpoint i say so out loud

behavioral validation needs multiple independent measurements and must not assume one observable maps to one construct. reduced movement is sedation or motor impairment or weakness or motivation or nausea or hypothermia. the model has to distinguish plausible causal explanations rather than classify movement as reduced

metrics by endpoint: MAE, RMSE, correlation, calibration error, log loss, Brier, AUROC, precision-recall, time to event error, distributional distance, trajectory similarity, interval coverage. behavioral distributions compared, not just means

reproducibility, every published prediction regenerable from model version, dataset version, parameter config, seed, solver, environment, intervention spec. containerised, hashed, archived

and the bar question i still havent settled. better than the animal study's own test-retest reliability is the defensible bar, and its lower than people assume because inter-lab reproducibility of rodent behavioral assays is genuinely poor. it feels like cheating. it isnt cheating. it still feels like it

### 2:12am, oct 30

woke up with this fully formed and i need to write it before it evaporates

what if the degeneracy in behavioral readouts isnt a bug in the measurement. what if its a bug in the ONTOLOGY

like. we keep saying reduced locomotion could be sedation or motor impairment or weakness or reduced motivation. and we treat that as a measurement problem, the readout is too coarse, we need better instruments

but what if sedation and motor impairment and reduced motivation arent actually separate things that the measurement is failing to distinguish. what if theyre human- invented categories that we projected onto the rat and the rat does not have them

the rat has a state. one state. a point in some high dimensional physiological space. we look at it and say thats sedation because it resembles a thing we have a word for. but the word came from human phenomenology and we handed it to an animal that cannot report

so the model shouldnt predict "is it sedated". it should predict the STATE and refuse to name it

and then the names become a separate question. you can ask which regions of state space the word sedation has historically been applied to, and you will find the word covers a weird non-convex blob that doesnt correspond to anything mechanical

this actually changes the architecture. the behavioral output shouldnt be a classifier over named behaviors. it should be a density over a learned state manifold, and the named categories are a post hoc labeling that i can provide for compatibility with existing literature but should not optimize against

which means the training objective changes. stop fitting labeled behavioral outcomes. fit the full pose and physiology trajectory distribution and let labels be a readout layer you can attach or not

and it means the degeneracy stops being a problem because im no longer asking the model to invert a many-to-one map. im asking it to predict the thing that was actually measured

ok also thinking about this more. the syllable approach is halfway there already. unsupervised segmentation of pose into recurring motifs, no human labels. the syllables ARE the ontology, discovered rather than imposed

but even syllables are a discretisation. maybe go further. switching linear dynamical system where the discrete states are inferred, so you get both the continuous trajectory and a soft assignment to regimes, and the regimes are whatever the data says rather than what a pharmacologist in 1962 named

TODO tomorrow: check whether anyone has done SLDS on drug-perturbed rodent pose data. feels like someone must have

(next day: people have done SLDS on pose. nobody has done it with a mechanistic generative model upstream of it. thats the gap. the behavioral model exists, the mechanism-to-behavior link does not)

(also rereading this in the morning it is less revolutionary than it felt at 2am but the core point survives: stop predicting labels, predict distributions, attach labels later. im changing the output layer)

## the roadmap

eight phases. writing them with exit criteria because a phase without an exit criterion is a vibe

phase 0, scientific infrastructure. biological ontology, rat reference genome, anatomical reference frame, unified data schema, evidence provenance system, model registry, simulation interface, validation framework

exit: independently developed subsystems can talk through a standardised biological representation

this is the boring phase, its most of the work, and skipping it is the single most common way these projects die. i have watched people start at phase II and spend three years rebuilding phase 0 badly

phase I, molecular and cellular foundations. molecular interaction engine, protein and receptor representation, intracellular signaling models, gene regulatory models, cellular state representation, cell specific perturbation prediction

exit: successful prospective prediction of defined cellular responses to unseen perturbations

this is where i actually am. roughly. partially

phase II, neural and tissue. neuronal populations, synaptic models, glial interactions, circuit architecture, spatial tissue models, neural activity prediction

exit: independently validated predictions across multiple neural and tissue measurements

phase III, whole body physiology. cardiovascular, hepatic, renal, respiratory, endocrine, immune, GI, integrated circulation and metabolism, whole body PK

exit: validated prediction of specified integrated physiological responses

phase IV, functional virtual rat. biomechanical body, sensory architecture, motor control, cognitive architecture, learning and memory, interactive environment, physiological feedback, naturalistic behavior generation

exit: reproducible independently validated responses across multiple behavioral domains

phase V, the unseen intervention challenge. novel compound evaluation, novel genetic perturbation, combination interventions, longitudinal prediction, population variation, uncertainty calibration, independent prospective benchmark

exit: performance exceeding existing approaches on explicitly defined domains, with evidence sufficient to justify replacing particular animal experiments

phase VI, replacement and regulatory validation. validated application specific models, standardised reporting, regulatory evidence packages, external lab validation, reproducible protocols, documented applicable domains and limitations

exit: acceptance of specific applications as scientifically adequate alternatives for defined purposes

phase VII, human translation. human genome and regulatory models, human cellular reference, human tissue and organ models, human neural architecture, human physiological integration, population variability, patient specific twins

exit: validated human relevant predictions across defined clinical and pharmacological domains

the thing i keep having to correct in my own planning: these are not sequential in the way the numbering implies. phase VII work can start now, because the human in vitro data exists now and the architecture ports before the parameters do. i had this as a strict waterfall for months and it was wrong

realistic timeline if everything goes well, which it wont

| phase | optimistic | realistic |
| --- | --- | --- |
| 0 | 1 yr | 2 |
| I | 2 | 3-4 |
| II | 3 | 5+ |
| III | 3 | ? |
| IV | 4 | ?? |
| V | ? |  |
| VI | regulatory time, not science time |  |
| VII |  |  |

thats about fifteen years to phase V if nothing goes wrong. things go wrong

## the dopamine slice, in detail

one vertical slice, all eleven links, as narrow as i can make it

why dopaminergic striatal pharmacology

best characterised system in neuropharmacology. D1 D2 D3 structures solved, kinetics for hundreds of ligands, signaling worked out, striatal microcircuit mapped better than almost anything else in rat brain

clean architectural split that does half my work. D1-MSNs direct pathway, Gs/olf, cAMP up. D2-MSNs indirect pathway, Gi/o, cAMP down. two populations, opposing effects on the same output, interdigitated in the same tissue. if the model cannot hold those apart it has no chance at anything else

motor readout, the least degenerate behavior available. locomotion, catalepsy, stereotypy, rotarod. still not clean, reduced movement still means eight things, but better than anything claiming to measure affect

enormous existing validation set. haloperidol, aripiprazole, amphetamine, cocaine, SKF- 81297, quinpirole, raclopride, sulpiride, all characterised to death in rats in archived literature

and the one that actually decided it, aripiprazole. partial agonist, functionally selective, clinical profile not predictable from affinity. exactly the case where similarity lookup fails and mechanism should succeed. so its my discriminating test. if rattus cannot get aripiprazole right from mechanism the thesis is wrong and i want to know that in year two not year six

molecular scope: D1, D2 (D2S and D2L separately, the ratio matters and differs by region and the two isoforms have different pre versus postsynaptic roles), D3. Gs/olf and Gi/o coupling. adenylyl cyclase, cAMP, PKA, DARPP-32, phosphatase cascade. arrestin recruitment, internalization, desensitization, recycling. DAT, VMAT2, MAO-A/B, COMT. relevant calcium and ion channel processes

DARPP-32 specifically because its the integrator. phosphorylation at Thr34 versus Thr75 flips it between PKA amplifier and PKA inhibitor, so it is a sign-switching node and it is where D1 and D2 signaling converge. get DARPP-32 wrong and both pathways are wrong

cellular scope: D1-MSN, D2-MSN, dopaminergic neuron, fast spiking GABAergic interneuron, cholinergic interneuron, astrocyte

astrocytes are in there deliberately. theyre usually left out of circuit models and they handle glutamate uptake and potassium buffering and they express receptors. a drug that hits astrocytic receptors changes circuit function without touching a neuron directly

circuit scope: striatum, substantia nigra pars compacta, VTA, GPe, GPi/SNr, STN, motor cortex, thalamus. the classic direct/indirect/hyperdirect architecture

behavioral scope: locomotion, motor coordination, repetitive behavior, behavioral flexibility, reward learning, motivation

the question the slice has to answer

can a mechanistically constrained model of dopaminergic biology predict downstream effects of unseen dopaminergic compounds more reliably than a model trained only on historical behavioral outcomes

tested at every intermediate level. a correct behavioral prediction with wrong cAMP is a coincidence and counts as a failure. that rule is going to be unpleasant to live by and i am writing it down now so future me cannot quietly drop it

first implementation also has to document the limits of each cellular representation and mark every parameter as measured, inferred or assumed. the slice is as much a methodology test as a science test

## what ships first

nine packages in the first development block

| name | what it is |
| --- | --- |
| rattus-core | shared biological state + simulation architecture |
| rattus-data | versioned biological reference database |
| rattus-mol | molecular interaction engine |
| rattus-cell | initial virtual dopaminergic cells |
| rattus-neuro | initial dopaminergic circuit model |
| rattus-pkpd | PK/PD integration |
| rattus-valid | independent validation framework |
| rattus-interface | researcher facing intervention interface |
| rattus-docs | technical and scientific documentation |

the interface package matters more than it looks. if a pharmacologist cannot specify an intervention without reading the source code, nobody uses it, and a tool nobody uses replaces zero animals. the whole project fails on UX before it fails on science. i keep having to remind myself of this because interface work feels like not-research

minimum viable prototype is not a complete rat. its an integrated system that accepts a defined pharmacological intervention and predicts its consequences through a limited but experimentally supported pathway

the MVP has to demonstrate, end to end

1. compound representation
2. molecular target interaction
3. receptor signaling
4. cellular response
5. neural circuit response
6. a measurable physiological or motor output
7. uncertainty estimation
8. independent validation

thats the first complete causal pathway through rattus. thin but unbroken

and the discipline i want to hold: every one of those eight has to be real before any of them gets widened. the temptation is to make the molecular band excellent because its the part i enjoy and the part i am good at, and leave the circuit link as a stub for another year. that produces an impressive demo with no causal chain, which is exactly the failure mode i criticised blue brain for

depth before breadth, but only after the chain closes

## governance, and the rules i expect to break

the project has to maintain explicit distinction between observed biological facts, established mechanisms, supported hypotheses, computational approximations, unvalidated assumptions, and unknown processes

thats what the provenance system is for. its not documentation, its the mechanism that makes the distinction checkable rather than rhetorical

no false completeness. never describe an organ, cell, neural system or organism as reconstructed just because a computational representation exists. completeness is domain specific and has to be justified by evidence

this one is going to be hard because the incentive to overclaim is enormous and the language for overclaiming is already standard in the field. "we built a virtual liver" reads better than "we built a model of hepatic clearance for a defined compound class validated against archived data". the second one is true

negative results preserved, all of them, classified by failure type: incorrect parameters, incomplete data, missing mechanisms, structural model errors, numerical instability, poor generalization, measurement error in the reference. the failure taxonomy tells me where to work next and its more valuable than the successes

independent review at every major milestone. model developers must not be the sole evaluators of their own claims. i cannot hold my own holdout set and i cannot grade my own validation

the ethical objective, written down plainly so it does not quietly erode

replacement is the priority, not refinement. where existing non-animal evidence suffices, animal experimentation should not be the default. where validation needs new biological evidence, prioritise human relevant methods, archived datasets, organoids, tissue systems, microphysiological platforms

the objective is scientific replacement, not computational imitation. a beautiful simulation that nobody trusts enough to skip an experiment has replaced nothing

and the rules i expect to break, listing them now so i can catch myself

i will be tempted to peek at holdout data. i will be tempted to report aggregate generalization numbers instead of per-category. i will be tempted to describe a subsystem as working when it runs without crashing. i will be tempted to drop the abstention output when someone asks for a cleaner result. i will be tempted to call the molecular band finished because its the part i like

writing them down does not stop them. it makes them embarrassing, which is close enough

## what kills this

ordered by probability of being the actual thing that ends it, not by drama

1, the circuit to behavior link never gets good enough. most likely by a wide margin. i can see the path to molecular, cellular, even circuit. behavior needs connectivity and plasticity rules nobody has for rat. if this is the wall the project still yields a validated molecular-to- cellular predictor, which has real value for tox and target engagement, but it does not replace behavioral studies and that was the point

2, identifiability wins. the sloppiness is worse than i think, the manifold too flat, predictions too sensitive to unconstrained directions, intervals too wide to be useful. partly mitigated by reparameterisation, maybe not enough

3, paired multiscale data never appears. i can only validate endpoints, and endpoint validation cannot distinguish correct mechanism from lucky mechanism. stuck with a model i cannot defend

4, it works and nobody adopts it. genuinely plausible and i underweight it because its not a science problem. regulators are conservative, pharma is liability driven, and the model said it was safe is a worse legal position than we ran the standard study even when the model is better. CiPA took decades and that was the easy case

5, i fool myself. the one i control and therefore the one that deserves the most worry. appears to work, i believe it, its wrong. the validation discipline is exactly what erodes under funding pressure

6, compute. if the defensible version needs molecular dynamics inside a whole body simulation it may be unaffordable. probably solvable with surrogates and adaptive fidelity, but i might be wrong about the cost

7, scope collapse. i narrow the context of use to get a validated result, then narrow again, and end up replacing one assay for one compound class. which is how CiPA worked, so its not failure exactly. but its not what i set out to do and i should decide in advance whether id accept it

(i would accept it. writing that down so i remember)

8, somebody does it better and first. would be delighted. this is not a thing i need to be the one to do

what i notice about this list is that none of them are the idea is wrong. theyre all execution, data, or adoption. either the idea is sound or im too close to see the flaw

TODO give this to someone who thinks the whole thing is nonsense and ask them to add to the list. i have not done this and im avoiding it and i know why

## HUMANS

ok. everything above this line is the warm up and i want to be clear about that

the rat was never the point. the rat is a model of a human. thats the only reason theres a rat in the building. so if im constructing a computational organism, the question that has been sitting underneath this entire document is why build the rat at all

short version: validation. i can check rat predictions against existing rat data. i cannot ethically generate human data the same way. so rat is the proving ground for the architecture, not the destination

longer version is the rest of this document and its the part where i get genuinely stuck in several places, so im writing those down too

before anything else, the stupid version of this idea, which i believed for about three weeks in july and want on record

the stupid version: build rattus, swap in the human genome, swap the anatomical atlas, rescale the masses by body weight, press play. human digital twin

this does not work. the reasons it does not work turned out to be the most interesting thing i have found all year, because every reason it fails is simultaneously a reason that the rat itself fails as a model of a human

and that symmetry is the actual argument. i keep coming back to it. the species gap is not an argument against computational human models. it is an argument against animal models. a rat IS a model of a human and it is a bad one, with an error that is unquantified, unquantifiable, and unimprovable

that last word is the one that matters. when my model is wrong i can find out which link broke and fix it. when a rat is wrong as a model of a human there is no fix. the rat cannot be made more human. you get the translation error you get, forever, and you do not even get to know its magnitude

so the bar for a computational human model is not perfection. its better than a rat. and the rat is a genuinely poor predictor. roughly 90% of drugs entering clinical trials fail, and a large share of those fail on efficacy or toxicity the animal studies did not predict

sit with that for a second. we kill millions of animals a year for a predictive instrument that is wrong most of the time, and we keep doing it largely because its what regulators are used to and nobody had anything better

the question was never can a simulation be as good as reality. its can a simulation be better than a bad proxy. completely different problem and a much lower bar

what follows is about twenty pages of me trying to clear that bar, failing in specific places, and finding three or four things along the way that i am fairly sure nobody has done

## why the genome swap fails

going through this properly because each failure mode points at a different piece of the eventual architecture

orthologs are not equivalents. a rat gene and its human ortholog can have different expression patterns, different splice variants, different binding partners, different regulation. sequence homology tells you they share ancestry, not that they do the same job

D2 is a clean example because its my own target. D2S and D2L differ by 29 amino acids in the third intracellular loop. the ratio differs between species and between brain regions, and the isoforms have different presynaptic versus postsynaptic distribution. so "the D2 receptor" is not one object you can port. its a tissue specific mixture and the mixing ratio is a parameter you have to re-measure

drug metabolism is where this gets actively dangerous. rat CYP2D1 versus human CYP2D6 are not interchangeable enzymes. humans have CYP2D6 copy number and functional polymorphism producing poor, intermediate, extensive and ultrarapid metabolizers, a 10 to 100 fold swing in exposure at the same dose. rats do not have that polymorphism structure at all

which means a rat PK model, however perfect, structurally cannot tell you about the subpopulation that will actually be harmed. the information is not degraded, its absent

same story for CYP2C19, CYP3A5, UGT1A1, TPMT, DPYD. the pharmacogenomic variance in humans is a dimension the rat does not have

allometry breaks where you need it. clearance scaling as mass^0.75 works until it doesnt. transporter expression differs. plasma protein binding differs, and free fraction is what drives effect. P-glycoprotein substrate specificity at the blood brain barrier differs, so CNS penetration does not scale with body size at all. and brain architecture is not a scaled rat brain, prefrontal expansion is structural not dimensional

immune system barely translates. TGN1412 is the case that should be taught in every pharmacology course. anti-CD28 superagonist monoclonal. preclinical in cynomolgus macaques showed essentially nothing, clean at doses far above what went into humans. six healthy volunteers in 2006, catastrophic cytokine storm within hours, multi organ failure. the explanation found afterward is that CD28 expression on effector memory T cells differs between the species, so the cells that drove the human response were not present in the macaque in the same way

a primate. closer to us than a rat by an enormous margin. and it missed completely, in the most catastrophic possible direction

microbiome, which i keep forgetting and shouldnt. rat gut microbiome differs substantially from human and it performs drug metabolism. azo reduction, glucuronide hydrolysis, and a long list of transformations that change exposure. so even the gut chemistry is species specific

body temperature, heart rate, metabolic rate, lifespan, all different by factors that change which processes dominate. a rat heart at 350bpm is not a small human heart. the repolarization currents that dominate at that rate are different ones, which is exactly why rat is a poor cardiac safety model and why CiPA used human ionic models instead

and that last point is the template for the whole human program. CiPA did not scale a rat heart. it built a human ventricular myocyte model from human channel data and validated against human outcomes. thats the move

## the inference ladder

hardest thinking in this document and its not finished. question: building the human version, what transfers, what has to be re-measured, what can never be inferred

worked it as rungs because i kept conflating different epistemic situations

rung 1, physics and chemistry. transfers completely. binding thermodynamics, diffusion, enzyme kinetics formalism, membrane biophysics, mass action. a kon is a kon. this layer is species free and its a larger fraction of the machinery than you'd guess. the entire molecular band ports with zero modification

rung 2, architecture. transfers, parameters dont. signaling topology is largely conserved. GPCR to G protein to adenylyl cyclase to cAMP to PKA is the same cascade in both. what changes is expression level, isoform ratio, rate constants, compartmentalisation. so the graph transfers and every weight on it has to be re-estimated

this is the sweet spot and its why the rat work isnt wasted. im learning structure, and structure is the expensive part. parameters are comparatively cheap once you know what to measure

rung 3, cell types. partially transfers, failures are sneaky. human and rat hepatocytes do the same job with different enzyme complements. cortical neurons broadly similar with real differences in channel expression and dendritic morphology. and some human cell types have no rodent equivalent at all, outer radial glia being the famous one, directly relevant to cortical development. so transfer the cell model, re-fit the cell identity, and keep an explicit list of cell types where the mapping fails

that list is a deliverable in its own right. a catalogue of where rodent neuroscience silently does not apply

rung 4, organ physiology. transfers as structure, badly as function. nephron works the same way, concentrating capacity differs. hearts beat, and as above the rate difference changes which currents dominate, so the equations change weight not just value. liver zonation same pattern, different enzyme gradients

rung 5, neural circuits. transfers poorly and i should stop pretending otherwise. basal ganglia loop architecture is conserved, which is genuinely useful and part of why dopamine was a good first choice. cortical microcircuit motifs similar-ish. but prefrontal organisation, cortical area count, white matter topology, and the entire hierarchy of association cortex differ structurally. for anything cognitive the rat is a weak prior and should be treated as one

rung 6, behavior and cognition. does not transfer. at all. and i want to be blunt because the field is not

a rat doing spatial navigation and a human doing a working memory task are not the same process in two species. the forced swim test is not a model of human depression, its a measure of immobility in a rat in water. the construct mapping was an assumption and it was mostly wrong. forty years of antidepressant screening on a readout whose interpretation is still argued about

so for the human version behavior is not ported. its modeled directly from human data, of which there is a great deal

rung 7, things i do not think can be inferred from any model. subjective experience, and whether the simulated system has any. im flagging that this question exists and then leaving it alone, because im not qualified and because it does not affect any prediction im making

but i notice it is strange that im building something whose rat version is explicitly meant to have no experience, when the entire moral motivation is that real rats do. not resolving it. it keeps occurring to me at inconvenient times

what the ladder buys me practically is an ordered transfer strategy. rungs 1-2 port directly. 3-4 port as structure with re-estimated parameters from human in vitro data that exists. 5 ports as a weak prior to be overwritten by human imaging. 6 is built fresh. 7 is not my problem

and the thing i did not expect from writing it out: the human version needs LESS from the rat version than i assumed. it needs the architecture, the tooling, the constraint machinery, the uncertainty framework, the validation discipline. not the parameters

so the rat phase is less about rats than i thought. its a methodology shakedown that happens to use a species where i can check my work

## the engagement invariance hypothesis

this is the first thing in the human program that i think is actually new, and its a direct consequence of the sandwich architecture rather than something i went looking for

the setup. the molecular band outputs an engagement vector. target identity, occupancy, efficacy, bias, kinetics. the learned band consumes it and knows nothing else

now ask: what in that pipeline is species specific

the molecular band is physics. a ligand binding a receptor is the same physical process in both species. the receptor sequences differ, so the affinities differ, but the MACHINERY computing affinity from structure does not differ. feed it the human D2 sequence and structure instead of the rat one and it works, with no retraining, because it was never trained on species, it was trained on physics

the learned band is where biology lives and biology is species specific

so the hypothesis

the engagement vector is a species invariant interface. the map from (molecule, target structure) to engagement transfers across species with only the structural inputs changed. the map from engagement to biological response does not transfer and must be re-learned

if thats right, and i think it mostly is, then the architectural consequence is clean. the human system is the same molecular band, pointed at human protein structures, feeding a different learned band trained on human data

which means roughly 40% of the computational machinery ports with zero retraining and the remaining 60% needs human data that largely exists

and it makes the species gap LOCALISABLE, which i think is the actually useful contribution. right now, when a rat study fails to predict a human outcome, nobody can say where the failure occurred. with this decomposition you can ask: did the engagement differ, or did the response to the same engagement differ

those have completely different implications. engagement differing means a PK or affinity problem, which is fixable by measurement. response differing at matched engagement means the downstream biology genuinely differs, which is a hard biological fact you have to model, not an error

so heres the testable version, and its cheap

take compounds with known rat and human data. compute predicted engagement vectors in both species using species specific structures. where rat and human in vivo outcomes agree, engagement vectors should be similar. where outcomes diverge, either engagement diverges (explainable) or it doesnt (meaning downstream biology differs)

that partitions the entire history of failed animal-to-human translation into two bins. and the size of those bins is a number nobody has, and it matters enormously, because it tells you what fraction of translation failure is in principle fixable by better pharmacology versus irreducibly biological

if most failures are engagement failures, the field should invest in better PK and affinity prediction and animal models are salvageable. if most are downstream failures, animal models are structurally doomed for those endpoints and no amount of refinement helps

i genuinely do not know which and i think its the most informative experiment in the human program. its also entirely computational and uses only existing data

TODO scope this properly. candidate set is withdrawn drugs plus compounds that failed in phase I for tox not predicted preclinically. maybe 60-100 compounds with enough data on both sides. that is a paper on its own and i should write it before any of the rest

(have started pulling the list. its harder than expected because failure data is not published systematically, which is itself the data problem from earlier wearing a different hat)

## the rosetta set

second new thing, and this one needs wet lab work but no animals

problem: to learn how rat biology maps to human biology i need paired observations. same perturbation, both species, matched conditions. that data does not exist in any systematic form

what exists is two separate literatures that cannot be aligned because everything differs at once. different compounds, doses, timepoints, readouts, labs

proposal: deliberately construct the paired set. matched in vitro panels

take N compounds spanning mechanism space. run them on matched rat and human primary or iPSC derived cells, same cell types, same doses, same timepoints, same readouts, same lab. transcriptomics, phosphoproteomics, calcium imaging, MEA for the electrically active ones

that gives a rosetta stone. not a translation of one text, a parallel corpus

cell types to start, ranked by how much they would buy

| cell type | why | rat source | human source |
| --- | --- | --- | --- |
| hepatocyte | tox, metabolism, the main killer | primary, archived | primary + iPSC |
| cardiomyocyte | the other main killer | primary | iPSC-CM, mature-ish |
| cortical neuron | CNS everything | primary | iPSC-derived |
| MSN / striatal | my slice | primary | iPSC protocols exist, hard |
| renal proximal tubule | clearance, tox | primary | iPSC + primary |
| astrocyte | neuroinflammation | primary | iPSC |
| microglia | ... | hard | iPSC, immature |
| immune PBMC-like | TGN1412 class | ? | primary, easy |

rat primary cells come from animals, which is the obvious objection and i want to meet it directly. two answers. one, those cells come from tissue that can be banked from animals already being used for other purposes, so its marginal zero. two, this is a one time cost that buys a translation operator reusable forever, against an ongoing cost of whole animal studies run per compound indefinitely. the integral strongly favours doing it once

scale estimate: 200-500 compounds, 8 cell types, 2 species, 4 doses, 3 timepoints. order 10^5 conditions with multiplexed readouts. thats a large but entirely conventional screening campaign. the technology is boring. nobody has done it because nobody had a reason to, because nobody was trying to learn the translation operator explicitly

the compound selection is the interesting design problem and i have been thinking about it more than the rest. you do not want 400 kinase inhibitors. you want maximal coverage of ENGAGEMENT space, which is a different thing from chemical space. select to maximise diversity of the engagement vector distribution, including deliberately including compounds with known species-divergent outcomes as positive controls for divergence

thats a D-optimal design problem over the engagement manifold and i can compute it with the molecular band i already have, which is a satisfying loop

also: the existing perturbation atlases are already half of the human side of this. the human arm might be substantially free if the cell types overlap, which means the real cost is the rat arm and the matching

## learning the translation operator

so given a rosetta set, how do you actually learn the map. this is where i spent most of october and where i got properly stuck twice

naive version: train a regression from rat response to human response, compound by compound. fails immediately. you never observe the same CELL in both species, only the same compound, so the pairing is at the compound level while the data is at the cell level. and cell populations differ in composition between the two preparations, so even matched cell types are distributions, not points

so its an unpaired distribution alignment problem with weak supervision from shared compounds. which is optimal transport

first attempt: entropic OT between rat and human response manifolds, with the cost masked so only shared compounds can be coupled. the mask IS the supervision. then barycentric projection to get targets and fit a map

`rattus/human/align.py`

```python
def learn_translation_operator(rat_resp, hum_resp, compounds):
    """THE IDEA. matched compounds assayed on rat + human
    primary cells. we never see the same cell twice so the
    pairing is at the COMPOUND level, not the cell level.
    entropic OT to align the two response manifolds, then
    learn the map on the coupling."""
    Zr = encoder_rat(rat_resp)          # (n_r, d)
    Zh = encoder_hum(hum_resp)          # (n_h, d)

    # cost = response distance, masked so only shared
    # compounds can be coupled. this mask IS the supervision.
    C = torch.cdist(Zr, Zh) ** 2
    C = C.masked_fill(~shared_compound_mask(compounds), 1e6)

    P = sinkhorn(C, eps=0.05, n_iter=500)     # (n_r, n_h)

    # barycentric projection -> targets for the operator
    Yr = (P / P.sum(1, keepdim=True)) @ Zh
    T  = ResidualMLP(d, d)
    opt = torch.optim.AdamW(T.parameters(), lr=3e-4)
    for _ in range(STEPS):
        loss = F.mse_loss(T(Zr), Yr) \
             + LAMBDA * gw_discrepancy(Zr, T(Zr), Zh)
        loss.backward(); opt.step(); opt.zero_grad()
    return T     # rat latent -> human latent
```

this sort of worked and then i got stuck on something i should have anticipated

stuck point one. OT finds a coupling that minimises transport cost, but minimum transport cost is not the same as biologically correct correspondence. the map it found was smooth and plausible and i had no way to check it was right. with 50 shared compounds in a 2000- dimensional response space the coupling is massively underdetermined

what got me out, partially: add structure. the response spaces arent arbitrary point clouds, they have known correspondences at the gene level via orthology. so instead of aligning raw response vectors, align in a space where the coordinates already mean the same thing. Gromov-Wasserstein on the pathway-level representation, where pathways are defined by orthologous gene sets

this constrains it a lot and it also makes the failures interpretable, because when GW cost stays high for a pathway thats a pathway where the species genuinely diverge, and that is a FINDING not a failure

stuck point two, worse, not resolved. the operator i learn is conditional on cell type and condition, and i have no guarantee it extrapolates to conditions outside the rosetta set. so i can translate rat to human for the compounds and contexts i measured, which is exactly the thing i could already do empirically. the whole point was to translate for compounds i have NOT measured

thinking about this for weeks. current best idea, not validated

dont learn a map between responses. learn a map between MECHANISMS. the engagement vector is species invariant by the hypothesis above. so condition the learned band on a species embedding and train both species jointly, sharing everything except a low rank species-specific adaptation

concretely: one backbone, LoRA-style low rank adapters per species on each layer. trained on both corpora simultaneously. the shared backbone carries conserved biology, the adapters carry divergence. and the RANK of the adapter needed to fit a given pathway is then a direct measure of how much that pathway diverges between species

that last part is the bit i like. the architecture produces a quantitative measure of species divergence per pathway as a byproduct of fitting. low rank needed means conserved, high rank means divergent. thats a number nobody has and it falls out for free

and it fixes the extrapolation problem in principle, because im no longer learning a response-to-response map, im learning two instantiations of one mechanism model. novel compounds route through the mechanism, not through the translation

still unvalidated. the thing that would falsify it is if the adapters have to be nearly full rank everywhere, which would mean the species share almost no mechanism, which would be a depressing but very publishable result

TODO: run this on the perturbation data i already have as a pilot. can test the rank- divergence idea on human-only data first by treating cell lines as the "species" axis. if adapter rank correlates with known lineage distance, the method works

## the human data situation is better than the rat one

which still surprises me when i say it out loud

the thing that makes the human version tractable is that the data exists, is already consented, and nobody had to be harmed to collect it

genomics at population scale. UK Biobank, half a million people with genotype plus imaging plus linked health records plus biochemistry plus now whole exome. All of Us. FinnGen. biobank Japan. this is a scale of individual variation data that does not exist for any model organism and never will, because model organisms are inbred specifically to remove variation

molecular and cellular. GTEx for tissue specific expression across dozens of tissues with matched genotype. Human Cell Atlas. human iPSC derived neurons, cardiomyocytes, hepatocytes, which you can perturb without limit. organoids including cerebral organoids. microphysiological systems, organ on chip, multi-organ chips with recirculating media. and the human perturbation atlases, which are the fastest moving thing in the field right now

imaging. structural and functional MRI at enormous scale with task, rest, diffusion. and PET for receptor occupancy in living humans, which is remarkable when you consider what getting that in a rat requires

PET deserves emphasis. i can get D2 receptor occupancy as a function of plasma concentration in a living human, non invasively, and that data exists for most marketed antipsychotics. thats a direct measurement of the engagement vector in vivo in the species i care about. there is no rat equivalent that doesnt involve killing the rat

clinical. decades of trial data, much now shareable through managed access. EHR at population scale. dense ICU timeseries. pharmacovigilance. and dose ranging data in humans from escalation studies that already happened

wearables, which i initially dismissed and now think are significant. continuous heart rate, activity, sleep, increasingly continuous glucose. thats dense longitudinal physiology at population scale, collected passively, and its the closest thing to rodent telemetry except its in the right species and nobody was implanted

so the comparison, which i find genuinely striking

|  | rat | human |
| --- | --- | --- |
| individual variation | poor, inbred by design | enormous |
| controlled invasive perturbation | yes, and thats the problem | no |
| natural genetic perturbation | limited | population scale |
| cellular perturbation data | moderate | large, growing fast |
| receptor occupancy in vivo | invasive, terminal | PET, non invasive |
| dense longitudinal physiology | implanted telemetry | wearables, free |
| tissue access | terminal | biopsy, surgical waste, post mortem |
| behavior | video, degenerate readouts | self report + task + digital phenotyping |
| relevance to humans | unknown and unimprovable | by construction |

read the last row again. the rat column has one unfixable problem and the human column does not have it

what humans genuinely lack is controlled invasive perturbation. you cannot give 500 people a novel compound and take brain tissue. thats precisely the gap the model fills, by combining mechanism from human in vitro systems with observational human population data

so the human architecture is not a port of the rat one and i should stop describing it that way

mechanism from human in vitro, iPSC, organoid, MPS, which is unlimited and ethical parameters from human population data validation against observational human data and existing trial archives individual variation from biobank genetics, which is the thing animals structurally cannot provide

this is where i stopped thinking of the human version as the risky speculative extension and started thinking of it as the part with better data. not where i expected to end up when i started

## meiosis already ran the knockouts

third new-ish thing. im less sure this is novel, parts of it are clearly standard in statistical genetics, but the systematic application to validating a mechanistic model i have not seen

the observation: human genetic variation is a knockout library. a very large one, already assayed, with outcomes recorded in health systems

loss of function variants exist in most genes. for any target in my model, somewhere in a biobank there are people carrying heterozygous or occasionally homozygous LoF in that gene, with decades of linked phenotype

so for any target the model represents, i can ask what happens to humans who have less of it. thats a knockout experiment. it already ran. the ethics committee is not needed because it happened in utero

the canonical success is PCSK9. humans with LoF have low LDL and reduced cardiovascular disease, found in population data, and that observation drove a drug class. one human genetics finding was worth more than any mouse study of cholesterol metabolism, and it came for free from variation that already existed

the systematic version, which is the part i think is underexploited

for every protein in the rattus-human target set, query biobank for LoF and missense variants, stratify carriers, extract linked phenotypes, and compare against what my mechanistic model predicts for reduced function of that protein

thats a validation set with hundreds to thousands of natural perturbations, each with human outcome data, for free

and its the right KIND of validation. its an intervention on a mechanism, with a downstream organism level readout, in humans. thats exactly the propagation chain i cannot otherwise test in humans

mendelian randomisation makes it causal rather than correlational. variants are randomised at conception, so carrying a LoF allele is independent of confounders in a way no observational comparison can be. it is as close to a randomised trial of lifelong partial inhibition of a target as you will ever get

limits

lifelong versus acute. a LoF carrier has had reduced function since conception, with developmental compensation. an acute pharmacological inhibition in an adult is a different perturbation. MR tells you about chronic partial inhibition, not about a single dose. this is a genuine limitation and its also precisely quantifiable if you have a model with developmental compensation in it, which eventually i will

partial not complete. heterozygous LoF is usually ~50% dose, homozygous often lethal or absent. so you get the low-inhibition end of the curve and have to extrapolate

survivorship. you only observe carriers who lived to be enrolled in a biobank

pleiotropy. a variant affecting your target may affect others. standard MR sensitivity analyses handle this, MR-Egger, weighted median, and theyre well developed

but as a source of human-relevant, causally identified, mechanism-level validation data costing nothing to generate, i think this is the single most underused resource available to anyone building human mechanistic models. im building a systematic pipeline for it and that pipeline is probably the most immediately publishable thing in the human program

TODO: start with the ~40 targets in the dopaminergic slice that have human orthologs with LoF carriers in UKB. DRD2 variants, SLC6A3, COMT val158met which is everywhere in the literature already, MAOA. the COMT one is good as a positive control because the effect sizes are known and contested, so if my model reproduces the contested effect size distribution thats informative either way

## human PK and the variability that actually matters

PBPK is the most mature part of the human program and the part where i am mostly assembling rather than inventing

human PBPK is genuinely good. physiological compartments with measured organ volumes and blood flows, tissue partition from logP and tissue composition, metabolic clearance scaled from human microsomes and hepatocytes, renal clearance from physiology. regulatory submissions use it. there is commercial software that does most of this well

what i add is coupling it to the mechanistic PD stack rather than to an empirical effect compartment, and carrying uncertainty through the coupling instead of using point estimates

the variability structure is where the human version beats the rat decisively

sources of inter individual variation that actually matter, roughly ordered by effect size for a typical CNS compound

CYP genotype. 2D6 2C19 3A5 2C9. poor metabolizers can have exposures an order of magnitude above extensive metabolizers at the same dose. this is the dominant source for many drugs and it is entirely absent in a rat model

transporter genotype. ABCB1, SLCO1B1. affects distribution and particularly CNS penetration

body composition. volume of distribution for lipophilic compounds scales with fat mass not total mass, so the same mg/kg dose gives different exposure in different body compositions

renal function. eGFR varies enormously with age and disease and drives clearance for renally eliminated compounds

hepatic function. disease state, and also simply age

age, and specifically ontogeny. this is its own problem and its one of the places i think the computational approach has the clearest advantage, because paediatric dosing is currently derived by scaling adult doses with crude allometry and it is known to be wrong. enzyme ontogeny is non monotonic. CYP3A7 dominates in the fetus and is replaced by 3A4 postnatally. UGT activity matures over months to years, which is the mechanism behind grey baby syndrome with chloramphenicol. so a child is not a small adult and the scaling exponents that work between adults do not work across development

modeling ontogeny properly needs enzyme maturation functions, which exist in the literature as fitted curves, plus organ growth trajectories. its tractable and its high value because the alternative is dosing children on extrapolation

pregnancy. physiological changes in volume, cardiac output, renal flow, enzyme induction. and pregnant people are systematically excluded from trials, so dosing in pregnancy is mostly extrapolation. another place where a model is not competing with good data, its competing with no data

and that framing matters generally. for the populations most excluded from clinical research, children, pregnant people, the very old, people with multiple comorbidities, the current evidence base is extrapolation anyway. a mechanistic model does not have to beat a well powered trial in those groups. it has to beat a guess

virtual populations are the deliverable here. sample parameter sets consistent with observed human variation, run the compound through thousands of them, get a response distribution rather than a mean. the plausible-patient methodology for generating virtual cohorts that match observed covariance structure is published and in use, i am not inventing it, i am pointing it at a mechanistic PD model instead of an empirical one

the outlier tail is the whole point. you are not trying to predict the average patient, the average patient is usually fine. you are trying to find who is in the tail and why

## where im completely stuck

psychiatry. i have been going round this for two months and i am not out of it

the setup. CNS is where animal models fail worst. psychiatric drug development has a catastrophic failure rate, worse than oncology. the animal models are the forced swim test and its relatives, whose construct validity has been argued about for forty years without resolution. so psychiatry is simultaneously where replacement would be most valuable and where i have the least traction

here is the problem stated as cleanly as i can

for cardiac safety, the thing being predicted is an arrhythmia. an arrhythmia is a physical event with an unambiguous definition and a mechanism that runs through ion channels. that is why CiPA worked

for depression, the thing being predicted is... what exactly

the DSM categories are consensus clinical descriptions, not natural kinds. two people with the same diagnosis can share almost no symptoms. the diagnosis is a label applied to a heterogeneous population, and the heterogeneity is probably mechanistic, meaning different people in the same diagnostic box have different underlying biology

so the prediction target is not well defined. and a model cannot predict a target that is not well defined. this is not a modeling problem, its upstream of modeling

this is the same realisation as the 2am entry about behavioral ontology, except much worse, because in the rat case i could at least say predict the pose distribution and let labels be a readout layer. in the human case the thing people care about is the subjective report, and there is no pose distribution underneath it that i can measure instead

things i have tried thinking about and where each one breaks

RDoC. the proposal to replace DSM categories with dimensional constructs grounded in neurobiology. philosophically right and exactly what a mechanistic model wants. in practice the constructs are themselves not well validated and the field has not converged, so i would be predicting a target that is contested in a different way

predict biomarkers instead of symptoms. target engagement, receptor occupancy, circuit level changes measurable by imaging. this is tractable and its what i will actually do first. but it dodges the question. predicting that a compound occupies 70% of 5-HT2A receptors is not predicting whether it helps anyone

and the occupancy-to-outcome link is precisely where psychiatric drug development fails. we are already good at hitting targets. we are bad at knowing which target to hit

predict treatment response from baseline. a prediction problem rather than a mechanism problem, and theres real clinical value, but it does not need my architecture, its a statistical learning problem on EHR and imaging data and other people do it better

predict side effects. this i can do. weight gain, QT, EPS, sedation, sexual dysfunction all have reasonably clear mechanisms. and side effects are a large part of why psychiatric drugs fail in practice, so its genuinely useful. but again, it is not the hard part

where this leaves me, and its unsatisfying

i can build a human CNS model that predicts pharmacology, occupancy, circuit level effects, side effects, and interactions. i cannot currently build one that predicts whether a person feels better. and the second one is what the field needs

my current position, which may be a cope: the first one is still worth building, because if you could reliably predict occupancy, selectivity, CNS penetration, side effect burden, and drug interactions in humans without animal studies, you would eliminate most preclinical animal use in CNS drug development even if you never touched efficacy

efficacy still needs the trial. fine. the trial is in humans anyway. the animals were in the preclinical phase, and the preclinical phase is the part i can attack

so the honest scope for psychiatry is: replace the preclinical animal package, not the clinical trial. thats smaller than i wanted and its still thousands of animals per program

still stuck on the real version though. writing it here so i dont pretend ive solved it

### 2:50am, nov 11

up again. this one is about the stuck thing above and i dont know if its good

what if the reason psychiatric prediction fails is that were trying to predict a population when the object is an individual

like. the whole apparatus of drug development is built on average treatment effect. run a trial, compare means, if the drug beats placebo on average it works. and for psychiatry the average effect sizes are famously small, and everyone says thats because the drugs are weak

but what if the drugs arent weak. what if theyre strong in a subset and useless or harmful in another subset and the average is washing it out

that would explain basically everything. the small average effects. the enormous clinical variability every psychiatrist reports anecdotally. the fact that trial and error prescribing works better than the trial data predicts it should

and if THATS true then the entire prediction target is wrong. youre not trying to predict "does this drug work". youre trying to predict "for whom"

which is a conditional prediction and conditional predictions need individual level mechanism, which is exactly what a twin is

ok so the thing that woke me up. you cannot validate a conditional prediction against trial data because trials report averages. BUT. if you have a mechanistic model that predicts responder subgroups, you can go back to ARCHIVED trial data with individual patient records and ask whether your predicted responders actually responded

that is a retrospective validation of a conditional prediction using data that already exists. no new trial. no new animals. and individual patient data from completed trials is increasingly available through managed access

so: predict responder stratification mechanistically, validate against archived IPD, prospectively lock before requesting access

thats... actually a protocol. thats a real thing i could do

and if it works it inverts the psychiatry problem. instead of predicting an average effect that is small and contested, predict heterogeneity, which is large and currently unexplained

TODO TOMORROW: find out what the access process looks like for completed psychiatric trial IPD. there are managed access platforms, i have never used one. also check whether anyone has done mechanistic responder prediction validated on archived IPD, feels like someone must have but the combination with a mechanistic rather than statistical model may be open

(next day. people do responder prediction, its mostly statistical ML on baseline features. mechanistic stratification validated against archived IPD i cannot find. the access process exists and is slow and requires a locked analysis plan, which is actually perfect because locking the analysis plan IS the prospective locking discipline, enforced by someone else)

(two days later, less excited. the IPD available is mostly from industry trials of drugs that worked. survivorship. the failed trials, which are the informative ones, are mostly not shared. this is the negative results problem again. it keeps being the negative results problem)

## the counterfactual problem

the deepest methodological issue in the human program and i want to state it properly rather than gesture at it

a twin predicts what would happen under an intervention that did not occur. thats a counterfactual. and counterfactuals are not directly observable, ever, by anyone. the fundamental problem of causal inference

for the rat i sidestep this by running the experiment. give the drug, measure. for humans i mostly cannot

so the human model makes claims that are structurally unverifiable at the individual level. i can verify them at the population level, predicted response distribution versus observed distribution in a trial, but the individual prediction, this patient would have responded, has no ground truth. ever. the patient either got the drug or didnt

this is not a data problem. more data does not fix it. its the structure of the question

what partially gets you out

consistency conditions. if the model predicts individual responses, those have to aggregate to the observed population response. thats a real constraint and models fail it more often than you'd expect. necessary not sufficient

natural experiments. instrumental variables where treatment assignment was effectively random for non-clinical reasons. formulary changes, supply shortages, physician prescribing preference as an instrument. these give quasi-randomised individual level comparisons and theyre underused

n-of-1 designs. for chronic stable conditions with reversible treatments you can randomise WITHIN a person. crossover, washout, repeat. that gives individual level causal estimates, actual ones, and its the only design that does. underused because its slow and unprofitable, not because its bad

mechanistic constraint as a substitute for counterfactual observation. this is the one i keep coming back to and im not certain its valid. the argument: if the model gets the mechanism right, verified at every intermediate level against observable data, then the counterfactual follows from the mechanism even though it cannot be observed. you trade unobservable outcome validation for observable mechanism validation

thats the entire epistemological bet of the project when you strip it down. mechanism validated piecewise substitutes for outcome validated directly

and i think its a reasonable bet and i want to be clear that it IS a bet. its the same bet engineering makes when it certifies a bridge by validating material properties and structural analysis rather than by loading bridges until they fall. that analogy is doing a lot of work and bridges are much simpler than people

the asymmetry that worries me: a bridge model that is wrong produces a bridge that falls down, visibly, and the model gets corrected. a twin that is wrong about an individual produces a treatment decision whose counterfactual nobody observes. the error is invisible by construction

so errors do not generate correction signal. that is the thing that makes this different from every other kind of modeling i have done and i do not have a good answer to it

partial answer: mandatory outcome tracking. if a twin informs a decision, the outcome gets recorded and fed back, and even without the counterfactual, systematic miscalibration shows up in aggregate over time. slow, weak, better than nothing

## virtual populations, the near thing

the least speculative item in the human program and the one i should lead with when talking to anyone who funds things

instead of one average patient, sample thousands of parameter sets consistent with observed human variation, run the compound through all of them, get a distribution of responses

the methodology exists. generate candidate virtual patients by sampling physiological parameters, then accept or reject against observed covariance structure and plausibility constraints so the virtual cohort matches real population statistics. its published, its used, i am not inventing it

what it answers that a mean cannot

who are the outliers. what makes them outliers mechanistically. what dose covers the population without poisoning the tail. what fraction of the population is at risk at a given exposure. which covariates predict membership in the tail

and critically: whether a safety signal seen in one person is consistent with the mechanism or indicates something the model does not contain

where this replaces animals right now, today, with no new science

dose range finding. currently done in animals to pick doses for first in human. a validated PBPK plus mechanistic PD virtual population can do this, and in some settings already does

DDI prediction. drug drug interaction studies are substantially model-based already, regulators accept PBPK predictions in place of some clinical DDI studies. this is settled practice, not a proposal

special population dosing. paediatric, renal impairment, hepatic impairment. model- informed dosing is already how a lot of this is handled

so part of my pitch is not a future capability, its pointing at something already working and saying extend it. the extension is coupling to mechanistic PD rather than empirical PD, which widens the applicable domain from exposure prediction to effect prediction

the gap in current practice, and where i actually add something: PBPK predicts exposure well. the link from exposure to effect is usually an empirical Emax curve fitted to clinical data. so you cannot use it for a compound with no clinical data, which is exactly the preclinical situation

replace the empirical PD with the mechanistic stack and the method extends to novel compounds. thats the contribution, stated narrowly, and its defensible

how id sequence it

start with a compound class where mechanism is well characterised and human data is abundant, so the mechanistic PD can be validated against known answers. D2 antagonists are perfect, theres PET occupancy data, clinical dose response, known side effect thresholds. show the mechanistic version reproduces what the empirical version gets, then show it extrapolates to a compound held out of the fitting

then move outward to classes with less data, where the empirical approach fails and the mechanistic one is the only option

that is a five year program with publishable results every year and it does not require solving psychiatry or building a whole human. i should probably be doing this instead of thinking about digital twins at 3am

## individual twins, the far thing

four things, roughly in order of how soon they could exist. the first two are nearly here and boring, the last two are exciting and distant, and the temptation is to pitch the exciting ones and deliver nothing

one, external control arms. patient enrolls, you have genotype, baseline labs, imaging, history. the model predicts their trajectory WITHOUT treatment. compare to observed trajectory on treatment. the control group is simulated from historical data

being done commercially already for prognostic models in some indications. fewer patients on placebo, which is its own ethical win, and validation is natural because you accumulate real trajectories continuously and can check calibration forever

my addition is mechanism. current approaches are statistical prognostic models. a mechanistic twin could predict response to a specific intervention mechanism rather than just disease progression, which is a much stronger claim and much harder

two, n-of-1 dose optimisation. the one i find most compelling clinically. patient on a drug with a narrow therapeutic window. twin initialised from genotype, CYP status, transporter variants, receptor variants, their labs, their measured response to the first dose. model proposes the next dose. observed response updates the twin. converges on the individual

this is sequential bayesian inference on a mechanistic model and most of the infrastructure exists. the barrier is regulatory and liability, not technical. warfarin dosing algorithms already do a crude version with pharmacogenomics

three, polypharmacy management. older patients on eight medications, interactions nobody has studied because the combinatorics make trials impossible. you cannot run a trial for every triple of drugs. a mechanistic model can evaluate the interaction without the trial because interactions emerge from shared metabolic and target pathways

this one i think is genuinely underrated as an application. the clinical need is enormous, the evidence base is nearly empty, and the combinatorial explosion means it will NEVER be filled empirically. a model is not competing with evidence here, its competing with nothing

four, the full twin. continuously updated from wearables, labs, imaging, prescriptions. ask it counterfactuals. what if we add this. what if we stop that. what does the combination do to QT given this persons specific CYP genotype and current regimen

fifteen plus years and i should say so every time i mention it

the thing that makes the whole ladder coherent is that each rung validates the one above. external control arms validate trajectory prediction. n-of-1 validates individual parameter inference. polypharmacy validates interaction mechanism. by the time you get to the full twin, every component has been separately validated in a setting where it had ground truth

thats the only path i can see that does not require trusting a twin before anything about it has been checked

## the part that bothers me

havent worked this out. writing it down because leaving it out because its inconvenient would be a choice

a predictive model of a specific person is not the same kind of object as their medical record

a record says what happened. a twin says what would happen. and it says it about situations that have not occurred, which means it generates information about a person that they do not have and cannot meaningfully consent to, because neither of us knows what it is until its computed

consent cannot really be informed. someone consents to a twin built from their genome, imaging, records. they cannot consent to every future query because the queries do not exist yet. then in six years someone asks the twin about their dementia risk, or addiction liability, or response to a drug they have not been offered. they consented to the model. did they consent to that

re-identification. a twin is fitted to an individual. its parameters ARE identifying, inherently, arguably more than a genome because it encodes dynamics not just sequence. you cannot anonymise a model whose entire value is its specificity. differential privacy breaks the thing that makes it useful. this is not a tooling gap, its a tension in the object itself

and the uses i do not control. insurers, employers, courts. a twin that predicts drug response also predicts disease trajectory, which is what an insurer wants. model says 60% chance of early onset X is an actuarial input. i would be building the most invasive health prediction instrument available and intending it for dose optimisation is not a safeguard, its a hope

the one that unsettles me most: a twin can be wrong about someone in a way they cannot contest. if the model says a patient will not respond and the treatment is therefore not offered, how does that patient argue. the counterfactual never runs. you cannot appeal a simulation

and this connects straight to the counterfactual problem from earlier. its the same structural fact wearing an ethical face. errors are invisible, so the people harmed by them have no evidence they were harmed

clinical prediction instruments already get used this way, so this is not hypothetical

partial answers, none sufficient

the abstention machinery helps more here than it does anywhere else. a model that refuses poorly supported questions refuses the actuarial ones too, because those are long horizon multifactorial predictions where uncertainty is genuinely enormous. so calibrated uncertainty functions as a privacy mechanism, which is a connection i did not expect and like a lot

narrow context of use helps. a twin qualified only for dose selection in drug class X should be architecturally incapable of answering a dementia question. not policy-prevented, incapable. if the model only represents pathways relevant to its COU then the other question has no referent in it

thats a design constraint i should adopt NOW, in the rat phase, because it will be impossible to retrofit. build narrow models that compose, never one general organism model. which is also better science and better regulatory strategy, so the ethical constraint and the engineering constraint point the same direction. ill take it

patient held twins, where the model lives with the person and queries need their key. technically plausible, institutionally unlikely, nobody builds infrastructure that way

none of this is solved. the position i actually hold is that this technology has a dual use structure and the mitigations are partial, and id rather write that down now while it costs me nothing than notice later that i had quietly stopped mentioning it

the rat version has no version of this problem, which is part of why its a comfortable place to work, and i should notice that i find it comfortable

## where i actually am

deflating this on purpose because everything above is planning and this is the part thats real

the prototype takes a novel compounds molecular structure and relevant pharmacological parameters, simulates its predicted receptor interactions, intracellular signaling effects, and downstream responses in a virtual dopaminergic cell. outputs predicted binding behaviour, signaling changes, cellular responses, and quantified uncertainty, benchmarked against published experimental datasets to see how closely its predictions reproduce independently observed results

that is one cell. one pathway. one receptor family

its not a rat. its nowhere near a rat. its maybe 1% of the first vertical slice and the slice is 1% of the thing

but its a complete causal chain. compound in, mechanism through, prediction out, with uncertainty, checked against data that already existed. thin and unbroken

what works

molecular band end to end for D2. structure to occupancy to efficacy to bias, calibrated against PDSP and the published BRET sets. generalizes to held out scaffolds at roughly the level my prior work does, which is to say decent for affinity, mediocre for bias

signaling cascade through cAMP and PKA to DARPP-32 phosphorylation state. mechanistic, parameters from literature plus fitted, reparameterised onto the identified manifold after the september mess

uncertainty propagation end to end. SBI posterior over the mechanistic parameters, ensemble over the learned part, conformal intervals on the output. calibration checked on held out compounds and it is roughly right, slightly overconfident at the tails which is the usual direction

abstention triggers. they fire. they fired correctly on three of four deliberately out of domain compounds and failed on the sigma-1 case described earlier

what does not work

circuit level. i have a striatal microcircuit that runs and produces plausible firing statistics and i do not trust it. the connectivity is from mesoscale tract tracing plus assumptions and i cannot validate the assumptions

behavior. nothing. not attempted beyond reading

chronic adaptation. the receptor trafficking model exists, it is not validated against anything longer than hours

the human side. nothing built. the engagement invariance test is scoped and not run. the rosetta set is a proposal. the adapter rank idea is untested. the LoF mining pipeline is the one piece i could start tomorrow and have not

nearest term things, in the order im actually going to do them

1. run the engagement invariance analysis on existing rat/human discordant compounds. entirely computational, uses existing data, and the result is informative whichever way it goes
2. LoF mining pipeline for the ~40 dopaminergic targets with UKB carriers
3. the adapter rank pilot using cell lines as a stand-in for species
4. write up the molecular band properly, it is publishable now and i have been sitting on it

and the thing i keep not doing, which is finding someone who thinks this is nonsense and making them read it

## not going to oversell the ending

the thing i keep landing on is that the current system is bad at the job it exists to do. around 90% of drugs entering clinical trials fail, and a large share fail on efficacy or toxicity the animal work did not predict. so we kill an enormous number of animals for a predictive instrument that is wrong most of the time, and the reason we keep doing it is institutional inertia rather than evidence that it works

thats not an animal welfare argument, its a methodology argument. the welfare argument points the same direction and i think thats worth saying rather than hiding. i got advised early to lead with the science and bury the ethics. i dont think thats right. the ethics is why the methodology question was worth asking in the first place

and the policy environment moved while i was writing this, which i did not expect. FDA announced in april 2025 a plan to phase out the animal testing requirement for monoclonal antibodies and other drugs, with a roadmap putting animal studies as the exception rather than the norm within three to five years. NIH stood up ORIVA in june 2026 and committed over $150M under Complement-ARIE, and said it would stop soliciting animal-only research. HHS added another wave in september

so the mandate and the money are both moving. the bottleneck is not permission, its evidence. nobody has qualified a neuro in silico model for anything. the opening is real and currently empty

CiPA is the proof the shape works. cardiac proarrhythmia, mechanistic human ionic model, in vitro channel data in, torsade risk out, accepted in place of animal and clinical studies. a mechanistic simulation replacing experiments, already done, in a different organ system

the reason it worked first for the heart is instructive and slightly discouraging. ion channel biophysics is unusually well characterised, the readout is electrical and unambiguous, the mechanism is narrow, and the models had thirty years of development. neuro has none of those. so the path exists and its longer

what i did not expect when i started, and the actual reason im still doing it: the human version is not the risky speculative extension. its the part with better data, less ethical friction, and more relevance. the rat was always the proving ground. we just forgot that and started treating it as the answer

long way off. most of this will turn out wrong. fine

discovery shouldnt require exploitation. thats the whole thing

if youve read this far, text me, i want to know which part you think is weakest. specifically looking for someone to tell me the behavior link is hopeless because i half believe it and want to be argued into or out of it

also the tables still dont finish
