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
  The logic gets duped. Any other function that needs a range has to rewrite the < min || > max check.  

Refactoring/What to improve 
Using Intro Parameters Object (p. 140) I’d start by creating a NumberRange class to hold the clump. I’d make it a class rather than a plain object so I have a place to put behavior later, and I’d leave out setters so it stays immutable, which is what Fowler recommends. Then I’d swap the new object into the function and remove the old min and max parameters one at a time, testing after each change. Last, I’d move the range check itself into the class as a contains method. Fowler says this last part is where the real value is: “replacing a clump of parameters with a real object is just the setup for the really good stuff” (p. 143)

Improved Code: 
class NumberRange {
  constructor(min, max) {
    this._data = {min: min, max: max};
  }
  get min() {return this._data.min;}
  get max() {return this._data.max;}
  contains(arg) {return (arg >= this.min && arg <= this.max);}
}

function readingsOutsideRange(station, range) {
  return station.readings
    .filter(r => !range.contains(r.temp));
}

// caller
const range = new NumberRange(operatingPlan.temperatureFloor, operatingPlan.temperatureCeiling);
alerts = readingsOutsideRange(station, range);

After the refactor, the function takes one range instead of two loose numbers, so I can’t swap min and max by accident. The naming mismatch between floor/ceiling and min/max is gone too. Now any time I need to check a range, I can reuse contains instead of writing the comparison again. I also like that this opens the door to more cleanup. Like Fowler suggests, I could keep an eye out for other min/max pairs in the code and replace them with NumberRange, starting with operatingPlan itself, which could store a single temperatureRange instead of separate floor and ceiling values.

