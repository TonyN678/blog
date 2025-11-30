+++
date = '2025-11-30T14:07:02+11:00'
draft = false
title = 'Volt/Amp Meter with INA219, Nano and LCD display'
+++

<h1>Building a DIY Power Meter & Logger with Arduino Nano and INA219</h1>

<p> In this project, I’ll walk you through how I built a simple but accurate power meter and data logger using three main components: an Arduino Nano, the INA219 current/voltage monitoring IC, and a 16×2 LCD display. The goal was to create a compact device that can measure voltage, current, and power in real time—and also log the data for later analysis. </p>

![Overall](/img4.png)


<h2> Explanation of why this circuit is <u>WRONG</u> for only measuring the BATTERY, not shunt voltage through load!!!</h2>
<ul> 
<li> The SDA and SDL pins of the INA219 and the LCD Display must be connect to the A4 and A5 pins of the Nano respectively ( A4 and A5 are in charge of I2C communication, on Arduino Uno board it has its own SDA and SDL pins).</li>
<li> The LCD Display is powered by 5V from 5V pin of Nano.</li>
<li> LOAD's VCC is connected to V- pin of INA219 while its GND is connected to GND. (load is literally anything that consume power ie. resistor convert electricity to heat, IC got powered by power, ....</li>
<li> Battery pack is ~3V, this is measured in the LCD to be around 3.2V across the load; The load is there for current to flow through so INA219 can measure shunt voltage; BATT+ --> INA219's V+, BATT- --> GND.</li>
</ul>   
 
![Schematic](/schematic.png)

<h2> TRUE schematic for measuring shunt voltage across a LOAD </h2>

![Schematic1](/img1.png)

![Real-Circuit](/img3.jpg)

<p> LOAD's GND --> BATT's GND, LOAD's VCC --> V- of INA219 !!! </p>
<p> The shunt voltage (voltage drop) across LED is around 1.0V with 92mA current </p)

<h2> Understanding the INA219 Power Monitor </h2>

<p>The INA219 is the key component that makes this power meter project both accurate and easy to build. It’s a precision high-side current and voltage sensor developed by Texas Instruments, and it allows the Arduino to measure:

<ul>
<li>Bus Voltage (the voltage of the system you’re powering)</li> 

<li>Shunt Voltage (the small <u> <b>voltage drop</b></u> across a shunt resistor)</li>

<li>Current (calculated using the shunt voltage)</li>

<li>Power (voltage × current) </li>
</ul>

What makes the INA219 especially useful for hobby projects is that it handles all the difficult analog measurements internally and provides clean, digital readings over I²C (basically mentioning SDL and SDA pins on the module apart from VCC & GND), so you don’t need ADC calibration or amplification stages. </p> 

<h2>Why You Need a Load in a Power Measurement Circuit</h2>

  <p>
    When using the INA219 (or any current-sensing device), you must have a 
    <strong>load</strong> connected in the circuit. The load is the component, module, or device 
    that actually consumes electrical power, such as a motor, LED strip, battery charger, or any 
    electronic device under test.
  </p>

  <h3>1. Current Only Flows When There Is a Load</h3>
  <p>
    The INA219 measures current by detecting the small voltage drop across the shunt resistor.
    But current only flows if there is something drawing power.  
    Without a load, there is no current through the shunt resistor — so the INA219 will read:
    <em>0 mA</em> and <em>0 mW</em>.
  </p>

  <h3>2. Power = Voltage × Current</h3>
  <p>
    Electrical power can only exist when both voltage and current are present.  
    Even if the power supply is connected, without a load:
  </p>
  <ul>
    <li>Voltage is present</li>
    <li>But current is zero</li>
  </ul>
  <p><strong>This means the power is also zero.</strong></p>

  <h3>3. The INA219 Needs a Voltage Drop to Measure Current</h3>
  <p>
    The INA219 works by measuring the tiny voltage drop across its built-in shunt resistor.  
    If no load is connected, no current flows, and the shunt voltage is:
    <strong>0 volts</strong>.  
    With nothing to measure, the sensor cannot calculate current or power.
  </p>

  <h3>4. The Load Determines Real-World Behaviour</h3>
  <p>
    Different loads draw different amounts of current — and this is what makes the power meter
    useful. A fan, LED strip, or DC motor will each produce unique readings. Measuring a real 
    load lets you:
  </p>
  <ul>
    <li>Profile power consumption</li>
    <li>Test efficiency</li>
    <li>Monitor battery discharge</li>
    <li>Compare different devices</li>
  </ul>

  <h3>5. Without a Load, You're Only Measuring the Power Supply Itself</h3>
  <p>
    If there’s no load, the INA219 ends up monitoring nothing but the open-circuit voltage of 
    your power supply — which isn’t useful for a power meter or logger.
  </p>

  <h3>Summary</h3>
  <p>
    You need a load because it creates current flow.  
    That current flow creates a voltage drop across the shunt resistor.  
    The INA219 uses that voltage drop to calculate current and power.
  </p>

<h2> Reference </h2>
<ul>
<li> https://www.adafruit.com/product/904 </li>
<li> https://learn.adafruit.com/pro-trinket-power-meter/usage </li> 
<li> https://lastminuteengineers.com/i2c-lcd-arduino-tutorial/ </li>
<li> https://www.instructables.com/Make-Your-Own-Power-MeterLogger/ </li>
</ul>
