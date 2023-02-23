# Unexpected gradients investigation
In the case of the first frames of ```occlusions_heavy.mov```, where there is only a left arm and left eye/nose being predicted by OpenPose the legs are being moved. This investigation aims at finding out why.

## Blend Shapes investigation
*Question: does removing blend shapes fix the spurious connections* \
*Answer: not quite*

* Below are results for the frame with only the left arm and head predictions, they show the gradients of the pose parameter, we would expect only the left arm pose parameters and their parents to have gradients, but many of them do. Not that the gradients are on 6d representations, so every group of 6 numbers represents the gradients for one joint.

Pose blend shapes: On \
Shape blend shapes: On

joints2d
```
all grads:
- left arm: tensor([[ 1.2154e+01, -1.3776e+00,  9.8066e+00,  5.9842e-01,  1.2704e+00,
  2.6562e+00,  2.3021e+00, -3.3919e+00,  2.7818e+00,  8.1626e+00,
  1.9428e+00,  3.3287e+00, -6.4479e-03,  3.6462e-03, -5.6378e-02,
  -1.0387e-03,  5.0147e-04, -8.8709e-03]], device='cuda:0')
- other: tensor([[-4.6163e-03, -3.8900e-02,  6.5268e-03,  6.8417e-04,  9.3344e-03,
  -4.7304e-02,  2.3312e-03, -3.0055e-02,  9.8306e-03,  2.1907e-03,
  -8.9546e-03,  4.4131e-02,  2.5453e-01, -9.5338e+00,  4.7897e+00,
  -2.6575e-01,  1.2580e+00,  4.2990e+00,  3.6896e-03, -3.2831e-02,
  4.1266e-03, -3.2163e-04,  3.6017e-03,  1.1995e-02,  3.3795e-03,
  2.4724e-02,  4.1190e-02,  5.3288e-04,  2.7943e-03,  8.3953e-03,
  2.9268e-02, -1.1105e+01,  5.2576e+00,  5.0747e-01,  6.7858e-01,
  7.7293e+00,  9.5455e-03,  2.2024e-02, -7.6872e-02, -4.6296e-03,
  -7.0823e-03,  3.8492e-02, -7.8336e-04, -1.1734e-02, -2.8843e-03,
  1.3553e-03, -8.3568e-03,  5.1970e-02,  8.9997e-02, -1.2838e+01,
  4.7114e+00,  1.6268e-01, -3.2830e-01,  7.7562e+00,  5.1337e-04,
  6.0335e-03,  6.7200e-02, -2.2313e-04, -8.7249e-04, -3.1764e-02,
  9.3948e-04, -7.5160e-02, -9.3304e-02, -1.1141e-04, -6.9457e-04,
  3.9916e-02,  7.1226e-01, -1.3250e+00,  2.8889e+00,  1.3115e+00,
  6.3757e-01,  4.6065e+00,  6.2626e+00, -9.8180e+00,  5.9575e+00,
  5.5584e-01,  4.0004e-01,  2.0845e+00,  4.3263e-03,  9.8020e-03,
  -3.0897e-05, -5.3523e-03,  2.8421e-03,  1.8902e-02,  1.9835e+00,
  7.4513e-01,  4.0170e+00,  8.2852e-01,  3.6241e-01,  1.6563e+00,
  -2.7315e-04,  3.9560e-02,  1.2276e-01,  7.4716e-03, -1.5981e-02,
  -6.6196e-02,  9.8557e-03,  1.1015e-02, -1.2899e-02, -5.6227e-03,
  1.9712e-04,  4.6341e-03]], device='cuda:0')
mean grads:
- left arm: 2.2314231395721436
- other: 0.26848435401916504
```

Pose blend shapes: Off \
Shape blend shapes: On

