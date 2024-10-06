# LifeArc-Job-Simulation-Biology-Research

## Repository Overview

This repository contains the results and analysis for a series of tasks focused on optimizing neuronal differentiation protocols to generate nociceptor neurons for drug screening purposes. The tasks were part of a job simulation exercise aimed at optimizing differentiation conditions, assessing functional outcomes, and synthesizing findings from experimental data.

Each task involved different aspects of experimental design, analysis, and collaboration with team members. The results are organized in this repository along with supporting materials, such as code, plots, presentation slides, and Excel files containing raw data and processed results.

---

## Tasks Overview

### **Task 1: Optimizing Neuronal Differentiation Conditions**

- **Objective**: Identify the optimal multiplicity of infection (MOI) of NGN2 virus and assess the effect of NT3 supplementation on neuronal differentiation efficiency.
- **Methodology**: Logistic regression analysis was performed on the experimental data to evaluate the proportion of MAP2-positive neurons across different NGN2 MOI levels and with/without NT3.
- **Key Finding**: The optimal conditions for neuronal differentiation were determined to be **5 MOI of NGN2 virus combined with NT3 supplementation**, which maximized the proportion of mature neurons expressing the MAP2 marker.

> For detailed information, refer to the `Task1_Report.md` file and the accompanying Excel file in the `task1/` folder.

---

### **Task 2: Refining the Differentiation Protocol**

- **Objective**: Further validate the protocol by assessing the efficiency of generating nociceptor neurons specifically (beyond just MAP2-positive neurons).
**What I Learned**:
  - How to analyze experimental data to find the best treatment combination.
  - How to apply logistic regression, a key statistical framework.
  - How to use R and RStudio for data analysis.
- **What I Did**:
  - Analyze data using the provided "Data Analysis Notebook" in R.
  - Modify the code and answer questions based on the data analysis.
- **Key Methods**:
  - **TRKA Marker Immunostaining**: Tested for the expression of the nociceptor-specific marker TRKA in the differentiated neurons.
  - **Ca2+ Imaging with Mustard Oil**: Functional tests were performed to profile neuron responses to noxious stimuli (mustard oil) using Ca2+ imaging.
- **Key Finding**: NT3 increased MAP2-positive neuron differentiation but had a slightly lower efficiency in generating nociceptor neurons compared to the non-NT3 condition. The functional test revealed more active nociceptor responses without NT3.

> The key plot from Task 2 can be found in the `task2/` folder.

---

### **Task 3: Synthesizing Evidence and Collaboration**

- **Objective**: Collaborate with a team member (Ela) to synthesize the results from both structural (MAP2/TRKA) and functional (Ca2+ imaging) experiments. Provide recommendations on whether to continue using NT3 in the differentiation protocol.
- **Key Consideration**: While NT3 enhances overall neuron differentiation (MAP2-positive cells), it may reduce the proportion of functional nociceptors. Based on this evidence, a more balanced protocol may be required to optimize nociceptor neuron generation for drug screening.
- **Key Finding**: NT3 may still be beneficial for differentiation, but its use may need to be adjusted depending on the primary goal (efficiency in generating nociceptors vs. general neurons).

> See the email response to Ela and further analysis in the `task3/` folder.

---

### **Task 4: Presenting the Results**

- **Objective**: Present the findings from the fibroblast-to-sensory-neuron differentiation protocol optimization experiment in an informal, work-in-progress style presentation.
- **What I Learned**:
  - How to create a concise scientific presentation.
  - How to summarize experimental data and conclusions.
- **What I Did**:
  - Populate the presentation template with key data and conclusions.
  - Record an informal presentation (approximately 10 minutes) for a team meeting.
- **Key Context**: The presentation covers the optimization experiment designed in **Task 1** and additional data provided by your colleague, Ela. The goal is to summarize the current findings, including whether there is clarity on the optimal experimental conditions or if further experimentation is required for high-throughput drug screening.
  
> The presentation slides and recording can be found in the `task4/` folder.

---

## Structure of the Repository

- **task1/**: Contains the report and Excel file for Task 1 (Optimizing Differentiation).
- **task2/**: Contains the key plot and findings from Task 2 (Refining the Protocol).
- **task3/**: Contains the email correspondence with Ela and conclusions from Task 3 (Synthesis and Collaboration).
- **task4/**: Contains the presentation slides and recording from Task 4 (Presenting the Results).
- **README.md**: This file, which provides an overview of the repository.

---

## Acknowledgments
I would like to express my gratitude to **LifeArc** for providing such an insightful job simulation, complete with helpful resources and clear instructions. Additionally, I thank **Forage** for offering this job simulation on their platform, enhancing my skills and knowledge through this valuable experience.
