### **Lazy Text Tool: A Developer's High-Level Overview**

The Lazy Text Tool is a Krita extension designed to provide a more powerful and interactive text editing experience by running a custom UI as a transparent overlay on top of the Krita canvas. This allows it to offer rich text features and immediate visual feedback while still integrating the final output as native Krita vector objects.

#### **1. Core Architecture: The Overlay Canvas**

The tool's fundamental design is a **transparent Qt widget (`TextCanvas`)** that is placed directly over Krita's viewport. This canvas hosts its own `QGraphicsScene`, which is a Qt environment for managing 2D objects. All interactive text creation and editing happens within this scene. This architecture allows the tool to use the full power of Qt's text rendering and event handling, bypassing the limitations of Krita's native tool at the time. The overlay is designed to be seamless, forwarding events like zooming and panning to Krita's underlying canvas.

#### **2. Key Components & File Roles**

The codebase is primarily split into three files:

*   **`LazyTextTool.py` (Integration & Orchestration):** This is the main plugin entry point. The `LazyTextTool` class handles registering the tool with Krita, managing its activation state, and listening for Krita-wide events (like changing layers or documents). It's responsible for launching and destroying the `TextCanvas` overlay at the right moments.

*   **`LazyTextToolFunc.py` (Core Functionality & Logic):** This file is the engine.
    *   **`LazyTextUtils`:** A critical utility class that acts as the "translator" between the rich text in the editor (represented as HTML) and the SVG format that Krita uses for vector shapes.
    *   **`LazyTextScene`:** The "brain" of the editing session. It manages the current state (e.g., drawing, editing, panning) and handles all mouse events to create or select text objects.
    *   **`LazyTextObject` & `LazyTextEdit`:** The building blocks for the on-canvas text. `LazyTextObject` is the container with move/resize handles, and `LazyTextEdit` is the actual rich text editor widget.
    *   **`LazyTextHelper`:** The code that powers the floating UI palette, connecting its buttons and controls to the active `LazyTextEdit` item.

*   **`LazyTextToolHelper.ui` (UI Definition):** An XML file that defines the layout of the floating `LazyTextHelper` palette. It is loaded by the code in `LazyTextToolFunc.py` to create the UI widgets.

#### **3. The Editing Workflow in Action**

1.  **Creation:** A user clicks on the canvas to start typing ("Typewriter mode") or drags to create a text box ("Text Wrap mode"). This creates a `LazyTextObject` on the overlay canvas.
2.  **Live Editing:** The user types into the `LazyTextEdit` widget. The `LazyTextHelper` palette appears, and any formatting changes (font, color, alignment) are applied instantly, as it's a standard Qt rich text item.
3.  **Committing to Krita:** When the user clicks outside the text object, the editing session ends. The tool takes the **HTML content** from the `LazyTextEdit` widget, uses `LazyTextUtils` to convert it into an **SVG string**, creates a new vector layer in Krita, and inserts the SVG into it. The temporary `LazyTextObject` on the overlay is then destroyed.
4.  **Editing Existing Text:** A user double-clicks an existing text shape. The tool reads the **SVG data** from that Krita layer, uses `LazyTextUtils` to convert it back into **HTML**, and populates a new `LazyTextObject` on the overlay with that content. To avoid visual glitches, the original Krita layer is hidden during the edit.

#### **4. Important Developer Considerations & Limitations**

*   **Destructive Editing:** The tool uses a "replace" workflow. When you edit text, the original layer is **deleted** and a new one is created. This means any layer styles or masks applied directly to the text layer will be lost.
*   **Metadata in SVG:** The tool cleverly embeds its own metadata (document resolution, wrap mode, etc.) into the `id` attribute of the root `<svg>` tag. This is how it perfectly restores the text object's properties when you re-edit it.
*   **Single Object per Layer:** The tool is designed with the restriction that each text object lives on its own dedicated vector layer. The logic is not built to handle multiple shapes on a single layer.
*   **No Transform Support:** The `README` explicitly states that scaling and rotation were not implemented due to difficulties in reading transformation matrix data from Krita's API at the time.
*   **Legacy Clipboard Hack:** For Krita versions before 5.0, the tool relied on a fragile clipboard "paste" workaround to write the SVG into the layer. Newer versions use a much more stable, direct API call.
