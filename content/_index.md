+++
title = "AMR-LB"
outputs = ["Reveal"]
interactiveImages = true
useTikz = true
+++

## Towards  dynamically  load-balanced reactive CFD using OpenFOAM

<div style="height: 10%;"></div>

Holger  Marschall,<br>
Mohammed  Elwardi  Fadeli

12/2021


---

<div style="height: 10%;"></div>
<div class="columns-container">

<div class="col" style="width: 30%; float:left; padding: 60px;">
<img data-src="images/profile.png">
</div>

<div style="flex: 2;">
<div style="height: 40px;"></div>
<section data-markdown>
<script type="text/template">

- Msc. in Petroleum Engineering <!-- .element: style="font-size: 0.8em;" -->
- Maintainer of an OpenFOAM-based Reservoir Engineering Toolkit (OpenRSR) <!-- .element: class="fragment" data-fragment-index="2"  style="font-size: 0.8em;"-->
  - Which provides **coupled solvers** for Black-Oil Equations <!-- .element: class="fragment" data-fragment-index="2"  style="font-size: 0.8em;"-->
- LFCE-certified (Linux Engineer) who likes frictions with Math problems <!-- .element: class="fragment" data-fragment-index="3"  style="font-size: 0.8em;"-->
</script>
</section>
</div>

</div>

---

## We'll be discussing ...

1. Adaptive Polyhedral Mesh Refinement
2. Dynamic Load-balancing applied to chemistry and AMR

---

# Native Adaptive Mesh Refinement

---

## Why would we opt for AMR?

<section data-markdown>
<script type="text/template">

- **Dynamically** densify the mesh in regions of interest (only) to increase solution accuracy
    - Uniformly fine meshes are usually expensive <!-- .element: class="fragment" data-fragment-index="2"  style="font-size: 0.8em;"-->
- Coarsen the mesh in other regions to save computation time <!-- .element: class="fragment" data-fragment-index="3" -->
    - Regions where no critical flow features are developing <!-- .element: class="fragment" data-fragment-index="3"  style="font-size: 0.8em;"-->
</script>
</section>

---

- Potential for unattended and cheap accuracy increase:
    1. Start solving on coarse mesh
    2. Compute discretization errors
    3. Refine where errors are high
    4. Repeat 1-3 until errors fall under a certain tolerance value
    5. Pass to the next time step

---

