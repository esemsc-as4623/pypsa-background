
- unclear how this differs from explainability
- not ambitious enough
- need to incorporate ARIA opportunity space language and theme more clearly
---
Tell them about me. What is your most ambitious goal and how does the Encode fellowship help me get there?
---
- worked on different things
	- private, public, academic
- I like having an impact on decision-making
- explainability & robustness
	- 

I have spent the last several years building models that are supposed to help people make decisions: recommending what a tourist should do with their afternoon in Singapore, flagging which jobseekers need government intervention before they fall too far behind the market, estimating financial crisis risk in emerging economies where data is scarce. Now, in my PhD, I am trying to help grid planners decide how much energy storage to build. This last one is the most critical and irreversible of all: a decision we make now, that will shape all future decisions indefinitely.

In all my projects, my contribution starts at messy, sparse, gap-filled data and ends at building a model and trying to understand what it is doing. This is something I care about. I have used the full suite of tools related to interpretability: disentanglement metrics, attention visualisations, SHAP values, ablation studies. A model encodes something, produces an output, and then someone maps that to some decision boundary. 

I am now interested in designing methods to test whether what a model is doing is wrong in a way that matters. The distinction between explaining a model and being able to falsify its claims is what I have been circling for a few years without having the language for it.

In my MSc thesis on financial crisis prediction, I spent more time questioning the label than training the model. A financial crisis is only apparent in hindsight. The ground truth is constructed retrospectively by economists. If the thing you are predicting is epistemically contested, the model's confidence is not just uncertain — it is answering a question that may not have been well-posed. I published that not as a limitation to bury in the appendix but as the central finding.

I read a paper on using a variational autoencoder to infer thermophysical properties of the lunar surface, bypassing a physics simulation that would have taken orders of magnitude longer to run. The result was compelling. The author, now a professor in my department, found that the latent space of an unsupervised model can be loosely correlated with simplified physics. I extended this work by designing a geophysical experiment that the model could lose. I identified a geological feature — a rille system in a crater — where the physics makes a directional prediction about what the model's proxy for thermal inertia should show, and I wrote the null hypothesis, a falsifiable claim, before running the test.

My PhD chapter asks the same question in a different domain. I am comparing two differing but defensible representations of energy system models, one that preserves temporal chronology with one that aggregates time into representative periods. My hypothesis is that this design choice, regardless of the temporal resolution, can result in the models disagreeing on the recommended storage capacity. The field knows this. What is missing is a diagnostic: under what mathematical conditions does the formulation choice corrupt the recommendation, and how do we know when we are in that regime? I am building that from the constraint matrix up — condition numbers, degeneracy metrics, sensitivity analysis — because "the models disagree" is not actionable and "here is why and here is a threshold for when it matters" is.

Decisions depend on whether a model's encoding is trustworthy. I am interested in building domain-specific, comprehensive, and scalable ways to stress-test the claim a model makes about the world. Given what a model claims to have encoded — thermal inertia, crisis precursor, storage arbitrage value — what observable contrast would be inconsistent with that claim, and does the model's own uncertainty quantify the risk of being wrong in a way the domain expert can act on?

---
What technical project / achievement am I most proud of and why?
---

I predicted lightning.
- predicting lightning
- lunar VAE
---
Do I have any experience working with cross-disciplinary teams?
---

- fram
- social choice theory

Technical Demo
- I care 