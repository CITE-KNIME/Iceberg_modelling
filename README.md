# Iceberg_modelling
NOTE: KNIME workflows used to conduct iceberg modelling; versioned based on the publication in which respective workflow was first used

## Description and instructions for KNIME workflow

A KNIME installation is required in order to run the workflows. The workflow can be imported into KNIME by importing each KNIME archive file (.knwf). The workflow has been established and tested with KNIME version 4.2.4. You may need to install various KNIME extensions in order to run the workflows.

### Scientific Background of Iceberg Modelling
Iceberg modelling using the bioanalytical equivalent concentration (BEQ) approach can be applied to determine the contribution of detected chemicals to the observed effect 1. BEQs derived from cell-based bioassays (BEQbio) relate the effect of the sample to the effect induced by the assay reference compound, whereas BEQs from chemical analysis (BEQchem) are calculated from the detected concentration of a chemical in a sample and its relative effect potency (REPi). The BEQ concept is based on the assumption that many chemicals in a mixture act in a concentration-additive manner.

### Use of components
In KNIME, components are nodes that contain a sub-workflow, which allows to bundle functionalities and to provide an own configuration dialog for each component. For more information, please see https://docs.knime.com/2020-07/analytics_platform_components_guide/index.html#introduction . In the iceberg modelling, the components are used to allow the selection of columns containing the information of interest, allow the user to define units used for the results of chemical analysis, allow the selection of the assay to be used for the iceberg modelling and so on. The input tables are more flexible in this way since the workflow is not depending on a defined column name. For more information, please see the example of the configuration dialog which allows the selection of the sample data of the results from chemical analysis. The dialog allows the user to select the sample data whose names differ between projects and eliminates the need to use standardized sample names in order to run the workflow.<br>
All component nodes are highlighted in yellow and should be configured in each run. Configuration only needs to be once, if the input is not changed between runs (e.g. same set of data for chemical analysis).

<img src="https://user-images.githubusercontent.com/108472923/176861969-e5624a45-4d48-4099-b411-51d9e584e4db.png" width=80% height=80%>

### Input files required to run the KNIME workflow (Input format is xlsx/xls)

NOTE: Unless otherwise specified, the exact spelling of the column names is not important, since the columns can be specifically selected by users in the workflow.

*File 1: Parameters for baseline QSAR* <br>
Baseline toxicity can be calculated on equation (1). Thus, the parameters a, b and c, which are specific for each cell line, have to be provided. For this study, these parameters are based on the publication of Lee et al 2021 2. The data have to be structured as follows:

<img src ="https://user-images.githubusercontent.com/108472923/191439029-4c167d00-9af9-438a-a714-85c6c41bc535.png">

*File 2: Parameters for references of bioassays* <br>
In order to calculate the REPi and the BEQbio, the EC10 or ECIR1.5 as well as the molecular weight of the respective reference chemicals need to be provided. The data have to be structured as follows:

<img src ="https://user-images.githubusercontent.com/108472923/191439267-b47e16e8-e5b5-425f-963f-809fc0f3969a.png">

*File 3. Chemical identifier, logDlip/w, effect and cytotoxicity results of single chemicals in bioassays* <br>
The workflow is using the InChiKey (column needs to be named INCHI_key) and the DTXSID (column needs to be named DTXSID) as identifiers to match the results of the bioassays for the single chemicals with the chemicals analysed. In addition, this file has to contain the log Dlip/w, the ECy and the ICy for each chemical and each assay of interest. The Excel tabel has to start with cell A1.

*File 4: Results of bioassay(s) for samples* <br>
This file provides the ECy and the ICy of each sample. Please note that the sample codes must be the same as the sample codes used in file 5. Results for each sample are given row-wise. For an example see the following:

<img src ="https://user-images.githubusercontent.com/108472923/191440167-f8d91b65-3283-41c2-88e7-a8eb5a2dc642.png">

