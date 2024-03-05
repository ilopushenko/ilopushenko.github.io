---
layout: page
title: Polarization state
permalink: /projects/stokes
description: Calculator of the polarization ellipse parameters and other data obtainable from the user-defined Stokes vector.
img: assets/img/1_wasm_StokesVector/Poincare_sphere.svg
importance: 1
category: polarization
giscus_comments: true
related_publications: false
---

<!------------------------------------------------------------ preload WASM shared library ------------------------------------------------------------>
<script async type="text/javascript" src="/wasm/wasm_StokesLibrary.js"></script>
<script>
	// defer WASM library load
	var Module = {
            onRuntimeInitialized: function () {
                // Run Function
                Module._wasm_StokesLibrary_initialize();
                Module._wasm_initStokesObject();
                //Module._mylibrary_terminate();
            }
        };
</script>
<!------------------------------------------------------------ preload PLOTLY shared library ------------------------------------------------------------>
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>


<!------------------------------------------------------------ PAGE CONTENT: introduction ------------------------------------------------------------>
This is a small app to compute and visualize parameters of either fully or partially polarized light from the supplied Stokes vector $$ S=[S_0, S_1, S_2, S_3]^T=[I, Q, U, V]^T $$. Source code for the underlying MATLAB® class will be available on GitHub and MATLAB Exchange. <!-- Examples provided below can also be launched in GNU Octave, particularly in its web version. -->

