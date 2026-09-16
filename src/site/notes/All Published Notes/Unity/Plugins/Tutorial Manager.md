---
{"dg-publish":true,"permalink":"/all-published-notes/unity/plugins/tutorial-manager/"}
---

created: 2026-09-05
updated: 2026-09-05

link: https://github.com/tglGames-Plugins/tgl-tutorial-manager

# Tutorial Handler  
A simple Tutorial Handler. we can use it to define the type of tutorial we want to play on a canvas.    
The Tutorial Handler assumes you want to block user interactions except letting the user click a fixed RectTransform.    
The Tutorial Handler does not manage lined screens, it works for one single event, and we need to call it multiple times if you want to call it for multiple values.    
The flexiblity allows you to make a tutorial manager or tutorial system for your specific modules without expecting too much from the tutorial handler.  
  
## Limitations  
- If we have dynamic content, we may need to wait a few frames to use the service as dynamic data needs time to settle down   
  - For example, layout groups need one frame before their positions are finalized.  
  - The suggestion is that you add a call to `BlockScreenEvent` event so that while screen changes, we do not end up with user interaction during loading or screen changes.  
- in case of dynamic content, deletion of items needs to be managed by the user.  
  - If we have deleted the parent object when we had `TutorialDisplayType.Reparent` the child cannot be reparent by the handler after tutorial action is invoked, so user has to delete the reparented child themselves.  
- If we have a scrollable layout where we are trying to show data, the scroll capablity needs to be stopped by the user  
  - This is again true for scenario that the plugin only provides screen blackout functionality.  
  - If scroll functionality is not blocked, the screen continues to scroll inside the cutout area, unless you are using `TutorialDisplayType.Reparent`   
- If we have animations or items that need user to wait, we have to call block instructions  
  - Block instruction blocks user interaction, you can pass the alpha of the color, so you can have completely transparent blocker while animations are clearly seen.  
  
  
  
### How to Use  
To use this plugin, follow these steps:  
- Users will communicate to the system through `TutorialHandler.cs` file, you can change code to use 'MessageBus' if that helps.  
- call events in `TutorialHandler` class to use the tutorial system.  
- if you do not want to highlight a rect transform, call `BlockScreenEvent` event.  
- if you want to highlight a game object,   
    - get rect transforms of the gameobjects that you want to emphasis during the tutorial  
    - Pass the rect transforms to `ShowTutorialEvent` event  
- Once done, call `HideTutorialEvent` event  
- In case of canvas not updating smoothly, call `RefreshTutorialCanvasEvent` event to force refresh the tutorial canvas  
- In case you need to stop the tutorial temporarily, for popup or other reasons, you can use `InturreptTutorialEvent` event  
	- Using this means you are inturrepting current tutorial step but not changing the progress  
	- if the pop up is something like no internet, after user re-connects, you can remove the popup and resume the tutorial.
  





---

[^1]: 
[^2]: 

