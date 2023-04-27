# TODO
## For Jakob
What I did while he was away:
- Contacts
    - Got contacts estimation working
    - Trained new model
- Evaluation
    - Evaluated models + made notes
- Autocomplete
    - Implemented handles conditioning
    - Trained model

High level notes:
- Root translation issues
- Might

Questions:
- How to avoid duplicating pipelines??
- Double check I'm using the contacts correctly
    - Contacts velocity loss
        - Using ground truth contacts
    - Predicted contacts
        - Not sure what it actually helps
- Need better evaluation
    - Inpainting random
        - Not actually realistic as we are
            1. keeping the root translation
            1. randomly removing orientations, but we should be somehow randomly removing positions (but the models don't explicitly have positions...) as in a real case we won't have just some orientations, rather missing positions
                - I beleive
        - Need to find a better test for randomly missing occlusions
    - Denoising
        - Noising it too far... not comparable to the actual situation we'd encounter

- Their main 'noise':
    1. Structural
        - Flips etc
    1. Confidence based
        - Could modift the inpainting to mix what we know with certain confidence
- New evaluation
    - Denoising
        - Try with less noise
    
- Future work section for thesis
    - Designing the model to be more task specific
        - Inbetweening: timestep embedding saying how long till completion
        - General inpainting: conditioning saying what is missing and what is not
    - More test cases
        - Motion switching
            - Inbetweening between separate motion clips (karate to sitting)

## TODO
- refactor
    - rename {normalize/denormalize}_all to {}_all_features to make it more explicit

- Afternoon
    - Continue trying to visualise the training pipeline for the stuck root
        - Check the pipeline is matching the training one
        - Currently: need to move the camera frame, not sure where the gt is

- Try to get the baseline running with the contacts
    - i.e without all the root to body frame stuff
    - Like the first version that was working, just with contacts loss as well
- Inbetweening
    - Visualisation
        - Try to visualise the output of the model at the final stage
            - What we are currently visualising is the output of the model mixed with the ground truth one last time
        - For inbetweening visualisation, can keyframe colors so as to have different colors for the inbetweened parts
    - Try run with training data to see if it looks similar
    - Try with 1 frame on each side
    - Notes: how to improve
        - Postprocessing
            - Mix the edge frames into the inbetweened motion
            - Have a 'slurp node' to do this, checks for the final iteration
- Note on a bug
    - There was a bug with the calculated velocity losses before, I was doing a finite difference along the batch dimensions
        - I'm surprised the model managed to learn something...
    - Also all the trainings were missing orientations fk loss...
- Random inpainting
    - Pipelines for random inpainting with replace_random_mask_each_denoise_step=FALSE
- Refactor
    - Some of the parameters I put in json are bools but I used strings, jsons accept bools though... 
- Evaluate
    - earlier models
        - root__orient__vel__full_data/1000_denoise_steps__epoch_150
        - baseline 150 (rerun)
    - elementwise autocomplete
- Autocomplete
    - Evaluate autocomplete model
- Contacts model
    - Try figure out what's going wrong
    - Overfit on a single sequence
    - Visualise contacts prediction
    - Issues
        - Wrong frame rates
        - Jumpy contacts
        - Problem with loss? Feet are now avoiding the ground
- Think
    - Why root is bad, and positions noise is so high?
    - How to improve inpainting random (cf above...)
    - Why eval goes up during training
- Contacts
    - Improve contacts prediction
    - Visualise contacts during eval
    - Evaluate model
- Evaluation
    - Evaluate some earlier models to see if there is a difference where it looks like it's overfitting
- Positions noise (Autocomplete)
    - Seems a bit extreme (I think it's just because the normalisation was minmax not elementwise)
    - See how the papers do it, what data they have, normalisation etc..


## High level goals
- Questions to adresss
    - Q: Is a viable model for various goals?
        - It is worthwhile exploring them from a general animation perspective (for the thesis), and to show the work in a good light, discussing all the possibilites
        - Goals
            - Tracking & Denoising (like for humor)
            - Inpainting, inbetweening, autocomplete
            - Pros/Cons/What can we use it for
    - Q: Why is it more interesting than humor? (why we explored it) (hypothesis)
        - More possible avenues
        - More easily encorporate what we know
- Focus on evaluation
    - How the baseline is performing
        - Get a baseline, fix all the issues
        - Then try improve
- Make sure the tests are informative
    - Evaluate for specific scenarios
        - e.g if we have an occluded sequence: 'we can handle 20% '
        - e.g 'we can inbetween 50 frames'
            - Good for tracking (longer inbetweening) or just inbetweening animation (shorter sequences)?

## TODOs
- **Motion**
    - Look into how much noise on positions
        - There seems to be too much
        - See what noise the papers add
    - Evaluation
        - Evaluate 
        - Evaluate earlier models in the training
        - Try inpainting random with with less joints, see if it works first
        - Eval metrics
        - Questions
            - When mode collapse?:   Try earlier trained models to see if we have more diversity
            - Sequence length?:   Try predict much longer sequences (200) than we trained on and see where the motion fails
    - Low priority
        - 2d inpainting for our data
            - run_synth2track.py only2d flag, give video path
            - Gives world space
            - Might need to crop around motion to emulate the training data
        - Documentation
            - Document how to train

- **Pose Auocomplete**
    - Next:
        - Try with just poses (p) or just orientations (q)
        - Goal: see if we get variablility, worry about controlling it later
        - ISSUE: training with just orient doesn't work, root missing?
    - Other
        - Always have the last 10 steps in the denoising, so it can converge better
        - Current issue
            - Handles aren't close enough, therefore the blending completely messes it up as the handles don't match
            - Try with more handles
        - It's still not completely taking into account the handles
            - New model:
                - mask the noising step (condition) to noise everything but the handles
                - 3-4 handles up to all handles
                - Punish the system for modifying the handles
                - Condition the system on which handles to keep
        - Visualisation
            - Diversity
                - Visualise 99s
                - 3d
                - Join them together
    - Notes
        - Hypothesis for why training on positions and orientations jointly doesn't work
            - Noise is so much larger on p that the system just learns to ignore it


- **Implementation**
    - Avoid duplicating the piplines
    - Evaluation metrics
        - Task specific
            - Denoising
                - Accuracy: denoised vs noised version
            - Missing joint inpainting
                - Diversity
                - Multi-modality
        - Common metrics
            - c.f https://arxiv.org/pdf/2007.15240.pdf for definitions
            - Diversity
            - Rest might only be for text conditioned generation
                - FID (Frechet Inception Distance)
                - Multi-modality
                - Recognition accuracy
                - KID?
    - DDIM?
- **Experiments**
    - Dataset (For later for 3d, currently don't have much root translation in training data so wouldn't work well)
        - **Would work for 2d**
        - For testing generalisation
        - Occlusions, other situations we care about
        - Ask Jakob:
            - Easiest way to run our videos in the common room through the model to get some 3d predictions we can try and clean
            - Single camera script
    - What sequence length is best
        - Try with 200 frames
- **Other**
    - Jakobs comment
        - If state contains joint orientations **and** positions
            - This is redundant, but helps predictions
            - In diffusion, we should probably only diffuse orientations, as we want the same noise on orientations and positions, then fk forward to get positions
    - Why do we sometimes get such a big change at the end? betas should be small?
        - e.g noise 100, frame 400, take11a
        - **ANSWER**: predicted x_0 seems to occasionally drastically change just at the end

- Useful unseen data:
    - mg_anim_take6
    - mg_anim_take2
        - Sitting on chair - frame 2400
- Other test data (I was using before)
    - mg_anim_take11a

**NOTES**
- Careful
    - Predicting longer sequences than the sequence length trained on results in no motion being prediction
        - I haven't check if it's still predicted in the beginning, or just that in the visualiser which is visualising the middle frames and can't see the first frames only shows the part of the predicted sequence with no motion

**Questions/Answers**
- Papers
    - What frame are they in?
        - root local: cf https://openaccess.thecvf.com//content/CVPR2022/papers/Guo_Generating_Diverse_and_Natural_3D_Human_Motions_From_Text_CVPR_2022_paper.pdf
    - How do they deal with root?
        - EDGE
            - just has a root translation in the state, alongside orientations/binary foot contacts
        - MDM
            - **TODO** not entirely sure,  doesn't seem to have an explicit root?? but that wouldn't make sese