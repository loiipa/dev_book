---
title: Inter-process communication (IPC)
type: docs
---

## IPC란?

- 프로세스 간 통신.
- 프로세스들 사이에 서로 데이터를 주고받는 행위 또는 그에 대한 방법이나 경로.

## IPC의 종류

### shared memory (공유 메모리)

- 여러 프로세스에 동일한 메모리 블록에 대한 접근 권한이 부여되어 프로세스가 서로 통신할 수 있도록 공유 버퍼를 생성하는 것을 말함.
- 과다한 복사를 피하거나, 해당 프로그램 간 통신을 위해 고안됨.
![공유 메모리 시스템 그림](/dev_book/shared_memory.png)  

{{< tabs "shared memory" >}}
{{< tab "shared memory - process 1" >}}

```c++
#include <windows.h>
#include <conio.h>
#include <iostream>

int main()
{
    std::wstring memoryName = L"MySharedMemory";
    std::wstring message = L"Hello ipc!";

    // 공유 메모리 크기 정의
    const int sharedMemorySize = 1024; // 예: 1KB

    // 파일 매핑 객체 생성
    HANDLE hMapFile = CreateFileMapping
    (
        INVALID_HANDLE_VALUE,   // 파일 핸들 (INVALID_HANDLE_VALUE: 파일 없음)
        NULL,                   // 보안 속성 (기본값: NULL)
        PAGE_READWRITE,         // 메모리 보호 모드 (읽기/쓰기 가능)
        0,                      // 공유 메모리 크기 (상위 32비트) (하위 32비트(2^32)를 넘을 경우,해당 비트를 지정)
        sharedMemorySize,       // 공유 메모리 크기 (하위 32비트)
        memoryName.data()       // 공유 메모리 이름 (문자열, NULL 또는 고유한 이름)
    );      

    if (hMapFile == nullptr)
    {
        std::cout << "error num : " << GetLastError() << std::endl;
        return 1;
    }

    // 공유 메모리에 연결
    LPVOID pSharedMemory = MapViewOfFile
    (
        hMapFile,               // 파일 매핑 객체 핸들
        FILE_MAP_ALL_ACCESS,    // 메모리 접근 권한
        0,                      // 파일 오프셋의 상위 32비트
        0,                      // 파일 오프셋의 하위 32비트
        sharedMemorySize        // 매핑할 메모리 크기
    );

    if (pSharedMemory == nullptr)
    {
        // 오류 처리
        std::cout << "error num : " << GetLastError() << std::endl;
        CloseHandle(hMapFile);
        return 1;
    }

    // 공유 메모리 사용
    wcscpy(static_cast<wchar_t*>(pSharedMemory), message.data());
    std::wcout << static_cast<wchar_t*>(pSharedMemory) << std::endl;

    //// 키보드 입력값을 하나를 받음.
    _getch();

    // 공유 메모리 해제
    UnmapViewOfFile(pSharedMemory);
    CloseHandle(hMapFile);
}
```

{{< /tab >}}

{{< tab "shared memory - process 2" >}}

```c++
#include <windows.h>
#include <iostream>

int main()
{
    std::wstring memoryName = L"MySharedMemory";
    
    // 공유 메모리 크기 정의
    const int sharedMemorySize = 1024;
    
    HANDLE hMapFile = OpenFileMapping
    (
        FILE_MAP_ALL_ACCESS,        // 메모리 매핑 객체에 대한 엑세스 권한
        FALSE,                      // 핸들 상속 유무
        memoryName.data()           // 메모리 매핑 객체의 이름 지정
    );

    if (hMapFile == nullptr)
    {
        std::cout << "error num : " << GetLastError() << std::endl;
        return 1;
    }

    LPVOID pSharedMemory = MapViewOfFile
    (
        hMapFile,
        FILE_MAP_ALL_ACCESS,
        0,
        0,
        sharedMemorySize
    );

    if (pSharedMemory == nullptr)
    {
        std::cout << "error num : " << GetLastError() << std::endl;
        CloseHandle(hMapFile);
        return 1;
    }

    MessageBox(NULL, static_cast<wchar_t*>(pSharedMemory), TEXT("Process2"), MB_OK);
    UnmapViewOfFile(pSharedMemory);
    CloseHandle(hMapFile);
}
```

{{< /tab >}}
{{< /tabs >}}

### file (파일)

- 디스크에 저장된 데이터 또는 파일 서버에서 제공한 데이터를 말함.

{{< tabs "file" >}}
{{< tab "producer" >}}

