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

![**Axes of rotation for Roll, Pitch, and Yaw.**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Roll_Pitch_Yaw.jpeg)

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

Euler Integration first-order numerical procedure for solving ODEs, approximating a curve by its tangent over small intervals. 

![**Graphical Representation of Euler Integration**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Graphical_Representation_of_Euler_Integration.png)

Euler integration in AHRS uses gyroscope data to obtain Euler angles (pitch, roll, and yaw). 
However, it can lead to gimbal lock, a condition where two rotational axes align, causing a loss of one degree of freedom, and making it impossible to determine the 
vehicle's complete orientation.

![**Gimbal Lock condition**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Gimbal_Lock_condition.png)

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

## Quaternion Integration with Complementary Filter
A Complementary Filter is a technique that combines the outputs from different sensors to compensate for their individual limitations. By weighting gyroscope data against 
accelerometer and magnetometer readings, it provides a stable orientation estimate. This filter helps reduce the bias drift in gyroscopes and the noise in accelerometer and
magnetometer readings.
It essentially stabilizes the results of the AHRS.

## Implementation Details

### SBG Ellipse N Specifications

![**SBG Ellipse N**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/SBG_Ellipse_N.jpg)

- 0.05° Roll and Pitch (RTK)
- 0.2° Heading (RTK high dynamics)
- 5 cm Real-time Heave
- 1 cm RTK GNSS Position
- Raw Data for Post-Processing
- High-quality sensors with internal FIR filters and coning & sculling integrals for vibration rejection
- Internal EKF for robust RTK position during GNSS outages

### Test Setup

![**Ports of the SBG Ellipse N**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/SBG_EllipseN_Ports.jpg)

The SBG Ellipse N is connected to a computer via an RS422-USB cable. The SBG Inertial SDK is used to conduct tests and collect data. During testing, the module is 
maneuvered to cover a full range of roll, pitch, and yaw values, ensuring comprehensive performance evaluation.

### Data Collection

- **Outputs**: Integrated Quaternion (q0, q1, q2, q3), Euler Angles (Roll, Pitch, Yaw)
- **Inputs**: Raw sensor data from accelerometers, gyroscopes, and magnetometers

## Algorithms

Three possible algorithms are implemented and compared to determine the best one.

### Euler Integration Algorithm

![**Euler Integration Algorithm**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Algorithm_EI.jpg)

This algorithm uses gyroscope data to estimate Euler angles by approximating orientation changes over time. While simple and suitable for initial orientation estimation, it is 
unreliable for complex maneuvers due to the risk of gimbal lock.

### Quaternion Integration Algorithm

![**Quaternion Integration Algorithm**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Algorithm_QI.jpg)

This algorithm provides continuous and accurate orientation tracking by integrating quaternions. It is more accurate than Euler integration and avoids gimbal lock. Bias compensation is
achieved by low-pass filtering of gyroscope data to estimate the atrificially injected biases, enhancing the stability and accuracy of the results.

### Quaternion Integration with Complementary Filter Algorithm

![**Quaternion Integration with Complementary Filter Algorithm**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Algorithm_QI_CF.jpg)

Here, combines gyroscope data with accelerometer and magnetometer readings to stabilize the outputs. By weighting the data from these sensors, the complementary filter 
reduces the impact of gyroscope bias drift and enhances the reliability of orientation estimates.

However, for this result to be acheived, accelerometer and magnetometer readings must be as noiseless as possible, in order to be a reliable counterweight for the 
gyroscope's bias drift.
If this is not guaranteed, implementation of a Complementary Filter will instead worsen the results obtained using Quaternion Integration.

## Results and Conclusions

### Euler Integration

![**Results of Euler Integration for Simple Maneuvers**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Result_EI_SM.jpg)

**Adequacy for Simple Maneuvers:**
Euler integration is a straightforward method for calculating the orientation of an object using gyroscope data. It works well for simple and short-duration maneuvers where 
the risk of gimbal lock is minimal. 

![**Results of Euler Integration for Complex Maneuvers**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Result_EI_CM.jpg)

**Limitations for Complex Maneuvers:**
However, Euler integration has significant limitations when it comes to complex or extended maneuvers. The primary issue is gimbal lock, a phenomenon where the axes of rotation align, resulting in a loss of one degree of freedom. This alignment makes it impossible to correctly determine the object's orientation because the system can't distinguish between certain rotational positions. This limitation means that while Euler integration might provide quick and easy calculations for basic tasks, it is unreliable for applications requiring continuous and accurate orientation tracking.

### Quaternion Integration with and without Bias Compensation

![**Results of Pure Quaternion Integration**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Result_QI.jpg)

**Accuracy and Stability:**
Quaternion integration offers a robust solution for 3D orientation tracking. Unlike Euler angles, quaternions do not suffer from gimbal lock, making them suitable for all types 
of maneuvers, including complex and prolonged ones. 

![**Results of Quaternion Integration with and without Bias Compensation**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Result_BC.jpg)

**Bias Compensation:**
Gyroscopes, though accurate in the short term, are prone to drift over time due to sensor biases and integration errors. Quaternion integration with bias compensation addresses this issue by incorporating low-pass filtering or other techniques to adjust for these biases. This approach stabilizes the final attitude readings, ensuring that the orientation data remains accurate and reliable over extended periods.

