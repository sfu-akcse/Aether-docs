
# SolidWorks Quick Start

## Purpose
This guide is designed for beginner team members to follow step-by-step and understand how to start using SolidWorks, create 2D sketches, and convert them into 3D models.

The goal is not just to explain features, but to enable users to follow instructions and complete tasks independently.


# 1. How to Start

## 1.1 Getting Started with SolidWorks

The first thing to do when using SolidWorks is to **standardize the working environment before modeling**.  

If basic settings such as units, view orientation, and sketch starting position are not properly configured, it can lead to errors later such as incorrect dimensions, orientation issues, or assembly problems.

---

## 1.2 Creating a New File

1. Launch SolidWorks  
2. Click **File** in the top-left corner  
3. Select **New**  
4. In the new document window, choose one of the following:  

   - **Part**: for creating a single component  
   - **Assembly**: for combining multiple parts  
   - **Drawing**: for technical drawings  

For beginners, it is recommended to start with **Part**.

---

## 1.3 Setting Units

Before modeling, always check the **unit system**.  

For mechanical parts and robot arm projects, **MMGS** is commonly used:

- **M**illimeter  
- **G**ram  
- **S**econd  

### How to Set Units

1. Check the unit display in the bottom-right corner  
2. If it shows inch or meter, click it  
3. Select **MMGS (millimeter, gram, second)**  

### Why It Matters

If units are incorrect, major errors can occur.  
For example, a 50 mm part could accidentally become 50 inches.

---

## 1.4 Saving Files

It is important to save your file before starting work.

### How to Save

1. Click **File → Save As**  
2. Choose your project folder  
3. Save using a consistent naming convention  

### Example File Names

- `base_plate_v1`  
- `joint_arm_A_v2`  
- `motor_mount_test`  

### Tip

Avoid unclear names:

- `part1` ❌  
- `newnewfinal` ❌  
- `arm_link_01` ⭕

---

## 1.5 Understanding the Interface

The SolidWorks interface consists of several key areas:

### (1) FeatureManager Design Tree  
Located on the left side.  
It shows the history of features such as sketches, extrusions, cuts, and fillets.

### (2) Graphics Area  
The main central workspace where you create and view models.

### (3) CommandManager  
The top menu tabs, including:
- Features  
- Sketch  
- Evaluate  

### (4) Heads-Up View Toolbar  
Located at the top of the graphics area.  
Used for zooming, rotating, and changing views.

---

## 1.6 Understanding Default Planes

SolidWorks uses three main reference planes:

- **Front Plane**  
- **Top Plane**  
- **Right Plane**  

These determine where your sketch begins.

### How to Choose

- Use **Top Plane** for top-down designs  
- Use **Front Plane** for front views  
- Use **Right Plane** for side views  

Most beginners start with **Top Plane** or **Front Plane**.

---

## 1.7 Starting a Sketch

1. Select a plane from the FeatureManager  
   - Example: Top Plane  
2. Right-click  
3. Select **Sketch**  

You are now in 2D sketch mode.

### Checkpoint

- The view aligns to the plane  
- The menu switches to Sketch tools  

---

## 1.8 Aligning the View (Normal To)

When sketching, it is important to view the plane **directly (perpendicular view)**.

### How to Use

1. Select the plane or sketch  
2. Click **Normal To**  
   or  
3. Press **Ctrl + 8**  

### Why It Matters

If the view is tilted, it becomes difficult to draw accurately or apply dimensions.

---

# 2. How to Draw 2D

## 2.1 What is a 2D Sketch?

A 2D sketch is the foundation of a 3D model.  
It is essentially a **flat drawing used to build 3D geometry**.

Examples:
- Draw a circle → create a cylinder  
- Draw a rectangle → create a block  

---

## 2.2 Starting a Sketch on Top Plane

Example: creating a circular base

### Steps

1. Select **Top Plane**  
2. Right-click → **Sketch**  
3. Use **Normal To**  
4. Begin sketching on the plane  

---

## 2.3 Drawing a Line

### Location

Sketch → Line

### Steps

1. Click **Line**  
2. Click the starting point  
3. Move the cursor  
4. Click the endpoint  
5. Continue drawing if needed  
6. Press Enter or Esc to finish  

### Tips

- Use guides for horizontal/vertical alignment  
- Close shapes by connecting the last point to the first  

---

## 2.4 Drawing a Circle

### Location

Sketch → Circle

### Steps

1. Click **Circle**  
2. Click the center point  
3. Drag outward  
4. Click to complete  

### Tip

Apply dimensions after drawing for accuracy.

---

