## Memory Consumption of a int.
An integer takes up 32bits(4 bytes) on a 32bit system and 64 bits(8 bytes) on a 64 bit system.
int8 takes up 8 bits, int16 takes up 16 bits.

## Memory Consumption of a string.
A string is nothing but a slice of bytes. In Go code a string is defined as:
```
type stringStruct struct {
    len int               //length of string
    str unsafe.Pointer    //pointer to actual string data
}
```
Thus a string takes up only 16 bytes (8 + 8) on stack and rest of the string data is kept on heap.

## Memory Consumption of a slice.
As per Go Coding terms, a slice is represented by following structure:
```
type SliceHeader struct {
        Data uintptr   // pointer to actual underlying array
        Len  int       // length of slice
        Cap  int       // Capacity of slice
}
```
Since, we know that integers take 4 bytes on 32-bit systems and 8 bytes on 64-bit systems. The `Len` and `Cap` will take 8 bytes each.
Similarly, a pointer also takes 8 bytes on a 64-bit system, Thus, `Data` field will also take 8 bytes.
Total Size of struct will be given as : 8 + 8 + 8 = 24 bytes.
Thus a slice takes up **24 bytes** (Ignoring the size of underlying Array). 

And since we know that each block of memory can have 1 byte of data. It will take 24 Memory blocks to store it.

## Memory Consumption of a interface{}, func() and channel.
Each takes up 8 bytes on stack, remaining memory is allocated on heap.

## Memory Consumption of structs:
Structs are stored in **contiguous memory** locations. Even when a struct contains other structs.
In case the struct contains of reference type structures such as: map, slice etc. The pointer to the reference structure is stored in contiguous
memory layout and not the reference structure data.

Lets see the truth of above statement via following examples.

In order to get the size of struct, we will use following function:
```
func getSize(i interface{}) {
	typ := reflect.TypeOf(i)
	fmt.Printf("Struct is %d bytes long\n", typ.Size())
}
```
1. First example below:
```
type User struct {
	Age  int 
	Info Data
}

type Data struct {
	Data string
}

func main() {
	user := User{12, Data{"A"}}
	getSize(user)
}
OUTPUT:
Struct is 24 bytes long
```

We see that it says the User struct is of size 24 bytes. Lets see how?
The User struct consists of two fields: Age and Info. 
*Age* field takes up 8 bytes(=64 bits), As we know that integers takes up 64 bits on 64 bit systems. 
*Info* field consists of a struct type *Data*, and this Data struct has only one field of type string. We know that a string field takes 16 bytes on stack and the actual string info is strored in heap.
Thus, the total size required by User struct is **24 bytes** of contiguos memory location.

2. Lets see one more example of the same, to understand better.
```
type Point struct {
  X, Y int
}
type Rect1 struct { 
  Min, Max Point 
}
type Rect2 struct { 
  Min, Max *Point 
}

r1 := Rect1{Point{10,20}, Point{30, 40}}
r2 := Rect2{&Point{10,20}, &Point{30, 40}}

getsize(r1)
getsize(r2)

OUTPUT:
Struct is 32 bytes
Struct is 16 bytes
```
In the above example a *Point* structure has 2 integer fields, thus have size of 16 bytes (8 + 8).
Since, Rect1 contains two *Point* structures it has size of 16 + 16 = 32 bytes.
Rect2 contains pointers to *Point* strcutures instead of the Point structures themselves.And a pointer takes up 8 bytes of memory, which means
Rect2 takes up 16 bytes of contiguos memory.  

Note That, its not that Rect2 takes up less space than Rect1, both actually needs the same amount of space. Infact Rect2 needs more space as it needs
to store pointers too. The difference we are talking about here is of *contiguos space* required by both.

Layout of Rect1 in memory:
Rect1{ Point{10,20}, Point{30, 40}}
[10][20][30][40]   //contiguous

Layout of Rect2 in memory:
Rect2{ &Point{10,20}, &Point{30, 40}}
[0Xsde345][0xsddeed12]  //contiguous
   \           \
[10][20]      [30][40]

