

- Noise only the non handles
- Give the handles as conditioning
- Loss on how much it changes the conditioned handles

- TODO:
    - Where are the handles input?

    - Network
        - Indicate this model uses handles in config
        - Indicate num handles range to train on
        - In model forward
            - Get handles, use positional encoding to add them to input?
    - Diffusion Noiser
        - Training
            - Randomly sample handles
            - Don't noise handles
            - Add handles to inputs to pass to model 'forward' func
        - Eval:
            - Don't noise handles during initial noising
    - evaluator denoising loop
        - Get handles from inpainting node config
        - Add them to the inputs before the initial noising
    - Loss
        - Penalise if it changes the indicated handles