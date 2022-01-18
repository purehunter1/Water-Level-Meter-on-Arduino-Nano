# Water-Level-Meter-on-Arduino-Nano

This project shows how to build up a water level meter with a self-made capacitive sensor based on an Arduino Nano with only very few additional components.
It can be configured with an optional temperature compensation realized with only 3 more cheap components.

Measurement values are averaged over n cycles and normalized to 255 to fit in one Byte for easy serial communication.
The program can be modified to get the output as an analog value instead or additional to the serial data output. This opens the possibility to use an analog meter to display the water level.
 
Measuring principle is a capacitive voltage divider. Two Caps in series are being charged by the "drive pin" of the Arduino. Partial voltage along the sensor is measured then at the "measuring pin". Very simple.
With Cref=200pf + Csens=200pf in series, the capacity to be charged is 100pF. With an output impedance of a digital pin of abou 100R, it takes less than 50ns.
The ADC sampling takes enough time (microseconds) to have stable conditions. It shows very constant results with a Nano.

The Reference Cap is between DrivePin and MeasPin. Use class 1 caps, e.g. C0G/NP0 dielectric (stability, aging characteristics)
Choose a Reference Cap in the Range of (min Sensor Cap. @ empty tank)*2 . This results in ADC readings:   750 > (AdcEmpty value) > 650 (with internal analog reference). (e.g. 100-200pF as a start value)
The Capacitive Sensor is between MeasPin (the core wire) and GND (the tube). Wires from the Arduino to sensor should be as short as any possible (few cm, 30cm worked fine for me).
 
The sensor is e.g. a 30-70cm INOX tube with 8-10mm inner diam. with an insulated centered drilled wire loop.
or better a INOX tube with 6mm inner diameter and a INOX (welding) wire with 1.6mm diameter well isolated in a shrinking tube and additional sealant at the ends. Or a rigid electrical wire, also sealed.
The wire shall be centered well in the tube, e.g. with 2 or 4 blobs of sealant in the middle of the wire, hardened before final assembly/insertion.
Side effect: water drops inside the sensor, especially at oscillating water levels may lead to slightly too high readings, decreasing after some time if the level remain constant again.
The thinner the tube is, the higher this side effect is.

Theory: The sensor capacity would change by factor 80 ideally when fully immersed if there was no measurable isolation thickness inside the sensor. In reality we can assume an isolation
on the inner wire of about 0.2mm for a shrinking tube. This 0.2mm are lets say 10% of the distance between the inner wire and the 6mm INOX tube. Its permittivity is low and will not change.
In consequence, only the capacity of the remaining 90% of the distance raises by factor 80 when we exchange the air by water, but in sum the capacity will change only by about factor 9.
This is not too bad, as we have a non-linear voltage divider: Umeas=(Udrive/(UCref+UCsens))*UCsens. UCsens should not approach zero or Uref here.
Empty Tank: UmeasMax=1/(UCref+UCsensMax)*UCsensMax = 1/(0.5+1)*1 = 66% of Udrive (Cref = 2*CsensMin)
Full Tank: UmeasMin=1/(UCref+UCsensMin)*UCsensMin = 1/(0.5+0.11)*0.11 = 18% of Udrive (Cref = 2*CsensMin)

As measured values are in a "narrow" Range (e.g. 18%...66% or 184...682 of 1023 ADC digits, see above), sent values can be stretched to (nearly) full range of 1 Byte (0...255) to keep higher resolution.

Above values are always in this range, as most parameters are "relative" to each other. Except e.g. the sensor cable.

