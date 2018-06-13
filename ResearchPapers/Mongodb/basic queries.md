## Mongo Queries:
--
```json
{a: 10}
```
Matches docs where ```a = 10``` or ```a = [12,10,22]```

```json
{a:10, b: "hello"}
```
Docs where a is ```10``` and b is ```hello```.

```json
{a: {$gt:10}} 
```
Docs where a is greater than ```10```. 
Use ```$lt``` for less than, ```$gte``` for greater than equal to, ```$ne``` for not equal to.

```json
{a:{$in:[10, "hello"]}}
```
Docs where a is ```10``` or hello.

```json
{"a.b": 10}
```
Docs where a is a map (embedded doc), with b equal to 10.

```json
{a : {
      $elemMatch:{$gt: 80, $lt: 85}
    }
}
```
Docs where a is an array and has atleast one element that is ```greater than 80``` and ```less than 85```.  
Sample:  
```{a:[80,56,78,82]}```, In this case its a match, as "82" is the element we are looking for.     
```{a:[80,34,56,78]}```, In this case there is no match as none element specifioes the criterion.

```json
{a: {$elemMatch:{b:1, c:2}}}
```
Docs where a is an array of embedded docs and atleast one embedded doc has ```b=1``` and ```c=2```.  
Sample:  
```{a:[{b:1,c:2,d:3}``` and ```{b:2,c:2,d:2}]``` both
Will match the first doc.

```json
{$or: [{a:1}, {b:2}]}
```
Get all docs where ```a=1``` OR ```b=2```.

```json
{a: /^m/}
```
Docs where a has a value that starts with "m"

```json
{a: {$mod:[10, 1]}}
```
Docs where a MOD 10 is 1

```json
{a:{$type:2}}
```
Docs where a is a string.  
Following is the type spec:  
1 - Double (integer)  
2 - String  
3 - Object  
4 - Array  
5 - ObjectID  
6 - Date
