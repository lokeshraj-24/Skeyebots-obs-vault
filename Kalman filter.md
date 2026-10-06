

Position vector cannot be used, gotta use angular kalman tracker

basic equation:

xt = F * xt-1
F - State transition matrix
x - Position matrix

for angular,
State matrix (X):
[azimuth, elevation, azimuth_vel, elevation_vel]

MOTION MODEL (Constant Velocity):
================================= 
azimuth(t+1) = azimuth(t) + velocity_az(t) * dt 
elevation(t+1) = elevation(t) + velocity_el(t) * dt 
velocity_az(t+1) = velocity_az(t) # Assumed constant 
velocity_el(t+1) = velocity_el(t)


In matrix form: 
$$
\begin{vmatrix}
az \\
el \\
V_az \\
V_el 
\end{vmatrix}
=
\begin{vmatrix}
1 & 0 & dt & 0 \\
0 & 1 & 0 & dt \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{vmatrix}
\begin{vmatrix}
az \\
el \\
V_az \\
V_el 
\end{vmatrix}
$$
X = F Xt-1



Important Matrices:

| **Variable** | **Name**                 | **Simple Meaning**           | **Tuning Guide**                                                                                                            |
| ------------ | ------------------------ | ---------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **R**        | **Sensor Noise**         | "How blurry is my camera?"   | **High R:** Output is smooth/slow (Trusts Physics).<br><br>  <br><br>**Low R:** Output is jittery/reactive (Trusts Sensor). |
| **Q**        | **Process Noise**        | "How chaotic is the object?" | **High Q:** Filter reacts fast to turns.<br><br>  <br><br>**Low Q:** Filter expects smooth, straight motion.                |
| **P**        | **Estimate Uncertainty** | "How lost am I right now?"   | Starts high (unsure). Shrinks as the filter locks on.                                                                       |
### 4. The Mathematics (Simplified)

**State Vector (x):** What we are tracking.

**Step A: Predict (Physics)**

1. **New State:** x=F.x
    
    - _Physics moves the object forward._
        
2. **New Uncertainty:** P=F.P.F<sup>T</sup>+Q
    
    - _Uncertainty grows because we haven't checked the sensor yet._
        

**Step B: Update (Correction)**

1. **Innovation (y):** y=z−Hx^
    
    - _Difference between Sensor (z) and Prediction (x^)._
        
    - **CRITICAL:** For Angular KF, perform `(y + 180) % 360 - 180` here.
        
2. **Kalman Gain (K):** K=PH<sup>T</sup>(HPH<sup>T</sup>+R)<sup>−1</sup>
	    
    - _The "Trust Factor". High R makes K small._
        
3. **Final State:** x=x^+K⋅y
    
    - _Nudge the guess towards the sensor measurement._



