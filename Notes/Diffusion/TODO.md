# TODO

# Hand in
[Guidelines](https://cgl.ethz.ch/Downloads/StudentProjects/BAMA_Guidelines_2021.pdf)

- Code
    - To ZIP
        - data folder
        - some models
        - some evaluations
- Hand in (to supervisor)
    - Print 2 copies of the report [here](https://thesisbuddy.ch/studentendruckerei.html)
        - Hardcover Bindung (black) + Klebebindung mit Textilband
        - Can request 40chf from the CGL secretary after hand in
    - USB stick
        - software (source code, executables, libraries)
        - documentation (Latex or FrameMaker documents including pdf files and images)
        - oral presentations (PowerPoint slides including all video material)

# Code TODOs
- Note in readme
    - Most important future work
        - Evaluation on in the wild data
        - Comparison of generated data to dataset (is it just replicating the data)
        - Foot sliding for the contact data
        - Metrics
- Morning
    - Collect results to put in the thesis/presentation
        - Inpainting a joint over the whole sequence
        - inbetweening - coloration to show what's inbetweened
        - etc...
    - Inpainting with 1 frame on each side for Jakob

- Train models
    - Without contacts
    - Without sticking the orientations but sticking translation
    - With stuck root and contacts
    - No stuck root again, but with foot sliding loss in world frame...
    - Try stuck root only sticking position and not orientations
        - Should be easier
    - Train without contacts as well to see if that is breaking it


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