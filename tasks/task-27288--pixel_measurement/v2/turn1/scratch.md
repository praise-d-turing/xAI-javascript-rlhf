Create a pixel measurement widget for me. These are the features I want:
- Allows getting the exact size of a portion of the screen using a rectangular measurement tool. The rectangular size selector should be resizable to allow for adjusting the measuring area.
- The measurement box should have handles at each edge and corner, including at mid points. In total, it should have the following handles: top-left, top-center, top-right, left-center, right-center, bottom-left, bottom-center, and bottom-right.
- Allows selecting (clicking) elements on the screen and get their exact dimensions. The measurement tool should also take the exact dimensions and position of the selected element.
- Display the full-screen dimensions at a corner of the screen. The dimensions should become invisible if I hover over it so that I can also perform measurements on any elements that may be underneath it.
- Display lines/arrows that extend from the center of each side of a measurement object (area/element) to the edges of the screen. That means that if a measurement box is drawn, I should see an arrow on each side of the rectangle, with its origin on the center of the side, extending to the edge of the screen. The length of those lines – which represents the distance from each side of the measurement object to the edge of the screen – should also be displayed.
- Display the correct directional resize cursor when a user hovers over a box-handle of the measurement tool.
- Also, add elements to the page with varying sizes that the user can use to play around with the measuring tool. Perhaps a simple website layout with a navbar, hero section, sidebars, e.t.c, but with just layout boxes instead of having actual content.
- Give the app a clean and pretty UI.
- I should be able to turn on/off the entire measurement feature at the click of a button and use the web-app normally. After the measurement widget has been disabled, everything relating to it disappears from the screens. The associated event listeners also be removed.

# Model A
## Instruction Following - 2
Places dummy layout content on a page to allow the user test out the measurement tool and adds the "Toggle measurement" button, but has the following issues:
- There are no arrows displaying the distance between each edge of the measurement area and the screen.
- The size of the measurement area is not displayed.
- The size of the screen is also not displayed.

## Accuracy - 2
- The measurement box can't be resized.
- The measurement box follows the cursor around.
- There are drag handles on only two points of the measurement box.

## Explanation - 5
No issues with the explanation. The response is free from grammatical errors, well punctuated, and uses proper markdown formatting syntax. The user is also informed about incompletely implemented features in the app.


# Model B
## Instruction - 2
Flaws:
- No free-form rectangular measurement
- Doesn't display the distance between the edges of the measurement areas and the edges of the screen.
- The dimensions of the measured area are not displayed.

## Accuracy - 2
- No free-form rectangular measurement.
- The measurement area cannot be resized from a drag handle.
- The drag handles are overlapping, making it appear that only two drag handles are present in the measurement area.

## Explanation

# Ideal
Here is a plan for creating the requested pixel measurement widget:

**HTML**
- A container for the widget UI
- Elements for displaying dimensions and distances
- Overlay elements for selecting and measuring areas/elements
- Full-screen dimension display that hides on hover
**CSS**
- Styles to make the widget draggable and resizable
- Styles for hiding and showing elements
- Visual styles for arrows and dimension displays
**JavaScript**
- Event listeners for drag, resize, hover, and click events
- Functions to calculate dimensions of elements or selected areas
- Functions to draw arrows from the center of each side of the selected area to the screen edges
- Update display functions for dimensions and distances
**Libraries**
- Use jQuery from a CDN for simplifying DOM manipulation and event handling
Below is the actual code:

