# CMPM121-S04-CodeSmells

Group members: Roman Hage, Jack Farris, Jason Holtman

# Code Smells: Duplicated Code, Data Clumps, Primitive Obsession

## Primitive Obsession.  

Primitive Obsession is a code smell where data that has a higher level of complexity is relegated to a primitive typing instead of its own dedicated class. A few examples of this are phone numbers being represented as a string or mass being represented as an integer. Phone numbers are not just strings of numbers; they have hyphens, area codes, and a specific length that they must be to qualify. When it comes to mass, it is very easy to misinterpret the units which can lead to miscalculations. In both of these cases, a specific class can guarantee type safety and clarity for the developer. This concept makes it easier to safely pass parameters and have a better understanding of the code itself. Using Fowlers methods, we can simply use "Replace Primitive with Object".

For example:
```java 
String phoneNumber;
``` 
It is clear that there is nothing stopping us from passing any kind of string to phoneNumber

We can instead create a class:  
```java
// Java
public class PhoneNumber {

    private String phoneNumber;

    public PhoneNumber(String number) {
        if (!isValid()) {
            throw new IllegalArgumentException("Phone number must be in XXX-XXX-XXXX format.");
        }
        phoneNumber = number;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }

    private Boolean isValid() {
        return (number != null && number.length() == 12 && number.charAt(3) == '-' && number.charAt(7) == '-');
    }
}
```
In doing so, we can be sure that any argument we pass for a parameter of type PhoneNumber is a valid phone number

## Data Clumps:
 
A data clump is a group of three or four values that keep showing up together, as fields in several classes or as parameters in many function signatures, without ever being given a type of their own, like x, y, z coordinates or min, max ranges. Clumps usually creep in through gradual growth, copy-paste reuse, thinking in primitives, boundary data that stays split after it’s read in, and prototyping shortcuts that outlive the prototype. The result is long, error-prone parameter lists, duplicated logic, shotgun surgery whenever a member is added, inconsistent naming, and ultimately a missing abstraction the domain already has.

# Preventing them: 

Preventing data clumps is mostly about catching them before they spread. A good rule of thumb is that once the same group of values appears three times, it should become its own type. Concepts from the problem domain are worth modeling early, and existing types from the language or framework should be used instead of passing their parts separately. Long parameter lists are an easy warning sign to watch for in code review. Since most modern languages make small types quick to define, there’s rarely a reason not to create one.


# Code examples (Fowler, pp 141 - 143): 
```typescript
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
```

# Why it smells/Whats wrong with it?:

min and max are clumped. Neither means anything alone, and they're always passed as a pair. If you deleted one, the other would make no sense, so they fail Fowler's deletion test found in Ch 3.
   
The pair gets split apart to travel. The caller pulls the two values out of operatingPlan and passes them separately.

The names don't match. The plan calls them temperatureFloor and temperatureCeiling, but the function calls them min and max.
   
The logic gets duped. Any other function that needs a range has to rewrite the < min || > max check.  


# Refactoring/What to improve 
I’d start by creating a NumberRange class to hold the clump. I’d make it a class rather than a plain object so I have a place to put behavior later, and I’d leave out setters so it stays immutable, which is what Fowler recommends. Then I’d swap the new object into the function and remove the old min and max parameters one at a time, testing after each change. Last, I’d move the range check itself into the class as a contains method. 

# Improved Code:
```typescript
class NumberRange {
  constructor(min, max) {
    this._data = {min: min, max: max};
  }
  get min() { return this._data.min; }
  get max() { return this._data.max; }
  contains(arg) { return (arg >= this.min && arg <= this.max); }
}

function readingsOutsideRange(station, range) {
  return station.readings
    .filter(r => !range.contains(r.temp));
}

// caller
const range = new NumberRange(operatingPlan.temperatureFloor, operatingPlan.temperatureCeiling);
alerts = readingsOutsideRange(station, range);
```

After the refactor, the function takes one range instead of two loose numbers, so I can’t swap min and max by accident. The naming mismatch between floor/ceiling and min/max is gone too. Now any time I need to check a range, I can reuse contains instead of writing the comparison again. I also like that this opens the door to more cleanup. Like Fowler suggests, I could keep an eye out for other min/max pairs in the code and replace them with NumberRange, starting with operatingPlan itself, which could store a single temperatureRange instead of separate floor and ceiling values.


# C# comparison:

```C#
// Smell
IEnumerable<Reading> ReadingsOutsideRange(Station station, double min, double max) =>
    station.Readings.Where(r => r.Temp < min || r.Temp > max);

// Refactored
public readonly record struct NumberRange(double Min, double Max)
{
    public bool Contains(double arg) => arg >= Min && arg <= Max;
}

IEnumerable<Reading> ReadingsOutsideRange(Station station, NumberRange range) =>
    station.Readings.Where(r => !range.Contains(r.Temp)); 
```

# Similarities:

The clump and the fix are the same as in the other two languages: group the values, then move the logic into the new type.

The main benefit is still the behavior that moves into NumberRange, not just the shorter parameter list.


# Differences: 
  
  Static typing makes repeated double min, double max signatures easier to spot. The compiler still won’t catch me swapping them, though, because     both are double.

  A record struct creates the whole value type in one line and includes the value-based equality Fowler recommends adding.

  C# offers tuples like (double min, double max) as a shortcut. Like the other languages’ shortcuts, it doesn’t give the group a real type to hold    behavior.


# Python comparison: 

# Smell
```Python
def readings_outside_range(station, min_temp, max_temp):
    return [r for r in station["readings"]
            if r["temp"] < min_temp or r["temp"] > max_temp]
```
# Refactored

```Python
from dataclasses import dataclass

@dataclass(frozen=True)
class NumberRange:
    min: float
    max: float

    def contains(self, arg: float) -> bool:
        return self.min <= arg <= self.max

def readings_outside_range(station, rng: NumberRange):
    return [r for r in station["readings"] if not rng.contains(r["temp"])]
```

# Similarities:
  
  Like JavaScript, Python is dynamically typed, so the clump is just as easy to overlook.
  
  The refactoring follows the same steps: create the type, pass it in, move contains into it.


# Differences: 
  `@dataclass(frozen=True)` creates an immutable value object in a few lines, with no hand-written constructor or getters.
 
  Python’s shortcuts are keyword arguments (`low=0`, `high=10`) or a tuple. Like the JavaScript shortcut, they make calls clearer but don’t give the group a home for behavior.
  



  