```
joints2d
all grads:
- left arm: tensor([[12.2084, -1.3776,  9.8605,  0.5701,  1.2102,  2.5304,  2.3055, -3.3321,
  2.7556,  8.2274,  1.9583,  3.3552,  0.0000,  0.0000,  0.0000,  0.0000,
  0.0000,  0.0000]], device='cuda:0')
- other: tensor([[ 2.2116e-03,  1.7734e-02,  8.5902e-03,  4.5712e-04,  6.2367e-03,
  -3.1606e-02,  4.2505e-03, -5.6934e-02,  1.4064e-02,  2.9300e-03,
  -1.1977e-02,  5.9024e-02,  4.0183e-01, -1.2817e+01,  5.1717e+00,
  -4.3837e-01,  2.0752e+00,  7.0917e+00,  3.4402e-03, -3.0489e-02,
  5.8912e-03, -1.1856e-03,  1.3277e-02,  4.4218e-02, -7.9354e-04,
  -1.0821e-02, -1.0393e-03,  7.7715e-04,  4.0752e-03,  1.2244e-02,
  -9.7233e-03, -1.3614e+01,  5.7246e+00,  6.4547e-01,  8.6310e-01,
  9.8311e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  1.0079e-01, -1.5066e+01,
  5.2993e+00,  1.9746e-01, -3.9848e-01,  9.4141e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  8.1985e-01, -2.0115e+00,  3.4380e+00,  1.1581e+00,
  5.6297e-01,  4.0675e+00,  6.2506e+00, -9.8050e+00,  5.9389e+00,
  5.7753e-01,  4.1565e-01,  2.1659e+00, -1.0311e-02, -1.9925e-02,
  4.4034e-03, -9.0569e-03,  4.8093e-03,  3.1985e-02,  2.3308e+00,
  1.0068e+00,  4.6650e+00,  7.3361e-01,  3.2090e-01,  1.4666e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00]], device='cuda:0')
mean grads:
- left arm: 2.2373263835906982
- other: 0.2807067036628723
```

Pose blend shapes: Off
Shape blend shapes: On

```
joints2d
all grads:
- left arm: tensor([[  8.9247,  -1.5072,   6.3960,   0.8754,   1.8584,   3.8857, -10.9380,
  -6.1432,  -2.8047,   6.9102,   1.6447,   2.8180,   0.0000,   0.0000,
    0.0000,   0.0000,   0.0000,   0.0000]], device='cuda:0')
- other: tensor([[-1.1732e-02, -9.4833e-02, -3.5752e-02, -1.9758e-03, -2.6956e-02,
  1.3661e-01,  1.4930e-02, -1.6429e-01,  1.1397e-01,  2.5495e-02,
  -1.0421e-01,  5.1358e-01,  4.7347e+00, -8.7838e+01, -6.6262e+00,
  -2.7062e+00,  1.2810e+01,  4.3778e+01,  1.8870e-02, -1.6801e-01,
  1.9403e-02, -6.5466e-03,  7.3312e-02,  2.4416e-01, -8.5182e-04,
  -1.1627e-02, -1.0964e-03,  8.0072e-04,  4.1988e-03,  1.2615e-02,
  -2.0976e+00, -7.4925e+01, -8.0810e-01,  2.1878e+00,  2.9255e+00,
  3.3322e+01,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00, -9.6930e-02, -6.9568e+01,
  -2.3029e+00,  4.5097e-01, -9.1008e-01,  2.1501e+01,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00, -4.1384e+00, -2.1599e+01, -9.9967e+00, -8.9087e+00,
  -4.3307e+00, -3.1290e+01,  7.5046e+00, -1.0201e+01,  9.0521e+00,
  1.1765e+00,  8.4671e-01,  4.4120e+00, -1.7585e-02, -3.2602e-02,
  9.2487e-03, -1.6622e-02,  8.8265e-03,  5.8702e-02, -1.1330e+00,
  9.9107e-01, -2.8926e+00, -8.3831e+00, -3.6669e+00, -1.6759e+01,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,  0.0000e+00,
  0.0000e+00,  0.0000e+00]], device='cuda:0')
mean grads:
- left arm: 0.6622113585472107
- other: -2.205143928527832
```

## Joints
*Question: find out where all the joints come from, and if there are spurious connections* \
*Answer: yes there are strange connections in the blend weights causing the gradients to flow to pose parameters we wouldn't expect*

### Where do the joints come from
J_Regressor:
- 52 joints are regressed, below the number of vertices that are used to regress each joint is shown:
```
(J_Regressor > 0).sum(axis=1)):
[ 22,  12,  10,   8,  10,  10,  11,   8,   7,  10,   5,   3,  10,  11, 11,  11,   7,   9,   7,   8,   9,   9, 154,  93,  98, 179,  99, 100, 138,  97,  99, 196,  94,  97, 133, 123, 100, 154,  93,  98, 179,  99, 100, 138,  97,  99, 196,  94,  97, 133, 123, 100]
from body_models.py SMPLH.J_regressor
```
* -> Not as sparse as you might expect for the later joints

