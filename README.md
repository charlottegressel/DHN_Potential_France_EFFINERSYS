# Assessing DHN potential in France

This work implements a method for assessing **DHNs' implementation potential in France**, using a mapping and optimization algorithm accross the French territory. In this application, we first apply our tool at the local scale, and then expand it to the department of Oise.

## Introduction

A District Heating and/or Cooling Network (DHCN) is a system distributing heat and cold energy to several users, with better operation and maintenance than individual heating/cooling solutions and fostering the integration of waste and renewable heat sources.

District heating networks have emerged as a crucial component in the transition towards sustainable urban energy systems in France. With the country's ambitious targets for reducing greenhouse gas emissions and increasing the share of renewable energy sources, DHNs offer significant potential for efficient heat distribution and integration of low-carbon energy sources. Today, these networks are powered 66\% by renewable sources and are the most affordable heating option for collective housing, ahead of gas, electricity, and fuel oil.

## Data Preparation

**Data Sources:**

The study is based on two main data files:  

- **Building dataset:** A dataset for a selected department, downloaded from the **Géoservices BD TOPO®** site (**BDNB database**) in **Geopackage** format.  
- **Road dataset:** A dataset for the same department, downloaded from the **Géoservices BD CARTO®** site in **Geopackage** format.  

Both files were processed using **QGIS** to extract and export only the relevant attributes for analysis:  

- **Buildings:** "*Bâtiments groupés → Classe DPE (DPE réels)*" attribute
- **Roads:** "*BDT_3-1 → tronçon_de_route*" attribute

## Methodology

## Results
