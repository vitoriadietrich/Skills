<div align="center">

<h1>Algorithmic Art Skill</h1>

<p><b>Creating algorithmic art using p5.js with seeded randomness and interactive parameter exploration.</b></p>

</div>

<hr>

<h2>Skill Metadata</h2>

<pre>
---
name: algorithmic-art
description: Creating algorithmic art using p5.js with seeded randomness and interactive parameter exploration. Use this when users request creating art using code, generative art, algorithmic art, flow fields, or particle systems. Create original algorithmic art rather than copying existing artists' work to avoid copyright violations.
license: Complete terms in LICENSE.txt
---
</pre>

<hr>

<h2>Overview</h2>

<p>
Algorithmic philosophies are computational aesthetic movements that are then expressed through code.
</p>

<p>
Output files include:
</p>

<ul>
<li><b>.md</b> files (philosophy)</li>
<li><b>.html</b> files (interactive viewer)</li>
<li><b>.js</b> files (generative algorithms)</li>
</ul>

<p>This happens in two steps:</p>

<ol>
<li><b>Algorithmic Philosophy Creation</b> (.md file)</li>
<li><b>Express by creating p5.js generative art</b> (.html + .js files)</li>
</ol>

<hr>

<h2>Algorithmic Philosophy Creation</h2>

<p>
To begin, create an <b>algorithmic philosophy</b> that will be interpreted through computational systems:
</p>

<ul>
<li>Computational processes and emergent behavior</li>
<li>Seeded randomness and noise fields</li>
<li>Particle systems and force fields</li>
<li>Parametric variation and controlled chaos</li>
</ul>

<h3>Critical Understanding</h3>

<ul>
<li><b>Input:</b> subtle user instructions used as a conceptual seed</li>
<li><b>Creation:</b> a generative art philosophy</li>
<li><b>Next step:</b> algorithmic implementation in p5.js</li>
</ul>

<p>
The philosophy should guide the next system to build a generative algorithm where
<strong>90% of the outcome comes from algorithmic generation and only 10% from explicit parameters.</strong>
</p>

<hr>

<h2>Structure of the Philosophy</h2>

<h3>Name the Movement</h3>

<p>Examples:</p>

<ul>
<li>Organic Turbulence</li>
<li>Quantum Harmonics</li>
<li>Emergent Stillness</li>
</ul>

<h3>Articulate the Philosophy</h3>

<p>
Write <b>4–6 paragraphs</b> describing the computational aesthetic including:
</p>

<ul>
<li>Mathematical relationships</li>
<li>Noise functions</li>
<li>Particle behavior</li>
<li>Field dynamics</li>
<li>Temporal system evolution</li>
<li>Emergent complexity</li>
</ul>

<hr>

<h2>Critical Guidelines</h2>

<ul>
<li>Avoid redundancy in explanations.</li>
<li>Emphasize algorithm craftsmanship and expert-level implementation.</li>
<li>Allow creative interpretation for the algorithm stage.</li>
<li>Focus on processes rather than static images.</li>
</ul>

<p>
The algorithm must feel like the result of <b>deep computational expertise and meticulous refinement.</b>
</p>

<hr>

<h2>Essential Principles</h2>

<ul>
<li><b>Algorithmic Philosophy</b> — computational worldview expressed through code</li>
<li><b>Process Over Product</b> — beauty emerges during execution</li>
<li><b>Parametric Expression</b> — ideas through mathematical relationships</li>
<li><b>Artistic Freedom</b> — implementation left open for interpretation</li>
<li><b>Pure Generative Art</b> — living algorithms instead of static imagery</li>
<li><b>Expert Craftsmanship</b> — refined through countless iterations</li>
</ul>

<hr>

<h2>Conceptual Seed Deduction</h2>

<p>
Before writing the algorithm, identify a subtle conceptual thread hidden in the user request.
</p>

<p>
This concept should act as the <b>invisible DNA of the algorithm</b>, embedded in:
</p>

<ul>
<li>parameters</li>
<li>emergent behaviors</li>
<li>field dynamics</li>
<li>randomness structure</li>
</ul>

<p>
The reference should be subtle — recognizable only to those familiar with the concept.
</p>

<hr>

<h2>P5.js Implementation</h2>

<h3>Seeded Randomness</h3>

<pre>
let seed = 12345;
randomSeed(seed);
noiseSeed(seed);
</pre>

<h3>Parameter Structure</h3>

<pre>
let params = {
  seed: 12345,
  // algorithm parameters
};
</pre>

<p>
Parameters should control quantities, scale, probability, angles, thresholds, and other system properties.
</p>

<hr>

<h2>Canvas Setup</h2>

<pre>
function setup() {
  createCanvas(1200, 1200);
}

function draw() {
  // generative algorithm
}
</pre>

<hr>

<h2>Craftsmanship Requirements</h2>

<ul>
<li>Visual balance between complexity and order</li>
<li>Thoughtful color harmony</li>
<li>Strong compositional flow</li>
<li>Performance optimized for real-time rendering</li>
<li>Reproducible results using seeds</li>
</ul>

<hr>

<h2>Output Format</h2>

<ol>
<li><b>Algorithmic Philosophy</b> (.md explanation)</li>
<li><b>Single HTML Artifact</b> with embedded p5.js algorithm</li>
</ol>

<p>
The HTML artifact should contain everything inline and run instantly in any browser.
</p>

<hr>

<h2>Interactive Artifact Requirements</h2>

<ul>
<li>Parameter sliders</li>
<li>Color pickers</li>
<li>Seed navigation (prev / next / random)</li>
<li>Regenerate and reset buttons</li>
<li>Download PNG feature</li>
</ul>

<hr>

<h2>Single Artifact Structure</h2>

<pre>
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.min.js"></script>
</head>

<body>

<div id="canvas-container"></div>

<div id="controls">
  <!-- parameter controls -->
</div>

<script>
// full p5.js algorithm
</script>

</body>
</html>
</pre>

<hr>

<h2>Creative Workflow</h2>

<p>
User Request → Algorithmic Philosophy → Generative Implementation
</p>

<ol>
<li>Interpret the user request</li>
<li>Create the algorithmic philosophy</li>
<li>Implement the algorithm</li>
<li>Define parameters</li>
<li>Create UI controls</li>
</ol>

<hr>

<h2>Resources</h2>

<ul>
<li><b>templates/viewer.html</b> — base template for the interactive viewer</li>
<li><b>templates/generator_template.js</b> — reference for algorithm structure</li>
</ul>

<p>
Always use the template as the starting point and embed the final algorithm inline in the HTML artifact.
</p>
