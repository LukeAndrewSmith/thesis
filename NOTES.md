# Some general notes

## Humor TestOps
- Split video sequence up into overlapping 3s clips
- fit_rgb_demo_no_split.py and fit_rgb_demo_use_split.py seem to take a similar amount of time, on the order of 20mins...
    - Should time them both to check
    - Actually I think the batch size in this case is 2s (60frames at 30Hz) hence 2 batches with 10frame overlap doesn't change much I presume...
    - Trying much smaller batch
- TODO: 
    - Check the gpu is actually being used properly
    - Understand the overlap_loss better
    - Understand the rollout
    - Finish reading the run_fitting.py file


## Common paper elements (should probably have in my final thesis)
- Dataset section
- Ablation studies
    - Removing parts of the architecture to see what has the biggest impact
- Latent space investigation
    - Visualisation
    - Investigation of how semantically similar things are related in latent space
- Explicit VAE comparison
- Different dataset comparison
    - c.f PoseEstimationBenchmarking for potentially useful datasets
    - Augmentation technique comparison also would be useful


## Training VAEs:
- Posterior collapse
    - Decoder ignores latent variable and overfits to the training sequences
    - MotionVAE: input latent variable at each stage of the decoder
    - Weighting of $\beta$-VAE
- Quality vs. generalisation
    - Depends on weighting of KL and Reconstructions Losses
    - MotionVAE find empirically that: $\textit{MotionVAE reconstruction and KL losses being within one order of magnitude of eachother is a good proxy to balance}$
- Stable sequence prediction
    - Not sure we care so much as we probably won't be extrapolating new sequences
    - MAVE: Scheduled sampling during training - model progressively learns to deal with it's own predictions