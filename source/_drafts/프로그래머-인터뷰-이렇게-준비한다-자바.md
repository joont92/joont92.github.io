# 1장 
전화면접에서 라이브코딩을 할 수도 있다  

지원한 회사의 기본 사항은 알아야한다. about us, 회사 블로그 등..  
작은 회사라면 CEO나 CTO의 정보를 찾아보는 것도 좋다.  

마지막에 질문 없냐는 질문에 질문할 것이 없다고 하면 안된다.  
내가 지원한 역할에 심사숙고 하지 않는 것 처럼 보이기 때문.  
처우 얘기는 나중에 하고, 질문은 준비해가야 한다.  

지원하는 팀에서 어떤 언어나 프레임워크를 사용하는 지 알아야 함  

코딩테스트 시에 단위테스트 작성 필수  

회사의 업무, 사용 기술을 명확히 알고 있어야 한다  

면접관은 내가 어느부분까지 아는가가 중요하지, 대답을 못했다는 사실을 중요하게 생각하지 않는다  

내가 팀에 어울리는지, 회사에서 경력을 어떻게 쌓을 것인지.  
커뮤니케이션 능력이 괜찮은지.  

채용이 되었으면 내가 주도권을 갖고 협상을 할 수 있게 된다.  
회사에서도 나를 채용하기 위해 많은 시간을 소요했고, 나와 같이 일하고 싶다는 뜻이기 때문.  
하지만 무리한 요구를 하지는 말자.  

면접은 시험을 치르는 것과 다르다.  
올바른 답을 말했다고 해서 합격되는 것이 아니다.  
그 외에 다른 부분도 맞아야하고, 팀에도 내가 맞아야 합격할 수 있는 것이다.  

면접을 잘 보는 것은 당연히, 철저히 준비하는 것이다!!!  
모든 질문에 대해 다 준비하자.  
그만두고 뭐했는지, 왜 여기 오고 싶은지, 왜 이직했는지,  
왜 퇴사했는지 등..  

# 2장
면접관들은 다 바쁘다! 간결하게 이력서를 작성하자.  
가장 보여주고 싶은 것, 가장 눈길을 끌수 있는 것을 첫 페이지에 작성하자.  
사진은 합격여부에 아무런 영향을 주지 않으므로 빼도 된다는 것이다. 지면 낭비를 하지 말자!  

앞서서 부끄러워 하지말고, 앞서서 몸 사리지 말자.  
어쩌피 판단은 고용하는 측에서 하게 될것이고,  
나는 그냥 내 있는 모습 그대로를 보여주면 된다.  
근자감을 장착하자!  

거짓말 하지말고, 이력서는 2줄을 넘지 않도록.  
그리고 현 상황에선 안타까운 말이지만, 이력서는 주기적으로 업데이트 해주는 것이 좋다.  
어느 부분이 필요한지, 필요하지 않은지 보면 볼 수록 보이기 때문.  

# 3장
문제 답은 읽기 쉽게 작성해야 한다.  
단위테스트를 잘 작성하면 면접관들은 대부분 감명을 받는다.. ^^ㅋ  
의사코드 작성에서는 문법이 그렇게 중요하지 않다.  
하지만 화이트보드에 자바 문법으로 쓰라고 하는경우(...) 문법을 정확히 쓰라는데.. 이런 기업을 가야하나?  

# 4장
리스트정렬  
Comporable은 클래스에 직접 선언, Comporator는 정렬 메서드에 인자로 줘서 정렬 방식을 직접 컨트롤 하고자 할 때 사용.    
클래스를 생성할 때 정렬을 원하면 Comporable 클래스를 구현해야 한다.  
Collections.sort 매개변수로 Comporable 참조 변수를 받기 때문.  
compare(T a, T b) 구현해야함  
a가 더크면 음수, 같으면 0, b가 더 크면 양수  

## Bubble Sort  
리스트 젤 앞부터 비교를 시작.  
첫번째 인덱스 부터 시작해서 앞의 2개를 비교해서 앞의 것이 더 크면 뒤로 보냄  
그리고 인덱스를 하나 증가.  
이렇게 1바퀴 돌면 마지막에 젤 큰 숫자가 들어가게 됨.  
다시 첫번째 인덱스부터 시작해서 1바퀴 돔. 마지막 숫자는 결정되 있으므로 돌 필요 없음.  

```java
@Test
public void bubbleSort(){
    int[] arr = {9,3,5,2,1,7};
    int[] expected = {1,2,3,5,7,9};

    for (int i = 1; i < arr.length; i++) {
        for (int j = 0; j < arr.length-i; j++) {
            if(arr[j] > arr[j+1]){
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }

    assertThat(arr, is(expected));
}
```

