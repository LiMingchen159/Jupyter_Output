---
title: Brick Model Tutorial- Meter Data to Building Data
date: '2025-01-19'
---


This tutorial uses data from the academic buildings of the Hong Kong University of Science and Technology (HKUST) as an example. Through this tutorial, you will learn:
1. How to use the Brick Schema to find the required meter inputs.
2. How to preprocess meter data.
3. How to convert meter data into building energy usage data.
4. Basic data visualization techniques.

## Step 1: Environment Setup

```python
from rdflib import Graph
import pandas as pd
import os
from tqdm import tqdm
import warnings
import matplotlib.pyplot as plt
```

## Step 2: Explore HKUST Brick Model
Here is a simple application of the `Query_Example` from this repository. Please download the `HKUST_Meter_Metadata.ttl` file from https://datadryad.org/stash/dataset/doi:10.5061/dryad.k3j9kd5h6.

```python
# Load the Brick Schema model into the graph
graph = Graph()
file_path = 'Data/HKUST_Meter_Metadata.ttl'
graph.parse(file_path, format='turtle')

# Query to retrieve all buildings
query_all_building = f"""
    SELECT ?building
    WHERE {{
        ?building a brick:Building .
    }}
"""

# Execute the query and display the results
buildings = graph.query(query_all_building)
print("All buildings in HKUST:")
for row in buildings:
    # Extract and print the building name from the URI
    print(str(row['building']).split('#')[1])
```

    All buildings in HKUST:
    Academic_Building
    Boat_House
    Cheng_Yu_Tung_Building
    Coastal_Marine_Lab
    IASR_Plant
    IAS_from_IASR_
    Indoor_Swim_Pool
    Jockey_Club_Hall
    LSK_Business_Buildings
    NFF2_Lab_NFF2_Lab_Elect
    Sea_Water_Pump_House
    Seafront_Indoor_Sport_Centre
    Seafront_Sport_Centre_Sport_Field
    Shaw_Auditorium
    Staff_Quarters_01_SSQ_T1_2___PL
    Staff_Quarters_02_SSQ_T3_4
    Staff_Quarters_03_SSQ_T5_7
    Staff_Quarters_04_SSQ_T8_14
    Staff_Quarters_05_SSQ_T15_19
    Staff_Quarters_06_SSQ_P_S
    Staff_Quarters_07_SSQ_APT1_48
    Staff_Quarters_07_SSQ_APT1_49
    Student_Hall_01_UG1
    Student_Hall_02_UG2
    Student_Hall_03_UG3
    Student_Hall_04_UG4
    Student_Hall_05_UG6
    Student_Hall_06_UG7
    Student_Hall_07_UG8_9
    Student_Hall_09_PG2
    Student_Hall_10_GGT
    Tower_A_D_Buildings
    Wind_Wave_Tunnel_Building
    HKUST_Campus_Zone_1_8
    Outdoor_Pool

