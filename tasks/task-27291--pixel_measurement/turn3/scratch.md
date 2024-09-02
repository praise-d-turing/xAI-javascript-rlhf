## Overview
To toggle the measurement feature on and off, we'll:
- Introduce buttons for turning on/off the measurement widget.
- Modify the JavaScript code to remove/attach all measurement associated event listeners.
- Hide the entire measurement widget depending on the toggle state.

## Steps
- First we'll need to introduce a button on the measurement widget itself because elements outside of it can't be interacted with while the measurement feature is active.

- We'll also need another copy of the button placed outside of the `#measurement-widget` element, in any other accessible part of your HTML code.  It **must** be outside the `#measurement-widget` element. For example you could place it before the `#measurement-widget` element.

- The `#measurement-widget` element needs to be initially hidden.

- The JavaScript code would need to be modified so that the event listeners which were previously attached on document load will only be set when the measurement functionality is activated. That means that the event listener setup code and executed functions will need to be consolidated into one setup function. The `mousedown` and `mouseup` listeners for `.box-handle` will also need to be converted from anonymous functions into a named functions.

### CSS 
Remove the inline styles on `#measurement-widget` and update its CSS declaration block to this:
```css
#measurement-widget {
    cursor: crosshair;
    user-select: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: none;
}
```

### HTML
Here is the full HTML code with the earlier mentioned changes effected. This should be the only HTML code in the `body` element, aside from the `script` tags:
```html
<button style="margin: 4em 18em;">Click on any element to get its measurements</button>
<button class="toggle-measurement-widget">Enable Measuring Tool</button>
<div style="position: absolute; top: 150px; left: 50px; width: 200px; height: 100px; background: #ffcc00; text-align: center; line-height: 100px; font-size: 20px;">Box 1</div>
<div style="position: absolute; top: 250px; right: 100px; width: 150px; height: 300px; background: #66ccff; border-radius: 15px; display: flex; justify-content: center; align-items: center; font-size: 18px;">Box 2</div>
<div style="margin: 10px auto; width: 80%; height: 50px; background: #90ee90; line-height: 50px; text-align: center;">Wide Box</div>
<img src="https://via.placeholder.com/300x200" alt="Sample Image" style="position: absolute; bottom: 50px; left: 50%; transform: translateX(-50%);">

<div id="measurement-widget">
    <button class="toggle-measurement-widget">Disable Measuring Tool</button>

    <div id="full-dimensions"></div>
    <div id="measurement-values"></div>

    <div class="measurement-container">
        <div class="box-handle top-left-handle" data-resize-axis="x,y" data-handle-direction="top,left"></div>
        <div class="box-handle top-center-handle" data-resize-axis="y" data-handle-direction="top"></div>
        <div class="box-handle top-right-handle" data-resize-axis="x,y" data-handle-direction="top,right"></div>
        <div class="box-handle middle-left-handle" data-resize-axis="x" data-handle-direction="left"></div>
        <div class="box-handle middle-right-handle" data-resize-axis="x" data-handle-direction="right"></div>
        <div class="box-handle bottom-left-handle" data-resize-axis="x,y" data-handle-direction="bottom,left"></div>
        <div class="box-handle bottom-center-handle" data-resize-axis="y" data-handle-direction="bottom"></div>
        <div class="box-handle bottom-right-handle" data-resize-axis="x,y" data-handle-direction="bottom,right"></div>
    </div>
    <div class="arrow" id="top-arrow">
        <div class="arrow-inner has-arrow-head arrow_head--vertical"><span class="arrow-line-width"></span></div>
    </div>
    <div class="arrow" id="right-arrow">
        <div class="arrow-inner has-arrow-head arrow_head--horizontal"><span class="arrow-line-width"></span></div>
    </div>
    <div class="arrow" id="bottom-arrow">
        <div class="arrow-inner has-arrow-head arrow_head--vertical"><span class="arrow-line-width"></span></div>
    </div>
    <div class="arrow" id="left-arrow">
        <div class="arrow-inner has-arrow-head arrow_head--horizontal"><span class="arrow-line-width"></span></div>
    </div>
</div>
```

