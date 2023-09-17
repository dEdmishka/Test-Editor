<template>
  <fieldset class="fieldset">
    <select id="fontFamily" class="select font-select" @change="changeFont($event)">
      <option value="">Select family...</option>
      <option value="Arial">Arial</option>
      <option value="Helvetica">Helvetica</option>
      <option value="Times New Roman">Times New Roman</option>
      <option value="Sans serif">Sans serif</option>
      <option value="Courier New">Courier New</option>
      <option value="Verdana">Verdana</option>
      <option value="Georgia">Georgia</option>
      <option value="Palatino">Palatino</option>
      <option value="Garamond">Garamond</option>
      <option value="Comic Sans MS">Comic Sans MS</option>
      <option value="Arial Black">Arial Black</option>
      <option value="Tahoma">Tahoma</option>
      <option value="Comic Sans MS">Comic Sans MS</option>
    </select>

    <label for="myColor" class="label-color blue-hover">Color
      <input class="color input-color blue-hover" type="color" @change="chooseColor($event)" id="myColor">
    </label>
    <label for="myBackground" class="label-color blue-hover">Background
      <input class="color input-background blue-hover" type="color" @change="chooseBackground($event)"
             id="myBackground">
    </label>

    <select id="fontSize" class="select size-select" @change="changeSize($event)">
      <option value="">Select size...</option>
      <option value="10">10</option>
      <option value="12">12</option>
      <option value="14">14</option>
      <option value="16">16</option>
      <option value="18">18</option>
      <option value="20">20</option>
      <option value="22">22</option>
      <option value="24">24</option>
    </select>

    <button type="button" class="btn parser blue-hover" id="parser" @click="parse">Parse</button>
    <button type="button" class="btn reset blue-hover" id="reset" @click="reset">Clear editor</button>
  </fieldset>

  <div
      ref="editor"
      id="editor"
      contenteditable
      @input="formatParagraph($event)"
      :class="classes"
      :style="{
        width: width + 'px',
        height: height + 'px'
      }"
  ></div>
</template>

