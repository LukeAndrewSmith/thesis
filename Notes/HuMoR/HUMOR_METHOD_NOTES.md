
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


## Batching Notes
* The video is split up into overlapping sequences in the DataSet
* A number of overlapping sequences (max 4 on my gpu) are joined into a batch and processed together, by the DataLoader
    * There is an overlap loss between batches, and within a batch (in 'fitting_loss.py')
        * Between batches: look for 'rgb_overlap_consist'  
        * In batch: look for 'seq_interval'


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

# Humor quirks
## Bugs
- x_0 bad velocities in initial rollout
        - c.f my pull request on their repo
## Quirks
- Angle representation
    - Model input: rotation matrix
    - Model output: axis angle
    - Optimisation variable: axis angle
- Issue
    - Apply and inverting cam2prior results in rotation vectors (root_orient) that represent the same values but are distinct (e.g flipped), this will mess up the optimisation if we have a consistency loss, as the two distinct aa's will be forced to change and move closer together even though they already represent the same rotation
    - Solution
        - use 6d rotation representation
        - their model takes rotation matrices (and returns aa for some reason?), therefore we can optimise in 6d to avoid this issue, and convert to aa for certain of their functions where necessary
    - Note:
        - This was the cause of the issue where x_0 was still moving even when we detached grads for x_decoded so no information should have been flowing back, we still had a consistency loss on x_0 and the angle representations were different (root_orient in this case)

# HuMoR Problems
Main issues of HuMoR
### VAE
- Missing useful motion data
    - The data sets don't describe some motions that might be important to us, like lying down, and motions like this would mess with our existing method for finding the ground plane
- Simplistic contact model
    - in our current pipeline and in HuMoR
### TestOps
- Ground plane reliance 
    - Used for rescaling of the pose to a canonical reference frame, in which the prior is trained
    - Won't generalise well to many real world situations (we might not care?) where the ground plane cannot be accurately estimated
    - And the method we estimate the ground plane is questionable
- Slow motion generation (TestOps)
    - Optimisation rather than direct prediction, could be a problem
- Does not recover if a single frame is missing a prediction
- Reliance on SMPL?
- Lot's of hand tuned regularisers?
