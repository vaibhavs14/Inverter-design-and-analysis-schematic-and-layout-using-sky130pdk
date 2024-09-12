# Inverter design and analysis using sky130pdk
### Design and Analysis of CMOS Inverter using the sky130 pdk and various open source tools
------
The motive of this project is to experiment with the working of an inverter and understanding the parameters involved along with it. The design will utilise the models that are present under the skywater 130nm pdk and various open source tools such as, Xschem, NGSPICE, MAGIC, Netgen, etc.

The whole process starts with analysis of NMOS and PMOS devices, specifically the 1.8v standard models available inside the pdk to determine a common working W/L ratio and also the gm, ron and similar values. After this we start with the design of a CMOS inverter that includes schematic, measurement of various parameters like delays, noise margin, risetime, falltime, etc. This part would also act as a case study on SPICE where we use it's programming capabilities to better our abilities in measurements of aforementioned parameters. Then we will engage in the design a layout for our inverter in __magic layout editor__. Here, we will also explore the different layers available to the user and how we utilise them in a design and what it translates to in terms of a mask. Lastly, we compare the two netlists, that is the schematic and the layout one, which is popularly referred to as __LVS__. If both the netlits match,then this project would then conclude.

![collage1](https://github.com/user-attachments/assets/350c8ad1-383d-46f2-8154-03c4cde4d4af)

---

## Contents
- [1. Analysis of MOSFET models](#2-Analysis-of-MOSFET-models)
- [2. CMOS Inverter Design and Analysis](#3-CMOS-Inverter-Design-and-Analysis)
- [3. Layout design](#3-Layout-design)
- [4. Layout versus schematic](#4-Layout-versus-schematic)

## 1. Analysis of MOSFET models
   ### 1.1 General MOS Analysis
In this section I start with the analysis of MOSFET models present in sky130 pdk. I am using the 1.8v transistor models to experiment among many other ones present there. Below is the schematic I created in Xschem.

![nmos_sch](https://github.com/user-attachments/assets/365c4782-1730-403f-ae31-a0599033acc6)


The components used are:<br>
```nfet_01v8.sym``` - from xschem_sky130 library<br>
```vsource.sym``` - from xschem devices library<br>
```code_shown.sym``` - from xschem devices library<br>

I used the above to plot the basic characteristic plots for an NMOS Transistor, That is ___Ids vs Vds___ and ___Ids vs Vgs___. To do that, just save the above circuit with the above mentioned specifications and component placement. After this just hit __Netlist__ then __Simulate__. ___ngspice___ would pop up and start doing the simulation based calculations. It will take time as all the libraries need to be called and attached to the simulation spice engine. Once that is done, you need to write a couple commands in the ngspice terminal:<br><br>
```display``` - This would display all the vectors available for plotting and printing.<br>
```setplot``` - This would list all the set of plots available for this simulation.<br>
_after this choose a plot by typing '''setplot <plot_name>'''. for example '''setplot tran1'''_<br>
```plot``` - to choose the vector to plot.<br>
_example : plot -vds#branch_<br><br>

Then you must see the plot below you, if you did a DC sweep on the __VGS__ source for different values of __VDS__:<br>

![collage2](https://github.com/user-attachments/assets/4793cf79-5013-4d8b-8a6a-ea1fea47e1de)

This definitely shows us that the threshold value is between __600mV to 700mV__ and I think I will be using ___650mV___ for my future calculations.
Similarly, when I sweep __VDS__ source for different values of __VGS__, I get the below plot:<br>

![plot_(-vds#branch)](https://github.com/user-attachments/assets/49cc6595-46c7-4379-bd82-071aea05e423)

Same can be done for a ___PMOS___. Motive is same, but expecially to extract the value of Aspect ratio for which the current is the same in both NMOS and PMOS. I have done some experimentation and found that at __W/L of PMOS__ = __3.5 * (Aspect ratio of NMOS)1__, the current value is pretty close. So, we found the NMOS had a current of __317 microamps__ while PMOS has the current of __322 microamps__ (both at |Vgs| = 1.8V). So the difference is 5 microamps apart.

![pmos_collage](https://github.com/user-attachments/assets/09fb4c92-b075-481a-b774-5ea7c7de7128)

### 1.2 Strong 0 and Weak 1
What does the above mean? Look at the graph below,<br><br>
![pdn_nmos_sch](https://github.com/user-attachments/assets/a278ef30-d510-419e-b19b-9ebe7b9b62bd)
![collage3](https://github.com/user-attachments/assets/074a4b1a-591a-4d44-873a-f4512fa5d3b6)


You can see that, when a square wave is applied to the input of NMOS, when it is __LOW(0V)__, the output goes to __HIGH(1.8V)__. But when the input is __HIGH(1.8V)__, the output goes to a value that is much larger than 0V. This is due to the fact that when Vgs is 1.8V, the NMOS is in linear region. This is where the MOSFET acts as a voltage controlled resistor. At this point, the output is connected to a Voltage Divider Configuration. That is the output takes the value which is defined by the voltage across the resistance of the mosfet. Hence, ___NMOS is able to transmit STRONG 0, but not a STRONG 1. So NMOS is Strong 0 but a Weak 1___<br><br>

### 1.3 Weak 0 and Strong 1
Again, some plots will clear the idea<br><br>

![pun_pmos_sch](https://github.com/user-attachments/assets/3a425043-315d-43a4-b33d-9eab575a0696)
![collage4](https://github.com/user-attachments/assets/2a8cf4fc-abec-4c30-82fb-321c092e31e0)

The reasoning is the same as the previous section<br><br>

Hence, neither NMOS nor PMOS would make a great inverter on their own. So a plethora of configurations were taken into account, but at the last, only one stands as the most popular format of circuit design using mosfets. It is referred to as a CMOS configuration.

---

## 2. CMOS Inverter Design and Analysis
### 2.1 Why CMOS circuits

   In the previous section, we realised that neither NMOS nor PMOS can be used for design that can produce either values, HIGH and LOW. But they complement each other. This gave rise to an idea of attaching them together. Since, __PMOS__ is a __Strong 1__, we put it between VDD and Vout and __NMOS__ being a __STRONG 0__, it is placed between Vout and GND. This way, either can act as a load to the other transistor, since __both are never ON together__. This is referred to as __Complimentary Metal Oxide Semiconductor__(CMOS) Configuration and it also represents the simplest circuit known as the __CMOS Inverter__.<br><br>

CMOS Circuits generally consists of a network split into two parts, Upper one referred to as a __pull up network__ and the lower half as a __pull down network__. The former consists of P-channel MOSFETs and later N-Channel MOSFETs. Reason is simple. As one transistor is one, another is off. This eliminates the issue of an resistive path to the ground and hence, no voltage division occurs(At least not a significant one). This way, one can easily achieve a Strong High and a Strong LOW from the same network. __PULL UP__ is what offers a low resistance path to the VDD and __PULL DOWN__ is what offers a low resistance path the GND.

![cmos_image](https://github.com/user-attachments/assets/7dd8eb92-9246-4838-bfbd-3ca8a968828f)

### 2.2 CMOS Inverter Analysis Pre-Layout

In electronics an inverter is very popularly explained as something that performs the __NOT__ logic, that is complements the input. So a __HIGH(1.8V)__ becomes __LOW(0V)__ and vice versa. Ideally, the output follows the input and there is no delay or propogation issues of the circuit. But in reality, an inverter can be a real piece of work. It can have serveral isseus like how fast can it react to the changes in the input, how much load can it tolerate before it's output breaks and so many more including noise, bandwidth, etc.

All these parameters are what will affect the operation of transistors in general. Hence, with inverters, many like to explore them all. I'll first start with a schematic diagram, then I'll evaluate all the parameters, that is, measuring them, experimenting with them, and reaching a conclusive value, and finally I'll reach a schematic circuit that is capable of things we lay down at the beginning.

So I designed a Schematic of the Inverter, where the whole thing is based on what we determined earlier. I have chosen __(W/L) of PMOS = 3.5/0.15 times (W/L) of NMOS__ and __(W/L) of NMOS is 1/0.15 in microns__. I've also designed a symbol of it, so that we can utilise that for further schematic creation.<br><br>

![collage5](https://github.com/user-attachments/assets/939041f8-490b-414d-b8df-6e71f6c3f87d)
A lot of calculations will now start from this point. Similar to how we analysed for MOSFETs individually. Also from now on, ___(W/L)___ would be mentioned as ___S___ or ___Aspect Ratio___ Simply. We would use the following testbecnch for future analysis.(Transient and DC)<br>

![inv_sym_code](https://github.com/user-attachments/assets/13279dae-d6be-4880-9325-afc920ebae9d)

#### 2.3 DC Analysis and Important design parameters

DC analysis would be used to plot a Voltage Transfer Characteristics (VTC) curve for the circuit. It will sweep the value of Vin from high to low to determine the  working of circuit with respect to different voltage levels in the input. The following plot is observed when simulated :<br>
![collage6](https://github.com/user-attachments/assets/acdab38e-ef5d-471a-9b26-ff35f5765dfa)

A voltage transfer characteristics paints a plot that shows the behavior of a device when it's input is changed(full swing). It shows what happens to the output as input changes. In our case, for an inverter we can see a plot that is like a square wave(non ideal), that changes it's nature around 0.89 volts of input. So one can say that there are like 3 regions in the VTC curve, the portion where output is high, the place of transistion and the one where the output goes low. But actually there are __five regions of operation__ and they are based on the working of inverter constituents, that is the NMOS and the PMOS transistors with respect to the change in the input potential. <br>
![inverter-operating-regions](https://github.com/user-attachments/assets/452c2e50-32f0-4867-b923-a8dc34be2641)

One can solve for them using the equations for individual transistors. There are important parameters of this device that are based off it's VTC curve.
- __VOH__ - Maximum output voltage when it is logic _'1'_.
- __VOL__ - Minimun output voltage when it is logic _'0'_.
- __VIH__ - Minimum input voltage that can be interpreted as logic _'1'_.
- __VIL__ - Maximum input voltage that can be interpreted as logic _'0'_.
- __Vth__ - Inverter Threshold voltage
- __Vm__ - Switching Threshold

Above five parameters are critical for an Inverter and can be seen on the __VTC__ curve of an inverter. One thing to point out now would be,
<p align=center><i><b>Vth should be at a value of VDD/2 for maximum noise margins</b></i></p>  



Let's calculate gain for our circuit. Gain is ratio of the change in output voltage to the change in input voltage **Gain= _d_ Vout/_d_ Vin**. High gain makes the inverter less susceptible to **noise** and helps achieve faster switching times.
Noise Margin is a measure of a digital circuit's ability to tolerate noise or interference without causing errors in its output. It's essentially the difference between the minimum input voltage required to guarantee a high output and the maximum input voltage that will still produce a low output.
Here is the plot of gain when it crosses 1.
![gain](https://github.com/user-attachments/assets/18234c32-1e76-4009-8f4c-d3a79d9ef9d2)

__VOH__ and __VOL__ are easy to determine as they are your aboslute values. In our case it is __1.8V__ and __0V__ respectively. For __Vih__ and __Vil__, we have another method. At __Vin__ = __VIH__, NMOS is in Saturation region and PMOS in Linear; while when __Vin__ = __VIL__, NMOS is in Linear and PMOS in Saturation. Another interesting thing about these points is that, _these are the points on the curve, when the magnitude of slope = 1_. So we can use ```measure``` commands to find them on the plot. In the plot shown below, look at the points that are at the intersection of the vout curve and the blue vertical line. These are our __VIH__ and __VIL__.<br>

![gain_vout_vin](https://github.com/user-attachments/assets/e77c1ddc-88e8-4632-a043-58a732684090)

And to calculate them, we use ```.meas``` statement with apt instructions. The result is down below.<br>
![meas_vil_vih](https://github.com/user-attachments/assets/87997776-2747-4135-ac99-cae3f72dc025)

Let's summarize the values obtained :
| Voltage | Value |
|---------|-------|
| Vth_inv | 0.89V |
|   VOH   | 1.8V  |
|   VOL   |  0V   |
|   VIH   | 1.02V |
|   VIL   | 0.77V |

The basic defining characteristics of an inverter are done. So we can find a couple more things and then proceed towards the transient analysis. Next is **Noise Margins**. **Noise margins** are defined as the range of values for which the device can work noise free or with high resistance to noise.
There are two such values of Noise margins for a binary system:<br>
<b>NML(Noise Margin for Low) = VIL - VOL</b><br>
<b>NMH(Noise Margin for HIGH) = VOH - VIH</b><br>

So for our calculated values, the device would have, __NML = 0.77V__ and __NML = 0.78V__.

Now, they aren't equal. But if we were to take some more effort to get the values of Vth closet to Vdd/2 (0.9V), then we can get NML = NMH. But for our case they are close enough.

#### 2.3.2 Propagation delay

Now let's calculate the Propagation delay and its associated parameters such as tpHL,tpLH, tf and tr. **Propagation Delay** is the time it takes for a signal to travel from the input of a logic gate to its output. It's a crucial factor in determining the overall speed and performance of a digital circuit.

![propagation_delay](https://github.com/user-attachments/assets/544950c5-9668-4144-a51a-7e06198a7c58)

**tpHL (Propagation Delay from High to Low)** is the timke taken for the output of a gate to transition from a high state to a low state after a change in input from low to high. It can be measured from 50% of the input rising signal to the 50% of the output falling signal.

**tpLH (Propagation Delay from Low to High)** is the time taken for the output of a gate to transition from a low state to a high state after a change in input from high to low. It can be measured from 50% of the input falling signal to the 50% of the output rising signal.

**tf (Fall Time)** is the time it takes for the output of a gate to transition from 90% of its high state to 10% of its high state.  
**tf (Rise Time)** is the time it takes for the output of a gate to transition from 10% of its low state to 90% of its low state.

Let's calculate all parameters associated from the circuit that we have designed. We will also add an load to the circuit to determine the delay of the signals as the type of load used will also affect the delay of the circuit. We are using an **(0.2pF)** capacitor at the output. Using larger load will also increase the delay time.

![collage8](https://github.com/user-attachments/assets/f4eb1a51-a288-4db8-aac9-e39dd1d8cca4)


Below are the measurements of the differents parameters of the delay as calculated with Ngspice tool:
![collage7](https://github.com/user-attachments/assets/b11cf5cc-e14e-44d0-b8a9-f388ce456bc4)

| Delays  | Values |
|---------|------- |
|  tpHL   | 0.41ns |
|  tpLH   | 0.31ns |
|   tr    | 0.59ns |
|   tf    | 0.68ns |

Here by some experiments done by us, I have see noticed that:
- By increasing the vdd of the circuit, there will be improvemnts in the rise and fall times delay. Also if the vdd is increased then the power consumption will also increase due to the quadractic effect of **vdd^2**.
- The output load also plays a role in the delay time,bigger the load then the delay time will also be high since the power consumpation to charge the larger load will also increase.
  
#### 2.4 Power Analysis
Let's also calculate the average power consumed by the circuit when connected with a load capacitor (0.2pF). We know that the average power is the average of the instantaneous power over one period.
![instantaneous_power_formula](https://github.com/user-attachments/assets/d686a48f-d8b1-442c-8cf2-466d3f4ec543)

 Below is the calculated value that is obtained from the ngspice:

 ![power_analysis_values](https://github.com/user-attachments/assets/32f87e85-2997-4a65-b80d-d3256d2e230a)

From the data calculated from above we can see that for our circuit with a **PMOS to NMOS** ratio of **3.5:1** the average power drawn is **66.13uA**. 
Similarlly I have calculated for an capacitive output load (0.1pF) the average power consumption is **(33.70uA)**.
- Power consumption can be reduced by, reducing the outload load(CL).
- Reducing the switching activity also reduces the average power consumption.
---

## 3. Layout design
We have seen the representation of the inverter in the form of the schematic and analysis of delay and power with help of waveforms and plot measurement data from ngspice in the above section. Now we can represent the same inverter in the form of an layout. Layout design defines the physical arrangement of components and interconnections on the silicon substrate. It's the blueprint that guides the fabrication process. A schematic design alone cannot be directly used to manufacture a chip.
We have used the layout specification that was avaialble from the open source sky water sky130 PDK too design the layout of our inverter. Here the ratio of the **pmos to nmos** is taken as **2:1**. Open source tool **magic** is used as layout editor to create our layout reprsentation of inverter.

![collage9](https://github.com/user-attachments/assets/f7c6f1cb-bfeb-47e7-8dbe-8f78bf001f72)

## 4. Layout versus schematic (LVS)
So far, in our above sections we have discussed and represented the CMOS inverter in both schematic and layout forms, ensuring a thorough understanding of each representation. The schematic provides a high-level overview of the circuit’s functionality, while the layout translates this functionality into a physical form that can be fabricated on a silicon wafer. By breifly explaining each representation, we have laid a solid foundation for the next step in our design process: **the Layout Versus Schematic (LVS)** check. Now, we advance to the next crucial step: performing a **Layout Versus Schematic (LVS)** check. As informed in the beggining this project will conclude with the performing a LVS check. This section will elaborate the significance of **LVS** in VLSI design and detail the process of conducting an **LVS** check for our CMOS inverter.

The **LVS** check is a critical step in the VLSI design process, ensuring that the layout accurately reflects the schematic. In essence, **LVS** is a verification process that compares the netlist extracted from the layout with the netlist derived from the schematic. The goal is to confirm wheter the physical layout accurately represents the schematic, verifying that all connections and components are correctly implemented. 

**LVS** helps identify discrepancies between the schematic and layout, such as missing connections, incorrect component placements, or unintended shorts and opens. By confirming that the layout matches the schematic, the design is ready for fabrication, minimizing the risk of costly errors during the manufacturing process. **LVS** provides valuable feedback that can be used to optimize the design, improving performance, reliability, and yield.

**Netlist:**
The first step in the **LVS** check for the design is by extracting both the netlists of the schematic and the layout and comparing them with the help of tool. A **netlist** is a textual representation of a circuit. It describes the interconnection between components, such as transistors, gates, and other electrical elements. Each component in the netlist is assigned a unique identifier, and the connections between components are specified using net names. For our case i'm using a open source **Netgen** to compare both the netlists. 

Below if the netlist that is extracted from the schematic of inverter in the format of spice file:

![xschem_netlist](https://github.com/user-attachments/assets/105f43e4-c5f8-4bb9-9481-d4d2b2a3aba9)

The netlist of the layout is also shown below:

![layout_netlist](https://github.com/user-attachments/assets/d929feb3-2a70-4382-b3d0-e82717940422)

Now both the netlists can be compared with each other from the **Netgen** tkcon window. use the command ```lvs``` <filename.spice> <filename.spice>. 
After the tool runs the command you can get to see the result in a tkcon window like shown below:

![lvs_result](https://github.com/user-attachments/assets/8810d7fd-6810-4184-b105-f29d6c122ef9)

You can see from the above results which shows the number of devices that our schematic and layout design contains; and also the number of total wires our design has. 
If both netlits matchs, then the final output shows as: **Circuits match uniquely**. 
This final output results will be dumped into a single file namesd "comp.out". We can use a editor tool to view this file. The "comp.out" file contains a side by side comparison between both the netlist which is shown below:
![comp_out_result](https://github.com/user-attachments/assets/16993ff0-b5f9-4a63-accb-6104ca99b540)

In this project, we have comprehensively analyzed the MOSFET models and the CMOS inverter. We represented the CMOS inverter in both schematic and layout forms, providing detailed explanations for each. Finally we performed a Layout Versus Schematic (LVS) check to ensure the accuracy and integrity of our design. With these steps completed, I conclude this project.