*File 5: Results of chemical analysis for samples* <br>
This file lists the concentration of the chemicals analysed in the samples in ng/L. The InChiKey (column needs to be named INCHI_key) and the DTXSID (column needs to be named DTXSID) have to be provided as identifiers for the chemicals to match the data with the input from file 3. Please note that the sample codes must be the same as the sample codes used in file 4. For an example see the following:

<img src ="https://user-images.githubusercontent.com/108472923/191441835-5ad28fd4-632e-4125-ad78-de558146c9b2.png">

### Principle for merging the data related to chemicals
The input from file 3 and 5 contains data per chemical. In order to join these data sets the InChiKey and the DTXSID have to be provided as identifiers. To make the matching based on the InChiKey more robust only the 1st block of the InChiKey is used. In order to match chemicals for which no InChiKey is available the data are also matched based on the DTXSID. The two different joins are then concatenated and duplicates originating from the two-tier joining process are automatically curated. Chemicals which do not have any of the two identifiers mentioned above are not merged at all and are excluded from the data set. 

### Equations used for the Iceberg Modelling
For more detailed information on the iceberg modelling and the equations refer to Neale et al 2020 and Lee et al 2021.<br> 
Calculations are performed in either Math Formula or Math Formula (Multi Column) Nodes. These two nodes are wrapped in Metanodes if some kind of data processing is necessary in order to run the calculation. In the KNIME workflow, the nodes are annotated in blue and labelled with the numbers for the equations used below. Metanodes containing calculations are highlighted in blue.

***Equation 1: Calculation of baseline toxicity***<br>
Baseline toxicity was only calculated if the logDlip/w of a chemical was larger than 0, since the chemical is classified as “too hydrophilic”, if the logDlip/w is lower than 0.

<img src ="https://user-images.githubusercontent.com/108472923/176865617-91c96d5e-6916-4f40-af4f-83e0dc08e12c.png" width=40%>

***Equation 2: Calculation of the relative effect potency of a chemical I (REPi)***

<img src = "https://user-images.githubusercontent.com/108472923/176868828-1da1a4b8-9bc1-4d73-b58c-b1b7cb9c5979.png" width=20%>

***Equation 3: Conversion of the concentration of a chemical in a sample from ng/L to mol/L***

<img src = "https://user-images.githubusercontent.com/108472923/176869182-dd4686bc-b91c-4501-ba50-9c2d76c9e92a.png" width=20%>

***Equation 4a: Calculation of BEQchem in ng/L of for each chemical***

<img src = "https://user-images.githubusercontent.com/108472923/176871606-64a8a96e-97ca-414e-989a-601b1b62270d.png" width = 30%>

***Equation 4b: Calculation of BEQchem in ng/L of all chemicals in a sample***

<img src = "https://user-images.githubusercontent.com/108472923/176871770-ee9c8db3-3f9b-44f3-8c32-6490cd0db5a5.png" width = 20%>

***Equation 5: Calculation of BEQbio in ng/L***

<img src = "https://user-images.githubusercontent.com/108472923/176871872-4e129e57-b57a-466a-b5d7-89e355c34b94.png" width = 40%>

***Equation 6: Calculation of % of effect explained***

<img src = "https://user-images.githubusercontent.com/108472923/176871986-ac3d5774-2166-49a6-b829-8c70ed5599ec.png" width=35%>

***Equation 7: Calculation of TUbio_cytotoxicity***

<img src = "https://user-images.githubusercontent.com/108472923/176872068-f107ff0c-f852-47e4-9f97-912d3ced539c.png" width=30%>

***Equation 8a: Calculation of TUchem_cytotoxicity of each chemical***

<img src ="https://user-images.githubusercontent.com/108472923/176872231-cab5037e-2fad-4496-81d0-ade69d0b424c.png" width=25%>

***Equation 8b: Calculation of TUchem_cytotoxicity of all chemicals in a sample***

<img src ="https://user-images.githubusercontent.com/108472923/176872324-d49b101e-4683-45d5-bdd6-f19a783f8a68.png" width=40%>

***Equation 9: Calculation of % of cytotoxicity explained based on TUchem_cytotoxicity***

