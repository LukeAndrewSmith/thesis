


- What I need
    - Directory contaianing optimisation fits, intermediate results, intermediate losses, for example videos
    - qualitative.py takes this and outputs visualisations
        

- File structure
    - out/$DATA_NAME/results_out
        - Contains the data for each video in subdirectories
            - Check it contains, intermediate results, losses, etc.
    - out/$DATA_NAME/qualitative
        - Contains the visualisation outputs for all the videos


- What I need to understand to do this
    - What is 16687732431 in Scene05_0005_16687732431?
        - MotionOptimiser splits into sequences?
    - Should the example videos be just iMapper?
        - If not we need a custom data loader that handles a number of RGB videos, rather than just one





    

- What we actally want now
    - Run the model on a bunch of videos we find interesting, try to break the system
    - Have the existing pipeline as a comparison

- Later
    - 