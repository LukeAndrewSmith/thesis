
Observation:  Initial z's -> rollout sitting down
Goal:         Acheive this forward prop. of information with optimiser
----------------------------------------------------------------------
Copy over
    - mimics rollout, sits down

# Fix z, optimise x
z taken as the stage 2 z's \
Sequence: ```examples_test_sitting_short``` \
Optimise x's, detach decoded sequences 

    Decode: 1 
        Losses: consistency    
            experiment_1_2
                - Clearly sitting down, but wobbly
                - Moves away from joints
            experiments_1_3
                - Higher learning rate than 0.001 doesn't converge properly

        (Test: keep close to joints)
        Losses: consistency & joints2d
            experiment_2_1
                Losses: consistency (1000x) + joints2d (1x)
                - Bit wobbly (oscillates about 2d joint predictions)
                - Sits down and stays close to joints
                - Strange effect with legs crossing at the end 
                    - Frame 21&22, hips are flipped
                    - Rollout from frame 20 results in legs crossing (experiment_4_2)
            experiment_2_2
                Losses: consistency (500x) + joints2d (1x)
                - Less wobbly
            experiment_2_3
                Losses: consistency (10x) + joints2d (1x)
                - Arm stays down
                - Jitters more, rotates more
                    - Probably due to openpose strange predictions, as now the joints2d loss has more effect

        (Test: see if contacts height loss is working properly)
        Losses: contacts_height
            experiment_5_2
                - Feet forced to ground
                - Doesn't seem to affect the number of contacts until the loss becomes very low, then perhaps begins to overfit and remove contacts
                - Upper body moves madly...?
                    - Another bug?

        (Test: encorporate contacts height to try avoid legs crossing)
        Losses: consistency & joints2d & contacts_height
            experiment_5_2
                Losses: consistency (500x) + joints2d (1x) + contacts_height (100x)
                - Doesn't seem to affect much
                - Leg crossing still occurs
            experiment_5_3
                Losses: consistency (500x) + joints2d (1x) + contacts_height (1000x)
                - TODO: run experiment

        (Test: encorporate pose prior to try avoid legs crossing)
        Losses: consistency & joints2d & pose_prior
            experiment_6_1
                Losses: consistency (500x) + joints2d (1x) + pose_prior (1x)
                - No visible effect
            experiment_6_2
                Losses: consistency (500x) + joints2d (1x) + pose_prior (100x)
                - Too strong, makes the person stay standing the whole time
            experiment_6_3
                Losses: consistency (500x) + joints2d (1x) + pose_prior (10x)
                - **Nice effect, stops the legs crossing**

        (Test: Encorporate contacts height to improve optimiser further)
        Losses: consistency & joints2d & pose_prior & contacts_height
            experiment_7_2
            Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (1x)
            Optimisation variables: x_1+, floor_plane
            - **Nice sitting down**
            - Floor plane is wonky but matches feet well
            - Jittery still, needs consistency

        ((BEST))
        (Test: Encorporate smoothness)
        Losses: consistency & joints2d & pose_prior & contacts_height & smoothness
            experiment_8_1/2
            Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (1x) + smoothness(1x, and 10x)
            Optimisation variables: x_1+, floor_plane
            - No desperately noticable difference
            - **Noticed that lots of the losses seem to converge rather quickly, but not the consistency loss**
                - It seems to take a while for the information to propagate forward\

            experiment_8_3
            Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (1x) + smoothness(100x)
            Optimisation variables: x_1+, floor_plane
            - Much smoother but now doesn't sit down very much
        
        (Conclusions: 
            - We can propagate the information forward from fixed z's and produce a reasonable result
            - The information propagates very slowly forward and the optimisation takes 16mins for the longest...
                - I get this impression as I ran some experiments with less iterations and it didn't manage to sit down properly
            - Slow: experiment_8_2 took 9 minutes
        )

    (Test: Does rolling out a little improve what we did above?)
    Decode: 5 (equal weighting)
        (Test: start same as with decode 1)
        Losses: consistency
            experiment_3_2
                - Sits down nicely
                - Much less wobbly that with only 1 decode
                - Seems to smooth the result
                - Drawback, will be slower with more losses...

        (Test: keep close to evidence)
        Losses: consistency & joints2d
            experiment_4_1
                Losses: consistency (100x) + joints2d (1x)
                - Sits down nicely, stays close to joints
                - Legs crossing effect happens later and less obvious
                - Optimiser reaches convergence more smoothly and stays there
                - Optimisation is slower
    
        ((BEST))
        (Test: try what was most successful with 1 decode, but with 5)
        Losses: consistency & joints2d & pose_prior & contacts_height & smoothness
            experiment_9_1
            Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (1x) + smoothness(10x)
            Optimisation variables: x_1+, floor_plane
            - **Makes it a lot smoother**
            - **Sits down nicely**
            - **Rotates less**
            - Rising arm comes back...
                - Probably due to the fact the weighting has changed
            - **NOTE: the loss weighting will not correspond to the single rollout as all the decoded losses are summed**
                - Probably should mean them, but requires a number of changes to code and I'm hoping I can just show that this method works enough to then move on swiftly...
        - **To try**: higher joints2d and pose prior loss

        (Conclusions:
            - Adding more rollouts seems to help regularise it
            - Extremely slow... experiment_9_1 took 16mins
            - Inconclusive: the weighting doesn't correspond to the decode 1 tests, as we have the same weight for the decoded sequences but now 5 of them, so effectively the losses on the decoded sequences are now proportionally 5x more
            - Thoughts:
                - Try adding more losses to decoded sequence rather than optimised sequence
                - Try increasing speed of information flow using copy over approach
                - Try optimising z's jointly
                - Try on longer sequence
        )