## Insertion Sort  
2번째 요소부터 비교 시작
자기 앞부분의 요소를 거꾸로 비교하며 자신보다 큰 숫자가 있는지 체크한다.  
찾은 숫자들 중 인덱스 0에서 가장 가까운 곳의 앞쪽으로 자신을 인서트 시킨다.  
(자신보다 작은 요소들은 이미 정렬되었다고 가정)  
자신이 들어갈 위치를 찾고 인서트하는 것이 중요하지, 뒤에서 도나 앞에서 도나는 상관없다.  
사실상 앞에서 도는게 더 효율적이다 ㅠ  

```java
@Test
    public void insertionSort(){
        List<Integer> arr = new LinkedList<>();
        arr.add(9);arr.add(3);arr.add(5);
        arr.add(2);arr.add(1);arr.add(7);

        List<Integer> expected = new ArrayList<>();
        expected.add(1);expected.add(2);expected.add(3);
        expected.add(5);expected.add(7);expected.add(9);

        for (int i = 1; i < arr.size(); i++) {
            int idx = i;

            // 뒤에서 깎는대신 0부터 하는게 더 빠르겠다..
            for (int j = i-1; j >= 0; j--) {
                if(arr.get(j) > arr.get(i)){
                    idx = j;
                } else{
                    break;
                }
            }

            if(i != idx){
                arr.add(idx, arr.remove(i));
            }
        }

        assertThat(arr, is(expected));
    }
```

## Quick Sort  
참고 : https://gmlwjd9405.github.io/2018/05/10/algorithm-quick-sort.html  
기준점(pivot)을 하나 정하고, pivot보다 작은 숫자들, pivot보다 큰 숫자들로 나눔  
나누어진 리스트에 똑같은 행위를 반복하여 리스트의 개수가 1개 이하로 떨어지게 함  
그 모든 리스트를 다 합침  

```java
private List<Integer> quickSortByRecursion(List<Integer> list){
    if(list.size() <= 1){
        return list;
    }

    List<Integer> small = new ArrayList<>();
    List<Integer> large = new ArrayList<>();

    int pivot = list.remove(0);

    for (Integer i : list) {
        if(i < pivot){
            small.add(i);
        } else{
            large.add(i);
        }
    }

    small = quickSortByRecursion(small);
    large = quickSortByRecursion(large);

    small.add(pivot);

    return Stream.of(small, large)
            .flatMap(List::stream)
            .collect(Collectors.toList());
}
```

## Merge Sort  
리스트를 2개로 계속 분할  
최종적으로 더이상 분할되지 않을 때 아래에서부터 하나씩 정렬하면서 합병  

분할  
21 10 12 20 25 13 15 22  
21 10 12 20 / 20 13 15 22  
21 10 / 12 20 // 20 13 / 15 22  

정렬  
-> 10 21 / 12 20 // 13 20 / 15 22  

합병/정렬  
2개의 리스트를 두고 작은값 부터 새로운 리스트에 넣는 방식으로 합병한다.  
10 21 / 12 20  
두 리스트의 첫번째 인덱스인 10과 12를 비교한다.  
10이 더 작으므로 새로운 배열에 10을 넣음.  
첫번째 리스트의 인덱스는 증가, 두번째 리스트의 인덱스는 그대로.  
21과 12비교. 12가 더 작으므로 새로운 배열에 넣음.  
이 과정을 반복하고, 한쪽 리스트가 다 소진될 경우 남은 리스트를 새로운 리스트에 append.  

```java
private List<Integer> mergeSortByRecursion(List<Integer> list){
    if(list.size() <= 3){
        Collections.sort(list);
        return list;
    }

    int mid = list.size() / 2;

    List<Integer> left = mergeSortByRecursion(list.subList(0, mid));
    List<Integer> right = mergeSortByRecursion(list.subList(mid, list.size()));

    List<Integer> newList = new ArrayList<>();
    int l = 0;
    int r = 0;

    while(l < left.size() && r < right.size()){
        if(left.get(l) < right.get(r)){
            newList.add(left.get(l++));
        } else{
            newList.add(right.get(r++));
        }
    }

    if(l < left.size()){
        newList.addAll(left.subList(l, left.size()));
    } else if(r < right.size()){
        newList.addAll(right.subList(r, right.size()));
    }

    return newList;
}
```

## binary search
정렬된 리스트를 순차적으로 탐색하지 않고 반으로 나눠가면서 찾는 방식  

# 5장
## 리스트
ArrayList, LinkedList  
둘다 List 인터페이스를 사용함  

