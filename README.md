# Code Snippets

### C++ Reverse Engineering for better graphics


``` C++

#include <iostream> // for conoutput
#include <Windows.h> // all will be done using windows api

#include <d3dx9.h> // dx9 for drawings and so on
#include <d3d9.h> 
#include "UI/Menu.h" //a new menu header for organized code 
#include "Detours/detours.h" // to hook dx9
#include "ImGui/imgui.h"  // imgui for modern graphics
#include "ImGui/imgui_impl_dx9.h"
#include "ImGui/imgui_impl_win32.h"

#pragma comment(lib, "d3d9.lib") // to not use the visual studio properties
#pragma comment(lib, "d3dx9.lib")
#pragma comment(lib, "Detours/detours.lib")





```
