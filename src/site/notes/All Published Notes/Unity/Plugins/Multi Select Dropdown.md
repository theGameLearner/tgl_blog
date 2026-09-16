---
{"dg-publish":true,"permalink":"/all-published-notes/unity/plugins/multi-select-dropdown/"}
---

created: 2026-08-15
updated: 2026-09-04

Link: https://github.com/tglGames-Plugins/tgl-multi-select-dropdown

A dropdown option which allows us to select multiple options.

Properties:
- OnValueChanged - Event with currently selected options
- CaptionImage - The top Image component which shows all currently selected images
- CaptionText - The top Text component which shows all currently selected Text
- Options - All available options to choose from (`List<MultiSelectDropdownOptionData>`)
- Template - Rect Transform of the template - (`MultiSelectDropdownOption optionPrefab`)

Public Methods:
- SetOptions()
- AddOptions()
- AddOption()
- SetValues() - Set the values and send the Notifications in 'onValueChanged'
- SetValuesWithoutNotify() - Set the values but do not make the callback
- ClearOptions()








---

[^1]: 
[^2]: 

