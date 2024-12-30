# ecmo-calculations
Extracorporeal membrane oxygenation (ECMO) calculations

## Calculating pressure drop across ECMO cannulas
Blood flow throw an ECMO circuit is typically limited by the cannula. Flow and pressure is determined by the Hagen-Pousille equation, whereby flow/pressure drop is proportional to fourth power of inner canulla diameter and inversely proportional to cannula length:

```math
\delta P = \frac{8 \nu \cdot L \cdot Q}{\pi {R}^{4}}
```

While Hagen-Pousille provides a good first approximation of flow, there are several important limitations:
* Blood is a non-Newtonian fluid - viscosity changes with shear
* Cannula geometry is complex - mutiple stages with multiple holes complicates flow

Thus it is more accurate to empirically measure the performance of each cannula. Each manufacturer provides the measured flow/pressure curves for each cannula. As a clinician it is necessary to consult the different cannula pressure/flow curves to determine the expected pressure drop and flow acheivable with a given ECMO configuration. Unfortunately this requires consulting many different PDFs to find the necessary information.

This is a simple web app that combines all the data together, making it easier to compare cannula and determine expected flows/pressure drops.

## Recircualtion


## Membrane lung efficiency


## License/disclaimer
This code is provided "as is", without warranty of any kind. Don't be stupid. Double check any calculations. Consult a medical professional if necessary. See the attached license for more information about the MIT License.

## References
