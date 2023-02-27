

TODO: enumerate the choices we could make

TODO: Talk to Dhruv, see what he has seen on diffusion models

- Motion representation
    - Full sequence
        - VAE, diffusion models
            - Idea to deal with variable length sequences (https://arxiv.org/pdf/2106.04004.pdf)
                - Slide a window and take the prediction only of the middle frame
                - e.g project each window onto a latent space, decode, and only take the middle frame, then carry on
                - Not sure how this would work for occluded situations in our case, what if the occlusion is longer that the sequence we operate on
    - Next pose
        - Change in pose
        - Direct next pose
        - Commonly a VAE

- State
    - 2d joints
    - 3d joints
    - Body model parameters (SMPL/STAR)

- Predictions
    - partial 2d -> 3d
    - noisy 3d -> 3d

- Jakobs preference
    - Diffusion model 
    - but perhaps try something quicker first 

- Extra things to predictions
    - Floor contacts
    - Stationary points/joints

- Motion rectifier
    - Direct prediction
        - Not seen any papers try this
    - Diffusion model
        - Conditioned by bad sequence, or sparse sequence
    - Optimisation
        - Rollout
        - Step by step


- General ideas
    - Diffusion rectifier
        - Method/idea
            - Run their system on the whole training set to get sequences to be corrected
            - Diffuse conditioned on their sequences to the ground truth
            - The diffusion then learns to correct their mistakes
        - Problem
            - Fixed length sequences, though we can use the idea described above, sliding window
                - Not sure how well the sliding window idea would work on occluded sequences
    - VAE (HuMoR inspired)
        - Method
            - Simpler state, no SMPL, etc.
            - Aim: make rollout cheap
        - TODO: check if other method's decoders take the previous frame as input
    - VAE (no rollout)
        - Method
            - As in the modified humor we have been running
            - Play with encoder/decoder architectures
                - GNN
                - Skeletal convolutions (https://arxiv.org/pdf/2106.04004.pdf)

- Questions
    - What do we care most about, occluded sequences?


## Notes on repository/code setup
What I've learned from the code so far

- Need to have
    - Good way of keeping track of experiments + trainings
        - Manner of comparing experiment configurations diffs quickly
        - Setup document to track what I've run
    - Have every option in a single config file
        - As a dictionary rather than a list of command line inputs...
    - Save the commit hash for each experiment
        - Never run an experiment without commiting first
    - Monads for loss function
        - Store json of all the losses internally


TO UNDERSTAND:
https://arxiv.org/pdf/2209.14916.pdf
- The model tries to denoise the sequence directly in 1 step
    - For sampling (Figure 2. right), we predict the denoised sequence then add a little noise back and repeat
    - Question: during training do we really build a whole chain each time from T to 0 and let the gradients flow all the way back, or do we, like in the image diffusers, randomly sample a state and just perform an update on that
        - May need to look at code to find the answer
    
- What is ddim sampling?