Extra joints:
* There are some extra joints taken for the op joints by directly selecting vertices on the posed mesh (mesh has 6890 vertices)
```
extra_joints_idxs = [ 332, 6260, 2800, 4071,  583, 3216, 3226, 3387, 6617, 6624, 6787, 2746, 2319, 2445, 2556, 2673, 6191, 5782, 5905, 6016, 6133] 
(from vertex_joints_selector.py forward() called from body_models.py SMPLH.forward()
names: https://github.com/vchoutas/smplx/blob/main/smplx/vertex_ids.py
```

Final joints vector: 
* Source: ```smplx.body_models.SMPLH.forward``` returns ```joints```
* shape: ```[73, 3]```
  * regressed: ```0:52```
    * regressed (sparse): ```0:22``` 
      * ```sparse := (J_Regressor > 0).sum(axis=1)) < 90```)
    * regressed (less sparse): ```22:52```
  * vertices: ```52:74```
* smpl joint -> op joints map: ```[ 52, 12, 17, 19, 21, 16, 18, 20, 0, 2, 5, 8, 1, 4, 7, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62] (utils.py) ```
  * i.e the joints we select from the 73x3 vector that we take as OpenPose joints
* Number of joints from each category:
  * regressed (sparse): ```14```
  * regressed (less sparse): ```1```
  * vertices: ```10```
* OP joints names for joints that are just vertices:
  ```['REye', 'LEye', 'REar', 'LEar', 'LBigToe', 'LSmallToe', 'LHeel, 'RBigToe', 'RSmallToe', 'RHeel']```
  * These joints are taken after the lbs has been applied
  * But there is no error on them in our problem case, so they shouldn't affect our results
    * *But turns out they do due to bad blend weights, c.f below*

Fitting loss:
* Some joints are ignore in the loss function
* ```ignore_op_joints: [1, 9, 12]```
  * ```op_map[ignore_op_joints]: [12, 2, 1]```
    * These index the 73x3 smpl joints
    * so we ignore some of the sparse regressed joints in the reprojection loss in ```fitting_loss.py``` 


Joints with non-zero confidence in our failure case:
* Failure case
  * We were investigating the first frames of the heavy occluded video, where there is only a left are and some head predicted
* Non zero confidence joints
  * ```
    OP_JOINTS_NAMES[torch.where(torch.squeeze(joints2d_obs_conf[0]) > 0)[0].cpu().numpy()]: 
    ['Nose', 'LShoulder', 'LElbow', 'LWrist', 'LEye', 'LEar']
    ```


OP with non-zero confidence investigation:
  - Method
    - Set all but each non zero confidence joint to 0 for each joint
    - Look at non zero grads
  - Results:
  ```
  OP name (joint source) -> list(SMPL names for pose params with non zero gradients)
  Nose (vertex) --> ['leftUpLeg' 'rightUpLeg' 'spine' 'leftLeg' 'rightLeg' 'spine1' 'spine2' 'neck' 'head']  
  Neck (regressed - sparse) --> []        
  RShoulder (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'rightShoulder']        
  RElbow (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'rightShoulder' 'rightArm']        
  RWrist (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'rightShoulder' 'rightArm' 'rightForeArm']        
  LShoulder (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'leftShoulder']        
  LElbow (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'leftShoulder']        
  LWrist (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'leftShoulder']        
  MidHip (regressed - sparse) --> []        
  RHip (regressed - sparse) --> []        
  RKnee (regressed - sparse) --> ['rightUpLeg']        
  RAnkle (regressed - sparse) --> ['rightUpLeg' 'rightLeg']        
  LHip (regressed - sparse) --> []        
  LKnee (regressed - sparse) --> ['leftUpLeg']        
  LAnkle (regressed - sparse) --> ['leftUpLeg' 'leftLeg']        
  REye (vertex) --> ['leftUpLeg' 'rightUpLeg' 'spine' 'leftLeg' 'spine1' 'spine2' 'neck' 'head']        
  LEye (vertex) --> ['leftUpLeg' 'rightUpLeg' 'spine' 'leftLeg' 'spine1' 'spine2' 'neck' 'head']        
  REar (vertex) --> ['spine' 'spine1' 'spine2' 'neck' 'leftShoulder' 'head']        
  LEar (vertex) --> []        
  LBigToe (vertex) --> ['leftUpLeg' 'rightUpLeg' 'leftLeg' 'rightLeg' 'leftFoot' 'rightFoot' 'leftToeBase' 'rightToeBase']        
  LSmallToe (vertex) --> ['leftUpLeg' 'spine' 'leftLeg' 'spine1' 'leftFoot' 'spine2' 'leftToeBase' 'rightShoulder' 'rightArm']        
  LHeel (vertex) --> ['leftUpLeg' 'leftLeg' 'leftFoot' 'leftToeBase']        
  RBigToe (vertex) --> ['leftUpLeg' 'rightUpLeg' 'spine' 'leftLeg' 'rightLeg' 'spine1' 'leftFoot' 'rightFoot' 'leftToeBase' 'rightToeBase']        
  RSmallToe (vertex) --> ['leftUpLeg' 'rightUpLeg' 'leftLeg' 'rightLeg' 'rightFoot' 'rightToeBase']        
  RHeel (vertex) --> ['rightUpLeg' 'spine' 'rightLeg' 'spine1' 'rightFoot' 'spine2' 'rightToeBase' 'neck']
  ```