### JavaScript
Here is the full JavaScript code with all changes effected:
```js
$(document).ready(function () {
    const $fullDimensions = $('#full-dimensions');
    const $measurementContainer = $('.measurement-container');
    const $arrows = $('.arrow');
    const measureElementBlockers = new Set;
    // Drag-related variables
    let startX, startY, endX, endY, isDragging = false;
    let widgetActivated = false;
    let boxHandleData = {
        dragStartX: null,
        dragStartY: null,
        dragEndX: null,
        dragEndY: null,
        resizeX: false,
        resizeY: false,
        initialWidth: 0,
        initialHeight: 0,
        initialLeft: 0,
        initialTop: 0,
        handleDirection: '', // Store the handle direction (e.g., 'left', 'right', 'top', 'bottom')
    };

    $('.toggle-measurement-widget').click(toggleMeasurementWidget);

    // All the logic related to setting up the widget has been moved here
    function enableMeasurementWidget() {
        console.log('on')
        updateFullDimensions();

        $(document).on('mousedown', startDrag)
            .on('mousemove', updateDrag)
            .on('mouseup', stopDrag)
            .on('click', measureElement)

        $('.box-handle')
            .on('mousedown', handleBoxHandleMouseDown)
            .on('mouseup', handleBoxHandleMouseUp);

        $('#measurement-widget').show();
        widgetActivated = true;
    }

    function disableMeasurementWidget() {
        console.log('off')
        $(document).off('mousedown', startDrag)
            .off('mousemove', updateDrag)
            .off('mouseup', stopDrag)
            .off('click', measureElement)

        $('.box-handle')
            .off('mousedown', handleBoxHandleMouseDown)
            .off('mouseup', handleBoxHandleMouseUp);

        $('#measurement-widget').hide();
        widgetActivated = false;
    }

    function toggleMeasurementWidget() {
        console.log('toggling')
        if (widgetActivated) {
            disableMeasurementWidget();
        } else {
            enableMeasurementWidget();
        }
    }

    function updateFullDimensions() {
        $fullDimensions.text(`Screen: ${window.innerWidth}x${window.innerHeight}`);
    }

    function startDrag(event) {
        if (isInteractableWidget(event))
            return;
        startX = event.clientX;
        startY = event.clientY;
        isDragging = true;
    }

    function stopDrag(e) {
        e.preventDefault();
        isDragging = false;
        // Restore the onclick handler that was removed in `updateDrag`
        setTimeout(() => {
            measureElementBlockers.delete(Symbol.for('creating_selection_area'))
        });
    }

    function updateDrag(event) {
        if (isDragging) {
            endX = event.clientX;
            endY = event.clientY;
            updateMeasurement();
            // A click event (which will trigger `measureElement`) will be triggered
            // after the dragging ends due to the `mousedown` -> `mouseup` sequence.
            // This prevents that from happening.
            measureElementBlockers.add(Symbol.for('creating_selection_area'));
        }
    }

    function updateMeasurement() {
        const width = Math.abs(endX - startX);
        const height = Math.abs(endY - startY);
        const posX = Math.min(startX, endX);
        const posY = Math.min(startY, endY);

        $measurementContainer.css({
            top: `${posY}px`,
            left: `${posX}px`,
            width: `${width}px`,
            height: `${height}px`
        });

        displayMeasurementAreaDimensions();

        updateArrows();
    }

    function displayMeasurementAreaDimensions() {
        const measurementContainerBBox = $measurementContainer[0].getBoundingClientRect();
        $('#measurement-values').html(`
            <b>w</b>: ${$measurementContainer.width()}px<br/>
            <b>h</b>: ${$measurementContainer.height()}px<br/>
        `);
    }

    function updateArrows() {
        const positionArrow = ($arrow, position) => {
            const measurementContainerBBox = $measurementContainer[0].getBoundingClientRect();
            let axisLength;
            switch (position) {
                case 'top': {
                    axisLength = measurementContainerBBox.top;
                    $arrow.css({
                        left: `${measurementContainerBBox.left + measurementContainerBBox.width * 0.5}px`,
                        height: `${axisLength}px`,
                        top: 0,
                    });
                    break;
                }
                case 'bottom': {
                    axisLength = window.innerHeight - measurementContainerBBox.bottom;
                    $arrow.css({
                        left: `${measurementContainerBBox.left + measurementContainerBBox.width * 0.5}px`,
                        height: `${axisLength}px`,
                        bottom: 0,
                    });
                    break;
                }
                case 'left': {
                    axisLength = measurementContainerBBox.left;
                    $arrow.css({
                        top: `${measurementContainerBBox.top + measurementContainerBBox.height * 0.5}px`,
                        width: `${axisLength}px`,
                        left: 0,
                    });
                    break;
                }
                case 'right': {
                    axisLength = window.innerWidth - measurementContainerBBox.right
                    $arrow.css({
                        top: `${measurementContainerBBox.top + measurementContainerBBox.height * 0.5}px`,
                        width: `${axisLength}px`,
                        right: 0,
                    });
                    break;
                }
            }
            $arrow.find('.arrow-line-width').text(`${axisLength}px`);
        };

        $('.arrow').each((i, el) => {
            positionArrow($(el), el.id.split('-')[0])
        })
    }

    /**
     * Determines if an event occurred on an interactible widget
     * @param {Event} The event object
     */
    function isInteractableWidget(e) {
        const interactableWidgetSelectors = [
            '.box-handle',
            '.toggle-measurement-widget',
        ];
        return !!interactableWidgetSelectors.find(s => e.target.closest(s));
    }

    function measureElement(e) {
        if (
            measureElementBlockers.size
            || isInteractableWidget(e)
            || ! e.target.closest('#measurement-widget')
        ) {
            return;
        }
        $('#measurement-widget').hide();
        {
            const el = document.elementFromPoint(e.clientX, e.clientY);
            if (!el)
                return;

            const elBBox = el.getBoundingClientRect();
            startX = elBBox.x;
            startY = elBBox.y;
            endX = elBBox.right;
            endY = elBBox.bottom;
        }
        $('#measurement-widget').show();
        updateMeasurement();
    };

    function dragBoxHandle(e) {
        boxHandleData.dragEndX = e.clientX;
        boxHandleData.dragEndY = e.clientY;

        const widthDelta = (boxHandleData.dragEndX - boxHandleData.dragStartX);
        const heightDelta = (boxHandleData.dragEndY - boxHandleData.dragStartY);

        // Handle horizontal resizing
        if (boxHandleData.resizeX) {
            if (boxHandleData.handleDirection.includes('left')) {
                const newWidth = boxHandleData.initialWidth - widthDelta;
                const newLeft = boxHandleData.initialLeft + widthDelta;
                if (newWidth > 0) {
                    $measurementContainer.css({
                        width: `${newWidth}px`,
                        left: `${newLeft}px`
                    });
                }
            } else if (boxHandleData.handleDirection.includes('right')) {
                const newWidth = boxHandleData.initialWidth + widthDelta;
                if (newWidth > 0) {
                    $measurementContainer.css({width: `${newWidth}px`});
                }
            }
        }

        // Handle vertical resizing
        if (boxHandleData.resizeY) {
            if (boxHandleData.handleDirection.includes('top')) {
                const newHeight = boxHandleData.initialHeight - heightDelta;
                const newTop = boxHandleData.initialTop + heightDelta;
                if (newHeight > 0) {
                    $measurementContainer.css({
                        height: `${newHeight}px`,
                        top: `${newTop}px`
                    });
                }
            } else if (boxHandleData.handleDirection.includes('bottom')) {
                const newHeight = boxHandleData.initialHeight + heightDelta;
                if (newHeight > 0) {
                    $measurementContainer.css({height: `${newHeight}px`});
                }
            }
        }
        updateArrows();
        displayMeasurementAreaDimensions();
    }

    function handleBoxHandleMouseDown(e) {
        measureElementBlockers.add(Symbol.for('box-handle-drag'));
        boxHandleData.dragStartX = e.clientX;
        boxHandleData.dragStartY = e.clientY;
        boxHandleData.initialWidth = $measurementContainer.width();
        boxHandleData.initialHeight = $measurementContainer.height();
        boxHandleData.initialLeft = $measurementContainer.offset().left;
        boxHandleData.initialTop = $measurementContainer.offset().top;

        const resizeAxis = e.target.dataset.resizeAxis.split(',');
        boxHandleData.resizeX = resizeAxis.includes('x');
        boxHandleData.resizeY = resizeAxis.includes('y');
        boxHandleData.handleDirection = e.target.dataset.handleDirection.split(','); // Get the handle direction

        $(document).mousemove(dragBoxHandle);
    }

    function handleBoxHandleMouseUp(e) {
        $(document).off('mousemove', dragBoxHandle);
        setTimeout(() => measureElementBlockers.delete(Symbol.for('box-handle-drag')));
    }
});
```

**Changes made:**

- The button which previously read "Click me to measure!" has been modified to read `"Click on any element to get its measurements"`. This is to avoid potential confusion between it and the toggle button.
- Two new buttons have been added to the DOM with the classname `.toggle-measurement-widget` for toggling the measurement widget.
- `.toggle-measurement-widget` has been added to the list of interactive widgets in the `isInteractableWidget` function. This allows the button to be interacted with normally.
- The inline styles on the `#measurement-widget` element have been transferred into the internal stylsheet.
- The JavaScript code has been modified such that the setup actions such as registering events and measuring the screen size are only called when the measurement widget is activated.


