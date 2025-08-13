
# OHD Radial Menu System



## System Architecture

### 1. Radial Menu Component
**Attached to:** Player Controller  
**Purpose:** Displays/hides a radial menu widget and triggers logic for selected buttons.

**Key Features:**
- Shows radial menu on request (e.g., pressing `T`, interacting on an object, etc).
- Maps **Radial Buttons** to **Radial Triggers** from a **Radial Button Data Table**.
- When a radial trigger class is called and a trigger instance of that class does not exist, an instance is created and stored in a **Radial Trigger Map** keyed by its class.
- Handles  **Radial Trigger** logic and as a bridge to RPC between Client and server  **Radial Trigger** instances
- Handles distance checking for interactive world objects.

---

### 2. Radial Triggers
**Purpose:** Handle the gameplay logic when a radial button is activated. 
**Types:**
1. **Single**
   - Calls `StartTrigger` logic once and finishes.
2. **ClickTickRelease**
   - Calls `StartTrigger` once, then executes `TickLogic` at a set tick rate.
   - Calls `FinishTrigger` when left-clicking; calls `CancelTrigger` on right-click (and other interuptors E.G. getting in/out a vehicle).
3. **DynamicSubnavigator**
   - Similar to **Single** but doesn't hide Radial Menu when called. Meant for dynamic in radial menu logic such as a SubNavigator that changes its target radial menu based on context  
   </ol>

 **Server/Client Implementation:**
- RPC calls are done by using replicated **Radial Menu Component** as a bridge. 
- Calling one of the RPC functions sends an RPC call with the given input to the Server/Owning Client, if no **Radial Trigger** instance exist of the **Radial Trigger** class of the instigating radial trigger, an instance will be created on the Server/Owning client and put into a map as normal. Then an event aligning with the RPC call of your choice will be called.
- Multiple RPC's are defined with different parameter configurations for mod-ability and different functions
  </ol>
  
 **Radial Button States:**

1. **Enabled**
	- Button is Opaque and can be activated.
2. **Disabled**
	- Button is translucent and can't be activated
3. **Hidden**
	- Button is invisible and can't be activated </ol>
The function **Set Radial Buttons State** in the **Radial Trigger** is called on each button whenever a new radial menu is displayed and can be used to determine the conditions which determine each **Radial Buttons** state. 



 **Implemented Radial Triggers**
1. **GetConstructionTool**
	**Purpose:** Equip Construction Tool from inventory
	**State:** Hidden if not Squad Leader, Disabled if kit does not have a construction tool, otherwise Enabled.
2. **GetInVehicle (Needs a Rename)**
	**Purpose:** Enters the vehicle owning the radial menu if not in a seat. Exits the vehicle if already seated
	**State:** Always enabled.
3. **Build**
	**Purpose:** Switches the equipped Constructions tools build to a class of building inputed in the Class Parameter.
	**State:** Always enabled.
4. **Navigator_Attachment**
	**Purpose:** Navigates the currently displayed radial menu to the radial menu found in the Interactive_Attachment component found onf the current weapon.
	**State:** Disabled if no  Interactive_Attachment component was found on the current weapon. Enabled otherwise.
4. **Navigator_Seat**
	**Purpose:** Navigates the currently displayed radial menu to the radial menu that is mapped to the seat the player is occupying.
	**State:** Disabled if the seat is not mapped with a radial menu. Hidden while not in a vehicle. Enabled otherwise.
4. **Resupply**
	**Purpose:** Changes the kit of the player to the kit referenced in the Data Parameter.
	**State:** Always enabled.



 
---
**Radial Menu Data Structs**
```plaintext
RadialMenuInfo
├─ HasCoreButton : bool
├─ CoreButton : RadialButtonStruct
└─ External Populator : DataTable (list of RadialButtonStruct)
    ├─ [0] : RadialButton
    ├─ [1] : RadialButton
    │    └─ ...
    └─ [n] : RadialButton
    
RadialButton
├─ Core Title : Name
├─ Core Description : Name
├─ Button Icon : Texture 2D
├─ Pipper Color : Linear Color
├─ Icon Tint : Linear Color
├─ IconSizeX : Integer
├─ IconSizeY : Integer
├─ Button Interactivity : ENUM(On Select, On Hold, On Hover)
├─ Button Action : ENUM(Action, Subnavigate, None)
├─ Radial Trigger : Radial Trigger Class
├─ Radial Trigger Data Parameter : Soft Object
├─ Radial Trigger Class Paramter : Soft Class
├─ Action Index : Integer
├─ Sub Menu (if Subnavigate) : Data Table Row Handle
    └─ Radial Menu Info (Data Table Row Handle)
├─ Activation Time : Float
└─ Slot Occupancy : Integer
  ```
### 3. Radial Menu Widget
**Purpose:** Visually represents the menu and populates it from a Target **Radial Menu Info Struct**.

**Key Features:**
- Handles Radial Menu UI using Materials
- Menu is Populated using data from data table data, populating its menu using **Radial Button Structs** as data.
- Widgets and Materials instances are created only when needed, recycling materials and widgets when able.
- **Radial Core** widget is used as a sub widget to handle displaying the Core description text and Icon's for **Core Buttons**.
- Initiates **Radial Trigger** Logic when activating a button. Removes menu on Right Mouse Button down
---

### 4. Radial Buttons
**Radial Trigger**
- **Radial Trigger** Class determines which class of radial trigger will be called.
- **Radial Trigger Parameters** (Data, Class, and Action Index) are parameters called to the radial trigger on activation. User can use these as parameters for radial trigger logic (E.G. which building to build, which kit class to change, etc.).


**Interactivity States:**
1. **Action** – Calls `StartTrigger` on its referenced Radial Trigger Class.
2. **Subnavigate** – Loads a new Radial Menu (Radial Menu Info Struct entry). No Radial Trigger called.
3. **None** – No action.

**Trigger Modes:**
- On Press (Left Mouse Button)
- On Hover (after `ActivationTime`)
- On Hold (after `ActivationTime`) (Left Mouse Button)

---

### 5. Interactive Component
**Purpose:** Attach to any world object to reference and display a radial menu.

**Features:**
- Can display assigned Radial Menu Info.
- Optional **Interact Widget** in-world with:
  - Distance-based visibility.
  - Player view tracking (expands when looked at).

**Setup for World Objects:**
- Call `SetInteractWidgets` in object construction.
- Provide:
  - Arrays of Widget Components (Interact Widget Class)
  - Mesh reference (for distance checking)
- Configure:
  - Visibility range
  - View tracking activation distance
  - Whether to initialize at start or on demand.
  - Message of widget
  - Input Action
  - etc.

---

### 6. Specialized Interaction Components
1. **AttachmentInteractible_Component**
   - For weapons with attachments (identified by triggers).
   - No additional logic.

2. **Interactible_Vehicle_Component**
   - Allows mapping of Radial Menu Info Structs (from Data Table Row Handles) to specific vehicle seats.

---

### 7. Player Controller Functions
- A **Base Radial Menu** is assigned to the player's kit.
- Displayed by pressing **T**.
- Right click to activate construction tool when equipped
- Press **F** near a item with **Interact** interface implemented to display its internal radial menu.


---

