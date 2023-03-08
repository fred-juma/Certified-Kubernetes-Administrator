#### 1 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Source Data:**

```json
[
  "Apple",
  "Google",
  "Microsoft",
  "Amazon",
  "Facebook",
  "Coca-Cola",
  "Samsung",
  "Disney",
  "Toyota",
  "McDonald's"
]
```

**Expected Output:**

```json
[
  "Apple"
]
```

**JSON Path query:**

```bash
$[0]
```

#### 2 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Expected Output:**

```json

[
  "Apple",
  "Facebook"
]
```

**JSON Path query:**

```bash
$[0,4]
```

#### 3 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Expected Output:**

```json
[
  "Apple",
  "Google",
  "Microsoft",
  "Amazon",
  "Facebook"
]
```

**JSON Path query:**

```bash
$[0:5]
```

#### 4 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Expected Output:**
```json
[
  "Microsoft",
  "Amazon",
  "Facebook",
  "Coca-Cola",
  "Samsung"
]
```
**JSON Path query:**

```bash
$[2:7]
```

#### 5 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Expected Output:**
```json
[
  "McDonald's"
]
```

**JSON Path query:**

```bash
$[-1:]
```

#### 6 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Expected Output:**
```json
[
  "Samsung",
  "Disney",
  "Toyota",
  "McDonald's"
]
```

**JSON Path query:**

```bash
$[6:]
```

#### 7 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Expected Output:**
```json
[
  "Amazon",
  "Facebook",
  "Coca-Cola",
  "Samsung",
  "Disney",
  "Toyota"
]
```

**JSON Path query:**

```bash
$[3:9]
```

#### 8 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Source Data:**
```json
[
  "Curabitur Vel Lectus Limited",
  "Libero Morbi Accumsan Industries",
  "Faucibus Ltd",
  "Eu Corp.",
  "Neque Sed Corporation",
  "Nunc Commodo Incorporated",
  "Taciti Sociosqu Industries",
  "Rutrum Lorem Corp.",
  "Proin Corp.",
  "Dolor Fusce Corporation",
  "Malesuada Ut Sem LLC",
  "Mattis Ornare PC",
  "Pede Nonummy Ut LLP",
  "Aliquam PC",
  "Eu Consulting",
  "Leo Morbi Neque Incorporated",
  "Suspendisse Institute",
  "In Tincidunt Congue Consulting",
  "Ipsum Inc.",
  "Nulla Aliquet Proin Consulting",
  "Lorem Luctus Ut Consulting",
  "Sed Sapien Nunc Associates",
  "Feugiat Tellus Industries",
  "Sem LLP",
  "Aliquam Enim Nec Inc.",
  "Feugiat Tellus PC",
  "Dis Parturient Limited",
  "Sed Dictum Corporation",
  "Eu PC",
  "Tellus Faucibus Leo Corp.",
  "Velit Company",
  "Mauris Aliquam Eu Corp.",
  "Rutrum Justo Praesent Industries",
  "Malesuada Fames Associates",
  "Vitae Sodales Foundation",
  "Amet Company",
  "Dignissim Tempor Limited",
  "Morbi Tristique Corp.",
  "Nisi Cum Sociis Foundation",
  "Donec Vitae Erat Incorporated",
  "Fringilla Industries",
  "Elit Incorporated",
  "Velit In Institute",
  "Odio A Purus Incorporated",
  "Hendrerit Id Institute",
  "Aliquet Magna Associates",
  "Dictum Eu Corporation",
  "Integer Mollis Integer Corp.",
  "Libero Proin Mi Foundation",
  "Purus Sapien Associates",
  "Fringilla Associates",
  "Ante Company",
  "Bibendum Company",
  "Convallis Ligula Donec Industries",
  "Elit Inc.",
  "Scelerisque Foundation",
  "Curae; Corp.",
  "Ornare PC",
  "Ut Ipsum Consulting",
  "Tortor Ltd",
  "Convallis Dolor Quisque Foundation",
  "Feugiat Metus Sit Corp.",
  "Nec Orci Incorporated",
  "Arcu Vel Institute",
  "Diam Duis Corp.",
  "Ut Cursus Luctus Incorporated",
  "Vitae LLP",
  "Sed Sem Company",
  "Pede Cum Ltd",
  "Laoreet Lectus Foundation",
  "Semper Dui Foundation",
  "Odio A Purus Inc.",
  "Rutrum Magna Cras PC",
  "A Felis Company",
  "Libero Et Tristique Incorporated",
  "Odio Etiam Associates",
  "Cum Sociis Natoque Industries",
  "Nulla Dignissim Maecenas Inc.",
  "Malesuada Incorporated",
  "Lorem Eu Metus Foundation",
  "In Company",
  "Class Aptent Incorporated",
  "Ac Arcu Nunc Institute",
  "Aliquet Molestie LLP",
  "Sed LLC",
  "Pede LLP",
  "Ante Ipsum Primis Corporation",
  "Eu Dolor Ltd",
  "A Aliquet Consulting",
  "Lacinia Limited",
  "Pretium Aliquet Limited",
  "Magna Nec Corp.",
  "Egestas Corporation",
  "Est Congue Associates",
  "Non Cursus Inc.",
  "Elit Fermentum Associates",
  "Consectetuer Adipiscing Elit Limited",
  "Accumsan Convallis PC",
  "In Ltd"
]
```

