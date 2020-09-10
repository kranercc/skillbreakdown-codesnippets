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


### Lua Game Automations

Illusion splitter to confuse enemies

```Lua

kraneIllusionSplitter = {}

kraneIllusionSplitter.Enabled = Menu.AddOption({"KRANE", "Illusions", "Illusion Splitter"}, "Illusion Splitter", "SUPPORTED HEROES:\n↓↓↓↓\nEVERYONE WITH AT LEAST MANTA STYLE")
kraneIllusionSplitter.Trigger = Menu.AddKeyOption({"KRANE", "Illusions", "Illusion Splitter"}, "Key bind", 0)
kraneIllusionSplitter.SplitUpEnabled = Menu.AddOption({"KRANE", "Illusions", "Illusion Splitter"}, "Split Up / Don\'t focuse main hero", "")

local everyBodyInRadius = nil
local myHero = nil
local myPosition = nil
local randomX = 0
local randomY = 0
local PossiblePosition = nil

function kraneIllusionSplitter.MainLoop()
    if Menu.IsEnabled(kraneIllusionSplitter.Enabled) then
        if Menu.IsKeyDown(kraneIllusionSplitter.Trigger) then

            myHero = Heroes.GetLocal()
            
            --get all the illusions
            everyBodyInRadius = Entity.GetHeroesInRadius(myHero, 1200, Enum.TeamType.TEAM_FRIEND, true)
            
            for k, v in pairs(everyBodyInRadius) do
                    --generate random number for every new entity 
                
                local asRandomAsItGets = os.clock()%1
                randomX = math.random(-600, 600)*asRandomAsItGets   
                randomY = math.random(-600, 600)*asRandomAsItGets
                if NPC.GetUnitName(v) == NPC.GetUnitName(Heroes.GetLocal()) then
                    if Menu.IsEnabled(kraneIllusionSplitter.SplitUpEnabled) then
                        myPosition = Entity.GetOrigin(v)
                    else
                        myPosition = Entity.GetOrigin(myHero)
                    end
                    PossiblePosition = Vector(myPosition:GetX()+randomX, myPosition:GetY()+randomY, myPosition:GetZ())
                    
                    if NPC.IsAttacking(v) == true or NPC.IsRunning(v) == false then
                        NPC.MoveTo(v, PossiblePosition, false, false)
                    end
                end     
            end

        end

    end

end

return kraneIllusionSplitter



```

Making use of the current items to use in the combo

```Lua

kraneItemsToUseInCombo = {}

kraneItemsToUseInCombo.EnableItemsToUsedInCombo = Menu.AddOption({"KRANE", "Items", "Items To Use In Combo"}, "Enable", "")


local ActiveItemsThatCanBeTargetedToEnemy = {
    "item_veil_of_discord",
    "item_dagon",
    "item_dagon_2",
    "item_dagon_3",
    "item_dagon_4",
    "item_dagon_5",
    "item_rod_of_atos",
    "item_solar_crest",
    "item_orchid",
    "item_nullifier",
    "item_sheepstick",
    "item_ethereal_blade",
    "item_urn_of_shadows",
    "item_spirit_vessel",
    "item_diffusal_blade",
    "item_heavens_halberd"

}


local ActiveItemsCastNoTargetBeforeEngaging =
{
    "item_necronomicon",
    "item_necronomicon_2",
    "item_necronomicon_3"
}


local ActiveItemsCastNoTargetIfInRange =
{
    "item_shivas_guard"

}

local ActiveItemsCastPosition =
{
    "item_veil_of_discord"

}


function kraneItemsToUseInCombo.MainLoop(comboKey, target)
    if Menu.GetValue(kraneItemsToUseInCombo.EnableItemsToUsedInCombo) == false then return end
    if target == nil then return end

    if Menu.IsKeyDown(comboKey) then
        
        for k,v in pairs(kraneUtilityFunctions.getAllItemsInInventory()) do 
           -- Console.Print(tostring())
            for k, itemName in pairs(ActiveItemsThatCanBeTargetedToEnemy) do
                if Ability.GetName(v) == itemName and kraneUtilityFunctions.getDistanceBetween2Vectors(Entity.GetAbsOrigin(Heroes.GetLocal()), Entity.GetAbsOrigin(target)) < Ability.GetCastRange(v) and krane_SpellHelper.IsOnCooldown(v) == false then
                    --use it on target
                    Ability.CastTarget(v, target, false)

                end
            end
    
            for k, itemName in pairs(ActiveItemsCastNoTargetBeforeEngaging) do
                if Ability.GetName(v) == itemName then
                    Ability.CastNoTarget(v, false)
                end
            end

            for k, itemName in pairs(ActiveItemsCastPosition) do
                if Ability.GetName(v) == itemName then
                
                    Ability.CastPosition(v, Entity.GetAbsOrigin(target), false)
                end
            end

            for k, itemName in pairs(ActiveItemsCastNoTargetIfInRange) do
                if Ability.GetName(v) == itemName then
                    if kraneUtilityFunctions.getDistanceBetween2Vectors(Entity.GetAbsOrigin(Heroes.GetLocal()), Entity.GetAbsOrigin(target)) < Ability.GetCastRange(v) and krane_SpellHelper.IsOnCooldown(v) == false then
                        Ability.CastNoTarget(v, false)
                    end
                end
            end

            
        
        end    
        

    end

end


return kraneItemsToUseInCombo

```