```python
# Define the SPARQL query to select all zones associated with academic building
query_all_zone = f"""
    SELECT ?zone
    WHERE {{
        ?zone a brick:Zone .  # Find all entities of type 'brick:Zone'
        FILTER EXISTS {{ bldg:Academic_Building brick:hasPart ?zone }}
    }}
"""

# Execute the SPARQL query on the graph
result = graph.query(query_all_zone)

# Iterate through the query results and print the zone names
for i in result:
    print(i['zone'].split("#")[-1])
```

    Annex_Lift_29_30_Elect
    Boiler_Water_Plant_LG1_BSU_Elect
    Building_Service_Unit_LG5_Elect
    Carpark_Elect
    Carpark_MCC_from_SB1_8A_Elect
    Carpark_TX3_MCC_From_SB_1_1A_Elect
    Enterprise_Centre_Elect
    Enterprise_Centre_Enterprise_Centre___L3_Elect
    Enterprise_Centre_Lab_L3_Elect
    G_F_Canteen_Lift_13_15_Elect
    Indoor_Carpark_Elect
    Indoor_Carpark_Switch_Board_2_Elect
    Information_Technology_Services_Centre_Data_Centre_RM2113_Elect
    Information_Technology_Services_Centre_Data_Centre_RM2124_Elect
    LG1_G_F_Canteen_Lift_13_15_Elect
    LG1_Sport_Hall_Elect
    LG5_Sub_Main_Switch_Room_Elect
    LG7_Canteen_Elect
    Library_Canteen_Building_Elect
    Library_Elect
    Phase_1_Academic_Building_Elect
    Phase_2_Academic_Building_Elect
    Plumbing_Drainage_Switch_Board_Elect
    Zone_A1_A2_Lift_3_4_Elect
    Zone_A1_Lift_4_Elect
    Zone_A2_Lift_3_Elect
    Zone_A3_A4_Lift_1_2_Elect
    Zone_A5_A6_Lift_17_18_Elect
    Zone_A5_Lift_17_18_Elect
    Zone_A7_Lift_19_Elect
    Zone_D_Lift_25_26_Elect
    Zone_E_Lift_27_28_Elect
    Zone_H1_Lift_23_Elect
    Zone_H1_Lift_24_Elect
    Zone_H2_Lift_24_Elect
    Zone_J1_Lift_20_Elect
    Zone_J2_Lift_21_Elect
    Zone_J_Lift_20_21_Elect
    Zone_L1_Lift_22_Elect
    Zone_L2_Lift_22_Elect
    Animal_Care_Facility_7_F_Elect

## Step 3: Extract All Electric Meter Data In Academic Building

