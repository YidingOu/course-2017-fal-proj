# Find the best college location in Boston and other related analysis

*by Lin Li, Yiding Ou, Tianqi Xu*

## 1. Introduction

we are focusing on building a website to help people find the best college location in Boston and provide corresponding analysis to their choices.

It's really complicated to define which area of Boston is the most suitable for universities, we simplify it by three main factors: `Safety`, `Convinience` and `Comfort` and map them to ratings in 1~5. Through our API, users could customize the ratings of these four aspects based on their personal requirements so that they could find the ideal place of living.

We are exploring the rating of living around the schools in Boston by calculating the safety rate, the comfort rate, and the convenience rate. Another objective of us is to use k-mean to find the area that needs a hospital the most. Our safety rate will include data from crime, crash and hospitals. Our comfort rate will include data from entertainment and restaurants. Our convenience rate will include data from crash, hubway, traffic signals and MBTA.


## 2. Datasets

- [Crash](http://datamechanics.io/data/eileenli_xtq_yidingou/crash.json)
- [MBTA](http://datamechanics.io/data/cyung20_kwleung/mbta-t-stops.json)
- [Hubway](http://datamechanics.io/data/eileenli_xtq_yidingou/Hubway_Stations.geojson)
- [Schools](http://datamechanics.io/data/eileenli_xtq_yidingou/Colleges_and_Universities.geojson)
- [Restaurant](http://datamechanics.io/data/eileenli_xtq_yidingou/Restaurant.json)
- [Crime](http://datamechanics.io/data/eileenli_xtq_yidingou/crime.json)
- [Hospitals](http://datamechanics.io/data/eileenli_xtq_yidingou/hospital.json)
- [Entertaiment](http://datamechanics.io/data/eileenli_xtq_yidingou/new.json)
- [Traffic Signals](http://datamechanics.io/data/eileenli_xtq_yidingou/Traffic_Signals.geojson)


## 3. Preprocessing

1. Extracting data from databases:
a). Comfort Section: we extract the coordinates of every entertainment from Entertainment database and coordinates of every restaurant from Restaurant database, and put them into a new dictionary.

b). Safety Section: we extract the coordinates of every crime insident from Crime database, the coordinates of every car crash from Crash database, and the coordinates of hospitals from Hospital database, and put them into a new dictionary.

c). Convenience Section: we extract the coordinates of every car crash from Crash database, the coordinates of every hubway from Hubway database, the coordinates of every traffic signals from Signals database and the coordinates of every MBTA from MBTA database, and put them into a new dictionary.

2.	Data Relation to School:
We first extracts the coordinates of every school from school database, and to calculate the distance from every coordinate of entertainment, restaurant, crime, crash, hospitals, hubway, traffic signals and MBTA. Then we will find the coordinates of those places that are within 2 miles from each school and put them into a new disctionary called "schoolfinal":
```json
		[{  "school": "Boston example School ",
            "properties": [
                {"coordinates": [42.2722, -71.0688]},
                {"safety": [{"hospitals": [[42.3928, -71.3847], [42.0428, -70.4928]...]},
               				{"crimes": [[42.3928, -71.3847], [42.0428, -70.4928]...]},
               				{"crash": [[42.3928, -71.3847], [42.0428, -70.4928]...]}
               				]},
               	{"comfort": [{"entertainment": [[42.3928, -71.3847], [42.0428, -70.4928]...]},
               				{"restaurants": [[42.3928, -71.3847], [42.0428, -70.4928]...]}
               				]},
               	{"traffic": [{"crash": [[42.3928, -71.3847], [42.0428, -70.4928]...]},
               				{"hubway": [[42.3928, -71.3847], [42.0428, -70.4928]...]},
               				{"signals": [[42.3928, -71.3847], [42.0428, -70.4928]...]},
               				{"MBTA": [[42.3928, -71.3847], [42.0428, -70.4928]...]}
               				]}
               ]
        },
        {...}, ...]
```

3.	Statistics Relation to School:
	Here is an example of our score statistics:

```json
	{
	"_id" : ObjectId("5a0b764d1c9de70b8e4f9936"),
	"school" : "Boston University Trustees",
	"properties" : [
		{"hospital" : 14},
		{"crime" : 219},
		{"crash" : 1196},
		{"restaurant" : 165},
		{"entertainment" : 342},
		{"hubway" : 85},
		{"traffic signal" : 226},
		{"MBTA" : 62},
		{"safety" : 9.85},
		{"comfort" : 5.07},
		{"traffic" : -9.71}
	]}
```
	where we calcualted the score of safety, comfort and traffic as follow:
	            {"safety": (1000 + hospital * 100 - crime - crash) / 100},
                {"comfort": (restaurant + entertainment) / 100},
                {"traffic": (1500 + MBTA + hubway - signal - crash * 2) / 100}
   	The higher each score is the better the university it.

4. K-means Analysis for best hospital place:
We use the k means algorithm to find the new optimal hospital place for any 2 schools, at where needs the hospital the most base on the rates of comfort, safety and convenience. 

We first collection the data we have about school and the hospitals around it as follow: ('Massachusetts General Hospital Dietetic Internship', [-71.0701413170573, 42.36259933182846], 11). They are accordingly the name of the school, the coordinates of the school and the number of hospital with the range of 2 miles of that school.

here are the code for the further process:

'''
        two_school_hospital = select(product(school_hospital, school_hospital), lambda t: t[0][0] != t[1][0])
        for i in two_school_hospital:
            two_school_hospital.remove((i[1], i[0]))
        sum_num = project(two_school_hospital, lambda t: ((t[0][0], t[0][1], t[1][0], t[1][1], t[0][2] + t[1][2])))
        target = ()
        sum_num = select(sum_num, lambda t: t[4] < 10 and distance(t[1], t[3]) < 4)
'''
we performed product -> select -> remove reverse duplicate -> project -> select -> minimum algorithm.
if there are no 2 universities that are within the range of 4 miles or their hospital number is bigger than 10,then there is no place that can build hospital and benefits 2 or more schools that need hospital. if there are, then we show them something like "The best place to build a hospital next is between Boston College and Massachusetts School of Professional Psychology at [-71.17503590191852, 42.306371114768865]"


# Run for Project 2
python3 execute.py eileenli_xtq_yidingou

However, entering an infinite loop when running.

 
