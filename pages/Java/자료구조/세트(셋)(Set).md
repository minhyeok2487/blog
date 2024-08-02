# 기초적인 Set
>[!INFO]
>셋은 인덱스가 없기 때문에 단순히 데이터를 넣고, 데이터가 있는지 확인하고, 데이터를 삭제하는 정도로 단순하게 구현 할 수 있습니다.
>단, 데이터를 추가할 때 중복 여부를 체크해야 합니다.

## 단순 구현 (add, contains)
```Java
public class MyHashSetV0 {  
  
    private int[] elementData = new int[10];  
    private int size = 0;  

	// 셋에 값을 추가한다. 중복 데이터는 저장하지 않는다.
    // O(n)
    public boolean add(int value) {  
        if (contains(value)) {  
            return false;  
        }  
        elementData[size] = value;  
        size++;  
        return true;  
    }  

	// 셋에 값이 있는지 확인한다.
    // O(n)  
    public boolean contains(int value) {  
        for (int data : elementData) {  
            if (data == value) {  
                return true;  
            }  
        }  
        return false;  
    }  
  
    public int size() {  
        return size;  
    }  
  
    @Override  
    public String toString() {  
        return "MyHashSetV0{" +  
                "elementData=" + Arrays.toString(Arrays.copyOf(elementData, size)) +  
                ", size=" + size +  
                '}';  
    }  
}
```

add()로 데이터를 추가할 때 
contains() 메소드로 셋에 중복 데이터가 있는지 전체 데이터를 항상 확인해야 합니다
-> 중복 데이터 검색 O(n) + 데이터 입력 O(1) = O(n)

>[!check]
>단, 이렇게 구성되면 데이터를 추가할 때마다 중복 데이터가 있는지 체크하기 위해 
>셋의 전체데이터를 확인해야 하므로 데이터가 많아질수록 효율은 매우 떨어집니다. 

이러한 문제를 해결하기 위한 알고리즘이 **해시(Hash) 알고리즘** 입니다.

## 해시 알고리즘
>[!INFO]
>배열은 인덱스의 위치를 사용해서 데이터를 찾을 때 O(1)로 매우 빠른 특징을 가지고 있습니다.
>해시는 이러한 특징을 활용해서 데이터의 값 자체를 배열의 인덱스로 사용하는 것 입니다.

### Index 활용

![[Pasted image 20240802090833.png]]
- 인덱스 1 -> 데이터 1
- 인덱스 2 -> 데이터 2
- 인덱스 5 -> 데이터 5
- 인덱스 8 -> 데이터 8

데이터가 인덱스 번호가 되므로 배열의 인덱스를 활용해서 
기존 O(n)의 검색 연산을 O(1)의 검색 연산으로 바꿀 수 있습니다.

```Java
public class HashStart2 {  
  
    public static void main(String[] args) {  
        //입력: 1, 2, 5, 8  
        //[null, 1, 2, null, null, 5, null, null, 8, null]        
        Integer[] inputArray = new Integer[10];  
        inputArray[1] = 1;  
        inputArray[2] = 2;  
        inputArray[5] = 5;  
        inputArray[8] = 8;  
        System.out.println("inputArray = " + Arrays.toString(inputArray));  
  
        int searchValue = 8;  
        Integer result = inputArray[searchValue]; //O(1)  
        System.out.println("result = " + result);  
    }  
}
```

**실행결과**
```
inputArray = [null, 1, 2, null, null, 5, null, null, 8, null] 
result = 8
```

![[Pasted image 20240802091213.png]]

>[!check]
>단, 이러한 방식은 배열의 크기를 입력 값의 범위만큼 지정해야 합니다.
>따라서 배열에 낭비되는 공간이 많이 발생하는 문제가 있습니다.

### 메모리 낭비
>[!question]
>만약 입력 값의 범위를 0~99로 넓히면 어떻게 될까요?