```python
query_ac_meter = f"""
    SELECT ?meter
    WHERE {{
        bldg:Academic_Building brick:hasPart ?zone .
        ?zone a brick:Zone .
        ?zone brick:hasPart ?meter .
        ?meter a brick:Electrical_Meter .
    }}
"""
result = graph.query(query_ac_meter)
Meter_list = []
for i in result:
    print(i['meter'].split("#")[-1])
    Meter_list.append(i['meter'].split("#")[-1])
print("Total number of meters in the academic building:", len(Meter_list))
```

    Meter_M0289
    Meter_M0290
    Meter_M0291
    Meter_M0293
    Meter_M0295
    Meter_M0296
    Meter_M0529
    Meter_M0530
    Meter_D0695
    Meter_D0696
    Meter_M0292
    Meter_M0294
    Meter_M0126
    Meter_M0127
    Meter_M0128
    Meter_M0129
    Meter_M0130
    Meter_M0121
    Meter_M0122
    Meter_M0123
    Meter_M0124
    Meter_M0125
    Meter_M0112
    Meter_D0705
    Meter_M0113
    Meter_M0114
    Meter_M0115
    Meter_M0116
    Meter_M0117
    Meter_M0118
    Meter_M0119
    Meter_M0120
    Meter_D0808
    Meter_D0809
    Meter_D0810
    Meter_D0811
    Meter_D0812
    Meter_D0813
    Meter_D0814
    Meter_D0815
    Meter_D0816
    Meter_D0818
    Meter_D0819
    Meter_D0820
    Meter_D0821
    Meter_D0823
    Meter_D0825
    Meter_D0826
    Meter_D0817
    Meter_D0822
    Meter_D0824
    Meter_M0035
    Meter_M0036
    Meter_M0037
    Meter_M0038
    Meter_M0039
    Meter_M0040
    Meter_M0041
    Meter_M0042
    Meter_D0408
    Meter_M0043
    Meter_M0044
    Meter_M0045
    Meter_M0046
    Meter_M0047
    Meter_M0048
    Meter_M0049
    Meter_M0050
    Meter_M0051
    Meter_M0052
    Meter_M0503
    Meter_M0528
    Meter_D0601
    Meter_D0602
    Meter_D0603
    Meter_D0604
    Meter_D0876
    Meter_D0877
    Meter_D0878
    Meter_M0564
    Meter_M0565
    Meter_M0566
    Meter_M0567
    Meter_M0568
    Meter_M0569
    Meter_M0098
    Meter_M0099
    Meter_M0100
    Meter_M0101
    Meter_M0103
    Meter_M0104
    Meter_M0105
    Meter_M0106
    Meter_M0107
    Meter_M0108
    Meter_M0109
    Meter_M0110
    Meter_M0111
    Meter_M0102
    Meter_H0080
    Meter_H0081
    Meter_H0082
    Meter_H0083
    Meter_H0084
    Meter_H0085
    Meter_H0066
    Meter_H0067
    Meter_H0068
    Meter_H0069
    Meter_H0070
    Meter_H0071
    Meter_H0072
    Meter_H0073
    Meter_H0074
    Meter_H0075
    Meter_H0076
    Meter_H0077
    Meter_H0078
    Meter_H0079
    Meter_M0527
    Meter_M0562
    Meter_M0563
    Meter_M0531
    Meter_M0532
    Meter_D0425
    Meter_D0426
    Meter_D0427
    Meter_D0428
    Meter_D0429
    Meter_D0403
    Meter_D0404
    Meter_D0409
    Meter_M0076
    Meter_M0077
    Meter_M0078
    Meter_M0079
    Meter_M0080
    Meter_M0081
    Meter_M0083
    Meter_M0084
    Meter_M0085
    Meter_M0086
    Meter_M0088
    Meter_M0089
    Meter_M0090
    Meter_M0091
    Meter_M0092
    Meter_M0094
    Meter_M0095
    Meter_M0096
    Meter_M0097
    Meter_M0549
    Meter_M0550
    Meter_M0551
    Meter_M0082
    Meter_M0087
    Meter_M0093
    Meter_M0602
    Meter_D0605
    Meter_D0607
    Meter_D0608
    Meter_D0609
    Meter_D0610
    Meter_D0611
    Meter_D0612
    Meter_D0615
    Meter_D0616
    Meter_D0617
    Meter_D0618
    Meter_D0867
    Meter_D0868
    Meter_D0869
    Meter_D0870
    Meter_D0871
    Meter_D0872
    Meter_D0884
    Meter_D0885
    Meter_D0886
    Meter_D0887
    Meter_H0165
    Meter_M0560
    Meter_M0561
    Meter_D0627
    Meter_D0628
    Meter_D0629
    Meter_D0630
    Meter_D0631
    Meter_D0632
    Meter_D0641
    Meter_D0642
    Meter_D0643
    Meter_D0644
    Meter_D0638
    Meter_M0131
    Meter_M0132
    Meter_M0133
    Meter_M0134
    Meter_M0135
    Meter_M0136
    Meter_M0137
    Meter_M0138
    Meter_M0139
    Meter_M0140
    Meter_M0141
    Meter_M0142
    Meter_M0143
    Meter_M0144
    Meter_M0145
    Meter_M0146
    Meter_M0147
    Meter_M0148
    Meter_M0149
    Meter_D0614
    Meter_D0874
    Meter_M0180
    Meter_M0181
    Meter_M0182
    Meter_M0183
    Meter_M0184
    Meter_M0185
    Meter_M0186
    Meter_M0187
    Meter_M0188
    Meter_M0189
    Meter_M0190
    Meter_M0191
    Meter_M0192
    Meter_M0193
    Meter_M0194
    Meter_M0195
    Meter_M0196
    Meter_M0197
    Meter_M0198
    Meter_M0199
    Meter_M0200
    Meter_M0201
    Meter_M0202
    Meter_M0203
    Meter_M0204
    Meter_M0205
    Meter_M0206
    Meter_M0207
    Meter_M0208
    Meter_M0209
    Meter_M0210
    Meter_M0211
    Meter_M0212
    Meter_M0518
    Meter_M0519
    Meter_M0520
    Meter_M0521
    Meter_D0873
    Meter_D0411
    Meter_D0620
    Meter_D0621
    Meter_D0613
    Meter_D0619
    Meter_D0875
    Meter_M0213
    Meter_M0214
    Meter_M0215
    Meter_M0216
    Meter_M0217
    Meter_M0218
    Meter_M0219
    Meter_M0220
    Meter_M0221
    Meter_M0222
    Meter_M0223
    Meter_M0224
    Meter_M0225
    Meter_M0226
    Meter_M0522
    Meter_M0523
    Meter_D0606
    Meter_M0227
    Meter_M0228
    Meter_M0229
    Meter_M0230
    Meter_M0231
    Meter_M0232
    Meter_M0233
    Meter_M0234
    Meter_M0235
    Meter_M0236
    Meter_M0237
    Meter_M0241
    Meter_M0242
    Meter_M0243
    Meter_M0244
    Meter_M0245
    Meter_M0246
    Meter_M0247
    Meter_M0248
    Meter_M0249
    Meter_M0250
    Meter_M0251
    Meter_M0252
    Meter_M0253
    Meter_M0254
    Meter_M0255
    Meter_M0256
    Meter_M0257
    Meter_M0524
    Meter_M0525
    Meter_M0526
    Meter_M0240
    Meter_M0238
    Meter_M0239
    Meter_D0410
    Meter_D0413
    Meter_D0623
    Meter_D0624
    Meter_D0625
    Meter_D0626
    Meter_M0258
    Meter_M0259
    Meter_M0260
    Meter_M0262
    Meter_M0263
    Meter_M0264
    Meter_M0261
    Meter_D0406
    Meter_D0637
    Meter_D0649
    Meter_M0265
    Meter_M0266
    Meter_M0268
    Meter_M0269
    Meter_M0273
    Meter_M0274
    Meter_M0436
    Meter_M0437
    Meter_M0438
    Meter_M0439
    Meter_M0440
    Meter_M0511
    Meter_M0545
    Meter_M0546
    Meter_M0547
    Meter_M0548
    Meter_D0668
    Meter_D0669
    Meter_M0267
    Meter_M0270
    Meter_M0271
    Meter_M0272
    Meter_D0407
    Meter_D0640
    Meter_D0650
    Meter_M0276
    Meter_M0277
    Meter_M0278
    Meter_M0279
    Meter_M0280
    Meter_M0284
    Meter_M0285
    Meter_M0286
    Meter_M0287
    Meter_M0288
    Meter_M0441
    Meter_M0442
    Meter_M0443
    Meter_M0444
    Meter_M0445
    Meter_M0446
    Meter_D0693
    Meter_D0694
    Meter_M0275
    Meter_M0281
    Meter_M0282
    Meter_M0283
    Meter_D0635
    Meter_D0647
    Meter_D0685
    Meter_D0687
    Meter_D0688
    Meter_M0297
    Meter_M0299
    Meter_M0301
    Meter_M0302
    Meter_M0303
    Meter_M0304
    Meter_M0305
    Meter_M0447
    Meter_M0448
    Meter_M0449
    Meter_M0450
    Meter_M0451
    Meter_M0452
    Meter_M0298
    Meter_M0300
    Meter_D0684
    Meter_D0686
    Meter_M0505
    Meter_D0636
    Meter_D0648
    Meter_D0680
    Meter_D0682
    Meter_D0683
    Meter_M0306
    Meter_M0308
    Meter_M0309
    Meter_M0311
    Meter_M0312
    Meter_M0313
    Meter_M0314
    Meter_M0453
    Meter_M0454
    Meter_M0455
    Meter_M0456
    Meter_M0457
    Meter_M0458
    Meter_D0681
    Meter_M0307
    Meter_M0310
    Meter_D0679
    Meter_D0671
    Meter_D0672
    Meter_D0674
    Meter_M0150
    Meter_M0151
    Meter_M0153
    Meter_M0154
    Meter_M0156
    Meter_M0157
    Meter_M0159
    Meter_M0160
    Meter_M0161
    Meter_M0420
    Meter_M0421
    Meter_M0422
    Meter_M0423
    Meter_M0424
    Meter_M0425
    Meter_M0426
    Meter_M0427
    Meter_M0428
    Meter_M0504
    Meter_M0152
    Meter_M0155
    Meter_D0670
    Meter_D0673
    Meter_D0676
    Meter_D0677
    Meter_D0678
    Meter_M0162
    Meter_M0164
    Meter_M0166
    Meter_M0168
    Meter_M0170
    Meter_M0171
    Meter_M0172
    Meter_M0173
    Meter_M0174
    Meter_M0175
    Meter_M0176
    Meter_M0177
    Meter_M0178
    Meter_M0179
    Meter_M0429
    Meter_M0430
    Meter_M0431
    Meter_M0432
    Meter_M0433
    Meter_M0434
    Meter_M0435
    Meter_M0163
    Meter_M0165
    Meter_M0167
    Meter_M0169
    Meter_D0675
    Meter_D0639
    Meter_D0651
    Meter_D0689
    Meter_D0690
    Meter_D0691
    Meter_D0692
    Meter_D0633
    Meter_D0645
    Meter_D0661
    Meter_D0663
    Meter_D0665
    Meter_D0667
    Meter_M0315
    Meter_M0316
    Meter_M0317
    Meter_M0319
    Meter_M0320
    Meter_M0321
    Meter_M0323
    Meter_M0324
    Meter_M0325
    Meter_M0326
    Meter_M0327
    Meter_M0328
    Meter_M0329
    Meter_M0330
    Meter_M0331
    Meter_M0332
    Meter_M0333
    Meter_M0459
    Meter_M0460
    Meter_M0461
    Meter_M0462
    Meter_M0463
    Meter_M0464
    Meter_M0465
    Meter_M0466
    Meter_M0467
    Meter_M0468
    Meter_M0469
    Meter_M0470
    Meter_M0512
    Meter_M0322
    Meter_D0660
    Meter_D0662
    Meter_D0664
    Meter_D0666
    Meter_D0634
    Meter_D0646
    Meter_M0334
    Meter_M0335
    Meter_M0336
    Meter_M0338
    Meter_M0339
    Meter_M0340
    Meter_M0341
    Meter_M0342
    Meter_M0471
    Meter_M0472
    Meter_M0473
    Meter_M0474
    Meter_M0475
    Meter_M0506
    Meter_M0507
    Meter_M0513
    Meter_M0514
    Meter_M0515
    Meter_M0516
    Meter_M0517
    Meter_H0001
    Meter_H0002
    Meter_H0003
    Meter_H0004
    Meter_H0005
    Meter_H0006
    Meter_H0007
    Meter_H0008
    Meter_H0009
    Meter_H0010
    Meter_H0011
    Meter_H0012
    Meter_H0013
    Meter_H0015
    Meter_H0016
    Meter_H0018
    Meter_H0019
    Meter_H0020
    Meter_H0021
    Meter_H0022
    Meter_H0024
    Meter_H0025
    Meter_H0026
    Meter_H0031
    Meter_H0033
    Meter_H0035
    Meter_H0036
    Meter_H0037
    Meter_H0038
    Meter_H0039
    Meter_H0040
    Meter_H0041
    Meter_H0042
    Meter_H0043
    Meter_H0044
    Meter_H0045
    Meter_H0046
    Meter_H0048
    Meter_H0050
    Meter_H0051
    Meter_H0056
    Meter_H0057
    Meter_H0058
    Meter_H0059
    Meter_H0060
    Meter_H0062
    Meter_H0063
    Meter_H0064
    Meter_H0065
    Meter_H0014
    Meter_H0017
    Meter_H0023
    Meter_H0027
    Meter_H0028
    Meter_H0029
    Meter_H0030
    Meter_H0032
    Meter_H0034
    Meter_H0047
    Meter_H0049
    Meter_H0052
    Meter_H0053
    Meter_H0054
    Meter_H0055
    Meter_H0061
    Total number of meters in the academic building: 606

```python
# Prepare a list to store the results
results_list = []

# Double check, query whether the meter belongs to the academic building
for meter in Meter_list:
    query_meter_belong = f"""
        SELECT ?building ?zone
        WHERE {{
            bldg:{meter} a brick:Electrical_Meter .
            ?zone brick:hasPart bldg:{meter} .
            ?zone a brick:Zone .
            ?building brick:hasPart ?zone .
            ?building a brick:Building .
        }}
    """
    result = graph.query(query_meter_belong)
    for i in result:
        # Append the results to the list
        results_list.append({
            'Building': i['building'].split("#")[-1], 
            'Zone': i['zone'].split("#")[-1], 
            'Meter': meter,
            'Meter_Datafile_Name': "GUI_NO.{}.xlsx".format(meter.split("_")[-1])
        })

# Convert the results list to a DataFrame
df = pd.DataFrame(results_list)

# Display the DataFrame
df
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Building</th>
      <th>Zone</th>
      <th>Meter</th>
      <th>Meter_Datafile_Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0289</td>
      <td>GUI_NO.M0289.xlsx</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0290</td>
      <td>GUI_NO.M0290.xlsx</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0291</td>
      <td>GUI_NO.M0291.xlsx</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0293</td>
      <td>GUI_NO.M0293.xlsx</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0295</td>
      <td>GUI_NO.M0295.xlsx</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>666</th>
      <td>NFF2_Lab_NFF2_Lab_Elect</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0054</td>
      <td>GUI_NO.H0054.xlsx</td>
    </tr>
    <tr>
      <th>667</th>
      <td>Academic_Building</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0055</td>
      <td>GUI_NO.H0055.xlsx</td>
    </tr>
    <tr>
      <th>668</th>
      <td>NFF2_Lab_NFF2_Lab_Elect</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0055</td>
      <td>GUI_NO.H0055.xlsx</td>
    </tr>
    <tr>
      <th>669</th>
      <td>Academic_Building</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0061</td>
      <td>GUI_NO.H0061.xlsx</td>
    </tr>
    <tr>
      <th>670</th>
      <td>NFF2_Lab_NFF2_Lab_Elect</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0061</td>
      <td>GUI_NO.H0061.xlsx</td>
    </tr>
  </tbody>
</table>
<p>671 rows × 4 columns</p>
</div>

From the results, it can be seen that all meters belong to the Academic Building. Some meters, however, belong to both the Animal Care Facility 7 F Eelectrical space and the NFF2 Lab, specifically the NFF2_Lab_Elect space.

```python
# Remove duplicate meters.
Pure_Academic_Building_df = df[df['Building'] == 'Academic_Building']
Pure_Academic_Building_df
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Building</th>
      <th>Zone</th>
      <th>Meter</th>
      <th>Meter_Datafile_Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0289</td>
      <td>GUI_NO.M0289.xlsx</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0290</td>
      <td>GUI_NO.M0290.xlsx</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0291</td>
      <td>GUI_NO.M0291.xlsx</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0293</td>
      <td>GUI_NO.M0293.xlsx</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Academic_Building</td>
      <td>Annex_Lift_29_30_Elect</td>
      <td>Meter_M0295</td>
      <td>GUI_NO.M0295.xlsx</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>661</th>
      <td>Academic_Building</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0052</td>
      <td>GUI_NO.H0052.xlsx</td>
    </tr>
    <tr>
      <th>663</th>
      <td>Academic_Building</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0053</td>
      <td>GUI_NO.H0053.xlsx</td>
    </tr>
    <tr>
      <th>665</th>
      <td>Academic_Building</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0054</td>
      <td>GUI_NO.H0054.xlsx</td>
    </tr>
    <tr>
      <th>667</th>
      <td>Academic_Building</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0055</td>
      <td>GUI_NO.H0055.xlsx</td>
    </tr>
    <tr>
      <th>669</th>
      <td>Academic_Building</td>
      <td>Animal_Care_Facility_7_F_Elect</td>
      <td>Meter_H0061</td>
      <td>GUI_NO.H0061.xlsx</td>
    </tr>
  </tbody>
</table>
<p>606 rows × 4 columns</p>
</div>

## Step 4: Data Preprocessing

The raw time series data used in this step can be downloaded from https://datadryad.org/stash/dataset/doi:10.5061/dryad.k3j9kd5h6.

```python
# Create a directory for preprocessed data if it doesn't exist
output_dir = "preprocessed_data"
os.makedirs(output_dir, exist_ok=True)

# Function to process each file
def process_and_save_file(file_name):
    try:
        # Load the Excel file
        data = pd.read_excel(file_name)
        
        # Ensure the DataFrame has a datetime index
        if 'time' not in data.columns:
            raise ValueError(f"'time' column is missing in {file_name}")
        data['time'] = pd.to_datetime(data['time'])
        data.set_index('time', inplace=True)
        
        # Resample the data to hourly intervals
        data_hourly = data.resample('h').mean()
        
        # Apply linear interpolation only to the 'number' column
        if 'number' in data_hourly.columns:
            data_hourly['number'] = data_hourly['number'].interpolate(method='linear')
        
        # Save the preprocessed data
        output_file_name = os.path.basename(file_name).replace('.xlsx', '.csv')
        output_file = os.path.join(output_dir, output_file_name)

        data_hourly.to_csv(output_file)
    
    except Exception as e:
        print(f"Error processing {file_name}: {e}")

# Iterate over the DataFrame with a progress bar
for _, row in tqdm(Pure_Academic_Building_df.iterrows(), total=len(Pure_Academic_Building_df), desc="Processing files"):
    file_name = "Data/Time-series data/" + row['Meter_Datafile_Name']
    process_and_save_file(file_name)
```

    Processing files:   7%|▋         | 42/606 [00:22<03:24,  2.75it/s]

    Error processing Data/Time-series data/GUI_NO.D0819.xlsx: [Errno 2] No such file or directory: 'Data/Time-series data/GUI_NO.D0819.xlsx'

    Processing files:  27%|██▋       | 166/606 [02:02<03:23,  2.16it/s]

    Error processing Data/Time-series data/GUI_NO.D0616.xlsx: [Errno 2] No such file or directory: 'Data/Time-series data/GUI_NO.D0616.xlsx'
    Error processing Data/Time-series data/GUI_NO.D0617.xlsx: [Errno 2] No such file or directory: 'Data/Time-series data/GUI_NO.D0617.xlsx'
    Error processing Data/Time-series data/GUI_NO.D0618.xlsx: [Errno 2] No such file or directory: 'Data/Time-series data/GUI_NO.D0618.xlsx'

    Processing files:  89%|████████▉ | 541/606 [07:40<00:40,  1.60it/s]

    Error processing Data/Time-series data/GUI_NO.H0001.xlsx: [Errno 2] No such file or directory: 'Data/Time-series data/GUI_NO.H0001.xlsx'
    Error processing Data/Time-series data/GUI_NO.H0002.xlsx: [Errno 2] No such file or directory: 'Data/Time-series data/GUI_NO.H0002.xlsx'
    Error processing Data/Time-series data/GUI_NO.H0003.xlsx: [Errno 2] No such file or directory: 'Data/Time-series data/GUI_NO.H0003.xlsx'

    Processing files: 100%|██████████| 606/606 [08:32<00:00,  1.18it/s]

