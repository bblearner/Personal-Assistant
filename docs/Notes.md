# Learning Tailwind CSS


### max width the largest width a class name take.
margin - mx-8 horizontal margin

### bg and text for working on the background color and the text color

### heirachy of space
margin
border
padding

### Use the hover: for change appearence on hover. Use transition for making the stuff change with style.

### Buttons are not inline so margins don't push them. Use inline for making them inline. Use inline-block for that.

Rounded for rounded stuff

Use the [] for setting custom values

### Layout - flex and grid - put content on columns and rows.
Grid is more for when you want to control the size of boxes.
Flex is simpler if you just want to put stuff in a row or column

Flex decide the parent and then the children will be in the flex box.
Use gap for horinzontal and vertical spacing
Justify to like take all the space and stuff
Usr margins for particular stuff for taking more space.
Use max width for margins a mx mx-auto px and py
wrap if you want to put children in a new is the spacing is not there

### Responsive desings
use breakpoints to creating responsive designs. sm, md, lg, xl etc are breakpoints.
Anthing beyond the breakpoint size will the use the utility class that you defines.

Using grid grid-col and grid-row for how many celss you want and gap for spacing in cell.

Align items to align in centre left or whatever you want

if you wanna change the order either put them in that order or use the order property something like order-2

auto-fit - browser decides how many columns can fit and fit them
minmax(a,b) set the min and max size, useful with the autofit.

use the col-span-full to put how many col you want a child to get
m<> for setting the margin stuff

### hero section - top section big picture or promotion

using letter spacing and uppercase. 

bg-linear for gradient stuff

### custom styles
themes in tailwind

@theme { 
    //tailwind utility tailwind will auto detect this
    --color-<> for colors use oklch
    --font or fonts
}

@utility <name of the thing> {
    // custom utility for things tailwind won't detect in the theme
}


### nth-child 
for targetting chils in a group

### pseudo element
after: before: are examples creates a "pseudo element" god knows what that means.

### layer and components
base styles 
@layer <layer name> directive 
base component utility
example bold for everything heading

@layer base{
    h1, h2, h3 {
        font-weight: 700;
    }
}

@layer component {
    .button {
        @apply <everything you want reuse>
    }
}