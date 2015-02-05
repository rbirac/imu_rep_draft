REP: XXX
Title: Standard Topics and Conventions for IMUs
Author: Paul Bovbel <pbovbel@clearpathrobotics.com>
Status: Draft
Type: Informational
Content-Type: text/x-rst
Created: 02-Feb-2015
Post-History: 02-Feb-2015


Abstract
========

This REP defines common topics, namespaces, and data output conventions for data provider (drivers) of sensors in the Inertial Measurement Unit (IMU) family. This includes accelerometers, gyroscopes, magnetomers, and any combination thereof.

Specification
============

Frame Conventions
-----------------

An IMU device measures data with respect to two frames, specified by the manufacturer:

* The **body frame** represents the internal device axes. These may be found either in the device specification documents, and sometimes printed directly on the device body. This frame is fixed to the device orientation.
  
* The **world frame** represents the external reference frame for the device. This frame is typically aligned either ENU (east north up) or NED (north east down), and the relevant conventions from REP 103 [1]_ apply.

* If the device does not have an absolute yaw reference (magnetometer), the world frame is only aligned with an external reference along the `z` axis. The `x` and `y` axes are aligned relative to the power-on position of the sensor.

* If the device does not have an absolute gravity reference (accelerometer), the world frame is not aligned to any external reference and instead aligned relative to the power-on position of the sensor.

* The `frame_id` for all message types published by an IMU represents the body frame - the default frame ID for IMUs is `imu_link`. In compliance with REP 0103 [1]_, and as a hint to integrators, the default frame name for IMUs that report in NED should be `imu_link_ned`.

  - The configuration of the body frame relative to other frames (e.g. `base_link`) represents the mounting position of the IMU [2]_.

  - The world frame definition is implied in the configuration of the body frame, and is not explicitly defined in the transform tree.

* The device has an associated *neutral orientation*, defined as the orientation of the device where the body and the world frame align [3]_.

Data Reporting
--------------

* To maintain interoperability with ROS conventions, all frames must be right handed.

* All data should be published by the driver as it is reported by the device. Any modifications to the data (e.g. filtering, transformations) should be delegated to a downstream consumer of the data [3]_.

  - The first exception to the above is if any data is reported inconsistently with the manufacturer's specified body frame for the device. In this case, the driver should transform as necessary to keep the reference frame consistent across all data sources.

  - The second exception to the above is if any data is reported left handed - it may be converted to right handed by the driver by inverting the `y` axis.

* A prominent note should be made in the driver documentation regarding any internal data manipulation that does not comply with the device manufacturer's specification.

Raw Data
''''''''

* Accelerometers

  - The accelerometers report linear acceleration data in the body frame of the device. The data takes the form of a 3D vector, with the components representing the deflection of the internal accelerometers. 

  - When the device is at rest, the vector will represent the deflection solely due to gravity, and will always point 'up' away from the earth's gravitational center.

* Gyroscopes

  - The gyroscopes report rotational velocity data in the body frame of the device. The data takes the form of a 3D vector, with the components representing velocity around each equivalent axis of the body frame.

  - The rotational velocity is right handed with respect to the axis, and independent of the orientation of the device.


* Magnetometers
  

  - The magnetometers report magnetic field strength in the body frame of the device. The data takes the form of a 3D vector, with the components representing magnetic field strength in each direction.


Filtered Data
'''''''''''''

* Orientation
  
  - The IMU driver implementation may provide a filtered orientation estimate based on a combination of the above sensor sources. This data is in the form of a quaternion, which represents the rotation of the body frame relative to the world frame.

  - In the neutral orientation, the body frame is aligned with the world frame, so the orientation will be the identity quaternion.


Transformation
--------------

Applying a transformation to IMU data requires applying an identical rotation to both the body and the world frames - this implies that no offset will be applied to any world-referenced data (accelerometers, magnetometers and, orientation). In essence, transformed data represents the output of a simulated IMU with the updated body and world frames, and the effect is that NED and ENU IMU data can be easily obtained from the same data source by transforming between the two world frames, regardless of the manufacturer's specification.

Topics
------

The following topics are expected to be common to many devices - an IMU device driver is expected to publish at least one. Note that some of these topics may be also published by support libraries, rather than the base driver implementation. All below message types are supplemented with a std_msgs/Header, containing time and coordinate frame information.


* `imu/data_raw` (sensor_msgs/Imu)

  - Sensor output grouping accelerometer (`linear_acceleration`) and gyroscope (`angular_velocity`) data. 

* `imu/data` (sensor_msgs/Imu)

  - Same as `imu/data_raw`, with an included quaternion orientation estimate (`orientation`).

* `imu/mag` (sensor_msgs/MagneticField)

  - Sensor output containing magnetometer data.


All message types provide a covariance matrix (see REP 103 [1]_) alongside the data field (`*_covariance`). If the data's covariance is unknown, the all elements of the covariance matrix should be set to 0. If a data field is unreported, the first element (`0`) of the covariance matrix should be set to `-1`.

Namespacing
-----------

By convention, IMU output topics are pushed down to a local namespace. The primary source of IMU data for a system is published in the `imu` namespace. Additional sources, such as secondary IMUs or unprocessed raw data should be published in alternative `imu_...` local namespaces. IMU driver implementations should take care to allow convenient remapping of the local namespace through a single remap argument (e.g. imu:=imu_raw), rather than separate remap calls for each topic.

Rationale
=========

This REP seeks to mitigate the variances in manufacturer specification and ROS driver development with regards to IMUs. Following these guidelines to data formatting and representation will aid in creating a consistent interface to the majority of IMU sensors, and avoid the inconvenience of updating ROS message definitions [3]_.

Backwards Compatibility
=======================

It is up to the maintainer of a driver to determine if the driver should be updated to follow this REP.  If a maintainer chooses to update the driver, the current usage should at minimum follow a tick tock pattern where the old usage is deprecated and warns the user, followed by removal of the old usage.  The maintainer may choose to support both standard and custom usage, as well as extend this usage or implement this usage partially depending on the specifics of the driver.

Reference Implementation
========================

A reference implementation for this REP is in development for the CHR-UM6 IMU [4]_ driver, targeting ROS Jade.

References
==========

.. [1] REP-0103 Standard Units of Measure and Coordinate Conventions
   (http://www.ros.org/reps/rep-0103.html)

.. [2] ROS Answers discussion
   (http://answers.ros.org/question/50870/what-frame-is-sensor_msgsimuorientation-relative-to/)

.. [3] ros-sig-drivers discussion
   (https://groups.google.com/forum/#!topic/ros-sig-drivers/Fb4cxdRqjlU)

.. [4] ROS Driver for CHR-UM6
   (http://wiki.ros.org/um6)

Copyright
=========

This document has been placed in the public domain.

..
   Local Variables:
   mode: indented-text
   indent-tabs-mode: nil
   sentence-end-double-space: t
   fill-column: 70
   coding: utf-8
   End:

