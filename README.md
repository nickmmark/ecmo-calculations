# ecmo-calculations
A set of web-based apps for Extracorporeal membrane oxygenation (ECMO) calculations

## Flow/pressure drop across ECMO cannulas
Blood flow throw an ECMO circuit is typically limited by the cannula. Flow and pressure is determined by the [Hagen-Pousille equation](https://en.wikipedia.org/wiki/Hagen%E2%80%93Poiseuille_equation), whereby flow/pressure drop is proportional to fourth power of inner canulla diameter and inversely proportional to cannula length:

```math
\Delta P = \frac{8 \mu \cdot L \cdot Q}{\pi \cdot {R}^{4}}
```

While **Hagen-Pousille** provides a good _first approximation_ of flow, there are several limitations:
* In _laminar_ flow modes (low Reynolds number), the **pressure flow relationship should be linear**: P ∝ Q for rigid, smooth tubes
* As flow increases _turbulence_ develops (high Reynolds numbers), the **pressure flow relationship becomes non-linear**: P ∝ $Q^{2} $
* **Blood is a [non-Newtonian fluid](https://en.wikipedia.org/wiki/Non-Newtonian_fluid#:~:text=In%20physics%20and%20chemistry%2C%20a,change%20when%20subjected%20to%20force.)** - viscosity changes with shear; specifically blood exhibits **shear thinning** whereby the apparent viscosity decreases with shear.
* **Cannula geometry is complex** - mutiple stages with multiple holes complicates flow dynamics
* At very high flow rates, additional phenomena like compressibility and cavitation may contribute

Thus instead of using Hagen-Pousille, it is more accurate to ***empirically measure the performance of each cannula***. Each manufacturer provides the measured flow/pressure nomogram curves for each cannula. As a clinician it is necessary to consult the different cannula pressure/flow curves to determine the expected pressure drop and flow acheivable with a given ECMO configuration. Unfortunately this requires finding & consulting many different PDFs to find the necessary information, especially if comparing multiple possible cannula configurations.

I made a simple web app that combines all the data together, making it easier to compare different cannula and determine the expected flows/pressure drops:

![](https://github.com/nickmmark/ecmo-calculations/blob/main/ECMO_cannula_calculator_v1_demo.gif)

The "maximum pressure drop" across a venous cannula in ECMO refers to the greatest difference in pressure between the blood entering the cannula from the patient's vein and the blood exiting the cannula into the ECMO circuit. Typically a value of 100 or 150 mmHg is considered the maximum pressure drop. The app allows the user to specify the maximum permissible pressure drop and it both calculates it for each cannula and displays it visually on the graph.


#### Data Abstraction Process
I used the published flow/pressure curves for each cannula, abstracting the data using [WebPlotDigitizer 4.0](https://apps.automeris.io/wpd4/) and plotted a polynomial curve to fit the datapoints. Theoretically, a polynomial can capture both linear and higher-order terms, accommodating the transition from laminar to turbulent flow as well as other complicating factors. Empircally, the polynomial curves that I generated fit with $R^{2}$ > 0.99 in all cases.

![Data abstraction process](https://github.com/nickmmark/ecmo-calculations/blob/main/ECMO_cannula_flow.png)

All the flow curves were obtained from the manufacturer. See below for references.


#### Guide for adding more cannula to the app
1. Abstract the curves using [WebPlotDigitizer](https://apps.automeris.io/wpd4/); align the axes & add at least 5 points for each line.
2. Paste the data into excel & create a `second order polynomial` trendline. Set the intercept = 0.0. Ensure the $R^{2}$ > 0.99 for quality. Copy the polynomial. For example, here is data for the ProTek Duo cannula:

| Cannula | Polynomial | R2 |
|---|---|---|
| ProTek Duo 29Fr inflow | y = 14.909x2 + 1.7309x | 0.9997 |
| ProTek Duo 29Fr drainage | y = -2.4682x2 - 2.1438x | 0.9983 |
| ProTek Duo 31Fr inflow | y = 6.9807x2 + 1.9931x | 0.9999 |
| ProTek Duo 31Fr drainage | y = -1.8139x2 - 4.5159x | 0.9997 |  

3. Add a new group of checkboxes for the new cannula. e.g.
```html
  <div class="group protek-group">
      <label><input type="checkbox" id="ProTek"><b> ProTek Duo Cannula</b></label>
      <label><input type="checkbox" id="ProTek29Fr"> 29 Fr <span class="flow-result" id="flow-ProTek29Fr"></span></label>
      <label><input type="checkbox" id="ProTek31Fr"> 31 Fr <span class="flow-result" id="flow-ProTek31Fr"></span></label>
  </div>
```
4. Update the equations and flow coefficients. e.g.
```javascript
  // equations
    "ProTek29FrDrain": (x) => -2.4682 * x ** 2 - 2.1438 * x;
    "ProTek31FrDrain": (x) => -1.8139 * x ** 2 - 4.5159 * x;
  // coefficients
    "ProTek29Fr": { a: 14.909, b: 1.7309 },
     "ProTek31Fr": { a: 6.9807, b: 1.9931 }
```
5. Update the dataset generation logic. e.g.
```javascript
const label = key.includes('Quantum') ? `Quantum ${type} Cannula ${size}` :
              key.includes('Avalon') ? `Avalon ${type} Cannula ${size}` :
              key.includes('ProTek') ? `ProTek Duo ${type} Cannula ${size}` : // NEW cannula added HERE
              `Crescent ${type} Cannula ${size}`;
```
6. Add logic to the checkbox event handler, so the cannula appear/disappear when boxes are checked. (Note that for single lumen cannula there would only be one cannula per checkbox event). e.g.
``` javascript
case "ProTek29Fr":
    chart.data.datasets.find(dataset => dataset.label === "ProTek Duo Inflow Cannula 29Fr").hidden = !checkbox.checked;
    chart.data.datasets.find(dataset => dataset.label === "ProTek Duo Drainage Cannula 29Fr").hidden = !checkbox.checked;
    break;
case "ProTek31Fr":
    chart.data.datasets.find(dataset => dataset.label === "ProTek Duo Inflow Cannula 31Fr").hidden = !checkbox.checked;
    chart.data.datasets.find(dataset => dataset.label === "ProTek Duo Drainage Cannula 31Fr").hidden = !checkbox.checked;
    break;
```
7. Run the updated code in your web-browser

#### Versions
* 1.0 (barely) managed to graph the curves
* 2.0 improved graphing, added groups of cannula
* 3.0 improved graphs with max flow calculated for a given maximum pressure drop
* 4.0 added additional cannula, improved UX, added hyperlinks to the manufacturer nomogram for transparency


#### Things to do
[x] add a "maximum predicted flow" for each cannula given the maximum pressure drop

[] add more cannula! (this probably means having a seperate database that the visualization app can access... ugh complex)

[] add schematics for each cannula showing details such as stages, number of inlets, and cross section (a.k.a. pretty drawings!)

[] add more organization of cannula types (IJ dual lumen, IJ single lumen, femoral single lumen, etc)

[] allow the user to specify combinations of different cannula (e.g. VAV, VAPa, etc); this will require adding flow together cannula in parallel. Some interesting math and physics to think about. Kirchoff's laws?



## Re-circualtion Calculator
**Recirculation** is a common issue with venovenous ECMO that occurs when oxygenated blood is withdrawn by the drainage cannula, instead of entering the patient's systemic circulation.

Recirculation (%) is defined as:
```math
Recirculation = \frac{{S}_{pre}O2 - {S}_{v}O2}{{S}_{post}O2 - {S}_{v}O2}
```
where:
* Recirculation - the % of blood that is recirculating
* SpreO2 - saturation of blood entering the oxygenator
* SpreO2 - saturation of blood leaving the oxygenator
* SvO2 - saturation of blood in the pulmonary artery

#### Interpretation
Recirculation >20% is generally considered clincally significant.
Recirculation can be caused by several factors:
* **Cannula migration** (e.g. obstruction of flow into the drainage cannula leading to more from infusion cannula)
* **Excessive flow** (e.g. drainage cannula is pulling in blood from infusion cannula)
* **Abnormal pressure differentials** (e.g. elevated intra-cardiac, intrathoracic, or intra-abdominal pressures)
Additionally, the absolute saturations can convey additional information; A SpreO2 <75% suggests that significant recirculation is unlikely.

#### Implementation
A simple web app for calculating re-circulation (%). It also provides some messages to help the user interpret/troubleshoot the results.
![](https://github.com/nickmmark/ecmo-calculations/blob/main/ECMO_Recirculation_demo1.gif)


## Membrane lung efficiency
**Membrane Lung Efficiency** is a quantitative measure of how well the ECMO membrane lung transfers oxygen (and carbon dioxide) between blood and gas. Changes in membrane lung efficiency can indicate potential issues such as thrombosis or circuit malfunction.
```math
VO2ML = BF \cdot ({C}_{post}O2 - {C}_{post}O2) \cdot 10
```
where:
* VO2ML - the oxygen transfer for the membrane lung (mL/min)
* BF - blood flow through the membrane lung (L/min)
* CpreO2 - oxygen content in blood entering the membrane lung (mL/dL)
* CpostO2 - oxygen content in blood leaving the membrane lung (mL/dL)

We can substitute in formula for oxygen content for both CpreO2 and CpostO2:
CaO2 = (Hb x 1.34 ml O2/g Hb x SaO2) + (PaO2 x 0.003 ml O2/mmHg/dL)

where:
* Hb - hemoglobin concentration (g/dL)
* SxO2 - the oxygen saturation (%) of the blood
* PxO2 - partial pressure of oxygen in the blood

## DO2 and VO2

## Lung Mechanics:Driving Pressure, Mechanical Power
![](https://github.com/nickmmark/pulmonary-calculations/blob/main/mechanical_power_demo2.gif)

[see my other git for more](https://github.com/nickmmark/pulmonary-calculations)

## ECMO prognostic scores
Several scores exist to (try to) predict prognosis in patients undergoing ECMO or to identify good candidates. These systems include
* RESP (Respiratory ECMO Survival Prediction) Score --> [see my other git for more](https://github.com/nickmmark/pulmonary-calculations)
* SAVE (Survival after Veno-Arterial ECMO) Score
* PRESET (PREdiction of Survival on ECMO Therapy) Score
* Murray Score
* Oxygenation Index
* & more
  

## License/disclaimer
This code is provided "as is", without warranty of any kind. Don't be stupid. Double check any calculations. Consult a medical professional if necessary. See the attached license for more information about the MIT License.

## References
#### Catheter data sources
* Medtronic --> [Crescent Dual Lumen Jugular Catheter](https://europe.medtronic.com/xd-en/healthcare-professionals/products/cardiovascular/extracorporeal-life-support/crescent-jugular-dual-lumen-catheter.html)
* Spectrum Medical --> [Quantum Dual Lumen Jugular Catheter](https://www.spectrummedical.com/en-us/quantum-perfusion-technologies/quantum-sterile-technologies-us/cannulas-us/dual-lumen-rv-to-pa-cannula)
* Getinge --> [Avalon Elite Bi-Caval Dual Lumen Catheter](https://www.getinge.com/int/products/avalon-elite-catheter/?tab=2)
* LivaNova --> [ProTek Duo Dual Lumen Catheter](https://www.livanova.com/advanced-circulatory-support/en-us/protekduo-kit)

#### Re-circulation
Abrams, D. Brodie, D. Identification and management of recirculation in venovenous ECMO. _ELSO Guidelines_ 2015
Xie A, Yan TD, Forrest P. **[Recirculation in venovenous extracorporeal membrane oxygenation.](10.1016/j.jcrc.2016.05.027)** _J Crit Care._ 2016
