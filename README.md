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


// Initializing imgui

void initImGui(IDirect3DDevice9* pDevice)
{
	D3DDEVICE_CREATION_PARAMETERS CP; // most of it is self explanatory
	pDevice->GetCreationParameters(&CP);
	HWND window = CP.hFocusWindow;
	ImGui::CreateContext();
	ImGuiIO& io = ImGui::GetIO(); (void)io;
	io.IniFilename = NULL;
	io.ConfigFlags |= ImGuiConfigFlags_NavEnableKeyboard;
	io.Fonts->AddFontDefault();

	ImGui_ImplWin32_Init(window);
	ImGui_ImplDX9_Init(pDevice);
	ImGui::StyleColorsDark();
	initialized = true;
	return;
}

// Hooking the endscene

void hookEndScene()
{
	IDirect3D9* pD3D = Direct3DCreate9(D3D_SDK_VERSION);
	if (!pD3D)
	{
		return;
	}

	D3DPRESENT_PARAMETERS d3dparams = { 0 };
	d3dparams.SwapEffect = D3DSWAPEFFECT_DISCARD;
	d3dparams.hDeviceWindow = GetForegroundWindow();
	d3dparams.Windowed = true;
	IDirect3DDevice9* pDevice = nullptr;
                                      //the adapter wihch is the primary display adaptor aka the graphics card 
                                      //the device type - > hardware to process graphics
                                      //pass the handle to a window which alerts the dx3d when the application switches from fg mode to bg mode
                                      //the behavoir flags 
                                      // ptr to presentation parameteres, the one that matters is if the gamewindow is .Windowed or not
                                      //d3d9 device
	HRESULT result = pD3D->CreateDevice(D3DADAPTER_DEFAULT, D3DDEVTYPE_HAL, GetForegroundWindow(), D3DCREATE_SOFTWARE_VERTEXPROCESSING,&d3dparams,&pDevice );

	if (FAILED(result) || !pDevice)
	{
		pD3D->Release();
		return;
	}

	void** vTable = *reinterpret_cast<void***>(pDevice);

	//detour the endscene function
	pEndScene = (endScene)DetourFunction((PBYTE)vTable[42], (PBYTE)hookedEndScene);

	pDevice->Release();
	pD3D->Release();
	
	
}

// Drawings on the endscene using our IMGUI modern GUI
HRESULT __stdcall hookedEndScene(IDirect3DDevice9* pDevice)
{
	if (!initialized) initImGui(pDevice);
	else {
  
		ininventory.drawInInventory(pDevice);
  
    /* Here you can draw anything and it will appear in game */
	


		
	}
	return pEndScene(pDevice);
}


```

```Lua

local a = 1;
for k, v in pairs(table) do
	local b = 12 
	
end

```
