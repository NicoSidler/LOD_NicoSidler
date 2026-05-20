
## Research Agenda

Taking an exploratory approach, we aim to propose a simplified representation of the structure of the scientific field of astronomy and its evolution over time, based on Pierre Bourdieu's concepts of "[field](https://fr.wikipedia.org/wiki/Champ_(sociology))" and "[habitus](https://fr.wikipedia.org/wiki/Habitus_(sociology))". We will focus on the socio-demographic profiles and intellectual positions of the agents, as well as their organizational membership and the resulting networks of relationships between actors and organizations. We will also endeavor to propose a model of how this structure evolves over time. Furthermore, as part of a prosopographic approach, we will systematically collect data on the characteristics of the agents to highlight their biographical and activity profiles.

## Research questions
* **RQ1 (Gender Distribution):** How does the representation of sociologists in the dataset vary by gender across the entire network?
* **RQ2 (Geographic & Generational Shift):** What patterns emerge when analyzing the continents of birth for sociologists born within the 1801–1990 window, and how do these geographic trajectories differ between gender groups and specific birth cohorts?
* **RQ3 (Structural Overlap):** To what extent do the structural networks of education, employment, and institutional membership overlap for sociologists, and does this core intersection display significant variation based on gender?
* **RQ4 (Institutional Pipelines):** How do historical time periods/cohorts explain the variation in how sociologists move from their educational roots into specific employment sectors and professional memberships?
* **RQ5 (Profiles of Centrality):** Who are the most central actors (gatekeepers or bridges) within the employment and membership networks according to betweenness and degree centrality metrics?
* **RQ6 (Gendered Capital):** Do structural centrality scores reveal systemic differences between male and female sociologists, suggesting gendered advantages or barriers in institutional positioning?
* **RQ7 (Sociological Tribes):** What distinct network clusters or communities emerge when looking at shared educational backgrounds or workplace co-occurrence?
* **RQ8 (Sub-community Profiles):** What do these identified clusters tell us about the intellectual or institutional division of labor in sociology? For instance, are certain clusters dominated by specific gender groups, geographic regions, or generational cohorts?


## Project Workflow & Network Construction

### 1. Data Harmonization & Cleaning
* **Temporal Ordering:** Filtered out and corrected chronologically inversed pairs in the primary dataset based on activity periods (`per_activ_x` and `per_activ_y`) to ensure proper directional/chronological logic before layer creation.
* **Metadata Integration:** Merged academic node attributes including gender, birth year, country of origin, and active historical periods from institutional datasets.

### 2. Multi-Layer Graph Construction
* **Education Layer (`edu_gu`):** Built a network mapping shared educational trajectories and academic pipelines among sociologists.
* **Employment Layer (`empl_gu`):** Constructed a co-occurrence network capturing shared institutional affiliations and workplace connectivity.
* **Membership Layer (`memb_gu`):** Established an institutional network tracking overlapping professional society memberships and academic associations.

### 3. Structural Alignment & Core Intersection
* **Multilayer Core Extraction:** Isolated the structurally aligned subgraphs (`GA_edu`, `GA_empl`, `GA_memb`) using the intersection of nodes present across all three operational dimensions.
* **Network Analysis Boundaries:** Set safety thresholds to handle varying graph sizes dynamically and prevent calculation runtime faults when computing metrics across smaller intersection populations.

### 4. Advanced Network Topologies & Centrality
* **Approximate Betweenness Centrality:** Calculated structural gatekeeping and systemic bridge scores across the aligned layers using a randomized node-sampling approach ($k$-value capped dynamically at $\min(500, N)$ to safely handle population limitations).
* **Community Profiles:** Exported the multi-layer graph topologies directly to Gephi formats (`.gexf`) for exploratory spatial layouts, community clustering, and high-fidelity sociogram visualizations.
    
