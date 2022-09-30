# Budibase Archery Component Plugin

![Multiple Targets](./assets/range.png)

## Description

This component tracks archery scores as they happen.

Features:
- Add archers to game
- Record scores for each archer as they happen
- Progress to next archer
- Save the results on the game when complete

Credit for the implementation of the target component, which has been ported from react to svelte in this plugin:
https://github.com/carabus/arrow-tracker

Find out more about [Budibase](https://github.com/Budibase/budibase).

## Settings

- `Target Size`
   - Change the size of the target (px) 
- `Target Type`
  - Choose between Olympic and NFAA style targets
- `Arrow Radius`
  - Change the size of arrows rendered on the target (px)
- `Stroke Width`
  - Change the size of lines between score boundaries on the target (px)
- `On Save Results`
  - Define the action that will receive the results of the game

### Handling Results

The results array has the following content
```js
[
  {
    name: "archer1", 
    score: 21
  },
  {
    name: "archer2",
    score: 28
  }
]
```

This object can be referenced from an action with the following bindings:
- Handlebars:
```handlebars
{{ Results }}
```
- JavaScript
```js
return $("Results")
```

#### Example: Saving to text field in Budibase DB

Configure a table as follows:

![String Column](./assets/string-column.png)
![Results String Table](./assets/results-string.png)

Configure an `On Save Results` action as follows:

![Save Results String](./assets/save-results-string.png)


### Example: Saving to json field in Budibase DB

![JSON Column](./assets/json-column.png)
![Results JSON Table](./assets/results-json.png)

Configure an `On Save Results` action as follows:

![Save Results JSON](./assets/save-results-json.png)

With the following JavaScript function:

![Save Results JSON JS](./assets/save-results-json-js.png)

## Screenshots

### New Component

![New Component](./assets/new.png)


### Add Archers

![Add Archers](./assets/add-archers.png)

### Game In-Progress

![Game In-Progress](./assets/in-progress.png)

### NFAA Target

![NFAA Target](./assets/nfaa.png)

### Multiple Targets

Multiple targets can be added to an app to model an archery range

![Multiple Targets](./assets/range.png)


## Instructions

To build this plugin run the following in your Budibase CLI:
```
budi plugins --build
```

You can also re-build everytime you make a change to your plugin with the command:
```
budi plugins --watch
```

