The Solar Flow Widget is a first try to give the possibility to have a Visualization of Current-Flows. It can be used for example in an HTML-Widget in ioBroker VIS.

![solFlowWidget](https://www.photovoltaikforum.com/core/attachment/244127-solar-widget-gif/)

The "Widget" is currently sperated in three Parts.

## CSS-Section
First Part is the Style Definitions. It is not completed yet and will later hold more Styles which then are used in the SVG-Section

## SVG-Section
The SVG Section holds the SVG-Elements used to generate the Flow-Visualization. It also is depending on the Icons given in the "used pictures"  Folder.

## Script-Section
The Script Section holds all Scripts needed to generate and run the Visualization.
The Definition Section within the Script, holds Variables to enable to change/configure the Widget at least a bit.

## Script-Start (onload)
When the Widget is loaded following Functions need to be called in given order:
* initSVG
* addAnimCircles
* getServerValues
* setInterval(getServerValues, 2000)

## Usage with ioBroker VIS
[see Wiki-Entry](https://github.com/derHaubi/solarwidget/wiki/usage-with-ioBroker-VIS)
