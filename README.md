# ecmo-calculations
A set of web-based apps for Extracorporeal membrane oxygenation (ECMO) calculations

## Flow/pressure drop across ECMO cannulas
Blood flow throw an ECMO circuit is typically limited by the cannula. Flow and pressure is determined by the [Hagen-Pousille equation](https://en.wikipedia.org/wiki/Hagen%E2%80%93Poiseuille_equation), whereby flow/pressure drop is proportional to fourth power of inner canulla diameter and inversely proportional to cannula length:

```math
\Delta P = \frac{8 \mu \cdot L \cdot Q}{\pi \cdot {R}^{4}}
```

While Hagen-Pousille provides a good _first approximation_ of flow, there are several limitations:
* In laminar flow modes (low Reynolds number), the **pressure flow relationship should be linear**: P ∝ Q for rigid, smooth tubes
* As flow increases, turbulence develops (high Reynolds numbers), the **pressure flow relationship becomes non-linear**: P ∝ $Q^{2} $
* Blood is a **non-Newtonian fluid** - viscosity changes with shear
* Cannula geometry is complex - mutiple stages with multiple holes complicates flow dynamics
* At very high flow rates, additional phenomena like compressibility and cavitation may contribute

Thus instead of using Hagen-Pousille, it is more accurate to ***empirically measure the performance of each cannula***. Each manufacturer provides the measured flow/pressure curves for each cannula. As a clinician it is necessary to consult the different cannula pressure/flow curves to determine the expected pressure drop and flow acheivable with a given ECMO configuration. Unfortunately this requires finding & consulting many different PDFs to find the necessary information, especially if comparing multiple possible cannula configurations.

I made a simple web app that combines all the data together, making it easier to compare different cannula and determine the expected flows/pressure drops.

![](https://github.com/nickmmark/ecmo-calculations/blob/main/ECMO_cannula_calculator_v1_demo.gif)


#### Process
I used the published flow/pressure curves for each cannula, abstracting the data using [WebPlotDigitizer 4.0](https://apps.automeris.io/wpd4/) and plotted a polynomial curve to fit the datapoints. Theoretically, a polynomial can capture both linear and higher-order terms, accommodating the transition from laminar to turbulent flow as well as other complicating factors. Empircally, the polynomial curves that I generated fit with $R^{2}$ > 0.99 in all cases.

![Data abstraction process](https://github.com/nickmmark/ecmo-calculations/blob/main/ECMO_cannula_flow.png)


#### Catheter data sources
* Medtronic --> [Crescent Dual Lumen Jugular Catheter](https://europe.medtronic.com/xd-en/healthcare-professionals/products/cardiovascular/extracorporeal-life-support/crescent-jugular-dual-lumen-catheter.html)
* Spectrum Medical --> [Quantum Dual Lumen Jugular Catheter](https://www.spectrummedical.com/en-us/quantum-perfusion-technologies/quantum-sterile-technologies-us/cannulas-us/dual-lumen-rv-to-pa-cannula)
* Getinge --> [Avalon Elite Bi-Caval Dual Lumen Catheter](https://www.getinge.com/int/products/avalon-elite-catheter/?tab=2)

#### Things to do
[] add more cannula!
[] schematics of the cannula showing details such as stages, number of inlets, and cross section
[] add more organization of cannula types (IJ dual lumen, IJ single lumen, femoral single lumen, etc)


## Recircualtion
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


## License/disclaimer
This code is provided "as is", without warranty of any kind. Don't be stupid. Double check any calculations. Consult a medical professional if necessary. See the attached license for more information about the MIT License.

## References
