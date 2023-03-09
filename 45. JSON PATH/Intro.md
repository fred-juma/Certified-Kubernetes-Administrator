#### JSON PATH:

This is a query language for parsing data  represented in json or yaml format

##### 1 / 13 Dictionary
Develop a JSON path query to extract the expected output from the Source Data.

**Source Data**

```json

{
  "property1": "value1",
  "property2": "value2"
}
```

**Expected value**

```json
[
  "value1"
]
```

**JSON Path Query:**  

```bash
$.property1
```

##### 2 / 13 Dictionary
Develop a JSON path query to extract the bus details from the Source Data.

**Source Data:**

```json
{
  "car": {
    "color": "blue",
    "price": "$20,000"
  },
  "bus": {
    "color": "white",
    "price": "$120,000"
  }
}
```




**Expected Output:**

```json
[
  {
    "color": "white",
    "price": "$120,000"
  }
]

```

**JSON Path Query:**  
```bash
$.bus
```

##### 3 / 13 Dictionary in Dictionary
Develop a JSON path query to extract the price of the bus.

**Expected Output:**

```json
[
  "$120,000"
]
```

**JSON Path Query:** 
```bash
$.bus.price
```

##### 4 / 13 Dictionary in Dictionary
Develop a JSON path query to extract the price of the car.

**Source Data:**

```json
{
  "vehicles": {
    "car": {
      "color": "blue",
      "price": "$20,000"
    },
    "bus": {
      "color": "white",
      "price": "$120,000"
    }
  }
}
```

**Expected Output:**

```json
[
  "$20,000"
]
```

**JSON Path Query:**  
```bash
$.vehicles.car.price
```

##### 5 / 13 List in Dictionary
Develop a JSON path query to extract data about the wheels of the car.

**Source Data:**

```json
{
  "car": {
    "color": "blue",
    "price": "$20,000",
    "wheels": [
      {
        "model": "KDJ39848T",
        "location": "front-right"
      },
      {
        "model": "MDJ39485DK",
        "location": "front-left"
      },
      {
        "model": "KCMDD3435K",
        "location": "rear-right"
      },
      {
        "model": "JJDH34234KK",
        "location": "rear-left"
      }
    ]
  }
}
```

**Expected Output:**

```json
[
  [
    {
      "model": "KDJ39848T",
      "location": "front-right"
    },
    {
      "model": "MDJ39485DK",
      "location": "front-left"
    },
    {
      "model": "KCMDD3435K",
      "location": "rear-right"
    },
    {
      "model": "JJDH34234KK",
      "location": "rear-left"
    }
  ]
]
```

**JSON Path Query:**  
```bash
$.car.wheels
```

##### 6 / 13 List in Dictionary
Develop a JSON path query to extract data about the third wheel of the car.

**Expected Output:**

```json
[
  {
    "model": "KCMDD3435K",
    "location": "rear-right"
  }
]
```

**JSON Path Query:**  
```bash
$.car.wheels[2]
```

##### 7 / 13 List in Dictionary
Develop a JSON path query to extract the model of the third wheel of the car.

**Expected Output:**

```json
[
  "KCMDD3435K"
]
```

**JSON Path Query:**  
```bash
$.car.wheels[2].model
```

##### 8 / 13 List in Dictionary
Retreive all the payslip details of the employee.

**Source Data:**

```json
{
  "employee": {
    "name": "john",
    "gender": "male",
    "age": 24,
    "address": {
      "city": "edison",
      "state": "new jersey",
      "country": "united states"
    },
    "payslips": [
      {
        "month": "june",
        "amount": 1400
      },
      {
        "month": "july",
        "amount": 2400
      },
      {
        "month": "august",
        "amount": 3400
      }
    ]
  }
}
```

**Expected Output:**

```json
[
  [
    {
      "month": "june",
      "amount": 1400
    },
    {
      "month": "july",
      "amount": 2400
    },
    {
      "month": "august",
      "amount": 3400
    }
  ]
]
```

**JSON Path Query:**  
```bash
$.employee.payslips[]
```

##### 9 / 13 List in Dictionary
Retreive the 3rd month's pay details of the employee.

**Expected Output:**

```json
[
  {
    "month": "august",
    "amount": 3400
  }
]
```

**JSON Path Query:**  
```bash
$.employee.payslips[2]
```

##### 10 / 13 List in Dictionary
Retreive the 3rd month's pay amount of the employee.

**Expected Output:**

```json
[
  3400
]
```

**JSON Path Query:**  
```bash
$.employee.payslips[2].amount
```

##### 11 / 13 List in Dictionary
Give this a shot. Try to find Malala in the below list of Noble Prize Winners.

**Source Data:**

