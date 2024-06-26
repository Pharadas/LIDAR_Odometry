This is a project made to create an implementation of an odometry system for LIDAR systems, in which 'beacons' are simple blobs of retroreflective tape, which are then identified using their positions relative to others in a map created when starting the program.

Before the code can be used for odometry, it is used to create a map of the position of every beacon, which then becomes the reference map.
First it filters the point cloud generated by our LIDAR sensor by intensity, then creates a grid to optimize the point seach and then explores it, creating 'clusters' which can be considered individual beacons, this process is shown with the following diagrams:

![g31935](https://github.com/Pharadas/LIDAR_Odometry/assets/60682906/c0322ee1-0cf3-434e-90ed-cbeae1fa29b4)


Once these beacons are found, the following process is used to determine which beacons are which in our original reference map:
Given the following original positions found for the beacons ($A^A$, $B^A$ and $C^A$), the newly found positions after moving the sensor ($D^L$, $E^L$ and $F^L$) we wish to find a translation and rotation such that the newly found points can be overlapped on the original points:
![g26660](https://github.com/Pharadas/LIDAR_Odometry/assets/60682906/1908cb4d-f106-4a26-8c78-f4cebb109dd9)

Once we have all beacon positions for a given position $p_a$, we can create the following table, where all distances ($d_i$) and beacon positions ($^AA, ^AB, ^AC$) are unique, once we have mapped these positions along with their distances, we can move to a new position $p_l$, where we will still get the same distances, but with new relative beacon positions ($^LD, ^LE, ^LF$), we round every distance to a meter in order to mitigate the noise generated by our LiDAR points

| Distance | Absolute Beacon | Absolute Beacon | Relative Beacon | Relative Beacon |
|----------|-----------------|-----------------|-----------------|-----------------|
| $d_1$    | $^AA$           | $^AB$           | $^LD$           | $^LE$           |
| $d_2$    | $^AC$           | $^AB$           | $^LF$           | $^LE$           |
| $d_3$    | $^AC$           | $^AA$           | $^LF$           | $^LD$           |


Since we don't know which relative position maps to which original beacon position, we can choose any pivot beacon and any other two beacons (we choose the three beacons made up of the most points) to be able to map our relative pivot to our absolute beacons, then we can use our pivot to consider the two distances it makes up with the other two beacons, if we choose $D$ as our pivot we get distances $d_1$ between $^LD$ and $^LE$ and $d_3$ between $^LD$ and $^LF$, however, since the distances between our beacons remains the same as the absolute ones, we can map our pivot to it's original position, simply finding the beacon that repeats in both distances in our original table. In our case, we can conclude that $^AA = ^LD$.

Once we can map one beacon, mapping a second one is trivial, we can take our distance $d_1$ and consider the other beacon in both tables, so we can say that $^AB = ^LE$, with this, we can create two vectors $\vec{^AA^AB}$ and $\vec{DE}$. The difference in angle between these two vectors will be the rotation of the observer at the new position $p_l$, with this we can rotate our pivot beacon to it's position after only the translation, we will call this new position $G$

$$
R=\begin{bmatrix}
cos(\theta) & -sin(\theta) \\
sin(\theta) & cos(\theta) \\
\end{bmatrix}
$$

$cos\theta=\frac{\vec{^AA^AB}\cdot\vec{^LD^LE}}{|\vec{^AA^AB}||\vec{^LD^LE}|}$

$sin\theta=\frac{\vec{^AA^AB}\times\vec{^LD^LE}}{|\vec{^AA^AB}||\vec{^LD^LE}|}$

$\theta=arctan(sin\theta, cos\theta)$

$G = R^LD$

Finally, we can establish that $p_l - p_a = G - ^AA$, but since we consider our origin to be $p_a$, then $p_a = \langle 0,0\rangle$, and our solution becomes $p_l = G - ^AA$.