In the following, we fully comply with [M. Born and E. Wolf (1980)](#further-reading), sections §1.4 and §10.8. <!-- in particular, decision and ispiration comes from .... --> For more details, please refer to these chapters or see theoretical reference at the bottom of the page.<!--, see theoretical reference at the bottom of the page, and/or have a look at the blog post. -->

Some short remarks before you begin working with the app:

<div id="showBtnLoadDemo"></div> <!-- just for the hyperlink to work. impossible to assign id to the button because then hyperlink jumps lower due to the page header -->

* Input are Stokes vector values (can be power in e.g. [W/m<sup>2</sup>], or in normalized dimensionless units). The main rule is that relations $$S_0^2\geq S_1^2+S_2^2+S_3^2$$ and $$S_0\geq0$$ must always hold;
* Several input vectors can be provided to compare them;
* Primary outputs are degrees of polarization and parameters of the polarization ellipse;
* Additional outputs are 6 intensity values $$I_H, I_V, I_D, I_A, I_R, I_L$$ (accessible via tab switch below) <!--, Wolf's coherency matrix $$\mathbf{J}$$--> and normalized Stokes vector values (accessible via Poincaré sphere);
* Computed ellipse and Poincaré sphere depiction of the $$S$$ vector are presented on 2D and 3D plots below.
<!-- * It is also possible to use 6 intensity measurements OR normalized Stokes vectors with DOP and power as input parameters. Submitted data is preserved when switching tabs. -->


<!------------------------------------------------------------ PAGE CONTENT: application ------------------------------------------------------------>
<!-- button style: https://www.w3schools.com/bootstrap/bootstrap_buttons.asp -->
<div align="right">
<button class="btn btn-info btn-sm" onclick="loadDemoData()">Load demo data</button> 
<!-- <button class="btn btn-primary btn-sm">Load from file</button> 
<button class="btn btn-primary btn-sm">Save to file</button> -->
<button class="btn btn-danger btn-sm" onclick="resetData()">Reset data</button> 
</div>

<!-- Tab switcher -->
<div class="tab">
<div class="row"> <!-- justify-content-center"> -->
  <div class="col-sm"><button class="tablinks" onclick="openTab(event, 'ArbitraryStokes')" id="mainTab">Stokes vector</button></div>
  <!-- <div class="col-sm"><button class="tablinks" onclick="openTab(event, 'NormalizedStokes')">Stokes vector (normalized)</button></div> -->
  <div class="col-sm"><button class="tablinks" onclick="openTab(event, 'IntensityDiv')">Intensity values</button></div>
  <!-- <div class="col-sm"><button class="tablinks" onclick="openTab(event, 'CoherencyDiv')">Wolf's coherency matrix</button></div> -->
</div>
</div>

<!-- Content of the 1st tab: to input arbitrary (custom) Stokes vector values -->
<div id="ArbitraryStokes" class="tabcontent">
<form name='StokesCustom'>
	<table cellspacing="0" class="table-sm table-striped" width="100%" id="customStokesTable">
	<thead>
       <tr>
	      <th style="width:10%">ID</th>
          <th style="width:15%">$$S_0$$</th>
          <th style="width:15%">$$S_1$$</th>
          <th style="width:15%">$$S_2$$</th>
		  <th style="width:15%">$$S_3$$</th>
		  <th style="width:20%"></th>
		  <th style="width:10%"></th>
       </tr>
	</thead>
	<tr style="display:none;">
		<th style="width:10%"></th>
          <th style="width:15%"></th>
          <th style="width:15%"></th>
          <th style="width:15%"></th>
		  <th style="width:15%"></th>
		  <th style="width:20%"></th>
		  <th style="width:10%"></th>
	</tr>
    </table>  
</form>
</div>

<!-- Content of the 2nd tab: to input normalized Stokes vector values  -->
<div id="NormalizedStokes" class="tabcontent">
<form name='StokesNormalizedForm'>
	<table cellspacing="0" class="table-bordered table-sm table-striped" width="100%" id="normalizedStokesTable">
	<thead>
       <tr>
	      <th style="width:10%">ID</th>
          <th style="width:15%">Power</th>
          <th style="width:15%">$$DoP$$</th>
          <th style="width:15%">$$s_1$$</th>
		  <th style="width:15%">$$s_2$$</th>
		  <th style="width:15%">$$s_3$$</th>
		  <th style="width:10%"></th>
		  <th style="width:5%"></th>
       </tr>
	</thead>
	<tr style="display:none;">
		  <th style="width:10%"></th>
          <th style="width:15%"></th>
          <th style="width:15%"></th>
          <th style="width:15%"></th>
		  <th style="width:15%"></th>
		  <th style="width:15%"></th>
		  <th style="width:10%"></th>
		  <th style="width:5%"></th>
	</tr>
    </table>  
</form>
</div>

<!-- Content of the 3rd tab: to input intensity values  -->
<div id="IntensityDiv" class="tabcontent">
<form name='IntensityForm'>
	<table cellspacing="0" class="table-striped table-condensed table-sm" width="100%" id="intensityTable">
	<thead>
       <tr>
	      <th style="width:10%">ID</th>
          <th style="width:15%">$$I_H$$</th>
          <th style="width:15%">$$I_V$$</th>
          <th style="width:15%">$$I_D$$</th>
		  <th style="width:15%">$$I_A$$</th> 
		  <th style="width:15%">$$I_R$$</th>
		  <th style="width:10%">$$I_L$$</th>
		  <th style="width:5%"></th>
       </tr>
	</thead> 
	<tr style="display:none;">
		  <th style="width:10%"></th>
          <th style="width:15%"></th>
          <th style="width:15%"></th>
          <th style="width:15%"></th>
		  <th style="width:15%"></th> 
		  <th style="width:15%"></th>
		  <th style="width:10%"></th>
		  <th style="width:5%"></th>
	</tr>
    </table>  
</form>
</div>

<!-- Content of the 4th tab: to input coherency matrix  -->
<div id="CoherencyDiv" class="tabcontent">
<form name='CoherencyForm'>
	<table cellspacing="0" class="table-bordered table-sm table-striped" width="100%" id="coherencyTable">
	<thead>
       <tr>
	      <th style="width:10%">ID</th>
          <th style="width:50%">$$\mathbf{J}$$</th>
		  <th style="width:20%"></th>
		  <th style="width:20%"></th>
       </tr>
       <!-- <tr>
	      <td>0</td>
		  <td>[ вот здесь матрица в формате div ??? ]</td>
		  <td><button onclick="myCreateFunction()" class="btn btn-info btn-xs">+</button></td>   
		  <td><button onclick="myDeleteFunction()" class="btn btn-danger btn-xs">-</button></td>
       </tr> -->
	</thead>   
	<tr style="display:none;">
		  <th style="width:10%"></th>
          <th style="width:50%"></th>
		  <th style="width:20%"></th>
		  <th style="width:20%"></th>
	</tr>
    </table>  
</form>
</div>

<!-- Results table  -->
<!-- https://www.w3schools.com/bootstrap/bootstrap_tables.asp -->
<!-- CONDENSED TABLE worked !!!!!!!!! and also inside container and responsive !!! -->
<div class="caption">Attention: tabs are simultaneously updated with the latest submitted data. <br /> Main results are returned in the table below.</div>
<table cellspacing="0" class="table-bordered table-sm table-striped table-condensed" id="resultsTable"> <!-- class="table-hover table table-condensed table-bordered"> data-toggle="table" data-click-to-select="true" data-search="true" -->
<thead>
		<tr>
			<th style="width:10%">ID</th>
			<th style="width:10%">$$DoP$$</th>
			<th style="width:10%">$$DoLP$$</th>
			<th style="width:10%">$$DoCP$$</th>
			<th style="width:10%"><div class="row justify-content-center"><div class="col-6">$$\mathrm{Azimuth}\:\psi$$</div><div class="col-1" data-tooltip="$$0\leq\psi<\pi$$">&#9432;</div></div></th>
			<th style="width:10%"><div class="row justify-content-center"><div class="col-6">$$\mathrm{Ellipticity}\:\chi$$</div><div class="col-1" data-tooltip="$$-\pi/4\leq\chi\leq\pi/4$$">&#9432;</div></div></th>
			<th style="width:10%"><div class="row justify-content-center"><div class="col-1">$$\delta$$</div><div class="col-1" data-tooltip="Phase difference">&#9432;</div></div></th>
			<th style="width:10%" data-tooltip="$$0\leq\psi<\pi$$">$$\mathrm{Helicity}$$</th>  <!-- $$\delta$$ -->
		</tr>
</thead>
	<tr style="display:none;">
		<td style="width:10%"></td>
		<td style="width:10%"></td>
		<td style="width:10%"></td>
		<td style="width:10%"></td>
		<td style="width:10%"></td>
		<td style="width:10%"></td>
		<td style="width:10%"></td>
		<td style="width:10%"></td>
	</tr>
</table>
<br />

<!-- Plotly graphs -->    
<div class="row">
	<div class="row">
		<div class="col-sm">
			<div id="ellipsePlot"></div> <br />
			<div class="caption">Figure 1. <a href="#showEllipseDefinition">Polarization ellipse</a> for each input beam.</div>
		</div>
		<div class="col-sm">
			<div id="spherePlot"></div> <br />
			<div class="caption">Figure 2. Depiction of the normalized Stokes vector on the <a href="#showEllipseDefinition">Poincaré sphere</a>.</div>
		</div>
	</div>
</div>


## Theoretical reference
#### Measurement of Stokes vector
Using polarizer-compensator configuration (experimental setup with e.g. [linear polarizer](https://en.wikipedia.org/wiki/Polarizer#Linear_polarizers) and [quarter-wave plate](https://en.wikipedia.org/wiki/Waveplate#Quarter-wave_plate)) it is possible to measure several possible [intensity values](https://en.wikipedia.org/wiki/Intensity_(physics)) of the light beam: intensity of the horizontally polarized component $$I_H$$, vertically polarized component $$I_V$$, diagonally polarized component $$I_D$$, antidiagonal component $$I_A$$, right $$I_R$$ and left $$I_L$$ circularly polarized components. This is in agreement with [Born and Wolf](#further-reading), §10.8, Eqns. (10) and (64). If light is unpolarized, then all components are equal to each other.

In the literature, there are several other notations for these intensity values, introduced by different authors with different backrounds and targeting different purposes. A good review is done by [M. Mishchenko (2014)](#further-reading) and [D. Goldstein (2011)](#further-reading). Most notably, in the experimental configuration with polarizer and compensating waveplate one would usually use angles $$\theta$$ and $$\varepsilon$$ of their rotation in order to describe the measured intensities: $$I=I(\theta,\varepsilon)$$. Then,

$$I_H=I(0,0), \:\: I_V=I\left(\dfrac{\pi}{2},0\right),$$ 

$$I_D=I\left(\dfrac{\pi}{4},0\right), \:\: I_A=\left(\dfrac{3\pi}{4},0\right),$$

$$I_R=I\left(\dfrac{\pi}{4},\dfrac{\pi}{2}\right), \:\: I_L=I\left(\dfrac{3\pi}{4},\dfrac{\pi}{2}\right).$$

Finally, Stokes vector can be computed as

$$S_0=I_H+I_V=I_D+I_A=I_R+I_L,$$

$$S_1=I_H-I_V,$$

$$S_2=I_D-I_A,$$

$$S_3=I_R-I_L.$$

{% details Click here to know more %} 
In fact, only 4 intensity measurements are enough to compute the Stokes vector. All modern polarimeters, commercial and in-house alike, both well-developed and prototyped with account for new technological advances, rely on these equations - in one way or another. 

> ##### WARNING
>
> In this app, currently there is no way to input intensity measurements. Right now, they are only recomputed from the Stokes vector provided by the user.
{: .block-warning }

{% enddetails %}


#### Degrees of polarization
Partially polarized light is characterized by its degree of polarization ($$DoP$$), [Born and Wolf](#further-reading), §10.8, Eqn. (68):

$$DoP=\sqrt{S_1^2+S_2^2+S_3^2}/S_0.$$

In turn, it can be further decomposed into degrees of linear ($$DoLP$$) and circular ($$DoCP$$) polarization:

$$DoLP=\sqrt{S_1^2+S_2^2}/S_0,$$

$$DoCP=\sqrt{S_3^2}/S_0=\vert S_3 \vert/S_0,$$

$$DoP^2=DoLP^2 + DoCP^2.$$

<!-- For more details on the given equations, please refer e.g. to [E. Collett]() [R. M. A. Azzam and N. M. Bashara (1977)], §, Eqn. (). --> Degrees of polarization are quite frequently employed in polarimetric applications. <!-- with one of the most typical scenarios ... . -->

#### Polarization ellipse and Poincaré sphere
Fully polarized light can be characterized with [polarization ellipse](https://en.wikipedia.org/wiki/Polarization_(waves)#Polarization_ellipse). 

> ##### TIP
>
> In case of the partially polarized light, this app plots polarization ellipse for the fully polarized component of light, which intensity is equal to $$\sqrt{S_1^2+S_2^2+S_3^2}$$. Here, your data is not normalized in any way, and ellipses for all provided Stokes vectors are displayed with proper scale, which allows to compare them between each other both in terms of ellipse form and amplitudes of the underlying light waves.
{: .block-tip }

Each ellipse is a result of a superposition of two orthogonally linearly polarized waves with phase difference $$\delta$$ between them. Assuming that we deal with plane time-harmonic wave propagating along $$z$$ direction, the electromagnetic wave field is then described as $$\mathbf{E}=(E_x,E_y,0)$$, [Born and Wolf](#further-reading), §1.4, Eqn. (12):

$$E_x = a_1 \cos{(\tau+\delta_1)}, \:\: E_y = a_2 \cos{(\tau+\delta_2)},$$ 

$$\tau=\omega t - \mathbf{kr}, \:\: \delta = \delta_2-\delta_1.$$

Here, amplitudes $$a_1$$ and $$a_2$$ of both orthogonally polarized light waves are introduced. We would allow ourselves not to comment upon all other variables, directing the inquiring reader to the given reference for further reading. 

<div id="showEllipseDefinition"></div>

Depending on the amplitudes and phase difference, polarization ellipse can take different forms. Its axes are generally not parallel to either $$Ox$$ or $$Oy$$ of the Cartesian coordinate system, and can be rotated by azimuth angle $$\psi$$ with respect to them (see Figure 3): 


<div class="row">
	<div class="row">
		<div class="col-sm justify-content-center text-center">
			<img src="/assets/img/1_wasm_StokesVector/Polarisation_ellipse2.svg" class="img-fluid rounded z-depth-1" width="70%" height="auto" title="Polarization ellipse" style="background-color: white;"><br />
			<div class="caption">Figure 3. Polarization ellipse and its parameters. <!--2a corresponds to the red ellipse axes, 2b corresponds to the blue axes, arrow-->Arrow corresponds to the helicity direction. Picture source: <a href="https://en.wikipedia.org/wiki/Polarization_(waves)#Polarization_ellipse">wiki</a>.</div>
		</div>
		<div class="col-sm justify-content-center text-center">		
			<img src="/assets/img/1_wasm_StokesVector/Poincare_sphere.svg" class="img-fluid rounded z-depth-1" width="70%" height="auto" title="Poincaré sphere" style="background-color: white;"><br />
			<div class="caption">Figure 4. Definition of the Poincaré sphere. Picture source: <a href="https://en.wikipedia.org/wiki/Unpolarized_light#Poincar%C3%A9_sphere">wiki</a>.</div>
		</div>
	</div>
</div>


<!-- ![Polarization ellipse](/assets/img/1_wasm_StokesVector/Polarisation_ellipse2.svg) -->
<!--

<div class="row justify-content-sm-center">
<div class="col-">

</div>
</div>
<div class="caption">Figure 3. Polarization ellipse and its parameters. 2a corresponds to the red ellipse axes, 2b corresponds to the blue axes, arrow corresponds to the helicity direction. Picture source: <a href="https://en.wikipedia.org/wiki/Polarization_(waves)#Polarization_ellipse">wiki</a>.</div>

-->

Larger $$a$$ and smaller $$b$$ ellipse semi-axes, as well as ellipticity $$\chi$$ and azimuth $$\psi$$ are related to Stokes vector in the following way, Born and Wolf §1.4, Eqns. (23),(32),(43),(45a)-(45c):

$$S_0=a_1^2+a_2^2=a^2+b^2,$$

$$S_1=S_0\cos2\chi\cos2\psi,$$

$$S_2=S_0\cos2\chi\sin2\psi,$$

$$S_3=S_0\sin2\chi,$$

$$\mathrm{tg}\chi=\mp b/a.$$

These relations, resembling [connection](https://en.wikipedia.org/wiki/Spherical_coordinate_system#Cartesian_coordinates) between [Cartesian](https://en.wikipedia.org/wiki/Cartesian_coordinate_system) and [spherical](https://en.wikipedia.org/wiki/Spherical_coordinate_system) coordinate systems, allow to depict Stokes vector as point on the sphere. This depiction of the polarization state is known as [Poincaré sphere](https://en.wikipedia.org/wiki/Unpolarized_light#Poincar%C3%A9_sphere).

<!--
Thus, while it is not possible to definitely/uniquely determine amplitudes $$a_1$$ and $$a_2$$ from the Stokes vector, it is still possible to obtain a unique polarization ellipse for any correctly defined set of $$S_0,S_1,S_2,S_3$$. --> <!-- Work with separate wave amplitudes is covered by a Plane Wave tool. -->

Obvious special cases are the so-called degenerate states of the polarization ellipse, which are available in the current tool via ["Load demo data"](#showBtnLoadDemo) button. In particular, in the 1st degenerate state (Linear Horizontal Polarization, LHP) ellipse is reduced to the line aligning with $$Ox$$ Cartesian axes, in the 2nd (Linear Vertical Polarization, LVP) - to the line aligning with $$Oy$$ Cartesian axes, in the 3rd (linear diagonal polarization, L+45) and 4th (linear antidiagonal polarization L-45) - to the lines rotated at $$\psi_D=\pi/4$$ and $$\psi_A=3\pi/4$$, correspondingly, and finally, in 5th (right circular polarization, RCP) and 6th (left circular polarization, LCP) ellipse is reduced to the circle which one can follow either clockwise or counter-clockwise.


<!--
While amplitudes of the light can not be однозначно determined from the Stokes vector measurements, ???their relative interplay is evaluated via 

...

Here, we have introduced $$a$$, $$b<a$$, $$\psi$$, $$\chi$$. All these parameters have clear geometric interpretation:

<picture of ellipse with everything>

It is possible to further dive in the topic with considering elliptically polarized wave as superposition of the two linearly polarized waves, but this is already outside the scope of the current base web app.
-->

#### Wolf's coherency matrix
It is worth noting that Stokes parameters bijectively (one-to-one) correspond to the coherency matrix introduced by Emil Wolf and which can be referenced in Born and Wolf, §10.8, Eqn. (63). In turn, this matrix is directly related to the concept of density matrix in quantum description of light: Wolf's matrix is in fact <!-- <can be treated as> --> a density matrix of the single photon. This is an important connection between wave and quantum properties of light which manifests itself through polarization. In the source code of this app, elements of the Wolf's coherency matrix are also evaluated, and there are plans on implementing web interface for the convenient coherency matrix analysis in this app - in the tab next to intensity values. For now, we conclude our short theoretical overview and provide all necessary references. 
<!-- если прога будет сама сразу говорить, что это за свет, и бывает ли такой, и если не бывает, то почему - наверное это будет круто -->

#### Further reading
1. [M. Born and E. Wolf. Principles of Optics, 6th Edition. Pergamon Press (1980).](https://en.m.wikipedia.org/wiki/Principles_of_Optics#:~:text=Principles%20of%20Optics%2C%20colloquially%20known,in%201959%20by%20Pergamon%20Press) <!-- https://www.sciencedirect.com/book/9780080264820/principles-of-optics#book-info -->
2. [M. I. Mishchenko. Electromagnetic Scattering by Particles and Particle Groups: An Introduction. Cambridge University Press (2014).](https://www.cambridge.org/9780521519922)
3. [D. H. Goldstein. Polarized Light, 3rd Edition. CRC Press (2011).](https://doi.org/10.1201/b10436)
4. Wiki: numerous in-line references to Wikipedia articles all around this page.
5. [MATLAB reference on polarization.](https://se.mathworks.com/help/phased/ug/polarized-fields.html)

In any of the provided web resources, I recommend to strongly beware of typos: unfortunately, these are still quite common in polarization-related articles even in help resources for the proprietary products. This is mostly connected to either sign uncertainty, chosen form of the harmonic wave time dependence, or to the different notations used by various authors to characterize polarization ellipse and other light parameters.


## Source code (available soon)

* MATLAB Exchange
* GitHub
* Octave Online


<div class="caption">Polarization state calculator. Preview version 0.3, 05.03.2024. <br /> 
This tool has been inspired by fundamental relation between polarization and quantum properties of light, and by processing of polarimetric measurements data at <a href="https://www.oulu.fi/en/research-groups/biophotonics">OPEM Unit</a> within <a href="https://ieeexplore.ieee.org/document/9622320">Academy of Finland</a> and <a href="https://www.sintef.no/en/projects/2022/metahilight/">MetaHiLight</a> projects. <br />
Author warmly acknowledges support and feedback from colleagues and friends. <br />
Built with <a href="https://www.mathworks.com/matlabcentral/fileexchange/69973-generatejavascriptusingmatlabcoder">MATLAB Coder and Emscripten</a>. Runs locally in the web browser with <a href="https://webassembly.org">WebAssembly</a>. </div>

<!--
[OPEM Unit](https://www.oulu.fi/en/research-groups/biophotonics)
[AoF](https://ieeexplore.ieee.org/document/9622320)
[MetaHiLight](https://www.sintef.no/en/projects/2022/metahilight/)
-->


<!------------------------------------------------------------ ADDITIONAL STYLES ------------------------------------------------------------>
<!-- hovering tooltips -->
<style>
    .tooltipHovering {
      position: fixed;
      padding: 10px 20px;
      border: 1px solid #b3c9ce;
      border-radius: 4px;
      text-align: center;
      font: italic 14px/1.3 sans-serif;
      color: var(--global-text-color);
      background: var(--global-bg-color);
      box-shadow: 3px 3px 3px rgba(0, 0, 0, .3);
    }
</style>
<!-- color: var(--global-theme-color); -->
<!-- color: #333;        background: #fff;    -->

<!-- tab styles -->
<style>
/* Style the tab */
.tab {
    overflow: hidden;
    border: none;
    background-color: var(--global-card-bg-color);
	//margin: auto;
	float: left;
    width: 50%;
	padding: 10px;
}
/* border: 1px solid var(--global-text-color-light); */

/* Style the buttons inside the tab */
.tab button {
    background-color: inherit;
    float: left;
    border: none;
    outline: none;
    cursor: pointer;
    padding: 14px 16px;
    transition: 0.3s;
    font-size: 16px;
	color: var(--global-text-color);
}

/* Change background color of buttons on hover */
.tab button:hover {
    background-color: var(--global-footer-text-color);
}

/* Create an active/current tablink class */
.tab button.active {
    background-color: var(--global-footer-text-color);
}

/* Style the tab content */
.tabcontent {
    display: none;
    padding: 6px 12px;
	border: none;
    border-top: none;
}
/* border: 1px solid var(--global-text-color-light); */
</style>



<!-------------------------------------------------------------------------------- JAVASCRIPTs -------------------------------------------------------------------------------->
<!---------------------------- TABS: https://html5css.ru/howto/howto_js_tabs.php ---------------------------->
<script>
function openTab(evt, tabName) {
    var i, tabcontent, tablinks;
	
    tabcontent = document.getElementsByClassName("tabcontent");
    for (i = 0; i < tabcontent.length; i++) {
        tabcontent[i].style.display = "none";
    }
    tablinks = document.getElementsByClassName("tablinks");
    for (i = 0; i < tablinks.length; i++) {
        tablinks[i].className = tablinks[i].className.replace(" active", "");
    }
    document.getElementById(tabName).style.display = "block";
    evt.currentTarget.className += " active";
}
</script>


<!---------------------------- HOVERING TOOLTIPS: https://learn.javascript.ru/task/behavior-tooltip  ---------------------------->
<script>
		let tooltipElem;

		document.onmouseover = function(event) {
		  //console.log('detected');
		  let target = event.target;

		  // если у нас есть подсказка...
		  let tooltipHtml = target.dataset.tooltip;
		  if (!tooltipHtml) return;

		  // ...создадим элемент для подсказки
		  tooltipElem = document.createElement('div');
		  tooltipElem.className = 'tooltipHovering';
		  tooltipElem.id = 'tooltipWithFormula';
		  
		  tooltipElem.innerHTML = tooltipHtml;
		  document.body.append(tooltipElem);

		  // спозиционируем его сверху от аннотируемого элемента (top-center)
		  let coords = target.getBoundingClientRect();

		  let left = coords.left + target.offsetWidth + 5; //(target.offsetWidth - tooltipElem.offsetWidth) / 2;
		  if (left < 0) left = 0; // не заезжать за левый край окна

		  let top = coords.top - tooltipElem.offsetHeight - 5;
		  if (top < 0) { // если подсказка не помещается сверху, то отображать её снизу
			top = coords.top + target.offsetHeight + 5;
		  }

		  tooltipElem.style.left = left + 'px';
		  tooltipElem.style.top = top + 'px';
		  //console.log(top);
		  
		  // https://www.sqlpac.com/en/documents/mathjax-advanced-javascript-rendering-methods.html
		  var HUB = window.MathJax;
		  node = document.getElementById('tooltipWithFormula');
		  MathJax.typeset([node]);
		};

		document.onmouseout = function(e) {
		  if (tooltipElem) {
			tooltipElem.remove();
			tooltipElem = null;
		  }
    };
</script>


<!---------------------------- HELPERS ---------------------------->
<script>
function determineHelicity(S3) {
	let out;
	if (S3>0){
		out = "Right";
	} else if (S3<0) {
		out = "Left";
	} else {
		out = "Linear";
	}
	return out;
}

function deletePlaceholderRows() {
	document.getElementById("customStokesTable").deleteRow(1);
	document.getElementById("normalizedStokesTable").deleteRow(1);
	document.getElementById("intensityTable").deleteRow(1);
	document.getElementById("coherencyTable").deleteRow(1);
	//document.getElementById("resultsTable").deleteRow(1);
}

function updateRowIds(table) {
  let rows = table.rows;//[idx];
  //x.cells[0].innerHTML = idx;
  rows.forEach((row) => {
    //temp = `(row #${row.rowIndex})`;
    row.cells[0].innerHTML = row.rowIndex;
  });
  rows[0].cells[0].innerHTML = "ID";
  
  //console.log(table.id);
  if (table.id!='resultsTable')
    rows[rows.length-1].cells[0].innerHTML = "";
}

function deleteRowAction(event) {
  event.preventDefault();
  let idx = event.target.closest('tr').rowIndex;//Number(event.target.name);
  if (document.getElementById("customStokesTable").rows.length==2){
	  alert('Impossible to delete the only row present in the table: there must be at least one row with data.');
  } else {
	  document.getElementById("resultsTable").deleteRow(idx);  
	  document.getElementById("customStokesTable").deleteRow(idx);
	  document.getElementById("normalizedStokesTable").deleteRow(idx);
	  document.getElementById("intensityTable").deleteRow(idx);
	  document.getElementById("coherencyTable").deleteRow(idx);
	  
	  updateRowIds(document.getElementById("resultsTable"));
	  updateRowIds(document.getElementById("customStokesTable"));
	  updateRowIds(document.getElementById("normalizedStokesTable"));
	  updateRowIds(document.getElementById("intensityTable"));
	  updateRowIds(document.getElementById("coherencyTable"));
	  
	  Plotly.deleteTraces(ellipsePlot, idx-1);
	  Plotly.deleteTraces(spherePlot, idx+1);
	  
	  let maxIDX = document.getElementById("resultsTable").rows.length-2;
	  let temp;
	  let j;
	  for (let i = 0; i < maxIDX; i++) {
		  j = i+1;
		  temp = {name: 'id ' + j};
		  Plotly.restyle(ellipsePlot, temp, i);
		  Plotly.restyle(spherePlot, temp, i+2);
	  }

	  Module._wasm_removeSelectedData(idx); // element of WASM interaction here !!
  }
}

function resetData() {
	let maxIDX = document.getElementById("resultsTable").rows.length-2;
	//console.log(maxIDX);
	for (let i = maxIDX; i>0; i--) {
		//console.log(i);
		document.getElementById("resultsTable").deleteRow(i);  
		document.getElementById("customStokesTable").deleteRow(i);
		document.getElementById("normalizedStokesTable").deleteRow(i);
		document.getElementById("intensityTable").deleteRow(i);
		document.getElementById("coherencyTable").deleteRow(i);
		Module._wasm_removeSelectedData(i); // element of WASM interaction here !!
	}
	initializeEllipsePlot();
	initializeSpherePlot();
}

function loadDemoData() {
	StokesForm.S0.value = 1;
	StokesForm.S1.value = 1;
	StokesForm.S2.value = 0;
	StokesForm.S3.value = 0;
	document.getElementById("inputCustomStokes").click();
	
	StokesForm.S0.value = 1;
	StokesForm.S1.value = -1;
	StokesForm.S2.value = 0;
	StokesForm.S3.value = 0;
	document.getElementById("inputCustomStokes").click();
	
	StokesForm.S0.value = 1;
	StokesForm.S1.value = 0;
	StokesForm.S2.value = 1;
	StokesForm.S3.value = 0;
	document.getElementById("inputCustomStokes").click();
	
	StokesForm.S0.value = 1;
	StokesForm.S1.value = 0;
	StokesForm.S2.value = -1;
	StokesForm.S3.value = 0;
	document.getElementById("inputCustomStokes").click();
	
	StokesForm.S0.value = 1;
	StokesForm.S1.value = 0;
	StokesForm.S2.value = 0;
	StokesForm.S3.value = 1;
	document.getElementById("inputCustomStokes").click();
	
	StokesForm.S0.value = 1;
	StokesForm.S1.value = 0;
	StokesForm.S2.value = 0;
	StokesForm.S3.value = -1;
	document.getElementById("inputCustomStokes").click();
}

function initializeEllipsePlot() {
	// https://plotly.com/javascript/line-charts/
	//var scale = Math.max(out2[5],out2[6]); // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/max
	//var scale = S0;
	let layout = {
		title: 'Polarization Ellipse',
		autosize: false,
		width: 350,
		height: 350,
		showlegend: true,
		colorway : [
			'#1f77b4',  // muted blue
			'#ff7f0e',  // safety orange
			'#2ca02c',  // cooked asparagus green
			'#d62728',  // brick red
			'#9467bd',  // muted purple
			'#8c564b',  // chestnut brown
			'#e377c2',  // raspberry yogurt pink
			'#7f7f7f',  // middle gray
			'#bcbd22',  // curry yellow-green
			'#17becf',   // blue-teal
		],//['#f3cec9', '#e7a4b6', '#cd7eaf', '#a262a9', '#6f4d96', '#3d3b72', '#182844'],
		xaxis: {
			//range: [-scale-0.1*scale, scale+0.1*scale],
			//autorange: false
		},
		yaxis: {
			scaleanchor: "x",
			//range: [-scale-0.1*scale, scale+0.1*scale],
			//autorange: false
		}
	};
	// https://plotly.com/javascript/configuration-options/
	Plotly.newPlot( ellipsePlot, [], layout, {scrollZoom: true, displaylogo: false, responsive: true});//{margin: { t: 0 } } );
}

function initializeSpherePlot() {
	// https://plotly.com/javascript/colorway/	
	let layoutSPH = {
		title: 'Poincaré Sphere',
		//autosize: false,
		width: 350,
		height: 350,
		showlegend: true,
		colorway : [
			'#bcbd22',  // curry yellow-green
			'#17becf',  // blue-teal
			'#1f77b4',  // muted blue
			'#ff7f0e',  // safety orange
			'#2ca02c',  // cooked asparagus green
			'#d62728',  // brick red
			'#9467bd',  // muted purple
			'#8c564b',  // chestnut brown
			'#e377c2',  // raspberry yogurt pink
			'#7f7f7f',  // middle gray
		],//['#f3cec9', '#e7a4b6', '#cd7eaf', '#a262a9', '#6f4d96', '#3d3b72', '#182844'],
		//margin: {
		//	l: 65,
		//	r: 50,
		//	b: 65,
		//	t: 90,
		//}
		scene: {
			xaxis:{title: 'x=s1'},
			yaxis:{title: 'y=s2'},
			zaxis:{title: 'z=s3'},
			},
		};
	let sphereMesh = computeSphere();
	//sphereMesh.push(trace1);
	//console.log(sphereMesh);
	Plotly.newPlot(spherePlot, sphereMesh, layoutSPH);
}

function computeSphere(){
	a = [];
	b = [];
	c = [];
	phiArr = [];
	thetaArr = [];
	function makeInterval(startValue, stopValue, numPoints) {
		var arr = [];
		var step = (stopValue - startValue) / (numPoints - 1);
		for (var i = 0; i < numPoints; i++) {
		  arr.push(startValue + (step * i));
		}
		return arr;
	  }

	////////////////
	// EDIT 1: calculate only upper half of the sphere

	phiArr = makeInterval(0, Math.PI/2, 20);  
	///////////////
	thetaArr = makeInterval(0, 2*Math.PI, 20); 

	for (i=0; i<thetaArr.length; i++){
		for (j=0; j<phiArr.length; j++){
			a.push(Math.cos(thetaArr[i]) * Math.sin(phiArr[j]));
			b.push(Math.sin(thetaArr[i]) * Math.sin(phiArr[j]));   
			c.push(Math.cos(phiArr[j]));
		}
	}

	const dataitem = {
		opacity: 0.2,
		color: 'rgb(211,211,211)',
		type: 'mesh3d',
		x: a,
		y: b,
		z: c,
		hoverinfo: 'skip',
	}

	//////////////////////
	// EDIT 2: obtain the second half of the sphere by duplicating 
	// the upper semisphere ("..." operator before "dataitem") and 
	// changing its "z" attribute into negative vales:

	var data = [
		dataitem,
		{...dataitem, z: c.map(v => -v)}
	];
	
	return data;
}

// compute polarization ellipse points 
// https://en.wikipedia.org/w/index.php?title=Ellipse&oldid=456212176#Ellipses_in_computer_graphics 
/*
* This functions returns an array containing 36 points to draw an
* ellipse.
*
* @param x {double} X coordinate
* @param y {double} Y coordinate
* @param a {double} Semimajor axis
* @param b {double} Semiminor axis
* @param angle {double} Angle of the ellipse
*/
function calculateEllipse(x, y, a, b, angle, steps) 
{
  if (steps == null)
    steps = 36;
  //var points = [];
  var points = {
	x: [],
	y: [],
	//name: [],
	type: 'scatter',
	legendgroup: 'group1',
  };
  
  // Angle is given by Degree Value
  var beta    = angle; //* (Math.PI / 180); //(Math.PI/180) converts Degree Value into Radians
  var sinbeta = Math.sin(beta);
  var cosbeta = Math.cos(beta);

  for (var i = 0; i < 360; i += 360 / steps) 
  {
    var alpha = i * (Math.PI / 180) ;
    var sinalpha = Math.sin(alpha);
    var cosalpha = Math.cos(alpha);

    var X = x + (a * cosalpha * cosbeta - b * sinalpha * sinbeta);
    var Y = y + (a * cosalpha * sinbeta + b * sinalpha * cosbeta);

	points.x.push(X);
	points.y.push(Y);
   }
  return points;
}


function plotResults(resultArray,isEdit,idx){
		// plot results				
			let S0 = resultArray[0];
			let output = resultArray;
			// calculate ellipse
			var maxSquare = {
			x: [S0, -S0, -S0,  S0,  S0],
			y: [S0,  S0, -S0, -S0,  S0],
			type: 'scatter'
			}
			let pointsPlus = calculateEllipse(0, 0, output[26], output[27], output[24], 50);
			pointsPlus.name = 'id ' + idx;

			// calculate sphere point
			let trace1 = {
				x: [output[4]], y: [output[5]], z: [output[6]],
				name: 'id ' + idx,
				mode: 'markers',
				marker: {
					size: 6,
					line: {
					color: 'rgba(217, 217, 217, 0.14)',
					width: 0.5},
					opacity: 0.8},
				type: 'scatter3d',
				legendgroup: 'group1',
			};
	if (isEdit){
		// https://stackoverflow.com/questions/60678586/update-x-and-y-values-of-a-trace-using-plotly-update
		// https://plotly.com/javascript/plotlyjs-function-reference/
		Plotly.deleteTraces(ellipsePlot, idx-1);
	    Plotly.deleteTraces(spherePlot, idx+1);
		Plotly.addTraces(ellipsePlot, pointsPlus, idx-1);		
		Plotly.addTraces(spherePlot, trace1, idx+1);
	} else {
		Plotly.addTraces(ellipsePlot, pointsPlus);		
		Plotly.addTraces(spherePlot, trace1);
	}	
}
</script>


<!---------------------------- WORK WITH DYNAMIC TABLES AND FORMS: basic commands (necessary for page initialization) ---------------------------->
<!-- ROW TEMPLATES -->
<script>
function addEmptyRow() {
	idx = document.getElementById("customStokesTable").rows.length;
	// предусматриваем поле для ввода следующего оптического сигнала во всех таблицах сразу
	addCustomStokesDataRow(idx);
	addNormalizedStokesDataRow(idx); 
	addIntensityDataRow(idx);
	addCoherencyDataRow(idx);
	
	// делаем поля для ввода следующего сигнала редактируемыми
	editCustomStokesRow(null,idx);
	editNormalizedStokesRow(null,idx);
	editIntensityRow(null,idx);
	editCoherencyRow(null,idx);	
}

function addCustomStokesDataRow(idx) {
  var table = document.getElementById("customStokesTable");
  var row   = table.insertRow(idx);
  var cell1 = row.insertCell(0);
  var cell2 = row.insertCell(1);
  var cell3 = row.insertCell(2);
  var cell4 = row.insertCell(3);
  var cell5 = row.insertCell(4);
  var cell6 = row.insertCell(5);
  var cell7 = row.insertCell(6);
  //cell1.innerHTML = idx;
  cell2.innerHTML = "CELL";
  cell3.innerHTML = "CELL";
}

function addNormalizedStokesDataRow(idx) {
  var table = document.getElementById("normalizedStokesTable");
  var row   = table.insertRow(idx);
  var cell1 = row.insertCell(0);
  var cell2 = row.insertCell(1);
  var cell3 = row.insertCell(2);
  var cell4 = row.insertCell(3);
  var cell5 = row.insertCell(4);
  var cell6 = row.insertCell(5);
  var cell7 = row.insertCell(6);
  var cell8 = row.insertCell(7);
  //cell1.innerHTML = idx;
  cell2.innerHTML = "CELL";
  cell3.innerHTML = "CELL";
}

function addIntensityDataRow(idx) {
  var table = document.getElementById("intensityTable");
  var row   = table.insertRow(idx);
  var cell1 = row.insertCell(0);
  var cell2 = row.insertCell(1);
  var cell3 = row.insertCell(2);
  var cell4 = row.insertCell(3);
  var cell5 = row.insertCell(4);
  var cell6 = row.insertCell(5);
  var cell7 = row.insertCell(6);
  var cell8 = row.insertCell(7);
  //cell1.innerHTML = idx;
  cell2.innerHTML = "CELL";
  cell3.innerHTML = "CELL";
}

function addCoherencyDataRow(idx) {
  var table = document.getElementById("coherencyTable");
  var row   = table.insertRow(idx);
  var cell1 = row.insertCell(0);
  var cell2 = row.insertCell(1);
  var cell3 = row.insertCell(2);
  var cell4 = row.insertCell(3);
  //cell1.innerHTML = idx;
  cell2.innerHTML = "CELL";
  cell3.innerHTML = "CELL";
}

function addResultsDataRow(idx) {
  var table       = document.getElementById("resultsTable");
  var row         = table.insertRow(idx);
  var cell1       = row.insertCell(0);
  var cell2       = row.insertCell(1);
  var cell3       = row.insertCell(2);
  var cell4       = row.insertCell(3);
  var cell5       = row.insertCell(4);
  var cell6       = row.insertCell(5);
  var cell7       = row.insertCell(6);
  var cell8       = row.insertCell(7);
  //cell1.innerHTML = idx;
  cell2.innerHTML = "CELL";
  cell3.innerHTML = "CELL";
  cell4.innerHTML = "CELL";
  cell5.innerHTML = "CELL";
  cell6.innerHTML = "CELL";
  cell7.innerHTML = "CELL";
  cell8.innerHTML = "-";
}

// edit (event OR call) on the Custom Stokes Table
function editCustomStokesRow(event, idx) {
  let table = document.getElementById("customStokesTable");	
  if (event==null){
	  let row   = table.rows[idx];
	  row.cells[1].innerHTML = "<input type='number' name='S0' id='S0input' min='0' max='1' step='0.01' size='4' value='" + 1 +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[2].innerHTML = "<input type='number' name='S1' id='S1input' min='-1' max='1' step='0.01' size='4' value='0.1' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[3].innerHTML = "<input type='number' name='S2' id='S2input' min='-1' max='1' step='0.01' size='4' value='0.2' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[4].innerHTML = "<input type='number' name='S3' id='S3input' min='-1' max='1' step='0.01' size='4' value='-0.3' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[5].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddCustomStokes(event)' id='inputCustomStokes'>Submit</button>";
  } else {
	  event.preventDefault();
	  
	  // блокируем стандартный input
	  disableInputRow();
	  
	  // вырубаем все кнопки editRowBtn
	  let editBtns = document.getElementsByName("editRowBtn");
	  editBtns.forEach((editBtn) => {
		editBtn.disabled = true;
	  });
	  
	  let removeBtns = document.getElementsByName("removeRowBtn");
		removeBtns.forEach((removeBtn) => {
			removeBtn.disabled = true;
		});
	  
	  // создаем input на месте редактируемой строки
      let idx = event.target.closest('tr').rowIndex;
	  let row = table.rows[idx];
	  row.cells[1].innerHTML = "<input type='number' name='S0' id='S0edit' min='0' max='1' step='0.01' size='4' value='" + Number(row.cells[1].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[2].innerHTML = "<input type='number' name='S1' id='S1edit' min='-1' max='1' step='0.01' size='4' value='" + Number(row.cells[2].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[3].innerHTML = "<input type='number' name='S2' id='S2edit' min='-1' max='1' step='0.01' size='4' value='" + Number(row.cells[3].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[4].innerHTML = "<input type='number' name='S3' id='S3edit' min='-1' max='1' step='0.01' size='4' value='" + Number(row.cells[4].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[5].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddCustomStokes(event)'>Apply</button>";
  }
 
  // var rowCount = table.rows.length; var row = table.insertRow(rowCount);
}

// edit (event OR call) on the Normalized Stokes Table
function editNormalizedStokesRow(event, idx) {
  let table = document.getElementById("normalizedStokesTable");		
	if (event==null){
		let row   = table.rows[idx];
		row.cells[1].innerHTML = "<input type='number' name='power' id='powerInput' min='0' max='99' step='0.01' size='3' value='1' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<input type='number' name='dop' id='dopInput' min='0' max='1' step='0.01' size='3' value='1' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[3].innerHTML = "<input type='number' name='s1' id='s1Input' min='-1' max='1' step='0.01' size='3' value='0' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[4].innerHTML = "<input type='number' name='s2' id='s2Input' min='-1' max='1' step='0.01' size='3' value='0' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[5].innerHTML = "<input type='number' name='s3' id='s3Input' min='-1' max='1' step='0.01' size='3' value='0' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[6].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddNormalizedStokes(event)' id='inputNormalizedStokes'>Submit</button>";
	} else {
		event.preventDefault();
		
		// блокируем стандартный input
	    disableInputRow();
		
		// вырубаем все кнопки editRowBtn
		let editBtns = document.getElementsByName("editRowBtn");
		editBtns.forEach((editBtn) => {
			editBtn.disabled = true;
		});
		
		let removeBtns = document.getElementsByName("removeRowBtn");
		removeBtns.forEach((removeBtn) => {
			removeBtn.disabled = true;
		});
		
		let idx = event.target.closest('tr').rowIndex;
	    let row = table.rows[idx];
	    row.cells[1].innerHTML = "<input type='number' name='power' id='powerEdit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[1].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	    row.cells[2].innerHTML = "<input type='number' name='dop' id='dopEdit' min='0' max='1' step='0.01' size='3' value='" + Number(row.cells[2].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	    row.cells[3].innerHTML = "<input type='number' name='s1' id='s1Edit' min='-1' max='1' step='0.01' size='3' value='" + Number(row.cells[3].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	    row.cells[4].innerHTML = "<input type='number' name='s2' id='s2Edit' min='-1' max='1' step='0.01' size='3' value='" + Number(row.cells[4].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[5].innerHTML = "<input type='number' name='s3' id='s3Edit' min='-1' max='1' step='0.01' size='3' value='" + Number(row.cells[5].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	    row.cells[6].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddNormalizedStokes(event)'>Apply</button>";
	}
}

// edit (event OR call) on the Intensity Table
function editIntensityRow(event, idx) {
  let table = document.getElementById("intensityTable");		
	if (event==null){
		let row   = table.rows[idx];
		//row.style = "display:none"; // TEMPORARY, WHILE INTENSITY INPUT DOES NOT WORK
		/*
		row.cells[1].innerHTML = "<input type='number' name='iH' id='IHinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<input type='number' name='iV' id='IVinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[3].innerHTML = "<input type='number' name='iD' id='IDinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[4].innerHTML = "<input type='number' name='iA' id='IAinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[5].innerHTML = "<input type='number' name='iR' id='IRinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[6].innerHTML = "<input type='number' name='iL' id='ILinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[7].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddIntensity(event)' id='inputIntensity' style='display:none;'>Submit</button> <button class='btn btn-primary btn-sm' disabled='true'>Submit</button>";
		*/
	} else {
		event.preventDefault();
		
		// блокируем стандартный input
	    disableInputRow();
		
		// вырубаем все кнопки editRowBtn
		let editBtns = document.getElementsByName("editRowBtn");
		editBtns.forEach((editBtn) => {
			editBtn.disabled = true;
		});
		
		let removeBtns = document.getElementsByName("removeRowBtn");
		removeBtns.forEach((removeBtn) => {
			removeBtn.disabled = true;
		});
		
		let idx = event.target.closest('tr').rowIndex;
	    let row = table.rows[idx];
		//row.style = "display:none";
		/*
		row.cells[1].innerHTML = "<input type='number' name='iH' id='IHedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[1].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<input type='number' name='iV' id='IVedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[2].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[3].innerHTML = "<input type='number' name='iD' id='IDedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[3].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[4].innerHTML = "<input type='number' name='iA' id='IAedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[4].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[5].innerHTML = "<input type='number' name='iR' id='IRedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[5].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[6].innerHTML = "<input type='number' name='iL' id='ILedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[6].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[7].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddIntensity(event)'>Apply</button> <br /> <button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>";
		*/
	}
}

// edit (event OR call) on the Coherency Table
function editCoherencyRow(event, idx) {
  let table = document.getElementById("coherencyTable");		
	if (event==null){	
		let row   = table.rows[idx];
		row.cells[1].innerHTML = ""; //"[ <input type='number' name='J11' id='J11input' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>, <input type='number' name='J12real' id='J12realinput' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> + j <input type='number' name='J12imag' id='J12imaginput' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> <br /> [ <input type='number' name='J21real' id='J21realinput' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> - j <input type='number' name='J21imag' id='J21imaginput' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>, <input type='number' name='J22' id='J22input' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddCoherency(event)' id='inputCoherency' style='display:none;'>Submit</button> <button class='btn btn-primary btn-sm disabled' disabled='true'>Submit</button>";
		document.getElementById('inputCoherency').disabled = true;
	} else {
		event.preventDefault();
		
		// блокируем стандартный input
	    disableInputRow();
		
		// вырубаем все кнопки editRowBtn
		let editBtns = document.getElementsByName("editRowBtn");
		editBtns.forEach((editBtn) => {
			editBtn.disabled = true;
		});
		
		let removeBtns = document.getElementsByName("removeRowBtn");
		removeBtns.forEach((removeBtn) => {
			removeBtn.disabled = true;
		});
		
		
		let idx = event.target.closest('tr').rowIndex;
	    let row = table.rows[idx];
		
		// TODO: makeConstant функцию изобразить аккуратно через div с требуемыми id
		row.cells[1].innerHTML = ""; //"[ <input type='number' name='J11' id='J11edit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>, <input type='number' name='J12real' id='J12realedit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> + j <input type='number' name='J12imag' id='J12imagedit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> <br /> [ <input type='number' name='J21real' id='J21realedit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> - j <input type='number' name='J21imag' id='J12imagedit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>, <input type='number' name='J22' id='J22edit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddCoherency(event)'>Apply</button>";
	}
}
</script>


<!---------------------------- INTERACTION WITH WASM LIBRARY: to migrate into separate js file ---------------------------->
<!-- INTERACTION CORE: functions for passing <double> arrays into WASM -->
<script>
	// JavaScript Array to Emscripten Heap
	function _arrayToHeap(typedArray) {
		  var numBytes = typedArray.length * typedArray.BYTES_PER_ELEMENT;
		  var ptr = Module._malloc(numBytes);
		  var heapBytes = new Uint8Array(Module.HEAPU8.buffer, ptr, numBytes);
		  heapBytes.set(new Uint8Array(typedArray.buffer));
		  return heapBytes;
	}
	// Emscripten Heap to JavaScript Array
	function _heapToArray(heapBytes, array) {
		  return new Float64Array(
			heapBytes.buffer,
			heapBytes.byteOffset,
			heapBytes.length / array.BYTES_PER_ELEMENT);
	}
	// Free Heap
	function _freeArray(heapBytes) {
		  Module._free(heapBytes.byteOffset);
	}
</script>

<!-- INTERACTION INTERFACE pt1: functions that accept input data, call WASM and read output -->
<script>	
		function processCustomStokes(S0,S1,S2,S3,idx,isEdit) {
			// create Stokes vector (array)
			let Sarr     = [];
			Sarr.push(S0);
			Sarr.push(S1);
			Sarr.push(S2);
			Sarr.push(S3);
			//console.log(Sarr);
			
			// prepare to pass data into WASM
			var S64      = new Float64Array(Sarr); // [1x4] 
			
			// pass input data to heap
			var Sbytes   = _arrayToHeap(S64);
					
			// allocate memory for the output data
			var OUT      = new Float64Array(29);    // [1x29]
			 
			// pass output data memory to heap
			var OUTbytes = new _arrayToHeap(OUT);
			
			// run WASM functions operating on heap arrays
			//Module._wasm_StokesVector_initialize(); 
			if (isEdit) {
				Module._wasm_editCustomStokes(Sbytes.byteOffset,idx);
			} else {
				Module._wasm_addCustomStokes(Sbytes.byteOffset);
			}
			Module._wasm_getAll(idx,OUTbytes.byteOffset); 
            //Module._wasm_StokesVector_terminate();
			
			// grab output data from heap
			OUT          = _heapToArray(OUTbytes, OUT);
			let out      = Array.from(OUT);
			
            // free memory in heap
            _freeArray(OUTbytes);
			_freeArray(Sbytes);
			
			return out;	
		}
		
function processNormalizedStokes(pwr,dop,s1,s2,s3,idx,isEdit){
	// create Stokes vector (array)
	let input = [];
	input.push(pwr);
	input.push(dop);
	input.push(s1);
	input.push(s2);
	input.push(s3);
	
	// prepare to pass data into WASM
	let inputArr   = new Float64Array(input); // [1x5] 
			
	// pass input data to heap
	let inputBytes = _arrayToHeap(inputArr);
					
	// allocate memory for the output data
	let OUT      = new Float64Array(29);    // [1x29]
			 
	// pass output data memory to heap
	let OUTbytes = new _arrayToHeap(OUT);
	
	// run WASM functions operating on heap arrays
	if (isEdit) {
		Module._wasm_editNormalizedStokes(inputBytes.byteOffset,idx);
	} else {
		Module._wasm_addNormalizedStokes(inputBytes.byteOffset);
	}
	Module._wasm_getAll(idx,OUTbytes.byteOffset); 
	
	// grab output data from heap
	OUT          = _heapToArray(OUTbytes, OUT);
	let out      = Array.from(OUT);
			
	// free memory in heap
	_freeArray(OUTbytes);
	_freeArray(inputBytes);
			
	return out;	
}

function processIntensity(IH,IV,ID,IA,IR,IL,idx,isEdit){
	// create Stokes vector (array)
	let input = [];
	input.push(IH);
	input.push(IV);
	input.push(ID);
	input.push(IA);
	input.push(IR);
	input.push(IL);
	
	// prepare to pass data into WASM
	let inputArr   = new Float64Array(input); // [1x5] 
			
	// pass input data to heap
	let inputBytes = _arrayToHeap(inputArr);
					
	// allocate memory for the output data
	let OUT      = new Float64Array(29);    // [1x29]
			 
	// pass output data memory to heap
	let OUTbytes = new _arrayToHeap(OUT);
	
	// run WASM functions operating on heap arrays
	if (isEdit) {
		Module._wasm_editIntensitySet(inputBytes.byteOffset,idx);
	} else {
		Module._wasm_addIntensitySet(inputBytes.byteOffset);
	}
	Module._wasm_getAll(idx,OUTbytes.byteOffset); 
	
	// grab output data from heap
	OUT          = _heapToArray(OUTbytes, OUT);
	let out      = Array.from(OUT);
			
	// free memory in heap
	_freeArray(OUTbytes);
	_freeArray(inputBytes);
			
	return out;	
}
</script>


<!-- INTERACTION INTERFACE pt2: click (submit) event handlers -->
<script>
// ArbitraryStokes form: process click of the "submit" button
function onAddCustomStokes(event){
	//StokesForm.submit();
	//StokesForm.reportValidity();
	event.preventDefault();
	let idx = event.target.closest('tr').rowIndex;
	let isEdit = idx!=document.getElementById("customStokesTable").rows.length-1;
	
	// read form values
	let S0     = StokesForm.S0.value;
	let S1     = StokesForm.S1.value;
	let S2     = StokesForm.S2.value;
	let S3     = StokesForm.S3.value;
	
	// check validity of the input data
	if (S0 < 0) {
		alert("Power defined by S_0 parameter can not be negative!");
		return;
	}
	
	if (S1*S1 + S2*S2 + S3*S3 > S0*S0) {
		alert("Incorrect data provided: the inequality relating square values of Stokes vector elements is not satisfied!");
		return;
	}
	
	// WASM-расчеты
	let output = processCustomStokes(S0,S1,S2,S3,idx,isEdit); // если строка не последняя, то отрабатываем сценарий "редактирование строки"
	
	// round array values to match the specific pattern (amount of digits)
	// https://stackoverflow.com/questions/9671203/how-to-round-all-the-values-in-an-array-to-2-decimal-points
	// https://stackoverflow.com/questions/661562/how-to-format-a-float-in-javascript#comment51761240_661579
	let out2round = output.map(function(each_element){
		return Number( Math.round(each_element*1000)/1000 );
		//return Number( each_element.toFixed(2) );
	});
			
	// output results into table
	if (!isEdit)
		addResultsDataRow(idx);
	
	populateResultsRow(idx,out2round);	
	populateCustomStokesDataRow(idx,out2round);
	populateNormalizedStokesDataRow(idx,out2round);
	populateIntensityDataRow(idx,out2round);
	populateCoherencyDataRow(idx,out2round);
	
	if (idx==document.getElementById("customStokesTable").rows.length-1) {	
		addEmptyRow();	
		updateRowIds(document.getElementById("resultsTable"));
		updateRowIds(document.getElementById("customStokesTable"));
		updateRowIds(document.getElementById("normalizedStokesTable"));
		updateRowIds(document.getElementById("intensityTable"));
		updateRowIds(document.getElementById("coherencyTable"));
	} else {
		enableInputRow();
	}
	
	// plot results
	plotResults(output,isEdit,idx);
	
	// врубаем все кнопки editRowBtn
	let editBtns = document.getElementsByName("editRowBtn");
	editBtns.forEach((editBtn) => {
		editBtn.disabled = false;
	});
	
	let removeBtns = document.getElementsByName("removeRowBtn");
	removeBtns.forEach((removeBtn) => {
		removeBtn.disabled = false;
	});
}

// NormalizedStokes form: process click of the "submit" button
function onAddNormalizedStokes(event){
	event.preventDefault();
	let idx     = event.target.closest('tr').rowIndex;
	let isEdit  = idx!=document.getElementById("normalizedStokesTable").rows.length-1;
	
	// read form values
	let pwr     = StokesNormalizedForm.power.value;
	let dop     = StokesNormalizedForm.dop.value;
	let s1      = StokesNormalizedForm.s1.value;
	let s2      = StokesNormalizedForm.s2.value;
	let s3      = StokesNormalizedForm.s3.value;
	
	// WASM-расчеты
	let output = processNormalizedStokes(pwr,dop,s1,s2,s3,idx,isEdit);	
	let out2round = output.map(function(each_element){
		return Number( Math.round(each_element*1000)/1000 );
	});
	
	// output results into table
	if (!isEdit)
		addResultsDataRow(idx);
	
	populateResultsRow(idx,out2round);
	populateCustomStokesDataRow(idx,out2round);
	populateNormalizedStokesDataRow(idx,out2round);
	populateIntensityDataRow(idx,out2round);
	populateCoherencyDataRow(idx,out2round);
	
	if (idx==document.getElementById("normalizedStokesTable").rows.length-1) {	
		addEmptyRow();	
		updateRowIds(document.getElementById("resultsTable"));
		updateRowIds(document.getElementById("customStokesTable"));
		updateRowIds(document.getElementById("normalizedStokesTable"));
		updateRowIds(document.getElementById("intensityTable"));
		updateRowIds(document.getElementById("coherencyTable"));
	} else {
		enableInputRow();
	}
	
	// plot results
	plotResults(output,isEdit,idx);
	
	// врубаем все кнопки editRowBtn
	let editBtns = document.getElementsByName("editRowBtn");
	editBtns.forEach((editBtn) => {
		editBtn.disabled = false;
	});
	
	let removeBtns = document.getElementsByName("removeRowBtn");
	removeBtns.forEach((removeBtn) => {
		removeBtn.disabled = false;
	});
}

// Intensity form: process click of the "submit" button
function onAddIntensity(event){
	event.preventDefault();
	let idx     = event.target.closest('tr').rowIndex;
	let isEdit  = idx!=document.getElementById("intensityTable").rows.length-1;
	
	// read form values
	let IH     = IntensityForm.iH.value;
	let IV     = IntensityForm.iV.value;
	let ID     = IntensityForm.iD.value;
	let IA     = IntensityForm.iA.value;
	let IR     = IntensityForm.iR.value;
	let IL     = IntensityForm.iL.value;
	
	// WASM-расчеты
	let output = processIntensity(IH,IV,ID,IA,IR,IL,idx,isEdit);	
	let out2round = output.map(function(each_element){
		return Number( Math.round(each_element*1000)/1000 );
	});
	
	// output results into table
	if (!isEdit)
		addResultsDataRow(idx);
	
	populateResultsRow(idx,out2round);
	populateCustomStokesDataRow(idx,out2round);
	populateNormalizedStokesDataRow(idx,out2round);
	populateIntensityDataRow(idx,out2round);
	populateCoherencyDataRow(idx,out2round);
	
	if (idx==document.getElementById("intensityTable").rows.length-1) {	
		addEmptyRow();	
		updateRowIds(document.getElementById("resultsTable"));
		updateRowIds(document.getElementById("customStokesTable"));
		updateRowIds(document.getElementById("normalizedStokesTable"));
		updateRowIds(document.getElementById("intensityTable"));
		updateRowIds(document.getElementById("coherencyTable"));
	} else {
		enableInputRow();
	}
	
	// plot results
	plotResults(output,isEdit,idx);
	
	// врубаем все кнопки editRowBtn
	let editBtns = document.getElementsByName("editRowBtn");
	editBtns.forEach((editBtn) => {
		editBtn.disabled = false;
	});
	
	let removeBtns = document.getElementsByName("removeRowBtn");
	removeBtns.forEach((removeBtn) => {
		removeBtn.disabled = false;
	});
}
</script>


<!-- INTERACTION INTERFACE pt3: populate table rows with results -->
<script>
function populateResultsRow(idx,resultArray) {
  let table = document.getElementById("resultsTable");
  //let idx   = event.target.closest('tr').rowIndex;
  let row   = table.rows[idx];
  row.cells[1].innerHTML = resultArray[21]; // DOP
  row.cells[2].innerHTML = resultArray[22]; // DOLP
  row.cells[3].innerHTML = resultArray[23]; // DOCP
  row.cells[4].innerHTML = resultArray[24]; // Azimuth psi
  row.cells[5].innerHTML = resultArray[25]; // Ellipticity chi
  row.cells[6].innerHTML = resultArray[28]; // Phase difference delta
  if (resultArray[1]*resultArray[1]+resultArray[2]*resultArray[2]+resultArray[3]*resultArray[3]>0){
	row.cells[7].innerHTML = determineHelicity(resultArray[3]);
  }else{
	row.cells[7].innerHTML = "Unpolarized";
  }
}


function populateCustomStokesDataRow(idx,resultArray) {
  let table = document.getElementById("customStokesTable");
  //let idx   = event.target.closest('tr').rowIndex;
  let row   = table.rows[idx];
  row.cells[1].innerHTML = resultArray[0]; // S0
  row.cells[2].innerHTML = resultArray[1]; // S1
  row.cells[3].innerHTML = resultArray[2]; // S2
  row.cells[4].innerHTML = resultArray[3]; // S3
  row.cells[5].innerHTML = "<button class='btn btn-info btn-sm' onclick='editCustomStokesRow(event)' name='editRowBtn'>Edit</button>";
  row.cells[6].innerHTML = "<button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>";	
}

function populateNormalizedStokesDataRow(idx,resultArray){
  let table = document.getElementById("normalizedStokesTable");
  //let idx   = event.target.closest('tr').rowIndex;
  let row   = table.rows[idx];
  row.cells[1].innerHTML = resultArray[0];  // S0 = Power
  row.cells[2].innerHTML = resultArray[21]; // DOP
  row.cells[3].innerHTML = resultArray[4];  // s1
  row.cells[4].innerHTML = resultArray[5];  // s2
  row.cells[5].innerHTML = resultArray[6];  // s3
  row.cells[6].innerHTML = "<button class='btn btn-info btn-sm' onclick='editNormalizedStokesRow(event)' name='editRowBtn'>Edit</button>";
  row.cells[7].innerHTML = "<button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>";
}

function populateIntensityDataRow(idx,resultArray){
  let table = document.getElementById("intensityTable");
  //let idx   = event.target.closest('tr').rowIndex;
  let row   = table.rows[idx];
  row.style = ""; // TEMPORARY, WHILE INTENSITY INPUT DOES NOT WORK
  row.cells[1].innerHTML = resultArray[7];  // IH
  row.cells[2].innerHTML = resultArray[8];  // IV
  row.cells[3].innerHTML = resultArray[9];  // ID
  row.cells[4].innerHTML = resultArray[10]; // IA
  row.cells[5].innerHTML = resultArray[11]; // IR
  row.cells[6].innerHTML = resultArray[12]; // IL
  row.cells[7].innerHTML = "<!-- <button class='btn btn-info btn-sm' onclick='editIntensityRow(event)' name='editRowBtn'>Edit</button> <br /> --> <button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>";	
}

function populateCoherencyDataRow(idx,resultArray){ // need div with ids !!!!!! or?...
  let table = document.getElementById("coherencyTable");
  //let idx   = event.target.closest('tr').rowIndex;
  let row   = table.rows[idx];
  row.cells[1].innerHTML = '[ ' + resultArray[13] + ' ' + resultArray[14] + '+j' + resultArray[15] + ' ] <br /> [ ' + resultArray[16] + '+j' + resultArray[17] + ' ' + resultArray[18] + ' ]'; // будет комбо из 6ти полей!! и они будут на div ! !!!!!!!!!!!!!!!
  row.cells[2].innerHTML = "<span class='d-inline-block' tabindex='0' data-toggle='tooltip' title='Disabled tooltip'> <button class='btn btn-info btn-sm' onclick='editCoherencyRow(event)' disabled='true'>Edit</button> </span>";
  row.cells[3].innerHTML = "<button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>"; 	
}
</script>


<!-- INTERACTION FRONTEND: work with tables and forms, which is not required for page initialization, but required for WASM app to work -->
<script>
// input disabler
function disableInputRow() {
	document.getElementById('inputCustomStokes').disabled = true;
	document.getElementById('S0input').disabled = true;
	document.getElementById('S0input').name = "S0disabled";
	document.getElementById('S1input').disabled = true;
	document.getElementById('S1input').name = "S1disabled";
	document.getElementById('S2input').disabled = true;
    document.getElementById('S2input').name = "S2disabled";
	document.getElementById('S3input').disabled = true;
	document.getElementById('S3input').name = "S3disabled";
	  
	document.getElementById('inputNormalizedStokes').disabled = true;
	document.getElementById('powerInput').disabled = true;
	document.getElementById('powerInput').name = "powerDisabled";
	document.getElementById('dopInput').disabled = true;
	document.getElementById('dopInput').name = "dopDisabled";
	document.getElementById('s1Input').disabled = true;
	document.getElementById('s1Input').name = "s1Disabled";
	document.getElementById('s2Input').disabled = true;
	document.getElementById('s2Input').name = "s2Disabled";
	document.getElementById('s3Input').disabled = true;
	document.getElementById('s3Input').name = "s3Disabled";
	  
	/*
	document.getElementById('inputIntensity').disabled = true;
	document.getElementById('IHinput').disabled = true;
	document.getElementById('IHinput').name = "iHdisabled";
	document.getElementById('IVinput').disabled = true;
	document.getElementById('IVinput').name = "iVdisabled";
	document.getElementById('IDinput').disabled = true;
	document.getElementById('IDinput').name = "iDdisabled";
	document.getElementById('IAinput').disabled = true;
	document.getElementById('IAinput').name = "iAdisabled";
	document.getElementById('ILinput').disabled = true;
	document.getElementById('ILinput').name = "iLdisabled";  
	document.getElementById('IRinput').disabled = true;
	document.getElementById('IRinput').name = "iRdisabled";  
	*/
	
	/*
	document.getElementById('inputCoherency').disabled = true;
	document.getElementById('J11input').disabled = true;
	document.getElementById('J11input').name = "J11disabled";
	document.getElementById('J12realinput').disabled = true;
	document.getElementById('J12realinput').name = "J12realdisabled";
	document.getElementById('J12imaginput').disabled = true;
	document.getElementById('J12imaginput').name = "J12imagdisabled";
	document.getElementById('J21realinput').disabled = true;
	document.getElementById('J21realinput').name = "J21realdisabled";
	document.getElementById('J21imaginput').disabled = true;
	document.getElementById('J21imaginput').name = "J21imagdisabled";  
	document.getElementById('J22input').disabled = true;
	document.getElementById('J22input').name = "J22disabled";
    */  	
}

// input enabler
function enableInputRow() {
	document.getElementById('inputCustomStokes').disabled = false;
	document.getElementById('S0input').disabled = false;
	document.getElementById('S0input').name = "S0";
	document.getElementById('S1input').disabled = false;
	document.getElementById('S1input').name = "S1";
	document.getElementById('S2input').disabled = false;
	document.getElementById('S2input').name = "S2";
	document.getElementById('S3input').disabled = false;
	document.getElementById('S3input').name = "S3";
	
	document.getElementById('inputNormalizedStokes').disabled = false;
	document.getElementById('powerInput').disabled = false;
	document.getElementById('powerInput').name = "power";
	document.getElementById('dopInput').disabled = false;
	document.getElementById('dopInput').name = "dop";
	document.getElementById('s1Input').disabled = false;
	document.getElementById('s1Input').name = "s1";
	document.getElementById('s2Input').disabled = false;
	document.getElementById('s2Input').name = "s2";
	document.getElementById('s3Input').disabled = false;
	document.getElementById('s3Input').name = "s3";
	
	/*
	document.getElementById('inputIntensity').disabled = false;
	document.getElementById('IHinput').disabled = false;
	document.getElementById('IHinput').name = "iH";
	document.getElementById('IVinput').disabled = false;
	document.getElementById('IVinput').name = "iV";
	document.getElementById('IDinput').disabled = false;
	document.getElementById('IDinput').name = "iD";
	document.getElementById('IAinput').disabled = false;
	document.getElementById('IAinput').name = "iA";
	document.getElementById('ILinput').disabled = false;
	document.getElementById('ILinput').name = "iL";  
	document.getElementById('IRinput').disabled = false;
	document.getElementById('IRinput').name = "iR";  
	*/
	/*
	document.getElementById('inputIntensity').disabled = true;
	document.getElementById('IHinput').disabled = true;
	document.getElementById('IHinput').name = "iH";
	document.getElementById('IVinput').disabled = true;
	document.getElementById('IVinput').name = "iV";
	document.getElementById('IDinput').disabled = true;
	document.getElementById('IDinput').name = "iD";
	document.getElementById('IAinput').disabled = true;
	document.getElementById('IAinput').name = "iA";
	document.getElementById('ILinput').disabled = true;
	document.getElementById('ILinput').name = "iL";  
	document.getElementById('IRinput').disabled = true;
	document.getElementById('IRinput').name = "iR";  
	*/
	
	/*
	document.getElementById('inputCoherency').disabled = false;
	document.getElementById('J11input').disabled = false;
	document.getElementById('J11input').name = "J11";
	document.getElementById('J12realinput').disabled = false;
	document.getElementById('J12realinput').name = "J12real";
	document.getElementById('J12imaginput').disabled = false;
	document.getElementById('J12imaginput').name = "J12imag";
	document.getElementById('J21realinput').disabled = false;
	document.getElementById('J21realinput').name = "J21real";
	document.getElementById('J21imaginput').disabled = false;
	document.getElementById('J21imaginput').name = "J21imag";  
	document.getElementById('J22input').disabled = false;
	document.getElementById('J22input').name = "J22";  
	*/
}




// edit (event OR call) on the Custom Stokes Table
function editCustomStokesRow(event, idx) {
  let table = document.getElementById("customStokesTable");	
  if (event==null){
	  let row   = table.rows[idx];
	  row.cells[1].innerHTML = "<input type='number' name='S0' id='S0input' min='0' max='1' step='0.01' size='4' value='" + 1 +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[2].innerHTML = "<input type='number' name='S1' id='S1input' min='-1' max='1' step='0.01' size='4' value='0.1' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[3].innerHTML = "<input type='number' name='S2' id='S2input' min='-1' max='1' step='0.01' size='4' value='0.2' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[4].innerHTML = "<input type='number' name='S3' id='S3input' min='-1' max='1' step='0.01' size='4' value='-0.3' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[5].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddCustomStokes(event)' id='inputCustomStokes'>Submit</button>";
  } else {
	  event.preventDefault();
	  
	  // блокируем стандартный input
	  disableInputRow();
	  
	  // вырубаем все кнопки editRowBtn
	  let editBtns = document.getElementsByName("editRowBtn");
	  editBtns.forEach((editBtn) => {
		editBtn.disabled = true;
	  });
	  
	  let removeBtns = document.getElementsByName("removeRowBtn");
		removeBtns.forEach((removeBtn) => {
			removeBtn.disabled = true;
		});
	  // <!-- можно сделать еще отдельную галочку типа show on plot -- чисто в таблице с результатами !!!!!!!!!!!!!!!!!!!!!!! -->
	  
	  // создаем input на месте редактируемой строки
      let idx = event.target.closest('tr').rowIndex;
	  let row = table.rows[idx];
	  row.cells[1].innerHTML = "<input type='number' name='S0' id='S0edit' min='0' max='1' step='0.01' size='4' value='" + Number(row.cells[1].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[2].innerHTML = "<input type='number' name='S1' id='S1edit' min='-1' max='1' step='0.01' size='4' value='" + Number(row.cells[2].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[3].innerHTML = "<input type='number' name='S2' id='S2edit' min='-1' max='1' step='0.01' size='4' value='" + Number(row.cells[3].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[4].innerHTML = "<input type='number' name='S3' id='S3edit' min='-1' max='1' step='0.01' size='4' value='" + Number(row.cells[4].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	  row.cells[5].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddCustomStokes(event)'>Apply</button>";
  }
 
  // var rowCount = table.rows.length; var row = table.insertRow(rowCount);
}

// edit (event OR call) on the Normalized Stokes Table
function editNormalizedStokesRow(event, idx) {
  let table = document.getElementById("normalizedStokesTable");		
	if (event==null){
		let row   = table.rows[idx];
		row.cells[1].innerHTML = "<input type='number' name='power' id='powerInput' min='0' max='99' step='0.01' size='3' value='1' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<input type='number' name='dop' id='dopInput' min='0' max='1' step='0.01' size='3' value='1' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[3].innerHTML = "<input type='number' name='s1' id='s1Input' min='-1' max='1' step='0.01' size='3' value='0' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[4].innerHTML = "<input type='number' name='s2' id='s2Input' min='-1' max='1' step='0.01' size='3' value='0' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[5].innerHTML = "<input type='number' name='s3' id='s3Input' min='-1' max='1' step='0.01' size='3' value='0' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[6].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddNormalizedStokes(event)' id='inputNormalizedStokes'>Submit</button>";
	} else {
		event.preventDefault();
		
		// блокируем стандартный input
	    disableInputRow();
		
		// вырубаем все кнопки editRowBtn
		let editBtns = document.getElementsByName("editRowBtn");
		editBtns.forEach((editBtn) => {
			editBtn.disabled = true;
		});
		
		let removeBtns = document.getElementsByName("removeRowBtn");
		removeBtns.forEach((removeBtn) => {
			removeBtn.disabled = true;
		});
		
		let idx = event.target.closest('tr').rowIndex;
	    let row = table.rows[idx];
	    row.cells[1].innerHTML = "<input type='number' name='power' id='powerEdit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[1].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	    row.cells[2].innerHTML = "<input type='number' name='dop' id='dopEdit' min='0' max='1' step='0.01' size='3' value='" + Number(row.cells[2].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	    row.cells[3].innerHTML = "<input type='number' name='s1' id='s1Edit' min='-1' max='1' step='0.01' size='3' value='" + Number(row.cells[3].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	    row.cells[4].innerHTML = "<input type='number' name='s2' id='s2Edit' min='-1' max='1' step='0.01' size='3' value='" + Number(row.cells[4].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[5].innerHTML = "<input type='number' name='s3' id='s3Edit' min='-1' max='1' step='0.01' size='3' value='" + Number(row.cells[5].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
	    row.cells[6].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddNormalizedStokes(event)'>Apply</button>";
	}
}

// edit (event OR call) on the Intensity Table
function editIntensityRow(event, idx) {
  let table = document.getElementById("intensityTable");		
	if (event==null){
		let row   = table.rows[idx];
		// tooltip
		//row.dataset.tooltip = "Input for these parameters is not implemented yet.";
		row.style = "display:none"; // TEMPORARY, WHILE INTENSITY INPUT DOES NOT WORK
		/*
		row.cells[1].innerHTML = "<input type='number' name='iH' id='IHinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<input type='number' name='iV' id='IVinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[3].innerHTML = "<input type='number' name='iD' id='IDinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[4].innerHTML = "<input type='number' name='iA' id='IAinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[5].innerHTML = "<input type='number' name='iR' id='IRinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[6].innerHTML = "<input type='number' name='iL' id='ILinput' min='0' max='99' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[7].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddIntensity(event)' id='inputIntensity' style='display:none;'>Submit</button> <button class='btn btn-primary btn-sm disabled'>Submit</button>";
		document.getElementById('IHinput').disabled = true;
		document.getElementById('IVinput').disabled = true;
		document.getElementById('IDinput').disabled = true;
		document.getElementById('IAinput').disabled = true;
		document.getElementById('IRinput').disabled = true;
		document.getElementById('ILinput').disabled = true;
		*/
	} else {
		event.preventDefault();
		
		// блокируем стандартный input
	    disableInputRow();
		
		// вырубаем все кнопки editRowBtn
		let editBtns = document.getElementsByName("editRowBtn");
		editBtns.forEach((editBtn) => {
			editBtn.disabled = true;
		});
		
		let removeBtns = document.getElementsByName("removeRowBtn");
		removeBtns.forEach((removeBtn) => {
			removeBtn.disabled = true;
		});
		
		let idx = event.target.closest('tr').rowIndex;
	    let row = table.rows[idx];
		row.cells[1].innerHTML = "<input type='number' name='iH' id='IHedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[1].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<input type='number' name='iV' id='IVedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[2].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[3].innerHTML = "<input type='number' name='iD' id='IDedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[3].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[4].innerHTML = "<input type='number' name='iA' id='IAedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[4].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[5].innerHTML = "<input type='number' name='iR' id='IRedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[5].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[6].innerHTML = "<input type='number' name='iL' id='ILedit' min='0' max='99' step='0.01' size='3' value='" + Number(row.cells[6].innerHTML) +"' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[7].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddIntensity(event)'>Apply</button> <br /> <button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>";
	}
}

// edit (event OR call) on the Coherency Table
function editCoherencyRow(event, idx) {
  let table = document.getElementById("coherencyTable");		
	if (event==null){	
		let row   = table.rows[idx];
		row.cells[1].innerHTML = ""; //"[ <input type='number' name='J11' id='J11input' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>, <input type='number' name='J12real' id='J12realinput' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> + j <input type='number' name='J12imag' id='J12imaginput' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> <br /> [ <input type='number' name='J21real' id='J21realinput' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> - j <input type='number' name='J21imag' id='J21imaginput' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>, <input type='number' name='J22' id='J22input' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddCoherency(event)' id='inputCoherency'>Submit</button>";
		document.getElementById('inputCoherency').disabled = true;
	} else {
		event.preventDefault();
		
		// блокируем стандартный input
	    disableInputRow();
		
		// вырубаем все кнопки editRowBtn
		let editBtns = document.getElementsByName("editRowBtn");
		editBtns.forEach((editBtn) => {
			editBtn.disabled = true;
		});
		
		let removeBtns = document.getElementsByName("removeRowBtn");
		removeBtns.forEach((removeBtn) => {
			removeBtn.disabled = true;
		});
		
		
		let idx = event.target.closest('tr').rowIndex;
	    let row = table.rows[idx];
		
		// не реализована подгрузка текущих value -- надо будет makeConstant функцию модифицировать аккуратно через div с требуемыми id !!!!!
		row.cells[1].innerHTML = "[ <input type='number' name='J11' id='J11edit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>, <input type='number' name='J12real' id='J12realedit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> + j <input type='number' name='J12imag' id='J12imagedit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> <br /> [ <input type='number' name='J21real' id='J21realedit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'> - j <input type='number' name='J21imag' id='J12imagedit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>, <input type='number' name='J22' id='J22edit' step='0.01' size='3' value='' required pattern='\S(.*\S)?' title='This field is required'>";
		row.cells[2].innerHTML = "<button class='btn btn-primary btn-sm' onclick='onAddCoherency(event)'>Apply</button>";
	}
}


// функции, превращающие строки в не-редактируемые
function makeCustomStokesRowConstant(idx) {
  let x = document.getElementById("customStokesTable").rows[idx];
  x.cells[1].innerHTML = StokesForm.S0.value;
  x.cells[2].innerHTML = StokesForm.S1.value;
  x.cells[3].innerHTML = StokesForm.S2.value;
  x.cells[4].innerHTML = StokesForm.S3.value;
  x.cells[5].innerHTML = "<button class='btn btn-info btn-sm' onclick='editCustomStokesRow(event)' name='editRowBtn'>Edit</button>";
  x.cells[6].innerHTML = "<button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>";
}		  

function makeNormalizedStokesRowConstant(idx) {
  let x = document.getElementById("normalizedStokesTable").rows[idx];
  x.cells[1].innerHTML = StokesNormalizedForm.power.value;
  x.cells[2].innerHTML = StokesNormalizedForm.dop.value;
  x.cells[3].innerHTML = StokesNormalizedForm.s1.value;
  x.cells[4].innerHTML = StokesNormalizedForm.s2.value;
  x.cells[5].innerHTML = StokesNormalizedForm.s3.value;
  x.cells[6].innerHTML = "<button class='btn btn-info btn-sm' onclick='editNormalizedStokesRow(event)' name='editRowBtn'>Edit</button>";
  x.cells[7].innerHTML = "<button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>";
}

function makeIntensityRowConstant(idx) {
  let x = document.getElementById("intensityTable").rows[idx];
  x.cells[1].innerHTML = IntensityForm.iH.value;
  x.cells[2].innerHTML = IntensityForm.iV.value;
  x.cells[3].innerHTML = IntensityForm.iD.value;
  x.cells[4].innerHTML = IntensityForm.iA.value;
  x.cells[5].innerHTML = IntensityForm.iL.value;
  x.cells[6].innerHTML = IntensityForm.iR.value;
  x.cells[7].innerHTML = "<button class='btn btn-info btn-sm' onclick='editIntensityRow(event)' name='editRowBtn'>Edit</button> <br /> <button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>";
}

function makeCoherencyRowConstant(idx) {
  let x = document.getElementById("coherencyTable").rows[idx];
  x.cells[1].innerHTML = CoherencyForm.J11.value; // будет комбо из 6ти полей!! и они будут на div ! !!!!!!!!!!!!!!!
  x.cells[2].innerHTML = "<button class='btn btn-info btn-sm' onclick='editCoherencyRow(event)' disabled='true'>Edit</button>";
  x.cells[3].innerHTML = "<button class='btn btn-danger btn-sm' onclick='deleteRowAction(event)' name='removeRowBtn'>Remove</button>"; 
}
</script>










<!---------------------- EXECUTE ON PAGE LOAD ---------------------->
<script>
	// make the first tab active
	document.getElementById("mainTab").click();
	
	// create input row
	addEmptyRow();
	
	// add empty row to imitate unavailable intensity submission form
	// addIntensityDataRow(document.getElementById("intensityTable").rows.length);
	
	// clear placeholder row #1 which is required for the page to initialize correctly
	deletePlaceholderRows();
	
	// associate HTML forms with javascript code
	const StokesForm           = document.forms["StokesCustom"];
	const StokesNormalizedForm = document.forms["StokesNormalizedForm"]; 
	const IntensityForm        = document.forms["IntensityForm"]; 
	const CoherencyForm        = document.forms["CoherencyForm"]; 	

	// override actions on form submit
	StokesForm.onsubmit = function(e) {
			e.preventDefault();
		}
	StokesNormalizedForm.onsubmit = function(e) {
			e.preventDefault();
		}
	IntensityForm.onsubmit = function(e) {
			e.preventDefault();
		}
	CoherencyForm.onsubmit = function(e) {
			e.preventDefault();
		}	
		
	// initialize plot divs
	const ellipsePlot = document.getElementById('ellipsePlot');
	const spherePlot = document.getElementById('spherePlot');
	
	// defer plotly and Stokes initialization: https://github.com/plotly/dash/issues/1041
	var interval = setInterval(function(){
	   if(document.readyState === 'complete') {
		   initializeScriptsOnPage();
	   }
	}, 0);

	var initializeScriptsOnPage = function() {
		initializeEllipsePlot();
		initializeSpherePlot();
		let table1 = document.getElementById('customStokesTable');
		let table3 = document.getElementById('intensityTable');
		let table5 = document.getElementById('resultsTable');
		//table1.setAttribute("class", "table table-bordered table-sm table-striped table-hover");
		//table3.setAttribute("class", "table table-bordered table-sm table-striped table-hover"); // does not help much
		//table5.setAttribute("class", "table table-bordered table-sm table-striped table-hover");
		clearInterval(interval);
	}	
</script>