자바 네이티브 array와 다른 점은 크기를 지정하지 않아도 된다는 점.  
하지만 ArrayList의 경우는 내부적으로 자바 네이티브 array를 사용함.  
초반에 크기는 0, 첫 요소 추가시 10으로 늘림, 이후로는 1씩 늘림.  
배열의 크기 확장은 더 큰 배열을 만들고 거기에 요소를 복사하는 행위이므로  
메모리 낭비가 크다.  
그래서 잦은 삽입/삭제를 하는 배열의 경우 ArrayList는 효율적이지 않다.  
배열의 초기값을 지정해줌으로써 크기를 늘리기 위해 매번 새로운 배열로 copy 하는 행위를 어느정도 줄일 수 있다.  
new ArrayList<>(1000); 이런식으로 아예 큰 리스트의 경우 크기를 초기화 시켜버리는 것이다.  
이렇게 하면 배열의 뒷부분에 요소를 삽입/삭제할 경우 O(1)의 시간복잡도로 빠르지만,  
배열의 중간요소(특히 앞부분에 가까운 요소)에 삽입/삭제를 할 경우 뒷부분 요소들이 빈공간을 만들거나 채우기 위해 움직여야 하므로 부하가 크다.  
그래서 삽입/삭제가 빈번한 리스트의 경우 LinkedList를 사용하는 것이 낫다.  
하지만 탐색의 경우 ArrayList는 인덱스로 바로 접근(랜덤 엑세스) 가능하기 때문에 O(1)의 시간 복잡도를 가진다.  

LinkedList의 경우 Node끼리 연결된 리스트이다.  
Node는 value, next(재귀 형태)의 값을 가진 클래스이다.  
그러므로 삽입/삭제의 경우 굉장히 간단하게 수행 가능하다. next에 저장된 다음 Node의 순서만 바꿔주면 된다.  
하지만 탐색의 경우 앞에서 부터 순차 탐색을 해야 하기 때문에 ArrayList에 비해 느리다.  
next 대신 prev를 갖고 있는 double linked list도 있는데, 이럴 경우 후위 탐색도 가능하기 때문에 그나마 빠르다!  

둘의 특징을 잘 비교하여 적절한 리스트를 고르도록 해야한다.  
간단하게 잦은 탐색이 필요하고, 리스트의 크기가 클 경우 ArrayList가 낫고, 잦은 삽입/삭제(특히 앞부분)이 일어날 경우 LinkedList가 낫다.  

## 큐
선입선출로 데이터를 처리하는 자료구조.  
일상 생활에서 보는 줄서는 시스템이 전부 큐 라고 보면 된다.  
enqueue하는 데이터는 뒤에 붙고, dequeue하는 데이터는 앞에서 부터 빠진다.  
그러므로 en/dequeue가 활발히 일어날 경우 배열의 크기는 커지지만 앞부분의 원소는 다 비어있는 상태가 된다.  
이런 비효율을 막기 위해 나온것이 배열의 처음과 끝을 연결하는 원형큐 이다.  
front, rear 라는 변수로 인덱스를 저장해두고, 거기에 배열 사이즈를 % 연산을 해서 원형으로 데이터를 저장 가능하다.  
enqueue를 하게되면 rear 값을 증가시킨 뒤 데이터를 넣게되고,  
dequeue를 하게되면 front 인덱스의 데이터를 삭제하고 front 값을 증가시킨다.  

근데 이런식으로 하게되면 배열의 모든값을 다 비웠을 때 front의 값이 0번째가 아니라 1번째를 가리키게 되는 문제가 발생한다.  
이를 방지하기 위해 원형큐의 경우 하나의 공간을 더 주고, front가 그곳을 가리키게 한다.  
원래라면 front 인덱스의 데이터를 삭제하고 front를 증가시키던 행위를 front를 증가하고 front 인덱스의 데이터를 삭제하는 순서로 변경한 것이다.  
즉 배열의 0번째는 데이터를 담지 않게 되는 것이다.  
이렇게 할 경우 front==rear 일 경우 배열이 비어있다는 것을 의미하게 된다.  

## Dequeue
양쪽에서 접근 가능한 스택이라고 생각하면 된다.  
0번 인덱스인 front, rear를 가지고  
front에서 더하면 왼쪽으로 늘어나고, rear에서 더하면 오른쪽으로 늘어난다.  
자신이 넣은 방향에서 직접 뺼수 있고(스택), 자신의 반대 방향에서도 뺄 수 있다(큐)  
스택과 큐의 장점을 다 가져왔다고 생각하면 된다.  
대표적으로 스크롤이 데크 방식이다.  

## Tree
자식을 계속 가지는 구조를 말함.  
대표적으로는 이진 트리가 있다.  
root 노드를 최 상단으로 가지고, 계층형으로 자식 노드들을 가진다.  
맨 마지막 노드는 리프 노드라고 부른다.  
숫자를 가지고 이진 트리를 구성한다고 할 때,  
저장하는 숫자가 루트 노드보다 작으면 왼쪽, 크면 오른쪽에 생성한다.  