Performance tests of above code illustrates that its better to use Embedded structs than pointers:
```
With Pointers         BenchmarkFillRect2-4    2000000000               1.81 ns/op          
Without Pointers      BenchmarkFillRect1-4    2000000000               0.44 ns/op
```
However, as the size of Embedded struct grows bigger and bigger, the performance numbers stated above starts changing. Following is the benchmark
when *Point* struct was made to have size of 240 bytes (30 integer fields)
```
With Pointers         BenchmarkFillRect2-4    100000000               10.7 ns/op
Without Pointers      BenchmarkFillRect1-4    100000000               11.0 ns/op
```
So, we can deduce that if Struct is smaller in size i.e less than **250 bytes**, Its safe to use it without Pointer.
If its a large struct > 250 bytes try using the pointer approach.

#### Choosing b/w []int and *[]int.
Though Slices are referred to as Reference Type. They posses some interesting properties of value types. Stated below:
1. Assignment creates a copy.

```
slice1 := []string{"A", "B", "C"}
slice2 := slice1 //Assigning is not by reference, So slice2 is actually a copy of slice1 and nothing more. 
```
Below is the sample code to illustrate the same:
```
func main() {

	slice1 := []string{"A", "B", "C"}
	slice2 := slice1

	slice1 = slice1[:len(slice1)-1] //Remove last element of slice1

	fmt.Printf("%v\n", slice1)
	fmt.Printf("%v\n", slice2)
}
//OutPut:
[A B]
[A B C]
```
2. When passing through function also creates a copy. Below is the code example:
```
func main() {

	slice1 := []string{"A", "B", "C"}

	testslice(slice1)
	fmt.Printf("%v\n", slice1)
}

func testslice(sliceCopy []string) { //Remove last element
	sliceCopy = sliceCopy[:len(sliceCopy)-1]
}
//Output:
[A B C]
```

So, Now we know that a pointer to slice is slightly cheaper than passing the slice directly to the function. As passing directly, will create
a new copy of slice which will take up 24 bytes of additional memory, While, passing a pointer will take only 8 bytes of additional memory. 

Since the difference b/w the memory usage of the two is **not** enormously higher, It would make sense to pass a slice directly and only to use the
pointer approach when you need to modify the slice itself. Thus, avoiding the additional complexities imposed by pointer. 

#### Choosing b/w []struct and []*struct
Now, this is an interesting choice to make because we know that slices points to underlying array, and arrays are stored in contiguous memory locations.
Thus, as the size of slice increases so is the size of underlying array, and if the array is too large we need a large chunk of contiguos memory free
to accmulate the whole array, which can be hard to find out. Since the Structs also gets stored in contiguous memory location they act as a catalyst to this problem.
However, the benefit of using Structs instead of *Struct is that they will be kept in contiguos locations thus will help in faster access time.

So, a Benchmarking was necessary to get the proper insight. Following is the code benchmarked:
```
type Point struct {
	A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S,
	T, U, V, W, X, Y, Z, Z1, Z2, Z3, Z4, Z5, Z6 int
}

func fillRect1() {
	slice := make([]Point, 0)
	var i int
	for i < 100 {
		i++
		slice = append(slice, Point{})
	}
	for _, point := range slice {
		point.A = 9
	}
}

func fillRect2() {
	slice := make([]*Point, 0)
	var i int
	for i < 100 {
		i++
		slice = append(slice, &Point{})
	}
	for _, point := range slice {
		point.A = 9
	}
}
``` 
Below is the result of benchmarking, which shows that in this case its better to use Pointer to struct approach.
```
With Pointer       BenchmarkFillRect2-4      100000             13012 ns/op
Without Pointer    BenchmarkFillRect1-4      100000             23228 ns/op
```
However, if we change the size of Point struct to take about 100 bytes. Then the benchmarking results change completely.
```
With Pointer        BenchmarkFillRect2-4     2000000               869 ns/op
Without Pointer     BenchmarkFillRect1-4     3000000               560 ns/op
```
So, As a general rule of thumb if struct size is greater than **100 bytes** use slice of Pointer to struct else use simply slice of structs.
Also, if the array contains more than 100 elements use pointer approach.s
