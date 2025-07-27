# Performance & Optimizations

World Bending Shaders has been rigorously tested in both the editor and packaged builds to ensure reliable performance. Delivering stability is a top priority, and numerous optimizations have been applied throughout development to achieve optimal results. In this section, we’ll explore common issues along with their solutions, and also highlight some limitations of the asset pack.

---

## Mesh Bounds

The first—and likely most common—issue you may encounter is meshes disappearing. This typically occurs when deformation pushes the mesh outside its original bounds, causing it to be culled. The fix is straightforward: increase the `Bounds Scale` in the **Details** panel. However, use this setting with care—setting it too high can negatively impact performance.

![type:video](../../videos/Tiny Planet Cull_1.mp4)

<p align=""><em>a. Hut in the back is culled</em></p>

![type:video](../../videos/Tiny Planet Cull_2.mp4)

<p align=""><em>b. Hut with higher Bounds Scale </em></p>

![alt text](<../images/Bounds Scale.png>)

---

## Max World Position Offset Displacement

Sometimes, increasing the `Bounds Scale` won’t affect the culling behavior. In these cases, you’ll need to raise the `Max World Position Offset Displacement` value in the Material’s **Details** panel. This will cause the mesh to *clip* beyond a certain point, and unfortunately, there’s no current workaround for this. To manage the clipping, you’ll have to get creative with hiding it. Also, if lowering the `Bounds Scale` doesn’t impact how the mesh deforms, keep it reduced to help preserve GPU performance as much as possible.

![alt text](<../images/WPO Max difference.png>)

<p align="center"><em>a. Max WPO Displacement = 0 | b. Max WPO Displacement = 7500 </em></p>

![alt text](<../images/Screenshot 2025-07-23 153427.PNG>)

### Nanite Holes

When using Nanite with certain shaders, you might notice *holes* appearing in the meshes. This issue is once again related to mesh bounds. In most cases, increasing the `Max World Position Offset Displacement` value will resolve the problem.

![alt text](<../images/Screenshot 2025-07-23 191803.png>)

<p align="center"><em>Nanite Holes when Max WPO Displacement is low.</em></p>

![alt text](<../images/Screenshot 2025-07-23 191826.png>)

<p align="center"><em>Max WPO Displacement = 5000.</em></p>

!!! note
    If your mesh uses **multiple** materials, you don't need to set the `Max WPO Displacement` value in every one of them. Setting it in just a single material is usually sufficient.
     
### Optimizations

Setting `Max WPO Displacement` too high can negatively impact performance. In shaders with strong Z-positive deformations—like `MF_Runner`—you might be tempted to increase the value to avoid clipping. But rather than applying a high value across the board, it’s better to split the mesh and assign a larger displacement only to the parts that actually deform upwards. This keeps unnecessary calculations to a minimum.

For example, in the **Inception Town** scene, only the upward-bending chunks use a high Max WPO Displacement, helping maintain performance.

![alt text](<../images/Screenshot 2025-07-24 133020.png>)

---

## Nanite Performance

It’s important to understand that Nanite doesn’t perform well with high `Max World Position Offset Displacement` values. The larger this value gets, the less effective Nanite becomes—and if it’s too high, it can actually result in worse performance than having Nanite disabled altogether.

Knowing when to use Nanite versus traditional LODs is key. The **Infinite Racer** example level is a good demonstration of this. At first, I enabled Nanite on all meshes, but performance took a hit because the `Max WPO Displacement` was too high. After switching to traditional LODs and disabling Nanite, performance improved significantly.

![alt text](<Screenshot 2025-07-23 192943.png>)

<p align="center"><em>Packaged Infinite Racer using Nvidia Stats</em></p>

---

## Nanite Oversimplification

In Unreal Engine versions 5.4 and above, Nanite can sometimes *over-simplify* meshes, leading to unwanted "low poly" artifacts in certain shaders. For example, in the image below, you can see how the floor texture becomes warped near the end of the hallway, and the mesh edges start appearing noticeably sharper.

![alt text](<../images/Nanite Oversimplification A.png>)

To fix this, go to your mesh settings, in the **Nanite** section increase the `Max Edge Length Factor` to something appropriate, in this case 100. 

