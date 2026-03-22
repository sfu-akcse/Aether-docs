# Claw Design Reseach #13
Objective - Research the optimal geometry, size, and grip mechanism for the claw. This iteration must determine the specific number of servo motors required to actuate the claw and provide a preliminary sketch of the finger components.

 ## A) Basic reasearch / [Link(A)](https://notes.myscript.com/app/page/ddc5efe2-1dfd-4be5-8551-cbaabeb7f887)
1. **Design consideration : On every project that deals which the robot arm needs 4 main consideration.**
    - DOF (degrees of freedom)
        AKA. Number of joints.
        - 2-3 ㅣ minimum requirement for limited 3D movement
        - 4-5 ㅣ requirement for basic 3D movement
        - 6 ㅣ minimal requirement for imitation of human arm movement.  
        *(Since one of the requirement is imitating, 4-5 is recommanded)*
    - Work Space
    - Payload
    - Precision
2. **Structure : This is basic structure for general robotic arm.**
    - Joint
        - a motorized, pivotal component that connects two rigid parts (links) of a robot, enabling relative movement, flexibility, and positioning.  
        (*Shoulder -> Elbow -> Wrist joint for 6 D.O.F*)
    - Link
        - a rigid, structural segment of a robot's manipulator or body that connects to other links via joints.
    - Base
        - the foundational structure or coordinate system from which a robot operates.
    - End Effector
        - devices attached to the end of a robotic arm, acting as the "hand" or specialized tool (End of Arm Tooling - EOAT) that interacts directly with the work environment.

3. **Joint Decision : for each joints/ motors in the structure, there is various implimentable method.**
    - Gear driven: heavy, easy to modify
         Here is unique set of drives / gearbox.
        - Harmonic
        -specialized, compact, and lightweight gear systems known for high reduction ratios, high torque capacity, and zero backlash.

        - Cycloidal
        -a high-torque, high-reduction speed mechanism that uses a cam-driven disc rolling inside a ring gear, rather than standard meshing teeth, to achieve motion reduction

         - Plantary
        -consists of a central sun gear, multiple planet gears mounted on a carrier, and an outer ring gear.
    - Cable driven: Light, hard to modify and manage.
        There is only difference in material. Below some key points.
        - Could tangle
        - Very good on resume (Newest method, adds complexity)
        - Since it's not for rotation, the motors for R will be needed.
4. **End Effector Discussion: Below is the list decision required for gripper(End Effector)**
    - Problem: has done the reseach about Effector as well but could not conclude to one design due to following reason.  
    (The research has been moved to *part B-sec2* of this document)
        - 1. Are we aiming for funtionality or imitating part? (Method)
        - 2. What are the objects that we are looking forward to grab? (Payload) (Method)
        - 3. What are the specs for the arm? (Payload)
        - 4. What is the budget? (Method)

## FeedBack on (A)
### Questions Answered   
- 1. Are we aiming for funtionality or imitating part? (Method)
    > Funtionality.
- 2. What are the objects that we are looking forward to grab? (Payload) (Method)
    > Simple soda can, pencil case?
- 3. What are the specs for the arm? (Payload)
    > not industrial, try to go minimal. 
- 4. What is the budget? (Method)
    > not much, considering ~200 for the End Effector.

 ## B) Design Decisions / [Link(A)ㅣ](https://notes.myscript.com/app/page/ddc5efe2-1dfd-4be5-8551-cbaabeb7f887) [Link(B)](https://notes.myscript.com/app/page/e3c82bde-cc74-46e0-99d5-8736ab7bd3b6)
1. **Functional requirement : Below is stated(known) functional requirement.**
    - Should cost under ~200
    - The machine has to maintain it's functionality in the process of picking up target weight of ~300g.
    - Should be functional/flexible enough to pick up target weight and different shape.
    - There should be no obstacles in the process of mimicking arm (sight).
