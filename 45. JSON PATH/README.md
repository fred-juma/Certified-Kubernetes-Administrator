#### JSON PATH:

This is a query language for parsing data  represented in json or yaml format

##### 1 / 13 Dictionary
Develop a JSON path query to extract the expected output from the Source Data.

**Source Data**
{
  "property1": "value1",
  "property2": "value2"
}

**Expected value**
[
  "value1"
]

**JSON Path Query:**  

```bash
$.property1
```

##### 2 / 13 Dictionary
Develop a JSON path query to extract the bus details from the Source Data.

**Source Data:**

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


{
  "car": {


**Expected Output:**

[
  {
    "color": "white",
    "price": "$120,000"
  }
]

** JSON Path Query:**  
```bash
$.bus
```

##### 3 / 13 Dictionary in Dictionary
Develop a JSON path query to extract the price of the bus.

**Expected Output:**

[
  "$120,000"
]

**JSON Path Query:** 
```bash
$.bus.price
```

##### 4 / 13 Dictionary in Dictionary
Develop a JSON path query to extract the price of the car.

**Source Data:**

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

**Expected Output:**

[
  "$20,000"
]

**JSON Path Query:**  
```bash
$.vehicles.car.price
```

##### 5 / 13 List in Dictionary
Develop a JSON path query to extract data about the wheels of the car.

**Source Data:**

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

**Expected Output:**

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

**JSON Path Query:**  
```bash
$.car.wheels
```

##### 6 / 13 List in Dictionary
Develop a JSON path query to extract data about the third wheel of the car.

**Expected Output:**

[
  {
    "model": "KCMDD3435K",
    "location": "rear-right"
  }
]

**JSON Path Query:**  
```bash
$.car.wheels[2]
```

##### 7 / 13 List in Dictionary
Develop a JSON path query to extract the model of the third wheel of the car.

**Expected Output:**

[
  "KCMDD3435K"
]

**JSON Path Query:**  
```bash
$.car.wheels[2].model
```