데이터가 0~99까지 입력될 수 있어야 하므로 크기 100의 배열이 필요합니다.
```Java
public class HashStart3 {  
  
    public static void main(String[] args) {  
        //입력: {1, 2, 5, 8, 14, 99}  
        //[null, 1, 2, null, null, 5, null, null, 8, null, ..., 14, ..., 99]
        Integer[] inputArray = new Integer[100];  
        inputArray[1] = 1;  
        inputArray[2] = 2;  
        inputArray[5] = 5;  
        inputArray[8] = 8;  
        inputArray[14] = 14;  
        inputArray[99] = 99;  
        System.out.println("inputArray = " + Arrays.toString(inputArray));  
  
        int searchValue = 99;  
        Integer result = inputArray[searchValue]; //O(1)  
        System.out.println("result = " + result);  
    }  
}
```

**실행결과**
```
inputArray = [null, 1, 2, null, null, 5, null, null, 8, null, null, null, null, null, 14, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 99] 
result = 99
```

만약 99을 넘어 int 숫자의 모든 범위를 입력할 수 있도록 하려면 배열의 크기를 얼마로 잡아야 할까요?
- int 범위: -2,147,483,648 ~ 2,147,483,647 
- 약 42억 사이즈의 배열 필요 (+- 모두 포함) 
- 4byte * 42억 = 약 17기가바이트 필요 (단순히 값의 크기로만 계산)

데이터의 값을 인덱스로 사용할 때, 입력할 수 있는 값의 범위가 int라면
한번의 연산에 컴퓨터의 메모리가 거의 다 소모되어 버립니다.

또한, 만약 사용자가 1, 2, 1000, 200000의 네 개의 값만 입력한다면 
나머지 대부분의 메모리가 빈 공간으로 낭비될 것입니다.
따라서 데이터의 값을 인덱스로 사용하는 방식은 입력 값의 범위가 넓다면 사용하기 어려워 보입니다.

이러한 문제를 해결하기 위한 방법이 **나머지 연산** 입니다.

### 나머지 연산 - 해시 인덱스(Hash Index)
앞에서 이야기한 것 처럼 모든 숫자를 입력할 수 있다고 가정하면,
입력 값의 범위가 너무 넓어져서 데이터의 값을 인덱스로 사용하기 어렵고 공간 낭비가 심합니다.

여기서 나머지 연산을 사용하여 공간도 절약하면서, 넓은 범위의 값을 사용할 수 있습니다.
저장할 수 있는 배열의 크기(CAPACITY)를 10이라고 가정하고, 그 크기에 맞추어 나머지 연산을 진행합니다.

- 1 % 10 = 1 
- 2 % 10 = 2 
- 5 % 10 = 5 
- 8 % 10 = 8 
- 14 % 10 = 4 
- 99 % 10 = 9

여기서 14, 99는 10보다 큰 값입니다. 
따라서 일반적인 방법으로는 크기가 10인 배열의 인덱스로 사용할 수 없습니다. 
하지만 나머지 연산의 결과를 사용하면 14는 4로, 99는 9로 크기가 10인 배열의 인덱스로 활용할 수 있습니다.

이렇게 배열의 인덱스로 사용할 수 있도록 원래의 값을 계산한 인덱스를 **해시 인덱스(Hash Index)** 라고 합니다.
해시 인덱스로 데이터를 저장 및 조회하면 아래와 같은 그림이 됩니다.

데이터 저장
![[Pasted image 20240802092617.png]]

데이터 조회
![[Pasted image 20240802092659.png]]

```Java
public class HashStart4 {  
  
    static final int CAPACITY = 10;  
  
    public static void main(String[] args) {  
        //입력: {1, 2, 5, 8, 14, 99}  
        System.out.println("hashIndex(1) = " + hashIndex(1));  
        System.out.println("hashIndex(2) = " + hashIndex(2));  
        System.out.println("hashIndex(5) = " + hashIndex(5));  
        System.out.println("hashIndex(8) = " + hashIndex(8));  
        System.out.println("hashIndex(14) = " + hashIndex(14));  
        System.out.println("hashIndex(99) = " + hashIndex(99));  
        Integer[] inputArray = new Integer[CAPACITY];  
        add(inputArray, 1);  
        add(inputArray, 2);  
        add(inputArray, 5);  
        add(inputArray, 8);  
        add(inputArray, 14);  
        add(inputArray, 99);  
        System.out.println("inputArray = " + Arrays.toString(inputArray));  
  
        //검색  
        int searchValue = 14;  
        int hashIndex = hashIndex(searchValue);  
        System.out.println("searchValue hashIndex = " + hashIndex);  
        Integer result = inputArray[hashIndex]; // O(1)  
        System.out.println(result);  
    }  
  
    private static void add(Integer[] inputArray, int value) {  
        int hashIndex = hashIndex(value);  
        inputArray[hashIndex] = value;  
    }  
  
    static int hashIndex(int value) {  
        return value % CAPACITY;  
    }  
}
```

