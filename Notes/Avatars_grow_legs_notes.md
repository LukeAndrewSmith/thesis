# Avatars Grow Legs - META motion diffusion paper

## Idea
Input:
- Sequence of sparse tracking data
    - e.g
        - Orientation/translation of the headset and two hand controller
        - Sequence of N observed joint features
    - But
        - Fixed length signal
- Output:
    - Global orientation of pelvis
        - TODO: what about translation?, p.3 of paper
        - Does global orientation mean 'rotation and position'?
    - Relative rotation of each joint
        - 22 SMPL joints

## Architectures
1. MLP based
    - Direct sparse to complete joints 
    - Claim: 'can acheive comparable performance to sota methods'
    - But has jittering artefacts
    - Notes
        - 12 blocks
        - Latent feature size: N x 256
2. MLP diffusion model
    - Same MLP architecture
    - Now diffused
        - Claim: 'to solve the jittering problem and create smooth motion'
    - Notes
        - Timestep embedding
            - Repeatidly injected before every block of the MLP network
                - Force the MLP to take account of it
            - Timestep projected through a fully connected layer to match the dimension of the others, then added to the intermediate activations 
        - AdamW optimiser

Ablation:
- Network comparisons
    - Transformer
    - The network from 'Avatar pose'

Speed:
- 196 frames diffused (5 DDIM sampling steps) in 35ms on an NVIDIA v100 
    - TODO: DDIM?

- Noted issues
    - Floor penetrations




# Notes for us
- How can we tell the system which joints to denois
    - Does only inpainting work?
- Issue with no conditioning?
    - Could be that we input confidences instead of p's
    - Or
- Baseline
    - No p's
- Others
    - What can we replace p with?
        - Confidences
            - Just randomly add confidences during training
- Or use confidences at inference time
    - Use confidences in inference, as a blending factor for the inpainting


TODO:
- How long does it take to train?
    - They don't say...
- Look at code
    - They don't have code
- Look at other systems
    - Maybe diffusion is not what we want?
        - Think it's worth trying a system that operates over a sequence of poses