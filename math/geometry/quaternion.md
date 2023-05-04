Quaternion
----------

This is largely based on the excellent paper found [here](https://hal.inria.fr/inria-00590039/document)

```lua
swan = require"swan"
```
```output[7](10/24/22 10:39:31)
```

Quaternion multiplication can be represented as a regular 
matrix multiplication.

```lua
swan.clear_syms()

r0 = swan.sym "r0"
rx = swan.sym "rx"
ry = swan.sym "ry"
rz = swan.sym "rz"

Qr = swan.mat {
    { r0, -rx, -ry, -rz },
    { rx, r0, -rz, ry },
    { ry, rz, r0, -rx },
    { rz, -ry, -ry, r0 }
}

q0 = swan.sym "q0"
qx = swan.sym "qx"
qy = swan.sym "qy"
qz = swan.sym "qz"

q = swan.mat {
    { q0 },
    { qx },
    { qy },
    { qz },
}

print((Qr * q):simplify())
```
```output[11](10/24/22 10:43:00)
[
  q0r0 + (-1)qxrx + (-1)qyry + (-1)qzrz
  q0rx + qxr0 + (-1)qyrz + qzry
  q0ry + qxrz + qyr0 + (-1)qzrx
  q0rz + (-1)qxry + (-1)qyry + qzr0
]
```