Extra joints non-zero lbs_weights investigation
  - Method
    - Look at each extra joint and see what weights are non zero
  - Results:
  ```
      Vertex name --> list(Non zero lbs weights for joints)
      nose --> ['rightUpLeg' 'leftLeg' 'rightLeg' 'head']
      reye --> ['rightUpLeg' 'leftLeg' 'neck' 'head']
      leye --> ['leftUpLeg' 'rightUpLeg' 'leftLeg' 'head']
      rear --> ['hips' 'spine2' 'leftShoulder' 'head']
      lear --> ['rightUpLeg' 'leftLeg' 'rightShoulder' 'head']
      LBigToe --> ['hips' 'leftFoot' 'leftToeBase' 'rightToeBase']
      LSmallToe --> ['leftLeg' 'leftFoot' 'leftToeBase' 'rightArm']
      LHeel --> ['leftUpLeg' 'leftLeg' 'leftFoot' 'leftToeBase']
      RBigToe --> ['spine1' 'rightFoot' 'leftToeBase' 'rightToeBase']
      RSmallToe --> ['leftLeg' 'rightLeg' 'rightFoot' 'rightToeBase']
      RHeel --> ['rightLeg' 'rightFoot' 'rightToeBase' 'neck']
  ```
  - Data sources
    - Vertex names: https://github.com/vchoutas/smplx/blob/main/smplx/vertex_ids.py
    - File: smplx body_model.py, line 734 +
  - Solution
    - Prune everything less than some tolerance (1e-2 chosen)
    - Results
    ```
    Extra Joints
    332: nose --> ['head']
    6260: reye --> ['head']
    2800: leye --> ['head']
    4071: rear --> ['head']
    583: lear --> ['head']
    3216: LBigToe --> ['leftFoot' 'leftToeBase']
    3226: LSmallToe --> ['leftFoot' 'leftToeBase']
    3387: LHeel --> ['leftLeg' 'leftFoot' 'leftToeBase']
    6617: RBigToe --> ['rightFoot' 'rightToeBase']
    6624: RSmallToe --> ['rightFoot' 'rightToeBase']
    6787: RHeel --> ['rightLeg' 'rightFoot' 'rightToeBase']
    2746: lthumb --> ['lthumb2']
    2319: lindex --> ['lindex2']
    2445: lmiddle --> ['lmiddle2']
    2556: lring --> ['lring2']
    2673: lpinky --> ['lpinky2']
    6191: rthumb --> ['rthumb2']
    5782: rindex --> ['rindex2']
    5905: rmiddle --> ['rmiddle2']
    6016: rring --> ['rring2']
    6133: rpinky --> ['rpinky2']
    ```
  - Non-zero pose gradients after fix (long video frame 702):
    ```
    Nose (vertex) --> ['spine' 'spine1' 'spine2' 'neck' 'head']        
    Neck (regressed - sparse) --> []        
    RShoulder (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'rightShoulder']        
    RElbow (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'rightShoulder' 'rightArm']        
    RWrist (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'rightShoulder' 'rightArm' 'rightForeArm']        
    LShoulder (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'leftShoulder']        
    LElbow (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'leftShoulder']        
    LWrist (regressed - sparse) --> ['spine' 'spine1' 'spine2' 'leftShoulder']        
    MidHip (regressed - sparse) --> []        
    RHip (regressed - sparse) --> []        
    RKnee (regressed - sparse) --> ['rightUpLeg']        
    RAnkle (regressed - sparse) --> ['rightUpLeg' 'rightLeg']        
    LHip (regressed - sparse) --> []        
    LKnee (regressed - sparse) --> ['leftUpLeg']        
    LAnkle (regressed - sparse) --> ['leftUpLeg' 'leftLeg']        
    REye (vertex) --> ['spine' 'spine1' 'spine2' 'neck' 'head']        
    LEye (vertex) --> ['spine' 'spine1' 'spine2' 'neck' 'head']        
    REar (vertex) --> ['spine' 'spine1' 'spine2' 'neck' 'head']        
    LEar (vertex) --> []        
    LBigToe (vertex) --> ['leftUpLeg' 'leftLeg' 'leftFoot' 'leftToeBase']        
    LSmallToe (vertex) --> ['leftUpLeg' 'leftLeg' 'leftFoot' 'leftToeBase']        
    LHeel (vertex) --> ['leftUpLeg' 'leftLeg' 'leftFoot' 'leftToeBase']        
    RBigToe (vertex) --> ['rightUpLeg' 'rightLeg' 'rightFoot' 'rightToeBase']        
    RSmallToe (vertex) --> ['rightUpLeg' 'rightLeg' 'rightFoot' 'rightToeBase']        
    RHeel (vertex) --> ['rightUpLeg' 'rightLeg' 'rightFoot' 'rightToeBase']
    ```

  * The fix requires the following code in the smpl package
    *  ```smplx.body_models.SMPL``` (fixing the lbs weights)
    ```python
    def __init__(...):
      ...
      lbs_weights = self.prune_lbs_weights(
            to_tensor(to_np(data_struct.weights), dtype=dtype)
      )
       ...

    def prune_lbs_weights(self, lbs_weights, tolerance=1e-2):
        # Clean the weights by removing the weights that are very close to zero
        lt_idxs = torch.where(
            torch.logical_and(lbs_weights < tolerance, lbs_weights != 0.0)
        )
        lbs_weights[lt_idxs] = 0.0
        lbs_weights_normalised = torch.nn.functional.normalize(lbs_weights, p=1, dim=1)
        return lbs_weights_normalised
    ```
    * ```smplx.lbs``` (deactivating the blend shapes)
    ```python
    def lbs(...)
      ...  
      # v_shaped = v_template + blend_shapes(betas, shapedirs)
      v_shaped = v_template.expand((betas.shape[0], -1, -1))
      ...
      # v_posed = pose_offsets + v_shaped
      v_posed = v_shaped
    ```





