# marking-picture

## Description
Vue component for applying marker as shapes on top of picture using canvas

## Example
<img src="https://raw.githubusercontent.com/vano20/marking-picture/main/src/assets/contoh%20toyota.png" />

## Slots
Accept several slots to add additional element on top of the created shapes.

`top, top-right, middle, bottom-right`

## Config
Config to setup component value
```json
{
  "bgUrl": "",
  "pointMargin": 20,
  "edit": true,
  "additional": {
    "top": {
      "x": 0,
      "y": 0
    },
    "right": {
      "x": 0,
      "y": 0
    }
  },
  "canvas": {
    "lineWidth": 3,
    "setLineDash": [10, 5],
    "strokeStyle": "black",
    "height": null,
    "width": null
  }
}
```
- **bgUrl**: value to set background image
- **pointMargin**: value to set margin to snap when a existing point meet with new line
- **edit**: value to allow drawing stroke on canvas
- **additional**: object that represent additional margin for specific slot
- **canvas**: object that represent canvas options
- **lineWidth**: set line width of canvas stroke
- **setLineDash**: set if stroked line on canvas are dashed or solid
- **strokeStyle**: set canvas stroke color
- **height**: set canvas height
- **width**: set canvas width
## Project setup
```
yarn install
```

### Compiles and hot-reloads for development
```
yarn serve
```

### Compiles and minifies for production
```
yarn build
```

### Lints and fixes files
```
yarn lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
