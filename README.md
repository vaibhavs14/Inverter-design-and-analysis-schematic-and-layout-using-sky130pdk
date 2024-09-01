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

efewfevf

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

  
