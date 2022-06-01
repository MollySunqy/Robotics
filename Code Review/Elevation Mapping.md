# Demo@RunTime

## Active Nodes

```
/elevation_mapping
/gazebo
/pcl_manager
/rosout
/rviz
/voxel_grid
/waffle_pose_publisher
/waffle_state_publisher
```

## Active Topics

```
/base_footprint_pose
/camera/depth/camera_info
/camera/depth/image_raw
/camera/depth/points
/camera/depth/points_downsampled
/camera/parameter_descriptions
/camera/parameter_updates
/camera/rgb/camera_info
/camera/rgb/image_raw
/camera/rgb/image_raw/compressed
/camera/rgb/image_raw/compressed/parameter_descriptions
/camera/rgb/image_raw/compressed/parameter_updates
/camera/rgb/image_raw/compressedDepth
/camera/rgb/image_raw/compressedDepth/parameter_descriptions
/camera/rgb/image_raw/compressedDepth/parameter_updates
/camera/rgb/image_raw/theora
/camera/rgb/image_raw/theora/parameter_descriptions
/camera/rgb/image_raw/theora/parameter_updates
/clicked_point
/clock
/cmd_vel
/elevation_mapping/elevation_map
/elevation_mapping/elevation_map_raw
/elevation_mapping/visibility_cleanup_map
/gazebo/link_states
/gazebo/model_states
/gazebo/parameter_descriptions
/gazebo/parameter_updates
/gazebo/performance_metrics
/gazebo/set_link_state
/gazebo/set_model_state
/imu
/initialpose
/joint_states
/move_base_simple/goal
/odom
/pcl_manager/bond
/rosout
/rosout_agg
/scan
/tf
/tf_static
/voxel_grid/parameter_descriptions
/voxel_grid/parameter_updates

```
---
# Step-by-Step

## `turtlesim3_waffle_demo.launch`

Steps:
1. start gazebo and spawn waffle int turtlebot3_house
2. publis tf`s
3. run a passthrough filter to down-sample pointcloud
4. launch **elevation_mapping**


### down-sample pointcloud
#### **`nodelet/voxel_grid`**:
  Input: `"/camera/depth/points"`

  Output: `/camera/depth/points_downsampled"`

### launch elevation_mapping

####  `postprocessing/postprocessor_pipeline.yaml`
Loaded to `elevation_mapping` node. Inpainting(Repairing) holes in the image and compute surface normals.

## Package: `elevation_mapping`

Executable: elevation_mapping_node

```
ros::init(argc, argv, "elevation_mapping");
elevation_mapping::ElevationMapping elevationMap(nodeHandle);
ros::AsyncSpinner spinner(nodeHandle.param("num_callback_threads", 1));  // Use n threads
```

### ElevationMapping
Upon Consturction: readParameters, setupSubscribers, setupServices, setupTimers. initialize.

#### readParameters()
```
bool ros::NodeHandle::param(const std::string & 	param_name,
                                             T & 	param_val,
                                             const T & 	default_val
                                             )		const

```
Parameters

param_name	The key to be searched on the parameter server.

[out]	param_val	Storage for the retrieved value.

default_val	Value to use if the server doesn't contain this parameter.

Returns

true if the parameter was retrieved from the server, false otherwise.

**Parameters are loaded through `.yaml` files**

#### setupSubscribers()
Call `inputSource_` (which is defined in .hpp file, of type InputeSourceManager).
Subscribe to sensor input(`inpute_source`) and robot pose(`robotPoseTopic`)

registered callback: `&ElevationMapping::pointCloudCallback`

The sensor input is defind and managed by InputSourceManager

#### setupService()
Setup services like `clearMap`, `saveMap` and `loadMap`

#### setupTimers()
Timer for `mapUpdate`, `fusedMapPublish` and `visibilityCleanup`.

#### initialize()
Start threads, timers, and initializeElevationMap
To initializeElevationMap: `PlanarFloorInitializer`, start tf, set robot on the map, and height around robot`s current position.


### ElevationMap
Existing topic/service upon construction:
- elevationMapFusedPublisher_: "elevation_map", no subscription
- underlyingMapTopic_: &ElevationMap::underlyingMapCallback
- visibilityCleanupMapPublisher_: "visibility_cleanup_map", unfound

### underlyingMapCallback

Parameter: grid_map_msgs::GridMap& underlyingMap
Steps:
1. convert msg to grid mapping `  grid_map::GridMapRosConverter::fromMessage(underlyingMap, underlyingMap_);` underlyingMap_ is a pointer to a GridMap object.

2. Check if basic layer exists, add missing layers.

3. Set basiclayer layer parameter.
Basic Layer: List of layers from `data_` that are the basic grid map layers. This means that for a cell to be valid, all basic layers need to be valid.

4. Copy data from `underlyingMap_` to `rawMap_`


### Input Management

#### InputSourceManager
Resigter sensers and get topic. Only used upon initialization. Iterate through the list and pass to `Input`

#### Input
Configure one input source.

---
