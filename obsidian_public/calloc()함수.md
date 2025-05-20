`calloc()` 함수는 C 언어 표준 라이브러리에서 제공하는 **동적 메모리 할당(dynamic memory allocation)** 함수 중 하나입니다. `malloc()` 와 유사하게 프로그램 실행 중에 힙(heap) 영역에 메모리를 할당하지만, **할당된 메모리를 0으로 초기화** 한다는 중요한 차이점이 있습니다.

## 함수의 원형
`calloc()` 함수를 사용하기 위해서는 `<stdlib.h>` 헤더 파일을 포함해야 합니다.
```c
#include <stdlib.h>
void* calloc(size_t num, size_t size);
```
 - `size_t num` : 할당하고자 하는 요소(element)의 개수입니다.
 - `size_t size` : 각 요소의 크기를 바이트(byte) 단위로 지정합니다
 - `void*` (반환타입)
	 - 메모리 할당 및 초기화에 **성공**하면, 할당된 메모리 블록의 시작 주소를 가리키는 `void` 포인터를 반환합니다. 이 `void` 포인터는 사용하려는 실제 데이터 타입의 포인터로 **형 변환(casting)** 하여 사용해야 합니다.
	 - 메모리 할당에 **실패**하면 (예: 시스템에 가용 메모리가 부족하거나 `num * size` 연산 시 오버플로우가 발생하는 경우), `NULL` 포인터를 반환합니다.


#### 작동 방식
1.  `calloc(num, size)` 가 호출되면, 시스템은 힙 영역에서 `num * size` 바이트만큼의 연속된 메모리 공간을 찾습니다.
2.  만약`num` 또는 `size` 가 0이면, `calloc()` 의 동작은 구현에 따라 다를 수 있습니다. `NULL` 을 반환하거나 유효한 포인터(but 역참조할 수 없는)를 반환할 수 있으므로, 일반적으로 `num` 과 `size` 는 0보다 큰 값을 사용합니다.
3.  만약 `num * size` 연산 결과가 오버플로우를 일으킬 정도로 크다면, 할당은 실패하고 `NULL` 이 반환될 수 있습니다.
4.  사용 가능한 공간이 있다면, 해당 공간은 프로그램이 사용할 수 있도록 예약합니다.
5.  **가장 중요한 특징으로, 할당된 메모리 블록의 모든 비트(bit)를 0으로 초기화 합니다.**
6.  성공적으로 할당 및 초기화되면 해당 메모리 블록의 시작 주소를 반환하고, 실패하면 `NULL` 을 반환합니다.

#### 메모리 초기화
- **정수형 타입 ( `int`, `char` 등):** 0으로 초기화됩니다.
- **부동 소수점 타입 ( `float`, `double`):** 0.0으로 초기화됩니다.
- **포인터 타입:** NULL 포인터로 초기화됩니다.
- **구조체나 배열:** 모든 멤버 또는 요소가 각각의 타입에 맞게 재귀적으로 0으로 초기화됩니다.

`malloc()`을 사용한 후 `memset()`으로 초기화하는 것과 유사한 결과를 얻지만, `calloc()`은 이 두 단계를 한 번에 처리합니다.

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    int n = 5;  // 배열의 요소 개수
    int* arr;

    // n개이 int 요소를 저장할 공간을 할당하고 0으로 초기화
    arr = (int*)calloc(n, sizeof(int));

    // 할당 성공 여부 확인(매우 중요!)
    if (arr == NULL) {
        printf("메모리 할당 실패\n");
        return 1; // 오류를 나타내는 값으로 프로그램 종료
    }

    // 할당된 메모리 사용 (모든 요소가 0으로 초기화되어 있음)
    printf("calloc으로 할당된 배열 요소:\n");
    for (int i = 0; i < n; i++) {
        printf("arr[%d] = %d\n", i, arr[i]);  // 출력 : arr[0] = 0, arr[1] = 0, arr[2] = 0, arr[3] = 0, arr[4] = 0...
    }

    // 사용이 끝난 동적 메모리 해제
    free(arr);
    arr = NULL;  // 댕글링 포인터 방지를 위해 NULL로 설정

    return 0;


}


```

**구조체 배열을 위한 메모리 할당**
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int id;
    char name[50];
    float score;
} Student;

int main() {
    int num_students = 2;
    Student* students;

    // num_students 수만큼의 Student 구조체를 저장할 공간을 할당하고 0으로 초기화
    students = (Student*)calloc(num_students, sizeof(Student));

    // 할당 성공 여부 확인
    if (students == NULL) {
        printf("구조체 배열 메모리 할당 실패\n");
        return 1;
    }

    // 할당된 메모리 사용(모든 멤버가 0 또는 NULL로 초기화됨)
    for (int i = 0; i < num_students; i++) {
        printf("학생 %d:\n", i + 1);
        printf("ID: %d\n", students[i].id);           // 출력 : ID: 0
        printf("이름: %s\n", students[i].name);       // 출력 : 이름: (null)
        printf("점수: %.2f\n", students[i].score);    // 출력 : 점수: 0.00
        printf("-------------------------------\n");
    }

    // 필요하다면 여기에 실제 데이터를 할당
    students[0].id = 101;

    snprintf(students[0].name, sizeof(students[0].name), "홍길동");
    students[0].score = 95.5f;

    // 사용이 끝난 동적 메모리 해제
    free(students);
    students = NULL;

    return 0;
}

```

#### 메모리 해제`free()` 
`calloc()`으로 할당된 동적 메모리도 `malloc()`과 마찬가지로 사용이 끝나면 반드시 `free()` 함수를 사용하여 명시적으로 해제해야 합니다.
```c
free(ptr); // ptr은 calloc이 반환한 포인터
```


### `calloc()`과 `malloc()` 비교

|특징|`calloc(size_t num, size_t size)`|`malloc(size_t size)`|
|:--|:--|:--|
|**인자**|요소의 개수(`num`), 각 요소의 크기(`size`)|할당할 총 바이트 수(`size`)|
|**메모리 초기화**|**모든 비트를 0으로 초기화**|**초기화하지 않음** (쓰레기 값 포함 가능)|
|**총 할당 크기**|`num * size` 바이트|`size` 바이트|
|**주 용도**|배열 할당 및 0으로 초기화 필요 시, 구조체 배열 초기화 시|일반적인 동적 메모리 할당, 초기화 불필요 시|
|**오버플로우**|`num * size` 계산 시 오버플로우 가능성을 내부적으로 처리 시도|인자로 전달된 `size` 자체가 너무 클 경우 문제|
|**성능**|`malloc` + `memset`과 유사, 약간의 초기화 오버헤드 발생 가능|초기화가 없어 `calloc`보다 빠를 수 있음|