<img src ="https://user-images.githubusercontent.com/108472923/176872458-e563f4ab-d404-4a74-b089-019ba83f0521.png" width=40%>

***Equation 10a: Calculation of TUchem_baseline for each chemical***

<img src ="https://user-images.githubusercontent.com/108472923/176872643-0dfb2750-fb40-4c1d-a3f7-162e4bfeaf46.png" width=30%>

***Equation 10b: Calculation of TUchem_baseline of all chemicals in a sample***

<img src ="https://user-images.githubusercontent.com/108472923/176872799-fb751943-d8ca-4e49-ace2-e91d0ddf7f41.png" width=30%>

***Equation 11: Calculation of % of cytotoxicity explained based on TUchem_baseline***

<img src ="https://user-images.githubusercontent.com/108472923/176873138-df4c95a6-e811-4b15-81ee-9bf71cc4f334.png" width=40%>

### Equations used in the workflow with no relevance for the iceberg modelling

***Equation 12: Calculation of toxic ratio TR***

<img src = "https://user-images.githubusercontent.com/108472923/176873318-0b80f698-f10e-49eb-bcd3-a0d0b81fc2b7.png" width=15%>

***Equation 13: Calculation of SRcytotoxicity***

<img src = "https://user-images.githubusercontent.com/108472923/176873453-f0c0eb22-7f17-4fcb-8e3c-1212a341c35a.png" width=20%>

***Equation 14: Calculation of SRbaseline***

<img src = "https://user-images.githubusercontent.com/108472923/176873560-8a44f3b5-8fad-4036-9bd7-64b6311c337c.png" width=20%>

### Output of KNIME workflow
The result is an Excel file which is named based on the following nomenclature in a standardized manner: 

YYYY-MM-DD_Project_Assay_iceberg_model.xlsx

The date is the date when the modelling was performed and it is automatically extracted from the workflow. The name of the project can be configured by the user in the component node “Define: 1. Project name, 2. Path to output folder”. The name of the assay is also automatically extracted by the workflow.<br>
In the component node “Define: 1. Project name, 2. Path to output folder” the user also has to provide the path to the folder where the result file is supposed to be saved.

The result file consists of several Excel files containing the following sheets with the data mentioned below:
1.	Read me describing the content of each sheet
2.	results_iceberg_model	
3.	INPUT_master_sheet	
4.	INPUT_Chemicals_analyzed 
5.	INPUT_effect_bioassay	
6.	TUchem_cytotox_chemical	
7.	TUchem_baseline_chemical	
8.	BEQchem_per_chemical	
9.	TR_and_SRs	
10.	data_reference	
11.	data_QSAR	


### References
(1)	Neale, P. A., Brack, W., Ait-Aissa, S., Busch, W., Hollender, J., Krauss, M., Maillot-Marechal, E., Munz, N. A., Schlichting, R., Schulze, T., Vogler, B., and Escher, B. I. (2018) Solid-phase extraction as sample preparation of water samples for cell-based and other in vitro bioassays. Environ. Sci.-Process Impacts 20, 493-504.<br>
(2)	Lee, J., Braun, G., Henneberger, L., Konig, M., Schlichting, R., Scholz, S., and Escher, B. I. (2021) Critical Membrane Concentration and Mass-Balance Model to Identify Baseline Cytotoxicity of Hydrophobic and Ionizable Organic Chemicals in Mammalian Cell Lines. Chem. Res. Toxicol. 34, 2100-2109.<br>
(3)	Neale, P. A., Braun, G., Brack, W., Carmona, E., Gunold, R., Konig, M., Krauss, M., Liebmann, L., Liess, M., Link, M., Schafer, R. B., Schlichting, R., Schreiner, V. C., Schulze, T., Vormeier, P., Weisner, O., and Escher, B. I. (2020) Assessing the Mixture Effects in In Vitro Bioassays of Chemicals Occurring in Small Agricultural Streams during Rain Events. Environ. Sci. Technol. 54, 8280-8290.










