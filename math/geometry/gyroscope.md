Gyroscope
=========


Integration
-----------

    
### Integration in tangent space (constant angular velocity)

$$q(t + \Delta t) = \frac 1 2 \omega q(t) \Delta t + q(t)$$

Nice derivation [here](https://fgiesen.wordpress.com/2012/08/24/quaternion-differentiation/)

### Integration on manifold (constant angular velocity)

$$q(t + \Delta t) = q(\Delta t \cdot \omega) q(t)$$

### Clarification on roll-pitch-yaw angular rate vs $\omega$ (axis-angle velocity)

Rate of change for Euler angles (XYZ rotation order) is the same as axis-angle velocity.

Can be confirmed with:

```python
from scipy.spatial.transform import Rotation as R

dt = 1e-4
print(R.from_euler('XYZ', [3*dt, 2*dt, 5*dt]).as_rotvec() / dt)
```

The underlying assumption being that all gyroscope use this very rotation order (although not mentionned anywhere afaik, so could be wrong).

IMU Pre-Integration
-------------------

The derivative of the IMU Preintegration resulting rotation with respect to the gyroscope bias seems linear ?
  => This happens when the tangent space integration (i.e. just summing angle axis) is approximatively equal to the on-manifold integration.
