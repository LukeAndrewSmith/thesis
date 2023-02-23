

# What we've tried
## Preliminary note
Note that the testing was not conclusive or extensive, and the errors might be fixed and the fixes might work better with a more optimal weighting of losses, learning rate, optimiser choice etc. But considering the size of the hyperparameter space extensive testing was not a real option, hence the experiments should be used as helpers for our intuition, as we can definitively say that under the conditions of the experiments the effect we saw was the case, and so it can give us a better understanding of the optimiser and it's issues.

## Optimisation method attempts
*Fundamental optimiser approaches*
- Copy over
    1. 1,2,5,10,20 rollout
        - \>10 rollouts converges (for their example video)
    2. Opt z 50-100 its then let it run
        - Doesn't help
    3. Opt z 10,20 frames then copy over
        - Helps for test video but I feel it can still diverge
    - Notes
        - Errors propagate forward and invalidate the updates in the future of the sequence
- Copy over blend
    - No extra rollout
        - 0.99 old + 0.01 new
            - Doesn't diverge
            - Didn't learn much?
        - 0.9 old + 0.1 new
            - Diverges
    - Notes
        - **To investigate further**
- Fix z's, optimise x's
    - Notes
        - The effect of z's is noticable, but as the z's are simply initialised as the mean of the encoder they don't necessarily create the best next x (mean is not necessarily the correct z to take), therefore the result is jittery
    - **Note to discuss with jakob**
        - Intuition
            - If rolled out, without optimising, we seem to sit down behind the sofa
        - Issue
            - If rolled out, without optimising, we end up accumulating errors, and diverging for all other sequences
            - Difficulty therefore is by increasing the chaining we might jointly improve the sitting and worsen the rest
                - Also it doesn't seem to improve the sitting, I think we got lucky with the point at which the rollout began for the humor example
                - We have no distinct starting point every 60 frames hence we are chaining from further back 
- Optimiser x,z jointly
    - Lot's of weights so not clear what's wrong
    - Notes
        - They seem to 'explain eachother away', the z's are made worse, the decoded x's stop having contacts, etc
        - Maybe because we've weighted everything wrong, testing wasn't extensive

## Issues and fixes
*Less fundamental ideas more little details*
- Bad velocity prediction
    - Issue
        - If we simply constrain the joint positions while optimising over the z's, we note that the velocity predictions (included in the state x) become bad, it seems that sometimes the easiest way for the optimiser to improve the joint location error also makes the velocities worse
    - Fixes
        1. Predict velocities and rollout at least 1 extra step
            - This ask the optimiser to 'find an z that decodes to a new x, and that this x can also decode to another sensible x through the next z', as we have the chain x_t -> z_t -> x_t+1 -> z_t+1 -> x_t+2. Hence the velocities must be sensible as the state must chain well to the sequence.
        2. Loss asking the velocities to be close to the finite difference estimate
            - Didn't seem to help much

- Mismatch between