```json
{
  "prizes": [
    {
      "year": "2018",
      "category": "physics",
      "overallMotivation": "\"for groundbreaking inventions in the field of laser physics\"",
      "laureates": [
        {
          "id": "960",
          "firstname": "Arthur",
          "surname": "Ashkin",
          "motivation": "\"for the optical tweezers and their application to biological systems\"",
          "share": "2"
        },
        {
          "id": "961",
          "firstname": "Gérard",
          "surname": "Mourou",
          "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
          "share": "4"
        },
        {
          "id": "962",
          "firstname": "Donna",
          "surname": "Strickland",
          "motivation": "\"for their method of generating high-intensity, ultra-short optical pulses\"",
          "share": "4"
        }
      ]
    },
    {
      "year": "2018",
      "category": "chemistry",
      "laureates": [
        {
          "id": "963",
          "firstname": "Frances H.",
          "surname": "Arnold",
          "motivation": "\"for the directed evolution of enzymes\"",
          "share": "2"
        },
        {
          "id": "964",
          "firstname": "George P.",
          "surname": "Smith",
          "motivation": "\"for the phage display of peptides and antibodies\"",
          "share": "4"
        },
        {
          "id": "965",
          "firstname": "Sir Gregory P.",
          "surname": "Winter",
          "motivation": "\"for the phage display of peptides and antibodies\"",
          "share": "4"
        }
      ]
    },
    {
      "year": "2018",
      "category": "medicine",
      "laureates": [
        {
          "id": "958",
          "firstname": "James P.",
          "surname": "Allison",
          "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
          "share": "2"
        },
        {
          "id": "959",
          "firstname": "Tasuku",
          "surname": "Honjo",
          "motivation": "\"for their discovery of cancer therapy by inhibition of negative immune regulation\"",
          "share": "2"
        }
      ]
    },
    {
      "year": "2018",
      "category": "peace",
      "laureates": [
        {
          "id": "966",
          "firstname": "Denis",
          "surname": "Mukwege",
          "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
          "share": "2"
        },
        {
          "id": "967",
          "firstname": "Nadia",
          "surname": "Murad",
          "motivation": "\"for their efforts to end the use of sexual violence as a weapon of war and armed conflict\"",
          "share": "2"
        }
      ]
    },
    {
      "year": "2018",
      "category": "economics",
      "laureates": [
        {
          "id": "968",
          "firstname": "William D.",
          "surname": "Nordhaus",
          "motivation": "\"for integrating climate change into long-run macroeconomic analysis\"",
          "share": "2"
        },
        {
          "id": "969",
          "firstname": "Paul M.",
          "surname": "Romer",
          "motivation": "\"for integrating technological innovations into long-run macroeconomic analysis\"",
          "share": "2"
        }
      ]
    },
    {
      "year": "2014",
      "category": "peace",
      "laureates": [
        {
          "id": "913",
          "firstname": "Kailash",
          "surname": "Satyarthi",
          "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
          "share": "2"
        },
        {
          "id": "914",
          "firstname": "Malala",
          "surname": "Yousafzai",
          "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
          "share": "2"
        }
      ]
    },
    {
      "year": "2017",
      "category": "physics",
      "laureates": [
        {
          "id": "941",
          "firstname": "Rainer",
          "surname": "Weiss",
          "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
          "share": "2"
        },
        {
          "id": "942",
          "firstname": "Barry C.",
          "surname": "Barish",
          "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
          "share": "4"
        },
        {
          "id": "943",
          "firstname": "Kip S.",
          "surname": "Thorne",
          "motivation": "\"for decisive contributions to the LIGO detector and the observation of gravitational waves\"",
          "share": "4"
        }
      ]
    },
    {
      "year": "2017",
      "category": "chemistry",
      "laureates": [
        {
          "id": "944",
          "firstname": "Jacques",
          "surname": "Dubochet",
          "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
          "share": "3"
        },
        {
          "id": "945",
          "firstname": "Joachim",
          "surname": "Frank",
          "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
          "share": "3"
        },
        {
          "id": "946",
          "firstname": "Richard",
          "surname": "Henderson",
          "motivation": "\"for developing cryo-electron microscopy for the high-resolution structure determination of biomolecules in solution\"",
          "share": "3"
        }
      ]
    },
    {
      "year": "2017",
      "category": "medicine",
      "laureates": [
        {
          "id": "938",
          "firstname": "Jeffrey C.",
          "surname": "Hall",
          "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
          "share": "3"
        },
        {
          "id": "939",
          "firstname": "Michael",
          "surname": "Rosbash",
          "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
          "share": "3"
        },
        {
          "id": "940",
          "firstname": "Michael W.",
          "surname": "Young",
          "motivation": "\"for their discoveries of molecular mechanisms controlling the circadian rhythm\"",
          "share": "3"
        }
      ]
    }
  ]
}
```

**Expected Output:**

```json
[
  {
    "id": "914",
    "firstname": "Malala",
    "surname": "Yousafzai",
    "motivation": "\"for their struggle against the suppression of children and young people and for the right of all children to education\"",
    "share": "2"
  }
]
```

**JSON Path Query:**  
```bash
$.prizes[[:].laureates[?(@.firstname == "Malala")]]
```

##### 12 / 13 Lists
Retreive the first element in the list.

**Source Data:**

```json
[
  "car",
  "bus",
  "truck",
  "bike"
]
```
**Expected Output:**

```json
[
  "car"
]
```

**JSON Path Query:**  
```bash
$[0]
```


##### 13 / 13 Lists
Retreive the first and 4th element in the list.

**JSON Path Query:**  
```bash
$[0,3]
```