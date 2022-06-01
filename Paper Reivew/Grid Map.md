# A Universal Grid Map Library: Implementation and Use Case for Rough Terrain Navigation

keywords: multi-layered map, Eigen， legged robot.

## Introduction

Rough terrain navigation need to consider 3D environment. Use gird map as 2.5D
representation. The multiple layers represent elevation, variance, colour,
surface normal, occupancy etc. Package provides ROS message converts: map -> OccupancyGrid, GridCells and PointCloud2

## Overview

### Software Components

`grid_map_core`: core algorithms and classes.

`grid_map`: ROS dependent, interface to convert gird map objects to ROS msg.

`grid_map_msgs`: ROS msg and srv intefaces.

`gird_map_visualizaiton`: Visualization in RViz.

`grid_map_demos`: demos.

`grid_map_filters`: ROS filters for grid map data.

## Use Case: Elevation Mapping

### Background

Assumption for an existing robot pose estimation and onboard range measurement sensor.
To address error caused by sensor and pose estimation, a probabilistic elecation
mapping process is formulated.

**Steps:**

 1. Measuremet Update: Fuse new range sensor measurement with existing data in
 the map by means of a Kalman filter.

 2. Map Update: Update robot moves reflecting estimation error.

 3. Map Fusion: Compute estimated cell heights.

 ### Implementation

 Subscription:

  - `PointCloud2`

  - `PoseWithCovarianceStamped`