From the current results, it can be observed that a small portion of the meters lack data. The reason for this is discussed in the Technical Validation section of the paper. For details, please refer to: https://www.nature.com/articles/s41597-024-04106-1.

## Step 5: Convert Meter Data to Building Energy Consumption Data

```python
# Ignore warnings
warnings.filterwarnings("ignore")

# Directory containing the preprocessed data files
input_dir = "preprocessed_data"
output_file = "Building_Consumption_Data.xlsx"

# Initialize an empty DataFrame to hold all data
combined_data = pd.DataFrame()

# Iterate through all CSV files in the directory
for file in tqdm(os.listdir(input_dir), desc="Combining files"):
    if file.endswith('.csv'):
        file_path = os.path.join(input_dir, file)
        
        # Load the file
        data = pd.read_csv(file_path)
        
        # Ensure it has the necessary 'time' and 'number' columns
        if 'time' in data.columns and 'number' in data.columns:
            data['time'] = pd.to_datetime(data['time'])  # Ensure time is datetime
            data.set_index('time', inplace=True)
            
            # Add the 'number' column to the combined DataFrame
            combined_data[file] = data['number']

# Sum all columns to create the 'Building_Consumption_data' column
combined_data['Building_Consumption_data'] = combined_data.sum(axis=1)

# Reset the index to include 'time' as a column
combined_data.reset_index(inplace=True)

# Save the result to an Excel file
combined_data[['time', 'Building_Consumption_data']].to_excel(output_file, index=False)

print(f"Building consumption data saved to {output_file}")
```

    Combining files: 100%|██████████| 599/599 [00:07<00:00, 80.12it/s] 

    Building consumption data saved to Building_Consumption_Data.xlsx

