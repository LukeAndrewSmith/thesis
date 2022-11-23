

# TODO Humor: 
## Current
- Next
    - Visualisations
        - Frame evolution
            - Double check it works with new intermediate_results
        - Loss
        - Convergence
    - Question
        - How does the prior deal with out of distribution motion
            - Issue, openpose has too many errors with e.g rolling. so we can't evaluate the prior properly
            - **Solution: dataset with labeled 2d joints (or run on amass and visualise just the motion without video)**
        - Take new video with occlusions
        - Want this to be done by the weekend so we can run over the weekend

- Losses
    - Can we comment out the ground-plane loss function for videos where the ground plane is definitely not useful, e.g not on the stairs

- Create visualisations
    - Losses and intermediate info
    - Progression of certain states over time
    - Run the model on a bunch of examples and see where it breaks
        - TODO: check all the videos are 30fps
- Ground truth
    - What exactly is the ground truth?
    - Anything like fraser_lying_floor
- **Understand how they batch in more detail**
    - Look exaclt at dimensions of everything
    - Notes on batching
        - To account for videos with not exactly a multiple of the 
            - They increase the overlap size?
            - c.f RGBVideoDataset

- How to speed up optimiser, does adam work?
- Steup vpn and ssh for ubuntu so I can login to the machine from home and check things over the weekend

- Other parameters to make notes on
    - Camera intrincs - how they are estimated

### Batching Notes
* To understand
    * In the batch rollout, they seem to rollout the batches together, how do we deal with overlap?
    * Is there overlap within the batches? yes right? so are we optimising the same variable multiple times? if so how is it aggregated? which ones do we take?
    * **The overlaps within a batch could be just references to the same variables, double check this**

## Eventually
- Comparison optimisation methods: **Eventually**
    - If we implement a new optimisation method, we need some way of comparing it to the old one
        - We also need a way of easliy plugging in a different optimiser and generating the whole suite of evaluations
        - **Smaller video for development**
    1. Quantitative
        - Errors computed in the paper over whole datasets
            - See what they're based on, might just be the i3DB stuff
    2. Qualitative
        - Visualisations of convergences to the final state (like the one from yesterday) for a selection of videos, e.g with specific cases we care about like occlusions
            - **Take the final optimisation x's as ground truth and compare the x's during optimisation to the 'ground truth' x's to see how they are converging**
                - Why is the $x_0$ so high at the beginnning
        - Videos of how certain states changes over the iterations
        - Plots of their losses over time
    - Optimisations to try out
        - **Try example with much smaller sequence length**
            - What is the difference between batch size and sequence length?
            - What batch size should it be?
        - **Jakob's Idea**
            - Aim: avoid rollout
            - Find $z_{t}^{iteration_{s}}$ based on running $x_{t-1}^{iteration_{s-1}}$ then run through decoder to find $x_{t}^{iteration_{s}}$ used for the next iteration step
                - Idea: rather than propagating consistency through rollout during a single iteration the influence of the $z_{t's}$ propagates through the iterations, and we can have many more iterations as this would be a much quicker process
            - Implementation: this would require modifying the function ```stage3_testop```
- Lower priority
    - Did they directly evaluate the VAE?
    - Code Details
        - Understand the overlap_loss better
        - Understand the rollout
            - humor_model.roll_out()





---
# Humor TestOps Notes
## Optimisation overview
1. $\textsf{Stage 1 - 30 steps}$
    1. Optimise global (root) translation and orientation, without humor
2. $\textsf{Stage 2 - 80 steps}$
    1. Optimise full pose and shape, using VPoser prior
3. $\textsf{Stage 3 - 70 steps}$ (c.f iPad notes for a graphical overview of the optimisation steps)
    1. $\textsf{Init}$
        1. $x_{1:T} \gets \textsf{estimate\_x}(\theta)$ (joints from poses, velocities from finite differences)
        2. $z_t \gets \textsf{encoder}(x_t, x_{t-1})$
            - No guarantee of globally consistent motion as we are stacking locally consistent latent variables
            - i.e Rollout on init look **bad**, lots of drift
    2. $\textsf{Optimisation}$
        1. $\textsf{lbfgs\_max\_iter} \gets 20$
        1. $\textsf{For } 1...30$
            1. $\textsf{rollout}(z_{1:15})$
            2. $\textsf{lbfgs.step}()$ for ${\bf x_0, z_{1:15}}, g, \beta$
        1. $\textsf{Freeze } x_0$
        1. $\textsf{For } 1...25$
            1. $\textsf{rollout}(z_{1:T})$
            2. $\textsf{lbfgs.step}()$ for ${\bf z_{1:T}}, g, \beta$
        1. $\textsf{Unfreeze } x_0$
        1. $\textsf{For } 1...15$
            1. $\textsf{rollout}(z_{1:T})$
            2. $\textsf{lbfgs.step}()$ for ${\bf x_0, z_{1:T}}, g, \beta$

## Other notes
- Optimisation objective
    - Usage of prior
        - Prior is included as a loss term in the optimisation
            - We sum the log probabilites of $z_t$ given $x_{t-1}$ where $x_{t-1}$ is given by the rollout up to that point
- Split video sequence up into overlapping 3s clips
- fit_rgb_demo_no_split.py and fit_rgb_demo_use_split.py seem to take a similar amount of time, on the order of 20mins...
    - Should time them both to check
    - Actually I think the batch size in this case is 2s (60frames at 30Hz) hence 2 batches with 10frame overlap doesn't change much I presume...
    - Trying much smaller batch
- Speed
    - GPU usage doesn't seem to go above 33%
    - Profiling notes
        - c.f humor/profiling/TestOpsProfile.prof
        - Stage III is significantly slower that I/II
            - I/II/III = 1/4/89% of total runtime
        - Stage III
            - ```rollout_latent_motion``` and ```backward``` take up equal amounts of time
        - LBFGS takes up lots of time in profiler - misleading
            - optimizer.step(closure): closure has the rollout and backward included in it, these take up time and are called in the lbfgs hence lbfgs looks slow



# Thoughts
Main issues of HuMoR (and a little on our method) (should be useful in determining the future directions of the work)
- Ground plane reliance 
    - Used for rescaling of the pose to a canonical reference frame, in which the prior is trained
    - Won't generalise well to many real world situations (we might not care?) where the ground plane cannot be accurately estimated
    - And the method we estimate the ground plane is questionable
- Missing useful motion data
    - The data sets don't describe some motions that might be important to us, like lying down, and motions like this would mess with our existing method for finding the ground plane
- Simplistic contact model
    - in our current pipeline and in HuMoR
- Slow motion generation (TestOps)
    - Optimisation rather than direct prediction, could be a problem
Not sure if good:
- Reliance on SMPL?
- Lot's of hand tuned regularisers?