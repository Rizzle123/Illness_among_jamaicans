# Illness among Jamaicans


This academic study explored the Jamaican reality of the relationship among sex, age, visits to the doctor, union status, hospital type and smoking on length of illnesses. These variables consistently presented themselves as priority variables associated with longevity of an illness. The study was therefore necessary as it provided additional insight into Jamaica’s unique case. 

The data used was the 2009 Jamaica Survey of Living Conditions and could not be made available on an open platform due to copy right infringement. The data set was the most complex I have worked on to date. Without intimate knowledge, mergeing the relevant data sets is virtually impossible. Consultation was therefore necessary with the Statistics Unit who prepared the data set(s) at the time. For more information on said data set, see https://www.pioj.gov.jm/product/jamaica-survey-of-living-conditions-jslc-2019/ and https://www.uwi.edu/salises-mona/databank/JSLC. I have included a sample of the 2012 version of the questionnaire that is openly available.

The code used for the study can be seen below:

Merging the data

1. DATASET ACTIVATE DataSet1.
MATCH FILES /FILE=*
  /RENAME (agemth fath_fig hhmember id_fath id_moth id_partn marriage moth_fig mths_inh partner 
    record relat serial whynotme = d0 d1 d2 d3 d4 d5 d6 d7 d8 d9 d10 d11 d12 d13) 
  /FILE='DataSet2'
  /RENAME (a1 a2_1 a2_2 a2_3 a2_4 a2_5 a2_6 a3 a5 a6 a8b_hrs a8b_min a8d_hrs a8d_min a8e a8f_hrs 
    a8f_min a8g a8h_hrs a8h_min a8i a8j_hrs a8j_min record serial = d14 d15 d16 d17 d18 d19 d20 d21 d22 
    d23 d24 d25 d26 d27 d28 d29 d30 d31 d32 d33 d34 d35 d36 d37 d38) 
  /BY ind
  /DROP= d0 d1 d2 d3 d4 d5 d6 d7 d8 d9 d10 d11 d12 d13 d14 d15 d16 d17 d18 d19 d20 d21 d22 d23 d24 
    d25 d26 d27 d28 d29 d30 d31 d32 d33 d34 d35 d36 d37 d38.
EXECUTE.

2. MATCH FILES /FILE=*
  /RENAME (a21 a22 a23 a24_1 a24_2 a24_3 a24_4 a24_5 a24_6 a24_7 a24_8 a24_9 a26 a27 a28 record 
    serial = d0 d1 d2 d3 d4 d5 d6 d7 d8 d9 d10 d11 d12 d13 d14 d15 d16) 
  /FILE='DataSet1'
  /BY ind
  /DROP= d0 d1 d2 d3 d4 d5 d6 d7 d8 d9 d10 d11 d12 d13 d14 d15 d16.
EXECUTE.

Recodes and frequencies

DATASET ACTIVATE DataSet1.
RECODE union_st (1 thru 2=1) (3 thru 4=0) (ELSE=SYSMIS) INTO Family_supportunit.
EXECUTE.

RECODE a4 (1 thru 3=1) (4 thru 7=2) (8 thru 14=3) (15 thru 180=4) (ELSE=SYSMIS) INTO a4_recoded.
EXECUTE.

FREQUENCIES VARIABLES=a25 ageyrs sex a7 a8a a8c Family_supportunit a4_recoded
  /STATISTICS=STDDEV VARIANCE RANGE MINIMUM MAXIMUM MEAN MEDIAN MODE SKEWNESS SESKEW
  /ORDER=ANALYSIS.

Testing assumptions

REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS BCOV R ANOVA COLLIN TOL
  /CRITERIA=PIN(.05) POUT(.10)
  /NOORIGIN 
  /DEPENDENT a4_recoded
  /METHOD=ENTER a25 ageyrs sex a7 a8a a8c Family_supportunit
  /RESIDUALS DURBIN.

Multinomial test

NOMREG a4_recoded (BASE=LAST ORDER=ASCENDING) BY a25 sex a8a a8c Family_supportunit WITH ageyrs a7
  /CRITERIA CIN(95) DELTA(0) MXITER(100) MXSTEP(5) CHKSEP(20) LCONVERGE(0) PCONVERGE(0.000001) 
    SINGULAR(0.00000001)
  /MODEL
  /STEPWISE=PIN(.05) POUT(0.1) MINEFFECT(0) RULE(SINGLE) ENTRYMETHOD(LR) REMOVALMETHOD(LR)
  /INTERCEPT=INCLUDE
  /PRINT=CELLPROB FIT PARAMETER SUMMARY LRT CPS STEP MFI IC
  /SAVE ESTPROB PREDCAT PCPROB ACPROB.