### Side note

SMPL skinning:
  - T pose
  - Shape blend shape
  - Regress joints
  - Pose blend shape
  - Apply transformation to vertices and joints
  - Select extra joints directly as vertices (nose, toes, heels, etc.) to match OpenPose


SMPL parents:
```
  form: parent -> child:
  0 hip is root
  1 hips -> leftUpLeg
  2 hips -> rightUpLeg
  3 hips -> spine
  4 leftUpLeg -> leftLeg
  5 rightUpLeg -> rightLeg
  6 spine -> spine1
  7 leftLeg -> leftFoot
  8 rightLeg -> rightFoot
  9 spine1 -> spine2
  10 leftFoot -> leftToeBase
  11 rightFoot -> rightToeBase
  12 spine2 -> neck
  13 neck -> leftShoulder (SMPL parents in SMPLH have spine2 -> leftShoulder)
  14 neck -> rightShoulder  (SMPL parents in SMPLH have spine2 -> rightShoulder)
  15 neck -> head
  16 leftShoulder -> leftArm
  17 rightShoulder -> rightArm
  18 leftArm -> leftForeArm
  19 rightArm -> rightForeArm
  20 leftForeArm -> leftHand
  21 rightForeArm -> rightHand
```

- Chain of discovery
  - Reprojection loss only for image with only left arm openpose joint predictions resulted legs and other limbs moving
  - Noticed unexpected gradients in the body pose parameters
  - Found spurious correlations in the pose PCA blend shapes (known issue), but some limbs still moved when we commented out both pose and shape blend shapes
      - This fixed the issues for the regressed then posed joints, but the issue persisted for some others.
  - Dug deeper and found that the lbs weights contain unexpected correlations for some vertices, for example the nose is skinned with weights on the order of e-3,e-4 by the legs... This combined with the fact that 10 or more vertices (nose, toes, heels, etc.) are taken directly as vertex points AFTER having been posed (to match OpenPose joints) thus the nose loss would flow back through to the leg pose parameters. 
      SMPL_JOINTS_NAMES[torch.where(results['W'][0,332] != 0)[0].cpu().numpy()]
      array(['rightUpLeg', 'leftLeg', 'rightLeg', 'head'], dtype='<U13')