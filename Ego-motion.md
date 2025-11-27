 
 
- **Ego-motion** = how the **camera itself** moved between two frames (e.g., the drone translated/rotated).
    
- If the scene is approximately **flat** (ground plane) and the camera motion between adjacent frames is not extreme, the apparent pixel motion can be approximated by a **2D transform**:
    
    - **Translation** (2 dof): shift only
        
    - **Similarity** (4 dof): rotation + uniform scale + translation
        
    - **Affine** (6 dof): rotation + anisotropic scale + shear + translation
        
    - **Homography** (8 dof): most flexible 2D projective warp for planar scenes
        
- Ht​ is a **homography matrix** (3×3) that maps points from the **current frame** It​ into the **previous (or reference) frame** It−1​:
    
    Pt-1 ~ Ht Pt
    
    (homogeneous coordinates, “∼” means equal up to a scale).

**Stabilization** = using that transform to **re-render the current frame** as if the camera didn’t move (align to a fixed reference). In code that’s a call like:
	- `warpPerspective(current, H_t, ...)` for homography, or
	- `warpAffine(current, A_t, ...)` for affine (robus and cheap, good enough for planar surface)