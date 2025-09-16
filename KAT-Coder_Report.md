# KAT-Coder: Unleashing the Full Potential of LLM in Agentic Coding
 
## Model
Today, we're thrilled to announce two groundbreaking models in our KAT series: KAT-Dev-32B and KAT-Coder. We are excited to introduce KAT-Dev-32B - our new open-source 32B model for software engineering tasks, and KAT-Dev-32B is released to the community for further research and development and you can find it at https://huggingface.co/Kwaipilot/KAT-Dev. Moreover, KAT-Coder is our most powerful variant and marks ultimate performance in code intelligence.

Key Contributions
Our KAT-Coder and KAT-Dev-32B are optimized via several stages of training, including a mid-training stage, supervised fine-tuning (SFT) & reinforcement fine-tuning (RFT) stage and an large-scale agentic reinforcement learning (RL) stage. In summary, our contributions include:
🎯 Mid-Training: We observe that adding extensive training for tool-use capability, multi-turn interaction, and instruction-following at this stage may not yield large performance gains in the current results (e.g., on leaderboards like SWE-bench), but it will have a significant impact on the subsequent SFT and RL stages;
🎯 SFT & RFT: We meticulously curated eight task types and eight programming scenarios during the SFT stage to ensure the model's generalization and comprehensive capabilities. Moreover, before RL, we innovatively introduced an RFT stage with "teacher trajectories" annotated by human engineers as guidance during training;
🎯 Agentic RL Scaling: Scaling agentic RL hinges on three challenges: efficient learning over nonlinear trajectory histories, leveraging intrinsic model signals, and building scalable high-throughput infrastructure. We address these with prefix caching on logprob computation, entropy-based trajectory pruning, and SeamlessFlow architecture.

You can learn more details about our models in https://kwaipilot.github.io/KAT-Coder/.


## Workflow
In addition to adopting a similar three-stage pipeline (Generation, Filtering, and Sorting) as prior work, we innovatively introduce a Reflection phase.
 

### 1. Generation
Benefiting from exceptional function-call and instruction compliance capabilities, the KAT-Coder model can autonomously utilize the following tools for contextual information retrieval and patch generation.
● Bash：The model can search for and read files by executing shell commands.
● str_replace_editor：Editing tool for viewing, creating and editing files.
● submit：Finish the interaction when the task is complete OR if the assistant cannot proceed further with the task.

We recognize that the diversity of generated patches is crucial to the final performance,, and therefore we employ a relatively high temperature value during generation process.

### 2. Filtering
We filter candidate patches through two steps to discard those containing obvious errors. Reducing the number of candidate answers helps improve the accuracy of the model's judgment in subsequent steps. Firstly, we perform deduplication based on code semantics. We employ a grammar-based representation model for code, where the encoding method effectively captures subtle distinctions between code patches. This is followed by execution-based filtering, utilizing the model to generate test cases specifically tailored to each issue. Meanwhile, inspired by the TTS strategy, we collect multiple possible test cases for each issue. The candidate patches are then ranked based on the number of passed test cases, and the top-k candidates are then selected for the next stage.

### 3. Sorting
We employ the LLM-as-a-judge approach, prompting the model to assign quality scores for the remaining patches, and the one with the highest score will be selected as the final submission.

### 4. Reflection
Previous approaches typically execute the entire workflow in a single round, resulting in low-quality submissions. To address this limitation, we introduce an additional Reflection phase. Rather than allocating the total budget for K generations in one batch, we distribute it across multiple rounds. An initial round is first conducted to accumulate experience, and the model’s analysis of previous answers is incorporated as reflective knowledge to guide subsequent generations.