**Expected Output:**
```json
[
  "In Ltd"
]
```

**JSON Path query:**

```bash
$[-1:]
```

#### 9 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Expected Output:**
```json
[
  "Consectetuer Adipiscing Elit Limited",
  "Accumsan Convallis PC",
  "In Ltd"
]
```

**JSON Path query:**

```bash
$[-3:]
```


#### 10 / 12 Lists
Develop a JSON path query to extract the expected output from the Source Data.

**Expected Output:**
```json
[
  "Magna Nec Corp.",
  "Egestas Corporation",
  "Est Congue Associates",
  "Non Cursus Inc.",
  "Elit Fermentum Associates",
  "Consectetuer Adipiscing Elit Limited"
]
```

**JSON Path query:**

```bash
$[-8:-2]
```

#### 11 / 12 Wildcard - List in Dictionary
Data about a ser of users are given. Develop a JSON path query to extract the phone numbers of first 5 users.

**Source Data:**
```json
[
  {
    "age": 35,
    "name": "Tameka Lane",
    "gender": "female",
    "phone": "+1 (850) 469-2827"
  },
  {
    "age": 26,
    "name": "Kristy Day",
    "gender": "female",
    "phone": "+1 (825) 558-2599"
  },
  {
    "age": 36,
    "name": "Nieves Hill",
    "gender": "male",
    "phone": "+1 (946) 495-3285"
  },
  {
    "age": 30,
    "name": "Dianna Holland",
    "gender": "female",
    "phone": "+1 (948) 406-2941"
  },
  {
    "age": 23,
    "name": "Marsh Robertson",
    "gender": "male",
    "phone": "+1 (903) 413-2132"
  },
  {
    "age": 33,
    "name": "Valenzuela Mcbride",
    "gender": "male",
    "phone": "+1 (998) 499-2074"
  },
  {
    "age": 40,
    "name": "Virginia Michael",
    "gender": "female",
    "phone": "+1 (898) 505-3869"
  },
  {
    "age": 38,
    "name": "Mueller Keller",
    "gender": "male",
    "phone": "+1 (805) 555-3665"
  },
  {
    "age": 37,
    "name": "Madeline Farley",
    "gender": "female",
    "phone": "+1 (954) 446-2747"
  },
  {
    "age": 23,
    "name": "Potter Casey",
    "gender": "male",
    "phone": "+1 (948) 538-3644"
  },
  {
    "age": 24,
    "name": "Melinda Hardy",
    "gender": "female",
    "phone": "+1 (944) 557-2486"
  },
  {
    "age": 34,
    "name": "Monique Carey",
    "gender": "female",
    "phone": "+1 (863) 424-2359"
  },
  {
    "age": 20,
    "name": "Marianne Britt",
    "gender": "female",
    "phone": "+1 (846) 462-2844"
  },
  {
    "age": 37,
    "name": "Guy Langley",
    "gender": "male",
    "phone": "+1 (905) 401-3848"
  },
  {
    "age": 40,
    "name": "Hurst Hogan",
    "gender": "male",
    "phone": "+1 (934) 587-3143"
  }
]
```

**Expected Output:**
```json
[
  "+1 (850) 469-2827",
  "+1 (825) 558-2599",
  "+1 (946) 495-3285",
  "+1 (948) 406-2941",
  "+1 (903) 413-2132"
]
```

**JSON Path query:**

```bash
$[0:5].phone
```

#### 12 / 12 Wildcard - List in Dictionary
Data about a set of users are given. Develop a JSON path query to extract the age of last 5 users.

**Expected Output:**
```json
[
  24,
  34,
  20,
  37,
  40
]
```

**JSON Path query:**

```bash
$[-5:].age
```