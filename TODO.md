# TODO

- Reading
    - Find papers related to humor, read and summarize
        - MotionVAE, MEVA papers have good references
    - DeepPhase
        - Get a better understanding of the latent space in DeepPhase
        - Why is phase periodic over a motion sequence (Fig. 3)
- Get HuMoR code downloaded and running

## Potential things to investigate
- HuMoR
    - Why is it so slow?
        - Rollout over the whole sequence (c.f HuMoR appendix)?
    - Can we train a CVAE on plain mocap data rather than AMASS?
- Where is the motion prior best used
    - Optimising on 2D or 3D joint sequences to output more realistic motion
        - Directly optimising in latent space like HuMoR
        - Input sequence of joints, output sequence of joints in the latent space, we are essentially projecting the sequence of joints we found onto the latent space of our model
        - Note: We have more confidence in our 2D joints
    - No direct latent space optimisation (Jakob's idea)
        - Use the latent space as an extra loss term when optimising something else
        - TODO: don't have a clear/concrete idea about this
- Data augmentation
    - Anything extra on top of what is in the MEVA paper?