## 2.5 Drawing a Rectangle

### Steps

1. Click **Rectangle**  
2. Use **Corner Rectangle**  
3. Click first corner  
4. Click opposite corner  

### Optional

Use **Center Rectangle** to draw from the center outward.

---

## 2.6 Using Centerline

A centerline is a **reference line**, not actual geometry.

### Steps

1. Click **Centerline**  
2. Draw like a normal line  

### When to Use

- Aligning centers  
- Creating symmetry  
- Defining revolve axes  

---

## 2.7 Applying Smart Dimension

### Steps

1. Click **Smart Dimension**  
2. Select geometry  
3. Place dimension  
4. Enter value  

### Examples

- Line length: 100 mm  
- Circle diameter: 40 mm  
- Distance between points: 20 mm  

### Why It Matters

Without dimensions, a sketch is not precise.

---

## 2.8 Sketch Relations

SolidWorks automatically or manually applies relationships:

- Horizontal  
- Vertical  
- Coincident  
- Midpoint  
- Tangent  

These help stabilize the sketch.

---

## 2.9 Fully Defined Sketch

A good sketch should be **Fully Defined**:

- Black: fully defined  
- Blue: under-defined  

### How to Achieve

- Add dimensions  
- Add relations  

### Why It Matters

Under-defined sketches can shift unpredictably.

---

## 2.10 Using Trim

### Steps

1. Sketch → Trim Entities  
2. Drag or click to remove unwanted parts  

### Example

Removing overlapping or extra lines

---

## 2.11 Offset Entities

Creates parallel geometry at a fixed distance.

### Use Cases

- Creating thickness  
- Creating inner/outer boundaries  

---

## 2.12 Positioning Holes

### Steps

1. Draw base shape  
2. Define reference distances  
3. Draw circle  
4. Apply dimensions  

### Example

- 20 mm from left  
- 15 mm from bottom  
- Diameter 8 mm  

---

# 3. Detail Sketch Functions

## 3.1 Creating Hole Sketch

### Steps

1. Draw base shape  
2. Define position using references  
3. Draw circle  
4. Apply dimensions  

---

## 3.2 Using Construction Lines

Use reference lines before drawing complex shapes:

- Centerlines  
- Symmetry lines  
- Midpoint references  

---

## 3.3 Creating Symmetry

### Steps

1. Draw centerline  
2. Draw half geometry  
3. Use Mirror Entities  
4. Complete symmetrical shape  

---

## 3.4 Adding Slots and Advanced Shapes

Additional sketch tools include:

- Slot  
- Arc  
- Spline  
- Chamfer (in sketch form)

---

# 4. How to Draw 3D

## 4.1 Converting 2D to 3D

Once the sketch is complete, convert it into a 3D model.  
The most basic method is **Extruded Boss/Base**.

---

## 4.2 Extruding

### Steps

1. Select closed sketch  
2. Click Features → Extruded Boss/Base  
3. Enter depth  
4. Confirm  

### Examples

- Circle → cylinder  
- Rectangle → plate  

---

## 4.3 Cutting (Extruded Cut)

### Steps

1. Select face  
2. Start sketch  
3. Draw shape  
4. Features → Extruded Cut  
5. Set depth or select Through All  
6. Confirm  

### Through All

Cuts completely through the model.

---

## 4.4 Using Hole Wizard

### Use Cases

- Bolt holes  
- Counterbore  
- Countersink  
- Threaded holes  

### Steps

1. Features → Hole Wizard  
2. Select type  
3. Set size  
4. Place location  
5. Confirm  

---

## 4.5 Using Fillet

Rounds edges.

### Steps

1. Features → Fillet  
2. Select edges  
3. Enter radius  
4. Confirm  

---

## 4.6 Using Chamfer

Creates angled edges.

### Steps

1. Features → Chamfer  
2. Select edges  
3. Set distance or angle  
4. Confirm  

### Difference

- Fillet: rounded  
- Chamfer: straight edge  

---

## 4.7 Using Revolve

### Steps

1. Draw profile  
2. Create centerline  
3. Features → Revolved Boss/Base  
4. Select axis  
5. Set angle  
6. Confirm  

---

## 4.8 View Controls

### Controls

- Mouse wheel: zoom  
- Click + drag: rotate  
- Shift + drag: pan  

### Common Views

- Front  
- Top  
- Right  
- Isometric  

---

## 4.9 Common Mistakes

1. Sketch not closed → cannot extrude  
2. Wrong units → incorrect scale  
3. Not fully defined → unstable model  
4. Wrong plane → incorrect orientation  
5. Not saving → risk of losing work  