### Quaternion Integration with Complementary Filter

![**Results of Quaternion Integration with Complementary Filter**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Result_QI_CF.jpg)

**Effectiveness in Offset Bias Drift:**
The complementary filter is designed to mitigate the weaknesses of individual sensors by fusing their data. It effectively combines high-frequency gyroscope data with low-frequency accelerometer and magnetometer data. The gyroscope data is reliable over short periods but drifts over time, while the accelerometer and magnetometer provide stable long-term references but can be noisy.

**Noise Susceptibility:**
While the complementary filter can significantly reduce the bias drift from the gyroscope, it is also susceptible to the noise present in accelerometer and magnetometer data. External accelerations and magnetic disturbances can introduce errors, distorting the final orientation estimate. Proper tuning of the filter coefficients is essential to balance the contributions from different sensors and minimize the impact of noise.

![**Results of Bias Compensation using Complementary Filter**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Result_CF.jpg)

### Summary
In summary, each method for attitude and heading estimation has its strengths and weaknesses:

- **Euler Integration** is simple and quick but falls short in complex scenarios due to gimbal lock.
- **Quaternion Integration with Bias Compensation** offers the most accurate and stable results, especially for demanding applications requiring continuous orientation tracking.
- **Complementary Filter** is effective in reducing gyroscope bias drift and is computationally efficient, but it can be affected by noisy accelerometer and magnetometer data.

The choice of method depends on the specific requirements of the application, including the level of accuracy needed, the complexity of the maneuvers, and the computational resources available.

### Error Comparison Plots

The error comparison plots provide a visual representation of the performance of different attitude and heading estimation methods. These plots compare the deviation of the estimated orientation from the true orientation over time, illustrating the strengths and weaknesses of Quaternion Integration with Bias Compensation, and the Complementary Filter.


#### Error Comparison: Pure Quaternion Integration vs. Complementary Filter

![**Pure Quaternion Integration vs. Complementary Filter**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Error_Comparison_QI_CF.jpg)

The quaternion integration error plot illustrates the effectiveness of this method in maintaining accurate orientation over time. Quaternions avoid the gimbal lock issue and, with bias compensation, correct the drift inherent in gyroscope measurements.

The complementary filter error plot shows the combined performance of gyroscope, accelerometer, and magnetometer data. This method aims to balance short-term accuracy with long-term stability. However, the noise in the accelerometer and magnetometer readings imapct the accuracy of these reults adversely.

**Key Observations:**
- **Initial Stability:** Initially, the error is well-controlled, benefiting from the complementary nature of sensor data fusion.
- **Noise Impact:** Over time, the plot may show fluctuations due to the noisy accelerometer and magnetometer data, which can distort the orientation estimates.
- **Drift Correction:** While the complementary filter effectively reduces gyroscope drift, the noise from other sensors can introduce errors, seen as minor oscillations in the error plot.

**Conclusion:**
The Quaternion Integration vs. Complementary Filter error plot demonstrates the high accuracy and reliability of Quaternion Integration making it suitable for high-precision applications.

#### Error Comparison: Quaternion Integration without Bias Compensation vs. Quaternion Integration with Bias Compensation

![**Quaternion Integration without Bias Compensation vs. Quaternion Integration with Bias Compensation**](https://github.com/AdithiBhatta/AHRS_Algorithm_Implementation/blob/main/Figures/Error_Comparison_QI_BC.jpg)

This plot shows the performance of quaternion integration without any mechanism to correct for gyroscope bias drift. Gyroscope bias drift refers to the slow change in the gyroscope output that accumulates over time, leading to increasing error in orientation estimates.

This plot also illustrates the performance of quaternion integration with a bias compensation mechanism. Bias compensation typically involves low-pass filtering or other methods to correct the gyroscope drift, ensuring more accurate long-term orientation estimates.

  **Key Observations:**
- **Stable Accuracy:** The error remains low and stable throughout the duration of the test, even during complex maneuvers.
- **Smooth Transition:** The absence of gimbal lock ensures smooth and continuous tracking of orientation without abrupt spikes.
- **Bias Correction:** The low-pass filtering or bias compensation effectively reduces long-term drift, keeping the error minimal.

**Conclusion:**
The complementary filter error plot indicates that while this method is effective in offsetting gyroscope bias drift, it is susceptible to noise from accelerometer and magnetometer data. Proper tuning of the filter is essential to minimize these fluctuations and maintain accuracy.

#### Summary of Error Comparison

The error comparison plots provide a clear visualization of the performance differences between the three methods:

- **Quaternion Integration with Bias Compensation:** Maintains low and stable error, demonstrating high accuracy and reliability, especially for continuous tracking.
- **Complementary Filter:** Balances short-term and long-term accuracy but exhibits some sensitivity to noisy sensor data, which can impact overall performance.

These plots are crucial for understanding the practical implications of each method's strengths and weaknesses, guiding the selection of the most appropriate technique for specific applications.

## Conclusion

AHRS, utilizing an IMU with gyroscopes, accelerometers, and magnetometers, provides essential data for navigation and control in various applications. By employing 
advanced algorithms like quaternion integration and complementary filters, AHRS systems achieve accurate and stable orientation tracking, overcoming the limitations of 
individual sensors.
