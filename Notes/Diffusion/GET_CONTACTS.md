
Take the contact for the RightForeFoot for the FootJoint, as the foot joint is shifted


How to get foot contacts, for each json:
1. "People"
1. Camera object: "BodyFrameRelativeToCamera"
    - Think: camera at zero and this tells you body location
1. Person: "keypoints-location"

What we care about:
- RelToCam data in People
Transforms:
- To world: add WorldTransform

Test 1:
- Render joint positions and see if it's in body frame
    - Root should be pinned, and feet sliding
- Go to world (add camera)
    - Check if the whole body is moving
    - Check the feet are close to the ground
    - Find foot contacts through velocity close to zero and foot close to ground
            - Can chat with mattia about this

- Final data
    - Root in camera frame
    - rest in body frame
    - foot contacts
- Check the other data
    - See what frame they are in
    - Jakob thinks contacts are easier to predict if we are in world frame
        - As in body frame, walking result in feet sliding, whereas in world frame the contacts will be static

- Take st2_original_skeleton
    - Careful, 4 foot joints
        "LeftFoot",
        "LeftForeFoot",
        "LeftToeBase",
        "LeftToeBaseEnd",
    - Add contacts to the jsons
    - Use run_preprocessing.py, with make_temporal_data_summaries flag to make .py 
        - Modify this to calculate and add the contacts during preprocessing
            - This allows sequence level modification, e.g 'make sure contacts sequences are at least 10 long', or other heuristics
