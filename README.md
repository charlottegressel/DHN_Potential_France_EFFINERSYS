
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

The methodology is conducted in two main stages: first, the mapping algorithm is developed and optimized at the **local scale** using data pre-processing, network mapping, and post-optimization processes applied to both the generated network and the optimal mapping constraint. In the second stage, the finalized algorithm is scaled up to the **departmental level**. Specifically, we selected the department of Oise and several cities within it as case studies. An additional Case Study was also pursued in the end of the research project, in order to test specific changes into the algorithm.

### Local Scale

#### Data Preprocessing

We begin by assessing the heat demand of each building using its DPE class and the following formula:

$$ 
Q \approx DPE \times S \times E\left(\frac{h}{h_0}\right) 
$$

To simulate a district heating network using **Dymola**, it is essential to estimate the **instantaneous heat demand** of buildings throughout the year. This is done by comparing the buildings' heat demands with typical demand profiles.  

We use three datasets that correspond to **standard demand curves** for buildings constructed during different periods:  

- **Before 1989 (excluding 1989):** [`RT_1974_Treated.csv`](RT_1974_Treated.csv) *(based on building regulations from 1974 to 1989)*  
- **Between 1989 (including) and 2005 (excluding):** [`RT_1989_Treated.csv`](RT_1989_Treated.csv) *(reflecting standards for buildings constructed between 1989 and 2005)*  
- **After 2005 (including 2005):** [`RT_2005_Treated.csv`](RT_2005_Treated.csv)  

These datasets are available in this **GitHub repository**.

**Buildings Filtering**

A criterion for selecting buildings to be connected to the district heating network is defined on the France Chaleur website. According to this guideline, beyond a peak power demand exceeding 30kW, a building should generally be considered for network integration. We thus filter the buildings above this threshold.

**Roads Pre-cutting**

Basically, roads are divided without considering the buildings' positions. To fix this, we implement a pre-cutting algorithm which consists in :
1) Merging roads untill intersection;
2) Projecting buildings points on roads and cutting roads on these projected points.
Now the data is ready for use.

An example of the pre-cutting application is given on the figure below for the city of Bretuil-sur-Noye.

![FIG_Breteuil_PrecuttingProcess](https://github.com/user-attachments/assets/60dc4c04-d07a-4c33-85b1-661e88c2d5b8)


#### Mapping Algorithm

To model the **District Heating Network (DHN)**, we define three types of nodes: **buildings, roads, and the heat production plant**. Each node is assigned attributes such as heat demand, spatial coordinates, and connectivity properties. Buildings are connected to their nearest roads, and roads are linked based on proximity, forming the network's structure.  

Edges represent possible pipe connections, ensuring efficient heat distribution. The **Dijkstra algorithm** is then applied to optimize network expansion, prioritizing either **heat demand** (highest consumers first) or **proximity to the heat plant**. Connections follow strict rules to balance **cost efficiency and computational performance**.  

This graph-based approach enables effective DHN planning, minimizing infrastructure costs while ensuring optimal heat distribution.

The **Dijkstra Algorithm** is used to iteratively connect buildings to a **District Heating Network (DHN)**, starting with those having the highest heat demand. A **priority order** ensures that buildings with the largest heat demand are connected first, but this order can be adjusted for specific constraints, such as geographical challenges.

Each building has a **characteristic radius** defined by its heat demand $Q$, and the **radius $R$** is calculated as:

$$
R = \frac{Q}{\lambda}
$$

Where:
- $Q$ is the annual heat demand of the building
- $\lambda = 1.5 \, \text{MWh.m}^{-1}\text{.y}^{-1}$ is the minimum linear heat density required for a network to be economically viable and eligible for subsidies.

The **radii constraint** ensures economic viability by comparing the available pipe length with the building's required connection distance. For a building $A$, the connection is viable if the shortest distance $d$ to the heat plant satisfies the following condition:

$$
d \leq R_A + R_{\text{network}}
$$

Where:
- $R_A$ is the characteristic radius of building $A$
- $R_{\text{network}}$ is the current radius of the network, initially set to zero.

If the connection is feasible, the radius of the network is updated by adding the excess pipe length:

$$
R_{\text{network}} \leftarrow R_{\text{network}} + R_A - d
$$

The algorithm uses **iterative loops** to connect buildings step by step. Buildings that do not meet the economic viability criterion are added to the queue for future consideration. This process ensures that the DHN layout is economically viable and meets the required heat demand coverage.

The connection process is implemented in the **Graph class** in the code, which includes several functions for initialization, radius calculation, network connection, and shortest path computation.

The entire process is summarized in the flowchart below.

![FIG_AlgoSummary](https://github.com/user-attachments/assets/3bc74ad0-5e43-4462-97d9-c9f1b2c3613c)

### Optimization method

After executing the mapping algorithm, some discrepancies may remain, such as roads being "too long". This is due to the fact that some projected buildings are projected on other roads than those which are actually connected to the network.

To resolve these issues and create an optimal network, we implement a two-step optimization process:

1. **Cutting**: Roads are segmented at the locations of projected buildings, following the method from the Pre-Cutting section.
2. **Optimization**: The algorithm removes unnecessary roads, specifically those positioned at the network's end that aren’t connected to any buildings. This is done by identifying extremal roads and deleting them.

This process significantly reduces the network size, improving its linear density and economic viability, as shown in the figure comparison below for the city of Breteuil-sur-Noye, before (left) and after (right) optimization.

![FIG_Breteuil_OptimizationComparison](https://github.com/user-attachments/assets/27544b70-f3e2-4015-844b-4f69885ebdd3)



## Results
