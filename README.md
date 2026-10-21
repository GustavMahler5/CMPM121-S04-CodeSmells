# CMPM121-S04-CodeSmells
Group members: Roman Hage, Jack Farris
Code Smells: Duplicated Code, Data Clumps 















Data Clumps:
Introduction:
A data clump is a group of three or four values that keep showing up together, as fields in several classes or as parameters in many function signatures, without ever being given a type of their own. Fowler puts it this way in Chapter 3 (p. 78): “Data items tend to be like children: They enjoy hanging around together.” A group that always travels together “really ought to find a home together.”

How data clumps arise: 
  Gradual growth
  copy-paste reuse
  thinking in primitives 
  boundary data
  prototyping

Why data clumps are a problem:
  Long parameters lists
  Duplicated logics
  Shotgun surgery 
  Inconsistent naming 
  A missing abstraction

Preventing Data clumps:
  Use the rule of three 
  Model domain concepts early 
  Prefer existing types 
  Watch parameter count in code review 
  Make new types cheap 

Code examples (Fowler, pp 141 - 143): 

const station = { name: "ZB1",
                  readings: [
                    {temp: 47, time: "2016-11-10 09:10"},
                    {temp: 53, time: "2016-11-10 09:20"},
                    {temp: 58, time: "2016-11-10 09:30"},
                    {temp: 53, time: "2016-11-10 09:40"},
                    {temp: 51, time: "2016-11-10 09:50"},
                  ]
                };

function readingsOutsideRange(station, min, max) {
  return station.readings
    .filter(r => r.temp < min || r.temp > max);
}

// caller
alerts = readingsOutsideRange(station,
                              operatingPlan.temperatureFloor,
                              operatingPlan.temperatureCeiling);

Why it smells/Whats wrong with it?:
  min and max are clumped. Neither means anything alone, and they're always passed as a pair. If you deleted one, the other would make no sense, so they fail Fowler's deletion test in Ch 3. 
  The pair gets split apart to travel. The caller pulls the two values out of operatingPlan and passes them separately. 
  The names don't match. The plan calls them temperatureFloor and temperatureCeiling, but the function calls them min and max. 
  The lgiv get duped. Any other function that needs a range has to rewrite the < min || > max check 
