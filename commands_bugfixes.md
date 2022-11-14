

# Commands
## Gpu stuff
 
Info about gpu
```nvidia-smi```   
Make sure the cuda compiler driver is available
```nvcc -V```



## Bugfixes

### LibGL issues
Error: ```pyglet.gl.ContextException: Could not create GL context``` when running ```python humor/fitting/viz_fitting_rgb.py```

Followed 2 suggestion at https://stackoverflow.com/questions/72110384/libgl-error-mesa-loader-failed-to-open-iris in humor miniconda3 env