```java
@Data
class BinaryTree<T extends Comparable>{
    private T value;
    private BinaryTree<T> left;
    private BinaryTree<T> right;

    public BinaryTree(T initialValue) {
        this.value = initialValue;
    }

    public void insert(T value){
        if(value.compareTo(this.value) < 0){
            if(left == null){
                left = new BinaryTree<>(value);
            } else{
                left.insert(value);
            }
        } else{
            if(right == null){
                right = new BinaryTree<>(value);
            } else{
                right.insert(value);
            }
        }
    }

    public boolean search(T value){
        if(this.value.equals(value)){
            return true;
        }

        if(value.compareTo(this.value) < 0 && left != null){
            return left.search(value);
        }

        if(value.compareTo(this.value) > 0 && right != null){
            return right.search(value);
        }

        return false;
    }
}
```

## Map

## Set
중복이 없고 순서가 없는 데이터를 저장하는데 사용되는 자료구조이다.  
HashSet, TreeSet, LinkedHashSet이 있으며 저장방식은 Map과 동일하다. (HashSet은 HashMap, TreeSet은 TreeMap과 방식 동일)  
CocurrentHashSet은 없는데 Collections의 newSetFromMap 메서드에 ConcurrentHashMap 인스턴스를 넣어서 생성하는 방법이 있다.  

## java getClass(), equals(), hashCode()
getClass() 함수는 해당 클래스의 정보를 가진 Class 클래스를 들고 올 수 있는 메서드이다.  
해당 메서드를 실행한 클래스의 Class 클래스를 가져온다. 

```java
class A{
    void print(){
        sout(getClass());
    }
}

class B extends A{}

class Main{
    psvm{
        B b = new B();
        b.print(); // B 출력
    }
}
```

위와 같은 상황에서도 B를 출력한다. getClass()를 호출한 클래스는 B 이기 때문이다.  

equals는 오브젝트의 내용이 동일한지 비교할 때 사용하는 메서드이다.  
Object 클래스가 가지고 있으며, 기본적으로 클래스를 생성하면 오버라이드 해줘야 한다.  
오버라이드 하지 않아서 발생하는 문제는 매우 찾기 힘들다.  
메서드의 내용들을 다 비교해서 오브젝트의 동등성을 체크해주면 된다.  
equals의 구현에는 몇가지 규약이 있다.  
1. 비교 대상이 null일 경우 false 리턴
2. 같은 레퍼런스는 무조건 true를 출력
3. 대칭성을 맞춰야함. a.equals(b)가 true이면 b.equals(a)도 true여야 한다.  
> 이게 쉬워보이지만 맞추기 까다롭다.  
equals 메서드의 인자는 Object이므로 사용하는 함수에 맞춰 형변환을 해줘야 하는데, 형변환을 가능 여부를 체크할 때 instanceof 키워드를 많이 이용한다.  
```java
public boolean equals(Object obj){
    if(obj == null){
        return false;
    }

    if(obj == this){
        return true;
    }

    if(obj instanceof TestClass){
        TestClass tc = (TestClass)obj;
        return age == tc.age && name.equals(tc.age);
    }
}
```
이 상황에서 TestClass와 TestClass를 상속한 클래스를 생성한 뒤,  
TestClass의 equals를 실행하면 true가 나오는 사태가 발생한다(..)  
instanceof는 다형성을 체크하는 키워드이기 때문이다.  
이럴 경우에는 위에서 언급한 getClass() 메서드를 사용하면 정확하게 구현 가능하다.  

사실은 동등성을 체크하기 위해 equals외에 hashCode 메서드도 오버라이드 해줘야 한다.  
hashCode도 구현 규약이 몇가지 있다.  
1. equals와 hashCode는 동떨어질 수 없다.  
1. a.equals(b)가 true이면 반드시 두 오브젝트는 같은 해시값을 리턴해야 한다.  
1. a.equals(b)가 false일때 반드시 두 오브젝트가 다른 값을 리턴할 필요는 없지만, 가능하면 다른 값을 리턴하는 것이 좋다.  

자바는 여기저기서 hashCode를 사용하는 곳이 많은데, 대표적으로 Map이 있다.  
Map에는 우리가 key,value 형태로 값을 집어넣지만 실제로 Map은 key를 저장하지 않고 key의 hashCode를 키로 사용한다.  
즉 두 오브젝트가 동등한데 hashCode가 다를 경우, 동등한 오브젝트가 계속해서 Map에 중복되서 들어가게 될것이며,  
두 오브젝트가 다른데 hashCode가 동일할 경우, 백날천날 넣어봐야 똑같은 키의 벨류를 덮어쓰게 된다는 뜻이다.  

그리고 Collection은 이 해시값을 가지고 오브젝트를 나눠서 저장하므로(빠르게 접근하기 위함) 같은 오브젝트는 동일한 값, 다른 오브젝트는 다른 값 이라는 원칙을 꼭 지켜주는 것이 좋다.  