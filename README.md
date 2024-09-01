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

---Hence, neither NMOS nor PMOS would make a great inverter on their own. So a plethora of configurations were taken into account, but at the last, only one stands as the most popular format of circuit design using mosfets. It is referred to as a CMOS configuration___

---

## 2. CMOS Inverter Design and Analysis
### 2.1 Why CMOS circuits

   In the previous section, we realised that neither NMOS nor PMOS can be used for design that can produce either values, HIGH and LOW. But they complement each other. This gave rise to an idea of attaching them together. Since, __PMOS__ is a __Strong 1__, we put it between VDD and Vout and __NMOS__ being a __STRONG 0__, it is placed between Vout and GND. This way, either can act as a load to the other transistor, since __both are never ON together__. This is referred to as __Complimentary Metal Oxide Semiconductor__(CMOS) Configuration and it also represents the simplest circuit known as the __CMOS Inverter__.<br><br>

CMOS Circuits generally consists of a network split into two parts, Upper one referred to as a __pull up network__ and the lower half as a __pull down network__. The former consists of P-channel MOSFETs and later N-Channel MOSFETs. Reason is simple. As one transistor is one, another is off. This eliminates the issue of an resistive path to the ground and hence, no voltage division occurs(At least not a significant one). This way, one can easily achieve a Strong High and a Strong LOW from the same network. __PULL UP__ is what offers a low resistance path to the VDD and __PULL DOWN__ is what offers a low resistance path the GND.

![cmos_image](https://github.com/user-attachments/assets/7dd8eb92-9246-4838-bfbd-3ca8a968828f)

### 3.2 CMOS Inverter Analysis Pre-Layout

In electronics an invereter is very popularly explained as something that performs the __NOT__ logic, that is complements the input. So a __HIGH(1.8V)__ becomes __LOW(0V)__ and vice versa. Ideally, the output follows the input and there is no delay or propogation issues of the circuit. But in reality, an inverter can be a real piece of work. It can have serveral isseus like how fast can it react to the changes in the input, how much load can it tolerate before it's output breaks and so many more including noise, bandwidth, etc.

All these parameters are what will affect the operation of transistor in general. Hence, with inverter many like to explore them all. I first start with a schematic diagram, then I evaluate all the parameters, that is, measuring them, experimentin with them and reaching a conclusive value and Finally reach a schematic circuit that is capable of things we lay down at the beginning.

So I designed a Schematic of the Inverter, where the whole thing is based on what we determined earlier. I have chosen __(W/L) of PMOS = 3.5 times (W/L) of NMOS__ and __(W/L) of NMOS is 1/0.15 in microns__. I also designed a symbol of it, so that we can utilise that for further schematic creation.<br><br>

![collage5](https://github.com/user-attachments/assets/939041f8-490b-414d-b8df-6e71f6c3f87d)
A lot of calculations will now start from this point. Similar to how we analysed for MOSFETs individually. Also from now on, ___(W/L)___ would be mentioned as ___S___ or ___Aspect Ratio___ Simply. We would use the following testbecnch for future analysis.(Transient and DC)<br>

![inv_sym_code](https://github.com/user-attachments/assets/13279dae-d6be-4880-9325-afc920ebae9d)

#### 2.3 DC Analysis and Important design parameters

DC analysis would be used to plot a Voltage Transfer Characteristics (VTC) curve for the circuit. It will sweep the value of Vin from high to low to determine the  working of circuit with respect to different voltage levels in the input. The following plot is observed when simulated :<br>
...image...

A voltage transfer characteristics paints a plot that shows the behavior of a device when it's input is changed(full swing). It shows what happens to the output as input changes. In our case, for an inverter we can see a plot that is like a square wave(non ideal), that changes it's nature around 0.89 volts of input. So one can say that there are like 3 regions in the VTC curve, the portion where output is high, the place of transistion and the one where the output goes low. But actually there are __five regions of operation__ and they are based on the working of inverter constituents, that is the NMOS and the PMOS transistors with respect to the change in the input potential. <br>
![inverter-operating-regions](https://github.com/user-attachments/assets/452c2e50-32f0-4867-b923-a8dc34be2641)

One can solve for them using the equations for individual transistors. There are important parameters of this device that are based off it's VTC curve.
- __VOH__ - Maximum output voltage when it is logic _'1'_.
- __VOL__ - Minimun output voltage when it is logic _'0'_.
- __VIH__ - Maximum input voltage that can be interpreted as logic _'0'_.
- __VIL__ - Minimum input voltage that can be interpreted as logic _'1'_.
- __Vth__ - Inverter Threshold voltage
- __Vm__ - Swithcing Threshold

Above five parameters are critical for an Inverter and can be seen on the __VTC__ curve of an inverter. One thing to point out now would be,
<p align=center><i><b>Vth should be at a value of VDD/2 for maximum noise margins</b></i></p>  