## Step 7: Data Visualization

```python
# Load the data from the Excel file
file_path = "Building_Consumption_Data.xlsx"
data = pd.read_excel(file_path)

# Ensure the 'time' column is a datetime type
data['time'] = pd.to_datetime(data['time'])

# Plot 1: Line chart of Building_Consumption_data
plt.figure(figsize=(10, 6))
plt.plot(data['time'], data['Building_Consumption_data'], label='Building Consumption', linewidth=1)
plt.xlabel('Time')
plt.ylabel('Consumption')
plt.title('Building Consumption Over Time')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

    
![png](output_17_0.png)
    

The results indicate that the data exhibits some fluctuations. These could be attributed to two possible reasons: the addition of new meter data or the restart of certain meters.

```python
# Calculate the first-order differences
data['Consumption_Difference'] = data['Building_Consumption_data'].diff()

# Remove outliers using the IQR method
Q1 = data['Consumption_Difference'].quantile(0.25)
Q3 = data['Consumption_Difference'].quantile(0.75)
IQR = Q3 - Q1

# Define the lower and upper bounds for outlier detection
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter the data to remove outliers
filtered_data = data[(data['Consumption_Difference'] >= lower_bound) & (data['Consumption_Difference'] <= upper_bound)]

# Plot the filtered first-order differences
plt.figure(figsize=(10, 6))
plt.plot(filtered_data['time'], filtered_data['Consumption_Difference'], label='Filtered Consumption Difference', linewidth=1, color='orange')
plt.xlabel('Time')
plt.ylabel('Difference')
plt.title('Building Consumption First-Order Differences (Filtered)')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

    
![png](output_19_0.png)
    

To calculate power, we performed differencing on the consumption data and removed outliers using the IQR method. From the results in the second chart, it is evident that the data quality has significantly improved.

## Conclusion

This tutorial demonstrates how to convert meter data into building energy consumption data using the Brick Model. The tutorial also covers data preprocessing techniques, including handling missing values using linear interpolation and removing outliers with the IQR method. Visualization techniques are also discussed. The results show that the data quality has improved significantly after preprocessing. This tutorial can be adapted to other buildings within this dataset.
