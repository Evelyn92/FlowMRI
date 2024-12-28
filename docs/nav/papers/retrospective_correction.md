Retrospective correction of head motion using measurements from an electromagnetic tracker

[Paper link](https://europepmc.org/backend/ptpmcrender.fcgi?accid=PMC6824937&blobtype=pdf)

- What sequence?
    - T1-weighted MPRAGE sequence
- What recon?
    
    - each k-space line from each coil -> corrected for phase ramps -> regridded in 3D k-space (NUFFT) with the estimated rigid body motion param as suggested by Gallichan [23]
    - Motion-corrected images were reconed for each coil -> the adaptive combine method was used to create the final image.
- What Electromagnetic tracking data look like?
    
    translation and rotation of each scan..
    
- How do we get the reference image?
    
    - Still images (include involuntary movements)
- What is the protocol?
    
    - Phantom experiments: a pineapple -> 8 times(after movements) \* 45s
    - Volunteer experiments: 6 volunteers, each volunteer -> 4 volumes
        
        - still -> used as no motion reference image
        - four movements (random position)
        - nodding up down
        - continuous head nodding: up-down, down-up every 30 seconds (not explained too much)
    - Patient scans: eight pediatric patients (Not explain too much)
        
    - Metrics:
        
        - normalized root-mean-square error (NRMSE) and the structural similarity (SSIM) index
            
            (using FSL tool BET [26]) relative to this motion-free reference image.
            
        - a reference-free metric, average edge strength (AES) was calculated for each scan
            
            As suggested by Aksoy et al. [27], AES values were normalized by the corresponding slice in the “no motion” dataset.
            
- Limitation: only in-plane motion, but inability of correction for though-plane motion in 2D sequence.