---

(Test: Try a longer sequence)
Sequence: ```examples_test_sitting```
Optimise x's, detach decoded sequences 

    Decode: 1
        (Test: Try with successful mix of losses)
        Loss on optimised sequence
        Losses: consistency & joints2d & pose_prior & contacts_height & smoothness  
            experiment_0_1
                Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (1x) + smoothness(10x)
                - Begins sitting down but legs go through floor
                    - Maybe the information needs more iterations to propagate forward
                - Not so succesful as with other video
            experiment_0_2
                Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (100x) + smoothness(10x)
                - same
            experiment_0_3
                Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (1000x) + smoothness(10x)
                - Feet stick to floor, but they slide all over...
            
        (Test: add the contacts_vel loss, but also try adding the losses on the decoded sequence rather than the optimised sequence)
        (TODO: perhaps changing too much at once, maybe I should try just adding the velocity loss to the previous test)
        Loss on decoded sequence
        Losses: consistency & joints2d & pose_prior & contacts_height & contacts_vel & smoothness  
            experiment_1_1
                Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (1000x) + contacts_vel (1000x) + smoothness(10x)
                - Predicts significantly less contacts
                - Stays standing mostly
            
            experiment_1_1
                Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (1000x) + contacts_vel (10x) + smoothness(10x)
                - Lies down a bit and has the turn rather strongly
                - Not very good


        ****************************************************************
        (Test: what happens if we just optimise z with all the current losses)
        Optimise z
        Losses: consistency & joints2d & pose_prior & contacts_height & contacts_vel & smoothness & latent likelihood
            (Loss on optimised sequence)
            experiment_2_1
                Losses: consistency (500x) + joints2d (1x) + pose_prior (10x) + contacts_height (1000x) + contact_vel (100x) + smoothness(10x) + latent likelihood (10x)
                - Feet sliding all over
                - Sits down, but has rotation quite strongly
                - Arms are reasonable
            
            (Test: loss on decoded sequence and lower consistency)
            (NOTE: not sure why I did this, was too many changes at once)
            experiment_2_2
                Losses: consistency (100x) + joints2d (1x) + pose_prior (10x) + contacts_height (1000x) + contacts_vel (100x) + smoothness(10x) + latent likelihood (1x)
                - Predicting less contacts
                - Feet look very strange

            (Test: loss on decoded sequence and lower consistency)
            experiment_2_3
                Losses: consistency (100x) + joints2d (1x) + pose_prior (10x) + contacts_height (1000x) + contacts_vel (100x) + smoothness(10x)
                - Bouncing on sitting down, and rotating
                - Legs don't bend as much as would be nice


        ***********************************************************
        (Test: Try speed up information flow through copy over method)
        (Test: use blending, when I ran without the joints2d seemed to explode, maybe I should let it run and see though)
        (NOTE: loss on decoded sequence as optimised sequence is now just overriden with decoded sequence)
            experiment_4_1
                Blending: 90/10
                Losses: joints2d (1x) + pose_prior (10x) + contacts_height (1000x) + contact_vel (10x) + smoothness(10x) + latent likelihood (1x) + joints consistency decoded (1x)
                - Legs go through ground
                - I presume not enough forward information propagation

            (Test: increase forward info flow by increasing blending)
            experiment_4_2
                Blending: 50/50
                Losses: same
                - Sits down a lot more, looks more like a squat, like the rollout...
                - Leg is too extreme, foot lifts
                - Not sticking to evidence very well

            experiment_4_3
                Blending: 50/50
                Losses: same, but joints (10x) not (1x)
                - Sticks to evidence a little more
                - Goes funky and the end
                - Look like it's actually trying to walk a little in occlusion
                - TIME: 165 seconds

            (Test: increase latent likelihood and see if the motion become nicer, also copy over only every 3 iterations, in a further attempt to allow time for z to become sensible)
            (NOTE: Maybe shouldn't have changed so many things at once)
            experiment_4_4
                Blending: 50/50
                Losses: same, but joints (10x) and latent (10x)
                Copy frequency: 3
                - Motion become strange
                - Floor plane prediction is wonky
                - Predicts less contacts
                
            (Test: copy over every iteration but try for more iterations, maybe that helps)
            (NOTE: Again maybe not close enough to previous experiment to conclude)
            experiment_4_5
                Blending: 50/50
                Losses: same, but joints (10x) and latent (10x)
                Copy frequency: 1
                - Divergence at end of sequence
                - Wonky floor plane

        experiment_4_6 (same as 4_4 but with copy over freq = 1)
                Blending: 50/50
                Losses: same, but joints (10x) and latent (10x)
                Copy frequency: 1
                - Doesn't stick very close to joints


        ********************************************************
        (Test: what if we don't blend but just copy over)
            experiment_5_1
                Losses: same as 4_4
                Iterations: 100 not 500 like 4_4
                Copy frequency: 1
                - Divergence at end of sequence
                - Floor plane improved
        (NOW: 4_4_ but with 100 its)

        *********************************************************
        Decode: 2
        (Test: Fix z, optimise x, with humor final z's)
        Losses: joints2d  
            experiment_13_1
                Losses: consistency
                - Oscillates a lot
                - Clearly sits down in a manner similar to the z's
            experiment_13_2: 
                Losses: as in "BEST" of 'examples_test_sitting_short' experiments (experiment_8_1/2)
                - Sit's down much better
                - No oscillations
                - Slides when sitting

## Conclusions
It seems that if the rollout is decent (examples_test_sitting_short experiment_8_1/2), we can encorporate the information if we fix the z's and optimise the x's. This is not tooo surprising as we are essentially just slowly pushing the information forward but constraining the information a bit.

When the rollout is worse (examples_test_sitting_short experiment_2_*, c.f experiment_0_0 for rollout) so far haven't managed to keep the x's close to the evidence and use the rollout where we have less evidence.

**TODO**: haven't tried with humor optimised z's

# Fix x, optimise z
x taken as humor optimised x's \
Sequence: ```examples_test_sitting``` \
Optimise z's, detach decoded sequences 

    Decode: 1
        (Test: Start simple)
        Losses: joints2d  
            experiment_12_2
                Losses: joints2d
                - Horrendous, no likelihood loss => z's overfit to matching the next frame

    Decode: 10
        (Test: And with more rollout?)
        Losses: joints2d and likelihood
            experiment_12_3
                Losses: joints2dx1 likelihoodx1
                - Not bad
                - Z's converge consistently to the final humor z's
                - Full rollout is decent even though it is only optimised with small rollouts in mind
    
    Decode: 5
        (Test: slightly less rollout)
        Losses: joints2d and likelihood
            experiment_12_4
                Losses: joints2dx1 likelihoodx1
                - Not bad
                - Z's converge consistently to the final humor z's but seem to plateau before experiment_12_3
                - Full rollout is again worse than 12_3 (not too surprising, if we rollout more we end up closer to theirs)

    Decode: 2
        (Test: slightly less rollout again)
        Losses: joints2d and likelihood
            experiment_12_5
                Losses: joints2dx1 likelihoodx1
                - Not bad
                - Z's converge consistently, seem to go to same place as 12_4
                - Rollout doesn't sit down as much

### Conclusions:
It seems that from 



# Notes
**NOW** 

- In the morning
    - Implement
        - L1 losses
            - To test
        - Average over decoded frames
        - Temporal consistency, try only make it 'forward'
    - Go through experiments again and see what's next to be tried
        - Where would optimising z be sensible
        - Decide direction of experiments and map them out
            - Run and read papers/plan next part of thesis while they're running in the background
        - Try optimising z's with small rollout
        - Look into how their optimiser works again
            - Why doesn't theirs jump around, if they don't have temporal consistency in stage 3
            - Maybe visualise frame evolutions for their method
    - Delete some old stuff, running out of memory



- Things that need improving
    - Turn backward still happening strongly on sit down sequence
        - Maybe look into the confidence on the joints that are being predicted for the occluded part of the body
        - Maybe we should be pruning the joint predictions below a certain confidence
    - Slow
        - Slow optimiser
            - Maybe should profile it and see why it's so slow
        - Slow forward information propagation
            - Maybe the information propagates forward so slowly due to it only propagating through consistency loss which passes it forward through tiny updates
            - Maybe try copy over again, but optimising lots


- Aim: optimise z as well
    - Not clear to me where and when to introduce the optimisation over z's
        - If we introduce it before the rollout has propagated forward 
        - Maybe all the losses should be applied to the decoded sequences and then we just have a loss to make the optimised sequence look like the decoded sequence
            - In this case it feels like a longer rollout would help
    - Think: what is the best path to try and get there?
        - First: try optimise with lots of losses and z, see if it works
        - Is it worth changing the losses to l1 and averaging over short range rollout? Or would it be quicker trying to prove this without bothering, we'd like to move along asap to our own code
            - **Averaging quick fix**: divide the decoded sequence loss weights by the number of rollouts.
    - What video has a large difference between stage2 and stage3, where z might be useful
        - Maybe one of the i3DB ones, the one outside
            - c.f https://geometry.stanford.edu/projects/humor/supp.html section 5. ablation, video around 27 seconds, humor makes it walk nicely
    - Is there a big difference between the starting z and the final z for them?
- Take short video, unoccluded to occluded to unoccluded, with walking motion
- Jakob wanted to look at
    - Stage2 vs stage3 for their videos


- Ask jakob about the torch.mean(reproj_err) in joints2d_loss_op

- Add more losses
    - Refactoring
        - joints2d_loss_op
            - Check what operation were actually performing
            - I think they had a sum of l2 norms, but they sum in one go, and we mean, so maybe we are not even performing a mean of l2s as we don't sum along some dimension before mean, I think we just have a mean of squared values
                - Not sure if this is terrible or not...?
        - L1 losses are more linear the l1, might be easier to see the effect when changing the losses
            - Mixing multipe l2s is often harder to balance
        - Decoded sequence losses
            - Could mean them rather than summing them
    - Add some losses
        - Contacts 
        - Pose prior
        - etc...
    - Try on their videos
        - After we've added some losses
- Other things to try
    - More rollout steps, 10, 20?
        - What happens
    - Higher learning rate
    - Contacts, smoothness

**Notes**
- Legs crossing
    - Hips flip at frames 21&22
    - If we rollout from frame 20, then the legs cross
        - Conclusions: usual unexpected cumulation of errors in rollout
        - Rollout from 20 is taken into account in the optimisation as we are rolling out from every frame  
    - Assumption
        - Flipped hips make slight velocities in with the legs that propagate forward
        - These are not so present if the person is sitting down more as inward velocity of the legs cannot so easily cross the legs
- Arms going up and sometimes snapping back
    - Joints2d not weighted enough?
        - Higher joints2d weight keeps them in place (not too surprising...)
    - But why snapping back?