Gyroscope
=========


Integration
-----------

    
### Integration in tangent space (constant velocity)

$$q(t + \Delta t) = \frac 1 2 \omega q(t) + q(t)$$

Nice derivation [here](https://fgiesen.wordpress.com/2012/08/24/quaternion-differentiation/)

### Integration on manifold (constant velocity)

$$q(t + \Delta t) = q(\Delta t \cdot \omega) q(t)$$

### Clarification on roll-pitch-yaw angular rate vs $\omega$ (axis-angle velocity)

Rate of change for Euler angles (XYZ axis convention) is the same as axis-angle velocity.

Can be confirmed with:

```python
from scipy.spatial.transform import Rotation as R

dt = 1e-4
print(R.from_euler('XYZ', [3*dt, 2*dt, 5*dt]).as_rotvec() / dt)
```

The underlying assumption being that all gyroscope use this very axis convention (although not mentionned here afaik, so could be wrong).
