<div align="center">

<h1>Brand Guidelines Skill</h1>

<p><b>Applies Anthropic's official brand colors and typography to artifacts.</b></p>

</div>

<hr>

<h2>Skill Metadata</h2>

<pre>
---
name: brand-guidelines
description: Applies Anthropic's official brand colors and typography to any sort of artifact that may benefit from having Anthropic's look-and-feel. Use it when brand colors or style guidelines, visual formatting, or company design standards apply.
license: Complete terms in LICENSE.txt
---
</pre>

<hr>

<h2>Anthropic Brand Styling</h2>

<h3>Overview</h3>

<p>
To access Anthropic's official brand identity and style resources, use this skill.
</p>

<p><b>Keywords:</b></p>

<ul>
<li>branding</li>
<li>corporate identity</li>
<li>visual identity</li>
<li>post-processing</li>
<li>styling</li>
<li>brand colors</li>
<li>typography</li>
<li>Anthropic brand</li>
<li>visual formatting</li>
<li>visual design</li>
</ul>

<hr>

<h2>Brand Guidelines</h2>

<h3>Colors</h3>

<h4>Main Colors</h4>

<ul>
<li><b>Dark:</b> <code>#141413</code> — Primary text and dark backgrounds</li>
<li><b>Light:</b> <code>#faf9f5</code> — Light backgrounds and text on dark</li>
<li><b>Mid Gray:</b> <code>#b0aea5</code> — Secondary elements</li>
<li><b>Light Gray:</b> <code>#e8e6dc</code> — Subtle backgrounds</li>
</ul>

<h4>Accent Colors</h4>

<ul>
<li><b>Orange:</b> <code>#d97757</code> — Primary accent</li>
<li><b>Blue:</b> <code>#6a9bcc</code> — Secondary accent</li>
<li><b>Green:</b> <code>#788c5d</code> — Tertiary accent</li>
</ul>

<hr>

<h3>Typography</h3>

<ul>
<li><b>Headings:</b> Poppins (fallback: Arial)</li>
<li><b>Body Text:</b> Lora (fallback: Georgia)</li>
<li><b>Note:</b> Fonts should be pre-installed in the environment for best results</li>
</ul>

<hr>

<h2>Features</h2>

<h3>Smart Font Application</h3>

<ul>
<li>Applies <b>Poppins</b> font to headings (24pt and larger)</li>
<li>Applies <b>Lora</b> font to body text</li>
<li>Automatic fallback to Arial/Georgia if custom fonts are unavailable</li>
<li>Maintains readability across different systems</li>
</ul>

<hr>

<h3>Text Styling</h3>

<ul>
<li>Headings (24pt+): Poppins font</li>
<li>Body text: Lora font</li>
<li>Smart color selection depending on background</li>
<li>Preserves hierarchy and formatting</li>
</ul>

<hr>

<h3>Shape and Accent Colors</h3>

<ul>
<li>Non-text shapes use accent colors</li>
<li>Cycles through orange, blue, and green accents</li>
<li>Maintains visual interest while staying on-brand</li>
</ul>

<hr>

<h2>Technical Details</h2>

<h3>Font Management</h3>

<ul>
<li>Uses system-installed <b>Poppins</b> and <b>Lora</b> fonts when available</li>
<li>Fallbacks: <b>Arial</b> for headings and <b>Georgia</b> for body text</li>
<li>No font installation required</li>
<li>Best results achieved if Poppins and Lora are pre-installed</li>
</ul>

<hr>

<h3>Color Application</h3>

<ul>
<li>Uses RGB values for precise brand color matching</li>
<li>Applied using <code>python-pptx</code> RGBColor class</li>
<li>Ensures consistent color fidelity across systems</li>
</ul>
