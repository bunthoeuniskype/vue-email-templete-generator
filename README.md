# Reactive Email Template Builder
This is a nice little proof-of-concept email template builder. It is made up of two frameworks: the amazing [Vue.js](http://vuejs.org) `v2.1` and the Sass CSS library [Bulma](http://bulma.io).

![Demo](http://i.imgur.com/QFn0gnQ.png?raw=true)


## Installation
I’m assuming the machine running this to be UNIX based, and I’m assuming you have Node.js and npm installed, see [these instructions](http://1.bp.blogspot.com/-AgUTyA3m7v8/Tae3lHr335I/AAAAAAAAAT4/T3iyFPNt30w/s1600/thumb.php.jpeg)
- Clone this repo
- `cd` into the repo root
- `npm run show-off`

If the build process fails or your browser never ends up showing you anything, email me and I will kick-off an [ngrok session](https://ngrok.com/) for demoing purposes.


## Usage
### Blocks
The main concept behind this template builder are drag-n-droppable elements that make up an email (e.g. some text, a few images, a button). For the purposes of the documentation, we’ll refer to them as blocks.

### Placing Blocks
Blocks  can be dragged and dropped into their desired position. Valid “dropzones” are highlighted with a blue border when a block has been picked up.  When a block can be dropped, it will turn green.

![Placing Blocks](http://i.imgur.com/avzn5Xf.png?raw=true)


### Resizing Blocks
Blocks can be resized in order to make room for a sibling block. Just grab the right side of the block and drag to shrink it. Once you’ve created enough room, you’ll see a new “dropzone” highlighted in green.

![Resizing Blocks](http://i.imgur.com/ycLCWd9.png?raw=true)


### Block Options
When hovering over a block you have added to the template, three icons appear in the top-right: Edit, Clone, and Delete, respectively. It’s pretty self-explanatory what they do… but I’ll tell you about it anyways.

#### Edit
Hover over a block and press the pencil icon, this switches the toolbar to ‘Edit Block’ mode; you should see the block highlighted in a color that is debatably either pink or red. Depending on what type of block you are editing, various options will be shown to you. The blocks are reactive so the changes are reflected immediately.

![Edit Block](http://i.imgur.com/FCQuERe.png?raw=true)


#### Clone
Hover over a block and press the icon that looks like two windows on top of each other. This will duplicate that block and place it in the row below. 

#### Delete
Hover over a block and press the X icon. It’s quick, I promise the block doesn’t even feel the pain.


### Saving
By default, the template is being saved to local browser storage whenever a change is made. To disable this feature, uncheck the setting “Automatically save template” at the bottom of the toolbar.

## Room for Improvement
There's a few things that I knowingly skipped, mostly because I could probably spend the next month adding neat features. Firstly, you can't drag and drop existing blocks to reorder them. I'm sure this would be simple enough to implement given how the code works, but I just didn't get around to it. Secondly, being able to double-click a block and have it add to the bottom of the template would be handy. I'm sure there's other stuff, but those are the ones I have in my notes.









