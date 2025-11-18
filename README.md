# Nano-GICP (ROS 2 / Colcon Version)

This repository contains a ROS 2-compatible version of **Nano-GICP**, integrating:

* **Fast-GICP** (efficient GPU-ready GICP formulation)
* **Nano-FLANN** (header-only FLANN fork for fast nearest-neighbor queries)

This fork has been adapted to build as an **ament package** and to be used as a dependency in other ROS 2 projects using **colcon**.

Original upstream source (DLO / DLIO):
[https://github.com/vectr-ucla/direct_lidar_odometry](https://github.com/vectr-ucla/direct_lidar_odometry)

---

## 1. Features

* Fully C++ implementation (header + compiled core library)
* Covariance-based GICP registration
* Multi-threading via OpenMP
* ROS 2 compatible (`ament_cmake`)
* Exported as a shared library for integration in other packages

---

## 2. Dependencies

The package requires:

| Dependency | Version                           |
| ---------- | --------------------------------- |
| C++        | ≥ C++17 (internally set to C++20) |
| CMake      | ≥ 3.10                            |
| PCL        | ≥ 1.8                             |
| Eigen      | ≥ 3.3.7                           |
| OpenMP     | ≥ 4.5                             |
| ROS 2      | Humble / Iron / Jazzy             |

All dependencies are resolved automatically via:

```cmake
find_package(nano_gicp REQUIRED)
```

---

## 3. Building

Clone into your ROS 2 workspace:

```bash
cd ~/ros2_ws/src
git clone https://github.com/<YOUR_FORK>/nano_gicp.git
cd ~/ros2_ws
colcon build --packages-select nano_gicp
source install/setup.bash
```

---

## 4. Using Nano-GICP Inside a ROS 2 Package

In your `CMakeLists.txt`:

```cmake
find_package(nano_gicp REQUIRED)

ament_target_dependencies(your_target
  ...
  nano_gicp
)
```

No manual include / link flags are needed.
`nano_gicp` automatically exports:

* include paths
* NanoFLANN backend
* PCL / Eigen / OpenMP linkage

### Example minimal usage

```cpp
#include <nano_gicp/point_type_nano_gicp.hpp>
#include <nano_gicp/nano_gicp.hpp>

using PointT = pcl::PointXYZI;
nano_gicp::NanoGICP<PointT, PointT> gicp;

// configuration
gicp.setMaxCorrespondenceDistance(1.0);
gicp.setNumThreads(8);
gicp.setCorrespondenceRandomness(20);

// data
pcl::PointCloud<PointT>::Ptr src(new pcl::PointCloud<PointT>());
pcl::PointCloud<PointT>::Ptr dst(new pcl::PointCloud<PointT>());
pcl::PointCloud<PointT> aligned;

*src = source_cloud;
*dst = target_cloud;

// alignment
gicp.setInputSource(src);
gicp.calculateSourceCovariances();
gicp.setInputTarget(dst);
gicp.calculateTargetCovariances();
gicp.align(aligned);

// result
if (gicp.hasConverged()) {
    double score = gicp.getFitnessScore();
    Eigen::Matrix4d pose = gicp.getFinalTransformation().cast<double>();
}
```

---

## 5. Package Structure (ROS 2 Edition)

```
nano_gicp
 ├── include/nano_gicp/*.hpp
 ├── src/nano_gicp.cc
 ├── src/nanoflann.cc
 ├── CMakeLists.txt   <-- ament export
 └── package.xml      <-- ROS2 metadata
```

The library exports two installable targets:

```
nanoflann     (internal dependency)
nano_gicp     (main library)
```

They are automatically available after `find_package(nano_gicp)`.

---

## 6. Reference Integration Example

A full working ROS 2 package using this library:

[https://github.com/engcang/FAST-LIMO-SAM-QN](https://github.com/engcang/FAST-LIMO-SAM-QN) (ROS1 original)

Converted ROS 2 example (excerpt):

```cmake
ament_target_dependencies(fast_limo_core
  rclcpp
  sensor_msgs
  pcl_conversions
  nano_gicp
)
```

---

## 7. License

This project is released under the MIT License.

---

## 8. Acknowledgements

* **FastGICP**
  Koide et al., “Voxelized GICP for Fast and Accurate 3D Point Cloud Registration,” ICRA 2021

* **NanoFLANN**
  Blanco & Rai, “NanoFLANN: a C++ Header-Only FLANN Fork,” 2014

* **DLO / DLIO**
  Chen et al., “Direct LiDAR Odometry: Fast Localization With Dense Point Clouds,” RA-L 2022