```c++
#include <iostream>
#include <fstream>

int main() {
    const char* filename = "data.txt";

    // 파일을 쓰기 모드로 열기
    std::ofstream file;
    file.open(filename);

    if (!file.is_open()) {
        std::cout << "Failed to open the file for writing." << std::endl;
        return 1;
    }

    // 데이터를 파일에 쓰기
    const char* data = "Hello, this is data from the producer!";
    file << data;

    // 파일 닫기
    file.close();

    return 0;
}
```

{{< /tab >}}

{{< tab "consumer" >}}

```c++
#include <iostream>
#include <fstream>

int main() {
    const char* filename = "data.txt";

    // 파일을 읽기 모드로 열기
    std::ifstream file;
    file.open(filename);

    if (!file.is_open()) {
        std::cout << "Failed to open the file for reading." << std::endl;
        return 1;
    }

    // 파일에서 데이터 읽어오기
    std::string data;
    std::getline(file, data);

    // 읽어온 데이터 사용 (예시로 콘솔에 출력)
    std::cout << "Received data from the file: " << data << std::endl;

    // 파일 닫기
    file.close();

    return 0;
}
```

{{< /tab >}}
{{< /tabs >}}

### socket (소켓)

- 동일한 컴퓨터의 다른 프로세스나 네트워크의 다른 컴퓨터로 네트워크 인터페이스를 통해 전송하는 데이터
- TCP, UDP

{{< tabs "socket" >}}
{{< tab "server" >}}

```c++
#include <iostream>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    const int PORT = 8888;

    // 소켓 생성
    int serverSocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serverSocket == -1) {
        std::cerr << "Error creating socket." << std::endl;
        return 1;
    }

    // 주소 설정
    struct sockaddr_in serverAddress;
    serverAddress.sin_family = AF_INET;
    serverAddress.sin_addr.s_addr = INADDR_ANY;
    serverAddress.sin_port = htons(PORT);

    // 바인딩
    if (bind(serverSocket, (struct sockaddr*)&serverAddress, sizeof(serverAddress)) < 0) {
        std::cerr << "Error binding socket." << std::endl;
        return 1;
    }

    // 클라이언트 연결 대기
    listen(serverSocket, 1);
    std::cout << "Server is listening on port " << PORT << "..." << std::endl;

    // 클라이언트 연결 수락
    int clientSocket = accept(serverSocket, NULL, NULL);
    if (clientSocket < 0) {
        std::cerr << "Error accepting connection." << std::endl;
        return 1;
    }

    // 데이터 수신
    char buffer[1024] = {0};
    read(clientSocket, buffer, sizeof(buffer));

    // 수신한 데이터 출력
    std::cout << "Received data from client: " << buffer << std::endl;

    // 소켓 닫기
    close(clientSocket);
    close(serverSocket);

    return 0;
}
```

{{< /tab >}}

{{< tab "client" >}}

```c++
#include <iostream>
#include <cstring>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main() {
    const int PORT = 8888;
    const char* SERVER_IP = "127.0.0.1"; // localhost

    // 소켓 생성
    int clientSocket = socket(AF_INET, SOCK_STREAM, 0);
    if (clientSocket == -1) {
        std::cerr << "Error creating socket." << std::endl;
        return 1;
    }

    // 주소 설정
    struct sockaddr_in serverAddress;
    serverAddress.sin_family = AF_INET;
    serverAddress.sin_port = htons(PORT);
    inet_pton(AF_INET, SERVER_IP, &serverAddress.sin_addr);

    // 서버에 연결
    if (connect(clientSocket, (struct sockaddr*)&serverAddress, sizeof(serverAddress)) < 0) {
        std::cerr << "Error connecting to server." << std::endl;
        return 1;
    }

    // 데이터 전송
    const char* data = "Hello, this is data from the client!";
    send(clientSocket, data, strlen(data), 0);

    // 소켓 닫기
    close(clientSocket);

    return 0;
}
```

{{< /tab >}}
{{< /tabs >}}

### unamed pipe (익명 파이프)

- 프로세스 간에 FIFO 방식으로 읽히는 임시 공간인 파이프를 기반으로 데이터를 주고받음
- 단방향 방식의 읽기 전용, 쓰기 전용 파이프를 만들어서 작동하는 방식.

{{< tabs "unamed pipe" >}}
{{< tab "부모 프로세스" >}}

