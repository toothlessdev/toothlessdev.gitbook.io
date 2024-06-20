# 소켓 프로그래밍에서 사용되는 C 기본 함수들

## 파일 입출력

### fread()

파일에서 `size * nitems` 크기만큼 읽어 메모리 ptr 에 저장합니다\
성공시 실제 읽은 데이터의 개수를 리턴하고, 실패시 -1 을 리턴합니다

```c
size_t fread(void *ptr, size_t size, size_t nitems, FILE *stream);
```

### fwrite()

파일에 버퍼 ptr 의 내용을 저장합니다\
성공시 실제 저장한 데이터의 개수를 리턴하고, 실패시 -1  을 리턴합니다

```c
size_t fwirte(const void *ptr, size_t size, size_t nitems, FILE *stream);
```

### fread, fwrite 예제

```c
#include <stdio.h>
int main() {
    FILE *fin, *fout;
    size_t ret;
    char rbuf[MAX_BUF_SIZE] = {0};
    char wbuf[] = "Hello Fread, Fwrite Function Test";
    
    fout = fopen("test.txt","w");
    if(fout) {
        ret = frwite(wbuf, sizeof(char), sizeof(wbuf), fout);
        printf("fwrite returned %d bytes\n", (int)ret);
        fclose(fout);
    }
    
    fin = fopen("test.txt", "r");
    if(fin) {
        ret = fread(rbuf, sizeof(char), MAX_BUF_SIZE-1, fin);
        printf("fread returned %d bytes\n", (int)ret);
        printf("%s\n", rbuf);
        fclose(fin);
    }
    return 0;
}
```

