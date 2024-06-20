# Introduction to AHRS

## Overview

AHRS stands for Attitude and Heading Reference System. It uses an IMU (Inertial Measurement Unit) with 3-axis MEMS (Micro-Electro-Mechanical Systems) inertial sensors to 
measure angular rate (gyroscopes), acceleration (accelerometers), and Earth's magnetic field (magnetometers). 

These sensors provide data on angular rates, acceleration, and magnetic field strength, respectively. An embedded Extended Kalman Filter (EKF) processes these measurements to estimate the object's 
attitude (orientation in space) and heading (direction of travel).


### Definitions

- **Heading**: The direction a vehicle's nose is pointing relative to true north (or magnetic north).
- **Attitude**: The orientation of a vehicle relative to a reference frame, typically the Earth, described by roll, pitch, and yaw angles.

### Rotational Degrees of Freedom
![**Axes of rotation for Roll, Pitch, and Yaw.**](https://www.google.com/imgres?q=roll%20pitch%20yaw%20x%20y%20z&imgurl=https%3A%2F%2Fwww.linearmotiontips.com%2Fwp-content%2Fuploads%2F2020%2F05%2FRoll-Pitch-Yaw.jpg&imgrefurl=https%3A%2F%2Fwww.linearmotiontips.com%2Fmotion-basics-how-to-define-roll-pitch-and-yaw-for-linear-systems%2F&docid=niPaiKV8724PuM&tbnid=7cn-F8quC2XTFM&vet=12ahUKEwiskZ6Xg-qGAxWs-TgGHTpHDLQQM3oECFAQAA..i&w=560&h=492&hcb=2&ved=2ahUKEwiskZ6Xg-qGAxWs-TgGHTpHDLQQM3oECFAQAA)

- **Roll**: Rotation about the longitudinal axis (ranges from -180° to +180°).
- **Pitch**: Rotation about the lateral axis (ranges from -90° to +90°).
- **Yaw**: Rotation about the vertical axis (ranges from -180° to +180°).

## Importance

AHRS data is crucial for navigation and control in systems requiring precise orientation information, such as aircraft, drones, UAVs, missiles, and marine vehicles. 
It ensures accurate positioning, stable operation, and reliable maneuvering.

## Sensor Roles

- **Accelerometer**: Measures gravity vector’s projection along the 3 axes, used for pitch and roll determination.
- **Magnetometer**: Measures Earth's magnetic field, used for yaw determination.
- **Gyroscope**: Measures angular velocity along the 3 axes, integrated to obtain attitude.

## Limitations and Solutions

- **Accelerometer and Magnetometer**: These sensors are susceptible to errors from external accelerations (e.g., vehicle movements) and magnetic disturbances (e.g., nearby
- ferrous objects). These interferences can lead to inaccurate readings.
- **Gyroscope**: Over time, gyroscopes suffer from bias drift, where their readings gradually deviate from the true value. The EKF corrects this drift by fusing data from
- all sensors, ensuring accurate orientation estimates.
  
## Methods of Integration

### Euler Integration

Euler Integration first-order numerical procedure for solving ODEs, approximating a curve by its tangent over small intervals. Euler integration in AHRS uses gyroscope data 
to obtain Euler angles (pitch, roll, and yaw). 
However, it can lead to gimbal lock, a condition where two rotational axes align, causing a loss of one degree of freedom, and making it impossible to determine the 
vehicle's complete orientation.
Gimbal lock occurs when Pitch reaches either limits - ±90 degrees. Gimbal lock is a problem attributed to the sequential nature of Euler integration rotations.

### Quaternions

Quaternions can be thought of as 3D equivalents of complex numbers.

Quaternions are composed of two components: the scalar and the vector.
When using quaternions to represent a rotation, the axis of rotation (V) and the effective angle of rotation (⍺) about this axis are encoded in the vector and scalar part of 
the quaternion respectively.

Quaternions provide a robust solution for 3D orientation tracking, avoiding gimbal lock. They represent rotations as a whole rather than sequentially, allowing smooth 
interpolation between orientations. Quaternions are more numerically stable and suitable for applications requiring reliable orientation tracking.

#### Quaternion Integration

Quaternion Integration involves integrating quaternions to maintain a continuous and accurate representation of the vehicle's orientation. This method is more stable and avoids the pitfalls of 
gimbal lock.They are integrated to obtain angular position, converting the pure quaternion of angular velocity into Euler angles.

## Complementary Filter
A Complementary Filter is a technique that combines the outputs from different sensors to compensate for their individual limitations. By weighting gyroscope data against 
accelerometer and magnetometer readings, it provides a stable orientation estimate. This filter helps reduce the bias drift in gyroscopes and the noise in accelerometer and
magnetometer readings.
It essentially stabilizes the results of the AHRS.

## Implementation Details

### SBG Ellipse N Specifications

- 0.05° Roll and Pitch (RTK)
- 0.2° Heading (RTK high dynamics)
- 5 cm Real-time Heave
- 1 cm RTK GNSS Position
- Raw Data for Post-Processing
- High-quality sensors with internal FIR filters and coning & sculling integrals for vibration rejection
- Internal EKF for robust RTK position during GNSS outages

### Test Setup
The SBG Ellipse N is connected to a computer via an RS422-USB cable. The SBG Inertial SDK is used to conduct tests and collect data. During testing, the module is 
maneuvered to cover a full range of roll, pitch, and yaw values, ensuring comprehensive performance evaluation.

### Data Collection

- **Outputs**: Integrated Quaternion (q0, q1, q2, q3), Euler Angles (Roll, Pitch, Yaw)
- **Inputs**: Raw sensor data from accelerometers, gyroscopes, and magnetometers

## Algorithms

Three possible algorithms are implemented and compared to determine the best one.

### Euler Integration Algorithm

This algorithm uses gyroscope data to estimate Euler angles by approximating orientation changes over time. While simple and suitable for initial orientation estimation, it is 
unreliable for complex maneuvers due to the risk of gimbal lock.

### Quaternion Integration Algorithm

This algorithm provides continuous and accurate orientation tracking by integrating quaternions. It is more accurate than Euler integration and avoids gimbal lock. Bias compensation is
achieved by low-pass filtering of gyroscope data to estimate the atrificially injected biases, enhancing the stability and accuracy of the results.

### Complementary Filter Algorithm

Here, combines gyroscope data with accelerometer and magnetometer readings to stabilize the outputs. By weighting the data from these sensors, the complementary filter 
reduces the impact of gyroscope bias drift and enhances the reliability of orientation estimates.

However, for this result to be acheived, accelerometer and magnetometer readings must be as noiseless as possible, in order to be a reliable counterweight for the 
gyroscope's bias drift.
If this is not guaranteed, implementation of a Complementary Filter will instead worsen the results obtained using Quaternion Integration.

## Results and Conclusions

- **Euler Integration**: Adequate for simple maneuvers but unreliable for complex maneuvers due to gimbal lock.
- **Quaternion Integration with Bias Compensation**: Provides the most accurate results, stabilizing the final attitude readings.
- **Complementary Filter**: Effective in offsetting gyroscope bias drift but noisy accelerometer/magnetometer data can distort results.

## Figures

1. Axes of rotation for Roll, Pitch, and Yaw.
2. Pitch, Roll, Yaw for an aircraft.
3. Graphical Representation of Euler Integration.
4. Gimbal Lock condition.
5. Euler Angles to Quaternion Conversion.
6. Direction Cosine Matrix (DCM).
7. Quaternion to Euler angle conversion using DCM.
8. Calculation of quaternion from angular velocity.
9. Ports on the SBG Ellipse N, connecting to SBG Center, and initialization.
10. Algorithm flowcharts for Euler Integration, Quaternion Integration without filters, and with complementary filters.
11. Comparison graphs of actual and calculated Euler angles.
12. Error injection and results with and without complementary filter.
13. Bias compensation results, low pass filtration of gyroscope data.
14. Error comparison graphs.

## Conclusion

AHRS, utilizing an IMU with gyroscopes, accelerometers, and magnetometers, provides essential data for navigation and control in various applications. By employing 
advanced algorithms like quaternion integration and complementary filters, AHRS systems achieve accurate and stable orientation tracking, overcoming the limitations of 
individual sensors.