```c++
#include <windows.h>
#include <iostream>

int main() {
    HANDLE hReadPipe, hWritePipe;
    SECURITY_ATTRIBUTES saAttr;
    saAttr.nLength = sizeof(SECURITY_ATTRIBUTES);
    saAttr.bInheritHandle = TRUE;
    saAttr.lpSecurityDescriptor = NULL;

    // 파이프 생성
    if (!CreatePipe(&hReadPipe, &hWritePipe, &saAttr, 0)) {
        std::cerr << "Failed to create anonymous pipe." << std::endl;
        return 1;
    }

    // 자식 프로세스 생성
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    si.hStdError = hWritePipe; // 자식 프로세스의 표준 출력을 파이프에 연결
    si.hStdOutput = hWritePipe; // 자식 프로세스의 표준 에러를 파이프에 연결
    si.dwFlags |= STARTF_USESTDHANDLES; // STARTUPINFO 구조체에 핸들 정보를 전달

    if (!CreateProcess(NULL, "ChildProcess.exe", NULL, NULL, TRUE, 0, NULL, NULL, &si, &pi)) {
        std::cerr << "Failed to create child process." << std::endl;
        return 1;
    }

    // 파이프를 닫음 (부모 프로세스는 쓰기용 핸들이 필요 없음)
    CloseHandle(hWritePipe);

    // 데이터를 자식 프로세스에 보냄
    const char* data = "Hello, this is data from the parent process!";
    DWORD numWritten;
    WriteFile(hReadPipe, data, strlen(data), &numWritten, NULL);

    // 파이프를 닫음
    CloseHandle(hReadPipe);

    // 자식 프로세스 종료 대기
    WaitForSingleObject(pi.hProcess, INFINITE);

    // 자식 프로세스의 핸들을 닫음
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    return 0;
}
```

{{< /tab >}}

{{< tab "자식 프로세스" >}}

```c++
#include <windows.h>
#include <iostream>

int main() {
    HANDLE hStdin = GetStdHandle(STD_INPUT_HANDLE);
    if (hStdin == INVALID_HANDLE_VALUE) {
        std::cerr << "Failed to get standard input handle." << std::endl;
        return 1;
    }

    // 파이프로부터 데이터를 읽음
    char buffer[1024];
    DWORD numRead;
    if (!ReadFile(hStdin, buffer, sizeof(buffer), &numRead, NULL)) {
        std::cerr << "Failed to read from pipe." << std::endl;
        return 1;
    }

    // 읽은 데이터를 콘솔에 출력
    std::cout << "Received data from parent process: " << buffer << std::endl;

    return 0;
}
```

{{< /tab >}}
{{< /tabs >}}

### named pipe (명명된 파이프)

- 명명된 단방향 또는 이중 파이프를 말함.
- 클라이언트/서버 통신을 위한 별도의 파이프 제공함
- 여러 파이프를 동시에 사용할 수도 있음.

{{< tabs "named pipe" >}}
{{< tab "server" >}}

```c++
#include <windows.h>
#include <iostream>

int main() {
    const char* pipeName = "\\\\.\\pipe\\my_named_pipe";
    const int bufferSize = 1024;

    // Named Pipe 생성
    HANDLE hNamedPipe = CreateNamedPipe(pipeName, PIPE_ACCESS_DUPLEX, PIPE_TYPE_MESSAGE | PIPE_READMODE_MESSAGE | PIPE_WAIT,
                                        PIPE_UNLIMITED_INSTANCES, bufferSize, bufferSize, 0, NULL);

    if (hNamedPipe == INVALID_HANDLE_VALUE) {
        std::cerr << "Failed to create named pipe." << std::endl;
        return 1;
    }

    // 클라이언트의 연결 대기
    if (!ConnectNamedPipe(hNamedPipe, NULL)) {
        std::cerr << "Failed to connect to client." << std::endl;
        CloseHandle(hNamedPipe);
        return 1;
    }

    // 클라이언트로부터 데이터 수신
    char buffer[bufferSize];
    DWORD numRead;
    if (ReadFile(hNamedPipe, buffer, bufferSize, &numRead, NULL)) {
        std::cout << "Received data from client: " << buffer << std::endl;
    }
    else {
        std::cerr << "Failed to read data from client." << std::endl;
    }

    // Named Pipe 종료
    CloseHandle(hNamedPipe);

    return 0;
}
```

{{< /tab >}}

{{< tab "client" >}}

```c++
#include <windows.h>
#include <iostream>

int main() {
    const char* pipeName = "\\\\.\\pipe\\my_named_pipe";

    // Named Pipe에 연결
    HANDLE hNamedPipe = CreateFile(pipeName, GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, 0, NULL);
    if (hNamedPipe == INVALID_HANDLE_VALUE) {
        std::cerr << "Failed to connect to named pipe." << std::endl;
        return 1;
    }

    // 데이터 전송
    const char* dataToSend = "Hello, this is data from the client!";
    DWORD numWritten;
    if (WriteFile(hNamedPipe, dataToSend, strlen(dataToSend) + 1, &numWritten, NULL)) {
        std::cout << "Sent data to server: " << dataToSend << std::endl;
    }
    else {
        std::cerr << "Failed to send data to server." << std::endl;
    }

    // Named Pipe 종료
    CloseHandle(hNamedPipe);

    return 0;
}
```

