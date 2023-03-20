# TODO
- **Next training**
    - Normalisation
        - Specify elementwise
    - State
        - Add velocities, see if it improves amount of movement predicted

- **Implementation**
    - Sequence prediction
        - Constraint on beginning/end of each subsequence
    - Evaluation metrics + pipeline
        - Need some better way of comparing the models
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
    - Losses
        - Foot sliding (3d)
            - Ask Jakob/Dominic about getting foot contact predictions
    - DDIM
- **Experiments**
    - ***Inpainting: randomly subsample and remove 30%, 50%, 80% of joints data***
    - Dataset 
        - For testing generalisation
        - Occlusions, other situations we care about
        - Ask Jakob:
            - Easiest way to run our videos in the common room through the model to get some 3d predictions we can try and clean
    - What sequence length is best
- **Other**
    - Jakobs comment
        - If state contains joint orientations **and** positions
            - This is redundant, but helps predictions
            - In diffusion, we should probably only diffuse orientations, as we want the same noise on orientations and positions, then fk forward to get positions
    - Why do we sometimes get such a big change at the end? betas should be small?
        - e.g noise 100, frame 400, take11
        - **ANSWER**: predicted x_0 seems to occasionally drastically change just at the end

- Useful unseen data:
    - Sitting on chair:
        - mg_anim_take2 - frame 2400

**Questions/Answers**
- Papers
    - What frame are they in?
        - root local: cf https://openaccess.thecvf.com//content/CVPR2022/papers/Guo_Generating_Diverse_and_Natural_3D_Human_Motions_From_Text_CVPR_2022_paper.pdf
    - How do they deal with root?
        - EDGE
            - just has a root translation in the state, alongside orientations/binary foot contacts
        - MDM
            - **TODO** not entirely sure,  doesn't seem to have an explicit root?? but that wouldn't make sese