To get a usable correlation of ADC output values with the immersion depth it is recommended to use a "polynomial regression" calculation
The calculation of the formula can be done e.g. with Open Office (+ CorelPolyGUI Addon) or Excel (Chart, Add Trendline, Polynomial) . -> X-values are the Sensor ADC output values, Y-values represent the immersion depth, volume, or else.
A polynom of 3rd order is suitable for most applications with sufficient accuracy, even for complex water container shapes.
Example: float levelNormal = 265.455 - 4.31*level + 0.024*pow(float(level),2) - 0.0000445*pow(float(level),3) ;  // polynomial regression
To calculate the polynom, it is required to measure about 10 values with your final HW setup in advance which gives you 10 value pairs, e.g. xn=sensor value; yn=water volume

If the calculation with the gained formula is being done on the host computer, the "Sensor Arduino" with this program can be housed sealed and inacessible and all
"calibrations" can be done on the host computer.

We can use the external reference voltage pin AREF of hte Arduino to further stretch the analog readings for a better resolution: with an external 10k resistor from AREF to +5V. AREF has about 32k to GND.
It lowers the internal 5V ADC reference to about 3.75V and causes the previously discussed maximum voltage UmeasMax to increase from 66% to >90% (this is the target).
If the ext reference is wanted, this has to be done before the calculation of the correlation of the ADC values with the needed output like immersion depth, volume, ...
To make use of this, find out, which resistor fits for your setup, try with 10k as initial value to get ADC readings of 900 to 950 with a dry sensor. This resistor is between +5V VCC and the AREF pin.
Additionally uncomment the line "analogReference(EXTERNAL);"

Instead of (only) extending the spread of the analog readings, we can also make use of the external reference to compensate thermal effects on the permittivity of water.
This can be helpful, if your setup is not located under constant temperatures like outdoor, or if you deal with "cold" or "hot" water.
The temperature influence on the permittivity of water is quite high. If you want to measure a water level in an outdoor tank, the temperature span can easily be
somewhere between 0°C and 50°C. Relative to 25°C the "error" without temperature compensation can be around +-11% in this temperature range.
If this is too much deviation, we just need an additional 47k NTC (EPCOS K164 characteristic (B25/100=4450 K)) and a 4k7 resistor for a few cents.
The NTC and the additional resistor are in series and located between the AREF pin to GND. The resistor from AREF to 5V has to be 6k8 in this case to maintain a maximum ADC reading of about 950 (<1023) with a "dry" sensor.
With this simple circuit the reference voltage changes by ca. +-10% @ 25°C +-25K (0°C to 50°C). This should compensate the change in the permittivity of the water sufficiently between 0°C and 100°C.
The NTC can be soldered on the Arduino directly, or it can be located at the bottom of the water tank if you expect "rapid" ambient temperature changes, or
if you need to measure the water level while filling the tank with "hot" or "cold" water.
Measurements in icy water (below 0°C) or in boiling water is "impossible".

Temperature influence on permittivity of water: 
0°C:87.81, 10°C:83.99, 20°C:80.27, 30°C:76.67 40°C:73.22 50°C:69.90 60°C:66.73 70°C:63.70 80°C:60.81 90°C:58.05 100°C:55.41 (IAPS, U.Grigull, München 1983)
In the range of "outdoor" conditions it changes with ~ -0.45%/K (~ -11.3%/25K referenced to 25°C in the range of 0°C to 50°C)

As we measure only relative values, all derived directly from the VCC, the stability of the reference voltage is no big issue.
If supplied with 12V, the Arduino has sufficient stability on 5V VCC.
As the circuit allows to have the shielding sensor tube connected to GND we also have no big problems with EMI, e.g. from cell phones or other sources.
The concept of the program targets low energy consumption. You can further decrease it by depopulating components of the Arduino. First of all, the Power-LED.
But you can also depopulate the CH340 (Nano Clone) once the program is uploaded and you are sure everything works fine.
Modify the measuring interval according to your needs, the maximum interval is about 8s, limited by the Watchdog capabilities.
This is the "update frequency" if you use the analog output. The analog signal is always present.

This is only a suggestion, please verify everything by yourself, I take no responsibility for any problems arising from it. I did not test all conditions.