{{< /tab >}}
{{< /tabs >}}

### Message Queue (메시지 큐)

- 메시지를 큐(queue) 데이터 구조 형태로 관리하는 것.
- 커널의 전역변수 형태 등 커널에서 전역적으로 관리됨.
- 다른 IPC 방식에 비해서 사용방법이 매우 직관적임.

{{< tabs "Message Queue" >}}
{{< tab "메시지 큐 생성" >}}

```c++
#include <windows.h>

int main() {
    // 메시지 큐 생성
    HANDLE hMsgQueue = CreateMsgQueue(NULL,  // 메시지 큐 이름 (NULL로 지정하여 기본 이름 사용)
                                     sizeof(MESSAGE), // 메시지 크기
                                     maxMessages,    // 메시지 큐에 보관될 최대 메시지 개수
                                     &securityAttributes); // 보안 속성 (NULL로 지정하여 기본값 사용)

    if (hMsgQueue == NULL) {
        // 메시지 큐 생성에 실패한 경우 오류 처리
        DWORD error = GetLastError();
        // 오류 처리 코드 작성
    }

    // 이후에 메시지 큐를 사용하여 데이터를 주고받을 수 있습니다.

    // 메시지 큐 닫기
    CloseMsgQueue(hMsgQueue);

    return 0;
}
```

{{< /tab >}}

{{< tab "메시지 전송" >}}

```c++
#include <windows.h>

// 메시지 구조체 정의 (임의로 예시로 사용)
struct MESSAGE {
    int type;
    char data[256];
};

int main() {
    HANDLE hMsgQueue = ...; // 이전에 생성한 메시지 큐 핸들

    // 메시지 구조체 생성
    MESSAGE messageToSend;
    messageToSend.type = 1;
    strcpy(messageToSend.data, "Hello, this is a message!");

    // 메시지 큐에 메시지 전송
    if (!MsgQueueSend(hMsgQueue, // 메시지 큐 핸들
                     &messageToSend, // 전송할 메시지의 주소
                     sizeof(messageToSend), // 메시지 크기
                     0, // 우선순위 (0으로 지정하여 기본값 사용)
                     0)) { // 타임아웃 (0으로 지정하여 무한 대기)
        // 메시지 전송에 실패한 경우 오류 처리
        DWORD error = GetLastError();
        // 오류 처리 코드 작성
    }

    return 0;
}
```

{{< /tab >}}

{{< tab "메시지 수신" >}}

```c++
#include <windows.h>

// 메시지 구조체 정의 (임의로 예시로 사용)
struct MESSAGE {
    int type;
    char data[256];
};

int main() {
    HANDLE hMsgQueue = ...; // 이전에 생성한 메시지 큐 핸들

    // 메시지 수신을 위한 버퍼 생성
    MESSAGE receivedMessage;

    // 메시지 큐로부터 메시지 수신
    DWORD flags; // 메시지 속성 (0으로 지정하여 기본값 사용)
    DWORD bytesRead; // 수신한 메시지의 크기를 저장할 변수
    if (!MsgQueueReceive(hMsgQueue, // 메시지 큐 핸들
                        &receivedMessage, // 수신할 메시지를 저장할 주소
                        sizeof(receivedMessage), // 메시지 크기
                        &bytesRead, // 수신한 메시지의 크기를 저장할 변수
                        0, // 타임아웃 (0으로 지정하여 무한 대기)
                        &flags)) { // 메시지 속성을 저장할 변수
        // 메시지 수신에 실패한 경우 오류 처리
        DWORD error = GetLastError();
        // 오류 처리 코드 작성
    }

    // 수신한 메시지 출력
    printf("Received message: Type: %d, Data: %s\n", receivedMessage.type, receivedMessage.data);

    return 0;
}
```

{{< /tab >}}
{{< /tabs >}}

{{< hint info >}}
**참고 자료**  
[프로세스 간 통신(wikipedia)](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4_%EA%B0%84_%ED%86%B5%EC%8B%A0)  
[컴퓨터 파일(wikipedia)](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%93%A8%ED%84%B0_%ED%8C%8C%EC%9D%BC)  
{{< /hint >}}