<section data-noproecess>
<h3> Geometric Multi-grid methods vs. AMR</h3>
<div class="tikz">
<script type="text/tikz">
\begin{tikzpicture}[every node/.style={scale=0.5}]
    \usetikzlibrary{positioning}
    \begin{scope}[
            yshift=-83,every node/.append style={
            yslant=0.5,xslant=-1},yslant=0.5,xslant=-1
            ]
        \fill[white,fill opacity=0.9] (0,0) rectangle (5,5);
        \draw[step=4mm, black] (0,0) grid (5,5); %defining grids
        %\draw[step=1mm, red!50,thin] (3,1) grid (4,2);  %Nested Grid
        \draw[black,very thick] (0,0) rectangle (5,5);%marking borders
        %\fill[red] (0.05,0.05) rectangle (0.35,0.35);
    \end{scope}

    \begin{scope}[
            yshift=0,every node/.append style={
            yslant=0.5,xslant=-1},yslant=0.5,xslant=-1
            ]
        \fill[white,fill opacity=0.9] (0,0) rectangle (5,5);
        \draw[step=6mm, black] (0,0) grid (5,5); %defining grids
        %\draw[step=1mm, red!50,thin] (3,1) grid (4,2);  %Nested Grid
        \draw[black,very thick] (0,0) rectangle (5,5);%marking borders
        %\fill[red] (0.05,0.05) rectangle (0.35,0.35);
    \end{scope}

    \begin{scope}[
    	yshift=90,every node/.append style={
    	yslant=0.5,xslant=-1},yslant=0.5,xslant=-1
    	             ]
        \fill[white,fill opacity=0.9] (0,0) rectangle (5,5);
        \draw[step=10mm, black] (0,0) grid (5,5); %defining grids
        %\draw[step=1mm, red!50,thin] (3,1) grid (4,2);  %Nested Grid
        \draw[black,very thick] (0,0) rectangle (5,5);%marking borders
        %\fill[red] (0.05,0.05) rectangle (0.35,0.35);
    \end{scope}
    	
    \begin{scope}[
            yshift=-83, xshift=500,every node/.append style={
            yslant=0.5,xslant=-1},yslant=0.5,xslant=-1
            ]
        \fill[white,fill opacity=0.9] (0,0) rectangle (5,5);
        \draw[step=10mm, black] (0,0) grid (5,5); %defining grids
        \draw[step=5mm, green!50!black] (1,1) grid (4,3);  %Nested Grid
        \draw[step=2.5mm, red] (3,1) grid (4,2);  %Nested Grid
        \draw[black,very thick] (0,0) rectangle (5,5);%marking borders
        %\fill[red] (0.05,0.05) rectangle (0.35,0.35);
    \end{scope}

    \begin{scope}[
    	yshift=90, xshift=500, every node/.append style={
    	yslant=0.5,xslant=-1},yslant=0.5,xslant=-1
    	             ]
        \fill[white,fill opacity=0.9] (0,0) rectangle (5,5);
        \draw[step=10mm, black] (0,0) grid (5,5); %defining grids
        %\draw[step=1mm, red!50,thin] (3,1) grid (4,2);  %Nested Grid
        \draw[black,very thick] (0,0) rectangle (5,5);%marking borders
        %\fill[red] (0.05,0.05) rectangle (0.35,0.35);
    \end{scope}
    	

    \draw[-latex,thick,draw=green!50!black](5.8,-.3) node[right,color=white]{Fine\hspace{3cm}mesh}
        to[out=180,in=90] (3.9,-1);
    \draw[-latex,thick,draw=green!50!black] (6.2,2) node[right,color=white]{Finer\hspace{4.5cm}mesh}
         to[out=180,in=90] (4,2);
    \draw[-latex,thick,draw=green!50!black](5.9,5) node[right,color=white] (coarse) {Coarse\hspace{5cm}mesh}
        to[out=180,in=90] (3.6,5);
    \draw[-latex,thick,draw=green!50!black](11.2,5) to[out=0,in=90] (14,5);
    \draw[-latex,thick,draw=green!50!black](10.2,-.3) to[out=0,in=90] (14,-1);

\end{tikzpicture}
</script>
</div>
</section>

---

### Success stories, when it works