<script>
export default {
  name: "VueEditor",
  props: {
    classes: [String],
    width: [Number],
    height: [Number],
  },
  data() {
    return {
      data: [],
      currentSpan: {},
    }
  },
  methods: {
    reset() {
      this.$refs.editor.innerText = '';
    },
    parse() {
      const paragraphs = this.$refs.editor.querySelectorAll('p');

      this.data = this.parseParagraphsToObjects(paragraphs);

      console.log('Parsed Data:')
      console.log(this.data);
    },

    parseParagraphsToObjects(paragraphs) {
      const parsedObjects = [];

      paragraphs.forEach((paragraph) => {
        const spanObjects = [];

        paragraph.querySelectorAll('span').forEach((span) => {
          span = {
            text: span.textContent,
            color: span.style?.color,
            backgroundColor: span.style?.backgroundColor,
            fontSize: span.style?.fontSize,
            fontFamily: span.style?.fontFamily,
          }

          const {text: text1, ...a} = span;

          const {text: text2, ...b} = this.currentSpan;

          if (JSON.stringify(a) === JSON.stringify(b)) {
            let lastSpan = spanObjects.pop();
            lastSpan.text = lastSpan.text + ' ' + span.text
            spanObjects.push(lastSpan);
          } else {
            spanObjects.push(span);
          }

          this.currentSpan = span;
        });
        parsedObjects.push(spanObjects);
      });

      return parsedObjects;
    },

    formatParagraph(event) {
      this.$emit('update:value', this.$refs.editor.innerHTML);

      const {firstChild} = event.target;
      if (firstChild && firstChild.nodeType === 3) {
        document.execCommand('formatBlock', false, '<p>');

      } else if (this.$refs.editor.innerHTML === '<br>') this.$refs.editor.innerHTML = '';
    },

    changeFont() {
      let font = document.getElementById("fontFamily").value;

      if (font) this.changeStyleForSpan(font, 'fontFamily');

      document.getElementById('fontFamily').value = "";
    },

    changeSize(e) {
      let size = e.target.value;

      if (size) this.changeStyleForSpan(size + 'px', 'fontSize');

      document.getElementById('fontSize').value = "";
    },

    chooseColor(e) {
      this.changeStyleForSpan(e.target.value, 'color');
    },

    chooseBackground(e) {
      this.changeStyleForSpan(e.target.value, 'backgroundColor');
    },

    changeStyleForSpan(style, styleName) {
      let sel = window.getSelection();

      if (!sel.isCollapsed) {
        let range = sel.getRangeAt(0);

        let span = document.createElement('span');
        span.style[styleName] = style;
        // range.surroundContents(span);
        span.appendChild(range.extractContents());

        if (range.startContainer.parentElement.tagName === "SPAN") {
          const originalSpan = range.startContainer.parentElement
          const text = originalSpan.textContent
          const span1 = document.createElement('span');
          let span2;
          span1.style.cssText = originalSpan.style.cssText;

          const splitIndex = range.startOffset - 1;
          const part1 = text.substring(0, splitIndex + 1);
          span1.textContent = part1;

          if (text.substring(splitIndex + 1)) {
            span2 = document.createElement('span');
            span2.style.cssText = originalSpan.style.cssText;
            const part2 = text.substring(splitIndex + 1);
            span2.textContent = part2;
          }
          if (!span1.textContent) {
            // insert before other words
            span1.style.cssText += span.style.cssText;
            span.style.cssText += span1.style.cssText;
            range.startContainer.parentNode.parentNode.replaceChild(span, originalSpan);
          } else {
            // insert between words
            range.startContainer.parentNode.parentNode.replaceChild(span1, originalSpan);
            range.startContainer.appendChild(span);
            if (span2) {
              range.startContainer.appendChild(span2);
            }
          }
          // for now do not work interpolation of two or more spans, it still leaves nested
        } else {
          range.insertNode(span);
        }
        sel.removeAllRanges();
      }
    },
  },
}
</script>

<style scoped>
body {
  font: 400 16px/1.4 'serif';
}
#editor {
  border: 1px solid grey;
  height: 100px;
  width: 602px;
  margin: 0 auto 0;
  padding: 10px;
}
select {
  appearance: none;
  background-color: transparent;
  border: none;
  padding: 0 1em 0 0;
  margin: 0;
  font-family: Arial, sans-serif;
  font-size: 16px;
  line-height: inherit;
  cursor: pointer;
  z-index: 1;

  outline: none;
}
select:focus + .focus {
  position: absolute;
  top: -1px;
  left: -1px;
  right: -1px;
  bottom: -1px;
  border: 2px solid #ccc;
  border-radius: inherit;
}

select::-ms-expand {
  display: none;
}

.fieldset {
  margin: 0 auto;
  padding: 5px 0;
  width: 700px;
  border: 1px solid darkblue;
}


.fieldset > * {
  padding: 0 4px;
}

.label-color {
  font-size: 16px;
  font-family: Arial, sans-serif;
  font-weight: 500;
  padding: 8px 2px 3px 5px;
  border-radius: 10px;
  align-content: center;
  margin: 0 5px;
}

.color {
  border: none;
  outline: none;
  border-radius: 10px;
}

.btn {
  margin: 0 5px;
  text-align: center;
  padding: 5px 7px;
  font-weight: 500;
  font-size: 16px;
  border: none;
  outline: none;
  border-radius: 10px;
}

.blue-hover {
  background: #3838cc;
  color: yellow;
  cursor: pointer;
  transition: all 0.3s ease-out;
  webkit-user-select: none; /* Safari */
  -ms-user-select: none; /* IE 10 and IE 11 */
  user-select: none; /* Standard syntax */
}

.blue-hover:hover {
  background: yellow;
  color: #3838cc;
}
</style>
