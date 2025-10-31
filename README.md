# IESA-Opt_ModelLinking
This repository contains the model versions and structure needed for hard-linking IESA-Opt-N with the AdOpt-NET0 cluster model. See the repository below for the AdOpt-NET0 model, linking code, and linking instructions.
https://github.com/julia1071/AdOpT-NET0_ModelLink

In the linking code, update the file paths to where you have stored the IESA-Opt_ModelLinking repository. The structure of this repository is set up so that hard-linking is possible via a python script. This script does not allow for changes within the AIMMS User Interface, so each of the two versions of IESA-Opt (Fossil Phaseout and Policy Path) corresponds to a scenario. The model code is identical, but default data inputs differ. Data files for the two scenarios contain different data (though file names are identical): IESA-Opt-N_linking.xlsx.
 
# IESA-Opt-N
**IESA-Opt-National (Dutch database)**
## Running the model
You need some pre-requistics before running the model:
- The IESA-Opt-N model is written in AIMMS. Thus, you need to install this windows-based application to be able to run the model. You can obtain your free academic licence from this link: https://licensing.cloud.aimms.com/license/academic.htm
- Although you can choose any mathematical solver for the solving procedure, the IESA-Opt-N model runs best with the Gurobi mathematical solver. You can obtain your free academic licence from this link: https://www.gurobi.com/academia/academic-program-and-licenses/

Please feel free to contact us if you face issues. 

## Model description

### From IESA-Opt to IESA-Opt-N
We use the IESA-Opt model implemented for the Netherlands to capture system-wide effects. This is a detailed open-source optimization ESM at the national level. IESA-Opt models investments of the energy system over the horizon from 2020 to 2050 in 5-year time steps while simultaneously accounting for hourly and daily operational constraints. The model's objective function minimizes the net present value of energy system costs to achieve total energy needs under certain techno-economic and policy constraints (e.g., a specific greenhouse gas (GHG) reduction target in a particular year). It is an open-source and flexible model that can be used for other regions or countries (e.g., the North Sea region).

In the IESA-Opt model, the operation of the electricity sector of the Netherlands and other EU countries (including Norway and Switzerland) is balanced hourly. Since the model's scope is at the national level, power sector investments occur only in the Netherlands. At the same time, the power capacity mix of EU nodes is fixed as exogenous scenario parameters. 

The energy infrastructure is modeled in ten networks for different voltage levels of electricity, and different pressures of natural gas, hydrogen, and single carbon capture, utilization, and storage (CCUS) and heat networks. The gaseous networks are balanced daily due to their relatively low intraday variation. 

The IESA-Opt model reflects the emission constraints of the EU Emission Trading System (ETS), the non-ETS sectors, and the international navigation and aviation sectors. Since ETS sector emissions are traded in the EU ETS market, we assume an exogenous ETS emission price projection as a scenario parameter. Because the national emission reduction policy targets both ETS and non-ETS sectors, we set the aggregate national emission constraint on both sectors. If the constraint is binding, the model generates an aggregated national emission shadow price, equal to the marginal increase in the system cost if the aggregated emission constraint gets one unit tighter. 

### The IESA-Opt-N model
Although IESA-Opt comes with several capabilities, it has some limitations. Therefore, this study modifies the model in two directions: objective function definition and cross-border electricity trade. The modified model is IESA-Opt-N, which stands for Integrated Energy System Analyses – Optimization – National.  

**Objective function definition**

The IESA-Opt model’s objective function refers to the system’s net present value resulting from the set of decision variables confirmed by annualized investments, decommissioning, retrofitting, and use of technologies. However, this objective function does not account for the cost of technology stock in periods after the investment period. While, the fixed operational and variable operation costs related to the invested technology continue to affect the objective function in the subsequent periods. In this formulation, the weight of investment decisions in earlier periods is undervalued. Therefore, the system tends to make more significant investments in earlier periods as it does not pay for the annualized capital cost in successive periods. Although this formulation minimizes the net present value of investment decisions, it does not minimize the system costs of the energy system. 

Therefore, we modify the objective function by adding the investment matrix before the capital component to represent total system costs. The binary investment matrix determines the presence of a technology option in each period based on its economic lifetime. 

Moreover, we add a social discount factor (SDF) to account for the net present value of costs at each period. This discount factor is based on the assumed exogenous social discount rate that describes how society values future investments. The social discount rate should not be confused with the capital depreciation rate. The capital depreciation rate or Weighted Average Cost of Capital (WACC) is used to annualize the overnight capital investment costs. Although WACC can be different for each technology, we assume a 5% rate for all technologies in the reference scenario. Thus, with the addition of the social discount rate, the new objective function calculates the sum of the net present value of energy transition costs:

![image](https://user-images.githubusercontent.com/63007753/162712451-14900ad1-9d37-48d5-b552-e02a8066fda9.png)

With the new objective function formulation, the capital cost of technologies is accounted for during their economic lifetime. However, the investments in the last modeling period may be distorted as the benefits of investments after this period are neglected. Since 2050 is crucial in current policies, we do not want these so-called end-of-horizon effects in 2050. Although this effect is already reduced by using annualized investment costs, we add two more periods (i.e., 2055 and 2060) to the model’s horizon to further reduce this effect. Since the additional periods aim to represent investment costs better, all energy system definitions, including activity levels and technological costs and potentials, are kept equal to their value in 2050. 

**Cross-border electricity trade**

The IESA-Opt model optimizes the hourly operation of the electricity sector of the Netherlands and other EU countries (including Norway and Switzerland). This requires the evolution of EU generators and interconnection capacities as input to the model. These exogenous values were obtained from the Ten Year Network Development Plan of ENTSO-E. However, the range for capacities is relatively high across different scenarios. Moreover, the power generation plan of each EU member state can vary significantly in time as it is strongly tied to political agendas. Therefore, we decided to decouple these uncertainties from the IESA-Opt model by removing the EU capacities.

The IESA-Opt-N model can use the cross-border electricity trade profile as an exogenous input. This profile determines the hourly availability and price of electricity at each period. Furthermore, the profile can get imported from other power system models (e.g., COMPETES and PyPSA-Eur). This method has two main advantages compared to IESA-Opt: first, the impact of the EU power system on the national system is quantified and measurable, and second, the computational load is lower, and thus the run-times are significantly quicker. However, it comes with one primary disadvantage: the inconsistency between the assumptions of national energy system and international power system models.  

The electricity import and export prices and availability profiles vary depending on the underlying assumptions of the Netherlands and its neighboring countries' scenarios. Since the profiles can vary in many directions (i.e., hourly prices multiplied by hourly availabilities), performing a sensitivity analysis is complex. Moreover, measuring the impact of profile variations on national power generation decisions can be problematic. Therefore, we use a flat price to import and export electricity in this study. Moreover, we set a maximum import and export quota for each year. Thus, the model can decide how much to trade at each hour of the period, considering the total trade volume is less than the assigned quota for that period. 

In summary, the modifications improve the solution’s accuracy considerably (mainly by improving the objective function definition) while increasing the solution’s stability and reducing the solving times substantially. Figure 1 demonstrates the visual methodological framework of the IESA-Opt-N model.

![image](https://user-images.githubusercontent.com/63007753/162712646-bae3845f-9e2a-40c5-94d6-938eefe6c7a7.png)

Figure 1. The methodological framework of the IESA-Opt-N model
