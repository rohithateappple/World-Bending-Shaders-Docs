# Real Transformations

WPOs aren’t always the ideal solution—especially when building large environments where performance is a priority. In such cases, it’s better to lean on what CPUs excel at: fast, real-time transformations. In this chapter, we’ll explore how to combine the power of WPOs with the **Blueprint Function Library** included in the World Bending Shaders to create efficient levels.

We'll be covering the same techniques used in the `Inception Optimized` example level, located at:
`WorldBendingToolkit/DemoMaps/TransformationExample`.

---

## Blueprint Function Library
WBS includes a set of Blueprint functions located in `WorldBendingToolkit/Blueprints/BFL_WorldBendingHelpers`. These functions are designed to convert any world location to its corresponding vertex-deformed position, along with the appropriate rotation. 

![alt text](<../images/Screenshot 2025-07-24 141321.png>)

To use these functions, simply right-click in your Blueprint graph and search for **"Transform To..."** followed by the name of the shader you want to convert to. For example, in this case, we'll be using `Transform To Spiral`. 

![alt text](<../images/Screenshot 2025-07-24 143303.png>)

For this demonstration, I’m using a simple spiral material setup based on `MF_Spiral_Vertical_X`. To streamline the workflow, I’ve also incorporated a Material Parameter Collection, which simplifies control from both the material and Blueprint sides.

![alt text](<../images/Screenshot 2025-07-24 143943.png>)

---

## Transforming To Shader Space

1. For demonstration purposes, we’ll sample the location from the green monkey, which uses an instance of the spiral shader. This makes it easier to visualize and understand the transformation process. Of course, you can use any location you prefer. The default monkey will act as the transformation subject.

    ![alt text](<../images/Screenshot 2025-07-24 144700.png>)

2. In the Blueprint Actor, (1) I’ve created a `Custom Event` that can be called in the Editor. Next, I used (2) `GetVectorParameter` and `GetScalarParameter` nodes to retrieve the necessary values from our Material Parameter Collection (MPC). Be sure to assign the correct MPC and parameter names. Finally, connect these values to the (3) `TransformToSpiral` node.

    ![alt text](<../images/Screenshot 2025-07-24 182306.png>)

3. For the `Location` input, I’ll sample the green monkey’s world position. I’ll also retrieve its **Forward** and **Right** vectors to plug into their respective inputs on the `TransformToSpiral` node.

    ![alt text](<../images/Screenshot 2025-07-24 183211.png>)

4. Lastly, I assign the output `Location` and `Rotation` from the `TransformToSpiral` node to the target actor. Make sure that both `SpiralDirection` and `ForwardAxis` match the settings used in your shader. Since I'm using the *vertical* and *X* variant of the spiral, I’ve set both values accordingly.

    ![alt text](<../images/Screenshot 2025-07-24 183707.png>)

5. Now, simply hit the editor button to convert the selected actor into its shader-space equivalent.

    ![type:video](../../videos/Transform to Spiral.mp4)

    <p align=""><em>Green Monkey uses WPO, Default Monkey uses real transformations.</em></p>

---

## Real-Time Transformations

Manually transforming objects is useful, but in many cases, you’ll want to replicate the dynamic, continuous nature of WPO bending. This can be done quite easily using what we’ve already set up—except instead of calling the transformation manually, we’ll use a `TimerByEvent` node to update our target actor every 0.045 seconds.
 
![alt text](<../images/Screenshot 2025-07-24 190111.png>)

Here’s the result: as the Material Parameter Collection updates, so does the target monkey’s position and rotation.

![type:video](../../videos/Real-Time Transform.mp4)

---

## Mix and Match

Real transformations combined with material WPOs allows us to create highly performant large-scale worlds. In the `Inception Optimized` level, for example, only the terrain utilizes WPOs, while everything else is transformed via Blueprints. This approach eases the GPU load and helps work around WPO limitations.

![alt text](<../images/Screenshot 2025-07-24 191636.png>)

---