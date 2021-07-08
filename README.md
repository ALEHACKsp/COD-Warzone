# Call Of Duty Warzone / Warface

##### ( Don't ask me how to use it. I think you're on GITHUB, you know where to put the source code in order to use it. )

 ##### 📌 Permanent Unlock Dark Matter/ Weapons/ Max Rank Source Code


#### With this release I'll show you how you can soft unlock the battlepass and every single microtranaction item in the game. That means every single weapon in the armory, every vehicle skin, operator skin, watch, decal, spray, taunt, reticle, calling card, all the special activision camos, and all the unreleased bundles and skins. And beacuse there aren't any checks for if you own the items you put on your account, once you reboot your game you keep everything you applied permanently 

#### You will need a hooking library like minhook or detours. You also need to hook the game after it runs some integrity checks on stayrtup so you don't crash immediately for hooking directly on the .text section. Just make sure you do this with a hook or the game does do a lazy check that will ban you for directly manipulating the loot data (Infinity Ward lazy idea for a "patch"). Hooking right after the intro screen when the game is connecting to online services should work good. Once you run the unlock function, you can just unhook directly after. You need two pretty known functions from the engine to work with the stringtables (StringTable_GetAsset, StringTable_GetColumnValueForRow) but thats all. You could get by without using those functions, but I'm going to use them in this example cause I don't want to post a stupidly long array again. Have fun, and enjoy. Again, I'll try to keep all offsets updated for guys


***

 
	void UnlockEverything() {
 
	auto pLootBase = base + 0x9F0C910;// signature 48 8D 0D ? ? ? ? 48 8D 44 24 ? C7 44 (LEA rcx, pLootBase)
 
	struct LootItem {
		int m_itemId;
		int m_itemQuantity;
	};
	
	struct StringTable {
		char* name;
		int columnCount;
		int rowCount;
	};
 
	auto pInventory = (LootItem*)((uintptr_t)pLootBase + 64);
 
	auto pNumItems = (uint32_t*)((uintptr_t)pLootBase + 240064);
 
	int curCount = *pNumItems;
 
	auto updateOrAddItem = [&] (int itemId, int quantity) {
 
		bool bFound = false;
 
		for (int i = 0; i < 30000; i++) {
			if (pInventory[i].itemId == itemId && pInventory[i].quantity < 1) {
				pInventory[i].quantity++;
				bFound = true;
				break;
			}
		}
 
		if (!bFound) {
			pInventory[curCount].itemId = itemId;
			pInventory[curCount].quantity = 1;
 
			curCount++;
			(*pNumItems)++;
 
			*(BYTE*)((uintptr_t)pLootBase + 240072) = 0;
		}
	};
 
	StringTable* loot_master = nullptr;
 
	StringTable_GetAsset("loot/loot_master.csv", &loot_master);
 
	for (int i = 1; i < loot_master->rowCount; i++) {
 
		char* loot_type = StringTable_GetColumnValueForRow(loot_master, i, 2);
 
		if (strstr(loot_type, "iw8_") || loot_type[0] == '#')
			continue;
			
		char buf[1024];
		
		sprintf_s(buf, "loot/%s_ids.csv", loot_type);
 
		StringTable * string_table = nullptr;
 
		StringTable_GetAsset(buf, &string_table);
 
		if (!string_table)
			continue;
 
		for (int s = 0; s < string_table->rowCount; s++) {
 
			updateOrAddItem(atoi(StringTable_GetColumnValueForRow(string_table, s, 0)), 1);
		}
	}
	}
 
	using MoveResponseToInventory_t = bool(__fastcall*)(LPVOID, int);
 
	MoveResponseToInventory_t fpMoveResponseOrig;
 
	pMoveResponseToInventory = 
 
	bool __fastcall MoveResponseToInventory_Hooked(LPVOID a1, int a2) {
 
	fpMoveResponseOrig(a1, a2);
 
	UnlockEverything();
 
	MH_RemoveHook(pMoveResponseToInventory);
 
	return false;
	}
 
	void SetupYourHook() {
	if (MH_CreateHook(pMoveResponseToInventory, MoveResponseToInventory_Hooked, &fpMoveResponseOrig) != MH_SUCCESS) {
	
		printf("Failed to hook...");
	}}
 
 
	Addresses (latest update):
 
	pMoveResponseToInventory: base + 0x576BB60 // signature 40 53 55 56 57 41 55 41 56 48 83 EC 28 4C
 
	pLootBase: base + 0xA0DC290 // signature 48 8D 0D ? ? ? ? 48 8D 44 24 ? C7 44
 
	StringTable_GetAsset: base + 0x2F56A70 // signature E8 ? ? ? ? 48 8D 15 ? ? ? ? 8D 4B 36
 
	StringTable_GetColumnValueForRow: base + 0x2F56AB0 // signature E8 ? ? ? ? 33 D2 48 8B C8 44 8D 42 16
***
