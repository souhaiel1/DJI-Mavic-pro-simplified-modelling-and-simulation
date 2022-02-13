# DJI Mavic pro simlified modelling and simulation 
Th objectives of this project are to establish the mathematical model and the physics behind a four propeller drone modelled after the DJI Mavic Pro, design a SIMULINK model simulating the complete dynamics of the drone as well as location and gusts and finally, design and implement a PID controller to control its position and attitude. 

**The SIMULINK models are provided for Matlab 2021a and 2018b**  
## The simplified DJI Mavic pro 
![](Images/drone_model.PNG)

## The components to be modeled 
![](Images/parts.PNG)

## The Foces at play 
![](Images/forces.PMG)

## The complete model Diagram
![](Images/diagram.PNG)

## Battery and Motors limit

We assume that : 
- The Mavic Pro’s battery consists in three high discharge capacity Lithium-Polymer cells mounted in Series.
- Each cell has an average capacity of 3830mAh and a nominal voltage of 3.8V.
- Its total capacity is 43.6Wh for a weight of about 240g.
- The nominal battery voltage: 𝑉 = 3.8 ∗ 3 = 11.4V 
- maximum discharge capacity is 20C (staying conservative): I = 3.830 ∗ 20 = 77A
Thus, the max operating voltage which we will take as 11.4V to consider an average over one battery discharge, and a max current of 77A. 

## Emperical modelling of the DC motors
We assume a 0% RPM drop from KV at 0V and a 25% RPM drop from KV at max voltage: 11.4V. The RPM equation (with loading assumption) can be extracted by fitting a trend line to the data using Excel: 
![](Images/motor_modelling.PNG)

## Emperical modelling of propellers 
The APC, a composites company which manufactures propellers has published the detailed performance of its propellers on its website: https://www.apcprop.com/technical-information/performance-data/. This data is used to calculate thrust and torque and will conduct our analysis in hover or at 0 forward speed.
### Thrust 
Thrust can be obtained in N from the coefficient of thrust using the following formula: 𝑇ℎ𝑟𝑢𝑠𝑡 = 𝐶𝑇 ∗ 𝜌 ∗ 𝑛² ∗ 𝐷
With 𝐶𝑇 the coefficient of thrust, 𝜌 the air density at sea level (1.225 kg/𝑚3), n the amount of revolutions per second and D the diameter of the propeller in meters.
This thrust vector will be taken as acting exactly perpendicularly to the drone, 90° to the horizontal.
![](Images/thrust.PNG)

### Torque 
In real life, the propeller will load the motor with the torque required to overcome drag at the current RPM. Here, we will extract the Torque from the APC experimental data in order to actuate and measure yaw (rotation around z axis) and estimate current consumption.
As we did with the KV constant we can fit a polynomial equation to the available data:
![](Images/torque.PNG)

### Relationship between Current and Torque
In a DC motor, torque and current are proportional to each other according to the following formula: 𝑇𝑜𝑟𝑞𝑢𝑒 = 𝐾𝑇 ∗ 𝐼 With 𝐾𝑇 the Torque Constant in Nm/Amps and I the current in Amps.
The Torque Constant of the motor is actually the inverse of the motor RPM constant KV, hence : 
![](Images/current.PNG)


## Drone Dynamics 

To keep things simple, we assume that our drone is made of two beams placed perpendicularly to each other. 
![](Images/shape.PNG)

### Linear and Rotational Dynamics

We assume that this mass is uniformly distributed across the drone. In order to study the motion of our drone in Space we need to predict its translational
and rotational motion in 3 dimensions (x, y and z).
According to the 2nd law of Newton: F = 𝑚𝑎 . 
Similar rules apply to rotational movements : 𝑀𝑥 = 𝐼𝑥 ∗ rotational acceleration , with M the external moments or torques in Nm, I the moments of inertia in
kg.m²

### Yaw moment of Inertia 
The moment of inertia of a point mass about a principal axis of rotation is: 𝐼 = 𝑚𝑟²
Developing this integral for a rod about its end we can write that: 𝐼 = 1/3 * 𝑀𝐿²,  with m the mass of the rod in kg and L its length in m.
About the z axis (yaw), we therefore obtain:
![](Images/yawmoment.PNG)

### Roll and Pitch momemnts of Inertia 
Because we assumed our drone is perfectly symmetrical (like a square) and looks the same from all sides, the roll and pitch moments of inertia are the same.
As a good approximation we can apply 𝐼 = 1/3 * 𝑀𝐿² , L now being the distance from the center of mass of the drone when looking sideways.
![](Images/yawrollmoment.PNG)

## Actuations : Momemnts and Disturbances
### Torques and propeller rotation 

- The torques necessary to rotate each propellers also act on the drone.
- Props 2 and 4 rotate in the same direction and oppositely to 1 and 3.
- Decreasing/increasing the power in each of these couples independently induces yaw.

![](Images/proptorque.PNG)

We can write : 𝑀𝑧 = 𝑇4 − 𝑇1 + 𝑇2 − 𝑇3

### Thrust and Moments: 

-F1, F2, F3 and F4 each represent the thrust of their respective propeller.
- mg is the force exerted by gravity on the center of mass of the drone with m the mass of the drone (0.743 kg) and g the gravitational acceleration 9.81 𝑚. 𝑠−2 
- F1 and F2 as well as F4 and F3 are respectively coupled to induce pitch.
- F4 and F1 as well as F3 and F2 are respectively coupled to induce roll.
![](Images/thrustandmoments.PNG)

### Pitch and Roll movements

![](Images/pitchroll.PNG)

When pitching positively F1 and F2 decrease while F3 and F4 increase, leading the drone to pitch by 𝜃𝑥. When rolling positively F3 and F2 increase while F4 and F1 decrease keeping total thrust constant but leading 𝜃𝑦 to vary.
The gravitational force acts through the center of mass so it does not induce a moment here. 
![](Images/rollpitchmoments.PNG)

### Drag and Disturbances 
![](Images/rollpitchmoments.PNG)
The drag is modeled in the most simple way possible : 
- In the x and y directions we use the constant projected area estimated in the design brief: A=0.0197m², the drag Coefficient Cd of a cube: 1.00 as well as the air density at sea level: ρ = 1.225 𝑘𝑔. 𝑚−3
- In the z direction, we use the same coefficient of drag and air density but double the area as we have two beams crossing eachother and add 30% to take into account the rotors: A=0.0512m².

In reality, projected area varies with respect to the pitch and roll of the aircraft as well as direction of flight :
 𝐷𝑟𝑎𝑔 = 1/2 ∗ ρ ∗ 𝑉²𝑤𝑖𝑛𝑑 ∗ A * 𝐶d

## Thrust Vectors and Linear Forces
### Roll and Pitch Thrust Vectors
![](Images/rollpitcthrust.PNG)

### Yaw Thrust Vector 
Yaw orients the drone in a certain direction depending on the angle 𝜃𝑧
![](Images/yawthrust.PNG)

The projections yield the following equations: 
![](Images/equations.PNG)

### Sum of Linear Forces : 
Thrust through the propellers is responsible for counteracting gravitational forces, drag, disturbances and accelerating the drone in the x, y or z directions.
- 𝐹𝑥 = 𝐹𝑝𝑟𝑜𝑝𝑥 (+/-) 𝐷𝑟𝑎𝑔x
- 𝐹y =𝐹𝑝𝑟𝑜𝑝y (+/-) 𝐷𝑟𝑎𝑔y
- 𝐹𝑧 = 𝐹𝑝𝑟𝑜𝑝𝑧 -mg (+/-) 𝐷𝑟𝑎𝑔z



