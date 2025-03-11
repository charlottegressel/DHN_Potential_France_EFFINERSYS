
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

<p align="center">
  <img src="https://github.com/user-attachments/assets/60dc4c04-d07a-4c33-85b1-661e88c2d5b8" width="700">
</p>

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

<p align="center">
  <img src="https://github.com/user-attachments/assets/7c162853-114c-46c4-89b8-70cc597041d2" width="500">
</p>

The connection process is implemented in the **Graph class** in the code, which includes several functions for initialization, radius calculation, network connection, and shortest path computation.

The entire process is summarized in the flowchart below.

![FIG_AlgoSummary](https://github.com/user-attachments/assets/3bc74ad0-5e43-4462-97d9-c9f1b2c3613c)

### Optimization method

After executing the mapping algorithm, some discrepancies may remain, such as roads being "too long". This is due to the fact that some projected buildings are projected on other roads than those which are actually connected to the network.

To resolve these issues and create an optimal network, we implement a two-step optimization process:

1. **Cutting**: Roads are segmented at the locations of projected buildings, following the method from the Pre-Cutting section.
2. **Optimization**: The algorithm removes unnecessary roads, specifically those positioned at the network's end that aren’t connected to any buildings. This is done by identifying extremal roads and deleting them.

<p align="center">
  <img src="https://github.com/user-attachments/assets/a2d59307-777a-4750-8ec8-ce17c5f5995b" width="700">
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/705814e8-e21b-4804-a6c2-b773252a4080" width="300">
</p>


This process significantly reduces the network size, improving its linear density and economic viability.

### Finding an optimal Lambda-constraint

In some cases, a high lambda constraint can prevent the network from expanding further, as no additional pipe length is included within the network radius. This is particularly true when the heat plant is located far from clustered buildings. To address this issue, we implement two methods that allow for a lower initial constraint, which can paradoxically result in a better linear density and necessarily more connected buildings. We aim to strike a balance between connectivity (heat coverage) and profitability (linear density), while also considering the municipality's priorities.

#### **Reinjecting the Reduction Percentage in the Linear Density Threshold**  
The algorithm initially applies a minimum linear density constraint of $\lambda = 1.5 \, \text{MWh.m}^{-1}\text{.y}^{-1}$ to ensure economic viability. After optimization, which reduces the network length by $x\%$, the new density constraint adjusts to $\lambda_x = \lambda \cdot (1 - x)$. This modification relaxes the constraint, allowing more buildings to connect while maintaining economically feasibility. 

<p align="center">
  <img src="https://github.com/user-attachments/assets/4a082873-5a5a-4863-a794-d95fde76ac71" width="300">
</p>

*For Breteuil-sur-Noye (Fig. above), the reduction reinjection method connects 7 more buildings to the network, while maintening a high economic viability.*

#### **Model Improvement: Finding Optimal Lambda**  
A fixed density constraint may be too restrictive in some areas, limiting the network's expansion. To address this, an optimized $\lambda$ value is determined by testing values from $0.1$ to $1.5 \, \text{MWh.m}^{-1}\text{.y}^{-1}$ and selecting the one that maximizes linear density. Additionally, a minimum number of connected buildings can be required to ensure sufficient coverage. This approach allows balancing economic efficiency with broader network reach. If no configuration meets both the economic and connectivity constraints, the network is rejected. This method improves network design by adapting to local conditions.

This method is also useful to build networks that cannot be created with $\lambda = 1.5 \, \text{MWh.m}^{-1}\text{.y}^{-1}$, as those defined earlier with a heat plant too far from buildings cluster.

<p align="center">
  <img src="https://github.com/user-attachments/assets/83f3481f-cf6e-4feb-a134-9426460d2b9c" width="300">
</p>

*For the case of Saint-Leu-d'Esserent above, a network could only be created a constraint inferior to* $\lambda = 1.4 \, \text{MWh.m}^{-1}\text{.y}^{-1}$ *, which is, moreover, the optimal lambda obtained.*

### Departmental Scale

#### **Departmental Scale Analysis**  
After refining the algorithm for a single commune, we extend the study to an entire department to assess its potential for district heating networks (DHNs). The algorithm is applied individually to each commune, using INSEE codes for organization. The computational complexity is $O(n_c \times n_{b,c})$ or $O(n_c \times n_{r,c})$, where $n_c$ is the number of communes, and $n_{b,c}$, $n_{r,c}$ are the number of buildings and roads per commune. Data preprocessing filters out buildings missing key attributes (e.g., construction year, height, or energy class), which may limit applicability in some regions.  

#### **DHN Creation and Optimization**  
A commune is considered viable for a DHN if it has more than **40 buildings exceeding a 30 kW threshold**, significantly reducing the number of eligible communes. The algorithm then iterates through the filtered set, generating DHNs and applying optimization methods to improve efficiency. Comparisons are made with real DHNs from France Chaleur to evaluate performance. Two key metrics are analyzed: **linear density** (economic efficiency) and the **heat demand coverage ratio** $\tau$, defined as:  

$$
\tau = \frac{Q_{net}}{Q_{tot}}
$$ 

if no DHN exists, and  

$$
\tau = \frac{Q_{net} - Q_{real}}{Q_{tot}}
$$  

if a real DHN is present. Municipalities can prioritize economic viability or heat coverage based on their goals. A negative heat coverage delta may indicate data limitations rather than poor model performance.

## Results

For the department of Oise, the following networks were created:

<p align="center">
  <img src="https://github.com/user-attachments/assets/d6049ef1-a130-4cf6-995f-c6e971a8ca56" width="500">
  <img src="https://github.com/user-attachments/assets/85d2aa18-eb02-44c3-8847-f6951c86556c" width="500">
  <img src="https://github.com/user-attachments/assets/6b543cc5-c71a-44cd-ae9c-b292660e0408" width="500">
  <img src="https://github.com/user-attachments/assets/6f08c2aa-d603-4a57-8679-f3daf5fc6443" width="500">
  <img src="https://github.com/user-attachments/assets/7f13ef82-c63e-4e20-9346-23c5ef8ccf18" width="500">
  <img src="https://github.com/user-attachments/assets/c35ef2c5-932f-449f-96d2-c617071bc4ff" width="500">
  <img src="https://github.com/user-attachments/assets/b3a4a513-1d44-4d65-9a8e-2e47d7c44f61" width="500">
  <img src="https://github.com/user-attachments/assets/ef28b102-34b8-45c6-a20a-dff2a4063d8b" width="500">
  <img src="https://github.com/user-attachments/assets/a3b9ac00-81ad-452a-bb4e-8925b4dbce24" width="500">
  <img src="https://github.com/user-attachments/assets/3b8f0683-f48b-4895-8f85-a3c0cbf8e322" width="500">
  <img src="https://github.com/user-attachments/assets/da60a667-6c85-4651-a4ef-704ec6f6f04f" width="500">
  <img src="https://github.com/user-attachments/assets/ec5120d8-b417-4032-9bb9-59965e0aa377" width="500">
</p>


And with comparison to real DHNs:

<p align="center">
  <img src="https://github.com/user-attachments/assets/f00d2e6a-5dad-41a8-9fa6-9be567e9a911" width="500">
  <img src="https://github.com/user-attachments/assets/bcd451c8-b304-474f-a68d-422f903e30fd" width="500">
  <img src="https://github.com/user-attachments/assets/16cda20d-a6ad-4452-8b5b-39f793bc171d" width="500">
  <img src="https://github.com/user-attachments/assets/685bf3e7-196d-41c2-be9e-e736816cf373" width="500">
  <img src="https://github.com/user-attachments/assets/9093fb78-88e3-4d64-b93f-75ce7a9a57e6" width="500">
</p>

**Results Summary**

At the departmental scale, for the 24 cities in Oise identified as suitable for district heating network development, our results indicate a **$56.09%$** coverage of the heat demand for buildings exceeding a $30$ kW maximum power threshold, significantly surpassing the current $19.14%$ covered by existing DHNs in the department. Additionally, our algorithm produces highly economically viable networks, achieving an average linear heat density of **$2.36$ GWh/km/year**, well above ADEME's minimum threshold of $1.5$ GWh/km/year. These results demonstrate the significant potential of the Oise department, both in terms of heat coverage and economic efficiency.

The departmental approach also allows for the identification of high-potential cities at the local scale, including Beauvais, Breteuil, Chambly, Chantilly, Nogent-sur-Oise, and Grandvilliers. These cities show an exceptional balance between linear density and heat coverage, consistently surpassing **$2.00$ GWh/km/year and $40%$**, respectively.

Overall, our tool effectively optimizes and expands district heating network development in Oise, unveiling France's growing potential for heat coverage. However, the unique strength of our model lies in its ability not only to estimate potential in terms of heat coverage but also to assess economic potential, pinpointing highly promising and specific areas. Consequently, this study offers a comprehensive overview of heat potential at the departmental level while also capturing the nuanced specificities at the local scale and adapting to the municipality's objectives and constraints.