![alt text](<../images/Screenshot 2025-07-23 203840.png>)

Once the settings are **applied**, the same hallway becomes much smoother.

![alt text](<../images/Nanite Oversimplification B.png>)

Here's another comparison, side by side. Image **A** shows significant over-simplification at the end, Image **B** shows desired effect.

![alt text](<../images/Nanite Oversimplification comparison.png>)

!!! note
    The `Max Edge Length Factor` setting is available only in Unreal Engine 5.4 and later. Earlier versions did not show this over-simplification behavior during testing.

---

## Instanced Static Meshes 

Instanced Static Meshes (ISMCs) tend to perform poorly with materials that use World Position Offset (WPO). I tested this asset using **1000** instances of a 250k triangle mesh (totaling 2.5 million triangles), and the performance dropped significantly as soon as WPO was applied. This isn’t a flaw specific to this asset pack—it's a known issue with how the ISMC rendering pipeline handles WPO.

The easiest fix is to enable Nanite, which can greatly boost performance. Just be sure to keep in mind the Nanite limitations we discussed earlier.

![alt text](<../images/ISMC No Nanite.png>)

<p align="center"><em>500 Instances @250k tris (No Nanite)</em></p>

![alt text](<../images/ISMC Nanite.png>)

<p align="center"><em>500 Instances @250k tris (Nanite)</em></p>


### Particle Pivot

If you want certain shaders like `MF_Balloon` to use the instance’s own local pivot instead of a global or fixed one, just connect the `Mesh Particle Pivot Location` node to the shader’s `Pivot` input. This allows the deformation to originate from each instance’s center.

![alt text](<../images/Screenshot 2025-07-24 110910.png>)

![alt text](<../images/Screenshot 2025-07-24 110838.png>)

<p align="center"><em>Individual Balloon Inflation</em></p>

### Particle Axis

For shaders like `MF_TheScream`, you might want the effect to align with the local direction each instance is facing. To achieve this, open `MF_WaveFunction` (found in `WorldBendingToolkit/MaterialFunctions/TheScreamFunctions`) and transform the input `Position` from **World Space** into **Instance & Particle Space**. This ensures the wave deformation respects the orientation of each instance.

![alt text](<../images/Screenshot 2025-07-24 111736.png>)

![alt text](<../images/ISMC Local Axis.png>)

<p align="center"><em>(L) Global Axis,  (R) Local Axis. Arrows indicate direction of wave.</em></p>

---

## Translucent and Other Materials

WPOs operate on the mesh itself, not the material type. This means you can apply vertex deformation to any mesh, regardless of the material it's using. For instance, in the Tiny Planet example, I applied WPO to a `Single Layer Water Material` without issue.

![alt text](<../images/Screenshot 2025-07-24 113805.png>)

<p align="center"><em>Simple water material using MF_TinyPlanet WPO</em></p>

### Render Velocities

Transparent materials give us a valuable chance to optimize performance by disabling Render Velocities. If your material has `Output Velocity` enabled, consider turning it off in the material’s detail panel. Velocity rendering is mainly used for effects like motion blur and temporal anti-aliasing, but it can be costly on the GPU. If your material doesn’t need it, disabling it can improve performance.

![alt text](<../images/Screenshot 2025-07-24 114643.png>)

---

## Dynamic Shadows 

Another way to optimize performance is by disabling `Dynamic Shadows`, especially in large environments like terrains or landscapes. However, only consider this if you're okay with sacrificing some visual quality.

---

## Landscapes

Landscapes can be a tricky subject with this asset. I’ve thoroughly tested each shader on landscapes and found that while WPO effects generally work fine, issues arise with normal compatibility. As discussed in the previous chapter, without dynamic normals, the landscape’s appearance can break down—especially with heavy deformations.

Additionally, optimizing performance on landscapes can be difficult and unpredictable. Because of that, I recommend importing your terrain as a separate mesh if you plan to use WPOs extensively. Still, you're encouraged to experiment and see what works for your setup!

!!! note
    Make sure to increase `Positive Bounds Extension` or `Negative Bounds Extension` via details panel depending on your shader. This reduces flickering and clipping.

![alt text](<../images/Screenshot 2025-07-24 133449.png>)

------