**실행 결과**
```
hashIndex(1) = 1
hashIndex(2) = 2
hashIndex(5) = 5
hashIndex(8) = 8
hashIndex(14) = 4
hashIndex(99) = 9
inputArray = [null, 1, 2, null, 14, 5, null, null, 8, 99]
searchValue hashIndex = 4
14
```

데이터 저장 시 인덱스만 해시 인덱스를 사용하고 값은 원래 값을 저장하고,
데이터 조회 시 해시 인덱스를 구하고 해시 인덱스를 배열의 인덱스로 사용해서 데이터를 조회합니다.

>[!check]
>이러한 방식으로 입력 값의 범위가 넓어도 실제 모든 값이 들어오지는 않기 때문에
>배열의 크기를 제한하고, 나머지 연산을 통해 메모리가 낭비되는 문제도 해결 할 수 있습니다.
>그런데, 저장할 위치가 충돌 할 수 있다는 한계가 있습니다.

### 해시 충돌 해결
>[!INFO]
>99, 9의 두 값은 10으로 나누면 나머지가 9 입니다.
>따라서 다른 값을 입력했지만 같은 해시 코드가 나오게 되는데 이것을 **해시 충돌** 이라고 합니다.

![[Pasted image 20240802093511.png]]

이 문제를 해결하기 위한 가장 단순한 방법은 CAPACITY를 값의 입력 범위만큼 키우는 겁니다.
여기서는 CAPACITY를 100으로 늘리면 충돌이 발생하지 않습니다.
하지만 앞서 보았듯이 이 방법은 메모리 낭비가 심합니다.

여기서 해결 방법은 해시 충돌이 일어 날 수 있다고 가정하고,
데이터를 배열로 저장하는 것 입니다.
![[Pasted image 20240802093702.png]]

```Java
public class HashStart5 {  
  
    static final int CAPACITY = 10;  
  
    public static void main(String[] args) {  
        //입력: {1, 2, 5, 8, 14, 99, 9}  
        //LinkedList는 가끔 충돌 일어남  
        LinkedList<Integer>[] buckets = new LinkedList[CAPACITY]; 
        System.out.println("buckets = " + Arrays.toString(buckets));  
        for (int i = 0; i < CAPACITY; i++) {  
            buckets[i] = new LinkedList<>();  
        }  
  
        System.out.println("buckets = " + Arrays.toString(buckets));  
  
        add(buckets, 1);  
        add(buckets, 2);  
        add(buckets, 5);  
        add(buckets, 8);  
        add(buckets, 14);  
        add(buckets, 99);  
        add(buckets, 9); //hashIndex 중복  
        System.out.println("buckets = " + Arrays.toString(buckets));  
  
        //검색  
        int searchValue = 9;  
        boolean contains = contains(buckets, searchValue);  
        System.out.println("buckets.contains(" + searchValue + ") = " + contains);  
    }  
  
    private static void add(LinkedList<Integer>[] buckets, int value) {  
        int hashIndex = hashIndex(value);  
        LinkedList<Integer> bucket = buckets[hashIndex]; // O(1)  
        if (!bucket.contains(value)) { // 중복 체크 O(n)
	        bucket.add(value);  
        }  
    }  
  
    private static boolean contains(LinkedList<Integer>[] buckets, int searchValue) {  
        int hashIndex = hashIndex(searchValue);  
        LinkedList<Integer> bucket = buckets[hashIndex]; // O(1)  
        return bucket.contains(searchValue); // O(n)  
    }  
  
    static int hashIndex(int value) {  
        return value % CAPACITY;  
    }  
}
```

**실행 결과**
```
buckets = [[], [1], [2], [], [14], [5], [], [], [8], [99, 9]] 
bucket.contains(9) = true
```
