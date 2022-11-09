


What data do we have to train on?
- Their method relies on the SMPL body model for training and regularisation
    - They also show that the SMPL regulariser is an important component in the evaluation section
- Their state contains root position, joint angles, joint locations, and importantly the **velocities** of the former, does our data have this?

Do we care about the speed of the system?
- Their main use case is, given a time series of 2D/3D joint locations, to optimise and fit an initial state and sequence of transformations that describe the motion, which is a slow process as it's not a direct prediction. But it does seem similar to our desired usecase.

Good formulation of the exact goal of this project:
- What about the current system needs improvement? What are the failure cases we are trying to improve upon?


Main issues of HuMoR (and a little on our method) (should be useful in determining the future directions of the work)
- Ground plane reliance 
    - Used for rescaling of the pose to a canonical reference frame, in which the prior is trained
    - Won't generalise well to many real world situations (we might not care?) where the ground plane cannot be accurately estimated
    - And the method we estimate the ground plane is questionable
- Missing useful motion data
    - The data sets don't describe some motions that might be important to us, like lying down, and motions like this would mess with our existing method for finding the ground plane
- Simplistic contact model
    - in our current pipeline and in HuMoR
- Slow motion generation (TestOps)
    - Optimisation rather than direct prediction, could be a problem
Not sure if good:
- Reliance on SMPL?
- Lot's of hand tuned regularisers?

Main benefits of humor
- Distribution over change in state
- Simple model: CVAE
- Nice state modelling 
    - Joint positions etc. + their velocities. The velocities nicely encode a context of the current state (they serve as a memory of the previous states)