- 10x Speedup for Adaptive Mesh Fluid Simulations on GPU, [P. Weng et al. (2009)](https://arxiv.org/pdf/0910.5547.pdf)
- 8x Speedup for Shock Hydrodynamics, [Berger and Colella (1988)](https://crd.lbl.gov/assets/pubs_presos/AMCS/ANAG/A113.pdf)

---

## What's already available?

- OpenFOAM has a mature Hex-based adaptive refinement engine
- Professors Jasak & Vuko have laid the foundation for polyhedral adaptive refinement
  in Foam-Extend 4, nextRelease branch

---

## What we’re bringing to the table?

- Native multi-criteria refinement & unrefinement for polyhedral meshes
    - Built on top of Foam-Extend's engine
    - With support for cell-count based Load Balancing

---

{{< slide background-color="white" >}}
### Interface-tracking refinement <br>(15k cells, initial mesh: 8.5k)

<img style="width:90%; margin:0; padding:0; position-top: 0;" data-src="images/alpha1.png">


---


<p style="text-align:center;">Interface-tracking refinement</p>

<section data-auto-animate data-auto-animate-unmatched="false">
{{< foamCode "1-3,5,7" >}}
<span class="hljs-title">dynamicFvMesh</span> dynamicPolyRefinementFvMesh<span class="hljs-punctuation">;</span>
<span class="hljs-type">dynamicRefineFvMeshCoeffs</span> <span class="hljs-punctuation">{</span>
    <span class="hljs-comment">// 1. Global refinement settings</span>
    ...
    <span class="hljs-comment">// 2. Refinements description</span>
    ...
<span class="hljs-punctuation">}</span>
{{< /foamCode >}}
</section>

<section data-auto-animate data-auto-animate-unmatched="false">
{{< foamCode "3-10" >}}
<span class="hljs-title">dynamicFvMesh</span> dynamicPolyRefinementFvMesh<span class="hljs-punctuation">;</span>
<span class="hljs-type">dynamicRefineFvMeshCoeffs</span> <span class="hljs-punctuation">{</span>
    <span class="hljs-comment">// 1. Global refinement settings</span>
    <span class="hljs-comment">// When to refine and unrefine</span>
    <span class="hljs-title">refineInterval</span> <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
    <span class="hljs-title">unrefineInterval</span> <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
    <span class="hljs-comment">// Stop refinining after reaching a certain level</span>
    <span class="hljs-title">maxRefinementLevel</span> <span class="hljs-number">3</span><span class="hljs-punctuation">;</span>
    <span class="hljs-comment">// Do refinement and refinement separately?</span>
    <span class="hljs-title">separateUpdates</span> <span class="hljs-keyword">false</span><span class="hljs-punctuation">;</span>
    <span class="hljs-comment">// 2. Refinements description</span>
    ...
<span class="hljs-punctuation">}</span>
{{< /foamCode >}}
</section>

<section data-auto-animate data-auto-animate-unmatched="false">
{{< foamCode "5-11" >}}
<span class="hljs-title">dynamicFvMesh</span> dynamicPolyRefinementFvMesh<span class="hljs-punctuation">;</span>
<span class="hljs-type">dynamicRefineFvMeshCoeffs</span> <span class="hljs-punctuation">{</span>
    <span class="hljs-comment">// 1. Global refinement settings</span>
    ...
    <span class="hljs-comment">// 2. Refinements description</span>
    <span class="hljs-title">refinements</span> <span class="hljs-punctuation">(</span>
        basedOnAlpha1 <span class="hljs-punctuation">{</span>
            ...
        <span class="hljs-punctuation">}</span>
        <span class="hljs-comment">// More refinement/unrefinement operations</span>
    <span class="hljs-punctuation">)</span><span class="hljs-punctuation">;</span>
<span class="hljs-punctuation">}</span>
{{< /foamCode >}}
</section>

<section data-auto-animate data-auto-animate-unmatched="false">
{{< foamCode "7-12,14" >}}
<span class="hljs-title">dynamicFvMesh</span> dynamicPolyRefinementFvMesh<span class="hljs-punctuation">;</span>
<span class="hljs-type">dynamicRefineFvMeshCoeffs</span> <span class="hljs-punctuation">{</span>
    <span class="hljs-comment">// 1. Global refinement settings</span>
    ...
    <span class="hljs-comment">// 2. Refinements description</span>
    <span class="hljs-title">refinements</span> <span class="hljs-punctuation">(</span>
        basedOnAlpha1 <span class="hljs-punctuation">{</span>
            <span class="hljs-comment">// Individual settings</span>
            <span class="hljs-title">refineInterval</span> <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
            <span class="hljs-title">unrefineInterval</span> <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
            <span class="hljs-title">maxRefinementLevel</span> <span class="hljs-number">3</span><span class="hljs-punctuation">;</span>
            <span class="hljs-title">separateUpdates</span> <span class="hljs-keyword">false</span><span class="hljs-punctuation">;</span>
            ...
        <span class="hljs-punctuation">}</span>
        <span class="hljs-comment">// More refinement/unrefinement operations</span>
    <span class="hljs-punctuation">)</span><span class="hljs-punctuation">;</span>
<span class="hljs-punctuation">}</span>
{{< /foamCode >}}
</section>

<section data-auto-animate data-auto-animate-unmatched="false">
{{< foamCode "7,10-18" >}}
<span class="hljs-title">dynamicFvMesh</span> dynamicPolyRefinementFvMesh<span class="hljs-punctuation">;</span>
<span class="hljs-type">dynamicRefineFvMeshCoeffs</span> <span class="hljs-punctuation">{</span>
    <span class="hljs-comment">// 1. Global refinement settings</span>
    ...
    <span class="hljs-comment">// 2. Refinements description</span>
    <span class="hljs-title">refinements</span> <span class="hljs-punctuation">(</span>
        basedOnAlpha1 <span class="hljs-punctuation">{</span>
            <span class="hljs-comment">// Individual settings</span>
            ...
            <span class="hljs-comment">// Cell selection</span>
            <span class="hljs-type">refinementSelection</span> <span class="hljs-punctuation">{</span>
                <span class="hljs-title">type</span>           interfaceRefinement<span class="hljs-punctuation">;</span>
                <span class="hljs-title">fieldNames</span>     <span class="hljs-punctuation">(</span> alpha1 <span class="hljs-punctuation">)</span><span class="hljs-punctuation">;</span> 
                <span class="hljs-title">innerRefLayers</span> <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
                <span class="hljs-title">outerRefLayers</span> <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
                <span class="hljs-title">cellPointCellSmoothing</span> <span class="hljs-keyword">on</span><span class="hljs-punctuation">;</span>
            <span class="hljs-punctuation">}</span>
        <span class="hljs-punctuation">}</span>
        <span class="hljs-comment">// More refinement/unrefinement operations</span>
    <span class="hljs-punctuation">)</span><span class="hljs-punctuation">;</span>
<span class="hljs-punctuation">}</span>
{{< /foamCode >}}
</section>

---

### More refinement/unrefinement criteria

In addition to the interface refinement criterion, we can track:
- The discretisation error
- Interesting ranges of field _values, gradient_ and/or _curl_
- User-supplied (dynamic) code for refinement selection

---

There is also support for ”**mounting**” these criteria on top of each other
- Unrefinement won’t interfere with the refined (and to-be refined) cells in the same timeStep

---

### User-supplied code for refinement selection

{{< foamCode "1-6|9-15|17-29|31-43|45-51|54-56" >}}
<span class="hljs-type">refineBasedOnResiduals</span>
<span class="hljs-punctuation">{</span>
    <span class="hljs-title">refineInterval</span>   <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
    <span class="hljs-title">unrefineInterval</span> <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
    <span class="hljs-title">maxRefinementLevel</span>   <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
    <span class="hljs-title">separateUpdates</span> <span class="hljs-keyword">false</span><span class="hljs-punctuation">;</span>

    <span class="hljs-comment">// Refinement selection criteria</span>
    <span class="hljs-type">refinementSelection</span>
    <span class="hljs-punctuation">{</span>
        <span class="hljs-title">type</span>        codedFieldBoundsRefinement<span class="hljs-punctuation">;</span>
        <span class="hljs-title">fieldName</span>   magResError<span class="hljs-punctuation">;</span>
        <span class="hljs-title">lowerBound</span>  <span class="hljs-number">0</span><span class="hljs-punctuation">;</span>
        <span class="hljs-title">upperBound</span>  <span class="hljs-number">1</span><span class="hljs-punctuation">;</span>
        <span class="hljs-title">cellPointCellSmoothing</span> <span class="hljs-keyword">on</span><span class="hljs-punctuation">;</span>

        <span class="hljs-title">codeInclude</span>
        <span class="hljs-punctuation">#{</span>
            #include "errorEstimate.H"
            #include "resError.H"
<span class="hljs-punctuation">        #}</span><span class="hljs-punctuation">;</span>
        <span class="hljs-title">codeLibs</span>
        <span class="hljs-punctuation">#{</span>
            -lerrorEstimation
<span class="hljs-punctuation">        #}</span><span class="hljs-punctuation">;</span>
        <span class="hljs-title">codeOptions</span>
        <span class="hljs-punctuation">#{</span>
            -I$(LIB_SRC)/errorEstimation/lnInclude
<span class="hljs-punctuation">        #}</span><span class="hljs-punctuation">;</span>

        <span class="hljs-title">code</span>
        <span class="hljs-punctuation">#{</span>
            // Return immediately if 1st timeStep,
            if (mesh().time().timeIndex() <= 1)
            {
                return labelList().xfer();
            }
            // Get refs to involved fields
            const auto& phi = mesh().lookupObject<surfaceScalarField>("phi");
            const auto& U = mesh().lookupObject<volVectorField>("U");
            const auto& nu = mesh().lookupObject<volScalarField>("nu");
            const auto& p = mesh().lookupObject<volScalarField>("p");
            const auto& rho = mesh().lookupObject<volScalarField>("rho");

            // Estimate the error vector
            errorEstimate<vector> ee
            (
                resError::div(phi, U)
                - resError::laplacian(nu, U)
                + fvc::grad(p/rho)
            );
            volScalarField magResError = mag(ee.error());
            scalar maxResErr = gMax(magResError);
            field_.internalField() = magResError.internalField();   
            lowerBound_ = 0.8 * maxResErr;
            upperBound_ = maxResErr;
<span class="hljs-punctuation">        #}</span><span class="hljs-punctuation">;</span>
    <span class="hljs-punctuation">}</span>
<span class="hljs-punctuation">}</span>
{{< /foamCode >}}

---

{{< slide background-color="white" >}}

### Multi-Criteria Refinement process (per timeStep)

<section data-transition="fade-out">
    <img style="scale:0.85;" data-src="images/17.png">
</section>

<section data-transition="fade-in fade-out">
    <img style="scale:0.85;" data-src="images/18.png">
</section>

<section data-transition="fade-in fade-out">
    <img style="scale:0.85;" data-src="images/19.png">
</section>

---

### Multi-Criteria Refinement process (per timeStep)

<script type="text/javascript">
var items = [
  {
    type: "text",
    title: "Cell Level 0",
    description: "Initial cell size",
    position: {
      left: 170,
      top: 90
    }
  },
  {
    type: "text",
    title: "Cell Level 1",
    description: "",
    position: {
      left: 185,
      top: 180
    }
  },
  {
    type: "text",
    title: "Cell Level 2",
    description: "",
    position: {
      left: 250,
      top: 155
    }
  },
  {
    type: "text",
    title: "Cell Level 3",
    description: "Max refinement level",
    position: {
      left: 350,
      top: 170
    }
  },
  //{
  //  type: "provider",
  //  providerName: "youtube",
  //  parameters: {
  //    videoId: "iPRiQ6SBntQ"
  //  },
  //  position: {
  //    left: 300,
  //    top: 200
  //  },
  //  sticky: true
  //},
];

var options = {
  allowHtml: true
};

$(document).ready(function() {
  $("#my-interactive-image").interactiveImage(items, options);
});
</script>

<div id="my-interactive-image" style='margin: auto; width: 800px; height: 463px; border-radius: 8px; background-size: cover; background-image: url("images/levels.png");'></div>

---

# Load Balancing

---

### Load-Balance the mesh after topology changes

- Refinement methods generally "unbalance" local regions,
  generating load balancing issues.

- The Load-Balancer engine in Foam-Extend can auto-sense processor
  imbalance with respect to cell-count
    - Using a trigger coefficient

- It also tries to migrates adjacent cells between processors if possible

---

{{< foamCode "1-3,13|4|5-12" >}}
<span class="hljs-title">dynamicFvMesh</span> loadBalanceDynamicPolyRefinementFvMesh<span class="hljs-punctuation">;</span>
<span class="hljs-type">loadBalanceFvMeshCoeffs</span>
<span class="hljs-punctuation">{</span>
    <span class="hljs-title">imbalanceTrigger</span> <span class="hljs-number">0.2</span><span class="hljs-punctuation">;</span>
    <span class="hljs-title">numberOfSubdomains</span> <span class="hljs-number">4</span><span class="hljs-punctuation">;</span>
    <span class="hljs-title">method</span>          hierarchical<span class="hljs-punctuation">;</span>
    <span class="hljs-type">hierarchicalCoeffs</span>
    <span class="hljs-punctuation">{</span>
        <span class="hljs-title">n</span>       <span class="hljs-punctuation">(</span> <span class="hljs-number">1</span> <span class="hljs-number">2</span> <span class="hljs-number">2</span> <span class="hljs-punctuation">)</span><span class="hljs-punctuation">;</span>
        <span class="hljs-title">delta</span>   <span class="hljs-number">0.001</span><span class="hljs-punctuation">;</span>
        <span class="hljs-title">order</span>   xyz<span class="hljs-punctuation">;</span>
    <span class="hljs-punctuation">}</span>
<span class="hljs-punctuation">}</span>
{{< /foamCode >}}

--- 

### Load Balance chemistry problems

What Bulut Tekgul et al. did with **[DLBFoam](https://github.com/Aalto-CFD/DLBFoam)**:

- Load balance the number of **Chemistry ODE problems** across processes
    - Solved independently during each CFD iteration
- Employ reference solution mapping to (thermo-chemically) similar cells

---

<p style="text-align: center;">An example of DLBFoam run with the Hex-based adaptive refinement engine (courtesy of Bulut)</p>
<video style="scale:0.85; margin:0; padding:0;" data-autoplay src="/videos/dlbfoam.mp4"></video>

---

# Next steps

- Algebraic form of AMR 
    - Run AMR with Multi-Grid solvers fully algebraically?
