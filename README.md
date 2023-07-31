# Implementation of Direct Preference Optimization 

[Paper Link](https://arxiv.org/pdf/2305.18290.pdf)

Direct Preference Optimization (DPO) is a way in which you can circumvent the reward modelling step in Reinforcement Learning from Human Feedback (RLHF) and directly train the underlying policy. If you do not know how to perform RLHF, then the paper has a good summary for it. I will rewrite it here for you too :) 

RLHF has 3 stages of development: 
1.  Firstly, you need to fine-tune a Large Language Model (LLM) to do your bidding. This can be anything from fine-tuning the model to create a chatbot, or to answer questions, or to follow instructions. This model is then called the Supervised Fine-Tuned (SFT) model. 
2.  Secondly, the SFT model is prompted with $x$ to create two outputs $(y_1, y_2)$ which are then ranked by human beings as being better and worse $(y_l, y_w)$. With this new dataset of good and bad outcomes, they are used to train another LLM model that will function as a reward model $(r_\phi)$. This reward model can be the same SFT model with a linear layer on top to serve as a classifier. 
3.  Finally, using the reward model to produce rewards, the LLM is then trained using a reinforcement learning algorithm (most probably Proximal Policy Optimization, also called PPO).

That is a cheap rundown of what happens in RLHF. The authors in the paper rightly state that it is a very complex and unstable process. They aim to change that and hence we have DPO. 

DPO has 3 stages of development: 
1.  The first step is very similar to RLHF's first step. A dataset is created with good and bad outcomes. 
2.  The second step is also very similar to step 2 of RLHF. Although, the paper suggests that using data generated by our own model is great, we will need to use datasets that are publicly available. The base models used to generate the publicly available data is different from our own models, so, we cannot realistically initialize $\pi^{SFT}$ with $\pi_{ref}$. Hence, we will need to fine-tune our own model by maximizing likelihood of preferred completions $(x, y_w)$. 
3.  Now, we will need to optimize the language model $\pi_\theta$ to minimize $L_{DPO}$. The optimization function is given by: $$L_{DPO}(\pi_\theta ; \pi_{ref}) = - E_{(x, y_w, y_l) ~ D}[\log \sigma(\beta \log(\frac{\pi_\theta(y_w|x)}{\pi_{ref}(y_w|x)}) - \beta \log(\frac{\pi_\theta(y_l|x)}{\pi_{ref}(y_l|x)}))]$$ 

After I trained the model on the DPO objective, I noted down the responses from the different models in the "results" folder. I trained the DPO model on good responses to create a "good" model, and I also trained it on all the bad responses to create the ultimate bad model. In the end, the models performed well. Since the dataset is meant to train models that are "helpful" and "harmless", that might just be what I got in return. The bad model were not as toxic as I thought they would be (maybe the dataset was not toxic like I thought they would be). Although, when I was playing around with the models, I found out that the "bad" model was very sassy, and was particularly sarcastic at some moments. In the end, I preferred the outputs given by the good model.

Overall, I think it might be a good substitute for RLHF. It is definitely more stable than RLHF, although I found out that while training the model in step 2 you should not train it to its optimum performance because then it doesn't learn anything in step 3.