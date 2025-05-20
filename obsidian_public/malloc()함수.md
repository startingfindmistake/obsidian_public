`malloc()` 함수는 C언어 표준 라이브러리 **동적 메모리 할당(dynamic memory allocation)** 함수 입니다.

### 1. 함수의 원형
`malloc()`함수를 사용하기 위해서는 `<stdlib.h>` 헤더 파일을 포함해야 합니다.

```c
#include <stdlib.h>
void* malloc(size_t size);
```
- `malloc()` 함수는 단 하나의 인자를 받습니다.`size_t size` . 이 인자는 할당 받고자 하는 메모리의 크기를 **바이트(byte)단위** 로 지정합니다
- `size_t` 는 C 표준 라이브러리에서 크기를 나타내는 부호 없는 정수형 타입입니다.

- `void*` **반환 타입:** 
	-  메모리 할당에 성공하면, 할당된 메모리 블록의 시작 주소를 가리키는 `void` 포인터를 반환합니다.
	- `void` 포인터는 "타입이 없는 포인터"를 의미하며, 어떤 특정 데이터 타입도 가리키지 않습니다.
	  따라서 실제로 이 메모리를 사용하기 위해서는 원하는 데이터 타입의 포인터로 **형 변환(casting)** 을 해주어야 합니다.
	- 메모리 할당에 실패하면, `NULL` 포인터를 반환합니다.

### 2. 작동방식
1. `malloc(size)` 가 호출되면, 시스템은 힙 영역에서 `size` 바이트만큼의 연속된 메모리 공간을 찾습니다.
2. 사용 가능한 공간이 있다면, 해당 공간을 프로그램이 사용할 수 있도록 예약하고, 그 시작 주소를 반환합니다.
3. 사용 가능한 공간이 없다면, `NULL` 을 반환하여 할당 실패를 알립니다.


# `malloc()` <span style="background-color:#fff5b1">함수는 할당된 메모리 공간을 초기화하지 않습니다.</span>

즉, 할당된 메모리에는 이전에 다른 데이터가 저장되어 있었던 "쓰레기 값(garbage value)"이 그대로 남아 있을 수 있습니다.

 - 만약 0으로 초기화된 메모리가 필요하다면 [[calloc()함수]] 함수를 사용하는 것이 좋습니다.

**단일 정수형 변수를 위한 메모리 할당**
```c
#include <stdio,h>
#include <stdlib.h>

int main() {
	int *ptr;
	
	// int 타입 하나의 크기만큼 메모리 할당
	ptr = (int*)malloc(sizeof(int));

	// 할당 성공 여부 확인 (매우 중요!)
	if (ptr == NULL) {
		printf("메모리 할당 실패!\n");
		return 1;  // 오류를 나타내는 값으로 프로그램 종료
	}

	// 할당된 메모리 사용
	*ptr = 100;
	printf("할당된 메모리의 값: %d\n", *ptr);
	printf("할당된 메모리의 주소: %p\n", (void*)ptr);

	// 사용이 끝난 동적 메모리 해제
	free(ptr);
	ptr = NULL;    // 댕글링 포인터를 방지를 위해 NULL로 설정

	reutrn 0;
}
```

**정수형 배열을 위한 메모리 할당**
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
	int n = 5;   // 배열의 크기
	int *arr;

	// n개의 int 요소를 저장할 수 있는 크기만큼 메모리 할당
	arr = (int*)malloc(n * sizeof(int));

	if (arr == NULL) {
		printf("배열 메모리 할당 실패!\n");
		return 1;
	}

	// 할당된 메모리 사용 (배열처럼 접근)
	for (int i = 0; i < n; i++) {
		arr[i] = i * 10;
		printf("arr[%d] = %d\n", i, arr[i]);
	}

	// 사용이 끝난 동적 메모리 해제
	free(arr);
	arr = NULL;

	return 0;
}
```

## 메모리 해제 `free()` 
`malloc()` (또는 `calloc()` , `realloc()`)으로 할당된 동적 메모리는 반드시 `free()` 함수를 사용하여 명시적으로 해제 해야 한다.
```c
void free(void* ptr);
```
- `ptr` : `malloc()` ,`calloc()` ,`realloc()` 에 의해 반환된 포인터여야 한다.
- `free(NULL)` 은 아무 동작도 하지 않으므로 안전하다.


#### `malloc()` 와 다른 동적 할당 함수
- **`calloc(size_t num, size_t size)`**: `num`개의 요소를 각각 `size` 바이트만큼 할당하고, 할당된 모든 바이트를 0으로 초기화합니다.
- **`realloc(void* ptr, size_t new_size)`**: 이미 할당된 메모리 블록(`ptr`)의 크기를 `new_size`로 변경합니다. 기존 내용을 최대한 보존하려고 시도합니다.