# Query lab exercise
[Based on modelling rep](https://github.optum.com/svishnum/graph-training/tree/docs/modeling/giraffle_optum/db_scripts)

## Lab #1

### Steps
* [iCreate schemna and Load data - loadExcersize4.gsql](https://github.optum.com/svishnum/graph-training/blob/docs/modeling/giraffle_optum/db_scripts/loadExercise4.gsql)

### Excercise #1
* Review the model
* How many patients are there in the graph?
* What is the name of the patient with SSN = 999-37-6355
* What is the date of birth of the patient with DL, S99915145

```
CREATE QUERY Exercise3Answers() FOR GRAPH @graphname@ { 
  
	/* Initialize Set*/ 
  patients = {Patient.*};
	
	/*How many patient vertices are there?*/
	PRINT patients.size();
	
	/*What is the name of the patient with SSN, “999-37-6355”?*/
	tmp = SELECT p FROM patients:p WHERE p.SSN == "999-37-6355";
	PRINT "What is the name of the patient with SSN, 999-37-6355'? " + tmp.FirstName + " " + tmp.LastName;
	
	/*What is the ethnicity of the patient with passport, “X55997743X”?*/
	tmp = SELECT p FROM patients:p WHERE p.Passport == "X55997743X";
	PRINT "What is the ethnicity of the patient with passport, 'X55997743X'?" + tmp.Ethnicity;
	
	/*What is the date of birth of the patient with DL, “S99915145”?*/
	tmp = SELECT p FROM patients:p WHERE p.DL == "S99915145";
	PRINT "What is the date of birth of the patient with DL, 'S99915145'?" + datetime_format(tmp.DateOfBirth, "%m/%d/%Y");
}
```

### Excercise #2
* Find the count of deceased patients and print thier names.
* Find patients with most visits and min visits.
* List 3 providers who had the most visits

## Lab #2
* [Create schema and load data - loadExercise7](https://github.optum.com/svishnum/graph-training/blob/docs/modeling/giraffle_optum/db_scripts/loadExercise7.gsql)

### Excercise #1
* Review the model
* How many patients are there in the graph? 

## Lab #3

* Find patients who are similar - based on similar allergies, age, gender,   

### Excercise #1 - Cosine similarity

* [Create and load data - createAndPopulateSchema.gsql](https://github.optum.com/svishnum/graph-training/blob/docs/algorithms/giraffle_optum/db_scripts/createAndPopulateSchema.gsql)

[Queries](https://github.optum.com/svishnum/graph-training/blob/docs/algorithms/giraffle_optum/db_scripts/algorithmQueries.gsql)

```
CREATE QUERY CosineSimilarityExample(VERTEX<Allergy> allergen) FOR GRAPH @graphname@ RETURNS (MapAccum<STRING, MapAccum<STRING, DOUBLE>>){
	SumAccum<DOUBLE> @@normA, @normB, @aDotB;
	SumAccum<DOUBLE> @age, @gender;
	SumAccum<INT> @@ageA, @@genderA;
	SetAccum<VERTEX<Patient>> @@patientSet;
  	MapAccum<STRING, MapAccum<STRING, DOUBLE>> @@patientSimilarityMap;
	
	allergies = {allergen};

	#Get all patients with the same allergies
	allergyPatients = SELECT p FROM allergies:a-(patientAllergy:e)->Patient:p 
	          ACCUM @@patientSet += p, 
	                p.@age += CalculatePatientAge(p),
	                IF p.Gender == "M" THEN
	                  p.@gender += 25
	                ELSE
	                  p.@gender += -25
	                END;
	
	PRINT allergyPatients;
	

	FOREACH p1 IN @@patientSet DO
	
	  	@@normA = 0.0;
	  	@@ageA = 0;
	  	@@genderA = 0.0;
	  	patientA = SELECT p FROM allergyPatients:p
	                    WHERE p == p1
	                    POST-ACCUM 
	                      @@normA += sqrt(  pow(p.@age, 2) + pow(p.@gender, 2) ),
	                      @@ageA += p.@age,
	                      @@genderA += p.@gender;
	
		patientBList = SELECT p FROM allergyPatients:p
	                    WHERE p != p1
	                    ACCUM 
	                      p.@normB += sqrt(  pow(p.@age, 2) + pow(p.@gender, 2) ),
	                      p.@aDotB += p.@age * @@ageA + p.@gender * @@genderA
	                    POST-ACCUM  
	                    @@patientSimilarityMap += ( p1.FirstName + " " + p1.LastName + ": Age:" + to_string(@@ageA) + " Sex:" + p1.Gender -> (
							p.FirstName + " " + p.LastName + ": Age:" + to_string(p.@age) + " Sex:" + p.Gender -> p.@aDotB / ( @@normA * p.@normB ) 
	                    ));
	

    		patientList = SELECT p FROM allergyPatients:p
	                    ACCUM 
	                      p.@normB = 0.0,
	                      p.@aDotB = 0.0;
	                   
	END;
	
	PRINT @@patientSimilarityMap;
	RETURN @@patientSimilarityMap;
}
```
