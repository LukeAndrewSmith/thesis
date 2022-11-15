

# TODO: 
- Did they directly evaluate the VAE?
- Understand the overlap_loss better
- Understand the rollout
    - humor_model.roll_out()
- Finish reading the run_fitting.py file
- Speed testing
    - Try smaller video for test
    - See smaller sequence length
- Make notes on the stages
    - **What is stage III initialisation doing and why is it so bad?**
        - Understand how it works better to see why it's bad?
        - I beleive:
            1. we estimate the states $x_{1:T}$ through the poses we find and through finite differences to estimate the velocities
            2. We project each pair $(x_t, x_{t-1})$ through the encoder to a $z_t$
            3. We rollout $z_{1:T}$
            - Which has no guarantee of globally consistent motion as we are stacking locally consistent latent vatriables
    - Try the example run_fitting.py on an occluded video and see what the stage 2/3 looks like 
    - Make some pseudocode out of what they're doing
- Think of how to evaluate changes in the optimisation sequence length
    - We'd like to know if relying on overlap loss and having a small sequence length significantly reduces the quality of the predictions
```python
TODO: pseudocode
```


## Humor TestOps
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



## Thoughts
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