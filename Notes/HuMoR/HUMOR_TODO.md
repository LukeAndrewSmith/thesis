

# Humor

## TODO

TODO:
- Metting at 10
    - Read paper beforehand
- fix x, optimise z
    - seem to be taking the wrong z, or maybe the rollout isn't working...
    - Jakob says this should be easy for the system find, but I don't agree as we don't have a loss on the decoded vs optimised sequence, hence it will have to learn it from reprojection loss etc which is not obvious 
- fix z, optimise x
    - Check what I've actually done

- Humor
    - Perturbation Experiments
        1. Perturb x's
        1. Fix z's and see if we can get x's
        1. Start with correct x's and take wrong z's, see how much they ruin the x's
        - TODO:
            - Noise each step of the rollout the x that we will roll out from (simulates what happens when updating the x's during the optimisation)
                - Can noise on either the z's or the x's
            - Noise at a sensible value
            - Think
                - How should we be noising, should we noise 
            - Other option
                - Look at difference in the delta the network predicts rather than the actual changed state?

    - Goal:
        - Understand better why our system is not working
        - Intuition: x and z are fighting
            - Try check this
                - Continue perturbation investigation
                - Think: do we just care about the delta or the changed state?
                - Experiment noising at each step seems good
                    - Simulates what happens if we change z in some small way without reference to the updated x's, or vice versa, as is happening in our current optimise as described below
                        - The rollout effect would be less in our case as we blend/optimise rather than copy over, but the information would still propagate forwards
                    - Large deviations would indicate that updating x's and z's separating the losses but at the same time might be too hard, and that they must be fighting
            - Define what fighting is and why it's a problem
                - x,z are both input to the decoder, so we couple them tightly
                - we don't copy over/roll out, so we blend the decoded x with the optimised x, or blend an update
                - we detach z when apply a loss to the optimised x's so information doesn't flow backwards
                    - This then definitely fights, x's are trying to make the z's stay standing rather than sit at the beginning (on test_sitting_short), so the z's produce less contacts and
                        - Experiment: rollout the z's after this fighting has happened to test the hypothesis, check it sit's down less
                - this implies an update to z is decoupled from the update to x when they are optimised together, but as a small deviation in x or z (TO CHECK) can result in a very different decoded x (as seen in in the perturbation experiments), the x and z could therefore be fighting to minimise their own losses rather than working together to jointly minimise the losses
        - If that's the case and they are fighting an iterative scheme would make sense
            - Check if there's hope for an iterative scheme by
                1. Taking their final z's and seeing if we can get sensible x's (mostly done already actually)
                    - Occlusions heavy short, experiment 10
                1. Taking their final x's and seeing if we can get sensible z's

    - Try to go backwards
        - Take their final data, input various bits into our optimiser and see what happens
        - Understand how different their z's are from the final state




- New direction
    - Literature search
        - Think what sort of model we'd like to try
    - Download their code and try to use it
    - Code walkthrough 
        - Choose a time and put in the calendar (probably thursday)
    - Get data
    - Discuss model to implement
- Writing down the experiments we tried
    - Explain the experiments we tried, the thoughts we had, the results we actually got
        - Run the main and interesting ones again to make sure we have the results without bugs
        - Shouldn't be too detailed, it's probably more for us and us in the future, it might not end up in the final thesis

- refactor dict(**{}, **{}) into a func 'combine_dicts'


- Idea:
    1. Optimise z's such that they produce a nice short range rollout FOR THE NON OCCLUDED JOINTS
    1. Optimise x's to match the short range rollout, weighting later decodes heavier to make sure the rollout is having an effect

- Ideas
    - 'Breaks': add barries (x's that aren't copied over to stop the error propagating forward)
    - Only use the information of the z's where we actually need it
        - Ideas
            - Blend only occluded joints
        - I don't see how this would be implemented sensibly

Jakob:
- Step by step keep parts their data and see what ruins it in our method
    - Keep the contacts fixed and see what happens
    - Keep the z's fixed
- To think about:
    - We want the change in x to be mostly from the z
    - Which loss function do we want to affect which optimisation variable?
        - e.g x_consistency, this is moving the z's, but do we actually want this to happen, the main purpose of x_consistency is to pass the information of an improved x forward in the chain so maybe we should flip flop x's and z's
        - Maybe detach z's for the x_consistency loss
- Why is it running out of memory for longer sequences?
    - How heavy is the smpl model

**Priority**
- Losses
    - Setting things high separately to see what they do individually
    - Certain combinations to see what they do
- To check?
    - Optimise over joints directly? or continue using joints3d
    - Are betas handled correctly?
        - Why are stage2 betas so bad for the second sequence? why isn't it just 1 betas per sequence?
- Extra losses
    - Loss stopping any joint being below the floor
        - This might help the contacts when occluded, and therefore help it sit down etc...?
     - Vposer prior?
        - Should we be using VPoser alongside the humor model to ensure the x we are optimising over remains a likely pose, currently nothing is doing this so x could become strange as it's only being constrained by projection and latent motion, and I can imagine these two can interact and diverge from a sensible pose
- Write some more in latex documents
- Humor default method is broken
    - Final results folder not created
        - see call to save_optim_result() in run_fitting.py
    - Results are much worse now

**Other**
- Look into
    - https://cs.yale.edu/homes/che/projects/nemf/
    - Skeletal convolutions: https://sites.google.com/view/hm-vae/home

**On the backburner**
- HUMOR bugs
    - Bug 1: they use the incorrect reference frame when estimating the velocities for x_0 for the initial rollout
        - In original code:
            - Rollout: motion_optimiser.py line 375, vel_trans/vel_root_orient are not actually used in estimate_velocity
            - Infer latent: motion_optimiser.infer_latent_motion line 825, the prior coordinates trans/root_orient are used
    - To check
        - They seem to have some strange logic relating to stage2 results 
        - Check what velocities we are using and predicting and check the constraint makes sense
    - Optimise in 6d not aa
        - Check through losses and see what needs to change
            - The joints in the humor state are not the same as the open pose joints, there are a few (3 which ones?) joints missing and a few extra that aren’t in openpose (neck, left/right hip), hence the reprojection loss uses the smpl regressed joints not the humor state joints. To avoid using the smpl joints (hence avoid converting to aa to get the smpl joints) I could ignore the joints that don’t match and use a subset (22 - 3 = 19 joints, instead of 22 + 3 - 3 = 22 joints for openpose)
        - Try loss on joints instead of joints3d to avoid having to convert to aa for smpl
    - Stage 2 results
        - They seem to save some stage3 results as if they are stage2...
- Investigations
    - Out of distribution investigation
        - How does the prior deal with out of distribution motion
            - Issue, openpose has too many errors with e.g rolling. so we can't evaluate the prior properly
            - **Solution: dataset with labeled 2d joints (or run on amass and visualise just the motion without video)**
    - Quantitative evaluation
        - What videos with ground truth do we actually have?
            - They evaluate on i3DB and PROX, seem to have ground truth 3D that overlaps with smpl
        - **Solution: dataset with labeled 2d joints (or run on amass and visualise just the motion without video)**
- Comprehension
    - Camera intrincs
        - How they are estimated
    - Understand the smpl model better
        - How do they get from their x's to the smpl model