2. **End Effecor Design : This is basic design research for End Effector.**
    + **Big path**
        - Anthoropomorphic
            - A robotic gripping device designed to mimic the form, kinematics, and functionality of a human hand.  
        - Functional
            - The peripheral device attached to the end of a robotic arm that interacts directly with the environment to perform specific tasks.
    + **Both path can be driven by:**
        - Wire driven
            - lighter, more complex, good for mimicking
            - Not for heavy objects. (300 ~ 600 with SG90 or MG996R.)
        - Motor driven
            - Heavier, less complex.
            - TOO HEAVY!! It amplifies the motor load.
    + **Current Designs : Designs that's being used currently.**
        1. Non newton meterial.[ㅣ@MIT](https://www.fastcompany.com/90319354/mit-invented-a-new-type-of-robot-hand-thats-adorable-and-terrifying)
            - Idea from MIT
            - Can exert 100x from it's weight.
            - Alt, granular joining tech.
            > this method needs vaccuming tech.  
        2. Imitationg Invertebrate [ㅣSpiRobs](https://www.sciencedirect.com/science/article/pii/S2666998624006033?__cf_chl_tk=bFlY1FpSMR6vtpGcBz.9H6Hqw5gnwclvYE0TkTJM.aE-1774135434-1.0.1.1-4ORpJGijdjkb.YG..b4QMUrXXu9ED53CUD._JIbID40)
            - Cable structure for grabbing
            - Light weight
            > hard to design, but worth trying.  
            > not a shape of hand so software difficulty is foreseeable.
        3. Human hand imitation (using servo).
            - Actual human hand shape (or less finger)
            - Good for mimicking
            > havy, too heavy.
        4. Model O (using cable).[ㅣ@YALE](https://www.eng.yale.edu/grablab/openhand/)
            - 3finger Human hand open project.
            - Open source, does have instruction.
            > payload issue, price issue, is it our own work?

    + **Industry Designs : Designs from humaniod robots (they have similar limitations as ours).**
        1. Tesla Optimus Gen 3[ㅣ@TESLA](https://www.tesla.com/en_ca/AI)
            - Cable driven 5 finger.
            - Controll motor is located at it's shoulder.
            - Has it's own unique joint system for the finger seg.
            > Too complicated.  
        2. Atlas [ㅣ@Boston Dynamics](https://bostondynamics.com/products/atlas/)
            - Motor driven 3 transformable finger
            - Can transform 2 finger to 3 finger or vise versa to acheive universial grasp.
            - Has camera on it's palm as well for the precise controll.
            > this method is too havy for our project.  
        


3. **Effector Decision : The suggestion.**[ㅣLink(B)](https://notes.myscript.com/app/page/e3c82bde-cc74-46e0-99d5-8736ab7bd3b6)
    - **Cable Driven Reconfigurable Gripper**
        - This method follows the same track with *Model 0* , *Atlas*,  
        As stated before, both were meeting design requirements except for weight, originality, price.  
        (*Although originality wasn't the design consideration, In larger scale, it is recommanded*)
        - So Instead of heaving unececery weight and function. Since we are focusing on the grabing not mimicking, we came up with simplest reconfiguring mechanism for our end effector design.
        - **Solution**[ㅣ(Desmos Simulation)](https://www.desmos.com/calculator/c1ttbfkdms)
            - A cable-driven robotic gripper designed for versatile grasping by reconfiguring its finger alignment using an integrated gear and crank system.
            - Description: 
                - Expansion: As the servo motor rotates, the crank assembly pushes the kinematic links, increasing the distance (gap) between Finger 1 and Finger 2.
                - Contraction: When the servo motor operates within the range of $\pi/2 > x > -3\pi/2$, the mechanism reverses its stroke, decreasing the gap between the fingers to accommodate smaller objects.
                - Static Reference: Finger 3 remains stationary, acting as a fixed mechanical stop to ensure stable three-point contact during manipulation.

# C) Cost Calculation (Based on Model 0) [ㅣsource](https://www.eng.yale.edu/grablab/openhand/)

| Class | Component | Specification | Count | Price |
|------|------------------|-----------------------------|------|-------|
| Actuator | Main Tendon Servo | DS3218 (20kg-cm, Metal Gear) | 1 | $25 |
| Actuator | Reconfig Servo | MG90S (Metal Gear) | 1 | $10 |
| Tendon | High-Strength Cable | Spectra/Dyneema Braided (200lb+) | 2m | $18 |
| Friction | Low-Friction Liner | PTFE Teflon Tubing (ID 1mm) | 1m | $8 |
| Bearing | Pulley Bearings | MR85ZZ / MR105ZZ | 6-10 | $12 |
| Joint | Joint Pivot Pins | M3 Stainless Shoulder Bolts | 5-8 | $20 |
| Grip | Friction Pads | VytaFlex Urethane or TPU | 1 | $10 |
| Controller | Microcontroller | Arduino Nano + PCA9685 | 1 | $15 |
| **Total** | | | **$118.00** |

(*This is a theoretical model subject to further refinement.*)