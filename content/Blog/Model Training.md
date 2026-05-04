## Overview of stages of training:
1) **Pre-training**: teach language modelling capabilities, general purpose knowledge formation
	- training objective: next token prediction
	- dataset: high-quality and low-quality data; different domains of data (e.g. english web data, multilingual, code, math)
2) **Mid-training**: 
	- model training is often multi-stage for two reasons: 
		1. increase context length steadily (e.g. 4K -> 32K -> 64K...)
		2. multi-stage curriculum of data of different quality/domain, in order to save the best for the last
	- mid-training is commonly used to sharpen core skills, such as math and code reasoning. we might train not only on natural datasets, but also distilled reasoning traces from better models. 
3) **Post-training**: further develop specific skills
	- e.g. task-specific models (math, coder...)
	- style adaptation (e.g. roleplay)
	- behavioural alignment/debiasing

	**Methods**:
	1. **SFT**: also next-token prediction, but on a more narrow dataset
		- while pre-training focuses on general language modelling, SFT might focus on a specific task formulation (e.g. instruction finetuning --> making sure that a model knows to respond as an assistant, instead of continuing the instruction.)
		- some modern SFT datasets are distillation datasets from stronger models. 
		- benefits: cheap and stable, downsides: fundamentally capped by quality of dataset (imitation learning)
	2. **RL**: action-based: given a task and an environment, depending on your action, the model receives a reward/punishment that changes its behaviour
		- **RLVR** makes use of verifiers to distribute rewards to the model
			- good for: binary signals, bad for: process rewards (e.g. multi-turn conversations)
		- the model's weights are then modified according to an RL algorithm (e.g. PPO/GRPO).
		- contrast with **RLHF**: use a neural reward model trained on (collected) human preferences, to approximate the true reward. Because the reward model is also an NN, it can be gamed or undergo out of distribution reward drift. In contrast, RLVR depends on a deterministic, rule-based reward schema that is reliable.


## Literature Review
#### [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155)
- curated a high-quality set of prompts consisting of instruction and query-response pairs, centred around user use-cases, sometimes infringing upon inappropriate behaviours
	- created 3 datasets based on that: 
		- SFT dataset with labeller demonstrations;
		- RM dataset with labeller rankings of model outputs;
		- PPO dataset used as input for RLHF with RM.
- SFT and RL serve as two avenues to encode human preferences.
- InstructGPT is more reliable and easier to control than GPT-3. 
	- general language modelling ability does not give rise to the ability to follow human preferences.

#### ⁠[DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning](https://arxiv.org/abs/2501.12948)
DeepSeek-R1-Zero:
- implements post-training with GRPO
- refrains from using neural reward models, resorting to verifiable rewards for accuracy and formatting. 
- What is observed during training under this reward system? The model learns to leverage CoT, with its reasoning traces getting longer during training. it learns reasoning behaviours such as systematic exploration, and reflection.
- This model acquired more robust and generalisable reasoning capabilities than those provided by human-produced SFT datasets, which often lack reflection and verification steps. We can distill this "new form of reasoning" onto other models to promote better reasoning.
DeepSeek-R1:
- uses RLHF for helpfulness, harmlessness, language consistency, mixed with verifiable rewards.

## Questions to think about:
### What exactly does SFT teach? 
Conventional wisdom tells us SFT is less useful for learning new facts, but more useful for learning how these facts are expressed. This is a broad description that includes learning: 
- output formatting and style
- model personalisation and behaviour
- task-specific understanding

On the other hand, fine-tuning has been shown consistently to fail at teaching models new, unseen knowledge. In fact, exposing models to unseen knowledge during fine-tuning might lead to an [increased tendency to hallucinate](https://arxiv.org/pdf/2405.05904). These hallucinations often originate from [using related concepts from the fine-tuning dataset erroneously](https://arxiv.org/pdf/2402.05119), indicating that the model has not generalised from the knowledge learned in the fine-tuning dataset. 

From a mechanistic perspective, fine-tuning has been found to change only a small part of the weights of the model, [enhancing](https://arxiv.org/abs/2402.14811) rather than altering existing representations. Additionally, any skills unlearned in fine-tuning [can be easily recovered](https://arxiv.org/pdf/2311.12786), showing that fundamental capabilities remain unchanged, and also motivating the field of machine unlearning at large.

More broadly, the question of what a model learns through a training process can also be reduced to the question of differences between any two networks. This relates to [model diffing](https://transformer-circuits.pub/2024/crosscoders/index.html#model-diffing-background).

### ⁠What types of tasks is RLVR well-suited for (or not)?
Naively, RLVR is suited for domains where tasks have easily verifiable outcomes and a clear correct/incorrect partition, such as math reasoning and programming tasks. 

However, the allure of RLVR as a post-training paradigm incentivises us to expand scoring-based rewards to more open-ended domains. Various methods include using LLMs-as-judges guided with rubrics, as well as simpler, rule-based verification. 

###### Useful References:
- [Frontier Model Training Methodologies by Alex Wa](https://djdumpling.github.io/2026/01/31/frontier_training.html)
- [Rubric-Based Rewards for RL by Cameron R. Wolfe](https://cameronrwolfe.substack.com/p/rubric-rl)
